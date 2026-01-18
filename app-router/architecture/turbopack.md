# Turbopack

**Turbopack**은 JavaScript와 TypeScript에 최적화된 증분 번들러로, Rust로 작성되었으며 Next.js에 내장되어 있습니다. Pages와 App Router 모두에서 훨씬 빠른 로컬 개발 경험을 제공합니다.

## 왜 Turbopack인가?

1. **통합 그래프** - 여러 컴파일러를 관리하는 대신 모든 환경(클라이언트 및 서버)에 대한 단일 통합 그래프
2. **최적화된 번들링** - 개발에서 번들링(네이티브 ESM 도구와 달리)하지만 대규모 앱을 빠르게 유지하도록 최적화된 방식으로
3. **증분 계산** - 코어 전반에 걸쳐 작업을 병렬화하고 함수 수준까지 결과를 캐시
4. **지연 번들링** - 개발 서버에서 실제로 요청하는 것만 번들링

## 시작하기

Turbopack은 Next.js의 **기본 번들러**로 설정이 필요하지 않습니다:

```json filename="package.json"
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

### 대신 Webpack 사용하기

```json filename="package.json"
{
  "scripts": {
    "dev": "next dev --webpack",
    "build": "next build --webpack",
    "start": "next start"
  }
}
```

## 지원되는 기능

### 언어 기능

- ✅ **JavaScript & TypeScript** - 내부적으로 SWC 사용
- ✅ **ECMAScript (ESNext)** - 최신 기능 지원
- ✅ **CommonJS** - `require()` 구문 기본 지원
- ✅ **ESM** - 정적 및 동적 `import` 완전 지원
- ✅ **Babel** - 자동 감지 및 사용 (Next.js 16+)

### 프레임워크 및 React 기능

- ✅ **JSX / TSX** - SWC가 컴파일 처리
- ✅ **Fast Refresh** - 설정 필요 없음
- ✅ **React Server Components (RSC)** - 올바른 서버/클라이언트 번들링

### CSS 및 스타일링

- ✅ **글로벌 CSS** - `.css` 파일 직접 가져오기
- ✅ **CSS Modules** - `.module.css` 파일 (Lightning CSS)
- ✅ **CSS 중첩** - 최신 CSS 중첩 지원
- ✅ **@import 구문** - 여러 CSS 파일 결합
- ✅ **PostCSS** - `postcss.config.js` 자동 처리
- ✅ **Sass / SCSS** - Next.js에서 기본 지원

### 에셋

- ✅ **정적 에셋** - 이미지, 폰트 가져오기 기본 지원
- ✅ **JSON 가져오기** - `.json`에서 명명 또는 기본 가져오기

### 모듈 해석

- ✅ **경로 별칭** - `tsconfig.json` paths와 baseUrl 읽기
- ✅ **수동 별칭** - `resolveAlias`로 구성
- ✅ **커스텀 확장자** - `resolveExtensions`로 구성

### 성능

- ✅ **Fast Refresh** - 전체 페이지 새로고침 없이 업데이트
- ✅ **증분 번들링** - 요청된 코드만 지연 빌드

## 구성

`next.config.js` 또는 `next.config.ts`를 통해 Turbopack을 구성합니다:

```js filename="next.config.js"
module.exports = {
  turbopack: {
    // 수동 별칭 생성
    resolveAlias: {
      underscore: 'lodash',
    },
    // 파일 확장자 변경 또는 확장
    resolveExtensions: ['.mdx', '.tsx', '.ts', '.jsx', '.js', '.json'],
    // 파일 변환을 위한 webpack 로더 정의
    rules: {
      // 로더 구성
    },
  },
}
```

## Webpack과의 알려진 차이점

### 파일시스템 루트

- 프로젝트 루트 외부의 파일(`npm link`, `yarn link`를 통해)은 기본적으로 해석되지 않음
- `turbopack.root` 옵션을 사용하여 구성

### CSS Module 순서

- Turbopack은 CSS 모듈에 대해 JS 가져오기 순서를 따름
- 미묘한 렌더링 변경을 일으킬 수 있음; 해결책: `@import` 사용 또는 CSS 특이성 업데이트

### 지원되지 않는 Webpack 기능

- 레거시 CSS Modules 기능 (`:local/:global` 의사 클래스, `@value`, `composes`)
- 커스텀 Sass 함수 (`sassOptions.functions`)
- `next.config.js`의 `webpack()` 구성
- **Webpack 플러그인** - 지원되지 않음; webpack 로더는 지원됨

## 성능 디버깅

성능/메모리 문제에 대한 트레이스 파일 생성:

```bash
NEXT_TURBOPACK_TRACING=1 next dev
```

GitHub 이슈에 포함할 `.next/dev/trace-turbopack` 파일을 생성합니다.

## 빌드 캐싱 (베타)

Turbopack의 파일시스템 캐시 활성화:

```js filename="next.config.js"
module.exports = {
  experimental: {
    turbopackFileSystemCacheForDev: true, // 기본적으로 활성화
    turbopackFileSystemCacheForBuild: true, // 선택적
  },
}
```

## 버전 기록

| 버전      | 변경 사항                                     |
| --------- | --------------------------------------------- |
| `v16.0.0` | Turbopack이 기본 번들러가 됨; 자동 Babel 지원 |
| `v15.5.0` | 빌드 베타 지원                                |
| `v15.3.0` | 실험적 빌드 지원                              |
| `v15.0.0` | 개발 안정화                                   |
