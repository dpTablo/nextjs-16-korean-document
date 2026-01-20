# getStaticProps

`getStaticProps`는 Pages Router에서 페이지를 **빌드 타임에 미리 렌더링**하는 함수입니다. 정적 사이트 생성(SSG)과 증분 정적 재생성(ISR)을 구현하는 데 사용됩니다.

---

## 기본 사용법

```tsx
import type { InferGetStaticPropsType, GetStaticProps } from 'next'

type Repo = {
  name: string
  stargazers_count: number
}

export const getStaticProps = (async (context) => {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const repo = await res.json()
  return { props: { repo } }
}) satisfies GetStaticProps<{ repo: Repo }>

export default function Page({
  repo,
}: InferGetStaticPropsType<typeof getStaticProps>) {
  return <p>Stars: {repo.stargazers_count}</p>
}
```

---

## Context 파라미터

`getStaticProps`는 `context` 객체를 파라미터로 받습니다.

| 파라미터 | 타입 | 설명 |
|----------|------|------|
| `params` | `object` | 동적 라우트의 파라미터 (예: `[id].js`는 `{ id: ... }`) |
| `preview` | `boolean` | Preview Mode 상태 (deprecated) |
| `previewData` | `any` | Preview 데이터 (deprecated) |
| `draftMode` | `boolean` | Draft Mode 활성화 여부 |
| `locale` | `string` | 현재 활성 로케일 |
| `locales` | `string[]` | 지원하는 모든 로케일 |
| `defaultLocale` | `string` | 기본 로케일 |
| `revalidateReason` | `string` | 함수 호출 이유: `"build"`, `"stale"`, `"on-demand"` |

---

## 반환값

`getStaticProps`는 다음 속성을 포함하는 객체를 반환해야 합니다.

### props

페이지 컴포넌트에 전달될 직렬화 가능한 객체입니다.

```js
export async function getStaticProps() {
  return {
    props: {
      message: 'Next.js is awesome',
    },
  }
}
```

### revalidate

페이지 재생성 간격(초 단위)입니다. **Incremental Static Regeneration(ISR)**을 활성화합니다.

```js
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  return {
    props: { posts },
    revalidate: 10, // 10초마다 재검증
  }
}
```

**캐시 상태** (`x-nextjs-cache` 헤더):

| 상태 | 설명 |
|------|------|
| `MISS` | 캐시 없음 (첫 방문 시) |
| `STALE` | 캐시 만료, 백그라운드 업데이트 중 |
| `HIT` | 캐시 유효, 캐시된 응답 반환 |

### notFound

`true`로 설정하면 404 상태를 반환합니다.

```js
export async function getStaticProps(context) {
  const res = await fetch(`https://api.example.com/data/${context.params.id}`)
  const data = await res.json()

  if (!data) {
    return {
      notFound: true,
    }
  }

  return {
    props: { data },
  }
}
```

### redirect

내부 또는 외부 리소스로 리다이렉트합니다.

```js
export async function getStaticProps(context) {
  const res = await fetch(`https://api.example.com/user`)
  const user = await res.json()

  if (!user) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
        // statusCode: 301, // permanent 대신 사용 가능
      },
    }
  }

  return {
    props: { user },
  }
}
```

---

## 실제 예제

### 외부 API에서 데이터 가져오기

```tsx
// pages/posts.tsx
import type { GetStaticProps, InferGetStaticPropsType } from 'next'

type Post = {
  id: number
  title: string
  body: string
}

export const getStaticProps: GetStaticProps<{ posts: Post[] }> = async () => {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts')
  const posts: Post[] = await res.json()

  return {
    props: { posts },
    revalidate: 60, // 1분마다 재검증
  }
}

export default function PostsPage({
  posts,
}: InferGetStaticPropsType<typeof getStaticProps>) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### 파일 시스템에서 데이터 읽기

