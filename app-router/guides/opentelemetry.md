# OpenTelemetry

Next.js 애플리케이션에서 OpenTelemetry를 사용하여 관찰성(Observability)을 구현하는 방법을 알아봅니다.

## 개요

OpenTelemetry는 Next.js 애플리케이션에서 관찰성을 활성화하는 플랫폼 독립적인 계측 프레임워크입니다. 코드를 수정하지 않고도 관찰성 제공업체를 변경할 수 있습니다.

### 주요 이점

- ✅ **능동적 문제 식별** - 문제가 사용자에게 영향을 미치기 전에 감지
- ✅ **성능 최적화** - 병목 지점 파악 및 최적화 인사이트
- ✅ **향상된 사용자 경험** - 더 나은 리소스 관리
- ✅ **내장 지원** - Next.js는 기본적으로 OpenTelemetry를 지원

---

## 시작하기

### `@vercel/otel`을 사용한 빠른 설정

가장 간단한 방법으로 OpenTelemetry를 시작할 수 있습니다.

**1. 패키지 설치:**
```bash
npm install @vercel/otel @opentelemetry/sdk-logs @opentelemetry/api-logs @opentelemetry/instrumentation
```

**2. 프로젝트 루트에 `instrumentation.ts` 생성:**

```ts
// instrumentation.ts (프로젝트 루트 또는 src 폴더)
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel({ serviceName: 'next-app' })
}
```

> **중요:**
> - 계측 파일은 루트 디렉토리에 배치하세요 (`app`이나 `pages` 내부가 아님)
> - `src` 폴더를 사용하는 경우 `src` 내부에 배치
> - `pageExtensions` 설정 옵션을 사용하는 경우 파일 이름을 일치하도록 업데이트

**3. Next.js 설정 업데이트:**

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    instrumentationHook: true,
  },
}

module.exports = nextConfig
```

**4. 환경 변수 설정 (선택사항):**

```bash
# .env.local
NEXT_OTEL_VERBOSE=1 # 상세 로깅 활성화
```

---

### 수동 OpenTelemetry 구성

고급 사용 사례를 위한 커스텀 구성:

**1. 패키지 설치:**
```bash
npm install @opentelemetry/sdk-node @opentelemetry/resources @opentelemetry/semantic-conventions @opentelemetry/sdk-trace-node @opentelemetry/exporter-trace-otlp-http
```

**2. 런타임 조건 확인 생성:**

**instrumentation.ts**
```ts
export async function register() {
  // Edge Runtime은 NodeSDK를 지원하지 않으므로 Node.js에서만 실행
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation.node.ts')
  }
}
```

**3. NodeSDK 구성:**

**instrumentation.node.ts**
```ts
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'
import { Resource } from '@opentelemetry/resources'
import { NodeSDK } from '@opentelemetry/sdk-node'
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-node'
import { ATTR_SERVICE_NAME } from '@opentelemetry/semantic-conventions'

const sdk = new NodeSDK({
  resource: new Resource({
    [ATTR_SERVICE_NAME]: 'next-app',
  }),
  spanProcessor: new SimpleSpanProcessor(new OTLPTraceExporter()),
})

sdk.start()
```

> **참고:**
> - NodeSDK는 Edge Runtime과 호환되지 않습니다
> - Edge Runtime 지원이 필요한 경우 `@vercel/otel` 사용

---

## 커스텀 Span 생성

애플리케이션의 특정 작업을 추적하기 위한 커스텀 span을 만들 수 있습니다.

**1. OpenTelemetry API 패키지 설치:**
```bash
npm install @opentelemetry/api
```

**2. 커스텀 span 생성:**

```ts
import { trace } from '@opentelemetry/api'

export async function fetchGithubStars() {
  return await trace
    .getTracer('nextjs-example')
    .startActiveSpan('fetchGithubStars', async (span) => {
      try {
        const res = await fetch('https://api.github.com/repos/vercel/next.js')
        const data = await res.json()

        // Span에 속성 추가
        span.setAttribute('github.stars', data.stargazers_count)

        return data.stargazers_count
      } finally {
        span.end()
      }
    })
}
```

### 고급 Span 사용

```ts
import { trace, SpanStatusCode } from '@opentelemetry/api'

