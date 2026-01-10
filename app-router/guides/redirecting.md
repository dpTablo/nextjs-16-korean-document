# Redirecting (리디렉션)

Next.js에서 리디렉션을 처리하는 다양한 방법을 알아봅니다.

## 개요

Next.js는 다양한 사용 사례에 적합한 여러 리디렉션 방법을 제공합니다:

| API | 목적 | 위치 | 상태 코드 |
|-----|------|------|-----------|
| `redirect()` | 변경/이벤트 후 리디렉션 | Server Components, Server Actions, Route Handlers | 307 (임시) 또는 303 (Server Action) |
| `permanentRedirect()` | 변경 후 영구 리디렉션 | Server Components, Server Actions, Route Handlers | 308 (영구) |
| `useRouter()` | 클라이언트 사이드 네비게이션 | Client Components의 이벤트 핸들러 | N/A |
| `next.config.js`의 `redirects` | 경로 기반 리디렉션 | 구성 파일 | 307 또는 308 |
| `NextResponse.redirect()` | 미들웨어의 조건부 리디렉션 | Middleware | 임의 |

---

## 1. `redirect()` 함수

변경 또는 이벤트 후 임시 리디렉션에 사용됩니다.

### 기본 사용법

```ts
// app/actions.ts
'use server'

import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'

export async function createPost(id: string) {
  try {
    // 데이터베이스 호출
    // await db.post.create(...)
  } catch (error) {
    // 에러 처리
    throw error
  }

  revalidatePath('/posts')
  redirect(`/post/${id}`) // 새 포스트로 이동
}
```

### Server Component에서 사용

```tsx
// app/team/[id]/page.tsx
import { redirect } from 'next/navigation'

async function fetchTeam(id: string) {
  const res = await fetch(`https://api.example.com/team/${id}`)
  if (!res.ok) return undefined
  return res.json()
}

export default async function TeamPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const team = await fetchTeam(id)

  if (!team) {
    redirect('/teams') // 팀이 없으면 목록으로 리디렉션
  }

  return <div>Team: {team.name}</div>
}
```

### 주요 포인트

- **기본 상태 코드:** 307 (Server Action에서는 303)
- **try/catch 외부에서 호출:** `redirect()`는 에러를 throw하므로 try/catch 블록 외부에서 호출해야 합니다
- **절대 URL 지원:** 외부 리디렉션을 위한 절대 URL 허용
- **클라이언트 제한:** 클라이언트 컴포넌트의 이벤트 핸들러에서는 사용 불가

---

## 2. `permanentRedirect()` 함수

정규 URL이 변경될 때 영구 리디렉션에 사용됩니다.

### 기본 사용법

```ts
// app/actions.ts
'use server'

import { permanentRedirect } from 'next/navigation'
import { revalidateTag } from 'next/cache'

export async function updateUsername(username: string, formData: FormData) {
  try {
    // 데이터베이스에서 사용자 이름 업데이트
    // await db.user.update(...)
  } catch (error) {
    // 에러 처리
    throw error
  }

  revalidateTag('username')
  permanentRedirect(`/profile/${username}`) // 새 프로필 URL로 영구 이동
}
```

### 사용 예시

```tsx
// app/posts/[oldId]/page.tsx
import { permanentRedirect } from 'next/navigation'

export default async function OldPostPage({
  params,
}: {
  params: Promise<{ oldId: string }>
}) {
  const { oldId } = await params

  // 구 ID를 신 ID로 매핑
  const newId = await getNewPostId(oldId)

  if (newId) {
    permanentRedirect(`/posts/${newId}`)
  }

  return <div>Post not found</div>
}
```

### 주요 포인트

- **상태 코드:** 308 (Permanent Redirect)
- **사용 시기:** 엔티티 URL이 영구적으로 변경될 때
- **절대 URL 지원:** 외부 URL로도 리디렉션 가능
- **SEO 친화적:** 검색 엔진이 새 URL을 인덱싱

---

## 3. `useRouter()` Hook

클라이언트 사이드 이벤트 핸들러에서의 네비게이션입니다.

### 기본 사용법

```tsx
// app/page.tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.push('/dashboard')}>
      대시보드로 이동
    </button>
  )
}
```

### 고급 사용 패턴

```tsx
'use client'

import { useRouter } from 'next/navigation'

export function NavigationButtons() {
  const router = useRouter()

  return (
    <div>
      {/* 앞으로 이동 */}
      <button onClick={() => router.push('/dashboard')}>
        Push
      </button>

      {/* 현재 항목 교체 */}
      <button onClick={() => router.replace('/dashboard')}>
        Replace
      </button>

      {/* 뒤로 가기 */}
      <button onClick={() => router.back()}>
        Back
      </button>

      {/* 앞으로 가기 */}
      <button onClick={() => router.forward()}>
        Forward
      </button>

      {/* 새로고침 */}
      <button onClick={() => router.refresh()}>
        Refresh
      </button>
    </div>
  )
}
```

### 조건부 네비게이션

```tsx
'use client'

