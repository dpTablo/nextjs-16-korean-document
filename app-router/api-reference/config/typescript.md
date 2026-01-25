# TypeScript 설정

Next.js는 React 애플리케이션을 구축하기 위한 TypeScript 우선 개발 경험을 제공합니다.

## TypeScript 시작하기

### 설정 방법

- **새 프로젝트**: `create-next-app` 사용 시 TypeScript가 자동으로 구성됩니다
- **기존 프로젝트**: 파일 이름을 `.ts` / `.tsx`로 변경하고, `next dev` 및 `next build`를 실행하면 종속성이 자동 설치되고 `tsconfig.json`이 생성됩니다

> **마이그레이션 팁**: `jsconfig.json`에서 마이그레이션하는 경우, `paths` 컴파일러 옵션을 새 `tsconfig.json`에 복사하고 이전 파일을 삭제하세요.

---

## IDE 플러그인

VS Code에서 Next.js의 커스텀 TypeScript 플러그인을 활성화합니다:

1. 명령 팔레트 열기 (`Ctrl/⌘` + `Shift` + `P`)
2. "TypeScript: Select TypeScript Version" 검색
3. "Use Workspace Version" 선택

### 플러그인 기능

- 세그먼트 설정 옵션 값 검증
- 컨텍스트 내 문서와 함께 사용 가능한 옵션 표시
- 올바른 `'use client'` 지시어 사용 보장
- 클라이언트 전용 훅(예: `useState`)이 클라이언트 컴포넌트에서 사용되는지 검증

---

## 엔드투엔드 타입 안전성

### 주요 기능

**1. 서버와 클라이언트 간 직렬화 없음**

```tsx
// app/page.tsx
async function getData() {
  const res = await fetch('https://api.example.com/...')
  // 반환 값이 직렬화되지 않음
  // Date, Map, Set 등을 반환할 수 있음
  return res.json()
}

export default async function Page() {
  const name = await getData()
  return '...'
}
```

**2. 간소화된 데이터 흐름**

- 루트 레이아웃을 위해 `_app` 제거
- 코로케이션된 데이터 페칭으로 서버/클라이언트 경계 타이핑 문제 제거
- TypeScript를 지원하는 모든 데이터베이스 또는 ORM과 작동

---

## 라우트 인식 타입 헬퍼

Next.js는 `next dev`, `next build` 또는 `next typegen` 중에 전역 헬퍼를 자동 생성합니다 (import 불필요):

- `PageProps`
- `LayoutProps`
- `RouteContext`

---

## `next-env.d.ts`

자동 생성 파일로 다음을 수행합니다:

- Next.js 타입 정의 참조
- TypeScript가 비코드 import(이미지, 스타일시트)를 인식하도록 활성화
- `next dev`, `next build` 또는 `next typegen`에서 재생성

**모범 사례**:
- `.gitignore`에 추가
- `tsconfig.json`의 `include` 배열에 포함 필수 (`create-next-app`에서 자동)

---

## 설정 예제

### Next.js 설정 파일의 타입 검사

**`next.config.ts` 사용** (TypeScript 프로젝트):

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  /* 설정 옵션 */
}

export default nextConfig
```

**`next.config.js` 사용** (JSDoc 타입 검사):

```js
// next.config.js
// @ts-check

/** @type {import('next').NextConfig} */
const nextConfig = {
  /* 설정 옵션 */
}

module.exports = nextConfig
```

### Node.js 네이티브 TypeScript 리졸버 (v22.10.0+)

최상위 `await` 및 동적 `import()`를 포함한 ESM 구문을 지원합니다.

**CommonJS 프로젝트용** (`.mts` 사용):

```ts
// next.config.mts
import type { NextConfig } from 'next'

const flags = await import('./flags.js').then((m) => m.default ?? m)

const nextConfig: NextConfig = {
  typedRoutes: Boolean(flags?.typedRoutes),
}

export default nextConfig
```

**ESM 프로젝트용** (`package.json`에서 `"type": "module"` 설정):

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  /* 설정 옵션 */
}

export default nextConfig
```

---

## 정적으로 타입된 링크

`next/link` 및 `next/navigation` 라우트에 대한 자동 타입 검사를 활성화합니다.

### 설정

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typedRoutes: true,
}

