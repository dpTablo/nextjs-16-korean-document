# Package Bundling 최적화

Next.js 애플리케이션의 번들 크기를 분석하고 최적화하는 방법을 알아봅니다.

## 개요

번들링은 애플리케이션 코드와 의존성을 최적화된 출력 파일로 결합합니다. 작은 번들은 다음을 개선합니다:

- **로드 시간** - 더 빠른 페이지 로드
- **JavaScript 실행 시간** - 더 빠른 상호작용
- **Core Web Vitals** - LCP, INP, CLS 개선
- **서버 콜드 스타트 시간** - 서버리스 환경에서 더 빠른 시작

Next.js는 코드 분할과 트리 셰이킹을 통해 자동으로 번들을 최적화하지만, 일부 경우에는 수동 최적화가 필요할 수 있습니다.

---

## 번들 분석 도구

### 1. Next.js Bundle Analyzer (실험적 - v16.1+)

Turbopack의 모듈 그래프와 통합되어 정확한 import 추적으로 서버와 클라이언트 모듈을 검사합니다.

#### 단계 1: 분석기 실행

```bash
npx next experimental-analyze
```

#### 단계 2: 모듈 필터링 및 검사

라우트, 환경(클라이언트/서버), 타입(JavaScript, CSS, JSON)별로 필터링하거나 파일로 검색합니다.

#### 단계 3: 모듈 추적

트리맵에서 모듈을 클릭하면 크기와 전체 import 체인을 볼 수 있습니다.

#### 단계 4: 출력 저장

분석 결과를 저장하여 공유하거나 비교합니다:

```bash
npx next experimental-analyze --output
```

출력은 `.next/diagnostics/analyze`에 저장됩니다:

```bash
# 리팩토링 전 결과 백업
cp -r .next/diagnostics/analyze ./analyze-before-refactor
```

---

### 2. @next/bundle-analyzer (Webpack)

Webpack을 사용하는 경우 기존 번들 분석기를 사용할 수 있습니다.

**설치:**
```bash
npm install @next/bundle-analyzer
```

**next.config.js 구성:**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {}

const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})

module.exports = withBundleAnalyzer(nextConfig)
```

**리포트 생성:**
```bash
ANALYZE=true npm run build
```

이 명령어는 세 개의 HTML 파일을 생성합니다:
- `client.html` - 클라이언트 번들
- `server.html` - 서버 번들
- `edge.html` - Edge 번들

---

## 대용량 번들 최적화

### 1. 많은 Export를 가진 패키지 최적화

아이콘 라이브러리나 유틸리티 라이브러리처럼 많은 export를 가진 패키지의 경우, `optimizePackageImports`를 사용하여 사용된 모듈만 로드합니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    optimizePackageImports: [
      'lucide-react',
      '@heroicons/react',
      '@radix-ui/react-icons',
      'lodash',
      'date-fns',
      '@mui/material',
      '@mui/icons-material',
    ],
  },
}

module.exports = nextConfig
```

**효과:**
```tsx
// 이 import는
import { Calendar, Clock, User } from 'lucide-react'

// 내부적으로 다음과 같이 최적화됨
import Calendar from 'lucide-react/dist/esm/icons/calendar'
import Clock from 'lucide-react/dist/esm/icons/clock'
import User from 'lucide-react/dist/esm/icons/user'
```

**기본적으로 최적화되는 패키지:**
- `lucide-react`
- `@heroicons/react`
- `date-fns`
- `lodash`
- `react-bootstrap`
- `@mui/material`
- `@mui/icons-material`

---

### 2. 무거운 클라이언트 작업 최적화

**문제:** 구문 강조, 차트 렌더링, 마크다운 파싱과 같은 라이브러리가 클라이언트 코드에 번들됩니다.

**해결책:** 비용이 많이 드는 렌더링을 Server Components로 이동합니다.

#### Before (클라이언트 컴포넌트)

```tsx
'use client'

import Highlight from 'prism-react-renderer'
import theme from 'prism-react-renderer/themes/github'

export default function Page() {
  const code = `export function hello() {
    console.log("hi")
  }`

  return (
    <article>
      <h1>블로그 글 제목</h1>
      {/* Prism 패키지가 클라이언트로 전송됨 */}
      <Highlight code={code} language="tsx" theme={theme}>
        {({ className, style, tokens, getLineProps, getTokenProps }) => (
          <pre className={className} style={style}>
            <code>
              {tokens.map((line, i) => (
                <div key={i} {...getLineProps({ line })}>
                  {line.map((token, key) => (
                    <span key={key} {...getTokenProps({ token })} />
                  ))}
                </div>
              ))}
            </code>
          </pre>
        )}
      </Highlight>
    </article>
  )
}
```

#### After (서버 컴포넌트)

```tsx
import { codeToHtml } from 'shiki'

export default async function Page() {
  const code = `export function hello() {
    console.log("hi")
  }`

  const highlightedHtml = await codeToHtml(code, {
    lang: 'tsx',
    theme: 'github-dark',
  })

  return (
    <article>
      <h1>블로그 글 제목</h1>
      {/* 클라이언트는 순수 마크업만 받음 */}
      <pre>
        <code dangerouslySetInnerHTML={{ __html: highlightedHtml }} />
      </pre>
    </article>
  )
}
```

**결과:**
- 클라이언트 번들에서 무거운 라이브러리 제거
- 클라이언트는 미리 렌더링된 HTML만 수신
- 더 빠른 페이지 로드와 상호작용

