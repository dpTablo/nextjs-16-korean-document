# Font Optimization

`next/font`는 Google Fonts 및 사용자 정의 폰트를 자동으로 최적화하고, 외부 네트워크 요청을 제거하여 프라이버시와 성능을 개선합니다.

---

## 주요 기능

- ✅ **자동 자체 호스팅**: 모든 폰트 파일에 대한 자동 자체 호스팅
- ✅ **레이아웃 시프트 제거**: 웹 폰트를 최적으로 로드
- ✅ **빌드 타임 다운로드**: CSS와 폰트 파일이 빌드 시간에 다운로드되고 정적 자산으로 자체 호스팅됨
- ✅ **Google 폰트 지원**: 브라우저에서 Google으로 요청이 전송되지 않음

---

## 기본 사용법

### Google Fonts 사용

```tsx
// pages/_app.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={inter.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

### 로컬 폰트 사용

```tsx
// pages/_app.tsx
import localFont from 'next/font/local'

const myFont = localFont({
  src: './my-font.woff2',
  display: 'swap',
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={myFont.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

---

## API 참조

### Google Fonts (`next/font/google`)

```typescript
import { FontName } from 'next/font/google'

const font = FontName({
  subsets: string[]
  weight?: string | string[]
  style?: string | string[]
  display?: 'auto' | 'block' | 'swap' | 'fallback' | 'optional'
  preload?: boolean
  fallback?: string[]
  adjustFontFallback?: boolean
  variable?: string
})
```

### Local Fonts (`next/font/local`)

```typescript
import localFont from 'next/font/local'

const font = localFont({
  src: string | Array<{path: string, weight?: string, style?: string}>
  weight?: string | string[]
  style?: string | string[]
  display?: 'auto' | 'block' | 'swap' | 'fallback' | 'optional'
  preload?: boolean
  fallback?: string[]
  adjustFontFallback?: boolean | string
  variable?: string
  declarations?: Array<{prop: string, value: string}>
})
```

---

## 주요 옵션

### weight

폰트 가중치를 지정합니다.

```tsx
// 단일 가중치
const roboto = Roboto({ weight: '400', subsets: ['latin'] })

// 다중 가중치
const roboto = Roboto({ weight: ['400', '700'], subsets: ['latin'] })

// 변수 폰트 범위
const inter = Inter({ weight: '100 900', subsets: ['latin'] })
```

### subsets

폰트 서브셋을 지정합니다 (Google Fonts만 해당).

```tsx
const inter = Inter({
  subsets: ['latin', 'latin-ext'],
})
```

### display

폰트 표시 전략을 설정합니다.

```tsx
const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // 기본값
})
```

**옵션:**
- `'swap'`: 폴백 폰트 먼저 표시, 폰트 로드 후 교체 (권장)
- `'block'`: 폰트 로드될 때까지 텍스트 숨김
- `'fallback'`: 짧은 차단 기간 후 폴백 폰트 표시
- `'optional'`: 폴백 폰트 사용, 네트워크 상태에 따라 폰트 적용
- `'auto'`: 브라우저 기본 동작

### variable

CSS 변수를 생성합니다.

```tsx
const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

// _app.tsx에서 사용
<main className={inter.variable}>
```

---

## 사용 예제

### 비변수 폰트

```tsx
// pages/_app.tsx
import { Roboto } from 'next/font/google'

const roboto = Roboto({
  weight: '400',
  subsets: ['latin'],
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={roboto.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

### 변수 폰트 (권장)

```tsx
// pages/_app.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={inter.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

### 다중 가중치 및 스타일

```tsx
import { Roboto } from 'next/font/google'

const roboto = Roboto({
  weight: ['400', '700'],
  style: ['normal', 'italic'],
  subsets: ['latin'],
})
```

### 로컬 폰트 - 다중 파일

```tsx
import localFont from 'next/font/local'

const roboto = localFont({
  src: [
    {
      path: './fonts/Roboto-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './fonts/Roboto-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
    {
      path: './fonts/Roboto-Italic.woff2',
      weight: '400',
      style: 'italic',
    },
  ],
})
```

---

## 다중 폰트 사용

### 방법 1: 폰트 파일로 관리

```ts
// lib/fonts.ts
import { Inter, Roboto_Mono } from 'next/font/google'

export const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
})

export const roboto_mono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
})
```

```tsx
// pages/_app.tsx
import { inter } from '../lib/fonts'

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={inter.className}>
      <Component {...pageProps} />
    </main>
  )
}
```

```tsx
// pages/index.tsx
import { roboto_mono } from '../lib/fonts'

export default function Page() {
  return <h1 className={roboto_mono.className}>My page</h1>
}
```

### 방법 2: CSS 변수 사용

```tsx
// pages/_app.tsx
import { Inter, Roboto_Mono } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

const roboto_mono = Roboto_Mono({
  subsets: ['latin'],
  variable: '--font-roboto-mono',
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={`${inter.variable} ${roboto_mono.variable}`}>
      <Component {...pageProps} />
    </main>
  )
}
```

```css
/* styles/globals.css */
html {
  font-family: var(--font-inter);
}

h1 {
  font-family: var(--font-roboto-mono);
}
```

---

## Tailwind CSS 통합

### Tailwind v4

```tsx
// pages/_app.tsx
import { Inter, Roboto_Mono } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

const roboto_mono = Roboto_Mono({
  subsets: ['latin'],
  variable: '--font-roboto-mono',
})

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={`${inter.variable} ${roboto_mono.variable}`}>
      <Component {...pageProps} />
    </main>
  )
}
```

```css
/* styles/globals.css */
@import 'tailwindcss';

@theme inline {
  --font-sans: var(--font-inter);
  --font-mono: var(--font-roboto-mono);
}
```

### Tailwind v3

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)'],
        mono: ['var(--font-roboto-mono)'],
      },
    },
  },
}
```

---

## 스타일 적용 방법

### 1. className 사용

```tsx
<p className={inter.className}>Hello World</p>
```

### 2. style 객체 사용

```tsx
<p style={inter.style}>Hello World</p>
```

### 3. CSS 변수 사용

```tsx
<div className={inter.variable}>
  <p className="font-sans">Hello World</p>
