---
원문: https://nextjs.org/docs/app/api-reference/functions/next-request
버전: 16.1.6
---

# NextRequest & NextResponse

## 개요

`NextRequest`와 `NextResponse`는 Next.js에서 HTTP 요청과 응답을 처리하기 위한 유틸리티 클래스입니다. 각각 Web Request API와 Web Response API를 확장하여 편리한 메서드를 추가합니다.

---

# NextRequest

`NextRequest`는 [Web Request API](https://developer.mozilla.org/docs/Web/API/Request)를 확장하여 Next.js에서 HTTP 요청을 처리하는 추가 편의 메서드를 제공합니다.

## Cookies API

### `cookies.set(name, value)`
요청에 쿠키를 설정합니다.
```ts
request.cookies.set('show-banner', 'false')
// 결과: Set-Cookie:show-banner=false;path=/home 헤더
```

### `cookies.get(name)`
이름으로 단일 쿠키 값을 반환하며, 찾을 수 없으면 `undefined`를 반환합니다.
```ts
request.cookies.get('show-banner')
// 반환: { name: 'show-banner', value: 'false', Path: '/home' }
```

### `cookies.getAll(name?)`
이름과 일치하는 모든 쿠키를 반환하며, 이름이 제공되지 않으면 모든 쿠키를 반환합니다.
```ts
request.cookies.getAll('experiments')
// 반환: [
//   { name: 'experiments', value: 'new-pricing-page', Path: '/home' },
//   { name: 'experiments', value: 'winter-launch', Path: '/home' }
// ]

request.cookies.getAll() // 모든 쿠키 가져오기
```

### `cookies.delete(name)`
요청에서 쿠키를 삭제합니다. 삭제되면 `true`, 그렇지 않으면 `false`를 반환합니다.
```ts
request.cookies.delete('experiments')
```

### `cookies.has(name)`
요청에 쿠키가 존재하는지 확인합니다.
```ts
request.cookies.has('experiments') // true 또는 false
```

### `cookies.clear()`
요청에서 모든 쿠키를 제거합니다.
```ts
request.cookies.clear()
```

## URL API (`nextUrl`)

Next.js 전용 속성으로 네이티브 [`URL` API](https://developer.mozilla.org/docs/Web/API/URL)를 확장합니다.

### 속성

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `basePath` | `string` | URL의 base path |
| `buildId` | `string \| undefined` | 빌드 식별자 (커스터마이징 가능) |
| `pathname` | `string` | URL의 pathname |
| `searchParams` | `Object` | URL의 search 매개변수 |

### 예제
```ts
// /home에 대한 요청인 경우
request.nextUrl.pathname // "/home"

// /home?name=lee에 대한 요청인 경우
request.nextUrl.searchParams // { 'name': 'lee' }
```

---

# NextResponse

`NextResponse`는 서버 측 코드에서 응답을 처리하기 위한 편의 메서드로 Web Response API를 확장하는 Next.js 유틸리티입니다.

## Cookies 관리

### `cookies.set(name, value)`
응답에 쿠키를 설정합니다.
```ts
let response = NextResponse.next()
response.cookies.set('show-banner', 'false')
// 생성: Set-Cookie:show-banner=false;path=/home
return response
```

### `cookies.get(name)`
이름으로 단일 쿠키 값을 가져옵니다 (여러 개가 있으면 첫 번째를 반환).
```ts
response.cookies.get('show-banner')
// 반환: { name: 'show-banner', value: 'false', Path: '/home' }
```

### `cookies.getAll(name?)`
이름과 일치하는 모든 쿠키를 가져오거나, 이름이 제공되지 않으면 모든 응답 쿠키를 가져옵니다.
```ts
response.cookies.getAll('experiments')
// 반환: [
//   { name: 'experiments', value: 'new-pricing-page', Path: '/home' },
//   { name: 'experiments', value: 'winter-launch', Path: '/home' },
// ]
response.cookies.getAll() // 모든 쿠키 가져오기
```

### `cookies.has(name)`
응답에 쿠키가 존재하는지 확인합니다.
```ts
response.cookies.has('experiments') // 불리언 반환
```

### `cookies.delete(name)`
응답에서 쿠키를 삭제합니다.
```ts
response.cookies.delete('experiments') // 삭제되면 true, 그렇지 않으면 false 반환
```

## 정적 메서드

### `NextResponse.json(body, init?)`
선택적 상태 및 헤더와 함께 JSON 응답을 생성합니다.
```ts
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  return NextResponse.json(
    { error: 'Internal Server Error' },
    { status: 500 }
  )
}
```

### `NextResponse.redirect(url)`
지정된 URL로 리디렉션합니다.
```ts
import { NextResponse } from 'next/server'

return NextResponse.redirect(new URL('/new', request.url))
```

**쿼리 매개변수 예제:**
```ts
const loginUrl = new URL('/login', request.url)
loginUrl.searchParams.set('from', request.nextUrl.pathname)
return NextResponse.redirect(loginUrl)
```

### `NextResponse.rewrite(url)`
브라우저에 표시되는 원본 URL을 유지하면서 URL을 재작성(프록시)합니다.
```ts
import { NextResponse } from 'next/server'

// 들어오는 요청: /about (브라우저에 표시)
// 재작성된 요청: /proxy (내부 라우팅)
return NextResponse.rewrite(new URL('/proxy', request.url))
```

### `NextResponse.next(options?)`
조기 반환 및 라우팅 계속을 허용합니다. 미들웨어 및 프록시에 유용합니다.

#### 기본 사용법
```ts
import { NextResponse } from 'next/server'

return NextResponse.next()
```

#### 수정된 헤더를 업스트림으로 전달
```ts
const newHeaders = new Headers(request.headers)
newHeaders.set('x-version', '123')

return NextResponse.next({
  request: {
    headers: newHeaders,
  },
})
```

---

## 사용 예제

### Middleware에서 NextRequest 사용

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // URL 검사
  if (request.nextUrl.pathname.startsWith('/admin')) {
    // 인증 쿠키 확인
    const token = request.cookies.get('auth-token')

    if (!token) {
      // 로그인 페이지로 리디렉션
      const loginUrl = new URL('/login', request.url)
      loginUrl.searchParams.set('from', request.nextUrl.pathname)
      return NextResponse.redirect(loginUrl)
    }
  }

  return NextResponse.next()
}
```

### Route Handler에서 NextResponse 사용

```ts
// app/api/users/route.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  // 쿼리 매개변수 가져오기
  const searchParams = request.nextUrl.searchParams
  const query = searchParams.get('query')

  // 데이터 가져오기
  const users = await getUsers(query)

  // JSON 응답 반환
  const response = NextResponse.json(users)

  // 쿠키 설정
  response.cookies.set('last-search', query || '', {
    maxAge: 60 * 60 * 24, // 24시간
  })

  return response
}

