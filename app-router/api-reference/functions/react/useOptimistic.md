# useOptimistic

## 개요

`useOptimistic`는 비동기 작업이 진행되는 동안 UI를 **낙관적으로(optimistically) 업데이트**할 수 있게 해주는 **React Hook**입니다. 네트워크 요청이나 다른 비동기 작업 중에도 즉각적인 UI 피드백을 제공하여 앱을 더 빠르고 반응적으로 느끼게 만듭니다.

> **참고:** Next.js의 Server Actions와 함께 사용하면 최상의 사용자 경험을 제공할 수 있습니다.

---

## 기본 사용법

```tsx
'use client'

import { useOptimistic } from 'react'

const [optimisticState, addOptimistic] = useOptimistic(state, updateFn)
```

---

## API 레퍼런스

### 함수 시그니처

```tsx
useOptimistic<State, OptimisticValue>(
  state: State,
  updateFn: (currentState: State, optimisticValue: OptimisticValue) => State
): [optimisticState: State, addOptimistic: (value: OptimisticValue) => void]
```

### 매개변수

| 매개변수 | 타입 | 설명 |
|----------|------|------|
| **`state`** | `State` | 초기값 및 작업이 진행 중이지 않을 때 반환되는 값 |
| **`updateFn`** | `function` | 현재 상태와 낙관적 값을 받아 병합된 낙관적 상태를 반환하는 순수 함수 |

#### updateFn 시그니처

```tsx
updateFn(currentState: State, optimisticValue: OptimisticValue): State
```

- **`currentState`**: 현재 상태
- **`optimisticValue`**: `addOptimistic`에 전달된 낙관적 값
- **반환 값**: 병합된 낙관적 상태

### 반환 값

배열로 2개의 값을 반환합니다:

| 인덱스 | 이름 | 타입 | 설명 |
|--------|------|------|------|
| `[0]` | **optimisticState** | `State` | 낙관적 상태 (작업 진행 중이 아니면 `state`와 동일) |
| `[1]` | **addOptimistic** | `function` | 낙관적 업데이트를 트리거하는 디스패치 함수 |

---

## Next.js에서의 실용적인 예제

### 1. 메시지 전송 (기본 예제)

```tsx
// app/actions.ts
'use server'

export async function sendMessage(formData: FormData) {
  const message = formData.get('message') as string

  // 데이터베이스에 메시지 저장 (시간이 걸림)
  await db.messages.create({ text: message })

  // 캐시 재검증
  revalidatePath('/chat')
}
```

```tsx
// app/components/MessageThread.tsx
'use client'

import { useOptimistic, useRef, startTransition } from 'react'
import { sendMessage } from '@/app/actions'

type Message = {
  text: string
  sending?: boolean
}

export default function MessageThread({
  messages,
}: {
  messages: Message[]
}) {
  const formRef = useRef<HTMLFormElement>(null)

  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage: string) => [
      ...state,
      {
        text: newMessage,
        sending: true, // 전송 중 표시
      },
    ]
  )

  async function formAction(formData: FormData) {
    const message = formData.get('message') as string

    // 낙관적 업데이트
    addOptimisticMessage(message)

    // 폼 초기화
    formRef.current?.reset()

    // 서버 액션 실행
    startTransition(async () => {
      await sendMessage(formData)
    })
  }

  return (
    <div>
      {optimisticMessages.map((message, index) => (
        <div key={index} className="message">
          {message.text}
          {message.sending && (
            <small className="text-gray-500"> (전송 중...)</small>
          )}
        </div>
      ))}

      <form action={formAction} ref={formRef}>
        <input
          type="text"
          name="message"
          placeholder="메시지를 입력하세요..."
        />
        <button type="submit">전송</button>
      </form>
    </div>
  )
}
```

### 2. 좋아요 버튼

