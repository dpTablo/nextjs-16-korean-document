# TypeScript

Next.js에서 TypeScript를 설정하고 사용하는 방법을 알아봅니다.

## 빠른 설정

### 새 프로젝트

Next.js는 `create-next-app`으로 프로젝트를 생성할 때 자동으로 TypeScript를 구성합니다:

```bash
npx create-next-app@latest
```

설치 과정에서 TypeScript 사용 여부를 선택할 수 있습니다.

### 기존 프로젝트

기존 프로젝트에 TypeScript를 추가하려면:

1. 파일 이름을 `.ts` / `.tsx`로 변경합니다
2. `next dev`와 `next build`를 실행하여 의존성을 자동 설치하고 `tsconfig.json`을 생성합니다

```bash
# 의존성 자동 설치 및 설정
npm run dev
```

> **마이그레이션 팁:** `jsconfig.json`이 있다면, `paths` 컴파일러 옵션을 새 `tsconfig.json`으로 복사하고 이전 파일을 삭제하세요.

---

## IDE TypeScript 플러그인

Next.js에는 VSCode 및 기타 편집기용 커스텀 TypeScript 플러그인이 포함되어 있습니다.

### VS Code에서 활성화

1. 명령 팔레트 열기 (`Ctrl/⌘ + Shift + P`)
2. "TypeScript: Select TypeScript Version" 검색
3. "Use Workspace Version" 선택

### 플러그인 기능

- **세그먼트 설정 옵션 검증**
- **컨텍스트 내 문서와 함께 자동 완성**
- **`'use client'` 지시어 사용 확인**
- **클라이언트 훅이 클라이언트 컴포넌트에만 나타나는지 확인**

**예시:**
```tsx
// ❌ TypeScript 플러그인이 오류 표시
export const dynamic = 'invalid-value'

// ✅ 올바른 값
export const dynamic = 'force-dynamic'
```

---

## End-to-End 타입 안전성

### 주요 기능

#### 1. 서버와 클라이언트 간 직렬화 불필요

Server Components에서 직접 데이터를 페치하며 데이터를 직렬화할 필요가 없습니다:

```tsx
// app/page.tsx
async function getData() {
  const res = await fetch('https://api.example.com/data')
  return res.json() // 직렬화 필요 없음
}

export default async function Page() {
  const data = await getData()

  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.description}</p>
    </div>
  )
}
```

네이티브 타입 사용:
```tsx
async function getData() {
  return {
    date: new Date(), // ✅ Date 객체 그대로 사용
    map: new Map([['key', 'value']]), // ✅ Map 사용
    set: new Set([1, 2, 3]), // ✅ Set 사용
  }
}

export default async function Page() {
  const data = await getData()
  console.log(data.date.toISOString()) // 타입 안전
  return <div>...</div>
}
```

#### 2. 간소화된 데이터 흐름

- 루트 레이아웃이 `_app`을 대체하여 더 명확한 데이터 흐름
- 공동 배치된 데이터 페칭으로 타입 안전성 격차 제거

---

## 라우트 인식 타입 헬퍼

Next.js는 자동으로 전역 헬퍼를 생성합니다 (import 불필요):

- `PageProps` - 페이지 컴포넌트의 props 타입
- `LayoutProps` - 레이아웃 컴포넌트의 props 타입
- `RouteContext` - 라우트 컨텍스트 타입

### 사용 예시

```tsx
// app/blog/[slug]/page.tsx
export default async function Page({
  params,
  searchParams,
}: PageProps<{ slug: string }>) {
  const { slug } = await params
  const search = await searchParams

  return (
    <div>
      <h1>Post: {slug}</h1>
    </div>
  )
}
```

**생성 시점:** `next dev`, `next build`, 또는 `next typegen` 실행 시

---

## `next-env.d.ts`

Next.js가 자동으로 생성하는 파일로:

- Next.js 타입 정의 참조
- TypeScript가 비코드 import(이미지, 스타일)를 인식하도록 설정
- Next.js 관련 타입 포함

**app/page.tsx**
```tsx
import Image from 'next/image'
import logo from './logo.png' // ✅ TypeScript가 이미지 import 인식

export default function Page() {
  return <Image src={logo} alt="Logo" />
}
```

### 베스트 프랙티스

- `.gitignore`에 추가 (자동 생성됨)
- `tsconfig.json`의 `include` 배열에 포함되어야 함

```json
// tsconfig.json
{
  "include": [
    "next-env.d.ts", // ← 필수
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ]
}
```

---

## 설정 예시

### 타입 안전한 Next.js 설정

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactStrictMode: true,
  images: {
    domains: ['example.com'],
  },
}

export default nextConfig
```

### CommonJS 프로젝트 (Node.js v22.10.0+)

`.next.config.mts` 사용:

```ts
// next.config.mts
import type { NextConfig } from 'next'

const flags = await import('./flags.js').then((m) => m.default ?? m)

const nextConfig: NextConfig = {
  typedRoutes: Boolean(flags?.typedRoutes),
  experimental: {
    reactCompiler: Boolean(flags?.reactCompiler),
  },
}

export default nextConfig
```

---

## 정적 타입 링크 (Typed Routes)

오타 및 네비게이션 오류를 방지하는 타입 안전 라우팅:

### 활성화

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typedRoutes: true,
}

export default nextConfig
```

**tsconfig.json 업데이트:**
```json
{
  "include": [
    "next-env.d.ts",
    ".next/types/**/*.ts",
    "**/*.ts",
    "**/*.tsx"
  ]
}
```

### 사용 예시

