# next.config.js

Next.js는 프로젝트 루트 디렉토리(`package.json`과 같은 위치)의 `next.config.js` 파일을 통해 설정할 수 있습니다.

## 기본 사용법

### JavaScript (CommonJS)

```js filename="next.config.js"
// @ts-check

/** @type {import('next').NextConfig} */
const nextConfig = {
  /* 설정 옵션들 */
}

module.exports = nextConfig
```

### ECMAScript Modules

ECMAScript 모듈을 사용하려면 `.mjs` 확장자를 사용하세요:

```js filename="next.config.mjs"
// @ts-check

/**
 * @type {import('next').NextConfig}
 */
const nextConfig = {
  /* 설정 옵션들 */
}

export default nextConfig
```

> **참고**: `.cjs` 또는 `.cts` 확장자는 현재 지원되지 않습니다.

## 함수형 설정

`next.config.js`는 객체 대신 함수를 내보낼 수 있습니다:

### 동기 함수

```js filename="next.config.mjs"
export default (phase, { defaultConfig }) => {
  /**
   * @type {import('next').NextConfig}
   */
  const nextConfig = {
    /* 설정 옵션들 */
  }
  return nextConfig
}
```

### 비동기 함수

Next.js 12.1.0부터 `async` 함수를 사용할 수 있습니다:

```js filename="next.config.js"
module.exports = async (phase, { defaultConfig }) => {
  /**
   * @type {import('next').NextConfig}
   */
  const nextConfig = {
    /* 설정 옵션들 */
  }
  return nextConfig
}
```

### Phase를 사용한 조건부 설정

`phase`는 현재 설정이 로드되는 컨텍스트입니다. 사용 가능한 phase를 확인하려면 `next/constants`를 참조하세요:

```js filename="next.config.js"
const { PHASE_DEVELOPMENT_SERVER } = require('next/constants')

module.exports = (phase, { defaultConfig }) => {
  if (phase === PHASE_DEVELOPMENT_SERVER) {
    return {
      /* 개발 전용 설정 옵션들 */
    }
  }

  return {
    /* 개발을 제외한 모든 phase의 설정 옵션들 */
  }
}
```

## TypeScript 지원

TypeScript를 사용하는 경우 `next.config.ts` 파일을 사용할 수 있습니다:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  /* 설정 옵션들 */
}

