---
원문: https://nextjs.org/docs/app/guides/sass
버전: 16.1.6
---

# Sass

Next.js에서 Sass를 사용하여 스타일을 작성하는 방법을 알아봅니다.

## 개요

Next.js는 `.scss` 및 `.sass` 파일 확장자를 사용하는 Sass를 기본적으로 지원합니다. CSS Modules를 통해 컴포넌트 레벨 Sass를 사용하려면 `.module.scss` 또는 `.module.sass` 확장자를 사용하세요.

---

## 설치

Sass 사용을 시작하기 전에 `sass` 패키지를 설치해야 합니다:

```bash
npm install --save-dev sass
```

또는

```bash
yarn add --dev sass
# 또는
pnpm add --save-dev sass
```

---

## 구문 옵션

Sass는 두 가지 구문을 지원합니다:

### SCSS (.scss)

CSS의 상위 집합으로, 모든 CSS는 유효한 SCSS입니다. 초보자에게 권장됩니다.

**styles.scss**
```scss
$primary-color: #0070f3;
$secondary-color: #7928ca;

.button {
  background-color: $primary-color;
  color: white;
  padding: 1rem 2rem;
  border-radius: 4px;

  &:hover {
    background-color: darken($primary-color, 10%);
  }

  &.secondary {
    background-color: $secondary-color;
  }
}
```

### Sass (.sass)

들여쓰기 구문을 사용하며, 중괄호와 세미콜론이 없습니다.

**styles.sass**
```sass
$primary-color: #0070f3
$secondary-color: #7928ca

.button
  background-color: $primary-color
  color: white
  padding: 1rem 2rem
  border-radius: 4px

  &:hover
    background-color: darken($primary-color, 10%)

  &.secondary
    background-color: $secondary-color
```

> **권장:** SCSS 구문 사용 - CSS와 더 유사하고 학습 곡선이 완만합니다.

---

## 사용 방법

### 전역 스타일시트

**app/globals.scss**
```scss
$font-stack: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
$primary-color: #0070f3;

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: $font-stack;
  color: #333;
  background: #fff;
}

a {
  color: $primary-color;
  text-decoration: none;

  &:hover {
    text-decoration: underline;
  }
}
```

**app/layout.tsx**
```tsx
import './globals.scss'

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

### CSS Modules와 함께 Sass 사용

컴포넌트별로 스타일을 격리할 수 있습니다.

**components/Button.module.scss**
```scss
$primary: #0070f3;
$secondary: #7928ca;

.button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  transition: all 0.2s ease;

  &Primary {
    @extend .button;
    background: $primary;
    color: white;

    &:hover {
      background: darken($primary, 10%);
    }
  }

  &Secondary {
    @extend .button;
    background: transparent;
    color: $primary;
    border: 2px solid $primary;

    &:hover {
      background: $primary;
      color: white;
    }
  }
}
```

**components/Button.tsx**
```tsx
import styles from './Button.module.scss'

export function Button({
  variant = 'primary',
  children,
}: {
  variant?: 'primary' | 'secondary'
  children: React.ReactNode
}) {
  const className =
    variant === 'primary' ? styles.buttonPrimary : styles.buttonSecondary

  return <button className={className}>{children}</button>
}
```

---

## 구성

### 기본 설정

`next.config.ts` 또는 `next.config.js`에서 Sass 옵션을 구성할 수 있습니다.

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  sassOptions: {
    additionalData: `$var: red;`,
  },
}

export default nextConfig
```

### Sass 구현 지정

기본 `sass` 패키지 대신 대체 구현을 사용할 수 있습니다:

```bash
npm install --save-dev sass-embedded
```

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  sassOptions: {
    implementation: 'sass-embedded',
  },
}

export default nextConfig
```

### Include 경로 설정

Sass 파일을 가져올 때 상대 경로 대신 절대 경로를 사용할 수 있습니다:

**next.config.ts**
```ts
import path from 'path'
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  sassOptions: {
    includePaths: [path.join(__dirname, 'styles')],
  },
}

export default nextConfig
```

**사용:**
```scss
// 이전: ../../styles/variables.scss
@import 'variables';

// 이후: styles/variables.scss로 바로 접근
```

---

## Sass 변수를 JavaScript로 내보내기

CSS Module 파일에서 Sass 변수를 JavaScript로 내보낼 수 있습니다.

**app/variables.module.scss**
```scss
$primary-color: #64ff00;
$secondary-color: #7928ca;
$font-size-base: 16px;
$spacing-unit: 8px;

:export {
  primaryColor: $primary-color;
  secondaryColor: $secondary-color;
  fontSizeBase: $font-size-base;
  spacingUnit: $spacing-unit;
}
```

**app/page.tsx**
```tsx
import variables from './variables.module.scss'

export default function Page() {
  return (
    <div>
      <h1 style={{ color: variables.primaryColor }}>
        Hello, Next.js!
      </h1>
      <p style={{
        color: variables.secondaryColor,
        fontSize: variables.fontSizeBase,
        marginBottom: variables.spacingUnit,
      }}>
        Sass 변수를 JavaScript에서 사용합니다.
      </p>
    </div>
  )
}
```

---

## 고급 기능

### Mixins

**styles/_mixins.scss**
```scss
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

@mixin button-variant($bg-color) {
  background: $bg-color;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background: darken($bg-color, 10%);
  }
}