export async function POST(request: NextRequest) {
  const body = await request.json()

  // 사용자 생성
  const user = await createUser(body)

  return NextResponse.json(user, { status: 201 })
}
```

### 헤더 수정

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // 요청 헤더 복제 및 수정
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-custom-header', 'my-value')

  // 수정된 헤더로 응답 생성
  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  })

  // 응답 헤더 설정
  response.headers.set('x-response-time', Date.now().toString())

  return response
}
```

### 쿠키 관리

```ts
// app/api/session/route.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  const { username, password } = await request.json()

  // 인증 확인
  const user = await authenticate(username, password)

  if (!user) {
    return NextResponse.json(
      { error: 'Invalid credentials' },
      { status: 401 }
    )
  }

  // 세션 생성
  const session = await createSession(user.id)

  const response = NextResponse.json({ success: true })

  // 세션 쿠키 설정
  response.cookies.set('session', session.token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7, // 1주일
  })

  return response
}

export async function DELETE(request: NextRequest) {
  const response = NextResponse.json({ success: true })

  // 세션 쿠키 삭제
  response.cookies.delete('session')

  return response
}
```

### Rewrite 예제

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // A/B 테스트
  const bucket = request.cookies.get('bucket')?.value

  if (bucket === 'a') {
    // 버전 A로 재작성
    return NextResponse.rewrite(new URL('/experiment/a', request.url))
  } else if (bucket === 'b') {
    // 버전 B로 재작성
    return NextResponse.rewrite(new URL('/experiment/b', request.url))
  }

  // 기본 페이지
  return NextResponse.next()
}
```

---

## 보안 권장 사항

### ⚠️ 중요: 헤더 전달 시 주의사항

`NextResponse.next({ headers })`를 사용하여 클라이언트에 헤더를 전달하지 마세요. 이는 다음과 같은 문제를 일으킬 수 있습니다:
- 프레임워크 기대치 재정의 (예: `Content-Type`)
- Server Actions 및 스트리밍 응답 중단
- 민감한 데이터 노출

### ✅ 모범 사례 - 허용 목록 헤더

```ts
function middleware(request: NextRequest) {
  const incoming = new Headers(request.headers)
  const forwarded = new Headers()

  for (const [name, value] of incoming) {
    const headerName = name.toLowerCase()
    // 안전한 헤더만 전달
    if (
      !headerName.startsWith('x-') &&
      headerName !== 'authorization' &&
      headerName !== 'cookie'
    ) {
      forwarded.set(name, value)
    }
  }

  return NextResponse.next({
    request: { headers: forwarded },
  })
}
```

### 주요 보안 권장 사항

1. **모든 요청 헤더 복사 피하기** - 허용 목록 접근 방식 사용
2. **인증 헤더를 클라이언트에 노출하지 않기**
3. **커스텀 헤더 필터링** (명시적으로 필요하지 않은 한 `x-*` 헤더 폐기)
4. **방어적으로 접근** - 외부 서비스로의 데이터 유출 방지를 위해 업스트림으로 헤더 전달 시

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.0.0 | `ip` 및 `geo` 속성 제거 |

**참고:** Pages Router의 국제화 속성은 App Router에서 사용할 수 없습니다.

---

## 관련 문서

- [Middleware](../file-conventions/middleware.md)
- [Route Handlers](../file-conventions/route.md)
- [cookies](./cookies.md)
- [headers](./headers.md)
- [redirect](./redirect.md)
