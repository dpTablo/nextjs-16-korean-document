# 실험적 기능

Next.js는 `next.config.js`의 `experimental` 플래그를 통해 다양한 실험적 기능을 제공합니다. 이러한 기능은 안정화되기 전까지 변경될 수 있습니다.

## authInterrupts

`forbidden`과 `unauthorized` API를 사용할 수 있게 합니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    authInterrupts: true,
  },
}

module.exports = nextConfig
```

### TypeScript 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    authInterrupts: true,
  },
}

export default nextConfig
```

### 관련 API

이 기능을 활성화하면 다음을 사용할 수 있습니다:

| 유형 | 이름 | 설명 |
|------|------|------|
| 함수 | `forbidden` | 금지된 접근 처리 |
| 함수 | `unauthorized` | 인증되지 않은 접근 처리 |
| 파일 규칙 | `forbidden.js` | 금지된 라우트용 특수 파일 |
| 파일 규칙 | `unauthorized.js` | 인증되지 않은 라우트용 특수 파일 |

> **참고**: 현재 canary 채널에서 사용 가능하며 변경될 수 있습니다.

## viewTransition

React의 [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)를 Next.js 애플리케이션에서 활성화합니다. 네이티브 브라우저 View Transitions API를 사용하여 UI 상태 간에 부드러운 전환을 만들 수 있습니다.

> **경고**: 이 기능은 실험적이며 프로덕션 사용은 권장되지 않습니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    viewTransition: true,
  },
}

module.exports = nextConfig
```

### 사용 방법

React에서 `<ViewTransition>` 컴포넌트를 import하여 사용합니다:

```jsx
import { ViewTransition } from 'react'
```

### 주요 사항

- `<ViewTransition>` 컴포넌트는 React의 Canary 릴리스 채널에서 사용 가능합니다.
- `experimental.viewTransition`은 네비게이션에 대한 자동 transition type 추가를 포함한 깊은 Next.js 통합을 활성화합니다.
- Next.js 전용 transition type은 아직 구현되지 않았습니다.
- API는 변경될 수 있습니다.

### 리소스

- [라이브 데모](https://view-transition-example.vercel.app) - Next.js View Transition 예제

## reactCompiler

React Compiler는 컴포넌트 렌더링을 자동으로 최적화하고 `useMemo`와 `useCallback`을 사용한 수동 메모이제이션의 필요성을 줄이는 성능 최적화 도구입니다.

> **참고**: `reactCompiler`는 안정적인 기능으로, `experimental` 블록 없이 직접 설정할 수 있습니다.

### 설치

```bash
npm install -D babel-plugin-react-compiler
```

### 기본 설정

전역으로 React Compiler를 활성화합니다:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactCompiler: true,
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactCompiler: true,
}

module.exports = nextConfig
```

### 주요 특징

- **커스텀 SWC 최적화**: Next.js는 JSX나 React Hooks가 있는 관련 파일에만 React Compiler를 적용하는 커스텀 SWC 최적화를 사용합니다.
- **빠른 빌드**: 이 선택적 접근 방식은 Babel 플러그인만 사용하는 것에 비해 빌드 성능 영향을 최소화합니다.

### Opt-In 모드 (어노테이션)

선택적 컴파일을 위해 어노테이션 모드를 사용합니다:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactCompiler: {
    compilationMode: 'annotation',
  },
}

export default nextConfig
```

`"use memo"` 지시어와 함께 사용:

```tsx
export default function Page() {
  'use memo'
  // 컴포넌트 코드
}
```

### 지시어

| 지시어 | 설명 |
|--------|------|
| `"use memo"` | 컴포넌트나 훅을 컴파일 대상으로 opt-in |
| `"use no memo"` | 컴포넌트나 훅을 컴파일에서 opt-out |

## typedRoutes

정적으로 타입이 지정된 링크를 지원합니다. 이 기능은 프로젝트에서 TypeScript를 사용해야 합니다.

> **참고**: `typedRoutes`는 이제 안정적인 기능으로, `experimental` 블록 없이 직접 설정할 수 있습니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  typedRoutes: true,
}

module.exports = nextConfig
```

### 주요 사항

- **상태**: 안정화됨 (더 이상 실험적이지 않음)
- **이전 이름**: `experimental.typedRoutes`는 `typedRoutes`로 대체되어 deprecated됨
- **요구사항**: 프로젝트에서 TypeScript를 사용해야 함
- **목적**: Next.js 애플리케이션에서 라우트 링크에 대한 정적 타입 검사 제공

이 기능을 활성화하면 라우트 네비게이션에 대한 더 나은 타입 안전성과 IDE 자동완성을 제공합니다.

## 기타 실험적 옵션

`experimental` 객체에서 설정할 수 있는 추가 옵션들:

| 옵션 | 설명 |
|------|------|
| `serverMinification` | 서버 코드 최소화 활성화 |
| `serverSourceMaps` | 서버 코드의 소스맵 생성 |
| `ppr` | Partial Pre-Rendering 활성화 |
| `dynamicIO` | 동적 I/O 최적화 |
| `staleTimes` | 클라이언트 라우터 캐시 무효화 시간 설정 |

> **참고**: 실험적 기능은 안정화되면 `experimental` 블록에서 최상위 레벨로 이동되거나 제거될 수 있습니다.

## 관련 문서

- [next.config.js 옵션](/docs/app/api-reference/config/next-config-js)
- [forbidden 함수](/docs/app/api-reference/functions/forbidden)
- [unauthorized 함수](/docs/app/api-reference/functions/unauthorized)
