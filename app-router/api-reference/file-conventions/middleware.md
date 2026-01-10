# middleware.js / proxy.js

> **중요**: Next.js 16부터 `middleware`가 `proxy`로 이름이 변경되었습니다. 기능은 동일하지만 더 명확한 의미를 전달하기 위해 이름이 바뀌었습니다.

## 개요

`middleware.js` (또는 v16+에서 `proxy.js`) 파일을 사용하면 **라우트가 렌더링되기 전에** 실행되는 서버 측 코드를 작성할 수 있습니다. 이를 통해 인증, 로깅, 리디렉션 및 요청/응답 수정과 같은 커스텀 로직을 구현할 수 있습니다.

## 파일 위치 및 이름 지정

- **위치**: 프로젝트 루트 또는 `src/` 디렉토리 (`pages` 또는 `app`과 동일한 레벨)
- **파일명**: `middleware.ts` 또는 `middleware.js` (v16+에서는 `proxy.ts` 또는 `proxy.js`)
- **커스텀 확장자**: 커스텀 `pageExtensions`를 사용하는 경우 그에 따라 이름 지정 (예: `middleware.page.ts`)

## 기본 구조

```typescript
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

export const config = {
  matcher: '/about/:path*',
}
```

**Next.js 16+에서:**
```typescript
// proxy.ts
import { NextResponse, NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

export const config = {
  matcher: '/about/:path*',
}
```

## 내보내기

### 필수: Middleware/Proxy 함수
다음 중 하나를 내보내야 합니다:
- 이름 지정 내보내기: `export function middleware(request) { }` (또는 `proxy`)
- 기본 내보내기: `export default function middleware(request) { }` (또는 `proxy`)

### 선택사항: Config 객체
경로 타겟팅을 위한 matcher 규칙을 정의합니다.

```javascript
export const config = {
  matcher: '/about/:path*',
}
```

## Matcher 설정

### 기본 구문

```javascript
// 단일 경로
matcher: '/about'

// 여러 경로 (배열)
matcher: ['/about', '/contact']

// 와일드카드 패턴 사용
matcher: ['/about/:path*', '/dashboard/:path*']

// 부정 매칭 (정규식 lookahead)
matcher: '/((?!api|_next/static|_next/image|.*\\.png$).*)'
```

### Matcher 규칙

1. **반드시 `/`로 시작해야 함**
2. **이름이 지정된 매개변수**: `/about/:path` (일치: `/about/a`, `/about/b`, 불일치: `/about/a/c`)
3. **매개변수에 대한 수정자**:
   - `*` = 0개 이상
   - `?` = 0개 또는 1개
   - `+` = 1개 이상
   - 예: `/about/:path*`는 `/about/a/b/c`와 일치
4. **정규식 패턴**: `/about/(.*)`는 `/about/:path*`와 동일
5. **경로 시작에 고정**: `/about`는 `/about` 및 `/about/team`과 일치하지만 `/blog/about`와는 일치하지 않음

### 고급 Matcher 객체

```javascript
export const config = {
  matcher: [
    {
      source: '/api/:path*',
      locale: false,
      has: [
        { type: 'header', key: 'Authorization', value: 'Bearer Token' },
        { type: 'query', key: 'userId', value: '123' },
      ],
      missing: [{ type: 'cookie', key: 'session', value: 'active' }],
    },
  ],
}
```

**Matcher 객체 키**:
- `source`: 일치시킬 경로 또는 패턴
- `locale` (선택사항): 로케일 기반 라우팅을 무시할지 여부를 나타내는 불리언
- `has` (선택사항): 헤더, 쿼리 파라미터 또는 쿠키의 존재에 대한 조건
- `missing` (선택사항): 요청 요소의 부재에 대한 조건

## Request 매개변수: NextRequest

```typescript
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // 요청 속성 접근
  const pathname = request.nextUrl.pathname
  const headers = request.headers
  const cookies = request.cookies
}
```

**NextRequest vs NextResponse**:
- `NextRequest`: 들어오는 HTTP 요청을 나타냄
- `NextResponse`: HTTP 응답을 조작하고 전송하기 위한 클래스

## NextResponse API

다음을 수행할 수 있습니다:
- **다른 URL로 리디렉션**
- **주어진 URL을 표시하여 응답 재작성**
- **API Routes 및 재작성에 대한 요청 헤더 설정**
- **응답 쿠키 설정**
- **응답 헤더 설정**
- **응답 직접 반환** (v13.1.0+)

### 일반적인 작업

```typescript
// 리디렉션
return NextResponse.redirect(new URL('/home', request.url))

// 재작성
return NextResponse.rewrite(new URL('/about-2', request.url))

// 통과
return NextResponse.next()

// 직접 응답
return Response.json({ message: 'error' }, { status: 401 })
```

## 쿠키 처리

```typescript
import { NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  // 요청에서 쿠키 읽기
  const cookie = request.cookies.get('nextjs')
  const allCookies = request.cookies.getAll()

  const hasAuth = request.cookies.has('auth')
  request.cookies.delete('session')
  request.cookies.clear()

  // 응답에 쿠키 설정
  const response = NextResponse.next()
  response.cookies.set('vercel', 'fast')
  response.cookies.set({
    name: 'token',
    value: 'abc123',
    path: '/',
    httpOnly: true,
    secure: true,
  })

  return response
}
```

## 헤더 관리

```typescript
import { NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  // 요청 헤더 복제 및 수정
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-custom-header', 'value')

  // 수정된 헤더를 업스트림으로 전달
  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  })

  // 응답 헤더 설정
  response.headers.set('x-response-header', 'value')

  return response
}
```

