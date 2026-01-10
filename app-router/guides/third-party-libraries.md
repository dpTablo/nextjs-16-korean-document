# Third-Party Libraries (서드파티 라이브러리)

Next.js에서 서드파티 라이브러리를 최적화하여 성능과 개발자 경험을 향상시키는 방법을 알아봅니다.

## 개요

**`@next/third-parties`**는 인기 있는 서드파티 서비스를 향상된 성능과 개발자 경험으로 로드할 수 있는 최적화된 컴포넌트와 유틸리티를 제공하는 실험적 Next.js 라이브러리입니다.

## 설치

```bash
npm install @next/third-parties@latest next@latest
```

> **⚠️ 실험적 기능:** 현재 실험 단계이므로 `@latest` 또는 `@canary` 플래그와 함께 사용하세요. 통합 기능이 활발히 추가되고 있습니다.

---

## Google 서드파티 서비스

모든 Google 통합은 `@next/third-parties/google`에서 가져옵니다.

### 1. Google Tag Manager (GTM)

추적 및 분석을 위한 GTM 컨테이너를 인스턴스화합니다.

#### 전체 앱에 적용 (Root Layout)

```tsx
// app/layout.tsx
import { GoogleTagManager } from '@next/third-parties/google'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <GoogleTagManager gtmId="GTM-XYZ" />
      <body>{children}</body>
    </html>
  )
}
```

#### 특정 라우트에만 적용

```jsx
// app/page.tsx
import { GoogleTagManager } from '@next/third-parties/google'

export default function Page() {
  return (
    <div>
      <GoogleTagManager gtmId="GTM-XYZ" />
      {/* 페이지 콘텐츠 */}
    </div>
  )
}
```

#### 이벤트 전송

클라이언트 컴포넌트에서 GTM 이벤트를 전송할 수 있습니다:

```jsx
'use client'

import { sendGTMEvent } from '@next/third-parties/google'

export function EventButton() {
  return (
    <div>
      <button
        onClick={() =>
          sendGTMEvent({
            event: 'buttonClicked',
            value: 'xyz',
          })
        }
      >
        이벤트 전송
      </button>
    </div>
  )
}
```

#### GTM 옵션

| 이름 | 타입 | 설명 |
|------|------|------|
| `gtmId` | 필수* | GTM 컨테이너 ID (`GTM-`로 시작) |
| `gtmScriptUrl` | 선택* | 커스텀 GTM 스크립트 URL |
| `dataLayer` | 선택 | 인스턴스화할 데이터 레이어 객체 |
| `dataLayerName` | 선택 | 데이터 레이어 이름 (기본값: `dataLayer`) |
| `auth` | 선택 | 환경 스니펫용 인증 파라미터 |
| `preview` | 선택 | 환경 스니펫용 미리보기 파라미터 |

*`gtmScriptUrl`이 제공되면 `gtmId`를 생략할 수 있습니다 (Google 태그 게이트웨이용).

#### 서버 사이드 태깅

Google Tag Manager의 서버 사이드 태깅을 사용하는 경우:

```tsx
<GoogleTagManager gtmId="GTM-XYZ" gtmScriptUrl="https://gtm.example.com/gtm.js" />
```

---

### 2. Google Analytics (GA4)

Google Analytics 4(gtag.js)를 로드하여 사용자 행동을 추적합니다.

#### 전체 앱에 적용

```tsx
// app/layout.tsx
import { GoogleAnalytics } from '@next/third-parties/google'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
      <GoogleAnalytics gaId="G-XYZ" />
    </html>
  )
}
```

#### 특정 라우트에만 적용

```jsx
// app/page.tsx
import { GoogleAnalytics } from '@next/third-parties/google'

export default function Page() {
  return (
    <div>
      <GoogleAnalytics gaId="G-XYZ" />
      {/* 페이지 콘텐츠 */}
    </div>
  )
}
```

#### 이벤트 전송

```jsx
'use client'

import { sendGAEvent } from '@next/third-parties/google'

export function EventButton() {
  return (
    <div>
      <button
        onClick={() =>
          sendGAEvent('event', 'buttonClicked', {
            value: 'xyz',
          })
        }
      >
        이벤트 전송
      </button>
    </div>
  )
}
```

#### 전환 추적 예시

```jsx
'use client'

import { sendGAEvent } from '@next/third-parties/google'

export function PurchaseButton() {
  const handlePurchase = () => {
    sendGAEvent('event', 'purchase', {
      transaction_id: 'T12345',
      value: 25.00,
      currency: 'USD',
      items: [
        {
          item_id: 'SKU_12345',
          item_name: '제품 이름',
          quantity: 1,
        },
      ],
    })
  }

  return <button onClick={handlePurchase}>구매하기</button>
}
```

#### 자동 페이지뷰 추적

