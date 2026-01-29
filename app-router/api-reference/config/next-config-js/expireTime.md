---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/expireTime
버전: 16.1.6
---

# expireTime

## 개요

CDN에서 사용할 수 있도록 ISR(Incremental Static Regeneration) 활성화 페이지의 `Cache-Control` 헤더에 사용자 정의 `stale-while-revalidate` 만료 시간을 지정할 수 있습니다.

## 설정

```js filename="next.config.js"
module.exports = {
  // 초 단위로 1시간
  expireTime: 3600,
}
```

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  expireTime: 3600, // 1시간 (초 단위)
}

export default nextConfig
```

## 동작 방식

`Cache-Control` 헤더를 전송할 때, 만료 시간은 특정 revalidate 기간에 따라 계산됩니다.

### 예시

- **revalidate 설정**: 15분 (900초)
- **expireTime**: 1시간 (3600초)
- **생성되는 Cache-Control 헤더**: `s-maxage=900, stale-while-revalidate=2700`

### 헤더 설명

| 지시어 | 값 | 설명 |
|--------|-----|------|
| `s-maxage` | 900 | 15분 동안 캐시 (revalidate 기간) |
| `stale-while-revalidate` | 2700 | 설정된 만료 시간(3600)에서 revalidate 기간(900)을 뺀 45분 동안 stale 상태 유지 가능 |

## 사용 사례

- CDN에서 ISR 페이지의 캐시 동작 제어
- stale 콘텐츠 제공 기간 조정
- 백그라운드 재검증 동안 사용자 경험 최적화
