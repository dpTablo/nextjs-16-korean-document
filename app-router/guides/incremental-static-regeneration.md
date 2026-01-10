# Incremental Static Regeneration (ISR)

전체 사이트를 다시 빌드하지 않고 정적 콘텐츠를 업데이트하는 방법을 알아봅니다.

## 개요

Incremental Static Regeneration (ISR)은 다음을 가능하게 합니다:

- **점진적 업데이트** - 전체 사이트 재빌드 없이 정적 콘텐츠 업데이트
- **서버 부하 감소** - 사전 렌더링된 정적 페이지 제공
- **자동 캐시 제어** - 적절한 `cache-control` 헤더 자동 추가
- **대규모 콘텐츠 처리** - 긴 빌드 시간 없이 대량의 콘텐츠 처리

---

## 기본 예시

가장 간단한 ISR 구현입니다.

**app/blog/[id]/page.tsx**
```tsx
export const revalidate = 60 // 60초마다 캐시 무효화

export async function generateStaticParams() {
  const posts = await fetch('https://api.vercel.app/blog').then((res) =>
    res.json()
  )
  return posts.map((post) => ({
    id: String(post.id),
  }))
}

export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post = await fetch(`https://api.vercel.app/blog/${id}`).then(
    (res) => res.json()
  )
  return (
    <main>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </main>
  )
}
```

### 작동 방식

1. **빌드 시** - 모든 알려진 블로그 게시물이 `next build`에서 생성됩니다
2. **요청 시** - 요청이 캐시되고 즉시 제공됩니다
3. **60초 후** - 다음 요청은 여전히 캐시된 (오래된) 페이지를 반환합니다
4. **백그라운드 재생성** - 백그라운드에서 페이지 재생성이 트리거됩니다
5. **캐시 업데이트** - 업데이트된 페이지가 이후 요청을 위해 캐시됩니다
6. **알 수 없는 경로** - 온디맨드로 생성됩니다 (`dynamicParams`로 구성 가능)

---

## 재검증 전략

### 시간 기반 재검증

지정된 시간 간격 후 캐시를 무효화합니다.

**app/blog/page.tsx**
```tsx
export const revalidate = 3600 // 1시간마다 무효화

export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return (
    <main>
      <h1>블로그 게시물</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </main>
  )
}
```

**권장사항:** 높은 재검증 시간을 사용하세요 (예: 1초 대신 1시간).

### 페이지별 재검증 설정

**app/posts/page.tsx**
```tsx
export const revalidate = 60 // 60초

