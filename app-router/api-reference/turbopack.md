# Turbopack

Turbopack은 JavaScript와 TypeScript에 최적화된 **증분 번들러**입니다. Rust로 작성되었으며 Next.js에 내장되어 있습니다.

---

## 개요

### 주요 특징

| 특징 | 설명 |
|------|------|
| **통합 그래프** | 모든 환경(클라이언트, 서버)을 위한 단일 통합 그래프 사용 |
| **번들링** | 개발 중 번들링 수행으로 대규모 앱의 과도한 네트워크 요청 방지 |
| **증분 계산** | 코어 간 병렬 처리 및 함수 수준의 결과 캐싱 |
| **지연 번들링** | 개발 서버에서 요청한 것만 번들링하여 초기 컴파일 시간 단축 |

---

## 시작하기

### 기본 사용 (Turbopack이 기본값)

Next.js 16부터 Turbopack이 기본 번들러입니다.

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

### Webpack 사용 (선택사항)

```json
{
  "scripts": {
    "dev": "next dev --webpack",
    "build": "next build --webpack",
    "start": "next start"
  }
}
```

---

## 지원되는 기능

### 언어 기능

| 기능 | 상태 | 설명 |
|------|------|------|
| JavaScript & TypeScript | ✅ 지원 | SWC 사용, 타입 체킹은 별도 필요 |
| ECMAScript (ESNext) | ✅ 지원 | 최신 ECMAScript 기능 지원 |
| CommonJS | ✅ 지원 | `require()` 지원 |
| ESM | ✅ 지원 | 정적/동적 `import` 지원 |
| Babel | ✅ 지원 | Next.js 16+에서 자동 감지 |

### 프레임워크 및 React 기능

| 기능 | 상태 |
|------|------|
| JSX / TSX | ✅ 지원 |
| Fast Refresh | ✅ 지원 |
| React Server Components (RSC) | ✅ 지원 |
| 루트 레이아웃 자동 생성 | ❌ 미지원 |

### CSS 및 스타일

| 기능 | 상태 | 설명 |
|------|------|------|
| Global CSS | ✅ 지원 | `.css` 파일 직접 import |
| CSS Modules | ✅ 지원 | `.module.css` 기본 지원 |
| CSS Nesting | ✅ 지원 | Lightning CSS 사용 |
| @import | ✅ 지원 | 여러 CSS 파일 병합 |
| PostCSS | ✅ 지원 | `postcss.config.js` 자동 처리 |
| Sass / SCSS | ✅ 지원 | 기본 지원, 커스텀 함수 미지원 |
| Less | ⏳ 계획 중 | 플러그인을 통해 향후 지원 |

### 자산(Assets)

| 기능 | 상태 |
|------|------|
| 정적 자산 (이미지, 폰트) | ✅ 지원 |
| JSON Imports | ✅ 지원 |

### 모듈 해석

| 기능 | 상태 | 설명 |
|------|------|------|
| 경로 별칭 | ✅ 지원 | `tsconfig.json` paths/baseUrl 읽기 |
| 수동 별칭 | ✅ 지원 | `resolveAlias` 설정 |
| 커스텀 확장자 | ✅ 지원 | `resolveExtensions` 설정 |
| AMD | 부분 지원 | 기본 변환만 동작 |

---

## 설정

### next.config.js 설정

```js
// next.config.js
module.exports = {
  turbopack: {
    // 웹팩 로더 추가
    rules: {
      '*.svg': {
        loaders: ['@svgr/webpack'],
        as: '*.js',
      },
    },

    // 수동 별칭 생성
    resolveAlias: {
      underscore: 'lodash',
      '@': './src',
    },

    // 파일 확장자 변경/확장
    resolveExtensions: [
      '.mdx',
      '.tsx',
      '.ts',
      '.jsx',
      '.js',
      '.json',
    ],

    // 파일시스템 루트 설정
    root: '../',
  },
}
```

### 설정 옵션

