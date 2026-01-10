# Script

`<Script>` 컴포넌트는 Next.js 애플리케이션에서 외부 스크립트를 최적화하여 로드하는 데 사용됩니다.

---

## 기본 사용법

```tsx
import Script from 'next/script'

export default function Dashboard() {
  return (
    <>
      <Script src="https://example.com/script.js" />
    </>
  )
}
```

---

## Props

| Prop | 타입 | 필수 | 설명 |
|------|------|------|------|
| `src` | String | ✓ (인라인 스크립트 제외) | 외부 스크립트 URL |
| `strategy` | String | - | 로딩 전략 (기본값: `afterInteractive`) |
| `onLoad` | Function | - | 스크립트 로드 후 실행 |
| `onReady` | Function | - | 스크립트 로드 후 및 재마운트 시 실행 |
| `onError` | Function | - | 로드 실패 시 실행 |

---

## 로딩 전략 (strategy)

### beforeInteractive

모든 Next.js 코드 및 하이드레이션 전에 로드됩니다.

**특징:**
- 루트 레이아웃 (`app/layout.tsx`)에만 배치 가능
- 사이트 전체에 필요한 스크립트용

**사용 사례:**
- Bot 감지기
- 쿠키 동의 관리자

```tsx
// app/layout.tsx
import Script from 'next/script'

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Script
          src="https://example.com/script.js"
          strategy="beforeInteractive"
        />
      </body>
    </html>
  )
}
```

### afterInteractive (기본값)

클라이언트 측에서 일부 하이드레이션 후 로드됩니다.

**특징:**
- 모든 페이지/레이아웃에 배치 가능

**사용 사례:**
- 태그 관리자
- 분석 도구

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="afterInteractive" />
    </>
  )
}
```

### lazyOnload

브라우저 유휴 시간에 로드됩니다.

**특징:**
- 모든 페이지 리소스 로드 후 실행

**사용 사례:**
- 채팅 플러그인
- 소셜 미디어 위젯

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="lazyOnload" />
    </>
  )
}
```

### worker (실험적)

웹 워커에서 오프로드합니다.

**특징:**
- `pages/` 디렉토리에서만 사용 가능
- `next.config.js`에서 활성화 필요

```js
// next.config.js
module.exports = {
  experimental: {
    nextScriptWorkers: true,
  },
}
```

---

## 콜백 함수

### onLoad

스크립트 로드 완료 후 실행됩니다.

> **주의**: Client Components에서만 사용 가능 (`beforeInteractive` 제외)

```tsx
'use client'

import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.20/lodash.min.js"
        onLoad={() => {
          console.log(_.sample([1, 2, 3, 4]))
        }}
      />
    </>
  )
}
```

### onReady

스크립트 로드 완료 후 및 컴포넌트 재마운트 시마다 실행됩니다.

> **주의**: Client Components에서만 사용 가능

```tsx
'use client'

import { useRef } from 'react'
import Script from 'next/script'

export default function Page() {
  const mapRef = useRef()

  return (
    <>
      <div ref={mapRef}></div>
      <Script
        id="google-maps"
        src="https://maps.googleapis.com/maps/api/js"
        onReady={() => {
          new google.maps.Map(mapRef.current, {
            center: { lat: -34.397, lng: 150.644 },
            zoom: 8,
          })
        }}
      />
    </>
  )
}
```

### onError

스크립트 로드 실패 시 실행됩니다.

> **주의**: Client Components에서만 사용 가능 (`beforeInteractive` 제외)

```tsx
'use client'

import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        onError={(e: Error) => {
          console.error('Script failed to load', e)
        }}
      />
    </>
  )
}
```

---

## 사용 예제

### Google Analytics

```tsx
// app/layout.tsx
import Script from 'next/script'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Script
          src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"
          strategy="afterInteractive"
        />
        <Script id="google-analytics" strategy="afterInteractive">
          {`
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', 'GA_MEASUREMENT_ID');
          `}
        </Script>
      </body>
    </html>
  )
}
```

### 인라인 스크립트

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script id="show-banner">
        {`document.getElementById('banner').classList.remove('hidden')`}
      </Script>
    </>
  )
}
```

### 외부 스크립트 동적 로딩

```tsx
'use client'

import { useState } from 'react'
import Script from 'next/script'

export default function Page() {
  const [loaded, setLoaded] = useState(false)

  return (
    <>
      <button onClick={() => setLoaded(true)}>
        Load Script
      </button>
      {loaded && (
        <Script
          src="https://example.com/script.js"
          onLoad={() => {
            console.log('Script loaded!')
          }}
        />
      )}
    </>
  )
}
```

### Facebook Pixel

```tsx
// app/layout.tsx
import Script from 'next/script'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Script
          id="fb-pixel"
          strategy="afterInteractive"
          dangerouslySetInnerHTML={{
            __html: `
              !function(f,b,e,v,n,t,s)
              {if(f.fbq)return;n=f.fbq=function(){n.callMethod?
              n.callMethod.apply(n,arguments):n.queue.push(arguments)};
              if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
              n.queue=[];t=b.createElement(e);t.async=!0;
              t.src=v;s=b.getElementsByTagName(e)[0];
              s.parentNode.insertBefore(t,s)}(window, document,'script',
              'https://connect.facebook.net/en_US/fbevents.js');
              fbq('init', 'YOUR_PIXEL_ID');
              fbq('track', 'PageView');
            `,
          }}
        />
      </body>
    </html>
  )
}
```

---

## 전략별 비교

| 전략 | 로딩 시점 | 사용 위치 | 사용 사례 |
|------|----------|----------|----------|
| `beforeInteractive` | 하이드레이션 전 | 루트 레이아웃만 | Bot 감지, 쿠키 동의 |
| `afterInteractive` | 하이드레이션 후 | 모든 곳 | 분석, 태그 관리자 |
| `lazyOnload` | 유휴 시간 | 모든 곳 | 채팅, 소셜 위젯 |
| `worker` | 웹 워커 | Pages Router만 | 무거운 스크립트 |

---

## 중요한 주의사항

> **Good to know**:
> * `onLoad`, `onReady`, `onError`는 Client Components에서만 사용 가능합니다
> * `beforeInteractive`는 루트 레이아웃에서만 작동합니다
> * 인라인 스크립트는 `id` prop이 필요합니다
> * 동일한 스크립트를 여러 컴포넌트에서 사용해도 한 번만 로드됩니다

> **성능 팁**:
> * 필수가 아닌 스크립트는 `lazyOnload` 사용
> * 분석 도구는 `afterInteractive` 사용
> * 중요한 폴리필이나 보안 스크립트만 `beforeInteractive` 사용

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.0.0 | `app` 디렉토리 지원 |
| v12.2.4 | `onReady` prop 추가 |
| v12.2.2 | `_document`에서 `beforeInteractive` 지원 |
| v11.0.0 | `Script` 컴포넌트 도입 |

---

## 관련 문서

- [Font Optimization](./font.md)
- [Image Optimization](../image.md)
- [Optimizing](../../getting-started/09-image-optimization.md)
