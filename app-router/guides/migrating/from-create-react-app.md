# Create React App에서 Next.js로 마이그레이션하기

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-10-17

## 개요

이 가이드는 기존 Create React App (CRA) 사이트를 Next.js로 마이그레이션하는 방법을 설명합니다.

---

## 전환 이유

### 느린 초기 페이지 로딩 시간
- CRA는 순수 클라이언트 렌더링 사용
- 브라우저가 React 코드와 전체 애플리케이션 번들을 다운로드하고 실행할 때까지 기다려야 함
- 새로운 기능과 의존성 추가에 따라 애플리케이션 코드 증가

### 자동 코드 분할 없음
- 수동 코드 분할 시도 시 네트워크 폭포 현상 발생 가능
- Next.js는 라우터와 빌드 파이프라인에 자동 코드 분할과 트리 쉐이킹 제공

### 네트워크 폭포 현상
- SPA에서 순차적 클라이언트-서버 요청 발생
- 자식 컴포넌트는 부모 컴포넌트의 데이터 로딩 완료 후에만 시작 가능
- Next.js는 서버에서 데이터 페칭 가능하여 폭포 현상 제거

### 빠르고 의도적인 로딩 상태
- React Suspense를 통한 스트리밍 지원
- UI의 어떤 부분이 먼저 로드되는지 정의 가능
- 레이아웃 시프트 제거

### 데이터 페칭 전략 선택
- 페이지/컴포넌트 레벨에서 데이터 페칭 전략 선택 가능
- SSG (빌드 시 렌더링) 또는 SSR (요청 시 렌더링) 사용 가능

### 프록시
- 요청 완료 전 서버에서 코드 실행 가능
- 인증 페이지 리다이렉팅, A/B 테스트, 국제화 등에 활용

### 내장 최적화
- 이미지, 폰트, 타사 스크립트 자동 최적화
- Next.js의 전문화된 컴포넌트와 API 제공

---

## 마이그레이션 단계

### Step 1: Next.js 의존성 설치

```bash
npm install next@latest
```

### Step 2: Next.js 설정 파일 생성

프로젝트 루트에 `next.config.ts` 생성:

```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  output: 'export', // Single-Page Application (SPA) 출력
  distDir: 'build', // 빌드 출력 디렉터리를 `build`로 변경
}

export default nextConfig
```

**참고**: `output: 'export'`는 정적 내보내기를 의미하며, SSR이나 API 같은 서버 기능을 지원하지 않습니다. 제거하면 Next.js 서버 기능을 활용할 수 있습니다.

### Step 3: 루트 레이아웃 생성

`src` 폴더 또는 프로젝트 루트에 `app` 디렉터리를 생성한 후, 그 안에 `layout.tsx` (또는 `layout.js`) 파일을 생성합니다:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <head>
        <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
        <title>React App</title>
        <meta name="description" content="Web site created..." />
      </head>
      <body>
        <div id="root">{children}</div>
      </body>
    </html>
  )
}
```

### Step 4: 메타데이터

Next.js는 `<meta charset="UTF-8" />`과 `<meta name="viewport" content="width=device-width, initial-scale=1" />`을 자동으로 포함하므로 제거할 수 있습니다.

메타데이터 파일 (`favicon.ico`, `icon.png`, `robots.txt` 등)을 `app` 디렉터리 최상단에 배치하면 자동으로 `<head>` 태그에 추가됩니다.

Metadata API를 사용하여 최종 `<head>` 태그 관리:

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'React App',
  description: 'Web site created with Next.js.',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <div id="root">{children}</div>
      </body>
    </html>
  )
}
```

### Step 5: 스타일

Next.js는 CRA처럼 CSS Modules과 전역 CSS 가져오기를 지원합니다:

```tsx
import '../index.css'

export const metadata = {
  title: 'React App',
  description: 'Web site created with Next.js.',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <div id="root">{children}</div>
      </body>
    </html>
  )
}
```

### Step 6: 엔트리포인트 페이지 생성

`app` 디렉터리 내에 `[[...slug]]` 디렉터리 생성:

```
app
 ┣ [[...slug]]
 ┃ ┗ page.tsx
 ┣ layout.tsx
```

`page.tsx`에 다음 추가:

```tsx
export function generateStaticParams() {
  return [{ slug: [''] }]
}

export default function Page() {
  return '...' // 나중에 업데이트
}
```

### Step 7: 클라이언트 전용 엔트리포인트 추가

`app/[[...slug]]/` 내에 `client.tsx` 생성:

```tsx
'use client'

import dynamic from 'next/dynamic'

const App = dynamic(() => import('../../App'), { ssr: false })

export function ClientOnly() {
  return <App />
}
```

