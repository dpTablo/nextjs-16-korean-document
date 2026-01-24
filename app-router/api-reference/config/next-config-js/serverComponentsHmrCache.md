# serverComponentsHmrCache

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

`serverComponentsHmrCache`는 로컬 개발 환경에서 Hot Module Replacement(HMR) 새로고침 시 Server Components의 `fetch` 응답을 캐싱하는 실험적 옵션입니다.

### 주요 이점

- 더 빠른 응답 속도
- 청구 API 호출 비용 감소

## 동작 방식

### 기본 동작 (활성화 상태)

- 모든 `fetch` 요청에 HMR 캐시 적용
- `cache: 'no-store'` 옵션이 있는 요청도 캐시됨
- HMR 새로고침 간에 캐시된 데이터 사용 (신선한 데이터 미표시)
- 페이지 네비게이션이나 전체 페이지 새로고침 시 캐시 초기화

## 설정

HMR 캐시를 비활성화하려면:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    serverComponentsHmrCache: false, // 기본값: true
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverComponentsHmrCache: false, // 기본값: true
  },
}

module.exports = nextConfig
```

## 권장사항

개선된 관찰성을 위해 `logging.fetches` 옵션 사용을 권장합니다. 개발 중 콘솔에 fetch 캐시 히트/미스를 로깅합니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}

module.exports = nextConfig
```