export async function processOrder(orderId: string) {
  const tracer = trace.getTracer('order-service')

  return await tracer.startActiveSpan('processOrder', async (span) => {
    try {
      // Span 속성 설정
      span.setAttribute('order.id', orderId)
      span.setAttribute('order.status', 'processing')

      // 중첩된 span 생성
      const order = await tracer.startActiveSpan('fetchOrder', async (childSpan) => {
        try {
          const result = await getOrder(orderId)
          childSpan.setAttribute('order.amount', result.amount)
          return result
        } finally {
          childSpan.end()
        }
      })

      // 이벤트 추가
      span.addEvent('order.validated', {
        'validation.result': 'success',
      })

      // 성공 상태 설정
      span.setStatus({ code: SpanStatusCode.OK })

      return order
    } catch (error) {
      // 에러 상태 설정
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error instanceof Error ? error.message : 'Unknown error',
      })

      // 에러 이벤트 기록
      span.recordException(error as Error)

      throw error
    } finally {
      span.end()
    }
  })
}
```

---

## Next.js의 기본 Span

Next.js는 여러 작업을 자동으로 계측합니다. 모든 span 속성은 [OpenTelemetry 의미론적 규칙](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/)을 따릅니다.

### 커스텀 Next.js 속성

| 속성 | 설명 | 예시 |
|------|------|------|
| `next.span_name` | Span 이름 복제 | `GET /api/user` |
| `next.span_type` | 고유한 span 타입 식별자 | `BaseServer.handleRequest` |
| `next.route` | 라우트 패턴 | `/[param]/user` |
| `next.rsc` | RSC 요청 여부 | `true` / `false` |
| `next.page` | 특수 파일로의 내부 라우트 | `page.ts`, `layout.ts` |

### 주요 자동 Span

| Span | 타입 | 목적 |
|------|------|------|
| `[http.method] [next.route]` | `BaseServer.handleRequest` | 들어오는 요청의 루트 span |
| `render route (app) [next.route]` | `AppRender.getBodyResult` | App Router 렌더링 프로세스 |
| `fetch [http.method] [http.url]` | `AppRender.fetch` | Fetch 요청 추적 |
| `executing api route (app) [next.route]` | `AppRouteRouteHandlers.runHandler` | API 라우트 핸들러 실행 |
| `getServerSideProps [next.route]` | `Render.getServerSideProps` | 서버 사이드 props 실행 |
| `getStaticProps [next.route]` | `Render.getStaticProps` | 정적 props 실행 |
| `generateMetadata [next.page]` | `ResolveMetadata.generateMetadata` | 메타데이터 생성 |

### Fetch Span 비활성화

커스텀 fetch 계측을 사용하려면 fetch span을 비활성화할 수 있습니다:

```bash
# .env.local
NEXT_OTEL_FETCH_DISABLED=1
```

---

## 계측 테스트

### 로컬 OpenTelemetry Collector 설정

**1. OpenTelemetry Collector 개발 환경 사용:**

```bash
# OpenTelemetry Collector 개발 설정 클론
git clone https://github.com/vercel/opentelemetry-collector-dev-setup.git
cd opentelemetry-collector-dev-setup

# Docker Compose로 실행
docker-compose up
```

**2. 애플리케이션 실행:**

```bash
npm run dev
```

**3. Trace 확인:**

브라우저에서 `http://localhost:16686` (Jaeger UI)에 접속하여 trace를 확인하세요.

### Trace 구조

- **루트 span**: `GET /requested/pathname`으로 표시
- **자식 span**: 루트 span 아래 중첩됨
- **상세 출력**: `NEXT_OTEL_VERBOSE=1` 환경 변수 설정

---

## 배포

### Vercel 배포

Vercel에서는 OpenTelemetry가 기본적으로 작동합니다:

1. **관찰성 제공업체 연결:**
   - [Vercel 문서](https://vercel.com/docs/observability/otel-overview/quickstart) 참조
   - Datadog, New Relic, Honeycomb 등 지원

2. **자동 구성:**
   - Vercel은 자동으로 OpenTelemetry를 구성합니다
   - 추가 설정 없이 trace 수집 시작

### 셀프 호스팅

**1. OpenTelemetry Collector 설정:**

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  logging:
    loglevel: debug
  # 다른 exporter 추가 (Jaeger, Zipkin 등)

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
```

**2. Collector 실행:**

```bash
# Docker로 실행
docker run -p 4318:4318 \
  -v $(pwd)/otel-collector-config.yaml:/etc/otel-collector-config.yaml \
  otel/opentelemetry-collector:latest \
  --config=/etc/otel-collector-config.yaml
```

**3. Next.js 앱 구성:**

```ts
// instrumentation.node.ts
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'

const exporter = new OTLPTraceExporter({
  url: 'http://localhost:4318/v1/traces',
})

// ... SDK 구성
```

### 커스텀 Exporter

OpenTelemetry Collector 없이 직접 관찰성 플랫폼으로 전송:

**Datadog 예시:**

```bash
npm install @opentelemetry/exporter-trace-otlp-http
```

```ts
// instrumentation.node.ts
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'

const exporter = new OTLPTraceExporter({
  url: 'https://trace.agent.datadoghq.com/v0.4/traces',
  headers: {
    'DD-API-KEY': process.env.DATADOG_API_KEY!,
  },
})
```

**New Relic 예시:**

```ts
const exporter = new OTLPTraceExporter({
  url: 'https://otlp.nr-data.net/v1/traces',
  headers: {
    'api-key': process.env.NEW_RELIC_LICENSE_KEY!,
  },
})
```

---

## 실전 예시

### 완전한 모니터링 설정

**instrumentation.ts**
```ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation.node')
  }

  // Edge Runtime 또는 공통 초기화
  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./instrumentation.edge')
  }
}
```

**instrumentation.node.ts**
```ts
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'
import { Resource } from '@opentelemetry/resources'
import { NodeSDK } from '@opentelemetry/sdk-node'
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-node'
import {
  ATTR_SERVICE_NAME,
  ATTR_SERVICE_VERSION,
} from '@opentelemetry/semantic-conventions'

