---
원문: https://nextjs.org/docs/app/guides/environment-variables
버전: 16.1.6
---

# Next.js에서 환경 변수 사용하기

## 개요

Next.js는 다음을 통해 환경 변수에 대한 내장 지원을 제공합니다:
- `.env` 파일에서 `process.env`로 변수 로드
- `NEXT_PUBLIC_` 접두사를 사용하여 브라우저용 변수 번들링

> **경고:** 기본 `create-next-app` 템플릿은 모든 `.env` 파일을 `.gitignore`에 추가합니다. 이러한 파일을 저장소에 커밋하지 마세요.

## 환경 변수 로드하기

Next.js는 `.env*` 파일에서 환경 변수를 자동으로 로드합니다:

```txt filename=".env"
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
```

### 여러 줄 변수

```bash
# .env
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
...
Kh9NV...
...
-----END DSA PRIVATE KEY-----"

# 또는 \n 이스케이프 사용
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\nKh9NV...\n-----END DSA PRIVATE KEY-----\n"
```

> **참고:** Next.js는 `/src` 폴더가 아닌 루트 디렉토리에서만 `.env` 파일을 로드합니다.

### Route Handlers에서 사용

```js filename="app/api/route.js"
export async function GET() {
  const db = await myDB.connect({
    host: process.env.DB_HOST,
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
  })
}
```

## Next.js 런타임 외부에서 변수 로드하기

ORM 설정이나 테스트 러너를 위해 `@next/env` 패키지를 사용하세요:

```bash
npm install @next/env
```

```tsx filename="envConfig.ts"
import { loadEnvConfig } from '@next/env'

const projectDir = process.cwd()
loadEnvConfig(projectDir)
```

```tsx filename="orm.config.ts"
import './envConfig.ts'

export default defineConfig({
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!,
  },
})
```

## 다른 변수 참조하기

```txt filename=".env"
TWITTER_USER=nextjs
TWITTER_URL=https://x.com/$TWITTER_USER
```

`process.env.TWITTER_URL`은 `https://x.com/nextjs`로 해석됩니다

> **팁:** 실제 값에 `$`를 사용하려면 이스케이프하세요: `\$`

## 브라우저용 변수 번들링

### `NEXT_PUBLIC_` 접두사

`NEXT_PUBLIC_`으로 접두사가 붙은 변수만 브라우저에서 접근 가능합니다. 비공개 변수는 서버 사이드에만 남습니다.

```txt filename=".env"
NEXT_PUBLIC_ANALYTICS_ID=abcdefghijk
```

이는 빌드 시점에 값을 인라인합니다:

```js filename="pages/index.js"
import setupAnalyticsService from '../lib/my-analytics-service'

// 빌드 시점에 setupAnalyticsService('abcdefghijk')로 변환됨
setupAnalyticsService(process.env.NEXT_PUBLIC_ANALYTICS_ID)

function HomePage() {
  return <h1>Hello World</h1>
}

export default HomePage
```

### 동적 조회 (인라인되지 않음)

```js
// 이것들은 인라인되지 않습니다
const varName = 'NEXT_PUBLIC_ANALYTICS_ID'
setupAnalyticsService(process.env[varName])

const env = process.env
setupAnalyticsService(env.NEXT_PUBLIC_ANALYTICS_ID)
```

> **중요:** `NEXT_PUBLIC_` 변수는 빌드 시점에 고정됩니다. 런타임 값이 필요하면 백엔드 API를 사용하세요.

## 런타임 환경 변수

서버 사이드 변수는 동적 렌더링 중 런타임에 평가됩니다:

```tsx filename="app/page.ts"
import { connection } from 'next/server'

export default async function Component() {
  await connection()
  // 동적 렌더링 컨텍스트
  const value = process.env.MY_VALUE
}
```

이를 통해 여러 환경에서 단일 Docker 이미지를 프로모션할 수 있습니다.

## 테스트 환경 변수

테스트 전용 설정을 위해 `.env.test`를 사용하세요:

```js
// Jest 전역 설정
import { loadEnvConfig } from '@next/env'

export default async () => {
  const projectDir = process.cwd()
  loadEnvConfig(projectDir)
}
```

**테스트 환경의 주요 차이점:**
- `.env.local`은 일관된 테스트 결과를 위해 로드되지 **않습니다**
- `.env.test`는 커밋되어야 하며 `.env.test.local`은 무시되어야 합니다
- `NODE_ENV=test` 필요

## 환경 변수 로드 순서

변수는 다음 우선순위 순서로 해석됩니다 (첫 번째 일치 우선):

1. `process.env`
2. `.env.$(NODE_ENV).local`
3. `.env.local` (`NODE_ENV`가 `test`이면 건너뜀)
4. `.env.$(NODE_ENV)`
5. `.env`

**예시:** `NODE_ENV=development`이고 `.env.development.local`과 `.env` 모두 변수를 정의하면 `.env.development.local`이 우선합니다.

## 주요 참고사항

- `.env.*` 파일은 `/src` 디렉토리가 있어도 프로젝트 루트에 있어야 합니다
- 허용되는 `NODE_ENV` 값: `production`, `development`, `test`
- `next dev`는 기본적으로 `development`, 다른 명령은 `production`이 기본값

## 버전 히스토리

| 버전 | 변경사항 |
|---------|---------|
| v9.4.0 | `.env` 및 `NEXT_PUBLIC_` 지원 도입 |