export default async function PostsPage() {
  const posts = await fetch('https://api.example.com/posts')
  const data = await posts.json()
  return (
    <div>
      {data.map((post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  )
}
```

**app/products/page.tsx**
```tsx
export const revalidate = false // 재검증 안 함 (무한 캐시)

export default async function ProductsPage() {
  // 정적 데이터
  return <div>Products</div>
}
```

---

## 온디맨드 재검증

### revalidatePath 사용

전체 라우트의 캐시를 무효화합니다.

**app/actions.ts**
```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost() {
  // 데이터베이스에 게시물 추가
  // ...

  // /posts 라우트 캐시 무효화
  revalidatePath('/posts')
}
```

**app/posts/create/page.tsx**
```tsx
'use client'

import { createPost } from '../actions'

export default function CreatePost() {
  async function handleSubmit(formData: FormData) {
    await createPost()
    // 캐시가 무효화되고 /posts 페이지가 재생성됨
  }

  return (
    <form action={handleSubmit}>
      <button type="submit">게시물 생성</button>
    </form>
  )
}
```

### 특정 경로 무효화

```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updatePost(id: string) {
  // 게시물 업데이트
  // ...

  // 특정 게시물 페이지만 무효화
  revalidatePath(`/posts/${id}`)
}
```

### 레이아웃 무효화

```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updateNavigation() {
  // 레이아웃 데이터 업데이트
  // ...

  // 레이아웃과 모든 하위 페이지 무효화
  revalidatePath('/', 'layout')
}
```

---

## 태그 기반 재검증

### revalidateTag 사용

특정 태그가 지정된 모든 데이터를 무효화합니다.

### Fetch 호출에 태그 지정

**app/blog/page.tsx**
```tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog', {
    next: { tags: ['posts'] },
  })
  const posts = await data.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### 데이터베이스 쿼리에 태그 지정

`unstable_cache`를 사용하여 데이터베이스 쿼리에 태그를 지정합니다.

**lib/data.ts**
```ts
import { unstable_cache } from 'next/cache'
import { db, posts } from '@/lib/db'

export const getCachedPosts = unstable_cache(
  async () => {
    return await db.select().from(posts)
  },
  ['posts'],
  { revalidate: 3600, tags: ['posts'] }
)
```

**app/blog/page.tsx**
```tsx
import { getCachedPosts } from '@/lib/data'

export default async function Page() {
  const posts = await getCachedPosts()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### 태그된 데이터 무효화

**app/actions.ts**
```ts
'use server'

import { revalidateTag } from 'next/cache'

export async function createPost() {
  // 데이터베이스에 게시물 추가
  // ...

  // 'posts' 태그가 지정된 모든 데이터 무효화
  revalidateTag('posts')
}
```

### 여러 태그 사용

**app/products/page.tsx**
```tsx
export default async function Page() {
  const products = await fetch('https://api.example.com/products', {
    next: { tags: ['products', 'featured'] },
  })
  const data = await products.json()
  return <div>{/* ... */}</div>
}
```

**app/actions.ts**
```ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updateFeaturedProducts() {
  // 'featured' 태그만 무효화
  revalidateTag('featured')
}

export async function updateAllProducts() {
  // 'products' 태그로 모든 제품 무효화
  revalidateTag('products')
}
```

---

## 고급 패턴

### 조건부 재검증

```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updatePost(id: string, isPublished: boolean) {
  // 게시물 업데이트
  // ...

  // 게시된 경우에만 캐시 무효화
  if (isPublished) {
    revalidatePath(`/posts/${id}`)
    revalidatePath('/posts') // 목록도 무효화
  }
}
```

### 여러 경로 무효화

```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updateCategory(categoryId: string) {
  // 카테고리 업데이트
  // ...

  // 관련된 모든 경로 무효화
  revalidatePath(`/categories/${categoryId}`)
  revalidatePath('/categories')
  revalidatePath('/') // 홈페이지에 카테고리 목록이 있는 경우
}
```

### 캐시 태그와 경로 조합

```ts
'use server'

import { revalidateTag, revalidatePath } from 'next/cache'

export async function publishPost(id: string) {
  // 게시물 게시
  // ...

  // 태그 기반 무효화
  revalidateTag('posts')
  revalidateTag(`post-${id}`)

  // 경로 기반 무효화
  revalidatePath('/posts')
  revalidatePath(`/posts/${id}`)
}
```

---

## Route Segment Config

### revalidate

페이지의 기본 재검증 시간을 설정합니다.

```tsx
export const revalidate = 3600 // 1시간

// 또는
export const revalidate = false // 무한 캐시
export const revalidate = 0 // 항상 동적
```

### dynamicParams

`generateStaticParams`에 없는 동적 세그먼트 처리 방법을 제어합니다.

```tsx
export const dynamicParams = true // 기본값: 온디맨드 생성
// 또는
export const dynamicParams = false // 404 반환
```

**예시:**
```tsx
// app/posts/[id]/page.tsx
export const dynamicParams = false

export async function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }]
}

// /posts/1, /posts/2, /posts/3 → 성공
// /posts/4 → 404 (dynamicParams = false이므로)
```

---

## 오류 처리

재검증이 실패하면 마지막으로 성공적으로 생성된 데이터가 계속 제공됩니다.

### 자동 재시도

```tsx
export default async function Page() {
  try {
    const data = await fetch('https://api.example.com/data', {
      next: { revalidate: 60 },
    })
    return <div>{/* 데이터 표시 */}</div>
  } catch (error) {
    // 페치 실패 시 이전 캐시 제공
    return <div>일시적으로 데이터를 불러올 수 없습니다</div>
  }
}
```

### 오류 추적

```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updateData() {
  try {
    // 데이터 업데이트
    revalidatePath('/data')
    return { success: true }
  } catch (error) {
    console.error('재검증 실패:', error)
    // 오류 추적 서비스에 보고
    return { success: false, error: error.message }
  }
}
```

---

## 문제 해결

### 로컬에서 캐시된 데이터 디버깅

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}

module.exports = nextConfig
```

