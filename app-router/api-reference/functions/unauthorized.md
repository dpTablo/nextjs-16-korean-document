# unauthorized

## 개요

`unauthorized` 함수는 Next.js **401 에러 페이지**를 렌더링하는 에러를 발생시킵니다. 사용자가 인증되지 않았을 때 인증 에러를 처리하는 데 사용됩니다.

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
import { unauthorized } from 'next/navigation'

export default async function DashboardPage() {
  const session = await verifySession()

  // 인증 확인
  if (!session) {
    unauthorized() // 401 페이지 표시
  }

  return <main>대시보드 콘텐츠</main>
}
```

---

## API 레퍼런스

### 함수 시그니처

```tsx
unauthorized(): never
```

### 매개변수

- 없음

### 반환 값

- `never` - 이 함수는 에러를 발생시키므로 반환하지 않습니다

---

## 실용적인 예제

### 1. 로그인 UI 표시

**페이지 컴포넌트:**

```tsx
// app/dashboard/page.tsx
import { verifySession } from '@/app/lib/dal'
import { unauthorized } from 'next/navigation'

export default async function DashboardPage() {
  const session = await verifySession()

  if (!session) {
    unauthorized()
  }

  return (
    <main>
      <h1>대시보드에 오신 것을 환영합니다</h1>
      <p>안녕하세요, {session.user.name}님.</p>
    </main>
  )
}
```

**커스텀 401 UI:**

```tsx
// app/unauthorized.tsx
import Login from '@/app/components/Login'

export default function UnauthorizedPage() {
  return (
    <main>
      <h1>401 - 인증 필요</h1>
      <p>이 페이지에 접근하려면 로그인이 필요합니다.</p>
      <Login />
    </main>
  )
}
```

### 2. Server Actions에서 인증 확인

```ts
// app/actions/update-profile.ts
'use server'

import { verifySession } from '@/app/lib/dal'
import { unauthorized } from 'next/navigation'
import { revalidatePath } from 'next/cache'

export async function updateProfile(formData: FormData) {
  const session = await verifySession()

  // 로그인하지 않은 경우
  if (!session) {
    unauthorized()
  }

  const name = formData.get('name')
  const email = formData.get('email')

  // 프로필 업데이트 수행
  await db.users.update({
    where: { id: session.userId },
    data: { name, email },
  })

  revalidatePath('/profile')
}
```

### 3. Route Handlers에서 사용

```tsx
// app/api/profile/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { verifySession } from '@/app/lib/dal'
import { unauthorized } from 'next/navigation'

export async function GET(req: NextRequest): Promise<NextResponse> {
  const session = await verifySession()

  if (!session) {
    unauthorized()
  }

  // 사용자 프로필 데이터 가져오기
  const profile = await db.users.findUnique({
    where: { id: session.userId },
  })

  return NextResponse.json(profile)
}
```

### 4. 여러 페이지에서 공통 인증 로직

```tsx
// app/lib/auth.ts
import { verifySession } from '@/app/lib/dal'
import { unauthorized } from 'next/navigation'

export async function requireAuth() {
  const session = await verifySession()

  if (!session) {
    unauthorized()
  }

  return session
}
```

```tsx
// app/dashboard/page.tsx
import { requireAuth } from '@/app/lib/auth'

export default async function DashboardPage() {
  const session = await requireAuth()

  return <main>대시보드</main>
}
```

```tsx
// app/profile/page.tsx
import { requireAuth } from '@/app/lib/auth'

export default async function ProfilePage() {
  const session = await requireAuth()

  return <main>프로필</main>
}
```

### 5. 토큰 기반 인증

```tsx
// app/api/protected/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { unauthorized } from 'next/navigation'
import { verifyToken } from '@/app/lib/jwt'

export async function POST(req: NextRequest): Promise<NextResponse> {
  const token = req.headers.get('Authorization')?.replace('Bearer ', '')

  if (!token) {
    unauthorized()
  }

  const payload = await verifyToken(token)

  if (!payload) {
    unauthorized()
  }

  // API 로직 처리
  return NextResponse.json({ success: true })
}
```

### 6. 세션 만료 처리

```tsx
// app/settings/page.tsx
import { verifySession } from '@/app/lib/dal'
import { unauthorized } from 'next/navigation'

