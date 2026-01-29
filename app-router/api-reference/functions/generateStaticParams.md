---
원문: https://nextjs.org/docs/app/api-reference/functions/generate-static-params
버전: 16.1.6
---

# generateStaticParams

`generateStaticParams`는 동적 라우트 세그먼트와 함께 사용하여 요청 시간이 아닌 빌드 시간에 라우트를 정적으로 생성하는 함수입니다.

---

## 사용 가능한 위치

- Pages (`page.tsx`/`page.js`)
- Layouts (`layout.tsx`/`layout.js`)
- Route Handlers (`route.ts`/`route.js`)

---

## 기본 예제

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
  return posts.map((post) => ({
    slug: post.slug,
  }))
}

export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  // ...
}
```

---

## 반환 타입

| 라우트 예제 | 반환 타입 |
|-----------|---------|
| `/product/[id]` | `{ id: string }[]` |
| `/products/[category]/[product]` | `{ category: string, product: string }[]` |
| `/products/[...slug]` | `{ slug: string[] }[]` |

---

## 단일 동적 세그먼트 예제

```tsx
// app/product/[id]/page.tsx
export function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }]
}

// 생성되는 경로: /product/1, /product/2, /product/3
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  return <div>Product: {id}</div>
}
```

---

## 다중 동적 세그먼트 예제

```tsx
// app/products/[category]/[product]/page.tsx
export function generateStaticParams() {
  return [
    { category: 'a', product: '1' },
    { category: 'b', product: '2' },
    { category: 'c', product: '3' },
  ]
}

// 생성되는 경로: /products/a/1, /products/b/2, /products/c/3
export default async function Page({
  params,
}: {
  params: Promise<{ category: string; product: string }>
}) {
  const { category, product } = await params
  return <div>Category: {category}, Product: {product}</div>
}
```

---

## Catch-all 동적 세그먼트 예제

```tsx
// app/product/[...slug]/page.tsx
export function generateStaticParams() {
  return [
    { slug: ['a', '1'] },
    { slug: ['b', '2'] },
    { slug: ['c', '3'] }
  ]
}

// 생성되는 경로: /product/a/1, /product/b/2, /product/c/3
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string[] }>
}) {
  const { slug } = await params
  return <div>Slug: {slug.join('/')}</div>
}
```

---

## 정적 렌더링 패턴

### 1. 빌드 시 모든 경로 렌더링

```tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

### 2. 빌드 시 부분 경로, 런타임 시 나머지 렌더링

```tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
  // 처음 10개만 빌드 시 렌더링
  return posts.slice(0, 10).map((post) => ({
    slug: post.slug,
  }))
}

// 빌드 시 렌더링되지 않은 경로는 404
export const dynamicParams = false
```

### 3. 런타임 시 모든 경로 렌더링

```tsx
export async function generateStaticParams() {
  return [] // 빈 배열 반환
}

// 또는
export const dynamic = 'force-static'
```

---

## Route Handlers와 함께 사용

```ts
// app/api/posts/[id]/route.ts
export async function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }]
}

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  // ID 1, 2, 3에 대해 정적으로 생성됨
  return Response.json({ id, title: `Post ${id}` })
}
```

---

## 다중 동적 세그먼트 - 상향식 접근

모든 동적 세그먼트를 한 번에 생성하는 방식입니다.

```tsx
// app/products/[category]/[product]/page.tsx
export async function generateStaticParams() {
  const products = await fetch('https://.../products').then((res) => res.json())

  return products.map((product) => ({
    category: product.category.slug,
    product: product.id,
  }))
}
```

---

## 다중 동적 세그먼트 - 하향식 접근

부모 세그먼트에서 자식 세그먼트로 순차적으로 생성하는 방식입니다.

```tsx
// app/products/[category]/layout.tsx
export async function generateStaticParams() {
  const products = await fetch('https://.../products').then((res) => res.json())
  return products.map((product) => ({
    category: product.category.slug,
  }))
}

// app/products/[category]/[product]/page.tsx
export async function generateStaticParams({
  params,
}: {
  params: Promise<{ category: string }>
}) {
  const { category } = await params
  const products = await fetch(
    `https://.../products?category=${category}`
  ).then((res) => res.json())

  return products.map((product) => ({
    product: product.id,
  }))
}
```

---

## dynamicParams 옵션

`generateStaticParams`에서 생성되지 않은 동적 세그먼트 방문 시 동작을 제어합니다.

```tsx
// app/blog/[slug]/page.tsx
export const dynamicParams = true // 기본값: 생성되지 않은 경로는 요청 시 생성
// export const dynamicParams = false // 생성되지 않은 경로는 404

export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
  return posts.slice(0, 10).map((post) => ({
    slug: post.slug,
  }))
}

export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  // ...
}
```

---

## 주요 특징

### ✅ 정적 생성

빌드 시 라우트를 미리 생성하여 성능을 향상시킵니다.

### ✅ 동적 세그먼트 지원

- 단일 동적 세그먼트: `[id]`
- 다중 동적 세그먼트: `[category]/[product]`
- Catch-all 세그먼트: `[...slug]`

### ✅ 자동 메모이제이션

`fetch` 요청은 자동으로 메모이제이션됩니다.

### ✅ dynamicParams 옵션

`generateStaticParams`에서 생성되지 않은 동적 세그먼트 방문 시 동작을 제어할 수 있습니다.

---

## 중요 참고사항

> **Good to know**:
> * `next dev` 실행 중: 라우트 네비게이션 시 `generateStaticParams` 호출
> * `next build` 실행 중: 해당 Layouts/Pages 생성 전에 `generateStaticParams` 실행
> * 리검증(ISR) 중: `generateStaticParams` 재호출 안 됨
> * `generateStaticParams`는 Pages Router의 `getStaticPaths`를 대체함
> * Cache Components 사용 시 최소 1개 이상의 param 반환 필요

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v13.0.0 | `generateStaticParams` 도입 |

---

## 관련 문서

- [Dynamic Routes](../../getting-started/04-linking-and-navigating.md)
- [generateMetadata](./generateMetadata.md)
- [Caching and Revalidating](../../getting-started/06-caching-and-revalidating.md)