- 클라이언트 사이드 네비게이션에서 페이지뷰가 자동으로 추적됩니다.
- Analytics 관리 패널에서 "향상된 측정"이 활성화되어 있는지 확인하세요.
- "브라우저 기록 이벤트 기반 페이지 변경" 옵션이 선택되어 있는지 확인하세요.

#### GA4 옵션

| 이름 | 타입 | 설명 |
|------|------|------|
| `gaId` | 필수 | 측정 ID (`G-`로 시작) |
| `dataLayerName` | 선택 | 데이터 레이어 이름 (기본값: `dataLayer`) |
| `nonce` | 선택 | Content Security Policy nonce |

> **권장사항:** 이미 GTM을 사용하고 있다면, 별도 컴포넌트가 아닌 GTM을 통해 GA4를 구성하세요.

---

### 3. Google Maps Embed

지연 로딩을 사용하는 Google Maps 임베드를 추가합니다.

#### 기본 사용법

```jsx
import { GoogleMapsEmbed } from '@next/third-parties/google'

export default function Page() {
  return (
    <GoogleMapsEmbed
      apiKey="YOUR_API_KEY"
      height={400}
      width="100%"
      mode="place"
      q="Brooklyn+Bridge,New+York,NY"
    />
  )
}
```

#### 다양한 모드 예시

**장소 모드:**
```jsx
<GoogleMapsEmbed
  apiKey="YOUR_API_KEY"
  height={400}
  width="100%"
  mode="place"
  q="서울역,대한민국"
/>
```

**방향 모드:**
```jsx
<GoogleMapsEmbed
  apiKey="YOUR_API_KEY"
  height={400}
  width="100%"
  mode="directions"
  origin="Seoul+Station"
  destination="Gangnam+Station"
/>
```

**뷰 모드:**
```jsx
<GoogleMapsEmbed
  apiKey="YOUR_API_KEY"
  height={400}
  width="100%"
  mode="view"
  center="37.5665,126.9780"
  zoom="15"
  maptype="satellite"
/>
```

#### Maps 옵션

| 이름 | 타입 | 설명 |
|------|------|------|
| `apiKey` | 필수 | Google Maps API 키 |
| `mode` | 필수 | 지도 모드 (`place`, `view`, `directions`, `search`, `streetview`) |
| `height` | 선택 | 임베드 높이 (기본값: `auto`) |
| `width` | 선택 | 임베드 너비 (기본값: `auto`) |
| `style` | 선택 | iframe 스타일 |
| `allowfullscreen` | 선택 | 전체화면 모드 활성화 |
| `loading` | 선택 | 로딩 전략 (기본값: `lazy`) |
| `q` | 선택 | 지도 마커 위치 (place 모드) |
| `center` | 선택 | 지도 뷰 중심 (위도,경도) |
| `zoom` | 선택 | 초기 줌 레벨 (0-21) |
| `maptype` | 선택 | 타일 유형 (`roadmap`, `satellite`) |
| `language` | 선택 | UI 언어 (예: `ko`) |
| `region` | 선택 | 지정학적 지역 설정 |

---

### 4. YouTube Embed

`lite-youtube-embed`를 사용하여 최적화된 성능으로 YouTube 임베드를 로드합니다.

#### 기본 사용법

```jsx
import { YouTubeEmbed } from '@next/third-parties/google'

export default function Page() {
  return (
    <YouTubeEmbed
      videoid="ogfYd705cRs"
      height={400}
      params="controls=0"
    />
  )
}
```

#### 고급 예시

```jsx
<YouTubeEmbed
  videoid="dQw4w9WgXcQ"
  height={400}
  params="controls=1&start=10&end=30&autoplay=1&mute=1"
  style={{ borderRadius: '8px' }}
/>
```

#### YouTube 옵션

| 이름 | 타입 | 설명 |
|------|------|------|
| `videoid` | 필수 | YouTube 비디오 ID |
| `width` | 선택 | 컨테이너 너비 (기본값: `auto`) |
| `height` | 선택 | 컨테이너 높이 (기본값: `auto`) |
| `playlabel` | 선택 | 접근성을 위한 재생 버튼 레이블 |
| `params` | 선택 | 플레이어 파라미터 (쿼리 문자열 형식) |
| `style` | 선택 | 컨테이너 스타일 |

#### 유용한 params 옵션

- `controls=0` - 플레이어 컨트롤 숨기기
- `start=10` - 10초부터 시작
- `end=30` - 30초에서 종료
- `autoplay=1` - 자동 재생
- `mute=1` - 음소거
- `loop=1` - 반복 재생
- `modestbranding=1` - YouTube 로고 최소화

---

## Script 컴포넌트를 사용한 일반 서드파티 스크립트

`@next/third-parties`에 포함되지 않은 서비스의 경우, `next/script` 컴포넌트를 사용하세요:

### 기본 사용법

```jsx
import Script from 'next/script'

export default function Page() {
  return (
    <>
      <Script src="https://example.com/script.js" />
      {/* 페이지 콘텐츠 */}
    </>
  )
}
```

### 로딩 전략

#### `beforeInteractive` (중요한 스크립트)

```jsx
<Script
  src="https://example.com/critical.js"
  strategy="beforeInteractive"
/>
```

#### `afterInteractive` (기본값, 권장)

```jsx
<Script
  src="https://example.com/analytics.js"
  strategy="afterInteractive"
/>
```

#### `lazyOnload` (낮은 우선순위)

```jsx
<Script
  src="https://example.com/chat-widget.js"
  strategy="lazyOnload"
/>
```

#### `worker` (실험적, Web Worker에서 실행)

```jsx
<Script
  src="https://example.com/analytics.js"
  strategy="worker"
/>
```

### 인라인 스크립트

```jsx
<Script id="show-banner">
  {`console.log('Banner shown')`}
</Script>
```

### 이벤트 핸들러

```jsx
<Script
  src="https://example.com/script.js"
  onLoad={() => {
    console.log('Script has loaded')
  }}
  onError={(e) => {
    console.error('Script failed to load', e)
  }}
/>
```

---

## 주요 최적화 전략

### ✅ 해야 할 것

1. **지연 로딩** - 컴포넌트는 하이드레이션 후 기본적으로 로드됩니다
2. **지연된 스크립트 로딩** - 페이지가 인터랙티브한 후 스크립트를 가져옵니다
3. **조건부 로딩** - 라우트별 또는 앱 전체에 로드합니다
4. **이벤트 추적** - 분석을 위해 `sendGTMEvent()` 및 `sendGAEvent()` 사용
5. **CSP 지원** - Content Security Policy를 위한 nonce 지원

### ❌ 피해야 할 것

1. **중복 로딩** - GTM과 GA4를 동시에 직접 로드하지 마세요
2. **블로킹 스크립트** - `beforeInteractive`는 정말 필요한 경우에만 사용
3. **과도한 서드파티** - 필요한 것만 로드하세요

---

## 성능 베스트 프랙티스

### 1. 전체 앱 vs 라우트별 로딩

**전체 앱 (Root Layout):**
- 모든 페이지에서 필요한 분석 도구
- GTM, GA4 등

**라우트별:**
- 특정 페이지에서만 필요한 기능
- Maps, YouTube 임베드 등

### 2. 임베드 최적화

```jsx
// 뷰포트 아래에 있는 임베드는 지연 로딩
<GoogleMapsEmbed
  apiKey="YOUR_API_KEY"
  height={400}
  width="100%"
  mode="place"
  q="Seoul Station"
  loading="lazy"
/>
```

### 3. 이벤트 배치 처리

```jsx
'use client'

import { sendGAEvent } from '@next/third-parties/google'

export function ProductCard({ product }) {
  const trackEvent = (action: string) => {
    sendGAEvent('event', action, {
      category: 'Product',
      label: product.name,
      value: product.price,
    })
  }

  return (
    <div>
      <button onClick={() => trackEvent('view_item')}>보기</button>
      <button onClick={() => trackEvent('add_to_cart')}>장바구니</button>
      <button onClick={() => trackEvent('purchase')}>구매</button>
    </div>
  )
}
```

### 4. Content Security Policy (CSP)

```tsx
import { GoogleAnalytics } from '@next/third-parties/google'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const nonce = 'random-nonce-value'

  return (
    <html lang="ko">
      <body>{children}</body>
      <GoogleAnalytics gaId="G-XYZ" nonce={nonce} />
    </html>
  )
}
```

---

## 실전 예시

### E-commerce 추적

```jsx
'use client'

import { sendGAEvent } from '@next/third-parties/google'

export function CheckoutButton({ cart }) {
  const handleCheckout = () => {
    sendGAEvent('event', 'begin_checkout', {
      currency: 'KRW',
      value: cart.total,
      items: cart.items.map((item) => ({
        item_id: item.id,
        item_name: item.name,
        price: item.price,
        quantity: item.quantity,
      })),
    })
  }

  return <button onClick={handleCheckout}>결제하기</button>
}
```

### 커스텀 데이터 레이어

```tsx
import { GoogleTagManager } from '@next/third-parties/google'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <GoogleTagManager
        gtmId="GTM-XYZ"
        dataLayer={{
          userId: 'user123',
          environment: 'production',
        }}
      />
      <body>{children}</body>
    </html>
  )
}
```

---

## 다음 단계

- [Script Component](../api-reference/components/script.md) - Script API 레퍼런스
- [Optimizing](./optimizing.md) - 성능 최적화 가이드
- [Analytics](./analytics.md) - 분석 가이드

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