```tsx
// app/actions.ts
'use server'

export async function likePost(postId: string) {
  await db.posts.update({
    where: { id: postId },
    data: { likes: { increment: 1 } },
  })

  revalidatePath('/posts')
}
```

```tsx
// app/components/LikeButton.tsx
'use client'

import { useOptimistic, startTransition } from 'react'
import { likePost } from '@/app/actions'

export default function LikeButton({
  postId,
  initialLikes,
}: {
  postId: string
  initialLikes: number
}) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    initialLikes,
    (state, amount: number) => state + amount
  )

  async function handleLike() {
    // 즉시 UI 업데이트
    addOptimisticLike(1)

    // 서버에 요청
    startTransition(async () => {
      await likePost(postId)
    })
  }

  return (
    <button onClick={handleLike}>
      👍 {optimisticLikes}
    </button>
  )
}
```

### 3. 할 일 목록 (Todo List)

```tsx
// app/actions.ts
'use server'

export async function addTodo(formData: FormData) {
  const text = formData.get('text') as string

  const todo = await db.todos.create({
    data: { text, completed: false },
  })

  revalidatePath('/todos')
  return todo
}

export async function toggleTodo(id: string) {
  const todo = await db.todos.findUnique({ where: { id } })

  await db.todos.update({
    where: { id },
    data: { completed: !todo.completed },
  })

  revalidatePath('/todos')
}
```

```tsx
// app/components/TodoList.tsx
'use client'

import { useOptimistic, useRef, startTransition } from 'react'
import { addTodo, toggleTodo } from '@/app/actions'

type Todo = {
  id: string
  text: string
  completed: boolean
}

export default function TodoList({ todos }: { todos: Todo[] }) {
  const formRef = useRef<HTMLFormElement>(null)

  const [optimisticTodos, updateOptimisticTodos] = useOptimistic(
    todos,
    (state, action: { type: 'add' | 'toggle'; todo?: Todo; id?: string }) => {
      if (action.type === 'add' && action.todo) {
        return [...state, action.todo]
      }
      if (action.type === 'toggle' && action.id) {
        return state.map((todo) =>
          todo.id === action.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      }
      return state
    }
  )

  async function handleAddTodo(formData: FormData) {
    const text = formData.get('text') as string

    // 낙관적 업데이트
    const tempTodo: Todo = {
      id: crypto.randomUUID(),
      text,
      completed: false,
    }
    updateOptimisticTodos({ type: 'add', todo: tempTodo })

    formRef.current?.reset()

    startTransition(async () => {
      await addTodo(formData)
    })
  }

  async function handleToggle(id: string) {
    // 낙관적 업데이트
    updateOptimisticTodos({ type: 'toggle', id })

    startTransition(async () => {
      await toggleTodo(id)
    })
  }

  return (
    <div>
      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => handleToggle(todo.id)}
            />
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none',
              }}
            >
              {todo.text}
            </span>
          </li>
        ))}
      </ul>

      <form action={handleAddTodo} ref={formRef}>
        <input type="text" name="text" placeholder="할 일 추가..." />
        <button type="submit">추가</button>
      </form>
    </div>
  )
}
```

### 4. 장바구니

```tsx
// app/actions.ts
'use server'

export async function addToCart(productId: string) {
  await db.cartItems.create({
    data: { productId, quantity: 1 },
  })

  revalidatePath('/cart')
}
```

```tsx
// app/components/AddToCartButton.tsx
'use client'

import { useOptimistic, startTransition } from 'react'
import { addToCart } from '@/app/actions'

type CartItem = {
  id: string
  productId: string
  quantity: number
}

export default function CartPage({
  initialItems,
}: {
  initialItems: CartItem[]
}) {
  const [optimisticItems, addOptimisticItem] = useOptimistic(
    initialItems,
    (state, newItem: CartItem) => [...state, newItem]
  )

  async function handleAddToCart(productId: string) {
    // 낙관적 업데이트
    addOptimisticItem({
      id: crypto.randomUUID(),
      productId,
      quantity: 1,
    })

    startTransition(async () => {
      await addToCart(productId)
    })
  }

  return (
    <div>
      <h2>장바구니 ({optimisticItems.length})</h2>
      <ul>
        {optimisticItems.map((item) => (
          <li key={item.id}>제품 {item.productId}</li>
        ))}
      </ul>

      <button onClick={() => handleAddToCart('product-123')}>
        장바구니에 추가
      </button>
    </div>
  )
}
```

