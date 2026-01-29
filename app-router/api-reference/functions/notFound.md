---
원문: https://nextjs.org/docs/app/api-reference/functions/not-found
버전: 16.1.6
---

# notFound

`notFound` 함수는 라우트 세그먼트 내에서 `not-found` 파일을 렌더링하고 `<meta name="robots" content="noindex" />` 태그를 주입할 수 있게 합니다.

---

## 함수 시그니처

```typescript
notFound(): never
```

---

## 기능 설명

`notFound()` 함수를 호출하면:

1. `NEXT_HTTP_ERROR_FALLBACK;404` 에러를 발생시킵니다
2. 함수가 호출된 라우트 세그먼트의 렌더링을 종료합니다
3. **not-found 파일**을 지정하면 해당 세그먼트 내에서 Not Found UI를 렌더링하여 에러를 처리할 수 있습니다

---

## 사용 예제

### 기본 사용법

```jsx
// app/user/[id]/page.js
import { notFound } from 'next/navigation'

async function fetchUser(id) {
  const res = await fetch('https://...')
  if (!res.ok) return undefined
  return res.json()
}

export default async function Profile({ params }) {
  const { id } = await params
  const user = await fetchUser(id)

  if (!user) {
    notFound()
  }

  return <div>User: {user.name}</div>
}
```

### not-found.js 파일과 함께 사용

```jsx
// app/user/[id]/not-found.js
export default function NotFound() {
  return (
    <div>
      <h2>사용자를 찾을 수 없습니다</h2>
      <p>요청하신 사용자가 존재하지 않습니다.</p>
    </div>
  )
}
```

### TypeScript 예제

```tsx
// app/products/[id]/page.tsx
import { notFound } from 'next/navigation'

interface Product {
  id: string
  name: string
  price: number
}

async function getProduct(id: string): Promise<Product | null> {
  const res = await fetch(`https://api.example.com/products/${id}`)
  if (!res.ok) return null
  return res.json()
}

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  if (!product) {
    notFound()
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>Price: ${product.price}</p>
    </div>
  )
}
```

### 조건부 렌더링

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation'

export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await getPost(slug)

  // 게시물이 존재하지 않거나 게시되지 않은 경우
  if (!post || !post.published) {
    notFound()
  }

  return <article>{post.content}</article>
}
```

---

## not-found.js 파일

`notFound()` 함수와 함께 `not-found.js` 파일을 생성하여 커스텀 Not Found UI를 제공할 수 있습니다.

```jsx
// app/not-found.js
import Link from 'next/link'

export default function NotFound() {
  return (
    <div>
      <h2>페이지를 찾을 수 없습니다</h2>
      <p>요청하신 리소스를 찾을 수 없습니다.</p>
      <Link href="/">홈으로 돌아가기</Link>
    </div>
  )
}
```

### 세그먼트별 not-found 파일

각 라우트 세그먼트에 개별 `not-found.js` 파일을 생성할 수 있습니다.

```
app/
├── not-found.js (전역 Not Found)
├── blog/
│   ├── not-found.js (블로그 Not Found)
│   └── [slug]/
│       └── page.js
└── user/
    ├── not-found.js (사용자 Not Found)
    └── [id]/
        └── page.js
```

---

## 주요 특징

### ✅ never 타입

TypeScript의 [`never`](https://www.typescriptlang.org/docs/handbook/2/functions.html#never) 타입을 사용하므로 `return notFound()`로 작성할 필요가 없습니다.

```tsx
// ✅ 올바른 사용
if (!user) {
  notFound()
}

// ⚠️ 불필요한 return
if (!user) {
  return notFound() // return 불필요
}
```

### ✅ SEO 최적화

`notFound()` 함수는 자동으로 `<meta name="robots" content="noindex" />` 태그를 주입하여 검색 엔진이 해당 페이지를 인덱싱하지 않도록 합니다.

### ✅ 에러 기반

내부적으로 에러를 throw하므로 함수 실행이 즉시 중단됩니다.

---

## 사용 시 주의사항

> **Good to know**:
> * `notFound()`는 `NEXT_HTTP_ERROR_FALLBACK;404` 에러를 발생시킵니다
> * TypeScript에서 `never` 타입을 반환하므로 `return` 문 불필요
> * 가장 가까운 `not-found.js` 파일을 렌더링합니다
> * 루트 레벨에 `not-found.js`가 없으면 기본 Next.js Not Found 페이지가 표시됩니다

---

## 관련 API

- **[not-found.js 파일 규칙](/docs/app/api-reference/file-conventions/not-found.md)**: 커스텀 Not Found UI 생성
- **[redirect](./redirect.md)**: 다른 URL로 리디렉션

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|---------|
| v13.0.0 | `notFound` 함수 도입 |

---

## 관련 문서

- [Error Handling](../../getting-started/07-error-handling.md)
- [redirect](./redirect.md)
