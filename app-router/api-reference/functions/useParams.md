---
원문: https://nextjs.org/docs/app/api-reference/functions/use-params
버전: 16.1.6
---

# useParams

`useParams`는 **Client Component** hook으로, 현재 URL에 의해 채워진 라우트의 **동적 파라미터**를 읽을 수 있습니다.

---

## 기본 사용법

```tsx
'use client'

import { useParams } from 'next/navigation'

export default function ExampleClientComponent() {
  const params = useParams<{ tag: string; item: string }>()

  // 라우트: /shop/[tag]/[item]
  // URL: /shop/shoes/nike-air-max-97
  // params -> { tag: 'shoes', item: 'nike-air-max-97' }

  return (
    <div>
      <p>Tag: {params.tag}</p>
      <p>Item: {params.item}</p>
    </div>
  )
}
```

---

## 함수 시그니처

```typescript
useParams<T = Params>(): T
```

### 매개변수

없음

### 반환값

현재 라우트의 동적 세그먼트로 채워진 객체를 반환합니다.

- 각 속성은 활성 동적 세그먼트
- 속성명 = 세그먼트 이름
- 속성값 = 채워진 값
- 값은 `string` 또는 `string[]` (catch-all 세그먼트의 경우)
- 동적 파라미터가 없으면 빈 객체 `{}` 반환
- Pages Router에서는 초기 렌더링 시 `null` 반환 가능

---

## 반환값 예제

| 라우트 | URL | 반환값 |
|--------|-----|--------|
| `app/shop/page.js` | `/shop` | `{}` |
| `app/shop/[slug]/page.js` | `/shop/1` | `{ slug: '1' }` |
| `app/shop/[tag]/[item]/page.js` | `/shop/1/2` | `{ tag: '1', item: '2' }` |
| `app/shop/[...slug]/page.js` | `/shop/1/2` | `{ slug: ['1', '2'] }` |
| `app/shop/[[...slug]]/page.js` | `/shop` | `{}` |
| `app/shop/[[...slug]]/page.js` | `/shop/1/2` | `{ slug: ['1', '2'] }` |

---

## 사용 예제

### 단일 동적 세그먼트

```tsx
// app/blog/[slug]/page.tsx
'use client'

import { useParams } from 'next/navigation'

export default function BlogPost() {
  const params = useParams<{ slug: string }>()

  return (
    <div>
      <h1>Blog Post</h1>
      <p>Slug: {params.slug}</p>
    </div>
  )
}
```

### 다중 동적 세그먼트

```tsx
// app/shop/[category]/[product]/page.tsx
'use client'

import { useParams } from 'next/navigation'

export default function ProductPage() {
  const params = useParams<{ category: string; product: string }>()

  return (
    <div>
      <h1>Product Details</h1>
      <p>Category: {params.category}</p>
      <p>Product: {params.product}</p>
    </div>
  )
}
```

### Catch-all 세그먼트

```tsx
// app/docs/[...slug]/page.tsx
'use client'

import { useParams } from 'next/navigation'

export default function DocsPage() {
  const params = useParams<{ slug: string[] }>()

  return (
    <div>
      <h1>Documentation</h1>
      <p>Path: {params.slug?.join('/')}</p>
    </div>
  )
}
```

### Optional Catch-all 세그먼트

```tsx
// app/shop/[[...categories]]/page.tsx
'use client'

import { useParams } from 'next/navigation'

export default function ShopPage() {
  const params = useParams<{ categories?: string[] }>()

  return (
    <div>
      <h1>Shop</h1>
      {params.categories ? (
        <p>Categories: {params.categories.join(' > ')}</p>
      ) : (
        <p>All Products</p>
      )}
    </div>
  )
}
```

### 데이터 페칭과 함께 사용

```tsx
'use client'

import { useParams } from 'next/navigation'
import { useEffect, useState } from 'react'

export default function UserProfile() {
  const params = useParams<{ id: string }>()
  const [user, setUser] = useState(null)

  useEffect(() => {
    async function fetchUser() {
      const res = await fetch(`/api/users/${params.id}`)
      const data = await res.json()
      setUser(data)
    }

    if (params.id) {
      fetchUser()
    }
  }, [params.id])

  return (
    <div>
      <h1>User Profile</h1>
      {user ? <p>Name: {user.name}</p> : <p>Loading...</p>}
    </div>
  )
}
```

### TypeScript 타입 안전성

```tsx
'use client'

import { useParams } from 'next/navigation'

type ProductParams = {
  category: string
  productId: string
}

export default function Product() {
  const params = useParams<ProductParams>()

  // TypeScript가 params의 타입을 알고 있음
  console.log(params.category) // ✅ 타입 안전
  console.log(params.productId) // ✅ 타입 안전
  // console.log(params.invalid) // ❌ 타입 오류

  return <div>...</div>
}
```

---

## 서버 컴포넌트에서

Server Components에서는 `params` prop을 사용하세요.

```tsx
// app/shop/[category]/[product]/page.tsx
type Props = {
  params: Promise<{ category: string; product: string }>
}

export default async function ProductPage({ params }: Props) {
  const { category, product } = await params

  return (
    <div>
      <h1>{product}</h1>
      <p>Category: {category}</p>
    </div>
  )
}
```

---

## 중요한 주의사항

> **Good to know**:
> * `useParams`는 **Client Component** hook이며 Server Components에서는 사용할 수 없습니다
> * Server Components에서는 `params` prop을 사용하세요
> * Pages Router와 함께 사용 시 초기 렌더링에서 `null`을 반환할 수 있습니다
> * Catch-all 세그먼트는 배열을 반환하고, 일반 동적 세그먼트는 문자열을 반환합니다

> **TypeScript 팁**:
> * `useParams<T>()` 제네릭을 사용하여 타입 안전성을 확보하세요
> * Optional catch-all 세그먼트는 `?`를 사용하여 선택적으로 표시하세요

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.3.0 | `useParams` 도입 |

---

## 관련 문서

- [useRouter](./useRouter.md)
- [usePathname](./usePathname.md)
- [useSearchParams](./useSearchParams.md)
- [Dynamic Routes](../../getting-started/04-linking-and-navigating.md)
