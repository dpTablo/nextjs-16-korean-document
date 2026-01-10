# page.js

`page` 파일은 **특정 라우트에만 고유한 UI**를 정의합니다. 파일에서 컴포넌트를 기본 내보내기(default export)하여 생성합니다.

---

## 기본 사용법

```tsx
// app/page.tsx
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

---

## 주요 특징

- ✅ `.js`, `.jsx`, `.tsx` 파일 확장자 사용 가능
- ✅ 라우트 서브트리의 **leaf(끝)** 역할
- ✅ **공개 접근 가능한 라우트 생성 필수**
- ✅ 기본적으로 Server Component (Client Component로 변경 가능)

---

## Props

### params (선택사항)

동적 라우트 매개변수를 포함한 **Promise 객체**입니다.

```tsx
// app/shop/[slug]/page.tsx
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  return <h1>Product: {slug}</h1>
}
```

**예제:**

| 라우트 | URL | params |
|-------|-----|--------|
| `app/page.js` | `/` | `undefined` |
| `app/shop/[slug]/page.js` | `/shop/1` | `Promise<{ slug: '1' }>` |
| `app/shop/[category]/[item]/page.js` | `/shop/1/2` | `Promise<{ category: '1', item: '2' }>` |
| `app/shop/[...slug]/page.js` | `/shop/1/2` | `Promise<{ slug: ['1', '2'] }>` |

### searchParams (선택사항)

URL의 **쿼리 문자열**을 포함한 Promise 객체입니다.

```tsx
// app/shop/page.tsx
export default async function Page({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const filters = await searchParams
  return <h1>Filters: {JSON.stringify(filters)}</h1>
}
```

**예제:**

| URL | searchParams |
|-----|--------------|
| `/shop` | `Promise<{}>` |
| `/shop?a=1` | `Promise<{ a: '1' }>` |
| `/shop?a=1&b=2` | `Promise<{ a: '1', b: '2' }>` |
| `/shop?a=1&a=2` | `Promise<{ a: ['1', '2'] }>` |

---

## 사용 예제

### 기본 페이지

```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <main>
      <h1>Welcome to Next.js</h1>
      <p>This is the home page</p>
    </main>
  )
}
```

### 동적 라우트

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params

  // 데이터 페칭
  const post = await getPost(slug)

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

### 쿼리 파라미터 사용

```tsx
// app/search/page.tsx
export default async function SearchPage({
  searchParams,
}: {
  searchParams: Promise<{ q?: string; page?: string }>
}) {
  const { q, page } = await searchParams

  return (
    <div>
      <h1>Search Results</h1>
      <p>Query: {q || 'None'}</p>
      <p>Page: {page || '1'}</p>
    </div>
  )
}
```

### params와 searchParams 함께 사용

```tsx
// app/shop/[category]/page.tsx
export default async function CategoryPage({
  params,
  searchParams,
}: {
  params: Promise<{ category: string }>
  searchParams: Promise<{ sort?: string; filter?: string }>
}) {
  const { category } = await params
  const { sort, filter } = await searchParams

  return (
    <div>
      <h1>Category: {category}</h1>
      <p>Sort: {sort || 'default'}</p>
      <p>Filter: {filter || 'none'}</p>
    </div>
  )
}
```

### Catch-all 세그먼트

```tsx
// app/docs/[...slug]/page.tsx
export default async function DocsPage({
  params,
}: {
  params: Promise<{ slug: string[] }>
}) {
  const { slug } = await params

  return (
    <div>
      <h1>Documentation</h1>
      <p>Path: {slug.join('/')}</p>
    </div>
  )
}
```

---

## Client Component에서 사용

```tsx
// app/dashboard/page.tsx
'use client'

import { use } from 'react'

export default function DashboardPage({
  params,
  searchParams,
}: {
  params: Promise<{ id: string }>
  searchParams: Promise<{ tab?: string }>
}) {
  const { id } = use(params)
  const { tab } = use(searchParams)

  return (
    <div>
      <h1>Dashboard: {id}</h1>
      <p>Active Tab: {tab || 'overview'}</p>
    </div>
  )
}
```

---

## TypeScript 타입 안정성

### PageProps 헬퍼 사용

```tsx
// app/blog/[slug]/page.tsx
import type { PageProps } from '.next/types/page'

export default async function Page(props: PageProps<'/blog/[slug]'>) {
  const { slug } = await props.params
  return <h1>Blog Post: {slug}</h1>
}
```

### 수동 타입 정의

```tsx
// app/products/[id]/page.tsx
type Props = {
  params: Promise<{ id: string }>
  searchParams: Promise<{ variant?: string }>
}

export default async function ProductPage({ params, searchParams }: Props) {
  const { id } = await params
  const { variant } = await searchParams

  return (
    <div>
      <h1>Product: {id}</h1>
      {variant && <p>Variant: {variant}</p>}
    </div>
  )
}
```

---

## 데이터 페칭

### Server Component (기본)

```tsx
// app/posts/[id]/page.tsx
async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`)
  return res.json()
}

export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post = await getPost(id)

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

### generateStaticParams와 함께 사용

```tsx
// app/products/[id]/page.tsx
export async function generateStaticParams() {
  const products = await fetch('https://api.example.com/products').then(res =>
    res.json()
  )

  return products.map((product) => ({
    id: product.id,
  }))
}

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  return <h1>{product.name}</h1>
}
```

---

## 파일 구조 예제

```
app/
├── page.tsx                    # /
├── about/
│   └── page.tsx               # /about
├── blog/
│   ├── page.tsx               # /blog
│   └── [slug]/
│       └── page.tsx           # /blog/:slug
└── shop/
    ├── page.tsx               # /shop
    └── [category]/
        ├── page.tsx           # /shop/:category
        └── [product]/
            └── page.tsx       # /shop/:category/:product
```

---

## 중요한 주의사항

> **Good to know**:
> * `page.js`는 **공개 접근 가능한 라우트를 만드는 데 필수**입니다
> * `page.js`가 없으면 해당 라우트에 접근할 수 없습니다
> * v15부터 `params`, `searchParams`가 **Promise**로 변경되었습니다
> * `searchParams`는 **Dynamic API**로, 동적 렌더링을 활성화합니다
> * Client Component에서는 `use()` 훅을 사용하여 Promise를 unwrap합니다

> **Dynamic API**:
> * `searchParams`를 사용하면 페이지가 자동으로 동적 렌더링됩니다
> * 정적 렌더링을 유지하려면 `searchParams`를 사용하지 마세요
> * 대신 Client Component에서 `useSearchParams` 훅을 사용하세요

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.0.0-RC | `params`, `searchParams`가 Promise로 변경 (codemod 지원) |
| v13.0.0 | `page.js` 규칙 도입 |

---

## 관련 문서

- [layout.js](./layout.md)
- [loading.js](./loading.md)
- [error.js](./error.md)
- [Layouts and Pages](../../getting-started/03-layouts-and-pages.md)
- [Dynamic Routes](../../getting-started/04-linking-and-navigating.md)
- [generateStaticParams](../functions/generateStaticParams.md)
