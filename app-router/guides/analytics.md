---
원문: https://nextjs.org/docs/app/guides/analytics
버전: 16.1.6
---

# Analytics (분석)

Next.js에서 성능 지표를 측정하고 보고하는 방법을 알아봅니다.

## 개요

Next.js는 성능 지표를 측정하고 보고하기 위한 내장 지원을 제공합니다. `useReportWebVitals` 훅을 사용하여 커스텀 리포팅을 구현하거나, Vercel의 관리형 분석 서비스를 활용할 수 있습니다.

---

## 클라이언트 계측 (Client Instrumentation)

루트 디렉토리에 `instrumentation-client.js` 또는 `instrumentation-client.ts` 파일을 생성하여 앱이 시작되기 전에 전역 분석을 설정할 수 있습니다:

**instrumentation-client.js**
```js
console.log('Analytics initialized')

// 전역 에러 추적 설정
window.addEventListener('error', (event) => {
  reportError(event.error)
})

// 전역 분석 도구 초기화 예시
if (typeof window !== 'undefined') {
  // Google Analytics 초기화
  window.gtag = window.gtag || function() {
    (window.dataLayer = window.dataLayer || []).push(arguments)
  }
}
```

> **알아두면 좋은 점:**
> - 이 파일은 앱이 시작되기 전에 실행됩니다.
> - 전역 에러 핸들러나 분석 도구 초기화에 적합합니다.

---

## 커스텀 분석 구축

### 1단계: Web Vitals 컴포넌트 생성

`useReportWebVitals` 훅을 사용하여 성능 지표를 수집하는 클라이언트 컴포넌트를 생성합니다:

**app/_components/web-vitals.js**
```jsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    console.log(metric)
  })
}
```

### 2단계: 루트 레이아웃에 가져오기

**app/layout.js**
```jsx
import { WebVitals } from './_components/web-vitals'

export default function Layout({ children }) {
  return (
    <html>
      <body>
        <WebVitals />
        {children}
      </body>
    </html>
  )
}
```

> **성능 팁:** 클라이언트 경계를 WebVitals 컴포넌트에만 국한시키기 위해 별도의 클라이언트 컴포넌트를 사용하세요.

---

## Web Vitals 지표

Next.js는 다음과 같은 핵심 성능 지표를 캡처합니다:

| 지표 | 설명 | 좋은 값 |
|------|------|---------|
| **TTFB** | Time to First Byte (첫 바이트까지의 시간) | < 800ms |
| **FCP** | First Contentful Paint (첫 콘텐츠 페인트) | < 1.8s |
| **LCP** | Largest Contentful Paint (최대 콘텐츠 페인트) | < 2.5s |
| **FID** | First Input Delay (첫 입력 지연) | < 100ms |
| **CLS** | Cumulative Layout Shift (누적 레이아웃 이동) | < 0.1 |
| **INP** | Interaction to Next Paint (다음 페인트까지의 상호작용) | < 200ms |

### Metric 객체 구조

```typescript
{
  name: string        // 지표 이름 (예: 'FCP', 'LCP')
  value: number       // 지표 값
  id: string          // 고유 식별자
  delta: number       // 이전 값과의 차이
  rating: string      // 'good' | 'needs-improvement' | 'poor'
  navigationType: string  // 'navigate' | 'reload' | 'back-forward'
}
```

### 특정 지표 처리

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'FCP': {
        // FCP 결과 처리
        console.log('First Contentful Paint:', metric.value)
        break
      }
      case 'LCP': {
        // LCP 결과 처리
        console.log('Largest Contentful Paint:', metric.value)
        break
      }
      case 'CLS': {
        // CLS 결과 처리
        console.log('Cumulative Layout Shift:', metric.value)
        break
      }
      case 'FID': {
        // FID 결과 처리
        console.log('First Input Delay:', metric.value)
        break
      }
      case 'TTFB': {
        // TTFB 결과 처리
        console.log('Time to First Byte:', metric.value)
        break
      }
      case 'INP': {
        // INP 결과 처리
        console.log('Interaction to Next Paint:', metric.value)
        break
      }
    }
  })
}
```

---

## 외부 시스템으로 결과 전송

### 일반 엔드포인트로 전송

```js
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    const body = JSON.stringify(metric)
    const url = 'https://example.com/analytics'

    // sendBeacon 사용 (권장)
    if (navigator.sendBeacon) {
      navigator.sendBeacon(url, body)
    } else {
      // fallback to fetch
      fetch(url, {
        body,
        method: 'POST',
        keepalive: true
      })
    }
  })
}
```

> **알아두면 좋은 점:**
> - `navigator.sendBeacon()`은 페이지 언로드 시에도 안정적으로 데이터를 전송합니다.
> - `keepalive: true` 옵션은 fetch 요청이 페이지 수명을 넘어서도 완료되도록 합니다.

### Google Analytics 통합

**Google Analytics 4 (GA4) 예시:**

```js
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Google Analytics로 Web Vitals 전송
    window.gtag('event', metric.name, {
      value: Math.round(
        metric.name === 'CLS' ? metric.value * 1000 : metric.value
      ),
      event_label: metric.id,
      non_interaction: true,
    })
  })
}
```

**Google Tag Manager (GTM) 사용 시:**

```js
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    window.dataLayer = window.dataLayer || []
    window.dataLayer.push({
      event: 'web-vitals',
      event_category: 'Web Vitals',
      event_action: metric.name,
      event_value: Math.round(
        metric.name === 'CLS' ? metric.value * 1000 : metric.value
      ),
      event_label: metric.id,
    })
  })
}
```

### Vercel Analytics 통합

Vercel 플랫폼에서 호스팅하는 경우, Vercel Analytics가 자동으로 Web Vitals를 수집합니다.

**설치:**
```bash
npm install @vercel/analytics
```

**사용:**
```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

