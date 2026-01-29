# useReportWebVitals

`useReportWebVitals` 훅은 [Core Web Vitals](https://web.dev/vitals/)를 보고하고 분석 서비스와 함께 사용할 수 있습니다.

## 사용법

```jsx filename="pages/_app.js"
import { useReportWebVitals } from 'next/web-vitals'

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    console.log(metric)
  })

  return <Component {...pageProps} />
}
```

## Metric 객체

콜백 함수에 전달되는 `metric` 객체는 다음 속성들로 구성됩니다:

| 속성 | 타입 | 설명 |
|------|------|------|
| `id` | `string` | 현재 페이지 로드 컨텍스트에서 메트릭의 고유 식별자 |
| `name` | `string` | 성능 메트릭의 이름 |
| `delta` | `number` | 현재 값과 이전 값의 차이 (밀리초 단위) |
| `entries` | `PerformanceEntry[]` | 메트릭과 관련된 Performance Entries 배열 |
| `navigationType` | `string` | 네비게이션 타입 |
| `rating` | `string` | 성능 평가 (`'good'`, `'needs-improvement'`, `'poor'`) |
| `value` | `number` | 메트릭의 실제 값 (밀리초 단위) |

## Web Vitals 메트릭

- **[TTFB](https://web.dev/ttfb/)** (Time to First Byte): 서버가 첫 바이트를 응답하는 시간
- **[FCP](https://web.dev/fcp/)** (First Contentful Paint): 첫 콘텐츠가 화면에 렌더링되는 시간
- **[LCP](https://web.dev/lcp/)** (Largest Contentful Paint): 가장 큰 콘텐츠가 화면에 렌더링되는 시간
- **[FID](https://web.dev/fid/)** (First Input Delay): 사용자의 첫 상호작용과 브라우저 응답 사이의 시간
- **[CLS](https://web.dev/cls/)** (Cumulative Layout Shift): 예상치 못한 레이아웃 이동의 누적 점수
- **[INP](https://web.dev/inp/)** (Interaction to Next Paint): 상호작용부터 다음 페인트까지의 시간

다음 예제는 이러한 메트릭들을 개별적으로 처리하는 방법을 보여줍니다:

```jsx filename="pages/_app.js"
import { useReportWebVitals } from 'next/web-vitals'

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'FCP': {
        // FCP 결과 처리
        break
      }
      case 'LCP': {
        // LCP 결과 처리
        break
      }
      // ...
    }
  })

  return <Component {...pageProps} />
}
```

## Next.js 커스텀 메트릭

위의 표준 메트릭 외에도, Next.js 전용 커스텀 메트릭이 있습니다:

- **Next.js-hydration**: 페이지가 hydration을 시작하고 완료하는 데 걸리는 시간 (밀리초)
- **Next.js-route-change-to-render**: 라우트 변경 후 페이지가 렌더링을 시작하는 데 걸리는 시간 (밀리초)
- **Next.js-render**: 라우트 변경 후 페이지가 렌더링을 완료하는 데 걸리는 시간 (밀리초)

다음과 같이 이러한 메트릭의 결과를 개별적으로 처리할 수 있습니다:

```jsx filename="pages/_app.js"
import { useReportWebVitals } from 'next/web-vitals'

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'Next.js-hydration':
        // hydration 결과 처리
        break
      case 'Next.js-route-change-to-render':
        // 라우트 변경 결과 처리
        break
      case 'Next.js-render':
        // 렌더링 결과 처리
        break
      default:
        break
    }
  })

  return <Component {...pageProps} />
}
```

## 외부 시스템으로 결과 전송

결과를 엔드포인트로 전송하여 사이트의 실제 사용자 성능을 측정하고 추적할 수 있습니다. 예를 들어:

```js
useReportWebVitals((metric) => {
  const body = JSON.stringify(metric)
  const url = 'https://example.com/analytics'

  // `navigator.sendBeacon()`이 사용 가능하면 사용하고, 그렇지 않으면 `fetch()`를 사용합니다.
  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, body)
  } else {
    fetch(url, { body, method: 'POST', keepalive: true })
  }
})
```

> **참고**: [Google Analytics](https://analytics.google.com/analytics/web/)를 사용하는 경우, `id` 값을 사용하면 메트릭 분포를 수동으로 구성할 수 있습니다(백분위수 계산 등).
>
> ```js
> useReportWebVitals(metric => {
>   // Google Analytics를 다음과 같이 초기화한 경우 `window.gtag`를 사용하세요:
>   // https://github.com/vercel/next.js/blob/canary/examples/with-google-analytics
>   window.gtag('event', metric.name, {
>     value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value), // 값은 정수여야 합니다
>     event_label: metric.id, // 현재 페이지 로드에 고유한 id
>     non_interaction: true, // 이탈률에 영향을 주지 않습니다.
>   });
> }
> ```
>
> [Google Analytics로 결과 전송](https://github.com/GoogleChrome/web-vitals#send-the-results-to-google-analytics)에 대해 자세히 알아보세요.

## 버전 정보

| 버전 | 변경사항 |
|------|---------|
| `v16.0.0` | `next/web-vitals` 사용 권장 |
| `v9.4.0` | `useReportWebVitals` 도입 |
