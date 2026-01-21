# API Routes

`pages/api` 폴더 내의 파일은 `/api/*`로 매핑되어 페이지 대신 API 엔드포인트로 작동합니다. 이들은 서버 사이드 전용 번들로, 클라이언트 번들 크기를 증가시키지 않습니다.

> **참고**: App Router를 사용하는 경우 API Routes 대신 [Server Components](/docs/app/getting-started/server-and-client-components) 또는 [Route Handlers](/docs/app/api-reference/file-conventions/route)를 사용할 수 있습니다.

## 기본 사용법

```ts filename="pages/api/hello.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

type ResponseData = {
  message: string
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResponseData>
) {
  res.status(200).json({ message: 'Hello from Next.js!' })
}
```

## Request/Response 객체

### 매개변수

API Route 핸들러는 두 개의 매개변수를 받습니다:

- `req`: [http.IncomingMessage](https://nodejs.org/api/http.html#class-httpincomingmessage) 인스턴스
- `res`: [http.ServerResponse](https://nodejs.org/api/http.html#class-httpserverresponse) 인스턴스

### Request 헬퍼

`req` 객체에는 다음과 같은 헬퍼가 포함되어 있습니다:

| 헬퍼 | 설명 |
|------|------|
| `req.cookies` | 요청으로부터 받은 쿠키 객체 |
| `req.query` | 쿼리 스트링을 포함한 객체 |
| `req.body` | `content-type`으로 파싱된 본문 |

### Response 헬퍼

`res` 객체에는 다음과 같은 헬퍼가 포함되어 있습니다:

| 헬퍼 | 설명 |
|------|------|
| `res.status(code)` | HTTP 상태 코드를 설정합니다 |
| `res.json(body)` | JSON 응답을 전송합니다 |
| `res.send(body)` | HTTP 응답을 전송합니다 (string, object, Buffer) |
| `res.redirect([status,] path)` | 지정된 경로로 리다이렉트합니다 |
| `res.revalidate(urlPath)` | 온디맨드로 페이지를 재검증합니다 |

## HTTP 메서드 처리

API Route에서 다양한 HTTP 메서드를 처리하려면 `req.method`를 사용하세요:

```ts filename="pages/api/user.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'POST') {
    // POST 요청 처리
    res.status(200).json({ message: '사용자가 생성되었습니다' })
  } else if (req.method === 'GET') {
    // GET 요청 처리
    res.status(200).json({ message: '사용자 목록' })
  } else {
    // 다른 HTTP 메서드 처리
    res.setHeader('Allow', ['GET', 'POST'])
    res.status(405).end(`Method ${req.method} Not Allowed`)
  }
}
```

## 에러 처리

```ts filename="pages/api/data.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const result = await someAsyncOperation()
    res.status(200).json({ result })
  } catch (err) {
    res.status(500).json({ error: '데이터를 불러오는 데 실패했습니다' })
  }
}
```

## Custom Config

각 API Route는 기본 설정을 변경하기 위해 `config` 객체를 내보낼 수 있습니다:

```js filename="pages/api/upload.js"
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '1mb',
    },
  },
  maxDuration: 5,
}
```

### 주요 설정 옵션

| 옵션 | 기본값 | 설명 |
|------|--------|------|
| `bodyParser` | `true` | body 파싱 활성화 여부 |
| `bodyParser.sizeLimit` | `'1mb'` | 최대 body 크기 |
| `externalResolver` | `false` | 외부 resolver 사용 여부 |
| `responseLimit` | `'4mb'` | 응답 크기 경고 임계값 |

### Body Parser 비활성화

Webhook 검증 등 raw body가 필요한 경우 body parser를 비활성화할 수 있습니다:

```js filename="pages/api/webhook.js"
export const config = {
  api: {
    bodyParser: false,
  },
}

export default async function handler(req, res) {
  const rawBody = await getRawBody(req)
  // raw body로 작업 수행
}
```

### 응답 크기 제한 설정

```js filename="pages/api/large-response.js"
export const config = {
  api: {
    responseLimit: '8mb',
  },
}

// 또는 비활성화
export const config = {
  api: {
    responseLimit: false,
  },
}
```

## 동적 라우트

### 기본 동적 라우트

```ts filename="pages/api/post/[pid].ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const { pid } = req.query
  res.end(`Post: ${pid}`)
}
```

`/api/post/abc` 요청 시 → 응답: `Post: abc`

### Catch All 라우트

```ts filename="pages/api/post/[...slug].ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const { slug } = req.query
  res.end(`Post: ${(slug as string[]).join(', ')}`)
}
```

| URL | `req.query` |
|-----|-------------|
| `/api/post/a` | `{ "slug": ["a"] }` |
| `/api/post/a/b` | `{ "slug": ["a", "b"] }` |
| `/api/post/a/b/c` | `{ "slug": ["a", "b", "c"] }` |

### Optional Catch All 라우트

이중 대괄호(`[[...slug]]`)를 사용하면 매개변수가 선택사항이 됩니다:

```ts filename="pages/api/post/[[...slug]].ts"
```

| URL | `req.query` |
|-----|-------------|
| `/api/post` | `{ }` |
| `/api/post/a` | `{ "slug": ["a"] }` |
| `/api/post/a/b` | `{ "slug": ["a", "b"] }` |

## 라우트 우선순위

라우트는 다음 순서로 매칭됩니다:

1. **고정 라우트** (predefined) - `/api/post/create`
2. **동적 라우트** - `/api/post/[pid]`
3. **Catch All 라우트** - `/api/post/[...slug]`

예를 들어, `pages/api/post/create.js`와 `pages/api/post/[pid].js`가 모두 존재하면 `/api/post/create`는 `create.js`에 의해 처리됩니다.

## 스트리밍 응답

Server-Sent Events(SSE) 등의 스트리밍 응답을 구현할 수 있습니다:

```ts filename="pages/api/stream.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-store',
    'Connection': 'keep-alive',
  })

  let i = 0
  while (i < 10) {
    res.write(`data: ${i}\n\n`)
    i++
    await new Promise((resolve) => setTimeout(resolve, 1000))
  }

  res.end()
}
```

## CORS 설정

API Routes는 기본적으로 same-origin만 허용합니다. CORS를 구현하려면:

```ts filename="pages/api/cors.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  // CORS 헤더 설정
  res.setHeader('Access-Control-Allow-Origin', '*')
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS')
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization')

  // OPTIONS 요청 처리 (preflight)
  if (req.method === 'OPTIONS') {
    res.status(200).end()
    return
  }

  // 실제 요청 처리
  res.status(200).json({ message: 'Hello World!' })
}
```

## TypeScript 타입 안정성

```ts filename="pages/api/user.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

type User = {
  id: number
  name: string
  email: string
}

type ErrorResponse = {
  error: string
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<User | ErrorResponse>
) {
  if (req.method === 'GET') {
    res.status(200).json({ id: 1, name: 'John', email: 'john@example.com' })
  } else {
    res.status(405).json({ error: 'Method not allowed' })
  }
}
```

> **참고**: `NextApiRequest.body`는 `any` 타입입니다. 런타임에 유효성 검사를 수행해야 합니다.

## 예제

### JSON 응답

```ts filename="pages/api/json.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  res.status(200).json({ name: 'John Doe', age: 30 })
}
```

### 리다이렉트

```ts filename="pages/api/redirect.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  res.redirect(307, '/new-page')
}
```

### 쿠키 설정

```ts filename="pages/api/set-cookie.ts"
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  res.setHeader('Set-Cookie', 'token=abc123; Path=/; HttpOnly')
  res.status(200).json({ message: '쿠키가 설정되었습니다' })
}
```

## 주의사항

- API Routes는 정적 내보내기(`output: 'export'`)와 함께 사용할 수 없습니다.
- `next.config.js`의 `pageExtensions` 설정이 API Routes에도 적용됩니다.
- App Router의 Route Handlers는 정적 내보내기를 지원합니다.
