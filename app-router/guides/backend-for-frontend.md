# Next.js를 프론트엔드의 백엔드로 사용하는 방법

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-10-17

## 개요

Next.js는 "Backend for Frontend" 패턴을 지원하여 HTTP 요청을 처리하고 HTML뿐만 아니라 모든 콘텐츠 유형을 반환하는 공개 엔드포인트를 생성할 수 있습니다. 데이터 소스에 접근하고 원격 데이터 업데이트와 같은 부작용을 수행할 수 있습니다.

### 빠른 시작

API 지원이 포함된 새 Next.js 프로젝트 생성:

```bash
npx create-next-app@latest --api
```

### 중요 참고

Next.js 백엔드 기능은 **완전한 백엔드 대체물이 아닙니다**. 다음 역할을 하는 API 레이어입니다:
- 공개적으로 접근 가능
- 모든 HTTP 요청 처리
- 모든 콘텐츠 유형 반환 가능

### 구현 도구

- [Route Handlers](/app-router/api-reference/file-conventions/route.md)
- [`proxy`](/app-router/api-reference/file-conventions/proxy.md)
- Pages Router의 경우: [API Routes](/pages-router/building-your-application/routing/api-routes.md)

---

## 공개 엔드포인트

Route Handlers는 모든 클라이언트가 접근할 수 있는 공개 HTTP 엔드포인트입니다.

### Route Handler 생성

`route.ts` 또는 `route.js` 파일 규칙 사용:

```ts
// /app/api/route.ts
export function GET(request: Request) {}
```

이것은 `/api`로 전송된 `GET` 요청을 처리합니다.

### 에러 처리

예외를 발생시킬 수 있는 작업에 `try/catch` 블록 사용:

```ts
// /app/api/route.ts
import { submit } from '@/lib/submit'

export async function POST(request: Request) {
  try {
    await submit(request)
    return new Response(null, { status: 204 })
  } catch (reason) {
    const message =
      reason instanceof Error ? reason.message : '예상치 못한 오류'

    return new Response(message, { status: 500 })
  }
}
```

**중요:** 클라이언트에 전송되는 에러 메시지에 민감한 정보를 노출하지 마세요.

### 접근 제어

접근을 제한하려면 인증 및 권한 부여를 구현하세요. [인증](/app-router/guides/authentication.md)을 참조하세요.

---

## 콘텐츠 유형

Route Handlers는 JSON, XML, 이미지, 파일, 일반 텍스트를 포함한 비UI 응답을 제공할 수 있습니다.

### 내장 파일 규칙

Next.js는 일반적인 엔드포인트에 파일 규칙을 사용합니다:

- [`sitemap.xml`](/app-router/api-reference/file-conventions/metadata/sitemap.md)
- [`opengraph-image.jpg`, `twitter-image`](/app-router/api-reference/file-conventions/metadata/opengraph-image.md)
- [favicon, app icon, apple-icon](/app-router/api-reference/file-conventions/metadata/app-icons.md)
- [`manifest.json`](/app-router/api-reference/file-conventions/metadata/manifest.md)
- [`robots.txt`](/app-router/api-reference/file-conventions/metadata/robots.md)

### 커스텀 콘텐츠 유형

다음과 같은 커스텀 엔드포인트를 정의할 수 있습니다:
- `llms.txt`
- `rss.xml`
- `.well-known`

예를 들어, `app/rss.xml/route.ts`는 `rss.xml`용 Route Handler를 생성합니다:

```ts
// /app/rss.xml/route.ts
export async function GET(request: Request) {
  const rssResponse = await fetch(/* rss 엔드포인트 */)
  const rssData = await rssResponse.json()

  const rssFeed = `<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
 <title>${rssData.title}</title>
 <description>${rssData.description}</description>
 <link>${rssData.link}</link>
 <copyright>${rssData.copyright}</copyright>
 ${rssData.items.map((item) => {
   return `<item>
    <title>${item.title}</title>
    <description>${item.description}</description>
    <link>${item.link}</link>
    <pubDate>${item.publishDate}</pubDate>
    <guid isPermaLink="false">${item.guid}</guid>
 </item>`
 })}
</channel>
</rss>`

  const headers = new Headers({ 'content-type': 'application/xml' })

  return new Response(rssFeed, { headers })
}
```

**중요:** 마크업 생성에 사용되는 모든 입력을 정제하세요.

---

## 요청 페이로드 소비

