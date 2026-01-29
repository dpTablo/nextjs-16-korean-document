---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/route
버전: 16.1.6
---

# route.js

Route Handlers를 사용하면 Web Request 및 Response API를 사용하여 주어진 라우트에 대한 커스텀 요청 핸들러를 생성할 수 있습니다.

```ts
export async function GET() {
  return Response.json({ message: 'Hello World' })
}
```

## 지원되는 HTTP 메서드

Route 파일은 다음 HTTP 메서드를 지원합니다:
- `GET`
- `HEAD`
- `POST`
- `PUT`
- `DELETE`
- `PATCH`
- `OPTIONS` (정의되지 않은 경우 Next.js가 자동으로 구현)

```ts
export async function GET(request: Request) {}
export async function POST(request: Request) {}
export async function PUT(request: Request) {}
export async function DELETE(request: Request) {}
export async function PATCH(request: Request) {}
export async function HEAD(request: Request) {}
export async function OPTIONS(request: Request) {}
```

## 매개변수

### `request` (선택사항)
Web Request API를 확장하는 `NextRequest` 객체로, 다음을 제공합니다:
- `cookies` 접근
- `nextUrl` - 확장되고 파싱된 URL 객체

```ts
import type { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  const url = request.nextUrl
}
```

### `context` (선택사항)
동적 라우트 매개변수로 해석되는 프로미스인 `params`를 포함합니다:

```ts
export async function GET(
  request: Request,
  { params }: { params: Promise<{ team: string }> }
) {
  const { team } = await params
}
```

**예제:**
| 라우트 | URL | params |
|-------|-----|--------|
| `app/dashboard/[team]/route.js` | `/dashboard/1` | `Promise<{ team: '1' }>` |
| `app/shop/[tag]/[item]/route.js` | `/shop/1/2` | `Promise<{ tag: '1', item: '2' }>` |
| `app/blog/[...slug]/route.js` | `/blog/1/2` | `Promise<{ slug: ['1', '2'] }>` |

### Route Context 헬퍼

강력하게 타입이 지정된 params를 위해 `RouteContext`를 사용하세요:

```ts
export async function GET(_req: NextRequest, ctx: RouteContext<'/users/[id]'>) {
  const { id } = await ctx.params
  return Response.json({ id })
}
```

*참고: 타입은 `next dev`, `next build` 또는 `next typegen` 중에 생성됩니다. import가 필요하지 않습니다.*

## 일반적인 사용 사례

### 쿠키

```ts
import { cookies } from 'next/headers'

export async function GET(request: NextRequest) {
  const cookieStore = await cookies()
  const a = cookieStore.get('a')
  const b = cookieStore.set('b', '1')
  const c = cookieStore.delete('c')
}
```

또는 기본 Web API를 사용하세요:
```ts
export async function GET(request: NextRequest) {
  const token = request.cookies.get('token')
}
```

### 헤더

```ts
import { headers } from 'next/headers'

export async function GET(request: NextRequest) {
  const headersList = await headers()
  const referer = headersList.get('referer')

  return new Response('Hello, Next.js!', {
    status: 200,
    headers: { referer: referer },
  })
}
```

### URL 쿼리 매개변수

```ts
import { type NextRequest } from 'next/server'

export function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const query = searchParams.get('query')
  // /api/search?query=hello의 경우 query는 "hello"
}
```

### Request Body

```ts
export async function POST(request: Request) {
  const res = await request.json()
  return Response.json({ res })
}
```

### FormData

```ts
export async function POST(request: Request) {
  const formData = await request.formData()
  const name = formData.get('name')
  const email = formData.get('email')
  return Response.json({ name, email })
}
```

### 스트리밍

```ts
import { openai } from '@ai-sdk/openai'
import { StreamingTextResponse, streamText } from 'ai'

export async function POST(req: Request) {
  const { messages } = await req.json()
  const result = await streamText({
    model: openai('gpt-4-turbo'),
    messages,
  })

  return new StreamingTextResponse(result.toAIStream())
}
```

### CORS

```ts
export async function GET(request: Request) {
  return new Response('Hello, Next.js!', {
    status: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  })
}
```

### Webhooks

```ts
export async function POST(request: Request) {
  try {
    const text = await request.text()
    // webhook payload 처리
  } catch (error) {
    return new Response(`Webhook error: ${error.message}`, {
      status: 400,
    })
  }

  return new Response('Success!', { status: 200 })
}
```

### 리디렉션

```ts
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  redirect('https://nextjs.org/')
}
```

### 데이터 재검증

```ts
export const revalidate = 60

export async function GET() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return Response.json(posts)
}
```

### 비-UI 응답

```ts
export async function GET() {
  return new Response(
    `<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
  <channel>
    <title>Next.js Documentation</title>
  </channel>
</rss>`,
    {
      headers: { 'Content-Type': 'text/xml' },
    }
  )
}
```

## 세그먼트 설정 옵션

```ts
export const dynamic = 'auto'
export const dynamicParams = true
export const revalidate = false
export const fetchCache = 'auto'
export const runtime = 'nodejs'
export const preferredRegion = 'auto'
```

## 버전 히스토리

| 버전 | 변경사항 |
|---------|---------|
| `v15.0.0-RC` | `context.params`가 이제 프로미스입니다; `GET`의 기본 캐싱이 동적으로 변경됨 |
| `v13.2.0` | Route Handlers 도입 |

## 관련 문서

- [Route Handlers](../../getting-started/11-route-handlers.md)
- [redirect](../functions/redirect.md)
- [cookies](../functions/cookies.md)
- [headers](../functions/headers.md)
- [revalidatePath](../functions/revalidatePath.md)