export default nextConfig
```

## 설정 옵션

다음은 사용 가능한 모든 설정 옵션입니다:

### 애플리케이션 설정

| 옵션 | 설명 |
|------|------|
| [`appDir`](/docs/app/api-reference/config/next-config-js/appDir) | App Router 활성화 |
| [`basePath`](/docs/app/api-reference/config/next-config-js/basePath) | 서브경로에 배포 |
| [`compress`](/docs/app/api-reference/config/next-config-js/compress) | gzip 압축 (기본값: true) |
| [`distDir`](/docs/app/api-reference/config/next-config-js/distDir) | 커스텀 빌드 디렉토리 |
| [`env`](/docs/app/api-reference/config/next-config-js/env) | 빌드 타임 환경 변수 추가 |
| [`generateBuildId`](/docs/app/api-reference/config/next-config-js/generateBuildId) | 빌드 ID 설정 |
| [`pageExtensions`](/docs/app/api-reference/config/next-config-js/pageExtensions) | 페이지 확장자 설정 |
| [`reactStrictMode`](/docs/app/api-reference/config/next-config-js/reactStrictMode) | React Strict Mode 활성화 |
| [`trailingSlash`](/docs/app/api-reference/config/next-config-js/trailingSlash) | 후행 슬래시 처리 |

### 이미지 및 에셋

| 옵션 | 설명 |
|------|------|
| [`assetPrefix`](/docs/app/api-reference/config/next-config-js/assetPrefix) | CDN 설정 |
| [`images`](/docs/app/api-reference/config/next-config-js/images) | next/image 로더 설정 |

### 라우팅

| 옵션 | 설명 |
|------|------|
| [`headers`](/docs/app/api-reference/config/next-config-js/headers) | 커스텀 HTTP 헤더 추가 |
| [`redirects`](/docs/app/api-reference/config/next-config-js/redirects) | 리다이렉트 추가 |
| [`rewrites`](/docs/app/api-reference/config/next-config-js/rewrites) | 리라이트 추가 |

### 빌드 및 최적화

| 옵션 | 설명 |
|------|------|
| [`output`](/docs/app/api-reference/config/next-config-js/output) | 배포를 위한 출력 설정 |
| [`optimizePackageImports`](/docs/app/api-reference/config/next-config-js/optimizePackageImports) | 패키지 임포트 최적화 |
| [`transpilePackages`](/docs/app/api-reference/config/next-config-js/transpilePackages) | 패키지 트랜스파일 |
| [`turbopack`](/docs/app/api-reference/config/next-config-js/turbopack) | Turbopack 설정 |
| [`webpack`](/docs/app/api-reference/config/next-config-js/webpack) | webpack 설정 커스터마이징 |

### Server Actions

| 옵션 | 설명 |
|------|------|
| [`serverActions`](/docs/app/api-reference/config/next-config-js/serverActions) | Server Actions 설정 |
| [`serverExternalPackages`](/docs/app/api-reference/config/next-config-js/serverExternalPackages) | Server Components 번들링에서 제외 |

### 캐싱

| 옵션 | 설명 |
|------|------|
| [`cacheLife`](/docs/app/api-reference/config/next-config-js/cacheLife) | 캐시 수명 설정 |
| [`cacheHandlers`](/docs/app/api-reference/config/next-config-js/cacheHandlers) | 커스텀 캐시 핸들러 |
| [`staleTimes`](/docs/app/api-reference/config/next-config-js/staleTimes) | Client Router Cache 무효화 시간 |

### TypeScript 및 ESLint

| 옵션 | 설명 |
|------|------|
| [`typescript`](/docs/app/api-reference/config/next-config-js/typescript) | TypeScript 에러 처리 |
| [`eslint`](/docs/app/api-reference/config/next-config-js/eslint) | ESLint 설정 |

### HTTP 및 네트워크

| 옵션 | 설명 |
|------|------|
| [`crossOrigin`](/docs/app/api-reference/config/next-config-js/crossOrigin) | script 태그 crossOrigin 설정 |
| [`httpAgentOptions`](/docs/app/api-reference/config/next-config-js/httpAgentOptions) | HTTP Keep-Alive 옵션 |
| [`poweredByHeader`](/docs/app/api-reference/config/next-config-js/poweredByHeader) | x-powered-by 헤더 |

### 개발

| 옵션 | 설명 |
|------|------|
| [`devIndicators`](/docs/app/api-reference/config/next-config-js/devIndicators) | 개발 표시기 설정 |
| [`logging`](/docs/app/api-reference/config/next-config-js/logging) | 데이터 페칭 로깅 |
| [`onDemandEntries`](/docs/app/api-reference/config/next-config-js/onDemandEntries) | 페이지 메모리 관리 |

### 기타

| 옵션 | 설명 |
|------|------|
| [`generateEtags`](/docs/app/api-reference/config/next-config-js/generateEtags) | etag 생성 |
| [`productionBrowserSourceMaps`](/docs/app/api-reference/config/next-config-js/productionBrowserSourceMaps) | 프로덕션 소스맵 |
| [`urlImports`](/docs/app/api-reference/config/next-config-js/urlImports) | 외부 URL 임포트 |
| [`webVitalsAttribution`](/docs/app/api-reference/config/next-config-js/webVitalsAttribution) | Web Vitals 귀속 |

### 실험적 기능

| 옵션 | 설명 |
|------|------|
| `authInterrupts` | `forbidden`과 `unauthorized` 사용 |
| `reactCompiler` | React Compiler로 자동 최적화 |
| `viewTransition` | ViewTransition API 활성화 |
| `typedRoutes` | 정적으로 타입 지정된 링크 |

## 단위 테스팅

Next.js 15.1부터 `next/experimental/testing/server` 모듈을 통해 `next.config.js`의 헤더, 리다이렉트, 리라이트를 단위 테스트할 수 있습니다:

```js
import {
  getRedirectUrl,
  unstable_getResponseFromNextConfig,
} from 'next/experimental/testing/server'

const response = await unstable_getResponseFromNextConfig({
  url: 'https://nextjs.org/test',
  nextConfig: {
    async redirects() {
      return [{ source: '/test', destination: '/test2', permanent: false }]
    },
  },
})
expect(response.status).toEqual(307)
expect(getRedirectUrl(response)).toEqual('https://nextjs.org/test2')
```

## 참고사항

- `next.config.js`는 일반 Node.js 모듈이며 JSON 파일이 아닙니다.
- Next.js 서버와 빌드 phase에서 사용되며 브라우저 빌드에는 포함되지 않습니다.
- Webpack, Babel, TypeScript로 트랜스파일되지 않으므로 현재 사용 중인 Node.js 버전에서 지원하는 기능만 사용해야 합니다.
