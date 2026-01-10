# revalidateTag

`revalidateTag`는 특정 캐시 태그에 대해 **온디맨드로 캐시된 데이터를 무효화**하는 함수입니다.

블로그 포스트, 제품 카탈로그, 문서처럼 약간의 업데이트 지연이 허용되는 콘텐츠에 이상적입니다. 사용자는 백그라운드에서 새로운 데이터가 로드되는 동안 오래된 콘텐츠를 받습니다.

---

## 함수 시그니처

```typescript
revalidateTag(tag: string, profile?: string | { expire?: number }): void
```

### 매개변수

#### tag (필수)

캐시 태그 문자열입니다.

- 최대 256자
- 대소문자 구분

#### profile (선택)

무효화 동작을 지정합니다.

- `'max'`: stale-while-revalidate 의미론 사용 (권장)
- `{ expire: 0 }`: 즉시 만료 (커스텀 캐시 프로필)
- 생략 시: deprecated 동작 (즉시 만료)

---

## 사용 가능 위치

- ✅ **Server Actions**
- ✅ **Route Handlers**
- ❌ Client Components
- ❌ Proxy

---

## 무효화 동작

두 번째 인수 제공 여부에 따라 동작이 달라집니다.

### 1. `profile="max"` (권장)

```ts
revalidateTag('posts', 'max')
```

**동작:**

- 태그 항목을 **stale(오래됨)로 표시**
- 다음 방문 시 **stale-while-revalidate** 의미론 사용
- 오래된 콘텐츠 제공 → 백그라운드에서 새 콘텐츠 가져오기

**장점:**

- 즉각적인 응답 (stale 콘텐츠)
- 백그라운드에서 캐시 업데이트
- 사용자 경험 향상

### 2. 커스텀 캐시 프로필

```ts
revalidateTag('posts', { expire: 0 })
```

**동작:**

- 애플리케이션에서 정의한 모든 캐시 라이프 프로필 사용 가능
- 커스텀 무효화 동작 설정 가능

### 3. 두 번째 인수 없음 (deprecated)

```ts
revalidateTag('posts')  // ⚠️ 권장하지 않음
```

**동작:**

- 태그 항목 즉시 만료
- 다음 요청에서 blocking revalidate/cache miss 발생
- 이제 deprecated 상태 → `profile="max"` 사용 권장

---

## 태그 지정 방법

### 1. fetch에서 `next.tags` 옵션 사용

```tsx
// app/blog/page.tsx
export default async function BlogPage() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  }).then(res => res.json())

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### 2. cacheTag 함수 사용

```tsx
// app/lib/data.ts
import { cacheTag } from 'next/cache'

export async function getPosts() {
  'use cache'
  cacheTag('posts')

  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json())

  return posts
}
```

---

## 사용 예제

### Server Action 예제

```ts
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  // 데이터베이스에 포스트 추가
  await addPostToDatabase({ title, content })

  // 'posts' 태그가 지정된 모든 캐시 무효화
  revalidateTag('posts', 'max')
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

### Route Handler 예제

```ts
// app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const tag = request.nextUrl.searchParams.get('tag')

  if (!tag) {
    return NextResponse.json(
      {
        revalidated: false,
        message: 'Missing tag to revalidate',
      },
      { status: 400 }
    )
  }

  revalidateTag(tag, 'max')

  return NextResponse.json({
    revalidated: true,
    now: Date.now(),
  })
}
```

**사용 예시:**

```bash
curl http://localhost:3000/api/revalidate?tag=posts
```

### 즉시 만료 예제 (웹훅/제3자 서비스용)

웹훅이나 제3자 서비스에서 호출 시 즉시 만료가 필요할 수 있습니다.

```ts
// app/api/webhook/route.ts
import { revalidateTag } from 'next/cache'
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const { event } = await request.json()

  if (event === 'post.created' || event === 'post.updated') {
    // 즉시 만료
    revalidateTag('posts', { expire: 0 })

    return NextResponse.json({ revalidated: true })
  }

  return NextResponse.json({ revalidated: false })
}
```

### 여러 태그 재검증

```ts
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updateProduct(id: string) {
  await updateProductInDatabase(id)

  // 여러 태그 재검증
  revalidateTag('products', 'max')
  revalidateTag('featured-products', 'max')
  revalidateTag(`product-${id}`, 'max')
}
```

### revalidatePath와 함께 사용

```ts
// app/actions.ts
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'

export async function updatePost(slug: string) {
  await updatePostInDatabase(slug)

  // 특정 경로 재검증
  revalidatePath(`/blog/${slug}`)

  // 태그된 모든 데이터 재검증
  revalidateTag('posts', 'max')
}
```

---

## revalidatePath와의 관계

| 함수 | 기능 |
|------|------|
| `revalidateTag` | **특정 태그를 가진 모든 페이지**에서 데이터 무효화 |
| `revalidatePath` | **특정 경로(페이지/레이아웃)**의 데이터 무효화 |

두 함수는 목적이 다르며, **포괄적인 데이터 일관성**을 위해 함께 사용해야 할 수 있습니다.

### 예제: 블로그 포스트 업데이트

```ts
// app/actions.ts
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'

export async function updateBlogPost(slug: string) {
  await updatePostInDatabase(slug)

  // 1. 특정 포스트 페이지 재검증
  revalidatePath(`/blog/${slug}`)

  // 2. 블로그 목록 페이지 재검증
  revalidatePath('/blog')

  // 3. 'posts' 태그를 사용하는 모든 곳 재검증
  revalidateTag('posts', 'max')
}
```

---

## 중요 주의사항

> **Good to know**:
> * ⚠️ **주의**: `profile="max"` 사용 시, `revalidateTag` 호출이 즉시 많은 재검증을 트리거하지 않습니다. 해당 태그를 사용하는 페이지가 다시 방문될 때만 무효화가 발생합니다.
> * Server Actions와 Route Handlers에서만 사용 가능
> * 태그는 최대 256자까지 지원
> * 대소문자를 구분합니다
> * `profile="max"` 사용을 권장합니다 (stale-while-revalidate 의미론)

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v13.0.0 | `revalidateTag` 도입 |

---

## 관련 문서

- [revalidatePath](./revalidatePath.md)
- [Caching and Revalidating](../../getting-started/06-caching-and-revalidating.md)
- [Caching Guide](../../guides/caching.md)