import { useRouter } from 'next/navigation'
import { useState } from 'react'

export function ConditionalRedirect() {
  const router = useRouter()
  const [isSubmitting, setIsSubmitting] = useState(false)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsSubmitting(true)

    try {
      const response = await fetch('/api/submit', {
        method: 'POST',
      })

      if (response.ok) {
        router.push('/success')
      } else {
        router.push('/error')
      }
    } finally {
      setIsSubmitting(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit" disabled={isSubmitting}>
        제출
      </button>
    </form>
  )
}
```

### 주요 포인트

- **클라이언트 전용:** 클라이언트 컴포넌트에서만 사용
- **Link 우선:** 프로그래매틱하지 않은 네비게이션은 `<Link>` 컴포넌트 사용
- **렌더링 중 불가:** 렌더링 단계에서는 리디렉션 불가

---

## 4. `next.config.js`의 `redirects`

빌드 시점에 들어오는 요청에 대한 리디렉션을 구성합니다.

### 기본 구성

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  async redirects() {
    return [
      // 기본 리디렉션
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
      // 정규식 경로 매칭
      {
        source: '/old-blog/:slug(\\d{1,})',
        destination: '/blog/:slug',
        permanent: true,
      },
    ]
  },
}

export default nextConfig
```

### 고급 패턴

**헤더 기반 리디렉션:**
```ts
{
  source: '/docs/:path*',
  has: [
    {
      type: 'header',
      key: 'x-redirect-me',
    },
  ],
  destination: '/other-docs/:path*',
  permanent: false,
}
```

**쿠키 기반 리디렉션:**
```ts
{
  source: '/dashboard',
  has: [
    {
      type: 'cookie',
      key: 'authorized',
      value: 'true',
    },
  ],
  destination: '/admin/dashboard',
  permanent: false,
}
```

**쿼리 파라미터 기반:**
```ts
{
  source: '/search',
  has: [
    {
      type: 'query',
      key: 'lang',
      value: 'ko',
    },
  ],
  destination: '/ko/search',
  permanent: false,
}
```

### 주요 포인트

- **경로, 헤더, 쿠키, 쿼리 매칭 지원**
- `permanent: true` → 308, `false` → 307
- **제한:** 최대 1,024개 (더 많이 필요하면 Middleware 사용)
- **실행 순서:** Middleware보다 **먼저** 실행됨

---

## 5. Middleware의 `NextResponse.redirect()`

런타임 조건에 따른 조건부 리디렉션입니다.

### 인증 체크

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'
import { authenticate } from '@/lib/auth'

export function middleware(request: NextRequest) {
  const isAuthenticated = authenticate(request)

  // 인증되지 않은 경우 로그인 페이지로 리디렉션
  if (!isAuthenticated) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: '/dashboard/:path*',
}
```

### 지역화 리디렉션

```ts
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

const locales = ['ko', 'en', 'ja']

function getLocale(request: NextRequest) {
  // 쿠키나 헤더에서 로케일 확인
  return request.cookies.get('locale')?.value || 'ko'
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl

  // 경로에 로케일이 있는지 확인
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  )

  if (pathnameHasLocale) return NextResponse.next()

  // 로케일로 리디렉션
  const locale = getLocale(request)
  request.nextUrl.pathname = `/${locale}${pathname}`
  return NextResponse.redirect(request.nextUrl)
}

export const config = {
  matcher: ['/((?!_next|api|favicon.ico).*)'],
}
```

### A/B 테스팅

```ts
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const bucket = request.cookies.get('bucket')?.value || Math.random() < 0.5 ? 'a' : 'b'

  const response = NextResponse.next()

  if (!request.cookies.get('bucket')) {
    response.cookies.set('bucket', bucket)
  }

  // 버킷에 따라 다른 페이지로 리디렉션
  if (bucket === 'b' && request.nextUrl.pathname === '/') {
    return NextResponse.redirect(new URL('/variant-b', request.url))
  }

  return response
}
```

### 주요 포인트

- **실행 시점:** `next.config.js` 리디렉션 **이후**, 렌더링 **이전**
- **런타임 조건:** 인증, 지역화, A/B 테스팅 등
- **임의 상태 코드:** 모든 HTTP 상태 코드 사용 가능
- **Edge Runtime:** 빠른 응답 시간

---

## 대규모 리디렉션 관리 (1000+)

수천 개의 리디렉션을 관리하려면 Middleware + 데이터베이스 + Bloom 필터를 사용하세요.

### 1. Bloom 필터 생성

```ts
// scripts/generate-bloom-filter.ts
import { ScalableBloomFilter } from 'bloom-filters'
import fs from 'fs'
import redirects from './redirects.json'

