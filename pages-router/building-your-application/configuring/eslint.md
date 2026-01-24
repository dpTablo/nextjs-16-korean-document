# ESLint

Next.js는 `eslint-config-next` 패키지를 제공하여 애플리케이션의 일반적인 문제를 쉽게 감지할 수 있게 합니다.

## 패키지 구성

`eslint-config-next` 패키지에는 다음이 포함됩니다:

- `@next/eslint-plugin-next` 플러그인
- `eslint-plugin-react` 권장 규칙
- `eslint-plugin-react-hooks` 권장 규칙

## 제공되는 설정

| 설정 | 설명 |
|------|------|
| `eslint-config-next` | 기본 설정 (JavaScript, TypeScript 지원) |
| `eslint-config-next/core-web-vitals` | 기본 설정 + Core Web Vitals 영향 규칙을 경고에서 오류로 업그레이드 |
| `eslint-config-next/typescript` | TypeScript 프로젝트용 추가 규칙 |

## 설치

```bash
# npm
npm install -D eslint eslint-config-next

# yarn
yarn add --dev eslint eslint-config-next

# pnpm
pnpm add -D eslint eslint-config-next
```

## ESLint 설정

`eslint.config.mjs` 파일을 생성합니다:

```js
// eslint.config.mjs
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

## ESLint 실행

```bash
# npm
npx eslint .

# yarn
yarn eslint .

# pnpm
pnpm exec eslint .
```

## 주요 규칙

### @next/eslint-plugin-next 규칙

| 규칙 | 설명 |
|------|------|
| `@next/next/google-font-display` | Google Fonts의 font-display 동작 강제 |
| `@next/next/google-font-preconnect` | Google Fonts와 함께 preconnect 사용 보장 |
| `@next/next/inline-script-id` | inline content가 있는 next/script에 id 속성 강제 |
| `@next/next/no-css-tags` | 수동 스타일시트 태그 방지 |
| `@next/next/no-head-element` | `<head>` 요소 사용 방지 |
| `@next/next/no-html-link-for-pages` | 내부 페이지 네비게이션에 `<a>` 요소 사용 방지 |
| `@next/next/no-img-element` | 느린 LCP로 인해 `<img>` 요소 사용 방지 |
| `@next/next/no-sync-scripts` | 동기 스크립트 방지 |

## 규칙 비활성화

```js
// eslint.config.mjs
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

## TypeScript 지원 추가

```js
// eslint.config.mjs
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

## Prettier와 함께 사용

```bash
npm install -D eslint-config-prettier
```

```js
// eslint.config.mjs
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

## Monorepo에서 루트 디렉토리 지정

```js
// eslint.config.mjs
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

`rootDir`는 경로(상대 또는 절대), glob(예: `"packages/*/"`) 또는 경로/glob 배열일 수 있습니다.

## Staged Files에서 Lint 실행

lint-staged와 함께 사용할 수 있습니다:

```js
// .lintstagedrc.js
const path = require('path')

const buildEslintCommand = (filenames) =>
  `eslint --fix ${filenames
    .map((f) => `"${path.relative(process.cwd(), f)}"`)
    .join(' ')}`

module.exports = {
  '*.{js,jsx,ts,tsx}': [buildEslintCommand],
}
```

## package.json 스크립트

```json
{
  "scripts": {
    "lint": "eslint",
    "lint:fix": "eslint --fix"
  }
}
```

## 주요 변경사항

- Next.js 16부터 `next lint` 명령이 제거되었습니다.
- ESLint CLI를 직접 사용해야 합니다.
- `next.config.js`의 `eslint` 옵션은 더 이상 필요하지 않습니다.