### 5. 댓글 시스템

```tsx
// app/actions.ts
'use server'

export async function addComment(postId: string, formData: FormData) {
  const text = formData.get('text') as string
  const user = await getCurrentUser()

  const comment = await db.comments.create({
    data: {
      postId,
      text,
      userId: user.id,
      createdAt: new Date(),
    },
  })

  revalidatePath(`/posts/${postId}`)
  return comment
}
```

```tsx
// app/components/Comments.tsx
'use client'

import { useOptimistic, useRef, startTransition } from 'react'
import { addComment } from '@/app/actions'

type Comment = {
  id: string
  text: string
  userId: string
  userName: string
  pending?: boolean
}

export default function Comments({
  postId,
  comments,
  currentUser,
}: {
  postId: string
  comments: Comment[]
  currentUser: { id: string; name: string }
}) {
  const formRef = useRef<HTMLFormElement>(null)

  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (state, newComment: Comment) => [...state, newComment]
  )

  async function handleSubmit(formData: FormData) {
    const text = formData.get('text') as string

    // 낙관적 업데이트
    addOptimisticComment({
      id: crypto.randomUUID(),
      text,
      userId: currentUser.id,
      userName: currentUser.name,
      pending: true,
    })

    formRef.current?.reset()

    startTransition(async () => {
      await addComment(postId, formData)
    })
  }

  return (
    <div>
      <h3>댓글</h3>

      <div className="comments">
        {optimisticComments.map((comment) => (
          <div
            key={comment.id}
            className={comment.pending ? 'opacity-50' : ''}
          >
            <strong>{comment.userName}</strong>
            <p>{comment.text}</p>
            {comment.pending && <small>전송 중...</small>}
          </div>
        ))}
      </div>

      <form action={handleSubmit} ref={formRef}>
        <textarea name="text" placeholder="댓글을 입력하세요..." />
        <button type="submit">댓글 작성</button>
      </form>
    </div>
  )
}
```

### 6. 북마크 토글

```tsx
// app/components/BookmarkButton.tsx
'use client'

import { useOptimistic, startTransition } from 'react'
import { toggleBookmark } from '@/app/actions'

export default function BookmarkButton({
  postId,
  initialBookmarked,
}: {
  postId: string
  initialBookmarked: boolean
}) {
  const [optimisticBookmarked, setOptimisticBookmarked] = useOptimistic(
    initialBookmarked,
    (state) => !state
  )

  async function handleToggle() {
    setOptimisticBookmarked(!optimisticBookmarked)

    startTransition(async () => {
      await toggleBookmark(postId)
    })
  }

  return (
    <button onClick={handleToggle}>
      {optimisticBookmarked ? '⭐ 북마크됨' : '☆ 북마크'}
    </button>
  )
}
```

---

## 주요 포인트

### 1. 순수 함수 (Pure Function)

`updateFn`은 **순수 함수**여야 합니다. 부수 효과(side effects)가 없어야 합니다:

```tsx
// ✅ 순수 함수
const [optimisticTodos, addOptimistic] = useOptimistic(
  todos,
  (state, newTodo) => [...state, newTodo] // 새 배열 반환
)

// ❌ 부수 효과 있음
const [optimisticTodos, addOptimistic] = useOptimistic(
  todos,
  (state, newTodo) => {
    state.push(newTodo) // 원본 배열 수정 (X)
    console.log('added') // 부수 효과 (X)
    return state
  }
)
```

