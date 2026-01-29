---
원문: https://nextjs.org/docs/app/guides/server-actions
버전: 16.1.6
---

# Server Actions 고급 패턴

## 개요

**Server Functions** (액션/뮤테이션 컨텍스트에서는 **Server Actions**라고도 함)는 서버에서 실행되며 클라이언트에서 네트워크 요청을 통해 호출할 수 있는 비동기 함수입니다.

---

## Server Functions 생성

### `use server` 지시어 사용

비동기 함수 또는 파일 상단에 `'use server'`를 배치합니다:

```ts
// 파일 레벨
'use server'
export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')
  // 데이터 업데이트
  // 캐시 재검증
}
```

### Server Components에서

Server Components에 인라인 Server Actions를 직접 작성:

```tsx
export default function Page() {
  async function createPost(formData: FormData) {
    'use server'
    // ...
  }
  return <form action={createPost}>{/* ... */}</form>
}
```

**장점**: Server Actions가 있는 폼은 JavaScript가 비활성화되어도 제출됩니다 (점진적 향상).

### Client Components에서

Client Components에서는 Server Functions를 정의할 수 없지만 가져올 수 있습니다:

```ts
// app/actions.ts
'use server'
export async function createPost() {}
```

```tsx
// app/ui/button.tsx
'use client'
import { createPost } from '@/app/actions'

export function Button() {
  return <button formAction={createPost}>생성</button>
}
```

**참고**: Client Components에서 JS가 로드되지 않으면 폼 제출이 큐에 대기하고, 하이드레이션 후 페이지 새로고침 없이 실행됩니다.

### Props로 Actions 전달

```tsx
<ClientComponent updateItemAction={updateItem} />

export default function ClientComponent({ updateItemAction }) {
  return <form action={updateItemAction}>{/* ... */}</form>
}
```

---

## Server Functions 호출

### 1. Forms (권장)

가장 일반적인 패턴. 폼은 자동으로 `FormData`를 전달합니다:

```tsx
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

```ts
'use server'
export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')
  // 데이터 업데이트
  // 캐시 재검증
}
```

### 2. 이벤트 핸들러

`onClick` 및 기타 이벤트를 통해 Server Actions 호출:

```tsx
'use client'
import { incrementLike } from './actions'
import { useState } from 'react'

export default function LikeButton({ initialLikes }) {
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

### 3. useEffect

컴포넌트 마운트 시 또는 의존성 변경 시 뮤테이션 트리거:

```tsx
'use client'
import { incrementViews } from './actions'
import { useState, useEffect, useTransition } from 'react'

export default function ViewCount({ initialViews }) {
  const [views, setViews] = useState(initialViews)
  const [isPending, startTransition] = useTransition()

  useEffect(() => {
    startTransition(async () => {
      const updatedViews = await incrementViews()
      setViews(updatedViews)
    })
  }, [])

  return <p>총 조회수: {views}</p>
}
```

---

## 모범 사례 및 패턴

### 대기 상태 표시

`useActionState` hook을 사용하여 로딩 상태 추적:

```tsx
'use client'
import { useActionState, startTransition } from 'react'
import { createPost } from '@/app/actions'
import { LoadingSpinner } from '@/app/ui/loading-spinner'

export function Button() {
  const [state, action, pending] = useActionState(createPost, false)

  return (
    <button onClick={() => startTransition(action)}>
      {pending ? <LoadingSpinner /> : '포스트 생성'}
    </button>
  )
}
```

### 캐시 재검증

뮤테이션 후 캐시된 데이터 새로고침:

```ts
'use server'
import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  // 데이터 업데이트
  // ...
  revalidatePath('/posts')
}
```

또는 태그가 지정된 캐시 항목에 `revalidateTag()` 사용.

### 페이지 새로고침

`next/cache`의 `refresh()`를 사용하여 현재 페이지 UI 새로고침:

```ts
'use server'
import { refresh } from 'next/cache'

export async function updatePost(formData: FormData) {
  // 데이터 업데이트
  refresh()
}
```

### 뮤테이션 후 리디렉션

```ts
'use server'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  // 데이터 업데이트
  revalidatePath('/posts')
  redirect('/posts')
}
```

**중요**: `redirect()`는 제어 흐름 예외를 발생시킵니다. 새 데이터가 필요한 경우 리디렉션 *전에* 재검증하세요.

### 쿠키 관리

```ts
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

쿠키가 설정/삭제되면 Next.js는 현재 페이지를 서버 측에서 다시 렌더링하여 클라이언트 상태를 유지하면서 새 값으로 UI를 업데이트합니다.

---

## 고급 패턴

### 낙관적 업데이트 (Optimistic Updates)

```tsx
'use client'
import { useOptimistic, startTransition } from 'react'
import { updatePost } from './actions'

export default function PostEditor({ post }) {
  const [optimisticPost, setOptimisticPost] = useOptimistic(
    post,
    (currentPost, newTitle) => ({ ...currentPost, title: newTitle })
  )

  async function handleSubmit(formData: FormData) {
    const newTitle = formData.get('title')

    startTransition(() => {
      setOptimisticPost(newTitle)
    })

    await updatePost(formData)
  }

  return (
    <form action={handleSubmit}>
      <h2>{optimisticPost.title}</h2>
      <input type="text" name="title" defaultValue={post.title} />
      <button type="submit">업데이트</button>
    </form>
  )
}
```

### 유효성 검사

```ts
// app/actions.ts
'use server'
import { z } from 'zod'
import { revalidatePath } from 'next/cache'

const PostSchema = z.object({
  title: z.string().min(1, '제목은 필수입니다'),
  content: z.string().min(10, '내용은 최소 10자 이상이어야 합니다'),
})

export async function createPost(formData: FormData) {
  // 검증
  const validatedFields = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  })

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }

  // 데이터 삽입
  try {
    await db.post.create({
      data: validatedFields.data,
    })
  } catch (error) {
    return { message: '포스트 생성 실패' }
  }

  revalidatePath('/posts')
  redirect('/posts')
}
```

```tsx
// app/ui/create-form.tsx
'use client'
import { useActionState } from 'react'
import { createPost } from '@/app/actions'

