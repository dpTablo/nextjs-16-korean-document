# forbidden

## 개요

`forbidden` 함수는 Next.js **403 에러 페이지**를 렌더링하는 에러를 발생시킵니다. 사용자가 인증되었지만 필요한 권한이 없을 때 권한 부여 에러를 처리하는 데 사용됩니다.

> **⚠️ 실험적 기능 (Experimental)**
>
> 이 함수는 실험적이며 변경될 수 있습니다. 프로덕션 환경에서는 권장되지 않습니다.

---

## 설정 요구사항

`next.config.js`에서 `authInterrupts` 설정을 활성화해야 합니다:

```ts
// next.config.js
const nextConfig = {
  experimental: {
    authInterrupts: true,
  },
}

export default nextConfig
```

---

## 사용 가능한 위치

| 위치 | 사용 가능 |
|------|----------|
| **Server Components** | ✅ 예 |
| **Server Actions** | ✅ 예 |
| **Route Handlers** | ✅ 예 |
| **Root layout** | ❌ 아니오 |

---

## 기본 사용법

```tsx
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export default async function AdminPage() {
  const session = await verifySession()

  // 권한 확인
  if (session.role !== 'admin') {
    forbidden() // 403 페이지 표시
  }

  return <main>관리자 콘텐츠</main>
}
```

---

## API 레퍼런스

### 함수 시그니처

```tsx
forbidden(): never
```

### 매개변수

- 없음

### 반환 값

- `never` - 이 함수는 에러를 발생시키므로 반환하지 않습니다

---

## 실용적인 예제

### 1. 역할 기반 페이지 보호

```tsx
// app/admin/page.tsx
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export default async function AdminPage() {
  const session = await verifySession()

  if (session.role !== 'admin') {
    forbidden()
  }

  return (
    <main>
      <h1>관리자 대시보드</h1>
      {/* 관리자만 볼 수 있는 콘텐츠 */}
    </main>
  )
}
```

### 2. Server Actions에서 권한 확인

```ts
// app/actions.ts
'use server'

import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export async function updateRole(formData: FormData) {
  const session = await verifySession()

  // 관리자만 역할 변경 가능
  if (session.role !== 'admin') {
    forbidden()
  }

  const userId = formData.get('userId')
  const newRole = formData.get('role')

  // 역할 업데이트 수행
  await db.users.update({
    where: { id: userId },
    data: { role: newRole },
  })

  revalidatePath('/users')
}
```

### 3. 특정 리소스 접근 제어

```tsx
// app/posts/[id]/edit/page.tsx
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'
import { getPost } from '@/app/lib/posts'

export default async function EditPost({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const session = await verifySession()
  const post = await getPost(id)

  // 본인의 포스트만 수정 가능
  if (post.authorId !== session.userId) {
    forbidden()
  }

  return (
    <form>
      {/* 포스트 수정 폼 */}
    </form>
  )
}
```

### 4. Route Handlers에서 사용

```ts
// app/api/admin/users/route.ts
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export async function DELETE(request: Request) {
  const session = await verifySession()

  if (session.role !== 'admin') {
    forbidden()
  }

  const { userId } = await request.json()

  await db.users.delete({
    where: { id: userId },
  })

  return Response.json({ success: true })
}
```

### 5. 다중 권한 확인

```tsx
// app/settings/billing/page.tsx
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export default async function BillingPage() {
  const session = await verifySession()

  // 여러 조건 확인
  const hasPermission =
    session.role === 'admin' ||
    session.role === 'owner' ||
    session.permissions.includes('billing:read')

  if (!hasPermission) {
    forbidden()
  }

  return <main>청구 설정</main>
}
```

### 6. 조직/팀 멤버십 확인

```tsx
// app/teams/[teamId]/settings/page.tsx
import { verifySession } from '@/app/lib/dal'
import { forbidden } from 'next/navigation'

export default async function TeamSettings({
  params,
}: {
  params: Promise<{ teamId: string }>
}) {
  const { teamId } = await params
  const session = await verifySession()

  // 팀 멤버십 확인
  const membership = await db.teamMembers.findFirst({
    where: {
      teamId,
      userId: session.userId,
    },
  })

  if (!membership || membership.role === 'viewer') {
    forbidden()
  }

  return <main>팀 설정</main>
}
```

---

## 커스터마이징

### 커스텀 403 페이지

`forbidden.js` 특수 파일을 사용하여 403 에러 페이지 UI를 커스터마이징할 수 있습니다:

```tsx
// app/forbidden.tsx
export default function ForbiddenPage() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-4xl font-bold">403</h1>
      <h2 className="text-2xl">접근 권한이 없습니다</h2>
      <p className="text-gray-600 mt-4">
        이 페이지에 접근할 권한이 없습니다.
      </p>
      <a href="/" className="mt-8 text-blue-600 hover:underline">
        홈으로 돌아가기
      </a>
    </div>
  )
}
```

---

## unauthorized()와 비교

