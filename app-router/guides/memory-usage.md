# Memory Usage

Next.js 애플리케이션의 메모리 사용량을 최적화하는 방법을 알아봅니다.

## 개요

애플리케이션이 성장함에 따라 로컬 개발이나 프로덕션 빌드 시 더 많은 리소스를 요구할 수 있습니다. 이 문서는 Next.js에서 메모리 사용량을 최적화하고 디버깅하는 전략을 다룹니다.

---

## 최적화 전략

### 1. 의존성 줄이기

대규모 의존성은 메모리 사용량을 증가시킵니다. **Bundle Analyzer**를 사용하여 불필요한 대용량 의존성을 식별하고 제거하세요.

```bash
# 번들 분석 실행
ANALYZE=true npm run build
```

**일반적인 대체 방법:**
- `moment.js` → `date-fns` 또는 `dayjs`
- `lodash` → `lodash-es` 또는 개별 함수
- `axios` → 네이티브 `fetch`

---

### 2. Webpack 메모리 최적화 (v15.0.0+)

`next.config.js`에 추가하여 Webpack의 메모리 사용량을 최적화합니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    webpackMemoryOptimizations: true,
  },
}

module.exports = nextConfig
```

**효과:**
- 최대 메모리 사용량 감소
- 컴파일 시간이 약간 증가할 수 있음
- 낮은 위험의 실험적 기능

---

### 3. 메모리 사용량 디버깅 (v14.2.0+)

빌드 중 메모리 사용량을 지속적으로 모니터링합니다.

```bash
next build --experimental-debug-memory-usage
```

**출력 정보:**
- 힙 사용량
- 가비지 컬렉션 통계
- 메모리가 한계에 도달하면 자동으로 힙 스냅샷 생성

> **참고:** 이 옵션은 Webpack 빌드 워커 옵션과 호환되지 않습니다.

---

### 4. 힙 프로파일 기록

Chrome DevTools에서 분석할 수 있는 힙 프로파일을 생성합니다.

```bash
node --heap-prof node_modules/next/dist/bin/next build
```

**분석 방법:**
1. 빌드 완료 후 `.heapprofile` 파일이 생성됨
2. Chrome DevTools를 열고 Memory 탭으로 이동
3. "Load Profile" 버튼을 클릭하여 파일 로드
4. 메모리 할당 패턴 분석

---

### 5. 힙 스냅샷 분석

실시간으로 메모리 스냅샷을 캡처하고 분석합니다.

```bash
# inspect 모드로 빌드 시작
NODE_OPTIONS=--inspect next build

# 또는 시작 시 중단점 설정
NODE_OPTIONS=--inspect-brk next build
```

**분석 방법:**
1. Chrome 브라우저에서 `chrome://inspect` 열기
2. Remote Target에서 Node.js 프로세스 선택
3. Memory 탭에서 "Take snapshot" 클릭
4. 스냅샷 분석

**`--experimental-debug-memory-usage` 모드에서:**
```bash
# 프로세스에 SIGUSR2 시그널을 보내 스냅샷 캡처
kill -SIGUSR2 <pid>
```

---

### 6. Webpack 빌드 워커 (v14.1.0+)

별도의 Node.js 워커에서 Webpack을 실행하여 메모리 사용량을 줄입니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    webpackBuildWorker: true,
  },
}

module.exports = nextConfig
```

**참고:**
- 커스텀 Webpack 구성이 없는 경우 기본적으로 활성화됨
- 일부 커스텀 Webpack 플러그인과 호환되지 않을 수 있음

---

### 7. Webpack 캐시 비활성화

메모리 캐시로 전환하여 디스크 I/O를 줄입니다.

**next.config.mjs**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  webpack: (config, { dev }) => {
    if (config.cache && !dev) {
      config.cache = Object.freeze({
        type: 'memory',
      })
    }
    return config
  },
}

export default nextConfig
```

**또는 완전히 비활성화:**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  webpack: (config, { dev }) => {
    if (!dev) {
      config.cache = false
    }
    return config
  },
}

export default nextConfig
```

---

### 8. 정적 분석 비활성화

TypeScript 타입 체크를 빌드에서 분리합니다.

**next.config.mjs**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  typescript: {
    ignoreBuildErrors: true,
  },
  eslint: {
    ignoreDuringBuilds: true,
  },
}

export default nextConfig
```

> **경고:** 이 설정은 빌드 오류를 무시하므로 CI에서 별도의 타입 체크 단계가 필요합니다.

**CI에서 별도 타입 체크:**
```yaml
# GitHub Actions 예시
jobs:
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run type-check

  build:
    needs: type-check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
```

---

### 9. 소스맵 비활성화

소스맵 생성을 비활성화하여 빌드 중 메모리를 절약합니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  productionBrowserSourceMaps: false,
  experimental: {
    serverSourceMaps: false,
  },
}

module.exports = nextConfig
```

**프리렌더 단계에서도 비활성화:**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    enablePrerenderSourceMaps: false,
  },
}

module.exports = nextConfig
```