export default function CreateForm() {
  const [state, action, isPending] = useActionState(createPost, null)

  return (
    <form action={action}>
      <div>
        <label htmlFor="title">제목</label>
        <input id="title" name="title" type="text" />
        {state?.errors?.title && (
          <p className="error">{state.errors.title}</p>
        )}
      </div>

      <div>
        <label htmlFor="content">내용</label>
        <textarea id="content" name="content" />
        {state?.errors?.content && (
          <p className="error">{state.errors.content}</p>
        )}
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? '생성 중...' : '포스트 생성'}
      </button>

      {state?.message && <p className="error">{state.message}</p>}
    </form>
  )
}
```

### 에러 처리

```ts
// app/actions.ts
'use server'

export async function deletePost(postId: string) {
  try {
    await db.post.delete({
      where: { id: postId },
    })
    revalidatePath('/posts')
    return { success: true }
  } catch (error) {
    console.error('Delete failed:', error)
    return {
      success: false,
      error: '포스트 삭제 중 오류가 발생했습니다',
    }
  }
}
```

```tsx
// app/ui/delete-button.tsx
'use client'
import { deletePost } from '@/app/actions'
import { useState } from 'react'

export default function DeleteButton({ postId }) {
  const [error, setError] = useState(null)

  async function handleDelete() {
    const result = await deletePost(postId)

    if (!result.success) {
      setError(result.error)
    }
  }

  return (
    <div>
      <button onClick={handleDelete}>삭제</button>
      {error && <p className="error">{error}</p>}
    </div>
  )
}
```

### 파일 업로드

```ts
// app/actions.ts
'use server'
import { writeFile } from 'fs/promises'
import { join } from 'path'

export async function uploadImage(formData: FormData) {
  const file = formData.get('image') as File

  if (!file) {
    return { error: '파일이 없습니다' }
  }

  const bytes = await file.arrayBuffer()
  const buffer = Buffer.from(bytes)

  const path = join(process.cwd(), 'public', 'uploads', file.name)
  await writeFile(path, buffer)

  return { success: true, path: `/uploads/${file.name}` }
}
```

```tsx
// app/ui/upload-form.tsx
'use client'
import { uploadImage } from '@/app/actions'
import { useState } from 'react'

