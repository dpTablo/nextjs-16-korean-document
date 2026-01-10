# generateViewport

## 개요
`generateViewport`는 Next.js에서 페이지의 초기 뷰포트를 커스터마이즈할 수 있는 동적 함수입니다. `params` 및 `searchParams`와 같은 런타임 정보에 접근하여 뷰포트 구성을 반환할 수 있습니다.

---

## 주요 제약 사항
- **Server Components만**: 내보내기는 Server Components에서만 지원됩니다
- **상호 배타적**: 동일한 라우트 세그먼트에서 `viewport` 객체와 `generateViewport` 함수를 모두 내보낼 수 없습니다
- **정적 선호**: 뷰포트가 런타임 정보에 의존하지 않는 경우 정적 `viewport` 객체를 사용하세요

---

## 기본 사용법

```tsx
// layout.tsx | page.tsx
export function generateViewport({ params }) {
  return {
    themeColor: '...',
  }
}
```

---

## Viewport 필드

### `themeColor`
브라우저의 테마 색상을 설정합니다. 단순 값 또는 미디어 쿼리를 지원합니다:

```tsx
// 단순 값
export const viewport = {
  themeColor: 'black',
}

// 미디어 쿼리 사용
export const viewport = {
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: 'cyan' },
    { media: '(prefers-color-scheme: dark)', color: 'black' },
  ],
}
```

생성되는 HTML:
```html
<meta name="theme-color" media="(prefers-color-scheme: light)" content="cyan">
<meta name="theme-color" media="(prefers-color-scheme: dark)" content="black">
```

### `width`, `initialScale`, `maximumScale`, `userScalable`
표준 뷰포트 메타 태그 구성:

```tsx
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  maximumScale: 1,
  userScalable: false,
}
```

생성되는 HTML:
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

### `colorScheme`
색 구성표 선호도를 지정합니다:

```tsx
export const viewport = {
  colorScheme: 'dark',
}
```

생성되는 HTML:
```html
<meta name="color-scheme" content="dark">
```

---

## 타입 안정성

### TypeScript
```tsx
import type { Viewport } from 'next'

export function generateViewport({ params }): Viewport {
  return {
    themeColor: 'black',
  }
}
```

### JavaScript (JSDoc)
```js
/** @type {import("next").Viewport} */
export const viewport = {
  themeColor: 'black',
}
```

---

## 실전 예제

### 정적 Viewport (권장)

```tsx
// app/layout.tsx
import type { Viewport } from 'next'

export const viewport: Viewport = {
  width: 'device-width',
  initialScale: 1,
  maximumScale: 5,
  userScalable: true,
  themeColor: '#000000',
}

export default function RootLayout({ children }) {
  return (
    <html>
      <body>{children}</body>
    </html>
  )
}
```

### 동적 Viewport (테마 색상)

```tsx
// app/layout.tsx
import type { Viewport } from 'next'

export function generateViewport(): Viewport {
  return {
    themeColor: [
      { media: '(prefers-color-scheme: light)', color: '#ffffff' },
      { media: '(prefers-color-scheme: dark)', color: '#000000' },
    ],
  }
}
```

### 매개변수 기반 Viewport

```tsx
// app/[locale]/layout.tsx
export function generateViewport({ params }) {
  const locale = params.locale

  // 로케일에 따라 다른 색상
  const themeColors = {
    en: '#0070f3',
    ko: '#ff6b6b',
    ja: '#51cf66',
  }

  return {
    themeColor: themeColors[locale] || '#0070f3',
    width: 'device-width',
    initialScale: 1,
  }
}
```

### 다크 모드 지원

```tsx
// app/layout.tsx
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: 'white' },
    { media: '(prefers-color-scheme: dark)', color: 'black' },
  ],
  colorScheme: 'dark light',
}
```

### 모바일 최적화

```tsx
// app/layout.tsx
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  maximumScale: 1,
  userScalable: false, // 모바일에서 확대/축소 방지
  viewportFit: 'cover', // 노치 영역 포함
}
```

### PWA 설정

```tsx
// app/layout.tsx
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  minimumScale: 1,
  maximumScale: 5,
  userScalable: true,
  themeColor: '#2196f3',
  colorScheme: 'light',
}
```

---

## Cache Components와 함께 사용

### 외부 데이터 사용 (권장)

```tsx
export async function generateViewport() {
  'use cache'
  const { width, initialScale } = await db.query('viewport-size')
  return { width, initialScale }
}
```

### 런타임 데이터 사용 (Suspense 필요)

```tsx
import { Suspense } from 'react'
import { cookies } from 'next/headers'

export async function generateViewport() {
  const cookieJar = await cookies()
  return {
    themeColor: cookieJar.get('theme-color')?.value || '#000000',
  }
}

export default function RootLayout({ children }) {
  return (
    <Suspense>
      <html>
        <body>{children}</body>
      </html>
    </Suspense>
  )
}
```

---

## 모든 Viewport 옵션

```tsx
import type { Viewport } from 'next'

export const viewport: Viewport = {
  // 뷰포트 크기
  width: 'device-width',
  height: 'device-height',

  // 확대/축소 설정
  initialScale: 1,
  minimumScale: 1,
  maximumScale: 5,
  userScalable: true,

  // 테마 색상
  themeColor: '#ffffff',
  // 또는 미디어 쿼리로
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: 'white' },
    { media: '(prefers-color-scheme: dark)', color: 'black' },
  ],

  // 색 구성표
  colorScheme: 'light dark',

  // iOS 특정
  viewportFit: 'cover', // 'auto' | 'contain' | 'cover'
}
```

---

## 모범 사례

### 1. 정적 값은 viewport 객체 사용

```tsx
// ✅ 좋은 예 - 정적 값
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  themeColor: '#000000',
}

// ❌ 나쁜 예 - 불필요한 함수
export function generateViewport() {
  return {
    width: 'device-width',
    initialScale: 1,
  }
}
```

### 2. 접근성 고려

```tsx
// ✅ 좋은 예 - 사용자 확대/축소 허용
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  userScalable: true, // 접근성을 위해 허용
}

// ❌ 나쁜 예 - 확대/축소 차단
export const viewport = {
  userScalable: false, // 접근성 문제
  maximumScale: 1,
}
```

### 3. 다크 모드 지원

```tsx
// ✅ 좋은 예 - 다크 모드 지원
export const viewport = {
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: '#ffffff' },
    { media: '(prefers-color-scheme: dark)', color: '#1a1a1a' },
  ],
  colorScheme: 'light dark',
}
```

### 4. 모바일 최적화

```tsx
// ✅ 좋은 예 - 모바일 친화적
export const viewport = {
  width: 'device-width',
  initialScale: 1,
  viewportFit: 'cover', // iOS 노치 대응
  themeColor: '#ffffff',
}
```

---

## 주의사항

### 1. 중복 내보내기 방지

```tsx
// ❌ 오류 - 둘 다 내보낼 수 없음
export const viewport = { ... }
export function generateViewport() { ... }

// ✅ 정적 값
export const viewport = { ... }

// ✅ 또는 동적 함수
export function generateViewport() { ... }
```

### 2. Server Components만 지원

```tsx
// ❌ Client Component에서 사용 불가
'use client'
export const viewport = { ... } // 오류!

// ✅ Server Component (layout 또는 page)
export const viewport = { ... }
```

---

## 버전 히스토리
- **v14.0.0**: `viewport` 및 `generateViewport` 도입

---

## 관련 문서

- [generateMetadata](./generateMetadata.md)
- [Metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
- [Layout](../file-conventions/layout.md)