| 함수 | HTTP 상태 | 의미 | 사용 시나리오 |
|------|----------|------|--------------|
| **`unauthorized()`** | 401 | 인증되지 않음 | 로그인 필요 |
| **`forbidden()`** | 403 | 권한 없음 | 로그인했지만 권한 부족 |

### 사용 예제 비교

```tsx
import { verifySession } from '@/app/lib/dal'
import { unauthorized, forbidden } from 'next/navigation'

export default async function ProtectedPage() {
  const session = await verifySession()

  // 로그인하지 않은 경우
  if (!session) {
    unauthorized() // 401 - 로그인 페이지로
  }

  // 로그인했지만 권한 없는 경우
  if (session.role !== 'admin') {
    forbidden() // 403 - 권한 없음 페이지
  }

  return <main>보호된 콘텐츠</main>
}
```

---

## 주요 사용 사례

### 1. 역할 기반 접근 제어 (RBAC)

사용자 역할에 따라 라우트를 제한하면서 인증된 사용자가 리디렉션 대신 적절한 403 응답을 받도록 합니다.

```tsx
if (session.role !== 'admin') {
  forbidden() // 403 페이지 표시
}
```

### 2. Server Actions 권한 부여

민감한 작업을 권한이 있는 사용자만 수행할 수 있도록 보호합니다:

```ts
'use server'

export async function deleteUser(userId: string) {
  const session = await verifySession()

  if (session.role !== 'admin') {
    forbidden()
  }

  await db.users.delete({ where: { id: userId } })
}
```

### 3. 리소스 소유권 확인

사용자가 자신의 리소스만 수정할 수 있도록 합니다:

```tsx
const post = await getPost(id)

if (post.authorId !== session.userId) {
  forbidden()
}
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **인증 확인 먼저, 권한 확인 나중에**
   ```tsx
   if (!session) {
     unauthorized() // 401
   }

   if (session.role !== 'admin') {
     forbidden() // 403
   }
   ```

2. **명확한 권한 로직**
   ```tsx
   const canEdit =
     session.role === 'admin' ||
     post.authorId === session.userId

   if (!canEdit) {
     forbidden()
   }
   ```

3. **Server Actions에서 항상 권한 확인**
   ```ts
   'use server'

   export async function sensitiveAction() {
     // 클라이언트 사이드 체크를 신뢰하지 말 것
     const session = await verifySession()
     if (!hasPermission(session)) {
       forbidden()
     }
   }
   ```

### ❌ 피해야 할 사항

1. **Root Layout에서 사용**
   ```tsx
   // ❌ Root Layout에서 사용 불가
   // app/layout.tsx
   export default function RootLayout() {
     forbidden() // 작동하지 않음
   }
   ```

2. **클라이언트 컴포넌트에서 사용**
   ```tsx
   // ❌ Server Component에서만 사용 가능
   'use client'

   export default function Page() {
     forbidden() // 작동하지 않음
   }
   ```

3. **인증과 권한 혼동**
   ```tsx
   // ❌ 잘못된 사용
   if (!session) {
     forbidden() // unauthorized()를 사용해야 함
   }
   ```

---

## 에러 처리

`forbidden()`은 에러를 발생시키므로 try/catch로 감쌀 필요가 없습니다:

```tsx
// ✅ 올바른 사용
if (!hasPermission) {
  forbidden() // 여기서 실행 종료
}

// 이 코드는 실행되지 않음
```

```tsx
// ❌ 불필요한 try/catch
try {
  if (!hasPermission) {
    forbidden()
  }
} catch (error) {
  // 불필요
}
```

---

## 실험적 기능 주의사항

> **⚠️ 프로덕션 사용 주의**
>
> `forbidden()`은 실험적 기능이므로:
> - API가 변경될 수 있습니다
> - 버그가 있을 수 있습니다
> - 프로덕션 사용 전 충분한 테스트 필요

**프로덕션 대안:**

```tsx
import { redirect } from 'next/navigation'

export default async function AdminPage() {
  const session = await verifySession()

  if (session.role !== 'admin') {
    redirect('/403') // 커스텀 403 페이지로 리디렉트
  }
}
```

---

## 타입 정의

```typescript
function forbidden(): never
```

---

## 관련 API

- [`unauthorized()`](./unauthorized.md) - 401 Unauthorized 응답
- [`notFound()`](./notFound.md) - 404 Not Found 응답
- [`redirect()`](./redirect.md) - 리디렉션
- [`forbidden.js` 파일 규칙](../file-conventions/forbidden.md) - 커스텀 403 페이지

---

## 버전 정보

- **도입 버전:** Next.js 15.1.0
- **현재 상태:** Experimental
- **안정화 예정:** 미정

---

## 요약

- **용도:** 403 Forbidden 에러 페이지 렌더링
- **상태 코드:** 403
- **의미:** 인증됨, 하지만 권한 없음
- **주 사용처:** 역할 기반 접근 제어, Server Actions 권한 확인
- **커스터마이징:** `forbidden.js` 파일로 UI 커스터마이징
- **제약:** Root Layout에서 사용 불가, Server Component에서만 사용
- **주의:** 실험적 기능, `authInterrupts` 설정 필요
