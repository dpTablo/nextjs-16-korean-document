# 스크립트 최적화

`next/script` 컴포넌트를 사용하여 서드파티 스크립트의 로딩과 실행을 최적화할 수 있습니다.

## 기본 사용법

모든 경로에서 서드파티 스크립트를 로드하려면 `next/script`를 import하고 `pages/_app.js`에 포함합니다:

```jsx
// pages/_app.js
import Script from 'next/script'

export default function MyApp({ Component, pageProps }) {
  return (
    <>
      <Component {...pageProps} />
      <Script src="https://example.com/script.js" />
    </>
  )
}
```

**특징:**
- 모든 경로 접근 시 로드 및 실행됩니다.
- Next.js는 여러 페이지를 네비게이션해도 **한 번만 로드**되도록 보장합니다.

> **권장사항:** 성능 영향을 최소화하기 위해 특정 페이지나 레이아웃에만 스크립트를 포함하세요.

## Strategy 속성

`strategy` 속성으로 스크립트 로드 동작을 세밀하게 조정할 수 있습니다:

| Strategy | 설명 | 사용 시기 |
|----------|------|-----------|
| `beforeInteractive` | Next.js 코드와 페이지 hydration 전에 로드 | 중요한 스크립트 (봇 감지, 쿠키 동의 등) |
| `afterInteractive` | **(기본값)** hydration 후 초기에 로드 | 태그 관리자, 분석 도구 |
| `lazyOnload` | 브라우저 유휴 시간 중에 나중에 로드 | 채팅 위젯, 소셜 미디어 플러그인 |
| `worker` | **(실험적)** 웹 워커에서 로드 | 메인 스레드에서 제외하고 싶은 스크립트 |

### 예제

```jsx
// 분석 스크립트 - hydration 후 로드
<Script
  src="https://example.com/analytics.js"
  strategy="afterInteractive"
/>

// 채팅 위젯 - 유휴 시간에 로드
<Script
  src="https://example.com/chat-widget.js"
  strategy="lazyOnload"
/>

// 쿠키 동의 - 페이지 로드 전 필수
<Script
  src="https://example.com/cookie-consent.js"
  strategy="beforeInteractive"
/>
```

## 웹 워커로 스크립트 오프로드 (실험적)

> **경고:** `worker` strategy는 아직 불안정하며 App Router에서 작동하지 않습니다.

### 설정 방법

**1단계: next.config.js 설정**

```js
// next.config.js
module.exports = {
  experimental: {
    nextScriptWorkers: true,
  },
}
```

**2단계: 필요한 패키지 설치**

```bash
npm install @builder.io/partytown
```

**3단계: 스크립트 사용**

```tsx
// pages/home.tsx
import Script from 'next/script'

export default function Home() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="worker" />
    </>
  )
}
```

### Partytown 설정 커스터마이징

`pages/_document.js`에서 Partytown 설정을 커스터마이징할 수 있습니다:

```jsx
// pages/_document.jsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
      <Head>
        <script
          data-partytown-config
          dangerouslySetInnerHTML={{
            __html: `
              partytown = {
                lib: "/_next/static/~partytown/",
                debug: true
              };
            `,
          }}
        />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

## 인라인 스크립트

외부 파일이 아닌 인라인 스크립트도 지원됩니다:

```jsx
// 중괄호 사용
<Script id="show-banner">
  {`document.getElementById('banner').classList.remove('hidden')`}
</Script>

// dangerouslySetInnerHTML 사용
<Script
  id="show-banner"
  dangerouslySetInnerHTML={{
    __html: `document.getElementById('banner').classList.remove('hidden')`,
  }}
/>
```

> **중요:** 인라인 스크립트는 Next.js 추적 및 최적화를 위해 `id` 속성이 필수입니다.

## 이벤트 핸들러

스크립트 로드 후 추가 코드를 실행할 수 있습니다:

| 핸들러 | 설명 |
|--------|------|
| `onLoad` | 스크립트 로드 완료 후 실행 |
| `onReady` | 스크립트 로드 및 컴포넌트 마운트마다 실행 |
| `onError` | 스크립트 로드 실패 시 실행 |

```tsx
// pages/index.tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        onLoad={() => {
          console.log('스크립트가 로드되었습니다')
        }}
      />
    </>
  )
}
```

### onReady 예제

```jsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/map.js"
        onReady={() => {
          // 스크립트가 로드되고 컴포넌트가 마운트될 때마다 실행
          window.initMap()
        }}
      />
      <div id="map" />
    </>
  )
}
```

### onError 예제

```jsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        onError={(e) => {
          console.error('스크립트 로드 실패:', e)
        }}
      />
    </>
  )
}
```

## 추가 속성

`next/script`에 전달된 DOM 속성은 최종 `<script>` 요소로 자동 전달됩니다:

```tsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        id="example-script"
        nonce="XUENAJFW"
        data-test="script"
      />
    </>
  )
}
```

## 일반적인 사용 사례

### Google Analytics

```jsx
// pages/_app.js
import Script from 'next/script'

export default function MyApp({ Component, pageProps }) {
  return (
    <>
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
      <Component {...pageProps} />
    </>
  )
}
```

### Google Tag Manager

```jsx
// pages/_app.js
import Script from 'next/script'

export default function MyApp({ Component, pageProps }) {
  return (
    <>
      <Script
        id="gtm-script"
        strategy="afterInteractive"
        dangerouslySetInnerHTML={{
          __html: `
            (function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
            new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
            j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
            'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
            })(window,document,'script','dataLayer','GTM-XXXXXX');
          `,
        }}
      />
      <Component {...pageProps} />
    </>
  )
}
```

## 성능 최적화 팁

1. **적절한 strategy 선택**: 스크립트의 중요도에 따라 전략을 선택하세요.
2. **필요한 페이지에만 로드**: 모든 페이지에 필요하지 않은 스크립트는 특정 페이지에만 포함하세요.
3. **인라인 스크립트 최소화**: 가능하면 외부 파일을 사용하세요.
4. **비중요 스크립트는 lazyOnload**: 사용자 경험에 즉각적인 영향이 없는 스크립트는 지연 로드하세요.
