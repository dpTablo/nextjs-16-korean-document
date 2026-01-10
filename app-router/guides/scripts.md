# Scripts

Next.js 애플리케이션에서 서드파티 스크립트를 로드하고 최적화하는 방법을 알아봅니다.

## 개요

이 가이드는 `next/script` 컴포넌트를 사용하여 Next.js 애플리케이션에서 서드파티 스크립트를 로드하고 최적화하는 방법을 다룹니다.

---

## 레이아웃 스크립트

특정 라우트에 대해 스크립트를 로드하려면 레이아웃 컴포넌트를 사용하세요.

**app/dashboard/layout.tsx**
```tsx
import Script from 'next/script'

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <>
      <section>{children}</section>
      <Script src="https://example.com/script.js" />
    </>
  )
}
```

**핵심 포인트:**
- 사용자가 `dashboard/page.js` 또는 중첩된 라우트에 접근할 때 스크립트가 로드됩니다
- Next.js는 여러 라우트 탐색에도 불구하고 스크립트가 **단 한 번만** 로드되도록 보장합니다

---

## 애플리케이션 스크립트

모든 라우트에 대해 전역적으로 스크립트를 로드하려면 루트 레이아웃을 사용하세요.

**app/layout.tsx**
```tsx
import Script from 'next/script'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
      <Script src="https://example.com/script.js" />
    </html>
  )
}
```

**권장사항:** 성능 영향을 최소화하기 위해 서드파티 스크립트는 특정 페이지나 레이아웃에만 포함하는 것이 좋습니다.

---

## 로딩 전략

`strategy` 속성을 사용하여 스크립트 로딩 동작을 제어할 수 있습니다.

### 전략 옵션

| 전략 | 동작 |
|------|------|
| `beforeInteractive` | Next.js 코드 및 페이지 하이드레이션 이전에 로드 |
| `afterInteractive` | **(기본값)** 일부 하이드레이션 후 초기에 로드 |
| `lazyOnload` | 브라우저 유휴 시간에 로드 |
| `worker` | **(실험적)** Web Worker에서 로드 |

### beforeInteractive

페이지가 인터랙티브하게 되기 전에 로드해야 하는 중요한 스크립트에 사용합니다.

**app/layout.tsx**
```tsx
import Script from 'next/script'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
      <Script
        src="https://example.com/script.js"
        strategy="beforeInteractive"
      />
    </html>
  )
}
```

**사용 사례:**
- 봇 탐지기
- 쿠키 동의 관리자
- 기타 중요한 기반 스크립트

### afterInteractive (기본값)

페이지가 인터랙티브하게 된 후 즉시 로드할 스크립트에 사용합니다.

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

**사용 사례:**
- 태그 관리자
- 분석 도구
- 채팅 위젯

### lazyOnload

브라우저 유휴 시간에 로드할 수 있는 낮은 우선순위 스크립트에 사용합니다.

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

**사용 사례:**
- 소셜 미디어 위젯
- 임베드 콘텐츠
- 광고 스크립트

---

## Web Worker 전략 (실험적)

[Partytown](https://partytown.builder.io/)을 사용하여 스크립트를 Web Worker로 오프로드할 수 있습니다.

### 1. 구성 활성화

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    nextScriptWorkers: true,
  },
}

module.exports = nextConfig
```

### 2. Partytown 설치

```bash
npm install @builder.io/partytown
```

### 3. 사용 예시

**app/page.tsx**
```tsx
import Script from 'next/script'

export default function Home() {
  return (
    <>
      <Script
        src="https://example.com/analytics.js"
        strategy="worker"
      />
    </>
  )
}
```

### 주의사항

⚠️ **경고:**
- 아직 안정적이지 않습니다
- App Router와 함께 작동하지 않습니다
- [Partytown 트레이드오프 문서](https://partytown.builder.io/trade-offs)를 검토하세요

**장점:**
- 메인 스레드에서 스크립트를 분리하여 성능 향상
- CPU 집약적인 스크립트에 유용

**단점:**
- 모든 서드파티 스크립트가 호환되는 것은 아닙니다
- 일부 기능(DOM 직접 접근)이 제한될 수 있습니다

---

## 인라인 스크립트

Script 컴포넌트는 인라인 스크립트도 지원합니다.

### 중괄호 사용

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

### dangerouslySetInnerHTML 사용

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        id="show-banner"
        dangerouslySetInnerHTML={{
          __html: `document.getElementById('banner').classList.remove('hidden')`,
        }}
      />
    </>
  )
}
```

