---
원문: https://nextjs.org/docs/app/guides/self-hosting
버전: 16.1.6
---

# 셀프 호스팅

## 핵심 인프라 설정

### 리버스 프록시
- **권장사항:** Next.js 서버를 인터넷에 직접 노출하지 말고 리버스 프록시(예: nginx)를 앞에 사용하세요
- **이점:** 잘못된 요청, 느린 연결 공격, 페이로드 크기 제한, 속도 제한 및 기타 보안 문제 처리
- **결과:** 서버 리소스가 요청 검증이 아닌 렌더링에 전념

## 기능 설정

### 이미지 최적화
- `next start`를 사용하여 배포할 때 제로 설정으로 작동
- 대안: 별도의 최적화 서비스를 위한 [커스텀 이미지 로더 설정](/docs/app/api-reference/components/image.md#loader)
- `next.config.js`의 커스텀 이미지 로더를 통한 [정적 내보내기](/docs/app/guides/static-exports.md#image-optimization)와 호환
- **참고:** 이미지는 빌드 시점이 아닌 런타임에 최적화됨
- **Linux 고려사항:** glibc 기반 시스템은 과도한 메모리 사용을 방지하기 위해 [추가 설정](https://sharp.pixelplumbing.com/install#linux-memory-allocator)이 필요할 수 있음

### Middleware/Proxy
- `next start` 사용 시 제로 설정으로 작동
- 낮은 지연 시간을 위해 Edge 런타임(Node.js API의 하위 집합) 사용
- 정적 내보내기에서는 지원되지 않음
- 대안: 전체 Node.js 런타임 사용 또는 로직을 [서버 컴포넌트](/docs/app/getting-started/server-and-client-components.md)로 이동

## 환경 변수

**핵심 원칙:** 환경 변수는 기본적으로 서버 전용입니다. 브라우저에 노출하려면 `NEXT_PUBLIC_`으로 접두사를 붙이세요.

```tsx filename="app/page.ts"
import { connection } from 'next/server'

export default async function Component() {
  await connection()
  const value = process.env.MY_VALUE
  // ...
}
```

**이점:** 다양한 값으로 여러 환경에서 프로모션되는 단일 Docker 이미지 사용

## 캐싱 전략

### 자동 캐싱 동작
- **불변 에셋:** `Cache-Control: public, max-age=31536000, immutable` (SHA 해시가 있는 파일)
- **ISR 페이지:** `Cache-Control: s-maxage: <revalidate>, stale-while-revalidate`
- **동적 페이지:** `Cache-Control: private, no-cache, no-store, max-age=0, must-revalidate`

### 커스텀 캐시 핸들러 (컨테이너 오케스트레이션용)

```jsx filename="next.config.js"
module.exports = {
  cacheHandler: require.resolve('./cache-handler.js'),
  cacheMaxMemorySize: 0, // 인메모리 캐싱 비활성화
}
```

```jsx filename="cache-handler.js"
const cache = new Map()

module.exports = class CacheHandler {
  constructor(options) {
    this.options = options
  }

  async get(key) {
    return cache.get(key)
  }

  async set(key, data, ctx) {
    cache.set(key, {
      value: data,
      lastModified: Date.now(),
      tags: ctx.tags,
    })
  }

  async revalidateTag(tags) {
    tags = [tags].flat()
    for (let [key, value] of cache) {
      if (value.tags.some((tag) => tags.includes(tag))) {
        cache.delete(key)
      }
    }
  }

  resetRequestCache() {}
}
```

**사용 사례:** 다중 Pod 배포를 위한 Redis, AWS S3 또는 모든 내구성 있는 스토리지

### 정적 에셋 배포
`next.config.js`의 `assetPrefix`를 사용하여 다른 도메인/CDN에서 에셋 호스팅:
```jsx filename="next.config.js"
module.exports = {
  assetPrefix: 'https://cdn.example.com',
}
```

## 빌드 설정

### 일관된 빌드 ID (다중 컨테이너 배포용)
```jsx filename="next.config.js"
module.exports = {
  generateBuildId: async () => {
    return process.env.GIT_HASH
  },
}
```

## 스트리밍 및 Suspense

**Nginx 설정:** 스트리밍 응답을 활성화하려면 버퍼링 비활성화:
```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*{/}?',
        headers: [
          {
            key: 'X-Accel-Buffering',
            value: 'no',
          },
        ],
      },
    ]
  },
}
```

## 버전 스큐

Next.js는 버전 스큐를 자동으로 완화하고 감지 시 새 에셋을 검색하기 위해 리로드합니다. 다음을 사용하여 상태 유지:
- URL 상태
- 로컬 스토리지
- 컴포넌트 상태(`useState`)에 의존하지 마세요

## 서버 종료

`after` 콜백으로 우아한 종료:
- `SIGINT` 또는 `SIGTERM` 신호 전송
- 대기 중인 콜백 함수/프로미스가 완료될 때까지 대기

## 추가 기능

- **캐시 컴포넌트:** 기본적으로 작동 (CDN 전용 아님)
- **`after` 함수:** `next start`로 완전 지원
- **동적 API:** CDN 호환성을 위해 자동으로 적절한 `Cache-Control` 헤더 설정

---

## 배포 체크리스트

### 프로덕션 준비

1. ✅ **리버스 프록시 설정** (nginx, Caddy 등)
2. ✅ **환경 변수 구성** (서버 전용 vs `NEXT_PUBLIC_`)
3. ✅ **캐시 핸들러 구현** (다중 인스턴스의 경우)
4. ✅ **이미지 최적화 확인** (`next start` 또는 커스텀 로더)
5. ✅ **CDN 설정** (정적 에셋용 `assetPrefix`)
6. ✅ **빌드 ID 일관성** (다중 컨테이너)
7. ✅ **스트리밍 활성화** (Nginx 버퍼링 비활성화)
8. ✅ **우아한 종료** (시그널 핸들링)

### 성능 최적화

- **캐싱 전략 검증**: ISR, 동적 경로, 정적 에셋
- **CDN 통합**: 정적 에셋 및 이미지
- **모니터링**: 메모리 사용량, 응답 시간, 캐시 히트율
- **로드 밸런싱**: 여러 인스턴스 간
- **헬스 체크**: 컨테이너 오케스트레이션용

### 보안

- **HTTPS 강제**
- **보안 헤더 설정** (CSP, HSTS 등)
- **속도 제한 구현**
- **환경 변수 보호** (시크릿 관리)
- **정기 업데이트**: Next.js 및 종속성
