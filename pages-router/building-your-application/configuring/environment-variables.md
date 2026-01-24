# 환경 변수

Next.js는 환경 변수를 위한 내장 지원을 제공합니다:

- `.env` 파일에서 환경 변수 로드
- `NEXT_PUBLIC_` 접두사를 통해 브라우저에 번들링

> **경고**: `.env` 파일은 `.gitignore`에 추가되어야 하며, 저장소에 커밋하지 않아야 합니다.

## 환경 변수 로드

Next.js는 `.env` 파일에서 `process.env`로 환경 변수를 로드하는 것을 내장 지원합니다.

```txt
# .env
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
```

이렇게 하면 `process.env.DB_HOST`, `process.env.DB_USER`, `process.env.DB_PASS`가 Node.js 환경에 자동으로 로드되어 데이터 페칭 메서드에서 사용할 수 있습니다.

```js
// pages/index.js
export async function getStaticProps() {
  const db = await myDB.connect({
    host: process.env.DB_HOST,
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
  })
  // ...
}
```

## @next/env 패키지

Next.js 런타임 외부에서 환경 변수를 로드해야 하는 경우 `@next/env` 패키지를 사용할 수 있습니다:

```bash
npm install @next/env
```

```tsx
import { loadEnvConfig } from '@next/env'

const projectDir = process.cwd()
loadEnvConfig(projectDir)
```

## 변수 참조

`.env` 파일에서 다른 변수를 참조할 수 있습니다:

```txt
TWITTER_USER=nextjs
TWITTER_URL=https://x.com/$TWITTER_USER
```

위 예제에서 `process.env.TWITTER_URL`은 `https://x.com/nextjs`가 됩니다.

> `$` 문자를 실제 값으로 사용하려면 `\$`로 이스케이프하세요.

## 브라우저 번들링 (NEXT_PUBLIC_)

기본적으로 환경 변수는 Node.js 환경에서만 사용 가능하며, 브라우저에 노출되지 않습니다.

브라우저에서 환경 변수를 사용하려면 `NEXT_PUBLIC_` 접두사를 사용하세요:

```txt
NEXT_PUBLIC_ANALYTICS_ID=abcdefghijk
```

```js
// pages/index.js
import setupAnalyticsService from '../lib/my-analytics-service'

// NEXT_PUBLIC_ 접두사가 있어 브라우저에서 사용 가능
setupAnalyticsService(process.env.NEXT_PUBLIC_ANALYTICS_ID)

function HomePage() {
  return <h1>Hello World</h1>
}

export default HomePage
```

### 중요 주의사항

- 값은 빌드 시간에 JavaScript 번들에 하드코딩됩니다.
- 빌드 후 런타임 변경이 반영되지 않습니다.
- 동적 조회는 인라인되지 않습니다:

```js
// ❌ 인라인 안 됨 (변수 사용)
const varName = 'NEXT_PUBLIC_ANALYTICS_ID'
setupAnalyticsService(process.env[varName])

const env = process.env
setupAnalyticsService(env.NEXT_PUBLIC_ANALYTICS_ID)
```

### 런타임 환경 변수

런타임 환경 변수가 필요한 경우:
- `getServerSideProps` 사용
- App Router 채택
- API 엔드포인트로 클라이언트에 제공

## 환경 변수 로드 순서

환경 변수는 다음 순서로 검색됩니다 (찾으면 멈춤):

1. `process.env`
2. `.env.$(NODE_ENV).local`
3. `.env.local` (NODE_ENV=test일 때 제외)
4. `.env.$(NODE_ENV)`
5. `.env`

예를 들어, `NODE_ENV=development`일 때 `.env.development.local`에 정의된 변수가 `.env`보다 우선합니다.

## 환경별 파일

| 파일 | 용도 |
|------|------|
| `.env` | 모든 환경에서 로드 |
| `.env.local` | 로컬 개발용 (저장소에 커밋하지 않음) |
| `.env.development` | 개발 환경용 |
| `.env.production` | 프로덕션 환경용 |
| `.env.test` | 테스트 환경용 |
| `.env.*.local` | 로컬 환경별 설정 (저장소에 커밋하지 않음) |

## 테스트 환경 변수

`.env.test` 파일은 `NODE_ENV=test`일 때 로드됩니다:

- `.env.local`은 로드되지 않음 (일관된 테스트 결과 보장)
- 저장소에 포함되어야 함
- `.env.test.local`은 `.gitignore`에 포함

### Jest 설정 예제

```js
// jest.setup.js
import { loadEnvConfig } from '@next/env'

export default async () => {
  const projectDir = process.cwd()
  loadEnvConfig(projectDir)
}
```

## 권장 .gitignore 설정

```gitignore
# 환경 변수 파일
.env*.local
```

## 주요 정보

- `/src` 디렉토리를 사용하는 경우에도 `.env.*` 파일은 **프로젝트 루트**에 위치해야 합니다.
- `NODE_ENV`가 설정되지 않은 경우:
  - `next dev` → development
  - 다른 명령 → production
