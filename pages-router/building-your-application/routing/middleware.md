# Middleware (미들웨어)

미들웨어를 사용하면 요청이 완료되기 전에 코드를 실행할 수 있습니다. 들어오는 요청에 따라 응답을 재작성, 리다이렉트, 요청 또는 응답 헤더 수정, 또는 직접 응답할 수 있습니다.

## 규칙

미들웨어를 정의하려면 프로젝트 루트에 `middleware.ts` (또는 `.js`) 파일을 사용합니다. 예를 들어 `pages`나 `app`과 같은 수준, 또는 해당하는 경우 `src` 내부에 배치합니다.

## 예제

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

// 매칭할 경로 설정
export const config = {
  matcher: '/about/:path*',
}
```

## 경로 매칭

미들웨어는 프로젝트의 **모든 라우트**에 대해 호출됩니다. 실행 순서는 다음과 같습니다:

1. `next.config.js`의 `headers`
2. `next.config.js`의 `redirects`
3. 미들웨어 (`rewrites`, `redirects` 등)
4. `next.config.js`의 `beforeFiles` (`rewrites`)
5. 파일시스템 라우트 (`public/`, `_next/static/`, `pages/`, `app/` 등)
6. `next.config.js`의 `afterFiles` (`rewrites`)
7. 동적 라우트 (`/blog/[slug]`)
8. `next.config.js`의 `fallback` (`rewrites`)

### Matcher 설정

`matcher`를 사용하면 특정 경로에서만 미들웨어가 실행되도록 필터링할 수 있습니다.

```js
export const config = {
  matcher: '/about/:path*',
}
```

배열 문법으로 여러 경로 매칭:

```js
export const config = {
  matcher: ['/about/:path*', '/dashboard/:path*'],
}
```

정규식을 사용한 고급 매칭:

```js
export const config = {
  matcher: [
    /*
     * 다음으로 시작하는 경로를 제외한 모든 요청 경로 매칭:
     * - api (API 라우트)
     * - _next/static (정적 파일)
     * - _next/image (이미지 최적화 파일)
     * - favicon.ico (파비콘 파일)
     */
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
}
```

### 조건부 문

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }

  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}
```

## NextResponse

`NextResponse` API를 사용하면:

- 들어오는 요청을 다른 URL로 `redirect`
- 주어진 URL을 표시하여 응답을 `rewrite`
- API 라우트, `getServerSideProps`, `rewrite` 목적지에 대한 요청 헤더 설정
- 응답 쿠키 설정
- 응답 헤더 설정

### 리다이렉트

```ts
import { NextResponse } from 'next/server'

return NextResponse.redirect(new URL('/new-page', request.url))
```

### 재작성

```ts
import { NextResponse } from 'next/server'

return NextResponse.rewrite(new URL('/proxy', request.url))
```

### 헤더 설정

```ts
import { NextResponse } from 'next/server'

// 요청 헤더 복제 및 새 헤더 설정
const requestHeaders = new Headers(request.headers)
requestHeaders.set('x-hello-from-middleware1', 'hello')

// NextResponse.next에서도 요청 헤더 설정 가능
const response = NextResponse.next({
  request: {
    headers: requestHeaders,
  },
})

// 응답 헤더 설정
response.headers.set('x-hello-from-middleware2', 'hello')

return response
```

## 쿠키 사용

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // 들어오는 요청에 "Cookie:nextjs=fast" 헤더가 있다고 가정
  // `cookies` API를 사용하여 요청에서 쿠키 가져오기
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

  cookie = response.cookies.get('vercel')
  console.log(cookie) // => { name: 'vercel', value: 'fast', Path: '/' }

  return response
}
```

## 응답 생성

미들웨어에서 `Response` 또는 `NextResponse` 인스턴스를 반환하여 직접 응답할 수 있습니다.

```ts
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  return new Response('Hello, world!')
}
```

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
}
```

## 인증 예제

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const token = request.cookies.get('token')?.value

  // 토큰이 없으면 로그인 페이지로 리다이렉트
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  // 토큰이 있으면 계속 진행
  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/profile/:path*'],
}
```

## 고급 플래그

### `skipTrailingSlashRedirect`

```js
// next.config.js
module.exports = {
  skipTrailingSlashRedirect: true,
}
```

```ts
// middleware.ts
const legacyPrefixes = ['/docs', '/blog']

export default async function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl

  if (legacyPrefixes.some((prefix) => pathname.startsWith(prefix))) {
    return NextResponse.next()
  }

  // 후행 슬래시 처리 적용
  if (
    !pathname.endsWith('/') &&
    !pathname.match(/((?!\.well-known(?:\/.*)?)(?:[^/]+\/)*[^/]+\.\w+)/)
  ) {
    return NextResponse.redirect(
      new URL(`${req.nextUrl.pathname}/`, req.nextUrl)
    )
  }
}
```

### `skipMiddlewareUrlNormalize`

URL 정규화를 비활성화하여 직접 방문과 클라이언트 전환을 동일하게 처리합니다.

```js
// next.config.js
module.exports = {
  skipMiddlewareUrlNormalize: true,
}
```

## 런타임

미들웨어는 현재 [Edge 런타임](/app-router/api-reference/edge-runtime)만 지원합니다. Node.js 런타임은 사용할 수 없습니다.

---

## 참고

- [NextRequest](/app-router/api-reference/functions/next-request-response.md)
- [NextResponse](/app-router/api-reference/functions/next-request-response.md)
- [Edge 런타임](/app-router/api-reference/edge-runtime)
