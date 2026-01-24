# 데이터 업데이트 (Updating Data)

Next.js에서 React의 [Server Functions](https://react.dev/reference/rsc/server-functions)를 사용하여 데이터를 업데이트할 수 있습니다. 이 페이지에서는 Server Functions를 생성하고 호출하는 방법을 다룹니다.

## Server Functions란?

**Server Function**은 서버에서 실행되는 비동기 함수입니다. 네트워크 요청을 통해 클라이언트에서 호출할 수 있으므로 필연적으로 비동기입니다.

`action` 또는 mutation 컨텍스트에서는 **Server Actions**라고 합니다.

> **알아두면 좋은 점:** Server Action은 폼 제출과 mutation을 처리하는 특정 방식으로 사용되는 Server Function입니다. Server Function이 더 넓은 개념입니다.

관례적으로 Server Action은 [`startTransition`](https://react.dev/reference/react/startTransition)과 함께 사용되는 async 함수입니다. 이것은 함수가 다음과 같이 사용될 때 자동으로 발생합니다:

- `action` prop을 사용하여 `<form>`에 전달될 때
- `formAction` prop을 사용하여 `<button>`에 전달될 때

Next.js에서 Server Actions는 프레임워크의 [캐싱](/app-router/guides/caching.md) 아키텍처와 통합됩니다. action이 호출되면 Next.js는 단일 서버 왕복으로 업데이트된 UI와 새 데이터를 모두 반환할 수 있습니다.

내부적으로 actions는 `POST` 메서드를 사용하며, 이 HTTP 메서드만 actions를 호출할 수 있습니다.

## Server Functions 생성하기

Server Function은 [`use server`](https://react.dev/reference/rsc/use-server) 지시어를 사용하여 정의할 수 있습니다. **비동기** 함수의 맨 위에 지시어를 배치하거나 별도의 파일 맨 위에 배치하여 모든 exports를 Server Functions로 표시할 수 있습니다.

### 예제 - 파일 수준 지시어

```ts filename="app/lib/actions.ts"
'use server'

export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')

  // 데이터 업데이트
  // 캐시 재검증
}

export async function deletePost(formData: FormData) {
  const id = formData.get('id')

  // 데이터 업데이트
  // 캐시 재검증
}
```

### 서버 컴포넌트

Server Functions는 함수 본문 맨 위에 `"use server"` 지시어를 추가하여 서버 컴포넌트에서 인라인으로 정의할 수 있습니다:

```tsx filename="app/page.tsx"
export default function Page() {
  // Server Action
  async function createPost(formData: FormData) {
    'use server'
    // ...
  }

  return <></>
}
```

> **알아두면 좋은 점:** 서버 컴포넌트는 기본적으로 점진적 향상을 지원합니다. 즉, JavaScript가 아직 로드되지 않았거나 비활성화된 경우에도 Server Actions를 호출하는 폼이 제출됩니다.

### 클라이언트 컴포넌트

클라이언트 컴포넌트에서는 Server Functions를 정의할 수 없습니다. 하지만 파일 맨 위에 `"use server"` 지시어가 있는 파일에서 가져와서 호출할 수 있습니다:

```ts filename="app/actions.ts"
'use server'

export async function createPost() {}
```

```tsx filename="app/ui/button.tsx"
'use client'

import { createPost } from '@/app/actions'

export function Button() {
  return <button formAction={createPost}>생성</button>
}
```

> **알아두면 좋은 점:** 클라이언트 컴포넌트에서 Server Actions를 호출하는 폼은 JavaScript가 아직 로드되지 않은 경우 제출을 대기열에 넣고 hydration을 우선시합니다. hydration 후에는 폼 제출 시 브라우저가 새로고침되지 않습니다.

### Props로 Actions 전달하기

action을 클라이언트 컴포넌트에 prop으로 전달할 수 있습니다:

```tsx filename="app/client-component.tsx"
'use client'

export default function ClientComponent({
  updateItemAction,
}: {
  updateItemAction: (formData: FormData) => void
}) {
  return <form action={updateItemAction}>{/* ... */}</form>
}
```

## Server Functions 호출하기

Server Function을 호출하는 두 가지 주요 방법이 있습니다:

1. 서버 및 클라이언트 컴포넌트의 **Forms**
2. 클라이언트 컴포넌트의 **Event Handlers** 및 **useEffect**

> **알아두면 좋은 점:** Server Functions는 서버 측 mutations을 위해 설계되었습니다. 클라이언트는 현재 한 번에 하나씩 dispatch하고 await합니다. 병렬 데이터 가져오기가 필요한 경우 서버 컴포넌트에서 데이터 가져오기를 사용하거나 단일 Server Function 또는 Route Handler 내에서 병렬 작업을 수행하세요.

### Forms

React는 HTML `<form>` 요소를 확장하여 `action` prop으로 Server Function을 호출할 수 있게 합니다.

폼에서 호출되면 함수는 자동으로 [`FormData`](https://developer.mozilla.org/docs/Web/API/FormData/FormData) 객체를 받습니다:

```tsx filename="app/ui/form.tsx"
import { createPost } from '@/app/actions'

export function Form() {
  return (
    <form action={createPost}>
      <input type="text" name="title" />
      <input type="text" name="content" />
      <button type="submit">생성</button>
    </form>
  )
}
```

```ts filename="app/actions.ts"
'use server'

export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')

  // 데이터 업데이트
  // 캐시 재검증
}
```

### Event Handlers

`onClick`과 같은 이벤트 핸들러를 사용하여 클라이언트 컴포넌트에서 Server Function을 호출할 수 있습니다:

```tsx filename="app/like-button.tsx"
'use client'

import { incrementLike } from './actions'
import { useState } from 'react'

export default function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes)

  return (
    <>
      <p>총 좋아요: {likes}</p>
      <button
        onClick={async () => {
          const updatedLikes = await incrementLike()
          setLikes(updatedLikes)
        }}
      >
        좋아요
      </button>
    </>
  )
}
```

## 예제

### 대기 상태 표시하기

React의 [`useActionState`](https://react.dev/reference/react/useActionState) 훅을 사용하여 Server Function 실행 중 로딩 표시기를 보여줍니다:

```tsx filename="app/ui/button.tsx"
'use client'

import { useActionState, startTransition } from 'react'
import { createPost } from '@/app/actions'
import { LoadingSpinner } from '@/app/ui/loading-spinner'

export function Button() {
  const [state, action, pending] = useActionState(createPost, false)

  return (
    <button onClick={() => startTransition(action)}>
      {pending ? <LoadingSpinner /> : '게시물 생성'}
    </button>
  )
}
```

### 새로고침

mutation 후 `next/cache`의 [`refresh`](/app-router/api-reference/functions/refresh.md)를 사용하여 현재 페이지를 새로고침하여 최신 데이터를 표시합니다:

```ts filename="app/lib/actions.ts"
'use server'

import { refresh } from 'next/cache'

export async function updatePost(formData: FormData) {
  // 데이터 업데이트
  // ...

  refresh()
}
```

이것은 클라이언트 라우터를 새로고침하여 UI가 최신 상태를 반영하도록 합니다. `refresh()` 함수는 태그가 지정된 데이터를 재검증하지 않습니다. 대신 [`updateTag`](/app-router/api-reference/functions/updateTag.md) 또는 [`revalidateTag`](/app-router/api-reference/functions/revalidateTag.md)를 사용하세요.

### 재검증

업데이트를 수행한 후 [`revalidatePath`](/app-router/api-reference/functions/revalidatePath.md) 또는 [`revalidateTag`](/app-router/api-reference/functions/revalidateTag.md)를 호출하여 Next.js 캐시를 재검증합니다:

```ts filename="app/lib/actions.ts"
import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  'use server'
  // 데이터 업데이트
  // ...

  revalidatePath('/posts')
}
```

### 리디렉션

[`redirect`](/app-router/api-reference/functions/redirect.md)를 사용하여 업데이트 후 사용자를 다른 페이지로 리디렉션합니다:

```ts filename="app/lib/actions.ts"
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  // 데이터 업데이트
  // ...

  revalidatePath('/posts')
  redirect('/posts')
}
```

> **중요:** `redirect`를 호출하면 프레임워크에서 처리하는 제어 흐름 예외가 발생합니다. 그 이후의 코드는 실행되지 않습니다. 새로운 데이터가 필요한 경우 미리 `revalidatePath` 또는 `revalidateTag`를 호출하세요.

### 쿠키

[`cookies`](/app-router/api-reference/functions/cookies.md) API를 사용하여 Server Action 내에서 쿠키를 가져오고, 설정하고, 삭제합니다:

```ts filename="app/actions.ts"
'use server'

import { cookies } from 'next/headers'

export async function exampleAction() {
  const cookieStore = await cookies()

  // 쿠키 가져오기
  cookieStore.get('name')?.value

  // 쿠키 설정
  cookieStore.set('name', 'Delba')

  // 쿠키 삭제
  cookieStore.delete('name')
}
```

쿠키를 설정하거나 삭제하면 Next.js는 서버에서 현재 페이지와 해당 레이아웃을 다시 렌더링하여 **UI가 새 쿠키 값을 반영**합니다.

> **알아두면 좋은 점:** 서버 업데이트는 현재 React 트리에 적용되어 필요에 따라 컴포넌트를 다시 렌더링하거나 마운트/언마운트합니다. 다시 렌더링된 컴포넌트의 클라이언트 상태는 유지되며, 의존성이 변경된 경우 effects가 다시 실행됩니다.

### useEffect

React [`useEffect`](https://react.dev/reference/react/useEffect) 훅을 사용하여 컴포넌트가 마운트되거나 의존성이 변경될 때 Server Action을 호출합니다:

```tsx filename="app/view-count.tsx"
'use client'

import { incrementViews } from './actions'
import { useState, useEffect, useTransition } from 'react'

export default function ViewCount({ initialViews }: { initialViews: number }) {
  const [views, setViews] = useState(initialViews)
  const [isPending, startTransition] = useTransition()

  useEffect(() => {
    startTransition(async () => {
      const updatedViews = await incrementViews()
      setViews(updatedViews)
    })
  }, [])

  // `isPending`을 사용하여 사용자에게 피드백을 제공할 수 있습니다
  return <p>총 조회수: {views}</p>
}
```

## API 참조

- [revalidatePath](/app-router/api-reference/functions/revalidatePath.md) - revalidatePath 함수의 API 참조
- [revalidateTag](/app-router/api-reference/functions/revalidateTag.md) - revalidateTag 함수의 API 참조
- [redirect](/app-router/api-reference/functions/redirect.md) - redirect 함수의 API 참조
