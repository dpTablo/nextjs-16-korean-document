# proxy.js

> **참고**: `middleware` 파일 규칙은 더 이상 사용되지 않으며 `proxy`로 변경되었습니다. 자세한 내용은 [Proxy로 마이그레이션](#proxy로-마이그레이션)을 참조하세요.

`proxy.js|ts` 파일은 [Proxy](/app-router/getting-started/16-proxy.md)를 작성하고 요청이 완료되기 전에 서버에서 코드를 실행하는 데 사용됩니다. 들어오는 요청을 기반으로 응답을 수정하거나 요청을 리다이렉트하고, 요청 또는 응답 헤더를 수정하거나 직접 응답할 수 있습니다.

Proxy는 라우트가 렌더링되기 전에 실행됩니다. 인증, 로깅 또는 리다이렉트 처리와 같은 사용자 정의 서버 측 로직을 구현하는 데 특히 유용합니다.

> **알아두기**:
>
> Proxy는 렌더링 코드와 별개로 호출되어야 하며, 최적화된 경우 CDN에 배포되어 빠른 리다이렉트/리라이트 처리를 위해 공유 모듈이나 글로벌에 의존해서는 안 됩니다.
>
> Proxy에서 애플리케이션으로 정보를 전달하려면 헤더, 쿠키, 리라이트, 리다이렉트 또는 URL을 사용하세요.

프로젝트 루트 또는 `src` 내부(해당되는 경우)에 `proxy.ts` (또는 `.js`) 파일을 생성하여 `pages` 또는 `app`과 같은 수준에 위치하도록 하세요.

```tsx filename="proxy.ts"
import { NextResponse, NextRequest } from 'next/server'

// 이 함수는 내부에서 `await`를 사용하는 경우 `async`로 표시할 수 있습니다
export function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

export const config = {
  matcher: '/about/:path*',
}
```

```js filename="proxy.js"
import { NextResponse } from 'next/server'

// 이 함수는 내부에서 `await`를 사용하는 경우 `async`로 표시할 수 있습니다
export function proxy(request) {
  return NextResponse.redirect(new URL('/home', request.url))
}

export const config = {
  matcher: '/about/:path*',
}
```

## 내보내기 (Exports)

### Proxy 함수

파일은 기본 내보내기 또는 `proxy`라는 명명된 내보내기로 단일 함수를 내보내야 합니다. 같은 파일에서 여러 proxy는 지원되지 않습니다.

```js filename="proxy.js"
// 기본 내보내기의 예
export default function proxy(request) {
  // Proxy 로직
}
```

### 구성 객체 (선택 사항)

선택적으로 Proxy 함수와 함께 구성 객체를 내보낼 수 있습니다. 이 객체에는 Proxy가 적용되는 경로를 지정하는 [matcher](#matcher)가 포함됩니다.

## Matcher

`matcher` 옵션을 사용하면 Proxy를 실행할 특정 경로를 대상으로 지정할 수 있습니다. 여러 방법으로 경로를 지정할 수 있습니다:

- 단일 경로의 경우: 문자열을 직접 사용하여 경로를 정의합니다 (예: `'/about'`)
- 여러 경로의 경우: 배열을 사용하여 여러 경로를 나열합니다 (예: `matcher: ['/about', '/contact']`)

```js filename="proxy.js"
export const config = {
  matcher: ['/about/:path*', '/dashboard/:path*'],
}
```

또한 `matcher` 옵션은 정규 표현식을 사용한 복잡한 경로 사양을 지원합니다:

```js filename="proxy.js"
export const config = {
  matcher: [
    // API 라우트, 정적 파일, 이미지 최적화 및 .png 파일 제외
    '/((?!api|_next/static|_next/image|.*\\.png$).*)',
  ],
}
```

### Matcher 규칙

구성된 matcher:

1. `/`로 시작해야 합니다
2. 명명된 매개변수를 포함할 수 있습니다: `/about/:path`는 `/about/a` 및 `/about/b`와 일치하지만 `/about/a/c`는 일치하지 않습니다
3. 명명된 매개변수(`:` 시작)에 수정자를 가질 수 있습니다: `/about/:path*`는 `/about/a/b/c`와 일치합니다. `*`는 *0개 이상*, `?`는 *0개 또는 1개*, `+`는 *1개 이상*입니다
4. 괄호로 묶인 정규 표현식을 사용할 수 있습니다: `/about/(.*)`는 `/about/:path*`와 동일합니다

## 매개변수 (Params)

### `request`

Proxy를 정의할 때, 기본 내보내기 함수는 `request`라는 단일 매개변수를 허용합니다. 이 매개변수는 들어오는 HTTP 요청을 나타내는 `NextRequest`의 인스턴스입니다.

```tsx filename="proxy.ts"
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  // Proxy 로직이 여기에 들어갑니다
}
```

## NextResponse

`NextResponse` API를 사용하면:

- 들어오는 요청을 다른 URL로 `redirect`
- 주어진 URL을 표시하여 응답을 `rewrite`
- API Routes, `getServerSideProps` 및 `rewrite` 대상에 대한 요청 헤더 설정
- 응답 쿠키 설정
- 응답 헤더 설정

## 실행 순서

Proxy는 **프로젝트의 모든 라우트에 대해** 호출됩니다. 따라서 `matcher`를 사용하여 특정 라우트를 정확히 대상으로 지정하거나 제외하는 것이 중요합니다. 다음은 실행 순서입니다:

1. `next.config.js`의 `headers`
2. `next.config.js`의 `redirects`
3. Proxy (`rewrites`, `redirects` 등)
4. `next.config.js`의 `beforeFiles` (`rewrites`)
5. 파일 시스템 라우트 (`public/`, `_next/static/`, `pages/`, `app/` 등)
6. `next.config.js`의 `afterFiles` (`rewrites`)
7. 동적 라우트 (`/blog/[slug]`)
8. `next.config.js`의 `fallback` (`rewrites`)

## 런타임

Proxy는 기본적으로 Node.js 런타임을 사용합니다.

## 예제

### 조건부 명령문

```ts filename="proxy.ts"
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }

  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}
```

### 쿠키 사용

```ts filename="proxy.ts"
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  // 들어오는 요청에 "Cookie:nextjs=fast" 헤더가 있다고 가정
  let cookie = request.cookies.get('nextjs')
  console.log(cookie) // => { name: 'nextjs', value: 'fast', Path: '/' }

  const allCookies = request.cookies.getAll()
  console.log(allCookies) // => [{ name: 'nextjs', value: 'fast' }]

  request.cookies.has('nextjs') // => true
  request.cookies.delete('nextjs')
  request.cookies.has('nextjs') // => false

  // 응답에 쿠키 설정
  const response = NextResponse.next()
  response.cookies.set('vercel', 'fast')
  response.cookies.set({
    name: 'vercel',
    value: 'fast',
    path: '/',
  })

  return response
}
```

### 헤더 설정

```ts filename="proxy.ts"
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  // 요청 헤더를 복제하고 새 헤더 설정
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-hello-from-proxy1', 'hello')

  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  })

  // 응답 헤더 설정
  response.headers.set('x-hello-from-proxy2', 'hello')
  return response
}
```

### CORS

```tsx filename="proxy.ts"
import { NextRequest, NextResponse } from 'next/server'

const allowedOrigins = ['https://acme.com', 'https://my-app.org']

const corsOptions = {
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
}

export function proxy(request: NextRequest) {
  const origin = request.headers.get('origin') ?? ''
  const isAllowedOrigin = allowedOrigins.includes(origin)

  // 사전 검사 요청 처리
  const isPreflight = request.method === 'OPTIONS'

  if (isPreflight) {
    const preflightHeaders = {
      ...(isAllowedOrigin && { 'Access-Control-Allow-Origin': origin }),
      ...corsOptions,
    }
    return NextResponse.json({}, { headers: preflightHeaders })
  }

  // 단순 요청 처리
  const response = NextResponse.next()

  if (isAllowedOrigin) {
    response.headers.set('Access-Control-Allow-Origin', origin)
  }

  Object.entries(corsOptions).forEach(([key, value]) => {
    response.headers.set(key, value)
  })

  return response
}

export const config = {
  matcher: '/api/:path*',
}
```

### 응답 생성

```ts filename="proxy.ts"
import type { NextRequest } from 'next/server'
import { isAuthenticated } from '@lib/auth'

export const config = {
  matcher: '/api/:function*',
}

export function proxy(request: NextRequest) {
  if (!isAuthenticated(request)) {
    return Response.json(
      { success: false, message: 'authentication failed' },
      { status: 401 }
    )
  }
}
```

## Proxy로 마이그레이션

### 변경 이유

`middleware`의 이름을 바꾼 이유는 "middleware"라는 용어가 종종 Express.js middleware와 혼동되어 목적을 잘못 해석하게 될 수 있기 때문입니다. "Proxy"라는 이름은 이 기능이 할 수 있는 것을 명확히 합니다.

### 마이그레이션 방법

Next.js는 `middleware.ts`에서 `proxy.ts`로 마이그레이션하기 위한 codemod를 제공합니다:

```bash
npx @next/codemod@canary middleware-to-proxy .
```

codemod는 파일 이름과 함수 이름을 `middleware`에서 `proxy`로 변경합니다:

```diff
// middleware.ts -> proxy.ts

- export function middleware() {
+ export function proxy() {
```

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| `v16.0.0` | Middleware는 더 이상 사용되지 않으며 Proxy로 이름이 바뀜 |
| `v15.5.0` | Middleware는 이제 Node.js 런타임을 사용할 수 있습니다 (안정 버전) |
| `v15.2.0` | Middleware는 이제 Node.js 런타임을 사용할 수 있습니다 (실험적) |
| `v13.1.0` | 고급 Middleware 플래그 추가 |
| `v13.0.0` | Middleware는 요청 헤더, 응답 헤더를 수정하고 응답을 보낼 수 있습니다 |
| `v12.2.0` | Middleware는 안정적입니다 |
| `v12.0.0` | Middleware (Beta) 추가 |

## 관련 문서

- [NextRequest](/app-router/api-reference/functions/next-request-response.md) - NextRequest의 API 참조
- [NextResponse](/app-router/api-reference/functions/next-request-response.md) - NextResponse의 API 참조
