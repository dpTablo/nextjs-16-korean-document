# Edge와 Node.js 런타임

Next.js에서는 두 가지 서버 런타임을 사용할 수 있습니다:

- **Node.js 런타임** (기본값): 모든 Node.js API와 호환되는 패키지에 접근할 수 있습니다.
- **Edge 런타임**: 제한된 API 세트를 지원하며 더 가벼운 환경을 제공합니다.

## 사용 사례

- **Node.js 런타임**은 복잡한 계산, 파일 시스템 접근, 데이터베이스 연결 등 완전한 Node.js 기능이 필요한 경우에 사용합니다.
- **Edge 런타임**은 낮은 지연 시간으로 간단한 함수를 실행해야 할 때 적합합니다.

## Edge 런타임 제약사항

1. Edge 런타임은 모든 Node.js API를 지원하지 않습니다. 일부 패키지가 작동하지 않을 수 있습니다.
2. ISR(Incremental Static Regeneration)을 지원하지 않습니다.
3. 배포 어댑터에 따라 스트리밍 지원 여부가 달라질 수 있습니다.

## Edge 런타임에서 지원되는 API

### 네트워크 API

| API | 설명 |
|-----|------|
| `Blob` | Blob 표현 |
| `fetch` | 리소스 가져오기 |
| `FetchEvent` | Fetch 이벤트 |
| `File` | 파일 표현 |
| `FormData` | 폼 데이터 |
| `Headers` | HTTP 헤더 |
| `Request` | HTTP 요청 |
| `Response` | HTTP 응답 |
| `URLSearchParams` | URL 검색 파라미터 |
| `WebSocket` | WebSocket 연결 |

### 인코딩 API

| API | 설명 |
|-----|------|
| `atob` | Base64 디코딩 |
| `btoa` | Base64 인코딩 |
| `TextDecoder` | 텍스트 디코딩 |
| `TextDecoderStream` | 스트림 텍스트 디코딩 |
| `TextEncoder` | 텍스트 인코딩 |
| `TextEncoderStream` | 스트림 텍스트 인코딩 |

### 스트림 API

| API | 설명 |
|-----|------|
| `ReadableStream` | 읽기 가능한 스트림 |
| `ReadableStreamBYOBReader` | BYOB 스트림 리더 |
| `ReadableStreamDefaultReader` | 기본 스트림 리더 |
| `TransformStream` | 변환 스트림 |
| `WritableStream` | 쓰기 가능한 스트림 |
| `WritableStreamDefaultWriter` | 기본 스트림 라이터 |

### 암호화 API

| API | 설명 |
|-----|------|
| `crypto` | 암호화 객체 |
| `CryptoKey` | 암호화 키 |
| `SubtleCrypto` | Subtle 암호화 |

### 웹 표준 API

Edge 런타임은 다음을 포함한 많은 웹 표준 API를 지원합니다:

`AbortController`, `AbortSignal`, `Array`, `ArrayBuffer`, `Atomics`, `BigInt`, `BigInt64Array`, `BigUint64Array`, `Boolean`, `clearInterval`, `clearTimeout`, `console`, `DataView`, `Date`, `decodeURI`, `decodeURIComponent`, `DOMException`, `encodeURI`, `encodeURIComponent`, `Error`, `EvalError`, `Float32Array`, `Float64Array`, `Function`, `Infinity`, `Int8Array`, `Int16Array`, `Int32Array`, `Intl`, `isFinite`, `isNaN`, `JSON`, `Map`, `Math`, `Number`, `Object`, `parseFloat`, `parseInt`, `Promise`, `Proxy`, `RangeError`, `ReferenceError`, `Reflect`, `RegExp`, `Set`, `setInterval`, `setTimeout`, `SharedArrayBuffer`, `String`, `Symbol`, `SyntaxError`, `TypeError`, `Uint8Array`, `Uint8ClampedArray`, `Uint16Array`, `Uint32Array`, `URIError`, `URL`, `URLPattern`, `URLSearchParams`, `WeakMap`, `WeakRef`, `WeakSet`, `WebAssembly`

### Next.js 전용 Polyfill

- `AsyncLocalStorage`: Node.js의 `AsyncLocalStorage` API polyfill

### 환경 변수

`process.env`를 통해 환경 변수에 접근할 수 있습니다.

## 지원되지 않는 API

Edge 런타임에는 다음과 같은 제한이 있습니다:

### Node.js 네이티브 API

- 파일 시스템 읽기/쓰기 (`fs`)
- 네이티브 모듈 (`child_process`, `cluster` 등)
- `require()` 직접 호출 - ES Modules를 사용해야 합니다

### 비활성화된 JavaScript 기능

보안상의 이유로 다음 기능은 Edge 런타임에서 비활성화됩니다:

```javascript
// ❌ 지원되지 않음
eval('console.log("hello")')
new Function('console.log("hello")')
WebAssembly.compile()
WebAssembly.instantiate()
```

## 동적 코드 평가 허용

특정 파일에서 동적 코드 평가가 필요한 경우 `unstable_allowDynamic` 설정을 사용할 수 있습니다:

```javascript
// pages/api/example.js
export const config = {
  runtime: 'edge',
  unstable_allowDynamic: [
    // 단일 파일 허용
    '/lib/utilities.js',
    // glob을 사용하여 서드파티 모듈의 모든 파일 허용
    '**/node_modules/function-bind/**',
  ],
}

export default function handler(req) {
  return new Response('Hello, Edge!')
}
```

> **주의:** 이 설정은 해당 패턴의 파일에서 동적 코드 평가가 실행될 때 런타임 에러를 발생시킬 수 있습니다. 주의해서 사용하세요.

## 런타임 설정

페이지나 API 라우트에서 런타임을 지정하려면 `config` 객체를 내보내세요:

```javascript
// pages/api/edge-example.js
export const config = {
  runtime: 'edge',
}

export default function handler(req) {
  return new Response('Hello from Edge!')
}
```

```javascript
// pages/api/node-example.js
export const config = {
  runtime: 'nodejs',
}

export default function handler(req, res) {
  res.status(200).json({ message: 'Hello from Node.js!' })
}
```

## 런타임 비교

| 특성 | Node.js 런타임 | Edge 런타임 |
|------|---------------|------------|
| Cold Start | 일반 | 빠름 |
| HTTP 스트리밍 | 지원 | 지원 |
| I/O | 모두 지원 | `fetch` |
| 확장성 | 일반 | 높음 |
| 보안 | 일반 | 높음 |
| 지연 시간 | 일반 | 낮음 |
| npm 패키지 | 모두 지원 | 제한적 |
| ISR | 지원 | 미지원 |

## 언제 어떤 런타임을 선택할까

### Node.js 런타임 선택

- 파일 시스템 접근이 필요한 경우
- 복잡한 계산이 필요한 경우
- 모든 npm 패키지를 사용해야 하는 경우
- ISR이 필요한 경우

### Edge 런타임 선택

- 낮은 지연 시간이 중요한 경우
- 간단한 함수 실행이 필요한 경우
- 글로벌 분산이 필요한 경우
- 빠른 Cold Start가 필요한 경우
