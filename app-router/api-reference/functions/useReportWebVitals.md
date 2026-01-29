---
원문: https://nextjs.org/docs/app/api-reference/functions/use-report-web-vitals
버전: 16.1.6
---

# useReportWebVitals

## 개요

`useReportWebVitals`는 [Core Web Vitals](https://web.dev/vitals/)를 보고하고 분석 서비스와 통합하기 위한 Next.js 유틸리티 hook입니다.

---

## 주요 특징

- TTFB, FCP, LCP, FID, CLS, INP를 포함한 성능 메트릭 보고
- "good", "needs-improvement", "poor" 정성적 평가 제공
- 외부 분석 플랫폼(Google Analytics, 커스텀 엔드포인트 등)과 연동
- `'use client'` 지시어 필요

---

## 기본 설정

### 1. Web Vitals 컴포넌트 생성

```jsx
// app/_components/web-vitals.js
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    console.log(metric)
  })

  return null
}
```

### 2. Root Layout에 추가

```jsx
// app/layout.js
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

---

## API 레퍼런스

### 함수 시그니처

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

useReportWebVitals((metric: Metric) => void)
```

### 매개변수

- **콜백 함수**: `Metric` 객체를 받는 함수

---

## Metric 객체 속성

### 전체 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| `id` | `string` | 메트릭의 고유 식별자 |
| `name` | `string` | 메트릭 이름 (TTFB, FCP, LCP, FID, CLS, INP) |
| `value` | `number` | 실제 성능 값 (밀리초) |
| `delta` | `number` | 이전 값과의 변화량 |
| `rating` | `'good' \| 'needs-improvement' \| 'poor'` | 정성적 평가 |
| `entries` | `PerformanceEntry[]` | PerformanceEntry 객체 배열 |
| `navigationType` | `string` | 네비게이션 타입 ("navigate", "reload", "back_forward", "prerender") |

---

## Core Web Vitals 메트릭

### 1. **TTFB (Time to First Byte)**
- 첫 바이트가 도착하기까지의 시간
- **Good:** < 800ms
- **Needs Improvement:** 800ms ~ 1800ms
- **Poor:** > 1800ms

### 2. **FCP (First Contentful Paint)**
- 첫 번째 콘텐츠가 렌더링되는 시간
- **Good:** < 1.8s
- **Needs Improvement:** 1.8s ~ 3.0s
- **Poor:** > 3.0s

### 3. **LCP (Largest Contentful Paint)**
- 가장 큰 콘텐츠 요소가 렌더링되는 시간
- **Good:** < 2.5s
- **Needs Improvement:** 2.5s ~ 4.0s
- **Poor:** > 4.0s

### 4. **FID (First Input Delay)** [Deprecated]
- 사용자 입력에 대한 첫 응답 지연 시간
- **Good:** < 100ms
- **Needs Improvement:** 100ms ~ 300ms
- **Poor:** > 300ms

### 5. **CLS (Cumulative Layout Shift)**
- 누적 레이아웃 이동 점수
- **Good:** < 0.1
- **Needs Improvement:** 0.1 ~ 0.25
- **Poor:** > 0.25

### 6. **INP (Interaction to Next Paint)**
- 사용자 상호작용부터 다음 페인트까지의 시간
- **Good:** < 200ms
- **Needs Improvement:** 200ms ~ 500ms
- **Poor:** > 500ms

---

## 실용적인 예제

### 1. 콘솔에 로깅

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    console.log({
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
    })
  })

  return null
}
```

### 2. Google Analytics로 전송

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Google Analytics 4
    window.gtag('event', metric.name, {
      value: Math.round(
        metric.name === 'CLS' ? metric.value * 1000 : metric.value
      ),
      event_label: metric.id,
      non_interaction: true,
    })
  })

  return null
}
```

### 3. 커스텀 분석 엔드포인트로 전송

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

function sendWebVitals(metric) {
  const body = JSON.stringify(metric)
  const url = 'https://example.com/analytics'

  // sendBeacon 사용 가능하면 사용 (페이지 종료 시에도 전송 보장)
  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, body)
  } else {
    // 폴백: fetch with keepalive
    fetch(url, {
      body,
      method: 'POST',
      keepalive: true,
      headers: {
        'Content-Type': 'application/json',
      },
    })
  }
}

export function WebVitals() {
  useReportWebVitals(sendWebVitals)
  return null
}
```

### 4. Vercel Analytics

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'
import { sendGAEvent } from '@next/third-parties/google'

export function WebVitals() {
  useReportWebVitals((metric) => {
    sendGAEvent({
      event: 'web-vitals',
      event_category:
        metric.rating === 'good'
          ? 'Web Vitals Good'
          : metric.rating === 'needs-improvement'
          ? 'Web Vitals Needs Improvement'
          : 'Web Vitals Poor',
      value: Math.round(metric.value),
      event_label: metric.id,
      non_interaction: true,
    })
  })

  return null
}
```