### 복잡한 인라인 스크립트

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script id="analytics-init" strategy="afterInteractive">
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', 'GA_MEASUREMENT_ID');
        `}
      </Script>
    </>
  )
}
```

⚠️ **중요:** 인라인 스크립트의 경우 Next.js가 추적하고 최적화할 수 있도록 `id` 속성이 **필수**입니다.

---

## 이벤트 핸들러

스크립트 이벤트 후 추가 코드를 실행할 수 있습니다.

### 클라이언트 컴포넌트에서 사용

이벤트 핸들러는 클라이언트 컴포넌트에서만 사용할 수 있으므로 `"use client"` 지시어가 필요합니다.

**app/page.tsx**
```tsx
'use client'

import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        onLoad={() => {
          console.log('스크립트가 로드되었습니다')
        }}
        onReady={() => {
          console.log('스크립트 준비 완료 및 마운트됨')
        }}
        onError={(e: Error) => {
          console.error('스크립트 로드 실패', e)
        }}
      />
    </>
  )
}
```

### 사용 가능한 핸들러

#### onLoad

스크립트 로드가 완료된 직후 실행됩니다.

```tsx
<Script
  src="https://example.com/script.js"
  onLoad={() => {
    // 스크립트의 전역 변수나 함수 사용
    console.log(window.MyLibrary)
  }}
/>
```

#### onReady

스크립트 로드가 완료되고 컴포넌트가 마운트될 때마다 실행됩니다.

```tsx
<Script
  src="https://example.com/script.js"
  onReady={() => {
    // 컴포넌트가 마운트되면 실행
    console.log('준비 완료')
  }}
/>
```

**차이점:**
- `onLoad`: 스크립트가 처음 로드될 때 한 번만 실행
- `onReady`: 스크립트 로드 후 및 컴포넌트가 마운트될 때마다 실행

#### onError

스크립트 로드가 실패할 때 실행됩니다.

```tsx
<Script
  src="https://example.com/script.js"
  onError={(e: Error) => {
    console.error('스크립트 로드 오류:', e)
    // 폴백 로직 또는 오류 추적
  }}
/>
```

---

## 추가 DOM 속성

`nonce`나 사용자 정의 데이터 속성 같은 추가 속성을 전달할 수 있습니다.

### nonce 속성

Content Security Policy를 위한 nonce 값을 추가합니다.

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        id="example-script"
        nonce="XUENAJFW"
      />
    </>
  )
}
```

### 사용자 정의 데이터 속성

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        id="example-script"
        data-test="script"
        data-custom="value"
      />
    </>
  )
}
```

모든 속성은 HTML의 최종 최적화된 `<script>` 요소로 자동 전달됩니다.

---

## 실전 예시

### Google Analytics

**app/layout.tsx**
```tsx
import Script from 'next/script'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
      <Script
        src={`https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID`}
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
    </html>
  )
}
```

### Facebook Pixel

**app/page.tsx**
```tsx
'use client'

import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        id="facebook-pixel"
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
    </>
  )
}
```

### Intercom 채팅 위젯

**app/layout.tsx**
```tsx
import Script from 'next/script'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
      <Script
        id="intercom-widget"
        strategy="lazyOnload"
        dangerouslySetInnerHTML={{
          __html: `
            window.intercomSettings = {
              app_id: "YOUR_APP_ID"
            };
            (function(){var w=window;var ic=w.Intercom;if(typeof ic==="function"){
              ic('reattach_activator');ic('update',w.intercomSettings);}else{
              var d=document;var i=function(){i.c(arguments);};i.q=[];i.c=function(args)
              {i.q.push(args);};w.Intercom=i;var l=function(){var s=d.createElement('script');
              s.type='text/javascript';s.async=true;s.src='https://widget.intercom.io/widget/YOUR_APP_ID';
              var x=d.getElementsByTagName('script')[0];x.parentNode.insertBefore(s,x);};
              if(document.readyState==='complete'){l();}else if(w.attachEvent){w.attachEvent('onload',l);}
              else{w.addEventListener('load',l,false);}}})();
          `,
        }}
      />
    </html>
  )
}
```

