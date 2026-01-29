---
원문: https://nextjs.org/docs/app/getting-started/route-handlers
버전: 16.1.6
---

# Route Handlers

## 개요
Route Handlers를 사용하면 Web Request 및 Response API를 사용하여 주어진 라우트에 대한 커스텀 요청 핸들러를 생성할 수 있습니다. `app` 디렉토리 내에서만 사용 가능하며, `pages` 디렉토리의 API Routes와 동등합니다.

## 규칙

Route Handlers는 `app` 디렉토리 내의 `route.js|ts` 파일에서 정의됩니다:

```ts filename="app/api/route.ts"
export async function GET(request: Request) {}
```

```js filename="app/api/route.js"
export async function GET(request) {}
```

**중요**: `page.js`와 같은 라우트 세그먼트 레벨에 `route.js` 파일이 있을 수 **없습니다**.

## 지원되는 HTTP 메서드

다음 HTTP 메서드가 지원됩니다:
- `GET`
- `POST`
- `PUT`
- `PATCH`
- `DELETE`
- `HEAD`
- `OPTIONS`

지원되지 않는 메서드는 `405 Method Not Allowed` 응답을 반환합니다.

## 확장된 API

Next.js는 네이티브 Request 및 Response API를 다음과 함께 확장합니다:
- [`NextRequest`](/docs/app/api-reference/functions/next-request.md)
- [`NextResponse`](/docs/app/api-reference/functions/next-response.md)

## 캐싱

Route Handlers는 **기본적으로 캐시되지 않습니다**. `GET` 메서드를 캐시하려면 라우트 설정 옵션을 사용하세요:

```ts filename="app/items/route.ts"
export const dynamic = 'force-static'

export async function GET() {
  const res = await fetch('https://data.mongodb-api.com/...', {
    headers: {
      'Content-Type': 'application/json',
      'API-Key': process.env.DATA_API_KEY,
    },
  })
  const data = await res.json()
  return Response.json({ data })
}
```

**참고**: 캐시된 `GET` 메서드와 함께 있어도 다른 HTTP 메서드는 **캐시되지 않습니다**.

## 캐시 컴포넌트

캐시 컴포넌트가 활성화되면 `GET` Route Handlers는 일반 UI 라우트와 동일한 모델을 따릅니다:

**정적 예시** (빌드 시점에 사전 렌더링됨):
```tsx filename="app/api/project-info/route.ts"
export async function GET() {
  return Response.json({
    projectName: 'Next.js',
  })
}
```

**동적 예시** (요청 시점으로 연기됨):
```tsx filename="app/api/random-number/route.ts"
export async function GET() {
  return Response.json({
    randomNumber: Math.random(),
  })
}
```

**런타임 데이터 예시** (요청별 데이터 사용):
```tsx filename="app/api/user-agent/route.ts"
import { headers } from 'next/headers'

export async function GET() {
  const headersList = await headers()
  const userAgent = headersList.get('user-agent')
  return Response.json({ userAgent })
}
```

**캐시된 동적 예시** (`use cache` 사용):
```tsx filename="app/api/products/route.ts"
import { cacheLife } from 'next/cache'

export async function GET() {
  const products = await getProducts()
  return Response.json(products)
}

async function getProducts() {
  'use cache'
  cacheLife('hours')
  return await db.query('SELECT * FROM products')
}
```

> **참고**: `use cache`는 Route Handler 본문에서 직접 사용할 수 없으며, 헬퍼 함수로 추출해야 합니다.

## 사전 렌더링이 중지되는 경우

다음의 경우 사전 렌더링이 중지됩니다:
- 네트워크 요청 또는 데이터베이스 쿼리 접근
- 비동기 파일 시스템 작업
- Request 객체 속성 (`req.url`, `request.headers`, `request.cookies`, `request.body`)
- 런타임 API: [`cookies()`](/docs/app/api-reference/functions/cookies.md), [`headers()`](/docs/app/api-reference/functions/headers.md), [`connection()`](/docs/app/api-reference/functions/connection.md)
- 비결정적 작업

## 특별한 Route Handlers

`sitemap.ts`, `opengraph-image.tsx`, `icon.tsx` 및 기타 메타데이터 파일과 같은 특별한 Route Handlers는 동적 API 또는 동적 설정 옵션을 사용하지 않는 한 기본적으로 정적입니다.

## 라우트 해결

Route Handlers는 `page`와 같은 레이아웃이나 클라이언트 사이드 네비게이션에 **참여하지 않습니다**.

| Page | Route | 결과 |
|------|-------|--------|
| `app/page.js` | `app/route.js` | 충돌 |
| `app/page.js` | `app/api/route.js` | 유효 |
| `app/[user]/page.js` | `app/api/route.js` | 유효 |

각 `route.js` 또는 `page.js` 파일은 해당 라우트의 모든 HTTP 동사를 차지합니다.

## Route Context Helper

TypeScript에서는 [`RouteContext`](/docs/app/api-reference/file-conventions/route.md#route-context-helper) 헬퍼를 사용하여 context 매개변수를 타이핑하세요:

```ts filename="app/users/[id]/route.ts"
import type { NextRequest } from 'next/server'

export async function GET(_req: NextRequest, ctx: RouteContext<'/users/[id]'>) {
  const { id } = await ctx.params
  return Response.json({ id })
}
```

**참고**: 타입은 `next dev`, `next build`, 또는 `next typegen` 중에 생성됩니다.
