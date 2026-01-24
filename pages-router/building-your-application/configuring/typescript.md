# TypeScript

Next.js는 React 애플리케이션 구축을 위한 TypeScript 우선 개발 경험을 제공합니다.

## 시작하기

`create-next-app`으로 프로젝트를 생성하면 TypeScript가 자동으로 설정됩니다.

기존 프로젝트에 TypeScript를 추가하려면 파일을 `.ts`/`.tsx`로 변경하고 `next dev` 또는 `next build`를 실행하세요. Next.js가 자동으로 필요한 설정을 추가합니다.

## next-env.d.ts

Next.js는 프로젝트 루트에 `next-env.d.ts` 파일을 자동 생성합니다. 이 파일은:

- Next.js 타입 정의를 참조합니다
- 이미지, 스타일시트 등 비코드 임포트를 인식합니다
- `next dev`, `next build`, `next typegen` 실행 시 재생성됩니다

**주의사항:**
- `.gitignore`에 추가하는 것을 권장합니다
- `tsconfig.json`의 `include` 배열에 포함되어야 합니다

## next.config.ts 사용

TypeScript로 Next.js 설정 파일을 작성할 수 있습니다:

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  /* config options here */
}

export default nextConfig
```

### JavaScript에서 JSDoc 타입 체크

```js
// next.config.js
// @ts-check

/** @type {import('next').NextConfig} */
const nextConfig = {
  /* config options here */
}

module.exports = nextConfig
```

## 정적 타입 링크

`Link` 컴포넌트에서 잘못된 경로를 방지하기 위해 정적 타입 링크를 활성화할 수 있습니다.

### 설정

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typedRoutes: true,
}

export default nextConfig
```

### tsconfig.json 업데이트

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
'use client'

import type { Route } from 'next'
import Link from 'next/link'
import { useRouter } from 'next/navigation'

export default function Example() {
  const router = useRouter()
  const slug = 'nextjs'

  return (
    <>
      {/* ✅ 유효한 라우트 */}
      <Link href="/about" />
      <Link href={`/blog/${slug}`} />

      {/* ❌ TypeScript 에러: 유효하지 않은 라우트 */}
      <Link href="/aboot" />

      {/* Router 사용 */}
      <button onClick={() => router.push('/about')}>About</button>
    </>
  )
}
```

## 환경 변수 타입 IntelliSense

환경 변수에 대한 타입 지원을 활성화할 수 있습니다:

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

## 데이터 페칭 함수 타입

```tsx
// pages/blog/[slug].tsx
import type { GetStaticProps, GetStaticPaths, GetServerSideProps } from 'next'

export const getStaticProps = (async (context) => {
  // ...
}) satisfies GetStaticProps

export const getStaticPaths = (async () => {
  // ...
}) satisfies GetStaticPaths

export const getServerSideProps = (async (context) => {
  // ...
}) satisfies GetServerSideProps
```

## API 라우트 타입

### 기본 예제

```ts
// pages/api/hello.ts
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  res.status(200).json({ name: 'John Doe' })
}
```

### 응답 데이터 타이핑

```ts
// pages/api/hello.ts
import type { NextApiRequest, NextApiResponse } from 'next'

type Data = {
  name: string
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<Data>
) {
  res.status(200).json({ name: 'John Doe' })
}
```

## 커스텀 App 타입

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'

export default function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

## 증분 타입 체크

v10.2.1부터 `tsconfig.json`에서 증분 타입 체크를 활성화할 수 있습니다. 대규모 애플리케이션에서 성능이 향상됩니다.

## 커스텀 tsconfig 경로

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

### 프로덕션 빌드용 조건부 설정

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

## 프로덕션에서 TypeScript 에러 무시

> **경고**: 이 옵션은 프로덕션에서 타입 에러가 있어도 빌드를 완료합니다. 위험하므로 주의해서 사용하세요.

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typescript: {
    // !! 경고 !!
    // 타입 에러가 있어도 프로덕션 빌드 완료
    ignoreBuildErrors: true,
  },
}

export default nextConfig
```

CI/CD에서 별도로 에러를 체크하세요:

```bash
tsc --noEmit
```

## 커스텀 타입 선언

`next-env.d.ts`는 자동 생성되므로 직접 수정하지 마세요. 대신 새 파일을 생성합니다:

```json
// tsconfig.json
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

## 권장 tsconfig.json 설정

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": ["node_modules"]
}
```
