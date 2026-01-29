---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/instrumentation
버전: 16.1.6
---

# instrumentation.js

## 개요
`instrumentation.js|ts` 파일은 관찰성 도구를 Next.js 애플리케이션에 통합하여 성능을 추적하고, 동작을 모니터링하며, 프로덕션 문제를 디버그하는 데 사용됩니다. 애플리케이션의 **루트** 또는 `src` 폴더 내부에 배치해야 합니다.

---

## 핵심 내보내기

### 1. `register` 함수 (선택사항)
새로운 Next.js 서버 인스턴스가 시작될 때 **한 번** 호출됩니다. 비동기일 수 있습니다.

**TypeScript 예제:**
```ts
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel('next-app')
}
```

**JavaScript 예제:**
```js
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel('next-app')
}
```

### 2. `onRequestError` 함수 (선택사항)
**서버 오류**를 추적하고 커스텀 관찰성 제공자에게 전송합니다.

**TypeScript 예제:**
```ts
import { type Instrumentation } from 'next'

export const onRequestError: Instrumentation.onRequestError = async (
  err,
  request,
  context
) => {
  await fetch('https://.../report-error', {
    method: 'POST',
    body: JSON.stringify({
      message: err.message,
      request,
      context,
    }),
    headers: {
      'Content-Type': 'application/json',
    },
  })
}
```

**JavaScript 예제:**
```js
export async function onRequestError(err, request, context) {
  await fetch('https://.../report-error', {
    method: 'POST',
    body: JSON.stringify({
      message: err.message,
      request,
      context,
    }),
    headers: {
      'Content-Type': 'application/json',
    },
  })
}
```

#### 매개변수

```ts
export function onRequestError(
  error: { digest: string } & Error,
  request: {
    path: string                    // 리소스 경로, 예: /blog?name=foo
    method: string                  // GET, POST 등
    headers: { [key: string]: string | string[] }
  },
  context: {
    routerKind: 'Pages Router' | 'App Router'
    routePath: string               // 라우트 파일 경로, 예: /app/blog/[dynamic]
    routeType: 'render' | 'route' | 'action' | 'proxy'
    renderSource: 'react-server-components'
               | 'react-server-components-payload'
               | 'server-rendering'
    revalidateReason: 'on-demand' | 'stale' | undefined
    renderType: 'dynamic' | 'dynamic-resume'  // PPR의 경우 'dynamic-resume'
  }
): void | Promise<void>
```

**주요 포인트:**
- `error`: 고유한 `digest` ID가 있는 캐치된 오류
- `request`: 읽기 전용 요청 정보
- `context`: 라우터 유형, 라우트 경로, 오류 컨텍스트 (Server Components, Route Handlers, Server Actions, Proxy)
- `onRequestError`에서 비동기 작업은 항상 await해야 함
- 오류 인스턴스는 React에서 처리될 수 있으므로 `digest` 속성을 사용하여 실제 오류 유형을 식별

---

## 런타임 구성

이 파일은 **Node.js** 및 **Edge** 런타임 모두에서 작동합니다. `process.env.NEXT_RUNTIME`을 사용하여 특정 환경을 타겟팅하세요:

```js
export function register() {
  if (process.env.NEXT_RUNTIME === 'edge') {
    return require('./register.edge')
  } else {
    return require('./register.node')
  }
}

export function onRequestError() {
  if (process.env.NEXT_RUNTIME === 'edge') {
    return require('./on-request-error.edge')
  } else {
    return require('./on-request-error.node')
  }
}
```

---

## 사용 예제

### Sentry 통합

```ts
// instrumentation.ts
import * as Sentry from '@sentry/nextjs'

export function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      tracesSampleRate: 1.0,
    })
  }

  if (process.env.NEXT_RUNTIME === 'edge') {
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      tracesSampleRate: 1.0,
    })
  }
}

export async function onRequestError(err, request, context) {
  Sentry.captureException(err, {
    contexts: {
      nextjs: {
        request,
        routerKind: context.routerKind,
        routePath: context.routePath,
        routeType: context.routeType,
      },
    },
  })
}
```

### OpenTelemetry 설정

```ts
// instrumentation.ts
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel({
    serviceName: 'next-app',
  })
}
```

### 커스텀 로깅 서비스