---

## 지표 분석 및 활용

### 백분위수 계산

`id` 값을 사용하여 지표 분포를 구성하고 백분위수를 계산할 수 있습니다:

```js
const metrics = []

useReportWebVitals((metric) => {
  metrics.push(metric.value)

  // 중앙값 계산
  const sorted = [...metrics].sort((a, b) => a - b)
  const median = sorted[Math.floor(sorted.length / 2)]

  // 75번째 백분위수 계산
  const p75 = sorted[Math.floor(sorted.length * 0.75)]

  console.log(`Median ${metric.name}:`, median)
  console.log(`P75 ${metric.name}:`, p75)
})
```

### 성능 임계값 설정

```js
const THRESHOLDS = {
  LCP: { good: 2500, poor: 4000 },
  FID: { good: 100, poor: 300 },
  CLS: { good: 0.1, poor: 0.25 },
  FCP: { good: 1800, poor: 3000 },
  TTFB: { good: 800, poor: 1800 },
  INP: { good: 200, poor: 500 },
}

useReportWebVitals((metric) => {
  const threshold = THRESHOLDS[metric.name]

  if (metric.value <= threshold.good) {
    console.log(`✅ Good ${metric.name}:`, metric.value)
  } else if (metric.value <= threshold.poor) {
    console.log(`⚠️ Needs improvement ${metric.name}:`, metric.value)
  } else {
    console.log(`❌ Poor ${metric.name}:`, metric.value)
  }
})
```

---

## TypeScript 타입

```typescript
import { useReportWebVitals } from 'next/web-vitals'
import type { Metric } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric: Metric) => {
    // metric은 완전한 타입 안정성을 가집니다
    console.log(metric.name, metric.value)
  })
}
```

---

## 베스트 프랙티스

### 1. **프로덕션에서만 활성화**

```js
export function WebVitals() {
  useReportWebVitals((metric) => {
    if (process.env.NODE_ENV === 'production') {
      // 프로덕션에서만 분석 데이터 전송
      sendToAnalytics(metric)
    } else {
      // 개발 환경에서는 콘솔에만 출력
      console.log(metric)
    }
  })
}
```

### 2. **샘플링 사용**

트래픽이 많은 경우, 모든 사용자의 지표를 수집하는 대신 샘플링을 사용하세요:

```js
export function WebVitals() {
  useReportWebVitals((metric) => {
    // 10%의 사용자만 샘플링
    if (Math.random() < 0.1) {
      sendToAnalytics(metric)
    }
  })
}
```

### 3. **배치 처리**

여러 지표를 모아서 한 번에 전송하여 네트워크 요청을 줄이세요:

```js
let queue = []
let timer = null

export function WebVitals() {
  useReportWebVitals((metric) => {
    queue.push(metric)

    // 5초마다 또는 10개씩 모아서 전송
    if (queue.length >= 10) {
      sendBatch()
    } else {
      clearTimeout(timer)
      timer = setTimeout(sendBatch, 5000)
    }
  })
}

function sendBatch() {
  if (queue.length > 0) {
    sendToAnalytics(queue)
    queue = []
  }
}
```

---

## 다음 단계

- [useReportWebVitals API](../api-reference/functions/useReportWebVitals.md) - API 레퍼런스
- [Optimizing](./optimizing.md) - 성능 최적화 가이드
- [Instrumentation](../api-reference/file-conventions/instrumentation.md) - 서버 사이드 계측

---
