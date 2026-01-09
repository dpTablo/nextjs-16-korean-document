# 설치

**버전:** 16.1.1
**최종 업데이트:** 2025-12-09

## 빠른 시작

### 패키지 매니저별 명령어

**pnpm:**
```bash
pnpm create next-app@latest my-app --yes
cd my-app
pnpm dev
```

**npm:**
```bash
npx create-next-app@latest my-app --yes
cd my-app
npm run dev
```

**yarn:**
```bash
yarn create next-app@latest my-app --yes
cd my-app
yarn dev
```

**bun:**
```bash
bun create next-app@latest my-app --yes
cd my-app
bun dev
```

**참고:** `--yes` 플래그는 프롬프트를 건너뛰고 TypeScript, Tailwind, ESLint, App Router, Turbopack을 기본으로 활성화하며 `@/*` import alias를 설정합니다.

## 시스템 요구사항
- **Node.js:** 최소 버전 20.9
- **운영체제:** macOS, Windows (WSL 포함), Linux

## 지원 브라우저
- Chrome 111+
- Edge 111+
- Firefox 111+
- Safari 16.4+

## CLI로 생성하기

```bash
npx create-next-app@latest
```

**설치 프롬프트:**
- 프로젝트 이름
- 권장 기본 설정 (TypeScript, ESLint, Tailwind CSS, App Router, Turbopack)
- 또는 커스터마이징: TypeScript, 린터 (ESLint/Biome/None), React Compiler, Tailwind CSS, src/ 디렉토리, App Router, import alias

## 수동 설치

### 의존성 설치

**pnpm:**
```bash
pnpm i next@latest react@latest react-dom@latest
```

**npm:**
```bash
npm i next@latest react@latest react-dom@latest
```

**yarn:**
```bash
yarn add next@latest react@latest react-dom@latest
```

**bun:**
```bash
bun add next@latest react@latest react-dom@latest
```

### package.json에 스크립트 추가

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "lint:fix": "eslint --fix"
  }
}
```

**스크립트 설명:**
- `next dev` - Turbopack을 사용한 개발 서버 시작 (기본값)
- `next build` - 프로덕션용 빌드
- `next start` - 프로덕션 서버 시작
- `eslint` - ESLint 린터 실행

**참고:** Turbopack이 기본 번들러입니다. `--webpack` 플래그로 번들러를 전환할 수 있습니다.

### App 디렉토리 생성

`app/layout.tsx` 생성 (루트 레이아웃 - 필수):

**TypeScript:**
```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

**JavaScript:**
```jsx
export default function RootLayout({ children }) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

`app/page.tsx` 생성 (홈 페이지):

**TypeScript:**
```tsx
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

**JavaScript:**
```jsx
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

### Public 폴더 생성 (선택사항)

프로젝트 루트에 `public` 폴더를 생성하여 정적 에셋을 저장합니다. 기본 URL(`/`)에서 파일을 참조할 수 있습니다.

**예시:**
```tsx
import Image from 'next/image'

export default function Page() {
  return <Image src="/profile.png" alt="프로필" width={100} height={100} />
}
```

## 개발 서버 실행

1. `npm run dev` 실행
2. `http://localhost:3000` 방문
3. `app/page.tsx`를 편집하고 저장하여 업데이트 확인

## TypeScript 설정

**요구사항:** TypeScript v5.1.0 이상

Next.js는 내장 TypeScript 지원이 있습니다. 파일을 `.ts`/`.tsx`로 변경하고 `next dev`를 실행하면 자동으로 의존성을 설치하고 `tsconfig.json`을 생성합니다.

### VS Code에서 TypeScript 플러그인 활성화

1. 명령 팔레트 열기 (`Ctrl/⌘` + `Shift` + `P`)
2. "TypeScript: Select TypeScript Version" 검색
3. "Use Workspace Version" 선택

## 린팅 설정

### ESLint (포괄적인 규칙)

```json
{
  "scripts": {
    "lint": "eslint",
    "lint:fix": "eslint --fix"
  }
}
```

### Biome (빠른 린터 + 포매터)

```json
{
  "scripts": {
    "lint": "biome check",
    "format": "biome format --write"
  }
}
```

**next lint에서 마이그레이션:**
```bash
npx @next/codemod@canary next-lint-to-eslint-cli .
```

**참고:** Next.js 16부터는 `next build`가 자동으로 린터를 실행하지 않습니다.

## 절대 경로 임포트 및 모듈 별칭

`tsconfig.json` 또는 `jsconfig.json` 설정:

**기본 설정:**
```json
{
  "compilerOptions": {
    "baseUrl": "src/"
  }
}
```

**경로 별칭 사용:**
```json
{
  "compilerOptions": {
    "baseUrl": "src/",
    "paths": {
      "@/styles/*": ["styles/*"],
      "@/components/*": ["components/*"]
    }
  }
}
```

**사용법:**
```jsx
// 이전
import { Button } from '../../../components/button'

// 이후
import { Button } from '@/components/button'
```
