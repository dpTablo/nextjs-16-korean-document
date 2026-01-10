# Tailwind CSS v3

Next.js 애플리케이션에 Tailwind CSS v3를 설치하고 구성하는 방법을 알아봅니다.

> **참고:** Tailwind 4를 사용하는 경우, [공식 Tailwind CSS 설정 가이드](https://tailwindcss.com/docs/installation)를 참고하세요.

---

## 설치 단계

### 1. 의존성 설치

패키지 매니저를 선택하고 적절한 명령어를 실행하세요:

```bash
# pnpm
pnpm add -D tailwindcss@^3 postcss autoprefixer
npx tailwindcss init -p

# npm
npm install -D tailwindcss@^3 postcss autoprefixer
npx tailwindcss init -p

# yarn
yarn add -D tailwindcss@^3 postcss autoprefixer
npx tailwindcss init -p

# bun
bun add -D tailwindcss@^3 postcss autoprefixer
bunx tailwindcss init -p
```

---

## 설정

### 2. 템플릿 경로 구성

`tailwind.config.js` 파일에 템플릿 경로를 추가하세요:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

> **알아두면 좋은 점:**
> - `content` 배열에는 Tailwind 클래스를 사용하는 모든 파일의 경로를 포함해야 합니다.
> - App Router를 사용하는 경우 `./app/**/*.{js,ts,jsx,tsx,mdx}` 경로가 필수입니다.
> - Pages Router를 함께 사용하는 경우 `./pages/**/*.{js,ts,jsx,tsx,mdx}` 경로도 포함하세요.

### 3. Tailwind 지시어 추가

`app/globals.css` 파일을 생성하거나 업데이트하여 Tailwind 지시어를 추가하세요:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 4. 전역 CSS 가져오기

루트 레이아웃(`app/layout.tsx` 또는 `app/layout.js`)에서 전역 CSS를 가져오세요:

**app/layout.tsx**
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

---

## Tailwind 클래스 사용하기

설치가 완료되면 컴포넌트에서 Tailwind 유틸리티 클래스를 사용할 수 있습니다:

**app/page.tsx**
```tsx
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <h1 className="text-4xl font-bold text-blue-600">
        Next.js에 오신 것을 환영합니다!
      </h1>
      <p className="text-lg text-gray-600">
        Tailwind CSS로 스타일링된 페이지입니다.
      </p>
    </main>
  )
}
```

---

## Turbopack 지원

Next.js 13.1 이상부터 Tailwind CSS와 PostCSS는 Turbopack에서 완전히 지원됩니다.

**개발 서버에서 Turbopack 사용:**
```bash
next dev --turbopack
```

---

## CSS Modules와 함께 사용

Tailwind CSS는 CSS Modules와 함께 사용할 수 있습니다:

**styles.module.css**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

.custom-container {
  @apply max-w-7xl mx-auto px-4 sm:px-6 lg:px-8;
}
```

**page.tsx**
```tsx
import styles from './styles.module.css'

export default function Page() {
  return (
    <div className={styles.customContainer}>
      <h1 className="text-3xl font-bold">커스텀 컨테이너</h1>
    </div>
  )
}
```

---

## 커스텀 설정

### 색상 팔레트 확장

**tailwind.config.js**
```js
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        'brand-blue': '#1E40AF',
        'brand-purple': '#7C3AED',
      },
    },
  },
  plugins: [],
}
```

### 커스텀 폰트 추가

```js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-geist-sans)'],
        mono: ['var(--font-geist-mono)'],
      },
    },
  },
}
```

---

## 베스트 프랙티스

### 1. **클래스 이름 정렬**
일관성을 위해 클래스 이름을 정렬하는 Prettier 플러그인을 사용하세요:

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

**.prettierrc**
```json
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

### 2. **컴포넌트 추출**
반복되는 스타일은 재사용 가능한 컴포넌트로 추출하세요:

```tsx
// components/Button.tsx
export function Button({ children }: { children: React.ReactNode }) {
  return (
    <button className="rounded-lg bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">
      {children}
    </button>
  )
}
```

### 3. **`@apply` 사용 자제**
가능하면 유틸리티 클래스를 직접 사용하고, `@apply`는 정말 필요한 경우에만 사용하세요.

---

## 프로덕션 최적화

Tailwind CSS는 프로덕션 빌드 시 사용하지 않는 CSS를 자동으로 제거합니다 (PurgeCSS).

**빌드 확인:**
```bash
npm run build
```

빌드 후 `.next/static/css` 폴더를 확인하여 최적화된 CSS 파일을 볼 수 있습니다.

---

## 문제 해결

### 스타일이 적용되지 않는 경우

1. **`content` 경로 확인:** `tailwind.config.js`의 `content` 배열에 모든 파일 경로가 포함되어 있는지 확인하세요.

2. **전역 CSS 가져오기 확인:** 루트 레이아웃에서 `globals.css`를 가져오고 있는지 확인하세요.

3. **개발 서버 재시작:** 구성 파일을 변경한 후에는 개발 서버를 재시작하세요.

4. **캐시 삭제:**
   ```bash
   rm -rf .next
   npm run dev
   ```

---

## 다음 단계

- [CSS Styling](https://nextjs.org/docs/app/guides/styling) - 다양한 CSS 스타일링 방법
- [Sass](./sass.md) - Sass 사용 가이드
- [CSS-in-JS](./css-in-js.md) - CSS-in-JS 라이브러리 사용

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