### Stripe

**app/payment/page.tsx**
```tsx
'use client'

import Script from 'next/script'
import { useState } from 'react'

export default function PaymentPage() {
  const [stripe, setStripe] = useState(null)

  return (
    <>
      <Script
        src="https://js.stripe.com/v3/"
        onLoad={() => {
          setStripe(window.Stripe('YOUR_PUBLISHABLE_KEY'))
        }}
      />
      <div>
        {stripe ? <PaymentForm stripe={stripe} /> : <p>로딩 중...</p>}
      </div>
    </>
  )
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **적절한 전략 선택**
   - 중요한 스크립트: `beforeInteractive`
   - 분석/위젯: `afterInteractive`
   - 소셜 미디어/광고: `lazyOnload`

2. **특정 페이지에만 로드**
   ```tsx
   // ✅ 좋은 예: 특정 레이아웃에만
   // app/dashboard/layout.tsx
   <Script src="dashboard-analytics.js" />
   ```

3. **인라인 스크립트에 ID 제공**
   ```tsx
   // ✅ 좋은 예
   <Script id="my-script">{`console.log('Hello')`}</Script>
   ```

4. **이벤트 핸들러 활용**
   ```tsx
   // ✅ 좋은 예
   <Script
     src="script.js"
     onLoad={() => console.log('Loaded')}
     onError={(e) => console.error('Error', e)}
   />
   ```

5. **CSP를 위한 nonce 사용**
   ```tsx
   <Script src="script.js" nonce={nonce} />
   ```

### ❌ 피해야 할 것

1. **모든 스크립트를 루트 레이아웃에 추가**
   ```tsx
   // ❌ 나쁜 예: 불필요한 전역 로드
   // app/layout.tsx
   <Script src="only-needed-in-dashboard.js" />
   ```

2. **인라인 스크립트에 ID 누락**
   ```tsx
   // ❌ 나쁜 예: ID 없음
   <Script>{`console.log('Hello')`}</Script>
   ```

3. **서버 컴포넌트에서 이벤트 핸들러 사용**
   ```tsx
   // ❌ 나쁜 예: 'use client' 없이 onLoad 사용
   <Script src="script.js" onLoad={() => {}} />
   ```

4. **중복 스크립트 로드**
   ```tsx
   // ❌ 나쁜 예: 동일한 스크립트를 여러 곳에서
   <Script src="same-script.js" />
   <Script src="same-script.js" />
   ```

---

## 전략별 비교

### 성능 영향

| 전략 | 로드 시점 | 성능 영향 | 사용 사례 |
|------|-----------|----------|----------|
| `beforeInteractive` | 가장 빠름 | 높음 | 중요한 기반 스크립트 |
| `afterInteractive` | 빠름 | 중간 | 분석, 태그 관리자 |
| `lazyOnload` | 늦음 | 낮음 | 광고, 소셜 위젯 |
| `worker` | 분리 | 매우 낮음 | CPU 집약적 스크립트 |

### 로딩 순서

```
1. beforeInteractive 스크립트
   ↓
2. Next.js 코드 번들
   ↓
3. 페이지 하이드레이션 시작
   ↓
4. afterInteractive 스크립트 (기본값)
   ↓
5. 페이지 완전히 인터랙티브
   ↓
6. lazyOnload 스크립트 (유휴 시간)
```

---

## TypeScript

### 타입이 지정된 이벤트 핸들러

```tsx
import Script from 'next/script'

export default function Page() {
  const handleLoad = () => {
    console.log('스크립트 로드됨')
  }

  const handleError = (e: Error) => {
    console.error('오류:', e.message)
  }

  return (
    <Script
      src="https://example.com/script.js"
      onLoad={handleLoad}
      onError={handleError}
    />
  )
}
```

### 전역 타입 확장

서드파티 스크립트가 전역 변수를 추가하는 경우:

**types/global.d.ts**
```ts
declare global {
  interface Window {
    gtag: (command: string, ...args: any[]) => void
    dataLayer: any[]
    fbq: (command: string, ...args: any[]) => void
    Intercom: (command: string, data?: any) => void
    Stripe: (key: string) => any
  }
}