**설명**:
- `'use client'` 지시문: 클라이언트 컴포넌트로 표시
- `dynamic` 임포트 `ssr: false`: `<App />` 컴포넌트의 서버 렌더링 비활성화

`page.tsx` 업데이트:

```tsx
import { ClientOnly } from './client'

export function generateStaticParams() {
  return [{ slug: [''] }]
}

export default function Page() {
  return <ClientOnly />
}
```

### Step 8: 정적 이미지 임포트 업데이트

CRA에서는 이미지 파일 임포트가 공개 URL 문자열을 반환합니다:

```tsx
import image from './img.png'

export default function App() {
  return <img src={image} />
}
```

Next.js에서는 정적 이미지 임포트가 객체를 반환합니다.

`/public`에서 임포트한 이미지의 절대 임포트 경로를 상대 임포트로 변환:

```tsx
// Before
import logo from '/logo.png'

// After
import logo from '../public/logo.png'
```

이미지 객체 전체 대신 `src` 속성을 `<img>` 태그에 전달:

```tsx
// Before
<img src={logo} />

// After
<img src={logo.src} />
```

### Step 9: 환경 변수 마이그레이션

Next.js는 CRA와 유사하게 환경 변수를 지원하지만, 브라우저에 노출할 모든 변수에 `NEXT_PUBLIC_` 접두사가 **필수**입니다:

```
# Before
REACT_APP_API_URL=...

# After
NEXT_PUBLIC_API_URL=...
```

### Step 10: `package.json` 스크립트 업데이트

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "npx serve@latest ./build"
  }
}
```

`.gitignore`에 추가:

```txt
# ...
.next
next-env.d.ts
```

실행:

```bash
npm run dev
```

[http://localhost:3000](http://localhost:3000)을 열어 애플리케이션이 Next.js에서 실행되는 것을 확인하세요.

### Step 11: 정리

CRA 특정 아티팩트 제거:

- `public/index.html`
- `src/index.tsx`
- `src/react-app-env.d.ts`
- `reportWebVitals` 설정
- `react-scripts` 의존성 (package.json에서 제거)

---

## 추가 고려 사항

### CRA의 커스텀 `homepage` 사용

CRA `package.json`의 `homepage` 필드를 사용하여 특정 하위 경로 아래에서 앱을 서빙한 경우, `next.config.ts`의 `basePath` 설정으로 Next.js에서 복제 가능:

```ts
import { NextConfig } from 'next'

const nextConfig: NextConfig = {
  basePath: '/my-subpath',
  // ...
}

export default nextConfig
```

### 커스텀 Service Worker 처리

CRA의 Service Worker 사용 시, Next.js로 Progressive Web Applications (PWA) 생성 방법을 학습할 수 있습니다.

### API 요청 프록시

CRA 앱이 요청을 백엔드 서버로 전달하기 위해 `package.json`의 `proxy` 필드를 사용한 경우, `next.config.ts`의 Next.js 리라이트로 복제 가능:

```ts
import { NextConfig } from 'next'

const nextConfig: NextConfig = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://your-backend.com/:path*',
      },
    ]
  },
}
```

### 커스텀 Webpack

CRA에서 커스텀 webpack 또는 Babel 설정이 있는 경우, `next.config.ts`에서 Next.js 설정을 확장할 수 있습니다:

```ts
import { NextConfig } from 'next'

const nextConfig: NextConfig = {
  webpack: (config, { isServer }) => {
    // 여기서 webpack 설정 수정
    return config
  },
}

export default nextConfig
```

---

## 번들러 호환성

CRA는 webpack을 사용합니다. Next.js는 이제 로컬 개발을 위해 기본적으로 Turbopack을 사용합니다:

```bash
next dev  # Turbopack 기본 사용
```

Webpack 사용 (CRA와 유사):

```bash
next dev --webpack
```

---

## 다음 단계

모든 작업이 성공했다면, SPA로 실행 중인 기능적 Next.js 애플리케이션을 보유하게 됩니다. 아직 SSR이나 파일 기반 라우팅 같은 Next.js 기능을 활용하지 않지만, 이제 점진적으로 가능합니다:

* **React Router에서 마이그레이션**: Next.js App Router로 이동
  - 자동 코드 분할
  - 스트리밍 서버 렌더링
  - React Server Components
* **이미지 최적화**: `<Image>` 컴포넌트 사용
* **폰트 최적화**: `next/font` 사용
* **타사 스크립트 최적화**: `<Script>` 컴포넌트 사용
* **ESLint 활성화**: Next.js 권장 규칙 사용

**참고**: 정적 내보내기 (`output: 'export'`) 사용 시 현재 `useParams` 훅이나 기타 서버 기능을 지원하지 않습니다. 모든 Next.js 기능을 사용하려면 `next.config.ts`에서 `output: 'export'`를 제거하세요.
