# turbopackFileSystemCache

Turbopack 파일 시스템 캐시는 빌드 간에 데이터를 `.next` 폴더에 저장하고 복원하여 `next dev` 또는 `next build` 명령에서 작업량을 줄이고, 후속 빌드와 개발 세션의 속도를 높입니다.

## 개요

- **상태:** 개발 환경에서는 안정적, 프로덕션 빌드에서는 실험적

## 설정

### TypeScript

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    // `next dev`에 대한 파일 시스템 캐싱 활성화
    turbopackFileSystemCacheForDev: true,
    // `next build`에 대한 파일 시스템 캐싱 활성화
    turbopackFileSystemCacheForBuild: true,
  },
}

export default nextConfig
```

### JavaScript

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    // `next dev`에 대한 파일 시스템 캐싱 활성화
    turbopackFileSystemCacheForDev: true,
    // `next build`에 대한 파일 시스템 캐싱 활성화
    turbopackFileSystemCacheForBuild: true,
  },
}

module.exports = nextConfig
```

## 설정 플래그

| 플래그 | 설명 |
|--------|------|
| `turbopackFileSystemCacheForDev` | `next dev` 명령에 대한 파일 시스템 캐싱 활성화 |
| `turbopackFileSystemCacheForBuild` | `next build` 명령에 대한 파일 시스템 캐싱 활성화 |

## 작동 방식

1. **첫 번째 빌드/개발 시작**: Turbopack이 컴파일 결과를 `.next` 폴더에 캐시합니다
2. **후속 빌드/개발 시작**: 이전 캐시에서 데이터를 복원하여 빌드 시간을 단축합니다
3. **파일 변경 감지**: 변경된 파일만 다시 컴파일합니다

## 이점

- **빠른 개발 서버 시작**: 캐시된 데이터를 재사용하여 초기 로드 시간 단축
- **빌드 시간 감소**: 증분 빌드로 전체 빌드 시간 개선
- **리소스 효율성**: 불필요한 재컴파일 방지

## 캐시 위치

캐시 데이터는 `.next` 폴더 내에 저장됩니다:

```
.next/
├── cache/
│   └── turbopack/     ← Turbopack 캐시 데이터
└── ...
```

## 캐시 무효화

캐시를 수동으로 무효화하려면 `.next` 폴더를 삭제합니다:

```bash
rm -rf .next
```

또는 다음 상황에서 자동으로 무효화됩니다:
- Next.js 버전 업그레이드
- `next.config.js` 설정 변경
- `package.json` 종속성 변경

## 버전 기록

| 버전 | 상태 |
|------|------|
| v16.1.0 | 개발 환경에서 파일 시스템 캐싱이 기본적으로 활성화됨 |
| v16.0.0 | 빌드와 개발을 위한 별도 플래그로 베타 릴리스 |
| v15.5.0 | canary에서 실험적 릴리스 |

---

## 참고

- [Turbopack](/app-router/api-reference/turbopack.md)
- [turbopack 설정](/app-router/api-reference/config/next-config-js/turbopack.md)
- [CI 빌드 캐싱](/app-router/guides/ci-build-caching.md)