export default function UploadForm() {
  const [preview, setPreview] = useState(null)

  async function handleSubmit(formData: FormData) {
    const result = await uploadImage(formData)

    if (result.success) {
      setPreview(result.path)
    }
  }

  return (
    <form action={handleSubmit}>
      <input type="file" name="image" accept="image/*" />
      <button type="submit">업로드</button>
      {preview && <img src={preview} alt="업로드된 이미지" />}
    </form>
  )
}
```

### 여러 Actions 조합

```ts
// app/actions.ts
'use server'

export async function createPostWithTags(formData: FormData) {
  // 포스트 생성
  const post = await db.post.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
    },
  })

  // 태그 처리
  const tags = formData.get('tags').split(',')
  await Promise.all(
    tags.map(tag =>
      db.tag.create({
        data: {
          name: tag.trim(),
          postId: post.id,
        },
      })
    )
  )

  revalidatePath('/posts')
  redirect(`/posts/${post.id}`)
}
```

### 인증 확인

```ts
// app/actions.ts
'use server'
import { auth } from '@/lib/auth'
import { redirect } from 'next/navigation'

export async function deletePost(postId: string) {
  const session = await auth()

  if (!session?.user) {
    redirect('/login')
  }

  // 권한 확인
  const post = await db.post.findUnique({
    where: { id: postId },
  })

  if (post.authorId !== session.user.id) {
    throw new Error('권한이 없습니다')
  }

  await db.post.delete({
    where: { id: postId },
  })

  revalidatePath('/posts')
}
```

---

## 실전 예제

### 좋아요 기능

```ts
// app/actions.ts
'use server'
import { revalidatePath } from 'next/cache'

export async function toggleLike(postId: string, userId: string) {
  const existingLike = await db.like.findFirst({
    where: { postId, userId },
  })

  if (existingLike) {
    await db.like.delete({ where: { id: existingLike.id } })
  } else {
    await db.like.create({
      data: { postId, userId },
    })
  }

  revalidatePath(`/posts/${postId}`)
}
```

```tsx
// app/ui/like-button.tsx
'use client'
import { toggleLike } from '@/app/actions'
import { useOptimistic } from 'react'

export default function LikeButton({ postId, userId, initialLiked, initialCount }) {
  const [optimisticState, setOptimisticState] = useOptimistic(
    { liked: initialLiked, count: initialCount },
    (state) => ({
      liked: !state.liked,
      count: state.liked ? state.count - 1 : state.count + 1,
    })
  )

  async function handleLike() {
    setOptimisticState()
    await toggleLike(postId, userId)
  }

  return (
    <button onClick={handleLike}>
      {optimisticState.liked ? '❤️' : '🤍'} {optimisticState.count}
    </button>
  )
}
```

### 검색 필터

```tsx
// app/search/page.tsx
import { Suspense } from 'react'
import SearchForm from './search-form'
import SearchResults from './search-results'

export default function SearchPage({ searchParams }) {
  return (
    <>
      <SearchForm />
      <Suspense fallback={<div>검색 중...</div>}>
        <SearchResults query={searchParams.q} />
      </Suspense>
    </>
  )
}
```

```tsx
// app/search/search-form.tsx
'use client'
import { useRouter } from 'next/navigation'

export default function SearchForm() {
  const router = useRouter()

  async function handleSearch(formData: FormData) {
    const query = formData.get('q')
    router.push(`/search?q=${query}`)
  }

  return (
    <form action={handleSearch}>
      <input name="q" type="search" placeholder="검색..." />
      <button type="submit">검색</button>
    </form>
  )
}
```

---

## 주요 포인트

- **순차 실행**: Server Actions는 현재 한 번에 하나씩 디스패치되고 await됩니다. 병렬 작업의 경우 Route Handlers를 사용하거나 단일 Server Action 내에서 작업을 수행하세요.
- **HTTP 메서드**: Actions는 `POST` 요청만 사용합니다.
- **통합**: Next.js는 Server Actions를 캐싱 아키텍처와 통합하여 단일 왕복 업데이트를 제공합니다.
- **점진적 향상**: Server Components의 폼은 JavaScript 없이 작동합니다.
- **타입 안정성**: 더 나은 개발 경험을 위해 TypeScript와 함께 작동합니다.

---

## 관련 문서

- [Forms](./forms.md)
- [revalidatePath](../api-reference/functions/revalidatePath.md)
- [revalidateTag](../api-reference/functions/revalidateTag.md)
- [redirect](../api-reference/functions/redirect.md)
- [cookies](../api-reference/functions/cookies.md)