export default async function SettingsPage() {
  const session = await verifySession()

  // 세션이 없거나 만료된 경우
  if (!session || session.expiresAt < Date.now()) {
    unauthorized()
  }

  return <main>설정</main>
}
```

---

## 커스터마이징

### 커스텀 401 페이지

`unauthorized.js` 특수 파일을 사용하여 401 에러 페이지 UI를 커스터마이징할 수 있습니다:

```tsx
// app/unauthorized.tsx
import Login from '@/app/components/Login'

export default function UnauthorizedPage() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-4xl font-bold">401</h1>
      <h2 className="text-2xl">인증이 필요합니다</h2>
      <p className="text-gray-600 mt-4">
        이 페이지에 접근하려면 로그인이 필요합니다.
      </p>
      <div className="mt-8">
        <Login />
      </div>
      <a href="/" className="mt-4 text-blue-600 hover:underline">
        홈으로 돌아가기
      </a>
    </div>
  )
}
```

### 로그인 컴포넌트 예제

```tsx
// app/components/Login.tsx
'use client'

import { useActionState } from 'react'
import { login } from '@/app/actions/auth'

export default function Login() {
  const [state, formAction, isPending] = useActionState(login, null)

  return (
    <form action={formAction} className="w-full max-w-md">
      <div className="mb-4">
        <label htmlFor="email" className="block text-sm font-medium mb-2">
          이메일
        </label>
        <input
          type="email"
          id="email"
          name="email"
          required
          className="w-full px-3 py-2 border rounded-lg"
        />
      </div>
      <div className="mb-4">
        <label htmlFor="password" className="block text-sm font-medium mb-2">
          비밀번호
        </label>
        <input
          type="password"
          id="password"
          name="password"
          required
          className="w-full px-3 py-2 border rounded-lg"
        />
      </div>
      {state?.error && (
        <p className="text-red-600 mb-4">{state.error}</p>
      )}
      <button
        type="submit"
        disabled={isPending}
        className="w-full bg-blue-600 text-white py-2 rounded-lg disabled:bg-gray-400"
      >
        {isPending ? '로그인 중...' : '로그인'}
      </button>
    </form>
  )
}
```

---

## forbidden()와 비교

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

  // 1단계: 인증 확인
  if (!session) {
    unauthorized() // 401 - 로그인 필요
  }

  // 2단계: 권한 확인
  if (session.role !== 'admin') {
    forbidden() // 403 - 권한 없음
  }

  return <main>관리자 전용 콘텐츠</main>
}
```

**흐름 차트:**

```
사용자 접근 시도
    ↓
로그인 여부 확인
    ↓
   NO → unauthorized() (401)
    ↓
   YES
    ↓
권한 확인
    ↓
   NO → forbidden() (403)
    ↓
   YES
    ↓
콘텐츠 표시
```

---

## 주요 사용 사례

### 1. 보호된 페이지 접근 제어

로그인하지 않은 사용자가 보호된 페이지에 접근할 때 로그인 페이지를 표시합니다.

```tsx
if (!session) {
  unauthorized() // 401 페이지 표시
}
```

### 2. Server Actions 인증

민감한 작업을 인증된 사용자만 수행할 수 있도록 보호합니다:

```ts
'use server'

export async function deleteAccount() {
  const session = await verifySession()

  if (!session) {
    unauthorized()
  }

  await db.users.delete({ where: { id: session.userId } })
}
```

### 3. API 라우트 보호

API 엔드포인트에 대한 인증을 강제합니다:

