# useActionState

## 개요

`useActionState`는 폼 action의 결과를 기반으로 상태를 업데이트할 수 있는 **React Hook**입니다. Next.js의 Server Actions와 함께 사용하면 폼 에러 처리, 로딩 상태 관리, 낙관적 업데이트 등을 쉽게 구현할 수 있습니다.

> **참고:** 이전 React Canary 버전에서는 이 API가 React DOM의 일부였으며 `useFormState`라고 불렸습니다. React 19에서 `useActionState`로 이름이 변경되었습니다.

---

## 기본 사용법

```tsx
'use client'

import { useActionState } from 'react'

const [state, formAction, isPending] = useActionState(fn, initialState, permalink?)
```

---

## API 레퍼런스

### 함수 시그니처

```tsx
useActionState(
  action: (previousState: State, formData: FormData) => State | Promise<State>,
  initialState: State,
  permalink?: string
): [state: State, formAction: (formData: FormData) => void, isPending: boolean]
```

### 매개변수

| 매개변수 | 타입 | 설명 |
|----------|------|------|
| **`action`** | `function` | 폼이 제출될 때 호출되는 함수 |
| **`initialState`** | `any` | 초기 상태 값 (직렬화 가능한 값) |
| **`permalink`** | `string` (선택) | Progressive Enhancement를 위한 URL |

#### action 함수 시그니처

```tsx
async function action(
  previousState: State,  // 이전 상태 (첫 호출 시 initialState)
  formData: FormData     // 폼 데이터
): Promise<State>
```

### 반환 값

배열로 3개의 값을 반환합니다:

| 인덱스 | 이름 | 타입 | 설명 |
|--------|------|------|------|
| `[0]` | **state** | `State` | 현재 상태 (초기에는 `initialState`) |
| `[1]` | **formAction** | `function` | `<form action>` 또는 `<button formAction>`에 전달할 action |
| `[2]` | **isPending** | `boolean` | 전환(transition)이 진행 중인지 여부 |

---

## Next.js에서의 실용적인 예제

### 1. 기본 카운터 예제

```tsx
// app/actions.ts
'use server'

export async function increment(previousState: number, formData: FormData) {
  return previousState + 1
}
```

```tsx
// app/components/Counter.tsx
'use client'

import { useActionState } from 'react'
import { increment } from '@/app/actions'

export default function Counter() {
  const [count, formAction, isPending] = useActionState(increment, 0)

  return (
    <form action={formAction}>
      <p>카운트: {count}</p>
      <button type="submit" disabled={isPending}>
        {isPending ? '증가 중...' : '증가'}
      </button>
    </form>
  )
}
```

### 2. 폼 에러 표시

```tsx
// app/actions.ts
'use server'

export async function createUser(
  previousState: { message: string } | null,
  formData: FormData
) {
  const name = formData.get('name')
  const email = formData.get('email')

  // 유효성 검사
  if (!name || name.length < 2) {
    return { message: '이름은 최소 2자 이상이어야 합니다.' }
  }

  if (!email || !email.includes('@')) {
    return { message: '올바른 이메일 주소를 입력하세요.' }
  }

  try {
    await db.users.create({ name, email })
    return { message: '사용자가 생성되었습니다!' }
  } catch (error) {
    return { message: '사용자 생성에 실패했습니다.' }
  }
}
```

```tsx
// app/components/UserForm.tsx
'use client'

import { useActionState } from 'react'
import { createUser } from '@/app/actions'

export default function UserForm() {
  const [state, formAction, isPending] = useActionState(createUser, null)

  return (
    <form action={formAction}>
      <input type="text" name="name" placeholder="이름" />
      <input type="email" name="email" placeholder="이메일" />
      <button type="submit" disabled={isPending}>
        {isPending ? '생성 중...' : '사용자 생성'}
      </button>
      {state?.message && (
        <p className="mt-2 text-sm text-red-600">{state.message}</p>
      )}
    </form>
  )
}
```

### 3. 구조화된 상태 관리

```tsx
// app/actions.ts
'use server'

type CartState = {
  success?: boolean
  message?: string
  cartSize?: number
}

export async function addToCart(
  previousState: CartState,
  formData: FormData
): Promise<CartState> {
  const itemId = formData.get('itemId')

  try {
    const cart = await db.cart.addItem(itemId)

    return {
      success: true,
      message: '장바구니에 추가되었습니다!',
      cartSize: cart.items.length,
    }
  } catch (error) {
    return {
      success: false,
      message: '장바구니 추가에 실패했습니다.',
    }
  }
}
```

