# Tailwind CSS

[Tailwind CSS](https://tailwindcss.com/)는 유틸리티 우선 CSS 프레임워크로, Next.js와 완벽하게 호환됩니다.

## 설치

Tailwind CSS와 PostCSS 플러그인을 설치합니다:

```bash
# npm
npm install -D tailwindcss @tailwindcss/postcss

# yarn
yarn add -D tailwindcss @tailwindcss/postcss

# pnpm
pnpm add -D tailwindcss @tailwindcss/postcss

# bun
bun add -D tailwindcss @tailwindcss/postcss
```

## PostCSS 설정

프로젝트 루트에 `postcss.config.mjs` 파일을 생성합니다:

```js
// postcss.config.mjs
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

## CSS 파일 설정

전역 스타일시트에 Tailwind를 import합니다:

```css
/* styles/globals.css */
@import 'tailwindcss';
```

## _app.js에서 Import

`pages/_app.js`에서 전역 스타일시트를 import합니다:

```jsx
// pages/_app.js
import '@/styles/globals.css'

export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

## 사용 예제

이제 Tailwind의 유틸리티 클래스를 사용할 수 있습니다:

```tsx
// pages/index.tsx
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <h1 className="text-4xl font-bold text-gray-900">
        Next.js에 오신 것을 환영합니다!
      </h1>
      <p className="mt-4 text-lg text-gray-600">
        Tailwind CSS로 스타일링된 페이지입니다.
      </p>
    </main>
  )
}
```

## 반응형 디자인

Tailwind의 반응형 접두사를 사용하여 다양한 화면 크기에 대응할 수 있습니다:

```tsx
export default function ResponsiveComponent() {
  return (
    <div className="p-4 sm:p-6 md:p-8 lg:p-12">
      <h1 className="text-xl sm:text-2xl md:text-3xl lg:text-4xl font-bold">
        반응형 제목
      </h1>
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 mt-8">
        <div className="bg-blue-100 p-4 rounded">카드 1</div>
        <div className="bg-blue-100 p-4 rounded">카드 2</div>
        <div className="bg-blue-100 p-4 rounded">카드 3</div>
      </div>
    </div>
  )
}
```

## 다크 모드

Tailwind의 `dark:` 접두사를 사용하여 다크 모드를 지원할 수 있습니다:

```tsx
export default function DarkModeExample() {
  return (
    <div className="bg-white dark:bg-gray-900 min-h-screen">
      <h1 className="text-gray-900 dark:text-white text-2xl font-bold">
        다크 모드 지원
      </h1>
      <p className="text-gray-600 dark:text-gray-300">
        시스템 설정에 따라 자동으로 테마가 변경됩니다.
      </p>
    </div>
  )
}
```

## 커스텀 설정

`tailwind.config.js` 파일을 생성하여 Tailwind를 커스터마이징할 수 있습니다:

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: '#0070f3',
        secondary: '#1a1a2e',
      },
      fontFamily: {
        sans: ['Pretendard', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

## CSS Modules와 함께 사용

Tailwind를 CSS Modules와 함께 사용할 수 있습니다:

```css
/* components/Button.module.css */
.button {
  @apply px-4 py-2 rounded font-semibold;
}

.primary {
  @apply bg-blue-500 text-white hover:bg-blue-600;
}

.secondary {
  @apply bg-gray-200 text-gray-800 hover:bg-gray-300;
}
```

```tsx
// components/Button.tsx
import styles from './Button.module.css'

export function Button({ variant = 'primary', children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  )
}
```

## 플러그인 사용

Tailwind 공식 플러그인을 사용할 수 있습니다:

```bash
npm install -D @tailwindcss/typography @tailwindcss/forms
```

```js
// tailwind.config.js
module.exports = {
  // ...
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
}
```

## 권장 사항

1. **Tailwind를 기본 스타일링 도구로 사용**: 대부분의 스타일링에 Tailwind 유틸리티 클래스를 사용하세요.

2. **복잡한 스타일은 CSS Modules로**: Tailwind만으로 표현하기 어려운 복잡한 스타일은 CSS Modules를 사용하세요.

3. **일관된 디자인 토큰 사용**: `tailwind.config.js`에서 색상, 간격, 폰트 등을 정의하여 일관성을 유지하세요.

4. **PurgeCSS 자동 적용**: 프로덕션 빌드 시 사용하지 않는 CSS가 자동으로 제거됩니다.

## 성능 최적화

Next.js는 프로덕션 빌드 시 자동으로 CSS를 최적화합니다:

- 사용하지 않는 CSS 제거
- CSS 파일 최소화
- CSS 파일 청킹 및 코드 분할
