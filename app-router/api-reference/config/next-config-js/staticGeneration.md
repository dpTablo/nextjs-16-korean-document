# staticGeneration*

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

`staticGeneration*` 옵션은 고급 사용 사례를 위한 정적 생성 프로세스를 구성할 수 있습니다.

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    staticGenerationRetryCount: 1,
    staticGenerationMaxConcurrency: 8,
    staticGenerationMinPagesPerWorker: 25,
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    staticGenerationRetryCount: 1,
    staticGenerationMaxConcurrency: 8,
    staticGenerationMinPagesPerWorker: 25,
  },
}

module.exports = nextConfig
```

## 옵션

| 옵션 | 설명 |
|------|------|
| `staticGenerationRetryCount` | 빌드 실패 전 실패한 페이지 생성을 재시도하는 횟수 |
| `staticGenerationMaxConcurrency` | 워커당 처리할 최대 페이지 수 |
| `staticGenerationMinPagesPerWorker` | 새로운 워커를 시작하기 전에 처리할 최소 페이지 수 |

## 사용 사례

- **불안정한 빌드**: `staticGenerationRetryCount`를 늘려 일시적인 실패를 극복
- **대규모 사이트**: `staticGenerationMaxConcurrency`와 `staticGenerationMinPagesPerWorker`를 조정하여 빌드 성능 최적화
- **메모리 제한 환경**: 동시성을 낮춰 메모리 사용량 감소
