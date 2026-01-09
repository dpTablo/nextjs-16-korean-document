# 폰트 최적화

**버전:** 16.1.1
**최종 업데이트:** 2025-06-11

## 개요

`next/font` 모듈은 폰트를 자동으로 최적화하고 개선된 프라이버시 및 성능을 위해 외부 네트워크 요청을 제거합니다. 모든 폰트 파일에 대한 **내장 셀프 호스팅**을 포함하여 레이아웃 시프트 없이 최적의 웹 폰트 로딩을 가능하게 합니다.

## 시작하기

`next/font/local` 또는 `next/font/google`에서 import하고, 옵션과 함께 함수로 호출한 다음 `className`을 설정하세요:

**TypeScript 예시 (app/layout.tsx):**
```tsx
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko" className={geist.className}>
      <body>{children}</body>
    </html>
  )
}
```

**JavaScript 예시 (app/layout.js):**
```jsx
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})

export default function Layout({ children }) {
  return (
    <html className={geist.className}>
      <body>{children}</body>
    </html>
  )
}
```

**중요 참고:** 폰트는 컴포넌트로 스코핑됩니다. 애플리케이션 전체에 폰트를 적용하려면 루트 레이아웃을 통해 적용하세요.

---

## Google Fonts

모든 Google Font를 자동으로 셀프 호스팅하여 도메인에서 제공되는 정적 에셋으로 사용하며, Google로 요청이 전송되지 않습니다.

**가변 폰트 예시:**
```tsx
import { Geist } from 'next/font/google'

const geist = Geist({
  subsets: ['latin'],
})
```

**비가변 폰트 (weight 지정 필요):**
```tsx
import { Roboto } from 'next/font/google'

const roboto = Roboto({
  weight: '400',
  subsets: ['latin'],
})
```

**권장사항:** 최상의 성능과 유연성을 위해 가변 폰트를 사용하세요.

---

## 로컬 폰트

`next/font/local`에서 import하고 `src` 경로를 지정하세요. 폰트를 `public` 폴더에 저장하거나 `app` 폴더에 함께 배치하세요.

**단일 폰트 파일:**
```tsx
import localFont from 'next/font/local'

const myFont = localFont({
  src: './my-font.woff2',
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko" className={myFont.className}>
      <body>{children}</body>
    </html>
  )
}
```

**여러 폰트 파일 (배열):**
```js
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

---

## API 참조

- **Font** (`/docs/app/api-reference/components/font.md`) - 내장 `next/font` 로더로 웹 폰트 로딩 최적화