---

### 3. 패키지를 번들링에서 제외

Server Components와 Route Handlers에서 사용하는 패키지를 번들에서 제외하려면 `serverExternalPackages`를 사용합니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  serverExternalPackages: ['sharp', 'prisma', '@aws-sdk/client-s3'],
}

module.exports = nextConfig
```

**사용 사례:**
- 네이티브 바이너리가 있는 패키지 (`sharp`)
- 대용량 서버 전용 SDK
- 번들링과 호환되지 않는 패키지

---

## 추가 최적화 기법

### 동적 Import

코드 분할을 통해 필요할 때만 모듈을 로드합니다.

```tsx
import dynamic from 'next/dynamic'

// 지연 로딩 컴포넌트
const HeavyChart = dynamic(() => import('./heavy-chart'), {
  loading: () => <div>차트 로딩 중...</div>,
  ssr: false,
})

export default function Dashboard() {
  return (
    <div>
      <h1>대시보드</h1>
      <HeavyChart />
    </div>
  )
}
```

### 조건부 Import

특정 조건에서만 모듈을 로드합니다.

```tsx
export default async function Page() {
  // 서버에서만 실행
  if (someCondition) {
    const heavyModule = await import('heavy-module')
    // 사용
  }

  return <div>페이지 콘텐츠</div>
}
```

### 트리 셰이킹 최적화

사용하지 않는 코드가 제거되도록 named import를 사용합니다.

```tsx
// ✅ 좋은 예: named import
import { format, parseISO } from 'date-fns'

// ❌ 나쁜 예: 전체 import
import * as dateFns from 'date-fns'
```

### Barrel 파일 최적화

Barrel 파일(`index.ts`)은 트리 셰이킹을 방해할 수 있습니다.

```tsx
// ❌ 나쁜 예: barrel 파일에서 import
import { Button } from '@/components'

// ✅ 좋은 예: 직접 import
import Button from '@/components/button'
```

또는 `optimizePackageImports`에 내부 경로를 추가합니다:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    optimizePackageImports: ['@/components'],
  },
}
```

---

## 번들 크기 모니터링

### 빌드 출력 분석

빌드 시 Next.js가 출력하는 크기 정보를 확인합니다:

```
Route (app)                              Size     First Load JS
┌ ○ /                                    5.2 kB         89 kB
├ ○ /about                               1.8 kB         85 kB
├ ● /blog/[slug]                         2.1 kB         86 kB
└ ○ /contact                             1.5 kB         85 kB
+ First Load JS shared by all            83.8 kB

○  (Static)   prerendered as static content
●  (SSG)      prerendered as static HTML (uses getStaticProps)
```

### CI에서 번들 크기 체크

GitHub Actions에서 번들 크기를 모니터링합니다:

**.github/workflows/bundle-size.yml**
```yaml
name: Bundle Size Check

on: [pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build and analyze
        run: |
          npm run build
          # 번들 크기를 JSON으로 추출
          cat .next/build-manifest.json | jq '.pages' > bundle-sizes.json

      - name: Comment bundle sizes
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const sizes = JSON.parse(fs.readFileSync('bundle-sizes.json'));
            // PR에 코멘트 추가
```

---

## 문제 해결

### 번들이 예상보다 큰 경우

1. **분석기 실행**
   ```bash
   npx next experimental-analyze
   ```

2. **대용량 의존성 확인**
   - moment.js → date-fns 또는 dayjs
   - lodash → lodash-es 또는 개별 함수
   - axios → fetch API

3. **클라이언트/서버 코드 분리**
   - 서버 전용 코드가 클라이언트에 포함되지 않았는지 확인

### 트리 셰이킹이 작동하지 않는 경우

1. **ESM 패키지 사용**
   - CommonJS 패키지는 트리 셰이킹이 제한됨

2. **Side effects 확인**
   - `package.json`에 `"sideEffects": false` 추가

3. **Named import 사용**
   ```tsx
   // ✅ 트리 셰이킹 가능
   import { specific } from 'package'

   // ❌ 전체 패키지 포함
   import package from 'package'
   ```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **정기적인 번들 분석**
   - 새 의존성 추가 시마다 분석

2. **Server Components 활용**
   - 무거운 라이브러리는 서버에서 처리

3. **동적 Import 사용**
   - 초기 로드에 필요하지 않은 컴포넌트는 지연 로딩

4. **의존성 최신 유지**
   - 새 버전은 종종 더 작은 번들 크기 제공

### ❌ 피해야 할 것

1. **불필요한 의존성 추가**
   - 간단한 기능에 대용량 라이브러리 사용 금지

2. **전체 라이브러리 Import**
   ```tsx
   // ❌ 피하기
   import _ from 'lodash'
   ```

3. **클라이언트에서 서버 전용 코드 Import**

4. **Barrel 파일 남용**

---

## 다음 단계

- [Lazy Loading](./lazy-loading.md) - 지연 로딩
- [Server Components](../getting-started/server-and-client-components.md) - 서버 컴포넌트
- [Rendering](./rendering.md) - 렌더링 전략
- [TypeScript](./typescript.md) - TypeScript 설정

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-15

**참고 자료:**
- [webpack Bundle Analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)
- [Import Cost VSCode Extension](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)
- [Bundlephobia](https://bundlephobia.com/)
