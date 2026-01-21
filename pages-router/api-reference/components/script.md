# <Script>

Next.js의 Script 컴포넌트는 외부 스크립트를 최적화하여 로드하기 위한 컴포넌트입니다.

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

## Props 참조

| Props | 예제 | 타입 | 필수 여부 |
|-------|------|------|---------|
| [`src`](#src) | `src="http://example.com/script"` | String | 인라인 스크립트 제외 필수 |
| [`strategy`](#strategy) | `strategy="lazyOnload"` | String | - |
| [`onLoad`](#onload) | `onLoad={onLoadFunc}` | Function | - |
| [`onReady`](#onready) | `onReady={onReadyFunc}` | Function | - |
| [`onError`](#onerror) | `onError={onErrorFunc}` | Function | - |

---

## 필수 Props

### `src`

외부 스크립트의 URL을 지정하는 경로 문자열입니다. 절대 URL 또는 상대 경로 모두 가능합니다. 인라인 스크립트를 사용하는 경우를 제외하고 필수입니다.

---

## 선택적 Props

### `strategy`

스크립트의 로딩 전략입니다. 다음 4가지 전략을 사용할 수 있습니다:

#### `beforeInteractive`

Next.js 코드 실행 전, 페이지 hydration 이전에 로드됩니다.

**특징:**
- 초기 HTML에 서버에서 주입됩니다.
- 모든 Next.js 모듈 전에 다운로드 및 순서대로 실행됩니다.
- Preload 및 Fetch되지만 hydration을 블로킹하지 않습니다.
- **`pages/_document.js`에만 배치 가능합니다.**
- **중요한 스크립트에만 사용을 권장합니다.**

**사용 사례:**
- Bot 탐지기
- 쿠키 동의 관리자

```jsx filename="pages/_document.js"
import { Html, Head, Main, NextScript } from 'next/document'
import Script from 'next/script'

export default function Document() {
  return (
    <Html>
      <Head />
      <body>
        <Main />
        <NextScript />
        <Script
          src="https://example.com/script.js"
          strategy="beforeInteractive"
        />
      </body>
    </Html>
  )
}
```

> **참고**: `beforeInteractive` 스크립트는 항상 HTML의 `<head>` 내에 주입됩니다.

#### `afterInteractive` (기본값)

클라이언트 사이드에서 HTML에 주입되며, 페이지 hydration이 어느 정도 진행된 후 로드됩니다.

**특징:**
- Script 컴포넌트의 **기본 전략**입니다.
- 모든 페이지/레이아웃에 배치할 수 있습니다.
- 해당 페이지가 브라우저에서 열릴 때만 로드 및 실행됩니다.

**사용 사례:**
- 태그 관리자
- 분석(Analytics)

```jsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="afterInteractive" />
    </>
  )
}
```

#### `lazyOnload`

브라우저 유휴 시간 동안 클라이언트 사이드에서 HTML에 주입됩니다. 페이지의 모든 리소스를 Fetch한 후 로드됩니다.

**특징:**
- 백그라운드 또는 낮은 우선순위 스크립트에 적합합니다.
- 모든 페이지/레이아웃에 배치할 수 있습니다.
- 해당 페이지가 브라우저에서 열릴 때만 로드 및 실행됩니다.

**사용 사례:**
- 채팅 지원 플러그인
- 소셜 미디어 위젯

```jsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="lazyOnload" />
    </>
  )
}
```

#### `worker` (실험 단계)

웹 워커로 오프로드되어 메인 스레드를 해제합니다.

**특징:**
- 주요 first-party 리소스만 메인 스레드에서 처리됩니다.
- 고급 사용 사례이며, 모든 third-party 스크립트가 작동하는 것을 보장하지 않습니다.
- **`pages/` 디렉토리에서만 사용 가능합니다.**

**설정:**

```js filename="next.config.js"
module.exports = {
  experimental: {
    nextScriptWorkers: true,
  },
}
```

```tsx filename="pages/home.tsx"
import Script from 'next/script'

export default function Home() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="worker" />
    </>
  )
}
```

---

### `onLoad`

스크립트 로드가 완료된 후 JavaScript 코드를 실행합니다.

**제한사항:**
- Server Components에서는 작동하지 않습니다. Client Components에서만 사용하세요.
- `beforeInteractive`와 함께 사용할 수 없습니다. `onReady` 사용을 권장합니다.
- `afterInteractive` 또는 `lazyOnload` 전략과 함께 사용하세요.

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

---

### `onReady`

스크립트 로드가 완료된 후, 그리고 매번 컴포넌트가 마운트될 때 JavaScript 코드를 실행합니다.

**특징:**
- Server Components에서는 작동하지 않습니다. Client Components에서만 사용하세요.
- 라우트 네비게이션 후 재마운트될 때마다 실행됩니다.

```jsx
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

---

### `onError`

스크립트 로드가 실패했을 때 에러를 처리합니다.

**제한사항:**
- Server Components에서는 작동하지 않습니다. Client Components에서만 사용하세요.
- `beforeInteractive` 전략과 함께 사용할 수 없습니다.

```jsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        onError={(e) => {
          console.error('스크립트 로드 실패', e)
        }}
      />
    </>
  )
}
```

---

## 인라인 스크립트

인라인 스크립트도 Script 컴포넌트에서 지원됩니다. 중괄호 안에 JavaScript를 작성하세요:

```jsx
<Script id="show-banner">
  {`document.getElementById('banner').classList.remove('hidden')`}
</Script>
```

또는 `dangerouslySetInnerHTML` 속성을 사용할 수 있습니다:

```jsx
<Script
  id="show-banner"
  dangerouslySetInnerHTML={{
    __html: `document.getElementById('banner').classList.remove('hidden')`,
  }}
/>
```

> **주의**: 인라인 스크립트에는 Next.js가 스크립트를 추적하고 최적화하기 위해 `id` 속성이 필요합니다.

---

## 추가 속성

Script 컴포넌트에서 사용하지 않는 많은 DOM 속성이 있습니다. `nonce`나 [커스텀 데이터 속성](https://developer.mozilla.org/docs/Web/HTML/Global_attributes/data-*)과 같은 속성들은 최종 렌더링된 `<script>` 요소로 전달됩니다.

```jsx
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

---

## 버전 이력

| 버전 | 변경 사항 |
|------|---------|
| `v13.0.0` | `beforeInteractive`와 `afterInteractive`가 `app` 디렉토리를 지원하도록 수정 |
| `v12.2.4` | `onReady` prop 추가 |
| `v12.2.2` | `next/script`를 `_document`에 배치 가능하도록 허용 |
| `v11.0.0` | `next/script` 도입 |
