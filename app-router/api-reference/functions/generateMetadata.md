# generateMetadata

`generateMetadata` 함수는 Next.js App Router에서 동적 메타데이터를 정의하는 방법입니다. 라우트 매개변수, 외부 데이터, 부모 세그먼트의 메타데이터에 따라 동적으로 메타데이터를 생성할 수 있습니다.

---

## 기본 사용법

### TypeScript 예제

```typescript
// app/products/[id]/page.tsx
import type { Metadata, ResolvingMetadata } from 'next'

type Props = {
  params: Promise<{ id: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}

export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  // 라우트 매개변수 읽기
  const { id } = await params

  // 데이터 가져오기
  const product = await fetch(`https://.../${id}`).then((res) => res.json())

  // 부모 메타데이터 접근 및 확장
  const previousImages = (await parent).openGraph?.images || []

  return {
    title: product.title,
    openGraph: {
      images: ['/some-specific-page-image.jpg', ...previousImages],
    },
  }
}

export default function Page({ params, searchParams }: Props) {}
```

### JavaScript 예제

```javascript
// app/products/[id]/page.js
export async function generateMetadata({ params, searchParams }, parent) {
  const { id } = await params
  const product = await fetch(`https://.../${id}`).then((res) => res.json())
  const previousImages = (await parent).openGraph?.images || []

  return {
    title: product.title,
    openGraph: {
      images: ['/some-specific-page-image.jpg', ...previousImages],
    },
  }
}

export default function Page({ params, searchParams }) {}
```

---

## 매개변수 (Parameters)

### Props 객체

#### 1. **params** - 동적 라우트 매개변수

현재 라우트의 동적 세그먼트에서 추출한 매개변수 객체입니다.

| 라우트 | URL | params |
|--------|-----|--------|
| `app/shop/[slug]/page.js` | `/shop/1` | `{ slug: '1' }` |
| `app/shop/[tag]/[item]/page.js` | `/shop/1/2` | `{ tag: '1', item: '2' }` |
| `app/shop/[...slug]/page.js` | `/shop/1/2` | `{ slug: ['1', '2'] }` |

#### 2. **searchParams** - URL 검색 매개변수

현재 URL의 쿼리 문자열 매개변수입니다.

| URL | searchParams |
|-----|--------------|
| `/shop?a=1` | `{ a: '1' }` |
| `/shop?a=1&b=2` | `{ a: '1', b: '2' }` |
| `/shop?a=1&a=2` | `{ a: ['1', '2'] }` |

#### 3. **parent** - 부모 메타데이터

Promise로 감싸인 부모 라우트 세그먼트의 해결된 메타데이터입니다.

```typescript
const parent: ResolvingMetadata
const previousMetadata = await parent
```

---

## 반환값 (Returns)

`generateMetadata`는 다음 메타데이터 필드를 포함할 수 있는 `Metadata` 객체를 반환합니다:

### 기본 필드

#### title

```typescript
export async function generateMetadata() {
  return {
    title: 'My Page Title'
  }
}
```

**템플릿 옵션:**

```typescript
// default - 자식 세그먼트의 기본값
title: {
  default: 'Acme'
}

// template - 자식 세그먼트에 접두사/접미사 추가
title: {
  template: '%s | Acme',
  default: 'Acme'
}