```tsx
export async function GET(request: Request) {
  const session = await verifySession()

  if (!session) {
    unauthorized()
  }

  return Response.json({ data: 'protected data' })
}
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **먼저 인증 확인, 나중에 권한 확인**
   ```tsx
   // ✅ 올바른 순서
   if (!session) {
     unauthorized() // 먼저 로그인 확인
   }

   if (session.role !== 'admin') {
     forbidden() // 그 다음 권한 확인
   }
   ```

2. **커스텀 401 페이지에서 로그인 UI 제공**
   ```tsx
   // app/unauthorized.tsx
   export default function UnauthorizedPage() {
     return (
       <div>
         <h1>로그인이 필요합니다</h1>
         <Login /> {/* 로그인 폼 제공 */}
       </div>
     )
   }
   ```

3. **재사용 가능한 인증 헬퍼 함수 작성**
   ```tsx
   // app/lib/auth.ts
   export async function requireAuth() {
     const session = await verifySession()
     if (!session) unauthorized()
     return session
   }
   ```

4. **Server Actions에서 항상 인증 확인**
   ```ts
   'use server'

   export async function updateData() {
     // 클라이언트 사이드 체크를 신뢰하지 말 것
     const session = await verifySession()
     if (!session) unauthorized()
   }
   ```

5. **세션 만료 처리**
   ```tsx
   if (!session || session.expiresAt < Date.now()) {
     unauthorized()
   }
   ```

### ❌ 피해야 할 사항

1. **Root Layout에서 사용**
   ```tsx
   // ❌ Root Layout에서 사용 불가
   // app/layout.tsx
   export default function RootLayout() {
     unauthorized() // 작동하지 않음
   }
   ```

2. **클라이언트 컴포넌트에서 사용**
   ```tsx
   // ❌ Server Component에서만 사용 가능
   'use client'

   export default function Page() {
     unauthorized() // 작동하지 않음
   }
   ```

3. **권한 문제에 unauthorized 사용**
   ```tsx
   // ❌ 잘못된 사용
   if (session.role !== 'admin') {
     unauthorized() // forbidden()를 사용해야 함
   }
   ```

4. **redirect()와 혼동**
   ```tsx
   // ❌ 잘못된 사용
   if (!session) {
     redirect('/login') // unauthorized()를 사용해야 함
   }
   ```

---

## 에러 처리

`unauthorized()`는 에러를 발생시키므로 try/catch로 감쌀 필요가 없습니다:

```tsx
// ✅ 올바른 사용
if (!session) {
  unauthorized() // 여기서 실행 종료
}

// 이 코드는 실행되지 않음
```

```tsx
// ❌ 불필요한 try/catch
try {
  if (!session) {
    unauthorized()
  }
} catch (error) {
  // 불필요
}
```

---

## 실험적 기능 주의사항

> **⚠️ 프로덕션 사용 주의**
>
> `unauthorized()`는 실험적 기능이므로:
> - API가 변경될 수 있습니다
> - 버그가 있을 수 있습니다
> - 프로덕션 사용 전 충분한 테스트 필요

**프로덕션 대안:**

```tsx
import { redirect } from 'next/navigation'

export default async function ProtectedPage() {
  const session = await verifySession()

  if (!session) {
    redirect('/login') // 로그인 페이지로 리디렉트
  }
}
```

---

## 세션 검증 예제

### 세션 검증 함수 구현

```ts
// app/lib/dal.ts (Data Access Layer)
import 'server-only'
import { cookies } from 'next/headers'
import { decrypt } from '@/app/lib/session'

export async function verifySession() {
  const cookieStore = await cookies()
  const cookie = cookieStore.get('session')?.value

  if (!cookie) {
    return null
  }

  const session = await decrypt(cookie)

  if (!session?.userId) {
    return null
  }

  return {
    userId: session.userId as string,
    user: session.user,
    role: session.role as string,
    expiresAt: session.expiresAt as number,
  }
}
```

### 세션 암호화/복호화

```ts
// app/lib/session.ts
import 'server-only'
import { SignJWT, jwtVerify } from 'jose'

const secretKey = process.env.SESSION_SECRET
const encodedKey = new TextEncoder().encode(secretKey)

export async function encrypt(payload: any) {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(encodedKey)
}

export async function decrypt(session: string | undefined = '') {
  try {
    const { payload } = await jwtVerify(session, encodedKey, {
      algorithms: ['HS256'],
    })
    return payload
  } catch (error) {
    console.error('Failed to verify session')
    return null
  }
}
```

---

## 타입 정의

```typescript
function unauthorized(): never
```

---

## 관련 API

- [`forbidden()`](./forbidden.md) - 403 Forbidden 응답
- [`notFound()`](./notFound.md) - 404 Not Found 응답
- [`redirect()`](./redirect.md) - 리디렉션
- [`unauthorized.js` 파일 규칙](../file-conventions/unauthorized.md) - 커스텀 401 페이지

---

## 버전 정보

- **도입 버전:** Next.js 15.1.0
- **현재 상태:** Experimental
- **안정화 예정:** 미정

---

## 요약

- **용도:** 401 Unauthorized 에러 페이지 렌더링
- **상태 코드:** 401
- **의미:** 인증되지 않음 (로그인 필요)
- **주 사용처:** 로그인 확인, 보호된 페이지/API 접근 제어
- **커스터마이징:** `unauthorized.js` 파일로 UI 커스터마이징
- **제약:** Root Layout에서 사용 불가, Server Component에서만 사용
- **주의:** 실험적 기능, `authInterrupts` 설정 필요
- **vs forbidden():** unauthorized는 401(로그인 필요), forbidden은 403(권한 부족)
