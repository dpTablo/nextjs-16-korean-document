---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/allowedDevOrigins
버전: 16.1.6
---

# allowedDevOrigins

## 개요

`allowedDevOrigins`는 개발 모드에서 크로스 오리진 요청을 허용할 추가 오리진을 지정할 수 있는 Next.js 설정 옵션입니다.

## 목적

- **현재 동작**: Next.js는 개발 중 크로스 오리진 요청을 자동으로 차단하지 않음
- **향후 동작**: 향후 메이저 버전에서 내부 에셋/엔드포인트에 대한 무단 액세스를 방지하기 위해 기본적으로 크로스 오리진 요청 차단 예정
- **해결책**: `allowedDevOrigins`를 사용하여 신뢰할 수 있는 오리진을 화이트리스트에 추가

## 설정

```js filename="next.config.js"
module.exports = {
  allowedDevOrigins: ['local-origin.dev', '*.local-origin.dev'],
}
```

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  allowedDevOrigins: ['local-origin.dev', '*.local-origin.dev'],
}

export default nextConfig
```

## 사용 방법

| 항목 | 설명 |
|------|------|
| 기본 오리진 | `localhost`는 기본적으로 허용됨 (서버가 초기화된 호스트네임) |
| 추가 오리진 | 이 옵션을 사용하여 기본값 이외의 오리진에서 요청 허용 |
| 와일드카드 지원 | 와일드카드 패턴 지원 (예: `*.local-origin.dev`) |

## 사용 사례

### 커스텀 개발 도메인

`localhost` 대신 `local-origin.dev`를 사용하려면:

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  allowedDevOrigins: ['local-origin.dev'],
}

module.exports = nextConfig
```

### 여러 서브도메인 허용

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  allowedDevOrigins: [
    'app.local-origin.dev',
    'admin.local-origin.dev',
    '*.local-origin.dev', // 모든 서브도메인
  ],
}

module.exports = nextConfig
```

## 참고사항

- 이 설정은 개발 모드에서만 적용됩니다
- 프로덕션 환경에서는 별도의 CORS 설정이 필요합니다