### 5. 메트릭별 처리

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'LCP':
        console.log('Largest Contentful Paint:', metric.value)
        break
      case 'FID':
        console.log('First Input Delay:', metric.value)
        break
      case 'CLS':
        console.log('Cumulative Layout Shift:', metric.value)
        break
      case 'FCP':
        console.log('First Contentful Paint:', metric.value)
        break
      case 'TTFB':
        console.log('Time to First Byte:', metric.value)
        break
      case 'INP':
        console.log('Interaction to Next Paint:', metric.value)
        break
    }
  })

  return null
}
```

### 6. Poor 메트릭 알림

```tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    if (metric.rating === 'poor') {
      // Poor 성능 메트릭에 대한 알림 전송
      fetch('/api/alerts', {
        method: 'POST',
        body: JSON.stringify({
          type: 'web-vitals-poor',
          metric: metric.name,
          value: metric.value,
          url: window.location.href,
        }),
      })
    }
  })

  return null
}
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **별도의 Client Component로 분리**
   ```tsx
   // ✅ 좋은 예: 별도 컴포넌트
   // app/_components/web-vitals.tsx
   'use client'

   export function WebVitals() {
     useReportWebVitals(logWebVitals)
     return null
   }
   ```

   ```tsx
   // app/layout.tsx (Server Component)
   import { WebVitals } from './_components/web-vitals'

   export default function RootLayout({ children }) {
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

2. **sendBeacon 우선 사용**
   ```tsx
   // 페이지 종료 시에도 전송 보장
   if (navigator.sendBeacon) {
     navigator.sendBeacon(url, body)
   } else {
     fetch(url, { body, method: 'POST', keepalive: true })
   }
   ```

3. **rating 기반 분류**
   ```tsx
   useReportWebVitals((metric) => {
     const category = `Web Vitals ${
       metric.rating === 'good'
         ? 'Good'
         : metric.rating === 'needs-improvement'
         ? 'Needs Improvement'
         : 'Poor'
     }`

     sendToAnalytics(category, metric)
   })
   ```

### ❌ 피해야 할 사항

1. **Server Component에서 직접 사용**
   ```tsx
   // ❌ 작동하지 않음
   export default function Layout() {
     useReportWebVitals(() => {}) // Error: Client Hook
   }
   ```

2. **과도한 처리**
   ```tsx
   // ❌ 콜백에서 무거운 작업 수행
   useReportWebVitals((metric) => {
     heavyComputation() // 성능 저하
     complexDataProcessing()
   })
   ```

3. **동기적 fetch 사용**
   ```tsx
   // ❌ keepalive 없이 사용하면 페이지 종료 시 전송 실패 가능
   fetch(url, { body, method: 'POST' }) // keepalive: true 누락
   ```

---

## Navigation Types

| 타입 | 설명 |
|------|------|
| `navigate` | 일반적인 네비게이션 (링크 클릭, URL 입력) |
| `reload` | 페이지 새로고침 |
| `back_forward` | 브라우저의 뒤로/앞으로 버튼 |
| `prerender` | 사전 렌더링된 페이지 |

---

## TypeScript 타입

```tsx
type Metric = {
  id: string
  name: 'TTFB' | 'FCP' | 'LCP' | 'FID' | 'CLS' | 'INP'
  value: number
  delta: number
  rating: 'good' | 'needs-improvement' | 'poor'
  entries: PerformanceEntry[]
  navigationType: 'navigate' | 'reload' | 'back_forward' | 'prerender'
}

type ReportWebVitalsCallback = (metric: Metric) => void

function useReportWebVitals(callback: ReportWebVitalsCallback): void
```

---

## 주의사항

### 1. Client Component 필수

```tsx
'use client' // 필수!

import { useReportWebVitals } from 'next/web-vitals'
```

### 2. Root Layout에 배치 권장

가능한 한 Root Layout에 배치하여 모든 페이지의 메트릭을 수집하세요.

### 3. FID는 Deprecated

`FID`(First Input Delay)는 deprecated되었으며, `INP`(Interaction to Next Paint)로 대체되었습니다.

---

## 관련 리소스

- [Web Vitals 공식 사이트](https://web.dev/vitals/)
- [Core Web Vitals 가이드](https://web.dev/articles/vitals)
- [Chrome User Experience Report](https://developers.google.com/web/tools/chrome-user-experience-report)

---

## 관련 API

- [Google Analytics Integration](https://nextjs.org/docs/app/building-your-application/optimizing/analytics)
- [Vercel Analytics](https://vercel.com/analytics)
- [`next/third-parties`](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries)

---

## 버전 정보

- **도입 버전:** Next.js 9.4.0
- **`next/web-vitals`로 이동:** Next.js 13.0.0
- **INP 추가:** Next.js 13.4.0
- **현재 상태:** Stable

---

## 요약

- **용도:** Core Web Vitals 성능 메트릭 수집 및 분석
- **주요 메트릭:** TTFB, FCP, LCP, CLS, INP (FID deprecated)
- **사용 위치:** Client Component에서만 사용 가능
- **권장 배치:** Root Layout
- **분석 연동:** Google Analytics, Vercel Analytics, 커스텀 엔드포인트
- **베스트 프랙티스:** 별도 Client Component로 분리, sendBeacon 사용