const sdk = new NodeSDK({
  resource: new Resource({
    [ATTR_SERVICE_NAME]: 'next-app',
    [ATTR_SERVICE_VERSION]: process.env.npm_package_version || '1.0.0',
    environment: process.env.NODE_ENV,
  }),
  spanProcessor: new SimpleSpanProcessor(
    new OTLPTraceExporter({
      url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4318/v1/traces',
    })
  ),
})

sdk.start()

// Graceful shutdown
process.on('SIGTERM', () => {
  sdk
    .shutdown()
    .then(() => console.log('OpenTelemetry terminated'))
    .catch((error) => console.error('Error terminating OpenTelemetry', error))
    .finally(() => process.exit(0))
})
```

### 커스텀 미들웨어 계측

```ts
// middleware.ts
import { trace, SpanStatusCode } from '@opentelemetry/api'
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const tracer = trace.getTracer('middleware')

  return tracer.startActiveSpan('middleware', (span) => {
    try {
      span.setAttribute('http.method', request.method)
      span.setAttribute('http.url', request.url)

      // 미들웨어 로직
      const response = NextResponse.next()

      span.setAttribute('http.status_code', response.status)
      span.setStatus({ code: SpanStatusCode.OK })

      return response
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error instanceof Error ? error.message : 'Unknown error',
      })
      span.recordException(error as Error)
      throw error
    } finally {
      span.end()
    }
  })
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **의미 있는 span 이름 사용** - 작업을 명확히 설명
2. **적절한 속성 추가** - 디버깅에 유용한 컨텍스트
3. **에러 기록** - `recordException()`으로 예외 추적
4. **span 종료 보장** - `finally` 블록 사용
5. **샘플링 구성** - 프로덕션에서 과도한 trace 방지

### ❌ 피해야 할 것

1. **과도한 span 생성** - 성능 오버헤드
2. **민감 정보 기록** - 비밀번호, 토큰 등 제외
3. **span 종료 누락** - 메모리 누수 발생
4. **동기 작업 블로킹** - 비동기 작업 사용

---

## 환경 변수

| 변수 | 설명 | 기본값 |
|------|------|--------|
| `NEXT_OTEL_VERBOSE` | 상세 로깅 활성화 | `0` |
| `NEXT_OTEL_FETCH_DISABLED` | Fetch span 비활성화 | `0` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OTLP exporter 엔드포인트 | - |
| `OTEL_SERVICE_NAME` | 서비스 이름 | - |

---

## 문제 해결

### Trace가 표시되지 않음

**1. 계측 파일 위치 확인:**
```
프로젝트-루트/
├── instrumentation.ts ✅ (올바름)
├── app/
│   └── instrumentation.ts ❌ (잘못됨)
```

**2. 실험적 기능 활성화 확인:**
```js
// next.config.js
module.exports = {
  experimental: {
    instrumentationHook: true,
  },
}
```

**3. Collector 연결 확인:**
```bash
# Collector가 실행 중인지 확인
curl http://localhost:4318/v1/traces
```

---

## 다음 단계

- [Instrumentation](../api-reference/file-conventions/instrumentation.md) - Instrumentation API
- [Analytics](./analytics.md) - 분석 가이드
- [Monitoring Best Practices](https://opentelemetry.io/docs/concepts/observability-primer/)

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11

**참고 자료:**
- [OpenTelemetry 공식 문서](https://opentelemetry.io/docs/)
- [`@vercel/otel` npm 패키지](https://www.npmjs.com/package/@vercel/otel)
- [Next.js OpenTelemetry 예시](https://github.com/vercel/next.js/tree/canary/examples/with-opentelemetry)
