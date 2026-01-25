# forbidden.js

**forbidden.js**는 인증 과정에서 `forbidden()` 함수가 호출될 때 사용자 정의 UI를 렌더링하는 데 사용되는 실험적 Next.js 파일 규칙입니다 (v15.1.0+). `403` 상태 코드를 반환합니다.

> **경고**: 이 기능은 실험적이며 변경될 수 있습니다. 프로덕션 환경에서의 사용은 권장하지 않습니다.

## 코드 예제

### TypeScript

```tsx
// app/forbidden.tsx
import Link from 'next/link'

export default function Forbidden() {
  return (
    <div>
      <h2>접근 금지</h2>
      <p>이 리소스에 접근할 권한이 없습니다.</p>
      <Link href="/">홈으로 돌아가기</Link>
    </div>
  )
}
```

### JavaScript

```jsx
// app/forbidden.jsx
import Link from 'next/link'

export default function Forbidden() {
  return (
    <div>
      <h2>접근 금지</h2>
      <p>이 리소스에 접근할 권한이 없습니다.</p>
      <Link href="/">홈으로 돌아가기</Link>
    </div>
  )
}
```

## 사용 예제

### 인증으로 라우트 보호하기

보호된 페이지 컴포넌트에서 사용자가 유효한 권한이 없을 때 `forbidden()` 함수를 호출합니다:

```tsx
// app/admin/page.tsx
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export default async function AdminPage() {
  const session = await verifySession()

  if (!session?.isAdmin) {
    forbidden()
  }

  return <div>관리자 대시보드</div>
}
```

`forbidden()`이 호출되면:
1. `forbidden.js` UI가 렌더링됩니다
2. `403` 상태 코드가 반환됩니다
3. 사용자는 사용자 정의 접근 금지 페이지를 보게 됩니다

## 레퍼런스

### Props

`forbidden.js` 컴포넌트는 **어떤 props도 받지 않습니다**.

## 주요 사항

- `forbidden.js`를 `app/` 디렉토리 루트에 배치합니다
- 사용자 정의 403 에러 UI 렌더링에 사용됩니다
- 보호된 라우트에서 `forbidden()` 함수 호출로 트리거됩니다
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

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| v15.1.0 | `forbidden.js` 도입 |

---

## 참고

- [forbidden() 함수](/app-router/api-reference/functions/forbidden.md)
- [unauthorized.js](/app-router/api-reference/file-conventions/unauthorized.md)
- [인증 가이드](/app-router/guides/authentication.md)
