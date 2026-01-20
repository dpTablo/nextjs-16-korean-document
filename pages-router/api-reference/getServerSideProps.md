# getServerSideProps

`getServerSideProps`는 페이지가 요청될 때마다 서버에서 데이터를 가져와 렌더링하는 **Server-Side Rendering(SSR)** 함수입니다. 자주 변경되는 데이터를 가져와 항상 최신 상태의 페이지를 제공할 때 유용합니다.

---

## 기본 사용법

```tsx
import type { InferGetServerSidePropsType, GetServerSideProps } from 'next'

type Data = {
  message: string
}

export const getServerSideProps: GetServerSideProps<{ data: Data }> = async (context) => {
  const res = await fetch('https://api.example.com/data')
  const data: Data = await res.json()

  return {
    props: { data },
  }
}

export default function Page({
  data,
}: InferGetServerSidePropsType<typeof getServerSideProps>) {
  return <p>{data.message}</p>
}
```

---

## Context 파라미터

`getServerSideProps`는 `context` 객체를 파라미터로 받습니다.

| 파라미터 | 타입 | 설명 |
|----------|------|------|
| `params` | `object` | 동적 라우트의 파라미터 (예: `[id].js`는 `{ id: ... }`) |
| `req` | `IncomingMessage` | HTTP 요청 객체 (쿠키 접근 가능) |
| `res` | `ServerResponse` | HTTP 응답 객체 |
| `query` | `object` | 쿼리 스트링 및 동적 라우트 파라미터 |
| `preview` | `boolean` | Preview Mode 상태 (deprecated) |
| `previewData` | `any` | Preview 데이터 (deprecated) |
| `draftMode` | `boolean` | Draft Mode 활성화 여부 |
| `resolvedUrl` | `string` | 정규화된 요청 URL (`_next/data` 프리픽스 제거) |
| `locale` | `string` | 현재 활성 로케일 |
| `locales` | `string[]` | 지원하는 모든 로케일 |
| `defaultLocale` | `string` | 기본 로케일 |

---

## 반환값

`getServerSideProps`는 다음 속성을 포함하는 객체를 반환해야 합니다.

### props

페이지 컴포넌트에 전달될 직렬화 가능한 객체입니다.

```js
export async function getServerSideProps() {
  return {
    props: {
      message: 'Hello from Server!',
    },
  }
}
```

### notFound

`true`로 설정하면 404 상태를 반환합니다.

```js
export async function getServerSideProps(context) {
  const res = await fetch(`https://api.example.com/users/${context.params.id}`)
  const user = await res.json()

  if (!user) {
    return {
      notFound: true,
    }
  }

  return {
    props: { user },
  }
}
```

### redirect

내부 또는 외부 리소스로 리다이렉트합니다.

```js
export async function getServerSideProps(context) {
  const session = await getSession(context.req)

  if (!session) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    }
  }

  return {
    props: { user: session.user },
  }
}
```

---

## 실제 예제

### 요청 헤더 및 쿠키 접근

```tsx
// pages/profile.tsx
import type { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async ({ req, res }) => {
  // 쿠키에서 토큰 가져오기
  const token = req.cookies.token

  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    }
  }

  // 사용자 데이터 가져오기
  const userRes = await fetch('https://api.example.com/me', {
    headers: {
      Authorization: `Bearer ${token}`,
    },
  })
  const user = await userRes.json()

  return {
    props: { user },
  }
}

