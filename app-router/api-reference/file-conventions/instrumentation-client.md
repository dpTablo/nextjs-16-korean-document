# instrumentation-client.js

`instrumentation-client.js|ts` 파일은 애플리케이션이 인터랙티브해지기 전에 실행되는 모니터링, 분석, 에러 추적 및 기타 부수 효과를 활성화합니다. 이 파일을 애플리케이션의 **루트** 또는 `src` 폴더 내에 배치합니다.

## 기본 사용법

특정 함수를 내보내지 않고 모니터링 코드를 파일에 직접 작성합니다:

```ts
// instrumentation-client.ts
performance.mark('app-init')
console.log('분석 초기화됨')

window.addEventListener('error', (event) => {
  reportError(event.error)
})
```

## 라우터 네비게이션 추적

네비게이션 알림을 받으려면 `onRouterTransitionStart`를 내보냅니다:

```ts
// instrumentation-client.ts
export function onRouterTransitionStart(
  url: string,
  navigationType: 'push' | 'replace' | 'traverse'
) {
  console.log(`네비게이션 시작: ${navigationType} → ${url}`)
  performance.mark(`nav-start-${Date.now()}`)
}
```

## 실행 타이밍

1. HTML 문서가 로드된 **이후**
2. React 하이드레이션이 시작되기 **전**
3. 사용자 상호작용이 가능해지기 **전**

## 성능 고려사항

- 인스트루멘테이션 코드를 가볍게 유지하세요
- 개발 환경에서 초기화가 16ms를 초과하면 Next.js가 경고를 표시합니다

## 전체 예제

### 에러 추적

```ts
// instrumentation-client.ts
import Monitor from './lib/monitoring'

Monitor.initialize()

export function onRouterTransitionStart(url: string) {
  Monitor.pushEvent({
    message: `${url}로 네비게이션`,
    category: 'navigation',
  })
}
```

### 분석 추적

```ts
// instrumentation-client.ts
import { analytics } from './lib/analytics'

analytics.init()

export function onRouterTransitionStart(url: string, navigationType: string) {
  analytics.track('page_navigation', {
    url,
    type: navigationType,
    timestamp: Date.now(),
  })
}
```

### 성능 모니터링

```ts
// instrumentation-client.ts
const startTime = performance.now()

const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry instanceof PerformanceNavigationTiming) {
      console.log('Time to Interactive:', entry.loadEventEnd - startTime)
    }
  }
})

observer.observe({ entryTypes: ['navigation'] })

export function onRouterTransitionStart(url: string) {
  performance.mark(`nav-start-${url}`)
}
```

### 폴리필

```ts
// instrumentation-client.ts
import './lib/polyfills'

if (!window.ResizeObserver) {
  import('./lib/polyfills/resize-observer').then((mod) => {
    window.ResizeObserver = mod.default
  })
}
```

## 중요 사항

- 강력한 에러 처리를 위해 try-catch 블록을 구현하세요
- 개별 추적 실패가 다른 기능에 영향을 미치지 않도록 합니다
- 이 파일은 클라이언트 사이드에서만 실행됩니다

## `instrumentation.js`와의 차이점

| 파일 | 실행 환경 | 용도 |
|------|----------|------|
| `instrumentation.js` | 서버 사이드 | 서버 모니터링, ORM 설정 등 |
| `instrumentation-client.js` | 클라이언트 사이드 | 프론트엔드 분석, 에러 추적 등 |

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| v15.3 | `instrumentation-client.js` 도입 |

---

## 참고

- [instrumentation.js](/app-router/api-reference/file-conventions/instrumentation.md)
- [OpenTelemetry 가이드](/app-router/guides/opentelemetry.md)
- [분석 가이드](/app-router/guides/analytics.md)
