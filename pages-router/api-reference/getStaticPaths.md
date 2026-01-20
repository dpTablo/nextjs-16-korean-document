# getStaticPaths

`getStaticPaths`는 동적 라우트를 사용하는 페이지에서 사전 렌더링할 경로를 지정하는 함수입니다. `getStaticProps`와 함께 사용하여 동적 라우트의 정적 페이지를 생성합니다.

---

## 기본 사용법

```tsx
import type {
  InferGetStaticPropsType,
  GetStaticProps,
  GetStaticPaths,
} from 'next'

type Post = {
  id: string
  title: string
}

export const getStaticPaths: GetStaticPaths = async () => {
  const res = await fetch('https://api.example.com/posts')
  const posts: Post[] = await res.json()

  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))

  return {
    paths,
    fallback: false,
  }
}

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const res = await fetch(`https://api.example.com/posts/${params?.id}`)
  const post = await res.json()

  return {
    props: { post },
  }
}

export default function PostPage({
  post,
}: InferGetStaticPropsType<typeof getStaticProps>) {
  return (
    <article>
      <h1>{post.title}</h1>
    </article>
  )
}
```

---

## 반환값

`getStaticPaths`는 `paths`와 `fallback` 속성을 포함하는 객체를 반환해야 합니다.

### paths (필수)

사전 렌더링할 경로 배열입니다.

```js
return {
  paths: [
    { params: { id: '1' } },
    { params: { id: '2' } },
    {
      params: { id: '3' },
      locale: 'ko', // 특정 로케일 지정
    },
  ],
  fallback: false,
}
```

**규칙:**

| 라우트 유형 | 예시 파일 | params 형태 |
|------------|----------|-------------|
| 단일 동적 | `pages/posts/[id].js` | `{ id: '1' }` |
| 다중 동적 | `pages/posts/[postId]/[commentId].js` | `{ postId: '1', commentId: '2' }` |
| Catch-all | `pages/[...slug].js` | `{ slug: ['a', 'b', 'c'] }` |
| 선택적 Catch-all | `pages/[[...slug]].js` | `{ slug: [] }` 또는 `null` |

> **주의**: `params` 값은 문자열이어야 하며 대소문자를 구분합니다.

### fallback

`fallback` 옵션은 `getStaticPaths`에서 반환하지 않은 경로의 처리 방식을 결정합니다.

---

## fallback 옵션

### fallback: false

반환되지 않은 경로는 **404 페이지**를 표시합니다.

**사용 시기:**
- 경로 수가 적을 때
- 새 페이지가 자주 추가되지 않을 때
- 모든 경로를 빌드 시점에 생성할 수 있을 때

```js
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  const paths = posts.map((post) => ({
    params: { id: String(post.id) },
  }))

  return {
    paths,
    fallback: false,
  }
}
```

### fallback: true

반환되지 않은 경로는 **fallback 페이지**를 먼저 표시한 후, 백그라운드에서 페이지를 생성합니다.

**동작 방식:**

1. 첫 요청 시 fallback 버전의 페이지 표시
2. 백그라운드에서 `getStaticProps` 실행 및 HTML 생성
3. 생성 완료 후 브라우저가 자동으로 전체 페이지로 전환
4. 이후 요청에서는 생성된 정적 페이지 제공

**사용 시기:**
- 수천 개 이상의 페이지가 있을 때
- 일부만 사전 생성하고 나머지는 온디맨드로 생성할 때

```jsx
import { useRouter } from 'next/router'