export default function ProfilePage({ user }) {
  return (
    <div>
      <h1>프로필: {user.name}</h1>
      <p>이메일: {user.email}</p>
    </div>
  )
}
```

### 쿼리 파라미터 사용

```tsx
// pages/search.tsx
import type { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async ({ query }) => {
  const { q, page = '1' } = query

  if (!q) {
    return {
      redirect: {
        destination: '/',
        permanent: false,
      },
    }
  }

  const res = await fetch(
    `https://api.example.com/search?q=${q}&page=${page}`
  )
  const results = await res.json()

  return {
    props: {
      query: q,
      results,
      currentPage: parseInt(page as string),
    },
  }
}

export default function SearchPage({ query, results, currentPage }) {
  return (
    <div>
      <h1>"{query}" 검색 결과</h1>
      <p>페이지: {currentPage}</p>
      <ul>
        {results.map((item) => (
          <li key={item.id}>{item.title}</li>
        ))}
      </ul>
    </div>
  )
}
```

### 데이터베이스 직접 쿼리

```tsx
// pages/posts/[id].tsx
import type { GetServerSideProps } from 'next'
import { db } from '@/lib/db'

export const getServerSideProps: GetServerSideProps = async ({ params }) => {
  const post = await db.post.findUnique({
    where: { id: params?.id as string },
    include: {
      author: true,
      comments: {
        orderBy: { createdAt: 'desc' },
      },
    },
  })

  if (!post) {
    return { notFound: true }
  }

  return {
    props: {
      post: JSON.parse(JSON.stringify(post)), // 직렬화
    },
  }
}
```

### 캐시 헤더 설정

```tsx
// pages/api-data.tsx
import type { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async ({ res }) => {
  // 10초 동안 캐시, 59초 동안 stale-while-revalidate
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  )

  const data = await fetch('https://api.example.com/data').then((r) => r.json())

  return {
    props: { data },
  }
}
```

### 에러 처리

```tsx
// pages/product/[id].tsx
import type { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async ({ params, res }) => {
  try {
    const product = await fetch(
      `https://api.example.com/products/${params?.id}`
    )

    if (!product.ok) {
      if (product.status === 404) {
        return { notFound: true }
      }
      throw new Error('Failed to fetch product')
    }

    const data = await product.json()

    return {
      props: { product: data },
    }
  } catch (error) {
    // 에러 로깅
    console.error('Error fetching product:', error)

    // 에러 페이지로 리다이렉트
    return {
      redirect: {
        destination: '/error',
        permanent: false,
      },
    }
  }
}
```

### 국제화(i18n) 지원

```tsx
// pages/index.tsx
import type { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async ({ locale }) => {
  const messages = await import(`../locales/${locale}.json`)

  return {
    props: {
      messages: messages.default,
      locale,
    },
  }
}

export default function HomePage({ messages, locale }) {
  return (
    <div>
      <p>현재 언어: {locale}</p>
      <h1>{messages.welcome}</h1>
    </div>
  )
}
```

---

## getStaticProps vs getServerSideProps

| 특성 | `getStaticProps` | `getServerSideProps` |
|------|-----------------|---------------------|
| 실행 시점 | 빌드 타임 (+ ISR) | 매 요청마다 |
| 캐싱 | CDN 캐시 가능 | 기본적으로 캐시 안 됨 |
| TTFB | 빠름 | 상대적으로 느림 |
| 데이터 신선도 | ISR로 제어 | 항상 최신 |
| 사용 사례 | 블로그, 문서 | 대시보드, 인증 필요 페이지 |

---

## 주요 특징

| 특징 | 설명 |
|------|------|
| 서버 전용 코드 | 클라이언트에서 실행되지 않음 |
| 데이터베이스 접근 | 직접 데이터베이스 쿼리 가능 |
| 번들 제외 | 클라이언트 번들에 포함되지 않음 |
| 요청 정보 접근 | `req`, `res`, 쿠키, 헤더 접근 가능 |
| 매 요청 실행 | 항상 최신 데이터 제공 |

---

## 중요한 주의사항

> **Good to know**:
> - `getServerSideProps`는 서버에서만 실행됩니다
> - 페이지 파일에서만 export할 수 있습니다
> - Time to First Byte(TTFB)가 `getStaticProps`보다 느립니다
> - 가능하면 `getStaticProps` + ISR을 먼저 고려하세요

> **성능 팁**:
> - `res.setHeader`로 캐시 헤더를 설정하여 성능을 개선하세요
> - 데이터베이스 쿼리를 최적화하세요
> - 불필요한 데이터를 props에 포함하지 마세요

> **App Router 마이그레이션**:
> - App Router에서는 Server Components와 `fetch`로 대체됩니다
> - `cookies()`, `headers()` 함수를 사용할 수 있습니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.4.0 | App Router 안정화 (데이터 페칭 간소화) |
| v10.0.0 | `locale`, `locales`, `defaultLocale`, `notFound` 옵션 추가 |
| v9.3.0 | `getServerSideProps` 도입 |

---

## 관련 문서

- [getStaticProps](./getStaticProps.md)
- [getStaticPaths](./getStaticPaths.md)
- [Data Fetching](../../app-router/guides/data-fetching-patterns.md)