이 설정은 다음과 같은 로그를 출력합니다:
```
GET https://api.vercel.app/blog 200 in 150ms (cache: HIT)
GET https://api.vercel.app/blog/1 200 in 100ms (cache: MISS)
```

### 프로덕션 동작 확인

```bash
# 프로덕션 빌드
next build

# 프로덕션 서버 시작
next start
```

**.env.local**
```env
NEXT_PRIVATE_DEBUG_CACHE=1
```

이렇게 하면 캐시 동작에 대한 자세한 로그가 출력됩니다.

### 재검증이 작동하지 않을 때

**1. revalidate 값 확인:**
```tsx
// ✅ 올바른 예
export const revalidate = 60

// ❌ 잘못된 예
const revalidate = 60 // export 누락
```

**2. 캐시 태그 확인:**
```tsx
// fetch에 태그가 있는지 확인
const data = await fetch('...', {
  next: { tags: ['my-tag'] }
})

// revalidateTag 호출 시 동일한 태그 사용
revalidateTag('my-tag')
```

**3. 경로 확인:**
```tsx
// ✅ 정확한 경로
revalidatePath('/posts/1')

// ❌ 재작성된 경로는 작동하지 않음
revalidatePath('/post-1') // /posts/1로 재작성되는 경우
```

---

## 주의사항

### ✅ 지원됨

- **Node.js Runtime** - 기본값, ISR 완전 지원
- **Docker Container** - 완전 지원
- **Vercel** - 완전 지원

### ❌ 지원되지 않음

- **Static Export** - ISR 미지원 (`output: 'export'`)
- **Edge Runtime** - 제한된 지원

### 중요한 고려사항

**여러 fetch 요청:**
```tsx
export default async function Page() {
  // 가장 낮은 revalidate 값이 사용됨
  await fetch('...', { next: { revalidate: 60 } })  // 60초
  await fetch('...', { next: { revalidate: 30 } })  // 30초
  // → 페이지는 30초마다 재검증됨
}
```

**동적 렌더링:**
```tsx
export default async function Page() {
  await fetch('...', { next: { revalidate: 60 } })
  await fetch('...', { cache: 'no-store' })  // 또는 revalidate: 0
  // → 전체 페이지가 동적으로 렌더링됨 (ISR 비활성화)
}
```

**프록시 로직:**
- 온디맨드 ISR 요청에는 프록시 로직이 실행되지 않습니다
- 미들웨어에서 헤더를 수정하는 경우 주의하세요

---

## 플랫폼 지원

| 옵션 | 지원 |
|------|------|
| Node.js Server | ✅ 예 |
| Docker Container | ✅ 예 |
| Static Export | ❌ 아니오 |
| Vercel | ✅ 예 |
| Netlify | ⚠️ 어댑터 필요 |
| Cloudflare | ⚠️ 제한적 |

---

## 실전 예시

### 블로그 시스템

**app/blog/page.tsx**
```tsx
export const revalidate = 3600 // 1시간마다

export default async function BlogPage() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] },
  })
  const data = await posts.json()

  return (
    <div>
      <h1>블로그</h1>
      {data.map((post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  )
}
```

**app/blog/[slug]/page.tsx**
```tsx
export const revalidate = 3600

export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts')
  const data = await posts.json()
  return data.map((post) => ({ slug: post.slug }))
}

export default async function PostPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { tags: [`post-${slug}`, 'posts'] },
  })
  const data = await post.json()

  return (
    <article>
      <h1>{data.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: data.content }} />
    </article>
  )
}
```

**app/actions.ts**
```ts
'use server'

import { revalidateTag, revalidatePath } from 'next/cache'

export async function publishPost(slug: string) {
  // CMS에 게시물 게시
  await fetch(`https://api.example.com/posts/${slug}/publish`, {
    method: 'POST',
  })

  // 캐시 무효화
  revalidateTag('posts') // 모든 게시물 목록
  revalidateTag(`post-${slug}`) // 특정 게시물
  revalidatePath('/blog') // 블로그 홈
}
```

### E-Commerce 제품 카탈로그

**app/products/page.tsx**
```tsx
export const revalidate = 600 // 10분마다

