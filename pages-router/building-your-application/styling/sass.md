# Sass

Next.js는 `.scss`와 `.sass` 확장자를 기본으로 지원합니다. CSS Modules를 통해 컴포넌트 수준의 Sass를 사용할 수 있습니다 (`.module.scss` 또는 `.module.sass` 확장자).

## 설치

먼저 `sass` 패키지를 설치합니다:

```bash
npm install --save-dev sass
```

## Sass 문법 선택

Next.js는 두 가지 Sass 문법을 모두 지원합니다:

- **`.scss` 확장자**: SCSS 문법 사용 (CSS의 상위집합, 권장)
- **`.sass` 확장자**: Indented Syntax 사용

### SCSS 예제 (권장)

```scss
// styles/main.scss
$primary-color: #0070f3;
$border-radius: 8px;

.container {
  padding: 20px;

  .title {
    color: $primary-color;
    font-size: 2rem;
  }

  .button {
    background-color: $primary-color;
    border-radius: $border-radius;
    color: white;
    padding: 10px 20px;

    &:hover {
      opacity: 0.8;
    }
  }
}
```

### Sass (Indented Syntax) 예제

```sass
// styles/main.sass
$primary-color: #0070f3
$border-radius: 8px

.container
  padding: 20px

  .title
    color: $primary-color
    font-size: 2rem

  .button
    background-color: $primary-color
    border-radius: $border-radius
    color: white
    padding: 10px 20px

    &:hover
      opacity: 0.8
```

## CSS Modules와 함께 사용

```scss
/* styles/Button.module.scss */
$primary: #0070f3;
$secondary: #666;

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s ease;
}

.primary {
  @extend .button;
  background-color: $primary;
  color: white;

  &:hover {
    background-color: darken($primary, 10%);
  }
}

.secondary {
  @extend .button;
  background-color: $secondary;
  color: white;

  &:hover {
    background-color: darken($secondary, 10%);
  }
}
```

```tsx
// components/Button.tsx
import styles from '../styles/Button.module.scss'

type ButtonProps = {
  variant?: 'primary' | 'secondary'
  children: React.ReactNode
}

export function Button({ variant = 'primary', children }: ButtonProps) {
  return (
    <button className={styles[variant]}>
      {children}
    </button>
  )
}
```

## Sass 옵션 커스터마이징

`next.config.js`에서 `sassOptions`를 사용하여 Sass 컴파일러를 설정할 수 있습니다.

### TypeScript 설정

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  sassOptions: {
    additionalData: `$var: red;`,
  },
}

export default nextConfig
```

### JavaScript 설정

```js
// next.config.js
const path = require('path')

module.exports = {
  sassOptions: {
    includePaths: [path.join(__dirname, 'styles')],
    additionalData: `$primary-color: #0070f3;`,
  },
}
```

### 주요 옵션

| 옵션 | 설명 |
|------|------|
| `includePaths` | Sass가 `@import` 시 검색할 경로 배열 |
| `additionalData` | 모든 Sass 파일 앞에 추가될 코드 |
| `implementation` | 사용할 Sass 구현 (`sass` 또는 `sass-embedded`) |

## Sass 구현 지정

기본값은 `sass` 패키지이며, 더 빠른 컴파일을 위해 `sass-embedded`로 변경할 수 있습니다:

```bash
npm install --save-dev sass-embedded
```

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  sassOptions: {
    implementation: 'sass-embedded',
  },
}

export default nextConfig
```

## Sass 변수 JavaScript로 내보내기

CSS Module의 `:export` 구문을 사용하여 Sass 변수를 JavaScript에서 사용할 수 있습니다.

### SCSS 파일

```scss
/* styles/variables.module.scss */
$primary-color: #64ff00;
$secondary-color: #0070f3;
$font-size-base: 16px;
$border-radius: 8px;

:export {
  primaryColor: $primary-color;
  secondaryColor: $secondary-color;
  fontSizeBase: $font-size-base;
  borderRadius: $border-radius;
}
```

### React 컴포넌트에서 사용

```jsx
// pages/_app.js
import variables from '../styles/variables.module.scss'

export default function MyApp({ Component, pageProps }) {
  console.log(variables.primaryColor) // '#64ff00'

  return (
    <Layout color={variables.primaryColor}>
      <Component {...pageProps} />
    </Layout>
  )
}
```

```tsx
// components/ThemedButton.tsx
import variables from '../styles/variables.module.scss'

export function ThemedButton({ children }) {
  return (
    <button
      style={{
        backgroundColor: variables.primaryColor,
        borderRadius: variables.borderRadius,
      }}
    >
      {children}
    </button>
  )
}
```

## Mixins 사용

Sass mixins을 활용하여 재사용 가능한 스타일을 정의할 수 있습니다:

```scss
/* styles/mixins.scss */
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

@mixin responsive($breakpoint) {
  @if $breakpoint == 'sm' {
    @media (max-width: 640px) { @content; }
  } @else if $breakpoint == 'md' {
    @media (max-width: 768px) { @content; }
  } @else if $breakpoint == 'lg' {
    @media (max-width: 1024px) { @content; }
  }
}
```

```scss
/* styles/component.module.scss */
@import './mixins';

.container {
  @include flex-center;
  height: 100vh;

  @include responsive('md') {
    flex-direction: column;
  }
}
```

## 전역 Sass 파일

`pages/_app.js`에서 전역 Sass 파일을 import할 수 있습니다:

```jsx
// pages/_app.js
import '../styles/globals.scss'

export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

```scss
/* styles/globals.scss */
@import './variables';
@import './mixins';
@import './reset';

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  margin: 0;
  padding: 0;
}

* {
  box-sizing: border-box;
}
```

## 권장 사항

1. **SCSS 문법 사용**: CSS와 호환되어 학습 곡선이 낮습니다.
2. **변수와 Mixins 활용**: 재사용 가능한 코드를 작성하세요.
3. **CSS Modules와 결합**: 컴포넌트별 스코프를 유지하세요.
4. **파일 구조 정리**: 변수, mixins, 컴포넌트 스타일을 분리하세요.
