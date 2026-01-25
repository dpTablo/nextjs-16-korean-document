# isolatedDevBuild

`isolatedDevBuild`는 개발 빌드와 프로덕션 빌드 출력을 서로 다른 디렉토리로 분리하는 Next.js의 실험적 기능입니다.

## 개요

- **기본값:** 활성화됨 (`true`)
- **상태:** 실험적이며 변경될 수 있음
- **프로덕션 환경에서의 사용은 권장하지 않음**

## 기능

활성화되면, 개발 서버(`next dev`)는 출력을 `.next` 대신 `.next/dev`에 작성합니다. 이는 다음과 같은 상황에서 충돌을 방지합니다:

- `next dev`와 `next build`를 동시에 실행할 때
- AI 에이전트와 같은 자동화 도구가 개발 서버가 실행 중인 동안 변경 사항을 검증할 때
- 빌드 프로세스가 개발 서버를 방해하지 않도록 할 때

## 설정

이 기능을 비활성화하려면 `isolatedDevBuild`를 `false`로 설정합니다:

### TypeScript

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    isolatedDevBuild: false, // 기본값은 true
  },
}

export default nextConfig
```

### JavaScript

```js
// next.config.mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    isolatedDevBuild: false, // 기본값은 true
  },
}

export default nextConfig
```

## 디렉토리 구조

### 활성화 시 (기본값)

```
.next/
├── dev/           ← 개발 빌드 출력
│   ├── cache/
│   └── ...
└── ...            ← 프로덕션 빌드 출력
```

### 비활성화 시

```
.next/             ← 개발 및 프로덕션 빌드 모두 여기에 출력
├── cache/
└── ...
```

## 사용 사례

### CI/CD 파이프라인

개발 서버를 실행하면서 동시에 프로덕션 빌드를 테스트해야 하는 경우:

```bash
# 터미널 1: 개발 서버 실행
next dev

# 터미널 2: 프로덕션 빌드 (개발 서버와 충돌 없음)
next build
```

### AI 코딩 도구

AI 도구가 코드 변경을 검증하면서 개발 서버가 계속 실행되어야 하는 경우 유용합니다.

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| v16.0.0 | `experimental.isolatedDevBuild` 도입 |

---

## 참고

- [distDir](/app-router/api-reference/config/next-config-js/distDir.md)
- [output](/app-router/api-reference/config/next-config-js/output.md)