```ts
// instrumentation.ts
import { type Instrumentation } from 'next'

export const onRequestError: Instrumentation.onRequestError = async (
  error,
  request,
  context
) => {
  // 커스텀 로깅 서비스에 전송
  await fetch('https://your-logging-service.com/api/errors', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.LOGGING_API_KEY}`,
    },
    body: JSON.stringify({
      timestamp: new Date().toISOString(),
      error: {
        message: error.message,
        digest: error.digest,
        stack: error.stack,
      },
      request: {
        path: request.path,
        method: request.method,
        headers: request.headers,
      },
      context: {
        routerKind: context.routerKind,
        routePath: context.routePath,
        routeType: context.routeType,
        renderSource: context.renderSource,
      },
    }),
  })
}
```

### 조건부 오류 보고

```ts
// instrumentation.ts
export async function onRequestError(error, request, context) {
  // 특정 오류만 보고
  if (error.message.includes('Database')) {
    await reportDatabaseError(error, request, context)
  } else if (error.message.includes('API')) {
    await reportAPIError(error, request, context)
  }

  // 프로덕션에서만 보고
  if (process.env.NODE_ENV === 'production') {
    await sendToMonitoringService(error, request, context)
  }
}
```

---

## 설정 활성화

`next.config.js`에서 instrumentation을 활성화해야 합니다:

```js
// next.config.js
module.exports = {
  experimental: {
    instrumentationHook: true,
  },
}
```

**Next.js 15+에서는 기본적으로 활성화되어 있습니다.**

---

## 파일 위치

```
프로젝트 루트/
├── instrumentation.ts    (또는 .js)
├── app/
│   └── ...
└── src/                  (선택사항)
    ├── instrumentation.ts
    └── app/
        └── ...
```

---

## 모범 사례

### 1. 성능 고려

```ts
export async function onRequestError(error, request, context) {
  // 비동기 작업을 차단하지 않도록 주의
  // 오류 보고가 애플리케이션을 느리게 하지 않도록 함

  // ❌ 나쁜 예
  await heavyProcessing()

  // ✅ 좋은 예 - 백그라운드로 실행
  fetch('...').catch(console.error) // await하지 않음
}
```

### 2. 오류 필터링

```ts
export async function onRequestError(error, request, context) {
  // 중요하지 않은 오류 필터링
  const ignoredErrors = ['ECONNRESET', 'EPIPE']
  if (ignoredErrors.some(msg => error.message.includes(msg))) {
    return
  }

  await reportError(error, request, context)
}
```

### 3. 환경별 설정

```ts
export function register() {
  if (process.env.NODE_ENV === 'development') {
    console.log('개발 모드: 상세 로깅 활성화')
    // 개발 환경 설정
  } else {
    // 프로덕션 환경 설정
    initProductionMonitoring()
  }
}
```

### 4. 오류 컨텍스트 활용

```ts
export async function onRequestError(error, request, context) {
  const errorReport = {
    error: {
      message: error.message,
      digest: error.digest,
    },
    // 풍부한 컨텍스트 정보 포함
    router: context.routerKind,
    route: context.routePath,
    type: context.routeType,
    // Server Component, Route Handler, Server Action 구분
    source: context.renderSource,
    // PPR 사용 여부 확인
    isPPR: context.renderType === 'dynamic-resume',
  }

  await logError(errorReport)
}
```

---

## 제한사항

### 파일 위치
- 루트 또는 `src/` 폴더에만 배치 가능
- `app/` 또는 `pages/` 내부에 배치 불가

### 실행 시점
- `register()`는 서버 시작 시 한 번만 실행
- 각 요청마다 실행되지 않음

### Edge 런타임 제한
- Node.js 전용 패키지 사용 불가
- 파일 시스템 접근 제한

---

## 디버깅

### register 함수 실행 확인

```ts
export function register() {
  console.log('✅ Instrumentation registered')
  console.log('Runtime:', process.env.NEXT_RUNTIME)
}
```

### 오류 추적 테스트

```ts
export async function onRequestError(error, request, context) {
  console.log('❌ Error caught:', {
    message: error.message,
    digest: error.digest,
    path: request.path,
    route: context.routePath,
  })
}
```

---

## 버전 히스토리

| 버전   | 변경사항 |
|-----------|---------|
| `v15.0.0` | `onRequestError` 도입, `instrumentation` 안정화 |
| `v14.0.4` | Turbopack 지원 추가 |
| `v13.2.0` | 실험적 기능으로 도입 |

---

## 관련 문서

- [OpenTelemetry](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry)
- [오류 처리](../../getting-started/07-error-handling.md)
- [디버깅](../../guides/debugging.md)