### 2. startTransition과 함께 사용

낙관적 업데이트는 일반적으로 `startTransition`과 함께 사용됩니다:

```tsx
startTransition(async () => {
  await serverAction(data)
})
```

### 3. 자동 롤백

서버에서 실제 데이터가 반환되면 낙관적 상태는 자동으로 실제 상태로 대체됩니다.

---

## 베스트 프랙티스

### ✅ 권장사항

1. **즉각적인 피드백 제공**
   ```tsx
   addOptimisticMessage(newMessage)
   // UI가 즉시 업데이트됨
   ```

2. **"전송 중" 표시 추가**
   ```tsx
   {
     text: message,
     sending: true // 상태 표시
   }
   ```

3. **낙관적 업데이트 후 폼 초기화**
   ```tsx
   addOptimistic(data)
   formRef.current?.reset()
   ```

4. **startTransition으로 감싸기**
   ```tsx
   startTransition(async () => {
     await serverAction()
   })
   ```

### ❌ 피해야 할 사항

1. **updateFn에 부수 효과 포함**
   ```tsx
   // ❌ 부수 효과
   (state, value) => {
     console.log(value) // 부수 효과
     return [...state, value]
   }
   ```

2. **복잡한 로직 포함**
   ```tsx
   // ❌ 너무 복잡
   (state, value) => {
     // 많은 조건문과 복잡한 변환
     // ...
   }
   ```

3. **낙관적 업데이트만 하고 서버 액션 호출 안함**
   ```tsx
   // ❌ 서버와 동기화 안됨
   addOptimistic(data)
   // 서버 액션 호출 없음!
   ```

---

## useActionState와 함께 사용

```tsx
'use client'

import { useOptimistic, useActionState } from 'react'

export default function TodoForm({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  )

  const [state, formAction] = useActionState(addTodo, null)

  return (
    <div>
      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>

      <form
        action={async (formData) => {
          const text = formData.get('text')
          addOptimisticTodo({
            id: crypto.randomUUID(),
            text,
            completed: false,
          })
          await formAction(formData)
        }}
      >
        <input type="text" name="text" />
        <button type="submit">추가</button>
      </form>
    </div>
  )
}
```

---

## 타입 정의

```typescript
type OptimisticUpdate<State, OptimisticValue> = [
  optimisticState: State,
  addOptimistic: (value: OptimisticValue) => void
]

function useOptimistic<State, OptimisticValue = State>(
  state: State,
  updateFn: (currentState: State, optimisticValue: OptimisticValue) => State
): OptimisticUpdate<State, OptimisticValue>
```

---

## 관련 API

- [`useActionState`](./useActionState.md) - 폼 상태 관리
- [`useFormStatus`](./useFormStatus.md) - 폼 제출 상태 추적
- [Server Actions](../../../guides/forms.md) - 폼 처리 가이드
- [`startTransition`](https://react.dev/reference/react/startTransition) - React 전환

---

## 추가 리소스

- [React 공식 문서 - useOptimistic](https://react.dev/reference/react/useOptimistic)
- [Next.js Forms 가이드](../../../guides/forms.md)
- [Server Actions 패턴](../../../guides/server-actions-patterns.md)

---

## 버전 정보

- **도입 버전:** React 19.0.0
- **Next.js 지원:** Next.js 14.0.0+
- **현재 상태:** Stable

---

## 요약

- **용도:** 비동기 작업 중 낙관적 UI 업데이트
- **반환 값:** `[optimisticState, addOptimistic]`
- **주 사용처:** 메시지 전송, 좋아요, Todo, 장바구니
- **장점:** 즉각적인 UI 피드백, 더 나은 UX
- **제약:** updateFn은 순수 함수여야 함
- **권장사항:** startTransition과 함께 사용
- **자동 롤백:** 서버 응답 시 자동으로 실제 상태로 업데이트
