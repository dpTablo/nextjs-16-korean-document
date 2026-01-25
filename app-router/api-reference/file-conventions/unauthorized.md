# unauthorized.js

**unauthorized.js**는 인증 과정에서 `unauthorized()` 함수가 호출될 때 사용자 정의 UI를 렌더링하는 데 사용되는 특수 Next.js 파일 규칙입니다. Next.js가 자동으로 `401` 상태 코드를 반환합니다.

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서의 사용은 권장하지 않습니다.

## 기본 구현

```tsx
// app/unauthorized.tsx
import Login from '@/app/components/Login'

export default function Unauthorized() {
  return (
    <main>
      <h1>401 - 인증되지 않음</h1>
      <p>이 페이지에 접근하려면 로그인해 주세요.</p>
      <Login />
    </main>
  )
}
```

### JavaScript

```jsx
// app/unauthorized.jsx
import Login from '@/app/components/Login'

export default function Unauthorized() {
  return (
    <main>
      <h1>401 - 인증되지 않음</h1>
      <p>이 페이지에 접근하려면 로그인해 주세요.</p>
      <Login />
    </main>
  )
}
```

## 사용 예제

### 인증으로 라우트 보호하기

보호된 페이지 컴포넌트에서 사용자가 유효한 인증이 없을 때 `unauthorized()` 함수를 호출합니다:

```tsx
// app/dashboard/page.tsx
import { verifySession } from '@/app/lib/dal'
import { unauthorized } from 'next/navigation'

export default async function DashboardPage() {
  const session = await verifySession()

  if (!session) {
    unauthorized()
  }

  return <div>대시보드</div>
}
```

`unauthorized()`가 호출되면:
1. `unauthorized.js` UI가 렌더링됩니다
2. `401` 상태 코드가 반환됩니다
3. 사용자는 로그인 옵션이 포함된 사용자 정의 미인증 페이지를 보게 됩니다

## 레퍼런스

### Props

`unauthorized.js` 컴포넌트는 **어떤 props도 받지 않습니다**.

## 주요 사항

- `unauthorized.js`를 `app/` 디렉토리 루트에 배치합니다
- 사용자 정의 401 에러 UI 렌더링에 사용됩니다
- 보호된 라우트에서 `unauthorized()` 함수 호출로 트리거됩니다
- 실험적 기능을 사용하려면 `next.config.js`에서 `authInterrupts`를 활성화해야 합니다

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    authInterrupts: true,
  },
}

export default nextConfig
```

## `forbidden`과의 차이점

| 파일 | 함수 | 상태 코드 | 용도 |
|------|------|----------|------|
| `unauthorized.js` | `unauthorized()` | 401 | 인증되지 않음 (로그인 필요) |
| `forbidden.js` | `forbidden()` | 403 | 인증됨, 하지만 권한 없음 |

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| v15.1.0 | `unauthorized.js` 도입 |

---

## 참고

- [unauthorized() 함수](/app-router/api-reference/functions/unauthorized.md)
- [forbidden.js](/app-router/api-reference/file-conventions/forbidden.md)
- [인증 가이드](/app-router/guides/authentication.md)
