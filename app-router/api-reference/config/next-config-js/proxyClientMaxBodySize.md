# proxyClientMaxBodySize

`proxyClientMaxBodySize`는 프록시를 사용할 때 버퍼링된 요청 본문의 크기 제한을 설정하는 실험적 Next.js 설정 옵션입니다. 이는 자동으로 복제되고 버퍼링된 요청 본문으로 인한 과도한 메모리 소비를 방지합니다.

**기본 제한:** 10MB

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서의 사용은 권장하지 않습니다.

## 설정 방법

### 1. 문자열 형식 (권장)

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    proxyClientMaxBodySize: '1mb',
  },
}

export default nextConfig
```

지원 단위: `b`, `kb`, `mb`, `gb`

### 2. 숫자 형식 (바이트)

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    proxyClientMaxBodySize: 1048576, // 1MB (바이트)
  },
}

export default nextConfig
```

## 제한 초과 시 동작

1. 설정된 제한까지 처음 N 바이트만 버퍼링됩니다
2. 문제가 되는 라우트를 나타내는 경고가 콘솔에 기록됩니다
3. 부분적인 본문 데이터로 요청 처리가 정상적으로 계속됩니다
4. **클라이언트에 오류가 반환되지 않습니다** - 요청이 실패하지 않습니다

## 중요 사항

- **실험적 기능** - 프로덕션에서는 권장하지 않음
- 애플리케이션에서 프록시를 사용할 때만 적용됩니다
- 요청당 제한 (동시 요청 전체에 대한 전역 제한이 아님)
- 대용량 파일 업로드의 경우 그에 맞게 제한을 늘리세요
- 제한이 초과되면 애플리케이션에서 부분 본문을 적절히 처리해야 합니다

## 권장사항

애플리케이션에 전체 요청 본문이 필요한 경우:
- `proxyClientMaxBodySize` 제한을 늘리세요
- 애플리케이션 로직에서 부분 본문 데이터의 적절한 처리를 구현하세요

### 예제: 대용량 파일 업로드 허용

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    proxyClientMaxBodySize: '50mb', // 50MB까지 허용
  },
}

export default nextConfig
```

### 예제: 메모리 사용 최소화

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    proxyClientMaxBodySize: '512kb', // 512KB로 제한
  },
}

export default nextConfig
```

---

## 참고

- [프록시 가이드](/app-router/getting-started/16-proxy.md)
- [메모리 사용](/app-router/guides/memory-usage.md)
- [셀프 호스팅](/app-router/guides/self-hosting.md)
