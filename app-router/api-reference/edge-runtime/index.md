# Edge Runtime

Edge Runtime은 Next.js에서 Middleware와 Edge API Routes에 사용되는 경량 런타임입니다. 표준 Web API를 기반으로 하며, 낮은 지연 시간으로 전 세계 엣지에서 코드를 실행할 수 있습니다.

---

## 개요

Edge Runtime은 다음 특징을 가집니다:

- **경량**: Node.js보다 훨씬 작은 런타임
- **표준 API**: Web 표준 API 기반
- **빠른 시작**: 콜드 스타트 시간 최소화
- **글로벌 배포**: 전 세계 엣지 로케이션에서 실행

---

## 지원되는 API

### Network APIs

| API | 설명 |
|-----|------|
| `fetch` | 네트워크 요청 수행 |
| `Request` | 요청 객체 |
| `Response` | 응답 객체 |
| `Headers` | HTTP 헤더 조작 |
| `URL` | URL 파싱 및 조작 |
| `URLSearchParams` | URL 쿼리 파라미터 조작 |
| `Blob` | 바이너리 데이터 |
| `File` | 파일 객체 |
| `FormData` | 폼 데이터 조작 |
| `WebSocket` | WebSocket 연결 |

### Encoding APIs

| API | 설명 |
|-----|------|
| `TextEncoder` | 문자열을 바이트로 인코딩 |
| `TextDecoder` | 바이트를 문자열로 디코딩 |
| `TextEncoderStream` | 스트리밍 텍스트 인코딩 |
| `TextDecoderStream` | 스트리밍 텍스트 디코딩 |
| `atob` | Base64 디코딩 |
| `btoa` | Base64 인코딩 |

### Stream APIs

| API | 설명 |
|-----|------|
| `ReadableStream` | 읽기 가능한 스트림 |
| `ReadableStreamDefaultReader` | 기본 스트림 리더 |
| `ReadableStreamBYOBReader` | BYOB 스트림 리더 |
| `WritableStream` | 쓰기 가능한 스트림 |
| `WritableStreamDefaultWriter` | 기본 스트림 라이터 |
| `TransformStream` | 변환 스트림 |

### Crypto APIs

| API | 설명 |
|-----|------|
| `crypto` | 암호화 기능 |
| `CryptoKey` | 암호화 키 |
| `SubtleCrypto` | 저수준 암호화 작업 |

### Web Standard APIs

**타입:**
- `Array`, `ArrayBuffer`, `Boolean`, `DataView`, `Date`
- `Number`, `String`, `BigInt`, `Object`, `Function`

**컬렉션:**
- `Map`, `Set`, `WeakMap`, `WeakSet`

**타입 배열:**
- `Uint8Array`, `Uint16Array`, `Uint32Array`
- `Int8Array`, `Int16Array`, `Int32Array`
- `Float32Array`, `Float64Array`, `BigInt64Array`, `BigUint64Array`

**에러:**
- `Error`, `TypeError`, `RangeError`, `SyntaxError`
- `ReferenceError`, `URIError`, `EvalError`

**제어 흐름:**
- `Promise`, `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`
- `queueMicrotask`, `AbortController`, `AbortSignal`

**유틸리티:**
- `JSON`, `Math`, `Intl`, `RegExp`
- `Proxy`, `Reflect`, `Symbol`, `Iterator`

### Next.js 특정 API

| API | 설명 |
|-----|------|
| `AsyncLocalStorage` | 비동기 컨텍스트 저장소 |
| `process.env` | 환경 변수 접근 |

---

## 미지원 API

Edge Runtime에서는 다음 API를 사용할 수 없습니다:

| API | 이유 |
|-----|------|
| Native Node.js APIs | 파일 시스템, child_process 등 |
| `require()` | CommonJS 모듈 시스템 |
| `eval()` | 보안상의 이유 |
| `new Function()` | 동적 코드 실행 |
| `WebAssembly.compile` | 동적 코드 컴파일 |
| `WebAssembly.instantiate` | 동적 인스턴스화 |