export default nextConfig
```

### `tsconfig.json` 업데이트

```json
{
  "include": [
    "next-env.d.ts",
    ".next/types/**/*.ts",
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": ["node_modules"]
}
```

### 사용 예제

```tsx
// app/example-client.tsx
'use client'

import type { Route } from 'next'
import Link from 'next/link'
import { useRouter } from 'next/navigation'

export default function Example() {
  const router = useRouter()
  const slug = 'nextjs'

  return (
    <>
      {/* Link: 리터럴 및 동적 */}
      <Link href="/about" />
      <Link href={`/blog/${slug}`} />
      <Link href={('/blog/' + slug) as Route} />
      {/* href가 유효한 라우트가 아니면 TypeScript 에러 */}
      <Link href="/aboot" />

      {/* Router: 리터럴 및 동적 문자열 검증 */}
      <button onClick={() => router.push('/about')}>About으로 이동</button>
      <button onClick={() => router.replace(`/blog/${slug}`)}>
        Blog로 대체
      </button>
      <button onClick={() => router.prefetch('/contact')}>
        Contact 프리페치
      </button>

      {/* 비리터럴 문자열의 경우 Route로 캐스트 */}
      <button onClick={() => router.push(('/blog/' + slug) as Route)}>
        비리터럴 Blog로 이동
      </button>
    </>
  )
}
```

### 제네릭을 사용한 커스텀 Link 컴포넌트

```tsx
import type { Route } from 'next'
import Link from 'next/link'

function Card<T extends string>({ href }: { href: Route<T> | URL }) {
  return (
    <Link href={href}>
      <div>My Card</div>
    </Link>
  )
}
```

---

## 환경 변수의 타입 IntelliSense

로드된 환경 변수 타입으로 `.d.ts` 파일을 자동 생성합니다.

### 설정

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    typedEnv: true,
  },
}

export default nextConfig
```

> **참고**: 타입은 개발 런타임에서 생성됩니다. 프로덕션 변수를 포함하려면 다음을 실행하세요:
> ```bash
> NODE_ENV=production next dev
> ```

---

## 고급 설정

### 비동기 서버 컴포넌트

TypeScript 5.1.3+ 및 `@types/react` 18.2.8+ 필요. `'Promise<Element>' is not a valid JSX element` 에러를 해결합니다.

### 증분 타입 검사

v10.2.1부터 지원. `tsconfig.json`에서 활성화하여 타입 검사 속도를 높입니다:

```json
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

const nextConfig: NextConfig = {
  typescript: {
    tsconfigPath: 'tsconfig.build.json',
  },
}

export default nextConfig
```

**환경별 조건부**:

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

---

## 프로덕션에서 TypeScript 에러 비활성화

### 설정

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typescript: {
    // !! 경고 !!
    // 프로젝트에 타입 에러가 있어도 프로덕션 빌드가
    // 성공적으로 완료되도록 위험하게 허용합니다.
    // !! 경고 !!
    ignoreBuildErrors: true,
  },
}

export default nextConfig
```

### 모범 사례 - 수동 타입 검사

```bash
tsc --noEmit
```

배포 전 에러를 확인하기 위한 CI/CD 파이프라인에 유용합니다.

---

## 커스텀 타입 선언

자동 생성된 `next-env.d.ts`를 수정하지 마세요. 대신 새 `.d.ts` 파일을 만드세요:

```json
{
  "compilerOptions": {
    "skipLibCheck": true
  },
  "include": [
    "new-types.d.ts",
    "next-env.d.ts",
    ".next/types/**/*.ts",
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": ["node_modules"]
}
```

---

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| `v15.0.0` | TypeScript 프로젝트용 `next.config.ts` 지원 추가 |
| `v13.2.0` | 정적으로 타입된 링크 베타 제공 |
| `v12.0.0` | TypeScript/TSX 컴파일에 기본적으로 SWC 사용 |
| `v10.2.1` | 증분 타입 검사 지원 추가 |

---

## 참고

- [TypeScript 가이드](/app-router/guides/typescript.md)
- [next.config.js typescript 옵션](/app-router/api-reference/config/next-config-js/typescript.md)
- [typedRoutes 옵션](/app-router/api-reference/config/next-config-js/typedRoutes.md)
