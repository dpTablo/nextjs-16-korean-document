# 폰트 최적화

`next/font` 모듈은 자동으로 폰트를 최적화하고 외부 네트워크 요청을 제거하여 개인정보 보호와 성능을 개선합니다.

## 주요 기능

- **내장 자체 호스팅**: 모든 폰트 파일을 자동으로 자체 호스팅합니다.
- **레이아웃 시프트 방지**: CSS `size-adjust` 속성을 사용하여 레이아웃 시프트 없이 웹 폰트를 최적 로딩합니다.
- **개인정보 보호**: Google에 대한 브라우저 요청이 발생하지 않습니다.

## Google Fonts 사용

Google Fonts를 자동으로 자체 호스팅합니다. 폰트가 정적 자산으로 저장되어 배포된 도메인에서 제공됩니다.

### 기본 사용법

```tsx
// pages/_app.tsx
import { Geist } from 'next/font/google'
import type { AppProps } from 'next/app'

const geist = Geist({
  subsets: ['latin'],
})

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <main className={geist.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

```jsx
// pages/_app.js
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={geist.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

### Variable Font 사용 (권장)

Variable Font를 사용하면 하나의 파일로 여러 폰트 굵기와 스타일을 사용할 수 있습니다:

```tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
})
```

### 특정 Weight 지정

Variable Font를 사용할 수 없는 경우 특정 weight를 지정해야 합니다:

```tsx
// pages/_app.tsx
import { Roboto } from 'next/font/google'
import type { AppProps } from 'next/app'

const roboto = Roboto({
  weight: '400',
  subsets: ['latin'],
})

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <main className={roboto.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

여러 weight를 지정할 수도 있습니다:

```tsx
const roboto = Roboto({
  weight: ['400', '700'],
  style: ['normal', 'italic'],
  subsets: ['latin'],
})
```

## _document.js에서 사용

HTML 요소에 직접 폰트를 적용할 수 있습니다:

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document'
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})

export default function Document() {
  return (
    <Html lang="ko" className={geist.className}>
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

## 로컬 폰트 사용

`next/font/local`을 사용하여 로컬 폰트 파일을 로드할 수 있습니다.

### 단일 폰트 파일

```tsx
// pages/_app.tsx
import localFont from 'next/font/local'
import type { AppProps } from 'next/app'

const myFont = localFont({
  src: './my-font.woff2',
})

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <main className={myFont.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

### 다중 폰트 파일

여러 폰트 파일을 하나의 폰트 패밀리로 사용할 수 있습니다:

```tsx
import localFont from 'next/font/local'

const roboto = localFont({
  src: [
    {
      path: './Roboto-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './Roboto-Italic.woff2',
      weight: '400',
      style: 'italic',
    },
    {
      path: './Roboto-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
    {
      path: './Roboto-BoldItalic.woff2',
      weight: '700',
      style: 'italic',
    },
  ],
})
```

## 한국어 폰트 사용

한국어를 지원하는 Google Fonts를 사용할 수 있습니다:

```tsx
import { Noto_Sans_KR } from 'next/font/google'

const notoSansKR = Noto_Sans_KR({
  weight: ['400', '700'],
  subsets: ['latin'],
  preload: false, // 한국어 폰트는 용량이 크므로 preload 비활성화 권장
})
```

## CSS 변수로 사용

`className` 대신 CSS 변수를 사용할 수도 있습니다:

```tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={inter.variable}>
      <Component {...pageProps} />
    </main>
  )
}
```

```css
/* styles/globals.css */
body {
  font-family: var(--font-inter), sans-serif;
}
```

## Tailwind CSS와 함께 사용

CSS 변수를 Tailwind 설정에 등록할 수 있습니다:

```tsx
// pages/_app.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={`${inter.variable} font-sans`}>
      <Component {...pageProps} />
    </main>
  )
}
```

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)'],
      },
    },
  },
}
```

## 폰트 옵션

| 옵션 | 설명 |
|------|------|
| `weight` | 폰트 굵기 (단일 값 또는 배열) |
| `style` | 폰트 스타일 (`'normal'`, `'italic'`) |
| `subsets` | 폰트 서브셋 (`'latin'`, `'latin-ext'` 등) |
| `display` | 폰트 디스플레이 전략 (`'swap'`, `'block'`, `'fallback'` 등) |
| `preload` | 폰트 프리로드 여부 (기본값: `true`) |
| `variable` | CSS 변수명 |
| `fallback` | 폴백 폰트 배열 |
| `adjustFontFallback` | 레이아웃 시프트 방지를 위한 폴백 폰트 조정 |

## 성능 팁

1. **Variable Font 사용**: 파일 크기를 줄이고 유연성을 높입니다.
2. **서브셋 지정**: 필요한 문자만 포함하여 파일 크기를 줄입니다.
3. **`display: 'swap'` 사용**: FOIT(Flash of Invisible Text)를 방지합니다.
4. **프리로드 최적화**: 중요한 폰트만 프리로드합니다.