```tsx
// app/example-client.tsx
'use client'

import type { Route } from 'next'
import Link from 'next/link'
import { useRouter } from 'next/navigation'

export default function Example() {
  const router = useRouter()

  return (
    <>
      {/* ✅ 유효한 라우트 */}
      <Link href="/about">About</Link>
      <Link href="/blog/nextjs">Blog Post</Link>

      {/* ❌ TypeScript 오류 - 존재하지 않는 라우트 */}
      {/* <Link href="/aboot">About</Link> */}

      <button onClick={() => router.push('/about')}>
        About으로 이동
      </button>

      <button onClick={() => router.replace('/blog/nextjs')}>
        블로그 교체
      </button>

      <button onClick={() => router.prefetch('/contact')}>
        Contact 프리페치
      </button>
    </>
  )
}
```

### 커스텀 컴포넌트에서 제네릭 사용

```tsx
import type { Route } from 'next'
import Link from 'next/link'

function Card<T extends string>({
  href,
}: {
  href: Route<T> | URL
}) {
  return (
    <Link href={href}>
      <div>My Card</div>
    </Link>
  )
}

// 사용
export default function Page() {
  return (
    <>
      <Card href="/about" />
      <Card href="/blog/typescript" />
    </>
  )
}
```

---

## 환경 변수에 대한 타입 IntelliSense

환경 변수 타입을 가진 `.d.ts` 파일 생성:

### 활성화

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    typedEnv: true,
  },
}

export default nextConfig
```

### 사용 예시

```tsx
// 타입 안전한 환경 변수
const apiUrl = process.env.NEXT_PUBLIC_API_URL // ✅ 타입 추론됨
const apiKey = process.env.API_SECRET_KEY // ✅ 타입 안전
```

> **참고:** 변수는 개발 런타임을 기반으로 생성됩니다. 프로덕션 변수를 포함하려면 `NODE_ENV=production next dev`를 실행하세요.

---

## 고급 설정

### 증분 타입 검사

대규모 애플리케이션에서 타입 검사 속도 향상:

```json
// tsconfig.json
{
  "compilerOptions": {
    "incremental": true
  }
}
```

### 커스텀 `tsconfig` 경로

```ts
// next.config.ts
import type { NextConfig } from 'next'

const isProd = process.env.NODE_ENV === 'production'

const nextConfig: NextConfig = {
  typescript: {
    tsconfigPath: isProd ? 'tsconfig.build.json' : 'tsconfig.json',
  },
}

export default nextConfig
```

**사용 사례:**
- 모노레포 설정
- CI 마이그레이션 중 검사 완화

### 커스텀 타입 선언

자동 생성된 `next-env.d.ts` 대신 새 파일 생성:

**new-types.d.ts**
```ts
// 커스텀 타입 선언
declare module '*.svg' {
  const content: React.FunctionComponent<React.SVGAttributes<SVGElement>>
  export default content
}

declare module '*.css' {
  const content: { [className: string]: string }
  export default content
}
```

**tsconfig.json**
```json
{
  "include": [
    "new-types.d.ts",
    "next-env.d.ts",
    ".next/types/**/*.ts",
    "**/*.ts",
    "**/*.tsx"
  ]
}
```

---

## 오류 처리

### 프로덕션에서 TypeScript 오류 비활성화

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typescript: {
    // ⚠️ 프로덕션 빌드에서 타입 오류 무시 (위험)
    ignoreBuildErrors: true,
  },
}

export default nextConfig
```

### 더 안전한 접근 방식

빌드 전 타입 검사:

```bash
# 타입 검사만 수행
tsc --noEmit

# 성공 시 빌드
npm run build
```

**package.json에 스크립트 추가:**
```json
{
  "scripts": {
    "type-check": "tsc --noEmit",
    "build:safe": "npm run type-check && next build"
  }
}
```

---

## 베스트 프랙티스

### 1. Strict Mode 사용

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true
  }
}
```

### 2. 타입 가드 활용

```tsx
type User = {
  id: number
  name: string
  email?: string
}

function isValidUser(user: unknown): user is User {
  return (
    typeof user === 'object' &&
    user !== null &&
    'id' in user &&
    'name' in user
  )
}

// 사용
const data = await fetch('/api/user').then(r => r.json())
if (isValidUser(data)) {
  console.log(data.name) // ✅ 타입 안전
}
```

### 3. 제네릭 컴포넌트

```tsx
type ListProps<T> = {
  items: T[]
  renderItem: (item: T) => React.ReactNode
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  )
}

// 사용
<List
  items={[1, 2, 3]}
  renderItem={(num) => <span>{num * 2}</span>}
/>
```

---

## 요구 사항

- **TypeScript 5.1.3 이상** - Async Server Components를 위해 필요
- **@types/react 18.2.8 이상** - Async Server Components를 위해 필요
- **Node.js v22.10.0 이상** - `next.config.ts`에서 네이티브 ESM 사용

### 설치

```bash
npm install --save-dev typescript @types/react @types/node
```

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|-----------|
| v15.0.0 | TypeScript 프로젝트를 위한 `next.config.ts` 지원 |
| v13.2.0 | 정적 타입 링크 (베타) |
| v12.0.0 | 더 빠른 TypeScript 빌드를 위한 SWC 컴파일러 |
| v10.2.1 | 증분 타입 검사 지원 |

---

## 다음 단계

- [ESLint](https://nextjs.org/docs/app/building-your-application/configuring/eslint) - 코드 품질 관리
- [Environment Variables](./environment-variables.md) - 환경 변수 설정
- [Next.js Compiler](https://nextjs.org/docs/architecture/nextjs-compiler) - SWC 컴파일러

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
