---
원문: https://nextjs.org/docs/app/getting-started/css
버전: 16.1.6
---

# CSS 스타일링

## 개요
Next.js는 여러 스타일링 방법을 제공합니다:
- Tailwind CSS
- CSS Modules
- Global CSS
- External Stylesheets
- Sass
- CSS-in-JS

---

## Tailwind CSS

커스텀 디자인을 구축하기 위한 유틸리티 우선 CSS 프레임워크입니다.

### 설치

**pnpm:**
```bash
pnpm add -D tailwindcss @tailwindcss/postcss
```

**npm:**
```bash
npm install -D tailwindcss @tailwindcss/postcss
```

**yarn:**
```bash
yarn add -D tailwindcss @tailwindcss/postcss
```

**bun:**
```bash
bun add -D tailwindcss @tailwindcss/postcss
```

### 설정

**1. PostCSS 설정** (`postcss.config.mjs`):
```js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

**2. 전역 CSS에서 import** (`app/globals.css`):
```css
@import 'tailwindcss';
```

**3. 루트 레이아웃에서 import** (`app/layout.tsx`):
```tsx
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

**4. 유틸리티 클래스 사용** (`app/page.tsx`):
```tsx
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <h1 className="text-4xl font-bold">Next.js에 오신 것을 환영합니다!</h1>
    </main>
  )
}
```

> **참고:** 구형 브라우저에서 더 넓은 브라우저 지원을 위해서는 Tailwind CSS v3 설정 지침을 참조하세요.

---

## CSS Modules

고유한 클래스 이름을 생성하여 CSS를 로컬로 스코핑하며 이름 충돌을 방지합니다.

### 사용법

**1. CSS Module 생성** (`app/blog/blog.module.css`):
```css
.blog {
  padding: 24px;
}
```

**2. Import 및 사용** (`app/blog/page.tsx`):
```tsx
import styles from './blog.module.css'

export default function Page() {
  return <main className={styles.blog}></main>
}
```

---

## Global CSS

전체 애플리케이션에 스타일을 적용합니다.

### 설정

**1. 전역 스타일시트 생성** (`app/global.css`):
```css
body {
  padding: 20px 20px 60px;
  max-width: 680px;
  margin: 0 auto;
}
```

**2. 루트 레이아웃에서 import** (`app/layout.tsx`):
```tsx
import './global.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

> **권장사항:** 진정으로 전역적인 스타일에는 Global CSS를, 컴포넌트 스타일링에는 Tailwind CSS를, 커스텀 스코프 CSS에는 CSS Modules를 사용하세요.

---

## 외부 스타일시트

`app` 디렉토리 어디서나 외부 패키지의 스타일시트를 import할 수 있습니다.

### 예시

```tsx
import 'bootstrap/dist/css/bootstrap.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body className="container">{children}</body>
    </html>
  )
}
```

> **참고:** React 19에서는 `<link rel="stylesheet" href="..." />`도 사용할 수 있습니다.

---

## CSS 순서 및 병합

Next.js는 프로덕션 빌드 중에 스타일시트를 자동으로 병합하고 최적화합니다. **CSS 순서는 코드의 import 순서에 따라 달라집니다**.

### 예시
```tsx
import { BaseButton } from './base-button'
import styles from './page.module.css'

export default function Page() {
  return <BaseButton className={styles.primary} />
}
```

`<BaseButton>`이 먼저 import되므로 `base-button.module.css`가 `page.module.css`보다 먼저 순서가 지정됩니다.

### 모범 사례

- CSS imports를 단일 JavaScript/TypeScript 진입 파일에 포함
- 전역 스타일 및 Tailwind를 애플리케이션 루트에서 import
- **대부분의 스타일링 요구사항에는 Tailwind CSS 사용**
- Tailwind가 충분하지 않을 때 컴포넌트 특정 스타일에는 CSS Modules 사용
- 일관된 명명 규칙 사용 (예: `<name>.module.css`)
- 공유 스타일을 공유 컴포넌트로 추출
- 린터의 자동 정렬 import 비활성화 (예: ESLint의 `sort-imports`)
- 고급 CSS 제어를 위해 `next.config.js`의 `cssChunking` 옵션 사용

---

## 개발 vs 프로덕션

| 측면 | 개발 | 프로덕션 |
|--------|-------------|-----------|
| **CSS 업데이트** | Fast Refresh로 즉시 적용 | N/A |
| **출력** | N/A | 최소화되고 코드 분할된 `.css` 파일로 연결 |
| **로딩** | Fast Refresh를 위해 JavaScript 필요 | JavaScript가 비활성화되어도 작동 |
| **성능** | N/A | 라우트당 최소한의 CSS만 로드 |

> **팁:** 최종 CSS 순서를 확인하려면 항상 프로덕션 빌드(`next build`)를 확인하세요.

---

## 다음 단계

- [Tailwind CSS v3](/docs/app/guides/tailwind-v3-css.md) - 더 넓은 브라우저 지원
- [Sass](/docs/app/guides/sass.md) - Sass 스타일링
- [CSS-in-JS](/docs/app/guides/css-in-js.md) - CSS-in-JS 라이브러리
