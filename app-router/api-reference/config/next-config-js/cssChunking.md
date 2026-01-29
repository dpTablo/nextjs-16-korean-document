---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/cssChunking
버전: 16.1.6
---

# cssChunking

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

CSS Chunking은 CSS 파일을 분할하고 재정렬하여 웹 애플리케이션의 성능을 개선하는 전략입니다. 전체 CSS를 한 번에 로드하는 대신 특정 라우트에 필요한 CSS만 로드할 수 있습니다.

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    cssChunking: true, // 기본값
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    cssChunking: true, // 기본값
  },
}

module.exports = nextConfig
```

## 옵션

| 옵션 | 설명 |
|------|------|
| `true` (기본값) | CSS 파일을 최대한 병합하여 청크 수를 줄이고 요청 수를 감소시킴 |
| `false` | CSS 파일 병합/재정렬 안 함 |
| `'strict'` | import 순서대로 CSS 파일을 로드하여 더 많은 청크와 요청 발생 가능 |

## 언제 'strict' 옵션을 사용해야 하나?

예상치 못한 CSS 동작이 발생할 때 고려해야 합니다. 예를 들어:

- `a.css`와 `b.css`가 다른 파일에서 다른 순서로 import될 때
- `true` 옵션은 파일 순서와 관계없이 병합하지만
- `b.css`가 `a.css`에 의존하는 경우 `'strict'`를 사용하여 import 순서대로 로드해야 합니다

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    cssChunking: 'strict',
  },
}

module.exports = nextConfig
```

## 권장사항

**대부분의 애플리케이션에는 `true`를 권장합니다** - 더 적은 요청과 향상된 성능을 제공합니다.

CSS 순서에 의존하는 스타일 문제가 발생하는 경우에만 `'strict'` 옵션을 사용하세요.
