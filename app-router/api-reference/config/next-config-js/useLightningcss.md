# useLightningcss

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

Next.js에서 webpack과 함께 [Lightning CSS](https://lightningcss.dev)를 사용하기 위한 실험적 지원입니다.

**주요 특징:**

- Lightning CSS는 Rust로 작성된 빠른 CSS 변환기 및 미니파이어입니다
- 이 옵션을 설정하지 않으면 Next.js는 webpack에서 기본적으로 PostCSS와 `postcss-preset-env`를 사용합니다
- **Turbopack**: Next 14.2 이후 기본적으로 Lightning CSS를 사용합니다 (이 설정은 무시됨)

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    useLightningcss: true, // webpack에서 PostCSS 비활성화
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    useLightningcss: true, // webpack에서 PostCSS 비활성화
  },
}

module.exports = nextConfig
```

## Turbopack과의 관계

Turbopack을 사용하는 경우 이 설정은 무시됩니다. Turbopack은 Next.js 14.2 이후 기본적으로 Lightning CSS를 CSS 프로세서로 사용합니다.

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| `15.1.0` | Turbopack에서 `useSwcCss` 지원 제거 |
| `14.2.0` | Turbopack의 기본 CSS 프로세서가 `@swc/css`에서 Lightning CSS로 변경. `useLightningcss`는 Turbopack에서 무시되며, `experimental.turbo.useSwcCss` 레거시 옵션 추가 |
