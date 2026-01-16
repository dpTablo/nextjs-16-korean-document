# Vite에서 Next.js로 마이그레이션하는 방법

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-10-17

## 개요

이 가이드는 기존 Vite 애플리케이션을 Next.js로 마이그레이션하는 방법을 설명합니다.

---

## 전환 이유

### 느린 초기 페이지 로딩 시간
- 기본 Vite React 플러그인은 순수 클라이언트 사이드(SPA) 애플리케이션 생성
- 브라우저가 React 코드와 전체 애플리케이션 번들을 다운로드하고 실행할 때까지 기다려야 함
- 새로운 기능과 의존성 추가에 따라 애플리케이션 코드 증가

### 자동 코드 분할 없음
- 수동 코드 분할 시 네트워크 폭포 현상 발생 가능
- Next.js는 라우터에 내장된 자동 코드 분할 제공

### 네트워크 폭포 현상
- SPA는 컴포넌트 마운트 후 플레이스홀더를 렌더링한 다음 데이터를 페치
- 자식 컴포넌트는 부모 컴포넌트의 로딩 완료 후에만 데이터 페치 가능
- Next.js는 서버에서 데이터 페칭하여 클라이언트-서버 폭포 현상 제거

### 빠르고 의도적인 로딩 상태
- React Suspense를 통한 스트리밍 지원
- UI의 어떤 부분이 먼저 로드되는지 제어 가능
- 더 빠르게 로드되는 페이지 구축 및 레이아웃 시프트 제거

### 데이터 페칭 전략 선택
- 빌드 시, 서버의 요청 시, 또는 클라이언트에서 페치
- 예: 빌드 시 CMS에서 페치하고 CDN에 캐시

### 프록시
- 요청 완료 전 서버에서 코드 실행 가능
- 인증되지 않은 콘텐츠 플래시 방지를 위해 로그인으로 리다이렉트
- 실험 및 국제화에 유용

### 내장 최적화
- 이미지, 폰트, 타사 스크립트 자동 최적화

---

## 마이그레이션 단계

### Step 1: Next.js 의존성 설치

```bash
npm install next@latest
```

### Step 2: Next.js 설정 파일 생성

루트에 `next.config.mjs` 생성:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export', // Single-Page Application (SPA) 출력
  distDir: './dist', // 빌드 출력 디렉터리를 `./dist/`로 변경
}

export default nextConfig
```

> Next.js 설정 파일에는 `.js` 또는 `.mjs`를 사용할 수 있습니다.

### Step 3: TypeScript 설정 업데이트

TypeScript를 사용하는 경우 `tsconfig.json` 업데이트:

1. `tsconfig.node.json`에 대한 프로젝트 참조 제거
2. `./dist/types/**/*.ts`와 `./next-env.d.ts`를 `include` 배열에 추가
3. `./node_modules`를 `exclude` 배열에 추가
4. `compilerOptions`의 `plugins` 배열에 `{ "name": "next" }` 추가
5. `esModuleInterop`을 `true`로 설정
6. `jsx`를 `react-jsx`로 설정
7. `allowJs`를 `true`로 설정
8. `forceConsistentCasingInFileNames`를 `true`로 설정
9. `incremental`을 `true`로 설정

예시 `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "allowJs": true,
    "forceConsistentCasingInFileNames": true,
    "incremental": true,
    "plugins": [{ "name": "next" }]
  },
  "include": ["./src", "./dist/types/**/*.ts", "./next-env.d.ts"],
  "exclude": ["./node_modules"]
}
```

### Step 4: 루트 레이아웃 생성

루트 레이아웃 파일은 모든 페이지를 감싸며 React Server Component입니다.

1. `src` 폴더에 새 `app` 디렉터리 생성
2. 해당 `app` 디렉터리 안에 새 `layout.tsx` 파일 생성:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return '...'
}
```

3. `index.html`의 내용을 `<RootLayout>` 컴포넌트로 복사:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <head>
        <link rel="icon" type="image/svg+xml" href="/icon.svg" />
        <title>My App</title>
        <meta name="description" content="My App is a..." />
      </head>
      <body>
        <div id="root">{children}</div>
      </body>
    </html>
  )
}
```

4. Next.js는 `meta charset`과 `meta viewport`를 기본으로 포함하므로 제거:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <head>
        <link rel="icon" type="image/svg+xml" href="/icon.svg" />
        <title>My App</title>
        <meta name="description" content="My App is a..." />
      </head>
      <body>
        <div id="root">{children}</div>
      </body>
    </html>
  )
}
```