Request [인스턴스 메서드](https://developer.mozilla.org/en-US/docs/Web/API/Request#instance_methods)인 `.json()`, `.formData()`, 또는 `.text()`를 사용하여 요청 본문에 접근합니다.

**참고:** `GET`과 `HEAD` 요청은 본문을 가지지 않습니다.

### JSON 페이로드

```ts
// /app/api/echo-body/route.ts
export async function POST(request: Request) {
  const res = await request.json()
  return Response.json({ res })
}
```

**중요:** 다른 시스템에 전달하기 전에 데이터를 검증하세요.

### FormData 페이로드 예제

```ts
// /app/api/send-email/route.ts
import { sendMail, validateInputs } from '@/lib/email-transporter'

export async function POST(request: Request) {
  const formData = await request.formData()
  const email = formData.get('email')
  const contents = formData.get('contents')

  try {
    await validateInputs({ email, contents })
    const info = await sendMail({ email, contents })

    return Response.json({ messageId: info.messageId })
  } catch (reason) {
    const message =
      reason instanceof Error ? reason.message : '예상치 못한 예외'

    return new Response(message, { status: 500 })
  }
}
```

### 요청 본문 여러 번 읽기

요청 본문은 한 번만 읽을 수 있습니다. 다시 읽어야 하는 경우 요청을 복제하세요:

```ts
// /app/api/clone/route.ts
export async function POST(request: Request) {
  try {
    const clonedRequest = request.clone()

    await request.body()
    await clonedRequest.body()
    await request.body() // 오류 발생

    return new Response(null, { status: 204 })
  } catch {
    return new Response(null, { status: 500 })
  }
}
```

---

## 데이터 조작

Route Handlers는 하나 이상의 소스에서 데이터를 변환, 필터링, 집계할 수 있습니다. 이렇게 하면 로직을 프론트엔드에서 분리하고 내부 시스템을 노출하지 않습니다. 또한 무거운 계산을 서버로 오프로드하여 클라이언트 배터리와 데이터 사용량을 줄일 수 있습니다.

```ts
// /app/api/weather/route.ts
import { parseWeatherData } from '@/lib/weather'

export async function POST(request: Request) {
  const body = await request.json()
  const searchParams = new URLSearchParams({ lat: body.lat, lng: body.lng })

  try {
    const weatherResponse = await fetch(`${weatherEndpoint}?${searchParams}`)

    if (!weatherResponse.ok) {
      /* 에러 처리 */
    }

    const weatherData = await weatherResponse.text()
    const payload = parseWeatherData.asJSON(weatherData)

    return new Response(payload, { status: 200 })
  } catch (reason) {
    const message =
      reason instanceof Error ? reason.message : '예상치 못한 예외'

    return new Response(message, { status: 500 })
  }
}
```

**알아두면 좋은 점:** 이 예제는 지리적 위치 데이터를 URL에 넣지 않기 위해 `POST`를 사용합니다. `GET` 요청은 캐시되거나 로깅되어 민감한 정보가 노출될 수 있습니다.

---

## 백엔드로 프록시

Route Handler를 다른 백엔드에 대한 `proxy`로 사용할 수 있습니다. 요청을 전달하기 전에 검증 로직을 추가하세요.

```ts
// /app/api/[...slug]/route.ts
import { isValidRequest } from '@/lib/utils'

export async function POST(request: Request, { params }) {
  const clonedRequest = request.clone()
  const isValid = await isValidRequest(clonedRequest)

  if (!isValid) {
    return new Response(null, { status: 400, statusText: 'Bad Request' })
  }

  const { slug } = await params
  const pathname = slug.join('/')
  const proxyURL = new URL(pathname, 'https://nextjs.org')
  const proxyRequest = new Request(proxyURL, request)

  try {
    return fetch(proxyRequest)
  } catch (reason) {
    const message =
      reason instanceof Error ? reason.message : '예상치 못한 예외'

    return new Response(message, { status: 500 })
  }
}
```

---

## NextRequest와 NextResponse

Next.js는 일반적인 작업을 단순화하는 메서드로 [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request)와 [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) Web API를 확장합니다.

`NextRequest`는 들어오는 요청에서 파싱된 값을 노출하는 [`nextUrl`](/app-router/api-reference/functions/next-request.md#nexturl) 속성을 포함하여 요청 경로명과 검색 매개변수에 쉽게 접근할 수 있습니다.

`NextResponse`는 `next()`, `json()`, `redirect()`, `rewrite()`와 같은 헬퍼를 제공합니다.

```ts
// /app/echo-pathname/route.ts
import { type NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const nextUrl = request.nextUrl

  if (nextUrl.searchParams.get('redirect')) {
    return NextResponse.redirect(new URL('/', request.url))
  }

  if (nextUrl.searchParams.get('rewrite')) {
    return NextResponse.rewrite(new URL('/', request.url))
  }

  return NextResponse.json({ pathname: nextUrl.pathname })
}
```

---

## 웹훅과 콜백 URL

Route Handlers를 사용하여 타사 애플리케이션에서 이벤트 알림을 받습니다.

### 웹훅 예제: 콘텐츠 변경 시 재검증

```ts
// /app/webhook/route.ts
import { type NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const token = request.nextUrl.searchParams.get('token')

  if (token !== process.env.REVALIDATE_SECRET_TOKEN) {
    return NextResponse.json({ success: false }, { status: 401 })
  }

  const tag = request.nextUrl.searchParams.get('tag')

  if (!tag) {
    return NextResponse.json({ success: false }, { status: 400 })
  }

  revalidateTag(tag)

  return NextResponse.json({ success: true })
}
```

### 콜백 URL 예제: 인증 리다이렉트

```ts
// /app/auth/callback/route.ts
import { type NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const token = request.nextUrl.searchParams.get('session_token')
  const redirectUrl = request.nextUrl.searchParams.get('redirect_url')

  const response = NextResponse.redirect(new URL(redirectUrl, request.url))

  response.cookies.set({
    value: token,
    name: '_token',
    path: '/',
    secure: true,
    httpOnly: true,
    expires: undefined, // 세션 쿠키
  })

  return response
}
```

---

## 리다이렉트

```ts
// app/api/route.ts
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  redirect('https://nextjs.org/')
}
```

[`redirect`](/app-router/api-reference/functions/redirect.md)와 [`permanentRedirect`](/app-router/api-reference/functions/permanentRedirect.md)에서 리다이렉트에 대해 자세히 알아보세요.

---

## 프록시

프로젝트당 하나의 `proxy` 파일만 허용됩니다. `config.matcher`를 사용하여 특정 경로를 타겟팅하세요. [`proxy`](/app-router/api-reference/file-conventions/proxy.md)에 대해 자세히 알아보세요.

`proxy`를 사용하여 요청이 라우트 경로에 도달하기 전에 응답을 생성합니다.

### 인증 확인 예제

```ts
// proxy.ts
import { isAuthenticated } from '@lib/auth'

export const config = {
  matcher: '/api/:function*',
}

export function proxy(request: Request) {
  if (!isAuthenticated(request)) {
    return Response.json(
      { success: false, message: '인증 실패' },
      { status: 401 }
    )
  }
}
```

### 프록시 요청 예제

```ts
// proxy.ts
import { NextResponse } from 'next/server'

export function proxy(request: Request) {
  if (request.nextUrl.pathname === '/proxy-this-path') {
    const rewriteUrl = new URL('https://nextjs.org')
    return NextResponse.rewrite(rewriteUrl)
  }
}
```

---

## 보안

### 헤더 작업

헤더가 어디로 가는지 신중하게 처리하고, 들어오는 요청 헤더를 나가는 응답에 직접 전달하지 마세요.

### 속도 제한

Next.js 백엔드에서 속도 제한을 구현할 수 있습니다:

```ts
// /app/resource/route.ts
import { NextResponse } from 'next/server'
import { checkRateLimit } from '@/lib/rate-limit'

export async function POST(request: Request) {
  const { rateLimited } = await checkRateLimit(request)

  if (rateLimited) {
    return NextResponse.json({ error: '속도 제한 초과' }, { status: 429 })
  }

  return new Response(null, { status: 204 })
}
```

### 페이로드 검증

들어오는 요청 데이터를 절대 신뢰하지 마세요. 콘텐츠 유형과 크기를 검증하고, 사용 전에 XSS에 대해 정제하세요.

### 보호된 리소스 접근

- 접근 권한 부여 전에 항상 자격 증명을 확인하세요. 인증과 권한 부여에 프록시만 의존하지 마세요.
- 응답과 백엔드 로그에서 민감하거나 불필요한 데이터를 제거하세요.
- 자격 증명과 API 키를 정기적으로 교체하세요.

---

## 주의사항

### Server Components

Server Components에서 Route Handlers를 통하지 않고 소스에서 직접 데이터를 페치하세요.

빌드 시 사전 렌더링되는 Server Components의 경우, Route Handlers를 사용하면 빌드 단계가 실패합니다. 빌드 중에는 이러한 요청을 수신하는 서버가 없기 때문입니다.

### Server Actions

Server Actions를 사용하면 클라이언트에서 서버 측 코드를 실행할 수 있습니다. 주요 목적은 프론트엔드 클라이언트에서 데이터를 변경하는 것입니다.

Server Actions는 대기열에 추가됩니다. 데이터 페칭에 사용하면 순차적 실행이 도입됩니다.

### `export` 모드

`export` 모드는 런타임 서버 없이 정적 사이트를 출력합니다. Next.js 런타임이 필요한 기능은 [지원되지 않습니다](/app-router/guides/static-exports.md#unsupported-features).

`export 모드`에서는 `dynamic` 라우트 세그먼트 구성을 `'force-static'`으로 설정한 `GET` Route Handlers만 지원됩니다:

```js
// app/hello-world/route.ts
export const dynamic = 'force-static'

export function GET() {
  return new Response('Hello World', { status: 200 })
}
```

### 배포 환경

일부 호스트는 Route Handlers를 람다 함수로 배포합니다. 이는 다음을 의미합니다:

- Route Handlers는 요청 간에 데이터를 공유할 수 없습니다.
- 환경이 파일 시스템에 쓰기를 지원하지 않을 수 있습니다.
- 장시간 실행되는 핸들러는 타임아웃으로 종료될 수 있습니다.
- WebSockets는 타임아웃이나 응답 생성 후 연결이 닫히기 때문에 작동하지 않습니다.

---

## API 참조

Route Handlers와 Proxy에 대해 자세히 알아보세요:

- [route.js](/app-router/api-reference/file-conventions/route.md) - route.js 특수 파일에 대한 API 참조.
- [proxy.js](/app-router/api-reference/file-conventions/proxy.md) - proxy.js 파일에 대한 API 참조.
