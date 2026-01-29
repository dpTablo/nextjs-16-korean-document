---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/cacheLife
버전: 16.1.6
---

# cacheLife

`cacheLife` 옵션을 사용하면 `use cache` 지시어 범위 내에서 컴포넌트와 함수 내부의 `cacheLife()` 함수와 함께 사용할 **커스텀 캐시 프로필**을 정의할 수 있습니다.

## 설정 요구 사항

1. `next.config.js`에서 `cacheComponents` 플래그를 활성화합니다
2. `cacheLife` 객체에 캐시 프로필을 정의합니다

## 구성 예제

### TypeScript (next.config.ts)

```typescript
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
  cacheLife: {
    blog: {
      stale: 3600, // 1시간
      revalidate: 900, // 15분
      expire: 86400, // 1일
    },
  },
}

export default nextConfig
```

### JavaScript (next.config.js)

```javascript
module.exports = {
  cacheComponents: true,
  cacheLife: {
    blog: {
      stale: 3600, // 1시간
      revalidate: 900, // 15분
      expire: 86400, // 1일
    },
  },
}
```

## 코드에서 사용

```typescript
import { cacheLife } from 'next/cache'

export async function getCachedData() {
  'use cache'
  cacheLife('blog')
  const data = await fetch('/api/data')
  return data
}
```

## 구성 매개변수

| 속성 | 타입 | 설명 | 요구 사항 |
|------|------|------|-----------|
| `stale` | `number` | 클라이언트가 서버 검증 없이 값을 캐시하는 기간 (초) | 선택 사항 |
| `revalidate` | `number` | 서버 사이드 캐시가 새로고침되는 빈도 (초); 재검증 중에 오래된 값이 제공될 수 있음 | 선택 사항 |
| `expire` | `number` | 값이 동적으로 전환되기 전에 오래된 상태로 유지되는 최대 기간 (초) | 선택 사항 - `revalidate` 이상이어야 함 |

## 관련 참조

- `use cache` 지시어
- `cacheHandlers` 구성
- `cacheLife()` 함수
