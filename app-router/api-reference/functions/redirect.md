# redirect

`redirect` 함수는 사용자를 다른 URL로 리디렉션합니다. Server Components, Client Components, Route Handlers, Server Actions에서 사용할 수 있습니다.

---

## 함수 시그니처

```typescript
redirect(path: string, type?: 'replace' | 'push'): never
```

### 매개변수

| 매개변수 | 타입 | 설명 |
|---------|------|------|
| `path` | `string` | 리디렉션할 URL (상대/절대 경로) |
| `type` | `'replace'` \| `'push'` | 리디렉션 방식 (기본값: Server Actions은 'push', 나머지는 'replace') |

---

## 동작 방식

`redirect` 함수는 컨텍스트에 따라 다르게 동작합니다:

- **스트리밍 컨텍스트**: 클라이언트 측에서 리디렉션을 수행하는 meta 태그 삽입
- **Server Actions**: 303 HTTP 리디렉션 응답 반환
- **기타 (Server Components, Route Handlers)**: 307 HTTP 리디렉션 응답 반환

---

## 주요 특징

- ✅ **반환값 없음**: `never` 타입 사용
- ✅ **절대 URL 지원**: 외부 링크로 리디렉션 가능
- ✅ **다양한 컨텍스트 지원**: Server Components, Client Components (Server Action을 통해), Route Handlers, Server Actions
- ✅ **에러 기반**: 내부적으로 에러를 throw하므로 `try/catch` 블록 밖에서 호출 필요

---

## 예제

### Server Component에서 사용

```tsx
// app/team/[id]/page.tsx
import { redirect } from 'next/navigation'

async function fetchTeam(id: string) {
  const res = await fetch('https://...')
  if (!res.ok) return undefined
  return res.json()
}

export default async function Profile({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const team = await fetchTeam(id)

  if (!team) {
    redirect('/login')
  }

  return <div>Team: {team.name}</div>
}
```

### Client Component에서 사용 (Server Action 활용)

```tsx
// app/client-redirect.tsx
'use client'
import { navigate } from './actions'

export function ClientRedirect() {
  return (
    <form action={navigate}>
      <input type="text" name="id" />
      <button>Submit</button>
    </form>
  )
}
```

```ts
// app/actions.ts
'use server'
import { redirect } from 'next/navigation'

export async function navigate(data: FormData) {
  redirect(`/posts/${data.get('id')}`)
}
```

### Route Handler에서 사용

```ts
// app/api/login/route.ts
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  // 인증 로직
  const isAuthenticated = false

  if (!isAuthenticated) {
    redirect('/login')
  }

  return new Response('Authenticated', { status: 200 })
}
```

### Server Action에서 사용

```tsx
// app/login/page.tsx
'use client'
import { login } from './actions'

export default function LoginPage() {
  return (
    <form action={login}>
      <input type="email" name="email" />
      <input type="password" name="password" />
      <button type="submit">Login</button>
    </form>
  )
}
```

```ts
// app/login/actions.ts
'use server'
import { redirect } from 'next/navigation'

export async function login(formData: FormData) {
  const email = formData.get('email')
  const password = formData.get('password')

  // 인증 로직
  const isValid = await validateCredentials(email, password)

  if (isValid) {
    redirect('/dashboard')
  }

  // 인증 실패 시 에러 처리
  throw new Error('Invalid credentials')
}
```

---

## try/catch 블록과 함께 사용

`redirect`는 내부적으로 에러를 throw하므로 `try/catch` 블록 **밖에서** 호출해야 합니다.

```tsx
// ❌ 잘못된 사용
export async function ServerAction() {
  try {
    const user = await getUser()
    if (!user) {
      redirect('/login') // 작동하지 않음!
    }
  } catch (error) {
    console.error(error)
  }
}

// ✅ 올바른 사용
export async function ServerAction() {
  try {
    const user = await getUser()
    if (!user) {
      // try 블록 밖에서 호출
      throw new Error('User not found')
    }
  } catch (error) {
    console.error(error)
  }

  // try/catch 밖에서 리디렉션
  if (!user) {
    redirect('/login')
  }
}
```

---

## 절대 URL 사용

외부 URL로 리디렉션할 수 있습니다.

```tsx
import { redirect } from 'next/navigation'

export async function ServerComponent() {
  // 외부 URL로 리디렉션
  redirect('https://example.com')
}
```

---

## type 매개변수

`type` 매개변수로 리디렉션 방식을 제어할 수 있습니다.

```tsx
import { redirect } from 'next/navigation'

// 기본값: Server Actions에서는 'push', 나머지는 'replace'
redirect('/dashboard') // replace

// 명시적으로 push 사용
redirect('/dashboard', 'push') // push (히스토리에 추가)

// 명시적으로 replace 사용
redirect('/dashboard', 'replace') // replace (히스토리 대체)
```

---

## HTTP 상태 코드

- **307**: 임시 리디렉션 (기본값, POST 메서드 보존)
- **303**: Server Actions에서 사용 (POST → GET 변환)
- **308**: 영구 리디렉션 (`permanentRedirect` 함수 사용)

---

## 사용 시 주의사항

> **Good to know**:
> * Client Components에서는 렌더링 프로세스 중에만 사용 가능 (이벤트 핸들러에서는 불가)
> * 이벤트 핸들러에서 리디렉션이 필요한 경우 `useRouter` 훅 사용
> * `redirect`는 에러를 throw하므로 `try/catch` 블록 밖에서 호출
> * TypeScript에서 `never` 타입을 반환하므로 `return` 문 불필요
> * Server Actions/Route Handlers에서 호출 시 응답 완료 후 리디렉션 수행

---

## 관련 함수

- **`permanentRedirect`**: 308 영구 리디렉션
- **`useRouter`**: 클라이언트 측 이벤트 핸들러에서의 네비게이션
- **`notFound`**: 404 페이지로 리디렉션

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v13.0.0 | `redirect` 도입 |

---

## 관련 문서

- [permanentRedirect](./permanentRedirect.md)
- [notFound](./notFound.md)
- [Linking and Navigating](../../getting-started/04-linking-and-navigating.md)