**⚠️ 경고**: 큰 헤더는 431 (Request Header Fields Too Large) 오류를 방지하기 위해 피하세요.

## 실행 순서

Middleware/Proxy는 다음 순서로 실행됩니다:

1. `next.config.js`의 `headers`
2. `next.config.js`의 `redirects`
3. **Middleware/Proxy** (재작성, 리디렉션 등)
4. `next.config.js`의 `beforeFiles` 재작성
5. 파일시스템 라우트 (`public/`, `_next/static/`, `pages/`, `app/`)
6. `next.config.js`의 `afterFiles` 재작성
7. 동적 라우트 (`/blog/[slug]`)
8. `next.config.js`의 `fallback` 재작성

## 일반적인 사용 사례

### 조건부 라우팅

```typescript
export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }

  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}
```

### 인증 확인

```typescript
import { isAuthenticated } from '@lib/auth'

export const config = { matcher: '/api/:function*' }

export function middleware(request: NextRequest) {
  if (!isAuthenticated(request)) {
    return Response.json(
      { success: false, message: 'authentication failed' },
      { status: 401 }
    )
  }
}
```

### CORS 헤더

```typescript
const allowedOrigins = ['https://acme.com', 'https://my-app.org']

export function middleware(request: NextRequest) {
  const origin = request.headers.get('origin') ?? ''
  const isAllowedOrigin = allowedOrigins.includes(origin)

  // Preflight 처리
  if (request.method === 'OPTIONS') {
    const preflightHeaders = {
      ...(isAllowedOrigin && { 'Access-Control-Allow-Origin': origin }),
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    }
    return NextResponse.json({}, { headers: preflightHeaders })
  }

  const response = NextResponse.next()
  if (isAllowedOrigin) {
    response.headers.set('Access-Control-Allow-Origin', origin)
  }

  return response
}

export const config = { matcher: '/api/:path*' }
```

### 후행 슬래시 처리

```javascript
// next.config.js
module.exports = {
  skipTrailingSlashRedirect: true,
}

// middleware.js
const legacyPrefixes = ['/docs', '/blog']

export default async function middleware(req) {
  const { pathname } = req.nextUrl

  if (legacyPrefixes.some((prefix) => pathname.startsWith(prefix))) {
    return NextResponse.next()
  }

  if (!pathname.endsWith('/') && !pathname.match(/\.\w+$/)) {
    return NextResponse.redirect(
      new URL(`${req.nextUrl.pathname}/`, req.nextUrl)
    )
  }
}
```

## 고급 기능

### 부정 매칭

```javascript
export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)',
  ],
}
```

### `missing` 및 `has`로 우회

```javascript
export const config = {
  matcher: [
    {
      source: '/((?!api|_next/static|_next/image).*)',
      missing: [
        { type: 'header', key: 'next-router-prefetch' },
      ],
    },
  ],
}
```

### 백그라운드 작업을 위한 `waitUntil`

```typescript
import { NextFetchEvent } from 'next/server'

export function middleware(req: NextRequest, event: NextFetchEvent) {
  event.waitUntil(
    fetch('https://analytics.com', {
      method: 'POST',
      body: JSON.stringify({ pathname: req.nextUrl.pathname }),
    })
  )

  return NextResponse.next()
}
```

### 고급 플래그

**`skipTrailingSlashRedirect`**: 자동 후행 슬래시 처리 비활성화
**`skipMiddlewareUrlNormalize`**: 정규화 없이 원본 URL 보존

```javascript
// next.config.js
module.exports = {
  skipMiddlewareUrlNormalize: true,
}
```

## 런타임

- **기본**: Node.js 런타임
- **사용 불가**: Middleware/Proxy 파일에서 `runtime` 설정 옵션을 설정할 수 없습니다
- **오류 발생**: 런타임 설정 시 오류가 발생합니다

## 플랫폼 지원

| 배포 | 지원 |
|-----------|-----------|
| Node.js 서버 | ✅ 예 |
| Docker 컨테이너 | ✅ 예 |
| 정적 내보내기 | ❌ 아니오 |
| 어댑터 | 🔧 플랫폼별 |

## Middleware에서 Proxy로 마이그레이션 (v16+)

codemod를 실행하여 마이그레이션하세요:

```bash
npx @next/codemod@canary middleware-to-proxy .
```

변경 사항:
```diff
- export function middleware() {
+ export function proxy() {
```

## 버전 히스토리

| 버전 | 변경사항 |
|---------|---------|
| v16.0.0 | Middleware가 deprecated되고 Proxy로 이름 변경 |
| v15.5.0 | Node.js 런타임 지원 (안정) |
| v15.2.0 | Node.js 런타임 지원 (실험적) |
| v13.1.0 | 고급 플래그 추가 |
| v13.0.0 | 헤더 수정 및 응답 전송 가능 |
| v12.2.0 | 안정 릴리스 |

## 관련 문서

- [인증](../../guides/authentication.md)
- [cookies](../functions/cookies.md)
- [headers](../functions/headers.md)
- [redirect](../functions/redirect.md)

## Sources

- [File-system conventions: proxy.js | Next.js](https://nextjs.org/docs/app/api-reference/file-conventions/proxy)
- [Renaming Middleware to Proxy | Next.js](https://nextjs.org/docs/messages/middleware-to-proxy)
- [Next.js 16 | Next.js](https://nextjs.org/blog/next-16)