@mixin responsive($breakpoint) {
  @if $breakpoint == mobile {
    @media (max-width: 767px) {
      @content;
    }
  } @else if $breakpoint == tablet {
    @media (min-width: 768px) and (max-width: 1023px) {
      @content;
    }
  } @else if $breakpoint == desktop {
    @media (min-width: 1024px) {
      @content;
    }
  }
}
```

**components/Card.module.scss**
```scss
@import '../styles/mixins';

.card {
  @include flex-center;
  padding: 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

  @include responsive(mobile) {
    padding: 1rem;
  }

  @include responsive(desktop) {
    padding: 3rem;
  }
}

.button {
  @include button-variant(#0070f3);
}
```

### 중첩 및 선택자

```scss
.nav {
  background: #333;
  padding: 1rem;

  ul {
    list-style: none;
    display: flex;
    gap: 1rem;

    li {
      a {
        color: white;
        text-decoration: none;
        padding: 0.5rem 1rem;

        &:hover {
          background: rgba(255, 255, 255, 0.1);
        }

        &.active {
          background: rgba(255, 255, 255, 0.2);
        }
      }
    }
  }
}
```

### 함수

**styles/_functions.scss**
```scss
@function rem($pixels, $base: 16) {
  @return #{$pixels / $base}rem;
}

@function color-contrast($color) {
  $lightness: lightness($color);

  @if $lightness > 50% {
    @return #000;
  } @else {
    @return #fff;
  }
}
```

**사용:**
```scss
@import './functions';

.heading {
  font-size: rem(24); // 1.5rem
  margin-bottom: rem(16); // 1rem
}

.button {
  background: #0070f3;
  color: color-contrast(#0070f3); // white
}
```

---

## 파일 구조 권장사항

```
my-app/
├── app/
│   ├── globals.scss
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── Button.module.scss
│   ├── Button.tsx
│   ├── Card.module.scss
│   └── Card.tsx
├── styles/
│   ├── _variables.scss
│   ├── _mixins.scss
│   ├── _functions.scss
│   └── _breakpoints.scss
└── next.config.ts
```

### 변수 파일

**styles/_variables.scss**
```scss
// Colors
$primary: #0070f3;
$secondary: #7928ca;
$success: #10b981;
$warning: #f59e0b;
$error: #ef4444;
$gray-100: #f3f4f6;
$gray-900: #111827;

// Typography
$font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
$font-size-base: 16px;
$font-size-sm: 14px;
$font-size-lg: 18px;

// Spacing
$spacing-xs: 0.25rem;
$spacing-sm: 0.5rem;
$spacing-md: 1rem;
$spacing-lg: 1.5rem;
$spacing-xl: 2rem;

// Borders
$border-radius: 4px;
$border-radius-lg: 8px;
```

### Breakpoints 파일

**styles/_breakpoints.scss**
```scss
$breakpoints: (
  'mobile': 767px,
  'tablet': 1023px,
  'desktop': 1024px,
);

@mixin respond-to($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      @content;
    }
  }
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **변수 사용** - 색상, 폰트 크기 등을 변수로 정의
2. **Mixins 활용** - 재사용 가능한 스타일 패턴
3. **중첩 제한** - 3단계 이하로 유지
4. **파일 분할** - 변수, 믹스인, 함수를 별도 파일로
5. **CSS Modules 사용** - 스타일 격리

### ❌ 피해야 할 것

1. **과도한 중첩** - 유지보수 어려움
2. **너무 많은 @extend** - 컴파일 시간 증가
3. **전역 스타일 남용** - 스타일 충돌 가능
4. **인라인 스타일과 혼용** - 일관성 부족

---

## 마이그레이션

### CSS에서 Sass로

기존 CSS 파일을 Sass로 쉽게 마이그레이션할 수 있습니다:

**이전 (styles.css):**
```css
.button {
  background: #0070f3;
  color: white;
  padding: 1rem 2rem;
}

.button:hover {
  background: #0051cc;
}
```

**이후 (styles.scss):**
```scss
$primary: #0070f3;

.button {
  background: $primary;
  color: white;
  padding: 1rem 2rem;

  &:hover {
    background: darken($primary, 10%);
  }
}
```

---

## 성능 최적화

### 1. 불필요한 스타일 제거

```scss
// ❌ 사용하지 않는 스타일
.unused-class {
  color: red;
}

// ✅ 실제 사용하는 스타일만
.button {
  color: blue;
}
```

### 2. CSS Modules 사용

전역 스타일 대신 CSS Modules를 사용하여 번들 크기를 줄입니다.

### 3. @import 대신 @use 사용 (Sass 최신 버전)

```scss
// ❌ 구식
@import 'variables';

// ✅ 최신
@use 'variables' as *;
```

---

## 문제 해결

### Sass가 설치되지 않음

```bash
# 오류: Cannot find module 'sass'
npm install --save-dev sass
```

### 경로 오류

```scss
// ❌ 잘못된 경로
@import '../../styles/variables';

// ✅ includePaths 설정 후
@import 'variables';
```

**next.config.ts 설정:**
```ts
import path from 'path'

const nextConfig = {
  sassOptions: {
    includePaths: [path.join(__dirname, 'styles')],
  },
}
```

---

## 다음 단계

- [Tailwind CSS](./tailwind-css.md) - Tailwind CSS 가이드
- [CSS-in-JS](./css-in-js.md) - CSS-in-JS 라이브러리
- [CSS Modules](https://nextjs.org/docs/app/building-your-application/styling/css-modules)

---