```tsx
// app/components/AddToCartForm.tsx
'use client'

import { useActionState } from 'react'
import { addToCart } from '@/app/actions'

export default function AddToCartForm({
  itemId,
  itemTitle,
}: {
  itemId: string
  itemTitle: string
}) {
  const [state, formAction, isPending] = useActionState(addToCart, {})

  return (
    <form action={formAction}>
      <h2>{itemTitle}</h2>
      <input type="hidden" name="itemId" value={itemId} />

      <button type="submit" disabled={isPending}>
        {isPending ? '추가 중...' : '장바구니에 추가'}
      </button>

      {state.success && (
        <div className="toast bg-green-100 text-green-800 p-4 rounded">
          {state.message}
          <p>현재 장바구니: {state.cartSize}개 항목</p>
        </div>
      )}

      {state.success === false && (
        <div className="error bg-red-100 text-red-800 p-4 rounded">
          {state.message}
        </div>
      )}
    </form>
  )
}
```

### 4. 로그인 폼 with 검증

```tsx
// app/actions.ts
'use server'

import { redirect } from 'next/navigation'

type LoginState = {
  errors?: {
    email?: string
    password?: string
  }
  message?: string
}

export async function login(
  previousState: LoginState,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get('email') as string
  const password = formData.get('password') as string

  const errors: LoginState['errors'] = {}

  if (!email) {
    errors.email = '이메일을 입력하세요.'
  }

  if (!password || password.length < 8) {
    errors.password = '비밀번호는 최소 8자 이상이어야 합니다.'
  }

  if (Object.keys(errors).length > 0) {
    return { errors }
  }

  try {
    await signIn(email, password)
    redirect('/dashboard')
  } catch (error) {
    return {
      message: '로그인에 실패했습니다. 이메일과 비밀번호를 확인하세요.',
    }
  }
}
```

```tsx
// app/components/LoginForm.tsx
'use client'

import { useActionState } from 'react'
import { login } from '@/app/actions'

export default function LoginForm() {
  const [state, formAction, isPending] = useActionState(login, {})

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          type="email"
          name="email"
          disabled={isPending}
        />
        {state.errors?.email && (
          <p className="text-red-500 text-sm">{state.errors.email}</p>
        )}
      </div>

      <div>
        <label htmlFor="password">비밀번호</label>
        <input
          id="password"
          type="password"
          name="password"
          disabled={isPending}
        />
        {state.errors?.password && (
          <p className="text-red-500 text-sm">{state.errors.password}</p>
        )}
      </div>

      {state.message && (
        <p className="text-red-500">{state.message}</p>
      )}

      <button type="submit" disabled={isPending}>
        {isPending ? '로그인 중...' : '로그인'}
      </button>
    </form>
  )
}
```

### 5. 검색 폼

```tsx
// app/actions.ts
'use server'

type SearchState = {
  results?: Array<{ id: string; title: string }>
  query?: string
  isEmpty?: boolean
}

export async function search(
  previousState: SearchState,
  formData: FormData
): Promise<SearchState> {
  const query = formData.get('query') as string

  if (!query || query.trim() === '') {
    return { isEmpty: true }
  }

  const results = await db.posts.search(query)

  return {
    results,
    query,
    isEmpty: results.length === 0,
  }
}
```

```tsx
// app/components/SearchForm.tsx
'use client'

import { useActionState } from 'react'
import { search } from '@/app/actions'

export default function SearchForm() {
  const [state, formAction, isPending] = useActionState(search, {})

  return (
    <div>
      <form action={formAction}>
        <input
          type="text"
          name="query"
          placeholder="검색..."
          disabled={isPending}
        />
        <button type="submit" disabled={isPending}>
          {isPending ? '검색 중...' : '검색'}
        </button>
      </form>

      {state.isEmpty && <p>검색 결과가 없습니다.</p>}

      {state.results && (
        <div>
          <h3>"{state.query}" 검색 결과:</h3>
          <ul>
            {state.results.map((result) => (
              <li key={result.id}>{result.title}</li>
            ))}
          </ul>
        </div>
      )}
    </div>
  )
}
```

### 6. Progressive Enhancement (permalink 사용)

```tsx
// app/components/CommentForm.tsx
'use client'

import { useActionState } from 'react'
import { addComment } from '@/app/actions'

export default function CommentForm({ postId }: { postId: string }) {
  const [state, formAction, isPending] = useActionState(
    addComment,
    { message: '' },
    `/posts/${postId}` // JavaScript 로드 전에도 작동
  )

  return (
    <form action={formAction}>
      <input type="hidden" name="postId" value={postId} />
      <textarea name="comment" disabled={isPending} />
      <button type="submit" disabled={isPending}>
        {isPending ? '댓글 작성 중...' : '댓글 작성'}
      </button>
      {state.message && <p>{state.message}</p>}
    </form>
  )
}
```

---

## 고급 패턴

### 1. 여러 액션 처리

