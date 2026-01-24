# reactCompiler

## 개요

Next.js에는 React Compiler가 내장되어 있습니다. 이 성능 최적화 도구는 `useMemo`와 `useCallback`을 사용한 수동 메모이제이션 없이도 컴포넌트 렌더링을 자동으로 최적화합니다.

## 주요 특징

- **자동 최적화**: 수동 메모이제이션 없이 컴포넌트 렌더링 자동 최적화
- **스마트 적용**: Next.js는 커스텀 SWC 최적화를 사용하여 관련 파일(JSX 또는 React Hooks가 있는 파일)에만 컴파일러를 적용하여 불필요한 컴파일 방지
- **성능**: 독립형 Babel 플러그인 사용과 비교하여 빌드 성능에 미치는 영향 최소화

## 설치

```bash
npm install -D babel-plugin-react-compiler
```

## 설정

### 기본 설정 (자동 모드)

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

### 어노테이션 모드 (옵트인)

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactCompiler: {
    compilationMode: 'annotation',
  },
}

export default nextConfig
```

## 어노테이션 사용법

`annotation` 모드 사용 시 `"use memo"` 지시어로 특정 컴포넌트를 옵트인할 수 있습니다:

```ts filename="app/page.tsx"
export default function Page() {
  'use memo'
  // 컴포넌트 코드...
}
```

> **참고**: `"use no memo"` 지시어를 사용하여 컴포넌트나 훅을 컴파일에서 옵트아웃할 수도 있습니다.

## 작동 방식

| 모드 | 설명 |
|------|------|
| `true` | 모든 관련 파일에 자동으로 React Compiler 적용 |
| `{ compilationMode: 'annotation' }` | `"use memo"` 지시어가 있는 컴포넌트에만 적용 |

## 사용 사례

- **성능 최적화**: 수동 메모이제이션 없이 리렌더링 최적화
- **코드 간소화**: `useMemo`, `useCallback` 보일러플레이트 제거
- **점진적 도입**: 어노테이션 모드로 점진적으로 적용