> **참고:** 일부 플러그인이 이 설정을 덮어쓸 수 있습니다.

---

### 10. 엔트리 사전 로딩 비활성화

기본적으로 Next.js는 시작 시 모든 페이지 JavaScript 모듈을 사전 로드합니다. 이를 비활성화하여 시작 시 메모리 사용량을 줄일 수 있습니다.

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const config: NextConfig = {
  experimental: {
    preloadEntriesOnStart: false,
  },
}

export default config
```

**트레이드오프:**
- ✅ 시작 시 메모리 사용량 감소
- ❌ 첫 번째 요청 시 지연 발생
- ⚠️ 시간이 지나 모든 페이지가 요청되면 최종 메모리 사용량은 동일

---

## 메모리 모니터링

### 프로덕션 메모리 모니터링

**process.memoryUsage() 사용:**
```ts
// instrumentation.ts
export async function register() {
  setInterval(() => {
    const usage = process.memoryUsage()
    console.log({
      heapUsed: Math.round(usage.heapUsed / 1024 / 1024) + 'MB',
      heapTotal: Math.round(usage.heapTotal / 1024 / 1024) + 'MB',
      rss: Math.round(usage.rss / 1024 / 1024) + 'MB',
    })
  }, 30000) // 30초마다
}
```

### API Route에서 메모리 확인

**app/api/health/route.ts**
```ts
import { NextResponse } from 'next/server'

export async function GET() {
  const usage = process.memoryUsage()

  return NextResponse.json({
    status: 'ok',
    memory: {
      heapUsed: Math.round(usage.heapUsed / 1024 / 1024),
      heapTotal: Math.round(usage.heapTotal / 1024 / 1024),
      rss: Math.round(usage.rss / 1024 / 1024),
      external: Math.round(usage.external / 1024 / 1024),
    },
    unit: 'MB',
  })
}
```

---

## 일반적인 메모리 문제

### 메모리 누수 패턴

**1. 전역 변수에 데이터 축적:**
```ts
// ❌ 나쁜 예
const cache: Record<string, any> = {}

export function addToCache(key: string, value: any) {
  cache[key] = value // 계속 증가
}

// ✅ 좋은 예: LRU 캐시 사용
import LRUCache from 'lru-cache'

const cache = new LRUCache({ max: 100 })
```

**2. 이벤트 리스너 정리 누락:**
```ts
// ❌ 나쁜 예
useEffect(() => {
  window.addEventListener('resize', handleResize)
}, [])

// ✅ 좋은 예
useEffect(() => {
  window.addEventListener('resize', handleResize)
  return () => window.removeEventListener('resize', handleResize)
}, [])
```

**3. 대용량 데이터 클로저:**
```ts
// ❌ 나쁜 예
const largeData = fetchLargeData()
app.get('/api', () => {
  return process(largeData) // largeData가 메모리에 유지됨
})

// ✅ 좋은 예: 필요할 때만 로드
app.get('/api', async () => {
  const data = await fetchLargeData()
  return process(data)
})
```

---

## 환경별 최적화

### 개발 환경

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    webpackMemoryOptimizations: true,
  },
}

module.exports = nextConfig
```

### CI/빌드 환경

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    webpackMemoryOptimizations: true,
    webpackBuildWorker: true,
  },
  productionBrowserSourceMaps: false,
  typescript: {
    ignoreBuildErrors: process.env.CI === 'true',
  },
}

module.exports = nextConfig
```

### 프로덕션 환경

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    preloadEntriesOnStart: false, // 메모리 제한 환경에서
  },
}

module.exports = nextConfig
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **정기적으로 의존성 감사**
   ```bash
   npm ls --depth=0
   npx depcheck
   ```

2. **메모리 사용량 모니터링**
   - 개발 중: `--experimental-debug-memory-usage`
   - 프로덕션: APM 도구 사용

3. **점진적 최적화**
   - 가장 영향이 큰 설정부터 적용
   - 각 변경 후 측정

4. **캐싱 전략 수립**
   - 크기 제한이 있는 캐시 사용
   - TTL 설정

### ❌ 피해야 할 것

1. **모든 최적화 동시 적용**
   - 문제 원인 파악이 어려워짐

2. **프로덕션에서 디버그 옵션 사용**
   - 성능 저하 발생

3. **무한히 증가하는 캐시**
   - 메모리 누수의 주요 원인

---

## 다음 단계

- [Package Bundling](./package-bundling.md) - 번들 최적화
- [CI Build Caching](./ci-build-caching.md) - CI 빌드 캐싱
- [Debugging](./debugging.md) - 디버깅 가이드

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-15

**참고 자료:**
- [Node.js Memory Management](https://nodejs.org/en/docs/guides/diagnostics/memory)
- [Chrome DevTools Memory Panel](https://developer.chrome.com/docs/devtools/memory-problems)
- [V8 Garbage Collection](https://v8.dev/blog/trash-talk)