function Post({ post }) {
  const router = useRouter()

  // fallback 상태 확인
  if (router.isFallback) {
    return <div>로딩 중...</div>
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}

export async function getStaticPaths() {
  // 인기 있는 게시물만 사전 생성
  const res = await fetch('https://api.example.com/posts?popular=true')
  const popularPosts = await res.json()

  const paths = popularPosts.map((post) => ({
    params: { id: String(post.id) },
  }))

  return {
    paths,
    fallback: true,
  }
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`)
  const post = await res.json()

  // 게시물이 없으면 404 반환
  if (!post) {
    return { notFound: true }
  }

  return {
    props: { post },
    revalidate: 60,
  }
}

export default Post
```

### fallback: 'blocking'

반환되지 않은 경로는 **서버 사이드 렌더링**을 수행한 후 전체 페이지를 반환합니다.

**동작 방식:**

1. 첫 요청 시 서버에서 HTML 생성 (로딩 상태 없음)
2. 생성 완료 후 전체 페이지 반환
3. 이후 요청을 위해 캐시됨

**사용 시기:**
- 로딩 상태 없이 즉시 전체 페이지를 표시하고 싶을 때
- SEO가 중요할 때 (크롤러가 완전한 페이지를 볼 수 있음)

```js
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: 'blocking',
  }
}
```

---

## fallback 옵션 비교

| 옵션 | 미등록 경로 처리 | 로딩 상태 | SEO |
|------|-----------------|----------|-----|
| `false` | 404 반환 | 없음 | - |
| `true` | Fallback 페이지 표시 후 생성 | 필요 | 주의 필요 |
| `'blocking'` | SSR 후 전체 페이지 반환 | 없음 | 최적 |

---

## 실제 예제

### 다중 동적 세그먼트

```tsx
// pages/posts/[postId]/comments/[commentId].tsx
export const getStaticPaths: GetStaticPaths = async () => {
  const res = await fetch('https://api.example.com/posts-with-comments')
  const data = await res.json()

  const paths = data.flatMap((post) =>
    post.comments.map((comment) => ({
      params: {
        postId: String(post.id),
        commentId: String(comment.id),
      },
    }))
  )

  return { paths, fallback: false }
}
```

### Catch-all 라우트

```tsx
// pages/docs/[...slug].tsx
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [
      { params: { slug: ['getting-started'] } },
      { params: { slug: ['guides', 'routing'] } },
      { params: { slug: ['api', 'reference', 'functions'] } },
    ],
    fallback: false,
  }
}
```

### 국제화(i18n) 지원

```tsx
// pages/posts/[id].tsx
export const getStaticPaths: GetStaticPaths = async ({ locales }) => {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  const paths = posts.flatMap((post) =>
    (locales || ['en']).map((locale) => ({
      params: { id: String(post.id) },
      locale,
    }))
  )

  return { paths, fallback: false }
}
```

### 대규모 사이트 최적화

```tsx
// pages/products/[id].tsx
export const getStaticPaths: GetStaticPaths = async () => {
  // 인기 상품 1000개만 사전 생성
  const res = await fetch('https://api.example.com/products?limit=1000&sort=popular')
  const products = await res.json()

  const paths = products.map((product) => ({
    params: { id: String(product.id) },
  }))

  return {
    paths,
    fallback: 'blocking', // 나머지는 온디맨드 생성
  }
}
```

---

## 제한사항

- `fallback: true` 및 `fallback: 'blocking'`은 `output: 'export'`와 함께 사용할 수 없습니다
- `getStaticPaths`는 `getStaticProps`와 함께 사용해야 합니다
- `getServerSideProps`와 함께 사용할 수 없습니다
- 개발 모드에서는 매 요청마다 실행됩니다

---

## 중요한 주의사항

> **Good to know**:
> - `getStaticPaths`는 빌드 시점에 실행됩니다
> - `revalidate`와 함께 사용하면 백그라운드에서 실행될 수 있습니다
> - `params` 값은 반드시 문자열이어야 합니다

> **App Router 마이그레이션**:
> - App Router에서는 `generateStaticParams`로 대체됩니다
> - `fallback` 대신 `dynamicParams` 설정을 사용합니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.4.0 | App Router 안정화, `generateStaticParams()` 추가 |
| v12.2.0 | On-Demand ISR 안정화 |
| v12.1.0 | On-Demand ISR 베타 추가 |
| v9.5.0 | ISR 안정화 |
| v9.3.0 | `getStaticPaths` 도입 |

---

## 관련 문서

- [getStaticProps](./getStaticProps.md)
- [getServerSideProps](./getServerSideProps.md)
- [Dynamic Routes](../../app-router/getting-started/04-linking-and-navigating.md)