| 옵션 | 설명 |
|------|------|
| `rules` | 파일 변환을 위한 웹팩 로더 정의 |
| `resolveAlias` | 모듈 별칭 설정 (Webpack의 `resolve.alias`와 유사) |
| `resolveExtensions` | 모듈 해석 시 파일 확장자 설정 |
| `root` | 파일시스템 루트 디렉토리 설정 |

---

## Webpack과의 차이점

### CSS 모듈 순서

Turbopack은 import 순서를 따릅니다.

```jsx
// Import 순서대로 스타일 적용
import utilStyles from './utils.module.css'    // 먼저
import buttonStyles from './button.module.css' // 나중
```

### Sass node_modules 임포트

**Webpack (이전):**
```scss
@import '~bootstrap/dist/css/bootstrap.min.css';
```

**Turbopack (현재):**
```scss
@import 'bootstrap/dist/css/bootstrap.min.css';
```

또는 별칭 설정으로 호환성 유지:

```js
// next.config.js
module.exports = {
  turbopack: {
    resolveAlias: {
      '~*': '*',
    },
  },
}
```

### 파일시스템 루트

프로젝트 외부 파일을 해석해야 하는 경우:

```js
// next.config.js
module.exports = {
  turbopack: {
    root: '../', // 프로젝트 외부 파일 해석
  },
}
```

### 빌드 캐싱

```js
// next.config.js
module.exports = {
  experimental: {
    turbopackFileSystemCacheForDev: true,   // 기본값
    turbopackFileSystemCacheForBuild: true, // 옵트인
  },
}
```

---

## 미지원 기능

### CSS 모듈 레거시 기능

- 독립형 `:local` / `:global` 의사 클래스
- `@value` 규칙
- `:import` / `:export` ICSS 규칙
- `composes`에서 `.css` 파일 참조

### 기타 미지원

| 기능 | 상태 |
|------|------|
| `sassOptions.functions` | 미지원 (커스텀 Sass 함수) |
| `webpack()` 설정 | 미지원 (`turbopack` 사용) |
| Webpack 플러그인 | 미지원 (로더만 지원) |
| Yarn PnP | 미계획 |
| `experimental.urlImports` | 미계획 |
| `experimental.esmExternals` | 미계획 |

---

## 성능 디버깅

### 트레이스 파일 생성

```bash
NEXT_TURBOPACK_TRACING=1 next dev
```

결과: `.next/dev/trace-turbopack` 파일이 생성됩니다.

GitHub 이슈 제출 시 이 파일을 첨부하면 Next.js 팀의 진단에 도움이 됩니다.

---

## 웹팩 로더 사용

Turbopack은 일부 웹팩 로더를 지원합니다.

### SVG 로더 예제

```js
// next.config.js
module.exports = {
  turbopack: {
    rules: {
      '*.svg': {
        loaders: ['@svgr/webpack'],
        as: '*.js',
      },
    },
  },
}
```

### MDX 로더 예제

```js
// next.config.js
module.exports = {
  turbopack: {
    rules: {
      '*.mdx': {
        loaders: ['@mdx-js/loader'],
        as: '*.js',
      },
    },
  },
}
```

---

## 중요한 주의사항

> **Good to know**:
> - Next.js 16부터 Turbopack이 기본 번들러입니다
> - Webpack을 사용하려면 `--webpack` 플래그를 추가하세요
> - Babel은 자동으로 감지되어 지원됩니다

> **마이그레이션 팁**:
> - Sass에서 `~` 접두사를 제거하세요
> - CSS Modules의 레거시 기능 사용을 피하세요
> - Webpack 플러그인 대신 로더를 사용하세요

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v16.0.0 | Turbopack이 Next.js 기본 번들러로 설정, Babel 자동 지원 |
| v15.5.0 | `build` 베타 지원 |
| v15.3.0 | `build` 실험적 지원 |
| v15.0.0 | `dev` 안정 버전 |

---

## 관련 문서

- [next CLI](./cli/next.md)
- [Turbopack Architecture](../architecture/turbopack.md)
- [webpack 설정](./next-config/webpack.md)
