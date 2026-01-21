# ESLint

Next.js는 애플리케이션의 일반적인 문제를 쉽게 잡을 수 있도록 `eslint-config-next` 패키지를 제공합니다.

## 개요

`eslint-config-next` 패키지에는 다음이 포함됩니다:
- `@next/eslint-plugin-next` 플러그인
- `eslint-plugin-react`의 권장 규칙
- `eslint-plugin-react-hooks`의 권장 규칙

## 설정 옵션

### Base Configuration

```
eslint-config-next
```

- Next.js, React, React Hooks 규칙 포함
- JavaScript 및 TypeScript 파일 지원

### Core Web Vitals Configuration

```
eslint-config-next/core-web-vitals
```

- 기본 설정의 모든 기능 포함
- Core Web Vitals에 영향을 미치는 규칙을 경고에서 오류로 업그레이드
- **대부분의 프로젝트에 권장**

### TypeScript Configuration

```
eslint-config-next/typescript
```

- `typescript-eslint`의 TypeScript 특정 규칙 추가
- 기본 또는 core-web-vitals 설정과 함께 사용

## 설정 방법

### 1단계: 설치

```bash
# pnpm
pnpm add -D eslint eslint-config-next

# npm
npm i -D eslint eslint-config-next

# yarn
yarn add --dev eslint eslint-config-next

# bun
bun add -d eslint eslint-config-next
```

### 2단계: eslint.config.mjs 생성

```js filename="eslint.config.mjs"
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'

const eslintConfig = defineConfig([
  ...nextVitals,
  globalIgnores([
    '.next/**',
    'out/**',
    'build/**',
    'next-env.d.ts',
  ]),
])

export default eslintConfig
```

### 3단계: ESLint 실행

```bash
# pnpm
pnpm exec eslint .

# npm
npx eslint .

# yarn
yarn eslint .

# bun
bunx eslint .
```

## @next/eslint-plugin-next 규칙

| 규칙 | 설명 |
|------|------|
| `@next/next/google-font-display` | Google Fonts의 font-display 동작 강제 |
| `@next/next/google-font-preconnect` | Google Fonts에서 preconnect 사용 보장 |
| `@next/next/inline-script-id` | inline 콘텐츠가 있는 next/script에 id 속성 강제 |
| `@next/next/next-script-for-ga` | Google Analytics 인라인 스크립트 사용 시 next/script 선호 |
| `@next/next/no-assign-module-variable` | module 변수 할당 방지 |
| `@next/next/no-async-client-component` | Client Components의 async 함수 방지 |
| `@next/next/no-before-interactive-script-outside-document` | pages/_document.js 외부에서 beforeInteractive 전략 사용 방지 |
| `@next/next/no-css-tags` | 수동 스타일시트 태그 방지 |
| `@next/next/no-document-import-in-page` | pages/_document.js 외부에서 next/document import 방지 |
| `@next/next/no-duplicate-head` | pages/_document.js에서 중복 `<Head>` 사용 방지 |
| `@next/next/no-head-element` | `<head>` 요소 사용 방지 |
| `@next/next/no-head-import-in-document` | pages/_document.js에서 next/head 사용 방지 |
| `@next/next/no-html-link-for-pages` | 내부 Next.js 페이지로 이동하기 위한 `<a>` 요소 사용 방지 |
| `@next/next/no-img-element` | `<img>` 요소 사용 방지 (낮은 LCP, 높은 대역폭) |
| `@next/next/no-page-custom-font` | 페이지 전용 커스텀 폰트 방지 |
| `@next/next/no-script-component-in-head` | next/head 컴포넌트에서 next/script 사용 방지 |
| `@next/next/no-styled-jsx-in-document` | pages/_document.js에서 styled-jsx 사용 방지 |
| `@next/next/no-sync-scripts` | 동기 스크립트 방지 |
| `@next/next/no-title-in-document-head` | next/document의 Head 컴포넌트에서 `<title>` 사용 방지 |
| `@next/next/no-typos` | Next.js 데이터 페칭 함수의 일반적인 오타 방지 |
| `@next/next/no-unwanted-polyfillio` | Polyfill.io의 중복 폴리필 방지 |

## 고급 설정

### 규칙 비활성화

```js filename="eslint.config.mjs"
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'

const eslintConfig = defineConfig([
  ...nextVitals,
  {
    rules: {
      'react/no-unescaped-entities': 'off',
      '@next/next/no-page-custom-font': 'off',
    },
  },
  globalIgnores([
    '.next/**',
    'out/**',
    'build/**',
    'next-env.d.ts',
  ]),
])

export default eslintConfig
```

### TypeScript 설정

```js filename="eslint.config.mjs"
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'
import nextTs from 'eslint-config-next/typescript'

const eslintConfig = defineConfig([
  ...nextVitals,
  ...nextTs,
  globalIgnores([
    '.next/**',
    'out/**',
    'build/**',
    'next-env.d.ts',
  ]),
])

export default eslintConfig
```

### Prettier와 통합

```js filename="eslint.config.mjs"
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'
import prettier from 'eslint-config-prettier/flat'

const eslintConfig = defineConfig([
  ...nextVitals,
  prettier,
  globalIgnores([
    '.next/**',
    'out/**',
    'build/**',
    'next-env.d.ts',
  ]),
])

export default eslintConfig
```

### Monorepo에서 루트 디렉토리 지정

```js filename="eslint.config.mjs"
import { defineConfig } from 'eslint/config'
import eslintNextPlugin from '@next/eslint-plugin-next'

const eslintConfig = defineConfig([
  {
    files: ['**/*.{js,jsx,ts,tsx}'],
    plugins: {
      next: eslintNextPlugin,
    },
    settings: {
      next: {
        rootDir: 'packages/my-app/',
      },
    },
  },
])

export default eslintConfig
```

### lint-staged와 함께 사용

```js filename=".lintstagedrc.js"
const path = require('path')

const buildEslintCommand = (filenames) =>
  `eslint --fix ${filenames
    .map((f) => `"${path.relative(process.cwd(), f)}"`)
    .join(' ')}`

module.exports = {
  '*.{js,jsx,ts,tsx}': [buildEslintCommand],
}
```

## 플러그인 직접 사용

기존 ESLint 설정에 Next.js 플러그인을 직접 추가할 수 있습니다:

```js filename="eslint.config.mjs"
import { defineConfig } from 'eslint/config'
import nextPlugin from '@next/eslint-plugin-next'

const eslintConfig = defineConfig([
  {
    files: ['**/*.{js,jsx,ts,tsx}'],
    plugins: {
      '@next/next': nextPlugin,
    },
    rules: {
      ...nextPlugin.configs.recommended.rules,
    },
  },
])

export default eslintConfig
```

## 주요 변경사항 (Next.js 16)

> **중요**: Next.js 16부터 `next lint` 명령과 `next.config.js`의 `eslint` 옵션이 제거되었습니다.

- ESLint CLI를 직접 사용하여 린팅을 수행하세요.
- 에디터 통합을 사용하여 개발 중 경고 및 오류를 직접 확인하는 것을 권장합니다.
- ESLint CLI로 마이그레이션을 위한 codemod가 제공될 수 있습니다.

## 관련 문서

- [ESLint 공식 문서](https://eslint.org/)
- [eslint-config-next npm 패키지](https://www.npmjs.com/package/eslint-config-next)