// absolute - 부모의 템플릿 무시
title: {
  absolute: 'About'
}
```

#### description

```typescript
export async function generateMetadata() {
  return {
    description: 'The React Framework for the Web'
  }
}
```

### 추가 필드

```typescript
export const metadata: Metadata = {
  generator: 'Next.js',
  applicationName: 'Next.js',
  referrer: 'origin-when-cross-origin',
  keywords: ['Next.js', 'React', 'JavaScript'],
  authors: [{ name: 'Seb' }, { name: 'Josh', url: 'https://nextjs.org' }],
  creator: 'Jiachi Liu',
  publisher: 'Sebastian Markbåge',
  formatDetection: {
    email: false,
    address: false,
    telephone: false,
  },
}
```

---

## 주요 메타데이터 필드

### openGraph

```typescript
export const metadata: Metadata = {
  openGraph: {
    title: 'Next.js',
    description: 'The React Framework for the Web',
    url: 'https://nextjs.org',
    siteName: 'Next.js',
    images: [
      {
        url: 'https://nextjs.org/og.png', // 절대 URL 필수
        width: 800,
        height: 600,
      },
      {
        url: 'https://nextjs.org/og-alt.png',
        width: 1800,
        height: 1600,
        alt: 'My custom alt',
      },
    ],
    videos: [
      {
        url: 'https://nextjs.org/video.mp4',
        width: 800,
        height: 600,
      },
    ],
    audio: [
      {
        url: 'https://nextjs.org/audio.mp3',
      },
    ],
    locale: 'en_US',
    type: 'website',
  },
}
```

**아티클 타입:**

```typescript
export const metadata: Metadata = {
  openGraph: {
    title: 'Next.js',
    description: 'The React Framework for the Web',
    type: 'article',
    publishedTime: '2023-01-01T00:00:00.000Z',
    authors: ['Seb', 'Josh'],
  },
}
```

### robots

```typescript
export const metadata: Metadata = {
  robots: {
    index: true,
    follow: true,
    nocache: false,
    googleBot: {
      index: true,
      follow: true,
      noimageindex: false,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
}
```

### twitter

```typescript
export const metadata: Metadata = {
  twitter: {
    card: 'summary_large_image',
    title: 'Next.js',
    description: 'The React Framework for the Web',
    siteId: '1467726470533754880',
    creator: '@nextjs',
    creatorId: '1467726470533754880',
    images: ['https://nextjs.org/og.png'],
  },
}
```

**앱 카드:**

```typescript
export const metadata: Metadata = {
  twitter: {
    card: 'app',
    app: {
      name: 'twitter_app',
      id: {
        iphone: 'twitter_app://iphone',
        ipad: 'twitter_app://ipad',
        googleplay: 'twitter_app://googleplay',
      },
      url: {
        iphone: 'https://iphone_url',
        ipad: 'https://ipad_url',
      },
    },
  },
}
```

### icons

```typescript
export const metadata: Metadata = {
  icons: {
    icon: '/icon.png',
    shortcut: '/shortcut-icon.png',
    apple: '/apple-icon.png',
    other: {
      rel: 'apple-touch-icon-precomposed',
      url: '/apple-touch-icon-precomposed.png',
    },
  },
}
```

**배열 형식:**

```typescript
export const metadata: Metadata = {
  icons: {
    icon: [
      { url: '/icon.png' },
      new URL('/icon.png', 'https://example.com'),
      { url: '/icon-dark.png', media: '(prefers-color-scheme: dark)' },
    ],
    apple: [
      { url: '/apple-icon.png' },
      { url: '/apple-icon-x3.png', sizes: '180x180', type: 'image/png' },
    ],
  },
}
```

### metadataBase

```typescript
export const metadata: Metadata = {
  metadataBase: new URL('https://acme.com'),
  alternates: {
    canonical: '/',
    languages: {
      'en-US': '/en-US',
      'de-DE': '/de-DE',
    },
  },
  openGraph: {
    images: '/og-image.png', // https://acme.com/og-image.png로 변환됨
  },
}
```

**URL 변환 테이블:**

| metadata 필드 | 변환된 URL |
|--------------|-----------|
| `/` | `https://acme.com` |
| `./` | `https://acme.com` |
| `payments` | `https://acme.com/payments` |
| `/payments` | `https://acme.com/payments` |
| `./payments` | `https://acme.com/payments` |
| `../payments` | `https://acme.com/payments` |
| `https://beta.acme.com/payments` | `https://beta.acme.com/payments` |

### alternates

```typescript
export const metadata: Metadata = {
  alternates: {
    canonical: 'https://nextjs.org',
    languages: {
      'en-US': 'https://nextjs.org/en-US',
      'de-DE': 'https://nextjs.org/de-DE',
    },
    media: {
      'only screen and (max-width: 600px)': 'https://nextjs.org/mobile',
    },
    types: {
      'application/rss+xml': 'https://nextjs.org/rss',
    },
  },
}
```

### appleWebApp

```typescript
export const metadata: Metadata = {
  itunes: {
    appId: 'myAppStoreID',
    appArgument: 'myAppArgument',
  },
  appleWebApp: {
    title: 'Apple Web App',
    statusBarStyle: 'black-translucent',
    startupImage: [
      '/assets/startup/apple-touch-startup-image-768x1004.png',
      {
        url: '/assets/startup/apple-touch-startup-image-1536x2008.png',
        media: '(device-width: 768px) and (device-height: 1024px)',
      },
    ],
  },
}
```

### verification

```typescript
export const metadata: Metadata = {
  verification: {
    google: 'google',
    yandex: 'yandex',
    yahoo: 'yahoo',
    other: {
      me: ['my-email', 'my-link'],
    },
  },
}
```

### appLinks

```typescript
export const metadata: Metadata = {
  appLinks: {
    ios: {
      url: 'https://nextjs.org/ios',
      app_store_id: 'app_store_id',
    },
    android: {
      package: 'com.example.android/package',
      app_name: 'app_name_android',
    },
    web: {
      url: 'https://nextjs.org/web',
      should_fallback: true,
    },
  },
}
```

### 기타 필드

```typescript
export const metadata: Metadata = {
  manifest: 'https://nextjs.org/manifest.json',
  facebook: {
    appId: '12345678',
    // 또는
    admins: ['12345678', '87654321'],
  },
  pinterest: {
    richPin: true,
  },
  archives: ['https://nextjs.org/13'],
  assets: ['https://nextjs.org/assets'],
  bookmarks: ['https://nextjs.org/13'],
  category: 'technology',
  other: {
    custom: 'meta',
  },
}
```

---

## 주요 특징

### ✅ Server Component 전용

- `generateMetadata`는 Server Component에서만 사용 가능
- 메타데이터를 초기 HTML 응답에 포함시킬 수 있음

### ✅ 클라이언트 기능 필요 시

```typescript
// app/page.tsx
import type { Metadata } from 'next'
import { InteractiveComponent } from './interactive-component'

export const metadata: Metadata = {
  title: 'My Page',
}

export default function Page() {
  return <InteractiveComponent />
}
```

```typescript
// app/interactive-component.tsx
'use client'

export function InteractiveComponent() {
  // 클라이언트 측 상호작용
}
```

### ✅ 자동 메모이제이션

`generateMetadata`, `generateStaticParams`, Layouts, Pages, Server Components에서 동일한 데이터에 대한 `fetch` 요청이 자동으로 메모이제이션됨

### ✅ 메타데이터 스트리밍

- v15.2.0부터 스트리밍 지원
- 페이지 렌더링 시 초기 UI 먼저 전송 후 메타데이터는 나중에 전송 가능
- `htmlLimitedBots` 설정으로 제어 가능

---

## 메타데이터 평가 및 병합

### 평가 순서

1. `app/layout.tsx` (루트 레이아웃)
2. `app/blog/layout.tsx` (중첩 레이아웃)
3. `app/blog/[slug]/page.tsx` (페이지)

### 병합 규칙

- 메타데이터 객체는 **얕은 병합(shallow merge)** 수행
- 중복된 키는 평가 순서에 따라 **대체됨**
- 중첩 필드(`openGraph`, `robots` 등)는 이전 세그먼트의 값이 **완전히 덮어씌워짐**

```typescript
// app/layout.js
export const metadata = {
  title: 'Acme',
  openGraph: {
    title: 'Acme',
    description: 'Acme is a...',
  },
}

// app/blog/page.js
export const metadata = {
  title: 'Blog',
  openGraph: {
    title: 'Blog',
  },
}

// 결과:
// <title>Blog</title>
// <meta property="og:title" content="Blog" />
// ⚠️ og:description은 사라짐
```

### 중첩 필드 공유

```typescript
// app/shared-metadata.js
export const openGraphImage = { images: ['http://...'] }

// app/page.js
import { openGraphImage } from './shared-metadata'

export const metadata = {
  openGraph: {
    ...openGraphImage,
    title: 'Home',
  },
}

// app/about/page.js
import { openGraphImage } from '../shared-metadata'

export const metadata = {
  openGraph: {
    ...openGraphImage,
    title: 'About',
  },
}
```

---

## 캐시 컴포넌트와 함께 사용

캐시 컴포넌트가 활성화되면 `generateMetadata`는 다른 컴포넌트와 동일한 규칙을 따릅니다.

```typescript
// app/page.tsx
export async function generateMetadata() {
  'use cache'
  const { title, description } = await db.query('site-metadata')
  return { title, description }
}
```

---

## 지원되지 않는 메타데이터

| 메타데이터 | 권장사항 |
|-----------|--------|
| `<meta http-equiv="...">` | HTTP Headers, Proxy, Security Headers 사용 |
| `<base>` | 레이아웃이나 페이지에서 직접 렌더링 |
| `<noscript>` | 레이아웃이나 페이지에서 직접 렌더링 |
| `<style>` | Next.js의 CSS 스타일링 사용 |
| `<script>` | `next/script` 사용 |

---

## 리소스 힌트 (Resource Hints)

```typescript
// app/preload-resources.tsx
'use client'

import ReactDOM from 'react-dom'

export function PreloadResources() {
  ReactDOM.preload('...', { as: '...' })
  ReactDOM.preconnect('...', { crossOrigin: '...' })
  ReactDOM.prefetchDNS('...')

  return null
}
```

### `<link rel="preload">`

```typescript
ReactDOM.preload(href: string, options: { as: string })
```

### `<link rel="preconnect">`

```typescript
ReactDOM.preconnect(href: string, options?: { crossOrigin?: string })
```

### `<link rel="dns-prefetch">`

```typescript
ReactDOM.prefetchDNS(href: string)
```

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v15.2.0 | `generateMetadata`에 스트리밍 지원 추가 |
| v13.2.0 | `viewport`, `themeColor`, `colorScheme` 제거 (generateViewport로 이동) |
| v13.2.0 | `metadata` 및 `generateMetadata` 도입 |

---

## 주의사항

> **Good to know**:
> * 메타데이터가 런타임 정보에 의존하지 않으면 정적 `metadata` 객체 사용 권장
> * `fetch` 요청은 자동으로 메모이제이션됨
> * `searchParams`는 `page.js` 세그먼트에서만 사용 가능
> * `redirect()`, `notFound()` 메서드를 `generateMetadata` 내에서 사용 가능
> * 파일 기반 메타데이터는 `generateMetadata` 함수보다 우선순위가 높음