</div>
```

---

## `<head>`에 폰트 추가

특정 페이지의 `<head>`에 폰트를 추가할 수도 있습니다:

```tsx
// pages/index.tsx
import { Inter } from 'next/font/google'
import Head from 'next/head'

const inter = Inter({ subsets: ['latin'] })

export default function Home() {
  return (
    <>
      <Head>
        <style jsx global>{`
          html {
            font-family: ${inter.style.fontFamily};
          }
        `}</style>
      </Head>
      <main>
        <p>Hello World</p>
      </main>
    </>
  )
}
```

---

## 프리로딩

폰트는 호출되는 위치에 따라 자동으로 프리로드됩니다:

- **페이지**: 해당 페이지의 라우트에만 프리로드
- **_app.js**: 모든 라우트에 프리로드
- **_document.js**: 모든 라우트에 프리로드

---

## 폰트 이름 규칙

Google Fonts를 import할 때:

```tsx
// ✅ 올바름
import { Roboto_Mono } from 'next/font/google'

// ❌ 잘못됨
import { Roboto Mono } from 'next/font/google'
```

**규칙:**
- 다중 단어 폰트명은 언더스코어(`_`) 사용
- 하이픈(`-`)은 사용하지 않음

---

## 중요한 주의사항

> **Good to know**:
> * 변수 폰트 사용을 권장합니다 (성능 최적화)
> * `preload: true`이고 `subsets`를 지정하지 않으면 경고가 발생합니다
> * 다중 폰트는 보수적으로 사용하세요 (성능 영향)
> * 폰트 정의 파일을 중앙화하여 관리하는 것이 좋습니다

> **성능 팁**:
> * 필요한 서브셋만 포함
> * 변수 폰트 사용으로 파일 크기 감소
> * `display: 'swap'` 사용 (기본값)

---

## Pages Router vs App Router 차이점

Pages Router에서 폰트를 사용할 때는 주로 다음 위치에서 적용합니다:

1. **`pages/_app.tsx`**: 전역 폰트 적용
2. **개별 페이지**: 특정 페이지에만 적용
3. **`pages/_document.tsx`**: HTML 문서 레벨에서 적용

App Router와 달리 `<html>` 태그에 직접 className을 적용할 수 없으므로 `_document.tsx` 또는 CSS 변수를 사용해야 합니다.

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.2.0 | `@next/font`가 `next/font`로 이름 변경. 설치 불필요 |
| v13.0.0 | `@next/font` 추가 |

---

## 관련 문서

- [Font Optimization Guide](../../getting-started/10-font-optimization.md)
- [Script Component](./script.md)
- [Image Optimization](../../getting-started/09-image-optimization.md)
