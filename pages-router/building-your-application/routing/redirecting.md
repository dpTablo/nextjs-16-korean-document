# Redirecting (리다이렉트)

Next.js에서 리다이렉트를 처리하는 방법에 대해 알아봅니다.

## 개요

Next.js에서 리다이렉트를 처리하는 세 가지 주요 방법이 있습니다:

| API | 목적 | 위치 | 상태 코드 |
|-----|------|------|-----------|
| `useRouter` | 클라이언트 사이드 네비게이션 수행 | 컴포넌트 | N/A |
| `next.config.js`의 `redirects` | 경로 기반 들어오는 요청 리다이렉트 | `next.config.js` 파일 | 307 (임시) 또는 308 (영구) |
| `NextResponse.redirect` | 조건 기반 들어오는 요청 리다이렉트 | 미들웨어 | 모든 코드 |

---

## 1. `useRouter()` 훅

컴포넌트 내에서 클라이언트 사이드 네비게이션을 위해 `useRouter`의 `push` 메서드를 사용합니다:

```tsx
// pages/index.tsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      대시보드
    </button>
  )
}
```

> **참고:** 프로그래밍 방식이 아닌 네비게이션에는 `<Link>` 컴포넌트를 대신 사용하세요.

### 다른 useRouter 메서드

```tsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return (
    <div>
      {/* 새 URL을 히스토리 스택에 추가 */}
      <button onClick={() => router.push('/about')}>About으로 이동</button>

      {/* 현재 URL을 대체 (히스토리 스택에 추가하지 않음) */}
      <button onClick={() => router.replace('/home')}>Home으로 대체</button>

      {/* 이전 페이지로 이동 */}
      <button onClick={() => router.back()}>뒤로 가기</button>
    </div>
  )
}
```

---

## 2. `next.config.js`의 `redirects`

빌드 시점에 들어오는 요청 경로를 다른 목적지로 리다이렉트합니다. 경로, 헤더, 쿠키, 쿼리 매칭을 지원합니다.

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  async redirects() {
    return [
      // 기본 리다이렉트
      {
        source: '/about',
        destination: '/',
        permanent: true,
      },
      // 와일드카드 경로 매칭
      {
        source: '/blog/:slug',
        destination: '/news/:slug',
        permanent: true,
      },
    ]
  },
}

export default nextConfig
```

### 고급 리다이렉트 설정

```js
// next.config.js
module.exports = {
  async redirects() {
    return [
      // 정규식을 사용한 리다이렉트
      {
        source: '/post/:slug(\\d{1,})',
        destination: '/news/:slug',
        permanent: false,
      },
      // 헤더 기반 리다이렉트
      {
        source: '/old-page',
        has: [
          {
            type: 'header',
            key: 'x-redirect-me',
          },
        ],
        destination: '/new-page',
        permanent: false,
      },
      // 쿠키 기반 리다이렉트
      {
        source: '/account',
        has: [
          {
            type: 'cookie',
            key: 'authorized',
            value: 'false',
          },
        ],
        destination: '/login',
        permanent: false,
      },
    ]
  },
}
```

**주요 사항:**
- `permanent` 옵션을 통해 307 (임시) 또는 308 (영구) 상태 코드 반환
- Vercel에서 1,024개의 리다이렉트로 제한
- 미들웨어 **이전에** 실행

---

## 3. 미들웨어에서 `NextResponse.redirect`

요청 완료 전에 코드를 실행하고 조건(인증, 세션 관리 등)에 따라 리다이렉트합니다:

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'
import { authenticate } from 'auth-provider'

export function middleware(request: NextRequest) {
  const isAuthenticated = authenticate(request)

  if (isAuthenticated) {
    return NextResponse.next()
  }

  return NextResponse.redirect(new URL('/login', request.url))
}

export const config = {
  matcher: '/dashboard/:path*',
}
```

> **참고:** 미들웨어는 `redirects` **이후**에, 렌더링 **이전에** 실행됩니다.

---

## 대규모 리다이렉트 관리 (1000개 이상)

많은 수의 리다이렉트를 위해 커스텀 솔루션과 함께 미들웨어를 사용합니다:

### 1단계: 리다이렉트 맵 생성

데이터베이스(Edge Config, Redis) 또는 JSON 파일에 리다이렉트를 저장합니다:

```json
{
  "/old": {
    "destination": "/new",
    "permanent": true
  },
  "/blog/post-old": {
    "destination": "/blog/post-new",
    "permanent": true
  }
}
```

### 2단계: 미들웨어에서 데이터베이스 읽기

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'
import { get } from '@vercel/edge-config'

type RedirectEntry = {
  destination: string
  permanent: boolean
}

export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  const redirectData = await get(pathname)

  if (redirectData && typeof redirectData === 'string') {
    const redirectEntry: RedirectEntry = JSON.parse(redirectData)
    const statusCode = redirectEntry.permanent ? 308 : 307
    return NextResponse.redirect(redirectEntry.destination, statusCode)
  }

  return NextResponse.next()
}
```

### 3단계: 블룸 필터로 최적화

더 나은 성능을 위해 전체 데이터베이스에 접근하기 전에 블룸 필터를 사용하여 리다이렉트가 존재하는지 확인합니다:

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'
import { ScalableBloomFilter } from 'bloom-filters'
import GeneratedBloomFilter from './redirects/bloom-filter.json'

type RedirectEntry = {
  destination: string
  permanent: boolean
}

const bloomFilter = ScalableBloomFilter.fromJSON(GeneratedBloomFilter as any)

export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname

  if (bloomFilter.has(pathname)) {
    const api = new URL(
      `/api/redirects?pathname=${encodeURIComponent(request.nextUrl.pathname)}`,
      request.nextUrl.origin
    )

    try {
      const redirectData = await fetch(api)

      if (redirectData.ok) {
        const redirectEntry: RedirectEntry | undefined =
          await redirectData.json()

        if (redirectEntry) {
          const statusCode = redirectEntry.permanent ? 308 : 307
          return NextResponse.redirect(redirectEntry.destination, statusCode)
        }
      }
    } catch (error) {
      console.error(error)
    }
  }

  return NextResponse.next()
}
```

---

## getServerSideProps에서 리다이렉트

```tsx
// pages/profile.tsx
import { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async (context) => {
  const { req } = context
  const session = await getSession(req)

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

## getStaticProps에서 리다이렉트

```tsx
// pages/old-page.tsx
import { GetStaticProps } from 'next'

export const getStaticProps: GetStaticProps = async () => {
  return {
    redirect: {
      destination: '/new-page',
      permanent: true,
    },
  }
}
```

---

## 참고

- [useRouter](/pages-router/api-reference/functions/use-router.md)
- [next.config.js redirects](/app-router/api-reference/config/next-config-js/redirects.md)
- [미들웨어](/app-router/api-reference/file-conventions/middleware.md)
