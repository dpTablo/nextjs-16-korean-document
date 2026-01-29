---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/authInterrupts
버전: 16.1.6
---

# authInterrupts

> **참고**: 이 기능은 canary 채널에서 사용 가능하며 변경될 수 있습니다.

## 개요

`authInterrupts` 설정 옵션을 통해 애플리케이션에서 [`forbidden`](/app-router/api-reference/functions/forbidden.md)과 [`unauthorized`](/app-router/api-reference/functions/unauthorized.md) API를 사용할 수 있습니다.

이 함수들은 실험적 기능이므로 사용하려면 `next.config.js` 파일에서 `authInterrupts` 옵션을 활성화해야 합니다.

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    authInterrupts: true,
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    authInterrupts: true,
  },
}

module.exports = nextConfig
```

## 관련 API

이 옵션을 활성화하면 다음 API를 사용할 수 있습니다:

### 함수

- [`forbidden`](/app-router/api-reference/functions/forbidden.md) - 403 Forbidden 응답을 트리거
- [`unauthorized`](/app-router/api-reference/functions/unauthorized.md) - 401 Unauthorized 응답을 트리거

### 파일 규칙

- `forbidden.js` - 403 에러 페이지를 정의하는 특수 파일
- `unauthorized.js` - 401 에러 페이지를 정의하는 특수 파일

## 사용 예제

```tsx filename="app/admin/page.tsx"
import { forbidden } from 'next/navigation'
import { checkUserRole } from '@/lib/auth'

export default async function AdminPage() {
  const role = await checkUserRole()

  if (role !== 'admin') {
    forbidden() // 403 페이지 표시
  }

  return <div>관리자 전용 콘텐츠</div>
}
```

```tsx filename="app/dashboard/page.tsx"
import { unauthorized } from 'next/navigation'
import { getUser } from '@/lib/auth'

export default async function DashboardPage() {
  const user = await getUser()

  if (!user) {
    unauthorized() // 401 페이지 표시
  }

  return <div>대시보드 콘텐츠</div>
}
```