export {}
```

**사용:**
```tsx
'use client'

import Script from 'next/script'

export default function Page() {
  return (
    <Script
      src="https://www.googletagmanager.com/gtag/js"
      onLoad={() => {
        // 타입 안전성 확보
        window.gtag('config', 'GA_MEASUREMENT_ID')
      }}
    />
  )
}
```

---

## 문제 해결

### 스크립트가 로드되지 않음

**1. 전략 확인:**
```tsx
// ✅ 올바른 전략 사용
<Script src="script.js" strategy="afterInteractive" />
```

**2. ID 확인 (인라인 스크립트):**
```tsx
// ✅ ID 필수
<Script id="my-inline-script">
  {`console.log('Hello')`}
</Script>
```

**3. 네트워크 오류:**
```tsx
<Script
  src="https://example.com/script.js"
  onError={(e) => {
    console.error('로드 실패:', e)
  }}
/>
```

### 스크립트가 중복 로드됨

**해결책:**
- 동일한 레이아웃/페이지에서 스크립트를 한 번만 포함
- Next.js는 라우트 간 중복을 자동으로 방지합니다

```tsx
// ✅ 좋은 예: 한 곳에서만
// app/layout.tsx
<Script src="analytics.js" />

// ❌ 나쁜 예: 여러 곳에서
// app/layout.tsx
<Script src="analytics.js" />
// app/page.tsx
<Script src="analytics.js" /> // 중복!
```

### 이벤트 핸들러가 작동하지 않음

**해결책:** 클라이언트 컴포넌트에서 사용

```tsx
// ✅ 올바른 예
'use client'

import Script from 'next/script'

export default function Page() {
  return (
    <Script
      src="script.js"
      onLoad={() => console.log('Loaded')}
    />
  )
}
```

### Web Worker 전략 오류

**문제:** App Router에서 `worker` 전략이 작동하지 않음

**해결책:**
- Pages Router 사용
- 또는 다른 전략으로 대체
- Partytown 호환성 확인

---

## 마이그레이션

### HTML script 태그에서

**이전:**
```html
<script src="https://example.com/script.js"></script>
```

**이후:**
```tsx
import Script from 'next/script'

<Script src="https://example.com/script.js" />
```

### _document.js에서 (Pages Router)

**이전 (Pages Router):**
```tsx
// pages/_document.js
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
      <Head>
        <script src="https://example.com/script.js" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

**이후 (App Router):**
```tsx
// app/layout.tsx
import Script from 'next/script'

export default function RootLayout({ children }) {
  return (
    <html lang="ko">
      <body>{children}</body>
      <Script src="https://example.com/script.js" strategy="beforeInteractive" />
    </html>
  )
}
```

---

## 성능 최적화

### 1. 스크립트 지연 로드

```tsx
// 즉시 필요하지 않은 스크립트
<Script src="non-critical.js" strategy="lazyOnload" />
```

### 2. 조건부 로드

```tsx
'use client'

import Script from 'next/script'
import { usePathname } from 'next/navigation'

export default function ConditionalScript() {
  const pathname = usePathname()
  const shouldLoadScript = pathname === '/checkout'

  return shouldLoadScript ? (
    <Script src="checkout-script.js" />
  ) : null
}
```

### 3. 번들 크기 확인

```bash
# 프로덕션 빌드 분석
npm run build

# 번들 분석기 사용
npm install --save-dev @next/bundle-analyzer
```

---

## API 참조

자세한 정보는 [Script 컴포넌트 API 참조](../api-reference/components/script.md)를 확인하세요.

---

## 다음 단계

- [Font Optimization](../getting-started/font-optimization.md) - 폰트 최적화
- [Image Optimization](../getting-started/image-optimization.md) - 이미지 최적화
- [Third Party Libraries](./third-party-libraries.md) - 서드파티 라이브러리
- [Content Security Policy](./content-security-policy.md) - CSP 구성

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11

**참고 자료:**
- [Script 컴포넌트 API](https://nextjs.org/docs/app/api-reference/components/script)
- [Partytown 문서](https://partytown.builder.io/)
- [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
