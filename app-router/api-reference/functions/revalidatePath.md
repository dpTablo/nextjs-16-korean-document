# revalidatePath

`revalidatePath`는 특정 경로에 대해 [캐시된 데이터](../../guides/caching.md)를 온디맨드 방식으로 무효화합니다.

---

## 함수 시그니처

```typescript
revalidatePath(path: string, type?: 'page' | 'layout'): void
```

### 매개변수

#### path (필수)

재검증할 데이터의 경로 패턴 또는 구체적인 URL입니다.

- 예: `/product/[slug]` (경로 패턴) 또는 `/product/123` (구체적인 URL)
- 최대 1024자
- 대소문자 구분

#### type (선택)

재검증할 경로의 타입입니다.

- `'page'`: 특정 페이지만 무효화
- `'layout'`: 레이아웃과 그 아래 모든 중첩 레이아웃 및 페이지 무효화

**주의**: 동적 세그먼트 포함 시 필수, 구체적인 URL인 경우 생략 가능

---

## 사용 방법

### 호출 가능한 위치

- ✅ **Server Actions**
- ✅ **Route Handlers**
- ❌ Client Components
- ❌ Proxy

---

## 동작 방식

### Server Actions에서 호출 시

- UI를 즉시 업데이트합니다 (현재 경로를 보면서)
- 모든 이전 방문 페이지도 다시 방문 시 새로고침됩니다

### Route Handlers에서 호출 시

- 경로를 재검증으로 표시합니다
- 다음 방문 시 재검증을 수행합니다

---

## 무효화 가능 대상

### Pages

특정 페이지를 무효화합니다.

```ts
import { revalidatePath } from 'next/cache'

revalidatePath('/blog/post-1')
```

### Layouts

레이아웃과 그 아래 모든 중첩 레이아웃 및 페이지를 무효화합니다.

```ts
import { revalidatePath } from 'next/cache'

revalidatePath('/blog/[slug]', 'layout')
```

### Route Handlers

경로 핸들러 내 데이터 캐시 항목을 무효화합니다.

```ts
import { revalidatePath } from 'next/cache'

revalidatePath('/api/posts')
```

---

## 사용 예제

### 특정 URL 재검증

```ts
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updatePost(id: string) {
  await updatePostInDatabase(id)
  revalidatePath('/blog/post-1')
}
```

### 동적 페이지 경로 재검증

```ts
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updateBlogPost(slug: string) {
  await updatePostInDatabase(slug)
  revalidatePath('/blog/[slug]', 'page')
}
```

### Route Groups와 함께 사용

```ts
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updatePost() {
  await updatePostInDatabase()
  revalidatePath('/(main)/blog/[slug]', 'page')
}
```

### Layout 경로 재검증

```ts
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updateBlogLayout() {
  await updateLayoutData()
  revalidatePath('/blog/[slug]', 'layout')
}
```

### 모든 데이터 재검증

루트 레이아웃을 재검증하면 모든 경로가 무효화됩니다.

```ts
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function clearAllCache() {
  revalidatePath('/', 'layout')
}
```

### Server Action 예시

```tsx
// app/blog/[slug]/page.tsx
import { revalidatePath } from 'next/cache'

export default function BlogPost({ params }: { params: { slug: string } }) {
  async function handleUpdate() {
    'use server'

    await updatePost(params.slug)
    revalidatePath(`/blog/${params.slug}`)
  }

  return (
    <form action={handleUpdate}>
      <button type="submit">Update Post</button>
    </form>
  )
}
```

### Route Handler 예시

```ts
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const path = request.nextUrl.searchParams.get('path')

  if (!path) {
    return NextResponse.json(
      {
        revalidated: false,
        now: Date.now(),
        message: 'Missing path to revalidate',
      },
      { status: 400 }
    )
  }

  revalidatePath(path)

  return NextResponse.json({
    revalidated: true,
    now: Date.now(),
  })
}
```

**사용 예시:**

```bash
curl http://localhost:3000/api/revalidate?path=/blog/post-1
```

### 폼 제출 후 재검증

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  await savePostToDatabase({ title, content })

  revalidatePath('/blog')
}
```

```tsx
// app/blog/new/page.tsx
import { createPost } from '../actions'

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" type="text" required />
      <textarea name="content" required />
      <button type="submit">Create Post</button>
    </form>
  )
}
```

---

## revalidateTag 및 updateTag와의 관계

### revalidatePath

특정 페이지/레이아웃 경로를 무효화합니다.

```ts
revalidatePath('/blog/[slug]', 'page')
```

### revalidateTag

특정 태그로 표시된 데이터를 **stale**로 표시합니다.

```ts
revalidateTag('posts')
```

### updateTag

특정 태그로 표시된 데이터를 만료시킵니다.

```ts
updateTag('posts')
```

### 함께 사용하는 패턴

```ts
// app/actions.ts
'use server'

import { revalidatePath, updateTag } from 'next/cache'

export async function updatePost(slug: string) {
  // 데이터베이스 업데이트
  await updatePostInDatabase(slug)

  // 특정 경로 재검증
  revalidatePath(`/blog/${slug}`)

  // 태그된 모든 데이터 만료
  updateTag('posts')
}
```

---

## 중요 주의사항

> **Good to know**:
> * Server Actions와 Route Handlers에서만 사용 가능
> * Client Components에서는 직접 호출 불가능 (Server Actions를 통해 간접 호출)
> * 동적 세그먼트 사용 시 `type` 매개변수 필수
> * 경로는 최대 1024자까지 지원
> * 대소문자를 구분합니다

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v13.0.0 | `revalidatePath` 도입 |

---

## 관련 문서

- [revalidateTag](./revalidateTag.md)
- [Caching and Revalidating](../../getting-started/06-caching-and-revalidating.md)
- [Caching Guide](../../guides/caching.md)
