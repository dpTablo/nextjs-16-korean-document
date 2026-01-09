# 에러 처리

**버전:** 16.1.1
**최종 업데이트:** 최신

## 개요
Next.js 애플리케이션의 에러는 두 가지 범주로 나뉩니다:
1. **예상되는 에러** - 정상 작동 중 발생 (폼 검증, 실패한 요청)
2. **잡히지 않은 예외** - 버그를 나타내는 예상치 못한 에러

---

## 예상되는 에러 처리

### 서버 함수

Server Functions와 함께 [`useActionState`](https://react.dev/reference/react/useActionState) 훅을 사용하세요. try/catch 블록을 사용하는 대신 예상되는 에러를 반환 값으로 모델링하세요.

**Server Action 예시:**
```typescript
'use server'

export async function createPost(prevState: any, formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')

  const res = await fetch('https://api.vercel.app/posts', {
    method: 'POST',
    body: { title, content },
  })
  const json = await res.json()

  if (!res.ok) {
    return { message: '포스트 생성 실패' }
  }
}
```

**useActionState를 사용하는 클라이언트 컴포넌트:**
```tsx
'use client'

import { useActionState } from 'react'
import { createPost } from '@/app/actions'

const initialState = {
  message: '',
}

export function Form() {
  const [state, formAction, pending] = useActionState(createPost, initialState)

  return (
    <form action={formAction}>
      <label htmlFor="title">제목</label>
      <input type="text" id="title" name="title" required />
      <label htmlFor="content">내용</label>
      <textarea id="content" name="content" required />
      {state?.message && <p aria-live="polite">{state.message}</p>}
      <button disabled={pending}>포스트 생성</button>
    </form>
  )
}
```

### 서버 컴포넌트

데이터를 페칭하고 조건부로 에러 메시지를 렌더링하거나 [`redirect`](/docs/app/api-reference/functions/redirect.md)를 사용하세요:

```tsx
export default async function Page() {
  const res = await fetch(`https://...`)
  const data = await res.json()

  if (!res.ok) {
    return '에러가 발생했습니다.'
  }

  return '...'
}
```

### Not Found 에러

[`not-found.js`](/docs/app/api-reference/file-conventions/not-found.md) 파일과 함께 [`notFound`](/docs/app/api-reference/functions/not-found.md) 함수를 사용하세요:

**페이지 컴포넌트:**
```tsx
import { getPostBySlug } from '@/lib/posts'

export default async function Page({ params }: { params: { slug: string } }) {
  const { slug } = await params
  const post = getPostBySlug(slug)

  if (!post) {
    notFound()
  }

  return <div>{post.title}</div>
}
```

**Not Found UI:**
```tsx
export default function NotFound() {
  return <div>404 - 페이지를 찾을 수 없습니다</div>
}
```

---

## 잡히지 않은 예외 처리

### 중첩 에러 경계

라우트 세그먼트 내에서 [`error.js`](/docs/app/api-reference/file-conventions/error.md) 파일을 사용하여 에러 경계를 생성하세요:

```tsx
'use client' // 에러 경계는 클라이언트 컴포넌트여야 합니다

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>문제가 발생했습니다!</h2>
      <button onClick={() => reset()}>
        다시 시도
      </button>
    </div>
  )
}
```

**핵심 사항:**
- 에러는 가장 가까운 부모 에러 경계로 버블링됩니다
- 세분화된 에러 처리를 위해 다른 레벨에 `error.tsx` 파일을 배치하세요
- 에러 경계는 이벤트 핸들러나 비동기 코드의 에러를 잡지 못합니다

### 이벤트 핸들러 에러 처리

`useState` 또는 `useReducer`를 사용하여 수동으로 에러를 잡으세요:

```tsx
'use client'

import { useState } from 'react'

export function Button() {
  const [error, setError] = useState(null)

  const handleClick = () => {
    try {
      throw new Error('예외')
    } catch (reason) {
      setError(reason)
    }
  }

  if (error) {
    /* 폴백 UI 렌더링 */
  }

  return (
    <button type="button" onClick={handleClick}>
      클릭하세요
    </button>
  )
}
```

### useTransition 사용하기

`startTransition` 내부의 처리되지 않은 에러는 가장 가까운 에러 경계로 버블링됩니다:

```tsx
'use client'

import { useTransition } from 'react'

export function Button() {
  const [pending, startTransition] = useTransition()

  const handleClick = () =>
    startTransition(() => {
      throw new Error('예외')
    })

  return (
    <button type="button" onClick={handleClick}>
      클릭하세요
    </button>
  )
}
```

### 전역 에러

[`global-error.js`](/docs/app/api-reference/file-conventions/error.md#global-error)를 사용하여 루트 레이아웃의 에러를 처리하세요:

```tsx
'use client' // 에러 경계는 클라이언트 컴포넌트여야 합니다

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <h2>문제가 발생했습니다!</h2>
        <button onClick={() => reset()}>다시 시도</button>
      </body>
    </html>
  )
}
```

**참고:** 전역 에러 UI는 자체 `<html>` 및 `<body>` 태그를 정의해야 합니다.

---

## API 참조

- [redirect](/docs/app/api-reference/functions/redirect.md)
- [error.js](/docs/app/api-reference/file-conventions/error.md)
- [notFound](/docs/app/api-reference/functions/not-found.md)
- [not-found.js](/docs/app/api-reference/file-conventions/not-found.md)