5. 메타데이터 파일 (`favicon.ico`, `icon.png`, `robots.txt`)을 `app` 디렉터리로 이동하고 `<link>` 태그 제거

6. 내보낸 `metadata` 객체와 함께 Metadata API 사용:

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'My App',
  description: 'My App is a...',
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

### Step 5: 엔트리포인트 페이지 생성

1. `app` 디렉터리에 `[[...slug]]` 디렉터리 생성 (선택적 캐치올 라우트 세그먼트)

2. `app/[[...slug]]/page.tsx` 생성:

```tsx
import '../../index.css'

export function generateStaticParams() {
  return [{ slug: [''] }]
}

export default function Page() {
  return '...' // 나중에 업데이트
}
```

3. Vite 앱을 감싸는 클라이언트 컴포넌트 생성 (`app/[[...slug]]/client.tsx`):

```tsx
'use client'

import React from 'react'
import dynamic from 'next/dynamic'

const App = dynamic(() => import('../../App'), { ssr: false })

export function ClientOnly() {
  return <App />
}
```

4. 새 컴포넌트를 사용하도록 페이지 업데이트:

```tsx
import '../../index.css'
import { ClientOnly } from './client'

export function generateStaticParams() {
  return [{ slug: [''] }]
}

export default function Page() {
  return <ClientOnly />
}
```

### Step 6: 정적 이미지 임포트 업데이트

**Vite 동작:** 이미지 임포트가 URL 문자열 반환
```tsx
import image from './img.png' // '/assets/img.2d8efhg.png' 반환
export default function App() {
  return <img src={image} />
}
```

**Next.js 동작:** 이미지 임포트가 객체 반환

1. 절대 임포트 경로를 상대 경로로 변환:

```tsx
// Before
import logo from '/logo.png'

// After
import logo from '../public/logo.png'
```

2. `<img>` 태그에 `src` 속성 전달:

```tsx
// Before
<img src={logo} />

// After
<img src={logo.src} />
```

또는 파일명으로 공개 URL 참조: `public/logo.png`는 `/logo.png`에서 제공됩니다.

### Step 7: 환경 변수 마이그레이션

**주요 차이점:**
- `VITE_` 접두사를 `NEXT_PUBLIC_`으로 변경
- `import.meta.env` 참조 업데이트:

| Vite | Next.js |
|------|---------|
| `import.meta.env.MODE` | `process.env.NODE_ENV` |
| `import.meta.env.PROD` | `process.env.NODE_ENV === 'production'` |
| `import.meta.env.DEV` | `process.env.NODE_ENV !== 'production'` |
| `import.meta.env.SSR` | `typeof window !== 'undefined'` |

**BASE_URL의 경우:**

1. `.env`에 추가:

```bash
NEXT_PUBLIC_BASE_PATH="/some-base-path"
```

2. `next.config.mjs` 업데이트:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  distDir: './dist',
  basePath: process.env.NEXT_PUBLIC_BASE_PATH,
}

export default nextConfig
```

3. `import.meta.env.BASE_URL` 사용을 `process.env.NEXT_PUBLIC_BASE_PATH`로 업데이트

### Step 8: `package.json` 스크립트 업데이트

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

`.gitignore` 업데이트:

```txt
# ...
.next
next-env.d.ts
dist
```

`npm run dev`를 실행하고 `http://localhost:3000`을 열어 애플리케이션이 Next.js에서 실행되는 것을 확인하세요.

### Step 9: 정리

Vite 관련 파일 삭제:
- `main.tsx`
- `index.html`
- `vite-env.d.ts`
- `tsconfig.node.json`
- `vite.config.ts`

Vite 의존성 제거.

---

## 다음 단계

마이그레이션 성공 후 Next.js 기능을 점진적으로 도입할 수 있습니다:

1. **React Router에서 Next.js App Router로 마이그레이션:**
   - 자동 코드 분할
   - 스트리밍 서버 렌더링
   - React Server Components

2. **이미지 최적화**: `<Image>` 컴포넌트 사용

3. **폰트 최적화**: `next/font` 사용

4. **타사 스크립트 최적화**: `<Script>` 컴포넌트 사용

5. **ESLint 설정 업데이트**: Next.js 규칙 지원