export default async function ProductsPage() {
  const products = await fetch('https://api.example.com/products', {
    next: { tags: ['products'] },
  })
  const data = await products.json()

  return (
    <div className="grid">
      {data.map((product) => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  )
}
```

**app/products/[id]/page.tsx**
```tsx
export const revalidate = 600

export async function generateStaticParams() {
  const products = await fetch('https://api.example.com/products')
  const data = await products.json()
  return data.slice(0, 100).map((product) => ({ id: String(product.id) }))
}

export const dynamicParams = true // 100개 이상의 제품도 허용

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await fetch(`https://api.example.com/products/${id}`, {
    next: { tags: [`product-${id}`, 'products'] },
  })
  const data = await product.json()

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <p>가격: ${data.price}</p>
    </div>
  )
}
```

**app/actions.ts**
```ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updateProductPrice(id: string, newPrice: number) {
  // 데이터베이스 업데이트
  await db.products.update({
    where: { id },
    data: { price: newPrice },
  })

  // 제품 페이지 캐시 무효화
  revalidateTag(`product-${id}`)
  revalidateTag('products') // 목록도 업데이트
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **적절한 재검증 시간 설정**
   ```tsx
   // ✅ 좋은 예: 콘텐츠 업데이트 빈도에 맞춤
   export const revalidate = 3600 // 블로그: 1시간
   export const revalidate = 300 // 뉴스: 5분
   export const revalidate = 86400 // 문서: 24시간
   ```

2. **태그를 계층적으로 사용**
   ```tsx
   // 세밀한 제어를 위한 여러 태그
   next: { tags: ['posts', `post-${id}`, `author-${authorId}`] }
   ```

3. **revalidatePath와 revalidateTag 조합**
   ```ts
   revalidateTag('posts') // 모든 게시물 데이터
   revalidatePath('/blog') // 블로그 페이지 UI
   ```

4. **높은 재검증 값 사용**
   ```tsx
   // ✅ 좋은 예
   export const revalidate = 3600 // 1시간

   // ❌ 나쁜 예
   export const revalidate = 1 // 너무 짧음
   ```

### ❌ 피해야 할 것

1. **과도하게 낮은 재검증 시간**
   ```tsx
   // ❌ 나쁜 예: 서버 부하 증가
   export const revalidate = 1
   ```

2. **불필요한 전역 재검증**
   ```ts
   // ❌ 나쁜 예: 모든 페이지 무효화
   revalidatePath('/', 'layout')

   // ✅ 좋은 예: 필요한 부분만
   revalidatePath('/posts')
   ```

3. **fetch 없이 revalidateTag 사용**
   ```tsx
   // ❌ 작동하지 않음: 태그가 없음
   const data = await fetch('...')
   // ...
   revalidateTag('my-tag') // 효과 없음

   // ✅ 올바른 예
   const data = await fetch('...', { next: { tags: ['my-tag'] } })
   ```

---

## API 참조

### Route Segment Config

- [`revalidate`](../api-reference/file-conventions/route-segment-config.md#revalidate)
- [`dynamicParams`](../api-reference/file-conventions/route-segment-config.md#dynamicparams)

### 함수

- [`revalidatePath()`](../api-reference/functions/revalidatePath.md)
- [`revalidateTag()`](../api-reference/functions/revalidateTag.md)
- [`unstable_cache()`](../api-reference/functions/unstable_cache.md)

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| `v14.1.0` | 커스텀 `cacheHandler` 안정화 |
| `v13.0.0` | App Router 도입 |
| `v12.2.0` | Pages Router 온디맨드 ISR 안정화 |
| `v9.5.0` | 안정적인 ISR 도입 |

---

## 다음 단계

- [Caching](./caching.md) - 캐싱 전략 심화
- [Data Fetching Patterns](./data-fetching-patterns.md) - 데이터 페칭 패턴
- [Server Actions Patterns](./server-actions-patterns.md) - Server Actions 패턴

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11

**참고 자료:**
- [Next.js 캐싱 문서](https://nextjs.org/docs/app/building-your-application/caching)
- [Vercel ISR 가이드](https://vercel.com/docs/concepts/incremental-static-regeneration)