---

## 사용 방법

### Middleware에서 사용

```ts
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Edge Runtime에서 실행됨
  const response = NextResponse.next()
  response.headers.set('x-custom-header', 'my-value')
  return response
}

export const config = {
  matcher: '/api/:path*',
}
```

### Route Handler에서 사용

```ts
// app/api/hello/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  return new Response('Hello from Edge!')
}
```

### 페이지에서 사용

```tsx
// app/page.tsx
export const runtime = 'edge'

export default function Page() {
  return <h1>Edge Rendered Page</h1>
}
```

---

## 환경 변수

Edge Runtime에서는 `process.env`를 통해 환경 변수에 접근할 수 있습니다.

```ts
export const runtime = 'edge'

export async function GET() {
  const apiKey = process.env.API_KEY
  return new Response(`API Key: ${apiKey}`)
}
```

> **주의**: 빌드 시점에 환경 변수가 인라인됩니다.

---

## 동적 코드 평가

동적 코드 평가가 필요한 라이브러리를 사용해야 하는 경우, `next.config.js`에서 `unstable_allowDynamic` 설정을 사용할 수 있습니다.

```js
// next.config.js
module.exports = {
  experimental: {
    serverActions: {
      allowedOrigins: ['my-proxy.com'],
    },
  },
}
```

> **경고**: 이 설정은 보안 위험이 있으므로 신중하게 사용하세요.

---

## Edge vs Node.js Runtime 비교

| 특성 | Edge Runtime | Node.js Runtime |
|------|-------------|-----------------|
| 콜드 스타트 | 빠름 | 느림 |
| 스트리밍 | ✅ | ✅ |
| 정적 렌더링 | ✅ | ✅ |
| 동적 렌더링 | ✅ | ✅ |
| 데이터 재검증 | ✅ | ✅ |
| Node.js API | ❌ | ✅ |
| 파일 시스템 | ❌ | ✅ |
| 번들 크기 | 제한 있음 | 제한 없음 |
| 실행 시간 | 제한 있음 (호스팅 제공자에 따라 다름) | 제한 없음 |

---

## 모범 사례

### 작은 번들 크기 유지

Edge Runtime은 번들 크기 제한이 있습니다. 다음을 권장합니다:

```ts
// 좋음: 필요한 것만 가져오기
import { parse } from 'some-library/parse'

// 피해야 함: 전체 라이브러리 가져오기
import * as library from 'some-library'
```

### 캐싱 활용

```ts
export const runtime = 'edge'

export async function GET() {
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }, // 60초마다 재검증
  })

  return Response.json(await data.json())
}
```

### 에러 처리

```ts
export const runtime = 'edge'

export async function GET() {
  try {
    const data = await fetchData()
    return Response.json(data)
  } catch (error) {
    return new Response('Error fetching data', { status: 500 })
  }
}
```

---

## 제한사항

- **실행 시간**: 호스팅 제공자에 따라 실행 시간 제한이 있습니다 (예: Vercel은 30초)
- **메모리**: 메모리 사용량 제한이 있습니다
- **번들 크기**: 코드 번들 크기 제한이 있습니다 (예: 1MB ~ 4MB)
- **API 제한**: Node.js 네이티브 API를 사용할 수 없습니다

---

## 중요한 주의사항

> **Good to know**:
> - Edge Runtime은 Middleware와 Edge API Routes에서 자동으로 사용됩니다
> - `runtime = 'edge'`를 export하여 명시적으로 설정할 수 있습니다
> - Node.js API가 필요한 경우 기본 Node.js Runtime을 사용하세요

> **성능 팁**:
> - 가능한 작은 의존성을 사용하세요
> - 빌드 시점에 가능한 많은 작업을 수행하세요
> - 캐싱을 적극 활용하세요

---

## 관련 문서

- [Middleware](../file-conventions/middleware.md)
- [Route Handlers](../../getting-started/11-route-handlers.md)
- [NextRequest/NextResponse](../functions/next-request-response.md)