```js
// pages/blog.js
import { promises as fs } from 'fs'
import path from 'path'

export async function getStaticProps() {
  const postsDirectory = path.join(process.cwd(), 'posts')
  const filenames = await fs.readdir(postsDirectory)

  const posts = await Promise.all(
    filenames.map(async (filename) => {
      const filePath = path.join(postsDirectory, filename)
      const fileContents = await fs.readFile(filePath, 'utf8')

      return {
        filename,
        content: fileContents,
      }
    })
  )

  return {
    props: {
      posts,
    },
  }
}

export default function BlogPage({ posts }) {
  return (
    <div>
      {posts.map((post) => (
        <article key={post.filename}>
          <pre>{post.content}</pre>
        </article>
      ))}
    </div>
  )
}
```

> **주의**: `__dirname` 대신 `process.cwd()`를 사용하세요. 컴파일 후 경로가 변경될 수 있습니다.

### 데이터베이스 직접 쿼리

```tsx
// pages/products.tsx
import type { GetStaticProps } from 'next'
import { db } from '@/lib/db'

export const getStaticProps: GetStaticProps = async () => {
  const products = await db.product.findMany({
    where: { published: true },
    orderBy: { createdAt: 'desc' },
  })

  return {
    props: {
      products: JSON.parse(JSON.stringify(products)), // 직렬화
    },
    revalidate: 300, // 5분마다 재검증
  }
}
```

### 동적 라우트와 함께 사용

동적 라우트에서는 `getStaticPaths`와 함께 사용해야 합니다.

```tsx
// pages/posts/[id].tsx
import type { GetStaticProps, GetStaticPaths } from 'next'

export const getStaticPaths: GetStaticPaths = async () => {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  const paths = posts.map((post: { id: number }) => ({
    params: { id: String(post.id) },
  }))

  return { paths, fallback: false }
}

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const res = await fetch(`https://api.example.com/posts/${params?.id}`)
  const post = await res.json()

  return {
    props: { post },
    revalidate: 60,
  }
}

export default function PostPage({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </article>
  )
}
```

### 국제화(i18n) 지원

```js
// pages/index.js
export async function getStaticProps({ locale }) {
  const messages = await import(`../locales/${locale}.json`)

  return {
    props: {
      messages: messages.default,
    },
  }
}
```

---

## 주요 특징

| 특징 | 설명 |
|------|------|
| 서버 사이드 전용 | 클라이언트에서 실행되지 않음 |
| 데이터베이스 접근 | 직접 데이터베이스 쿼리 가능 |
| 번들 제외 | 클라이언트 번들에 포함되지 않음 |
| 빌드 타임 실행 | 기본적으로 빌드 시 한 번 실행 |
| ISR 지원 | `revalidate`로 동적 재생성 가능 |

---

## 중요한 주의사항

> **Good to know**:
> - `getStaticProps`는 서버에서만 실행됩니다
> - 페이지 파일에서만 export할 수 있습니다 (컴포넌트 파일 불가)
> - 개발 모드에서는 매 요청마다 실행됩니다
> - Draft Mode에서는 빌드 타임이 아닌 요청 시점에 실행됩니다

> **App Router 마이그레이션**:
> - App Router에서는 `fetch`와 React Server Components로 대체됩니다
> - `revalidate`는 `fetch`의 `next.revalidate` 옵션으로 대체됩니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.4.0 | App Router 안정화 |
| v12.2.0 | On-Demand ISR 안정화 |
| v10.0.0 | `locale`, `locales`, `defaultLocale`, `notFound` 옵션 추가 |
| v9.5.0 | ISR 안정화 |
| v9.3.0 | `getStaticProps` 도입 |

---

## 관련 문서

- [getStaticPaths](./getStaticPaths.md)
- [getServerSideProps](./getServerSideProps.md)
- [Incremental Static Regeneration](../../app-router/guides/incremental-static-regeneration.md)
