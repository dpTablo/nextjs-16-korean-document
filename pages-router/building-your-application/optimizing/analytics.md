# 분석 (Analytics)

Next.js는 성능 메트릭 측정 및 보고를 위한 내장 지원을 제공합니다. `useReportWebVitals` 훅을 사용하여 직접 구축하거나, Vercel의 관리 서비스를 이용할 수 있습니다.

## 클라이언트 계측

애플리케이션 루트 디렉토리에 `instrumentation-client.js` 또는 `instrumentation-client.ts` 파일을 생성하여 분석을 초기화할 수 있습니다:

```js
// instrumentation-client.js

// 앱 시작 전 분석 초기화
console.log('분석이 초기화되었습니다')

// 전역 에러 추적 설정
window.addEventListener('error', (event) => {
  reportError(event.error)
})
```

## 직접 구축하기

`useReportWebVitals` 훅을 사용하여 Web Vitals 메트릭을 수집할 수 있습니다:

```jsx
// pages/_app.js
import { useReportWebVitals } from 'next/web-vitals'

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    console.log(metric)
  })

  return <Component {...pageProps} />
}

export default MyApp
```

## Web Vitals 메트릭

다음 6가지 핵심 메트릭이 수집됩니다:

| 메트릭 | 설명 | 좋음 | 개선 필요 |
|--------|------|------|----------|
| **TTFB** | Time to First Byte - 첫 바이트 수신까지의 시간 | < 800ms | > 1800ms |
| **FCP** | First Contentful Paint - 첫 콘텐츠 렌더링 시간 | < 1.8s | > 3s |
| **LCP** | Largest Contentful Paint - 가장 큰 콘텐츠 렌더링 시간 | < 2.5s | > 4s |
| **FID** | First Input Delay - 첫 입력 지연 시간 | < 100ms | > 300ms |
| **CLS** | Cumulative Layout Shift - 누적 레이아웃 이동 | < 0.1 | > 0.25 |
| **INP** | Interaction to Next Paint - 상호작용 후 다음 페인트까지 시간 | < 200ms | > 500ms |

## 메트릭별 처리

각 메트릭을 개별적으로 처리할 수 있습니다:

```jsx
useReportWebVitals((metric) => {
  switch (metric.name) {
    case 'FCP':
      console.log('FCP:', metric.value)
      break
    case 'LCP':
      console.log('LCP:', metric.value)
      break
    case 'CLS':
      console.log('CLS:', metric.value)
      break
    case 'FID':
      console.log('FID:', metric.value)
      break
    case 'TTFB':
      console.log('TTFB:', metric.value)
      break
    case 'INP':
      console.log('INP:', metric.value)
      break
  }
})
```

## 커스텀 메트릭

Next.js는 추가적인 커스텀 메트릭도 제공합니다:

| 메트릭 | 설명 |
|--------|------|
| `Next.js-hydration` | 페이지가 hydration을 시작하고 완료하는 데 걸린 시간 (ms) |
| `Next.js-route-change-to-render` | 라우트 변경 후 페이지가 렌더링을 시작하는 데 걸린 시간 |
| `Next.js-render` | 라우트 변경 후 페이지가 렌더링을 완료하는 데 걸린 시간 |

```jsx
useReportWebVitals((metric) => {
  if (metric.name.startsWith('Next.js-')) {
    console.log('Next.js 커스텀 메트릭:', metric.name, metric.value)
  }
})
```

## 외부 시스템으로 결과 전송

분석 서버로 메트릭을 전송할 수 있습니다:

```jsx
useReportWebVitals((metric) => {
  const body = JSON.stringify(metric)
  const url = 'https://example.com/analytics'

  // navigator.sendBeacon을 사용하면 페이지 언로드 시에도 전송 보장
  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, body)
  } else {
    fetch(url, { body, method: 'POST', keepalive: true })
  }
})
```

## Google Analytics 연동

Google Analytics와 연동하여 메트릭을 전송할 수 있습니다:

```jsx
// pages/_app.js
import { useReportWebVitals } from 'next/web-vitals'

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    // Google Analytics에 메트릭 전송
    window.gtag('event', metric.name, {
      value: Math.round(
        metric.name === 'CLS' ? metric.value * 1000 : metric.value
      ),
      event_label: metric.id,
      non_interaction: true,
    })
  })

  return <Component {...pageProps} />
}

export default MyApp
```

### Google Analytics 4 설정

```jsx
// pages/_app.js
import Script from 'next/script'
import { useReportWebVitals } from 'next/web-vitals'

const GA_MEASUREMENT_ID = 'G-XXXXXXXXXX'

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    window.gtag('event', metric.name, {
      value: Math.round(
        metric.name === 'CLS' ? metric.value * 1000 : metric.value
      ),
      event_label: metric.id,
      non_interaction: true,
    })
  })

  return (
    <>
      <Script
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_MEASUREMENT_ID}`}
        strategy="afterInteractive"
      />
      <Script id="google-analytics" strategy="afterInteractive">
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', '${GA_MEASUREMENT_ID}');
        `}
      </Script>
      <Component {...pageProps} />
    </>
  )
}

export default MyApp
```

## 커스텀 분석 엔드포인트

API 라우트를 사용하여 분석 엔드포인트를 만들 수 있습니다:

```js
// pages/api/analytics.js
export default function handler(req, res) {
  if (req.method === 'POST') {
    const metric = req.body

    // 데이터베이스에 저장하거나 외부 서비스로 전달
    console.log('메트릭 수신:', metric)

    res.status(200).json({ success: true })
  } else {
    res.status(405).end()
  }
}
```

```jsx
// pages/_app.js
useReportWebVitals((metric) => {
  fetch('/api/analytics', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(metric),
  })
})
```

## 디버깅 팁

개발 모드에서 메트릭을 확인하려면:

```jsx
useReportWebVitals((metric) => {
  if (process.env.NODE_ENV === 'development') {
    console.table({
      이름: metric.name,
      값: metric.value,
      등급: metric.rating,
      id: metric.id,
    })
  }
})
```

## 권장 사항

1. **프로덕션에서만 전송**: 개발 환경의 메트릭은 실제 사용자 경험을 반영하지 않습니다.

2. **샘플링 고려**: 트래픽이 많은 사이트에서는 모든 메트릭을 수집하지 않고 샘플링을 고려하세요.

3. **메트릭 개선에 집중**: 수집된 데이터를 기반으로 실제 성능 개선 작업을 수행하세요.

4. **알림 설정**: 메트릭이 임계값을 초과하면 알림을 받도록 설정하세요.