const bloomFilter = new ScalableBloomFilter()

Object.keys(redirects).forEach((pathname) => {
  bloomFilter.add(pathname)
})

const exported = bloomFilter.saveAsJSON()

fs.writeFileSync(
  './redirects/bloom-filter.json',
  JSON.stringify(exported)
)
```

### 2. Middleware 구현

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'
import { ScalableBloomFilter } from 'bloom-filters'
import GeneratedBloomFilter from './redirects/bloom-filter.json'

const bloomFilter = ScalableBloomFilter.fromJSON(
  GeneratedBloomFilter as any
)

export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname

  // Bloom 필터로 빠르게 체크
  if (bloomFilter.has(pathname)) {
    const api = new URL(
      `/api/redirects?pathname=${encodeURIComponent(pathname)}`,
      request.nextUrl.origin
    )

    try {
      const response = await fetch(api)

      if (response.ok) {
        const { destination, permanent } = await response.json()
        const statusCode = permanent ? 308 : 307
        return NextResponse.redirect(destination, statusCode)
      }
    } catch (error) {
      console.error('Redirect fetch error:', error)
    }
  }

  return NextResponse.next()
}
```

### 3. API Route 구현

```ts
// app/api/redirects/route.ts
import { NextRequest, NextResponse } from 'next/server'
import redirects from '@/redirects.json'

type RedirectEntry = {
  destination: string
  permanent: boolean
}

export function GET(request: NextRequest) {
  const pathname = request.nextUrl.searchParams.get('pathname')

  if (!pathname) {
    return new Response('Bad Request', { status: 400 })
  }

  const redirect = (redirects as Record<string, RedirectEntry>)[pathname]

  if (!redirect) {
    return new Response('No redirect', { status: 400 })
  }

  return NextResponse.json(redirect)
}
```

### 4. 리디렉션 데이터

```json
// redirects.json
{
  "/old-page-1": {
    "destination": "/new-page-1",
    "permanent": true
  },
  "/old-page-2": {
    "destination": "/new-page-2",
    "permanent": false
  }
}
```

### 이점

- ✅ **빠른 체크:** Bloom 필터가 데이터베이스 조회 전에 확인
- ✅ **메모리 효율:** 대용량 리디렉션 파일을 Middleware에 로드하지 않음
- ✅ **확장성:** 수천 개의 리디렉션 처리 가능
- ✅ **False Positive 허용:** 있을 수 있지만 API에서 최종 확인

---

## 비교 및 선택 가이드

| 시나리오 | 솔루션 | 이유 |
|----------|--------|------|
| 폼 제출 후 성공 페이지로 이동 | `redirect()` | Server Action 후 일회성 리디렉션 |
| 사용자 이름 변경 → 새 프로필 URL | `permanentRedirect()` | URL이 영구적으로 변경 |
| 버튼 클릭 → 네비게이션 | `useRouter()` | 클라이언트 사이드 프로그래매틱 네비게이션 |
| 알려진 URL 구조 변경 | `next.config.js` | 빌드 시점에 확정된 리디렉션 |
| 인증/권한 체크 | Middleware | 런타임 조건 체크 필요 |
| 1000+ 리디렉션 | Middleware + DB + Bloom 필터 | 확장성 및 성능 |

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **적절한 상태 코드 사용**
   - 307: 임시 리디렉션
   - 308: 영구 리디렉션

2. **SEO 고려**
   - 영구 URL 변경은 `permanentRedirect()` 사용
   - 검색 엔진이 새 URL을 인덱싱하도록 308 사용

3. **성능 최적화**
   - 정적 리디렉션은 `next.config.js` 사용
   - 동적 조건은 Middleware 사용

4. **보안**
   - 오픈 리디렉션 취약점 방지
   - 외부 URL 검증

### ❌ 피해야 할 것

1. **과도한 리디렉션 체인**
   ```
   /a → /b → /c (❌ 나쁨)
   /a → /c (✅ 좋음)
   ```

2. **클라이언트에서 민감한 체크**
   - 인증 체크는 서버나 Middleware에서

3. **try/catch 내부에서 redirect() 호출**
   ```ts
   // ❌ 잘못됨
   try {
     redirect('/somewhere')
   } catch (error) {}

   // ✅ 올바름
   try {
     // 작업
   } catch (error) {}
   redirect('/somewhere')
   ```

---

## 다음 단계

- [redirect() API](../api-reference/functions/redirect.md) - API 레퍼런스
- [permanentRedirect() API](../api-reference/functions/permanentRedirect.md) - API 레퍼런스
- [useRouter() API](../api-reference/functions/useRouter.md) - API 레퍼런스
- [Middleware](../api-reference/file-conventions/middleware.md) - Middleware 가이드

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