```tsx
'use client'

import { useActionState } from 'react'

type PostState = {
  status?: 'saved' | 'published' | 'error'
  message?: string
}

export default function PostEditor() {
  const [state, formAction, isPending] = useActionState(
    handlePost,
    { status: undefined }
  )

  return (
    <form action={formAction}>
      <textarea name="content" />

      <button
        type="submit"
        name="action"
        value="save"
        disabled={isPending}
      >
        저장
      </button>

      <button
        type="submit"
        name="action"
        value="publish"
        disabled={isPending}
      >
        발행
      </button>

      {state.status === 'saved' && <p>✓ 저장되었습니다</p>}
      {state.status === 'published' && <p>✓ 발행되었습니다</p>}
      {state.status === 'error' && <p>✗ {state.message}</p>}
    </form>
  )
}
```

### 2. 낙관적 업데이트와 함께 사용

```tsx
'use client'

import { useActionState, useOptimistic } from 'react'

type Todo = { id: string; text: string; completed: boolean }

export default function TodoList({ todos }: { todos: Todo[] }) {
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

## 주요 포인트 및 주의사항

### ⚠️ action 함수의 첫 번째 매개변수

`useActionState`를 사용할 때, action 함수는 **추가 첫 번째 매개변수**(현재 상태)를 받습니다. FormData는 **두 번째 매개변수**가 됩니다:

```tsx
// ❌ 잘못된 방법
async function action(formData: FormData) {
  const name = formData.get('name') // 작동하지 않음!
}

// ✅ 올바른 방법
async function action(previousState: State, formData: FormData) {
  const name = formData.get('name') // 작동함
}
```

### ⚠️ Server Components와 함께 사용

React Server Components와 함께 사용하면 JavaScript가 실행되기 전에도 상호작용 가능한 폼을 만들 수 있습니다 (Progressive Enhancement).

### ⚠️ 직렬화 가능한 상태만 사용

`initialState`와 action이 반환하는 상태는 직렬화 가능해야 합니다:

```tsx
// ✅ 직렬화 가능
const initialState = { count: 0, message: '' }

// ❌ 직렬화 불가능
const initialState = { callback: () => {} }
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **구조화된 상태 사용**
   ```tsx
   type State = {
     success?: boolean
     message?: string
     errors?: Record<string, string>
   }
   ```

2. **명확한 에러 메시지**
   ```tsx
   return {
     errors: {
       email: '이메일 형식이 올바르지 않습니다.',
       password: '비밀번호는 8자 이상이어야 합니다.',
     },
   }
   ```

3. **isPending으로 버튼 비활성화**
   ```tsx
   <button disabled={isPending}>
     {isPending ? '처리 중...' : '제출'}
   </button>
   ```

### ❌ 피해야 할 사항

1. **직렬화 불가능한 값 반환**
   ```tsx
   // ❌ 함수는 직렬화 불가능
   return { callback: () => {} }
   ```

2. **FormData 순서 혼동**
   ```tsx
   // ❌ previousState를 빼먹음
   async function action(formData: FormData) { }

   // ✅ 올바름
   async function action(previousState: State, formData: FormData) { }
   ```

---

## 마이그레이션 가이드

### useFormState에서 마이그레이션

```tsx
// Before (React < 19)
import { useFormState } from 'react-dom'

const [state, formAction] = useFormState(action, initialState)

// After (React 19+)
import { useActionState } from 'react'

const [state, formAction, isPending] = useActionState(action, initialState)
```

주요 변경사항:
- `react-dom`에서 `react`로 import 경로 변경
- Hook 이름이 `useFormState`에서 `useActionState`로 변경
- 세 번째 반환 값 `isPending` 추가

---

## 타입 정의

```typescript
type ActionState<State> = [
  state: State,
  formAction: (formData: FormData) => void,
  isPending: boolean
]

function useActionState<State>(
  action: (previousState: State, formData: FormData) => State | Promise<State>,
  initialState: State,
  permalink?: string
): ActionState<State>
```

---

## 관련 API

- [`useFormStatus`](./useFormStatus.md) - 폼 제출 상태 추적
- [`useOptimistic`](./useOptimistic.md) - 낙관적 UI 업데이트
- [Server Actions](../../../guides/forms.md) - 폼 처리 가이드
- [`revalidatePath`](../revalidatePath.md) - 경로 재검증

---

## 추가 리소스

- [React 공식 문서 - useActionState](https://react.dev/reference/react/useActionState)
- [Next.js Forms 가이드](../../../guides/forms.md)
- [Server Actions 패턴](../../../guides/server-actions-patterns.md)

---

## 버전 정보

- **도입 버전:** React 19.0.0 (`useFormState`로 시작)
- **이름 변경:** React 19.0.0 (`useActionState`로 변경)
- **Next.js 지원:** Next.js 14.0.0+
- **현재 상태:** Stable

---

## 요약

- **용도:** 폼 action 결과 기반 상태 관리
- **반환 값:** `[state, formAction, isPending]`
- **주 사용처:** 폼 에러 처리, 검증, 로딩 상태
- **Next.js 통합:** Server Actions와 완벽 호환
- **주의사항:** action 함수의 첫 번째 매개변수는 previousState
- **이전 이름:** `useFormState` (React 19 이전)
