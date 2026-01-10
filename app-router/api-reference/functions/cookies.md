# cookies

`cookies`는 **async 함수**로, Server Components에서 HTTP incoming request cookies를 읽고, Server Actions 또는 Route Handlers에서 outgoing request cookies를 읽고 쓸 수 있습니다.

---

## 기본 사용법

```tsx
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = await cookies()
  const theme = cookieStore.get('theme')
  return <div>Current theme: {theme?.value}</div>
}
```

---

## 주요 메서드

### get(name)

쿠키 이름으로 단일 쿠키를 반환합니다.

```tsx
const cookieStore = await cookies()
const theme = cookieStore.get('theme')
// { name: 'theme', value: 'dark' }
```

**반환값:**

```typescript
{
  name: string
  value: string
}
```

### getAll()

모든 쿠키를 배열로 반환합니다.

```tsx
const cookieStore = await cookies()
const allCookies = cookieStore.getAll()
// [{ name: 'theme', value: 'dark' }, { name: 'user', value: 'john' }]
```

**특정 이름의 모든 쿠키:**

```tsx
const cookieStore = await cookies()
const themeCookies = cookieStore.getAll('theme')
```

**반환값:**

```typescript
Array<{
  name: string
  value: string
}>
```

### has(name)

쿠키 존재 여부를 확인합니다.

```tsx
const cookieStore = await cookies()
const hasTheme = cookieStore.has('theme')
// true or false
```

**반환값:** `boolean`

### set(name, value, options)

쿠키를 설정합니다. **Server Actions 또는 Route Handlers에서만 사용 가능**합니다.

```tsx
'use server'
import { cookies } from 'next/headers'

export async function createCookie(data: FormData) {
  const cookieStore = await cookies()

  // 기본 설정
  cookieStore.set('name', 'lee')

  // 옵션과 함께 설정
  cookieStore.set('name', 'lee', {
    secure: true,
    httpOnly: true,
    maxAge: 60 * 60 * 24 * 7, // 7일
    path: '/',
  })

  // 객체 형식
  cookieStore.set({
    name: 'name',
    value: 'lee',
    httpOnly: true,
    secure: true,
    maxAge: 60 * 60 * 24 * 7,
    path: '/',
  })
}
```

### delete(name)

쿠키를 삭제합니다. **Server Actions 또는 Route Handlers에서만 사용 가능**합니다.

```tsx
'use server'
import { cookies } from 'next/headers'

export async function deleteCookie(data: FormData) {
  const cookieStore = await cookies()

  // 단일 삭제
  cookieStore.delete('name')

  // 여러 개 삭제
  cookieStore.delete(['name1', 'name2'])
}
```

### toString()

쿠키의 문자열 표현을 반환합니다.

```tsx
const cookieStore = await cookies()
const cookieString = cookieStore.toString()
// "theme=dark; user=john"
```

---

## 쿠키 옵션

### name (필수)

쿠키의 이름입니다.

```tsx
cookieStore.set('theme', 'dark')
```

### value (필수)

쿠키의 값입니다.

```tsx
cookieStore.set('theme', 'dark')
```

### expires

쿠키의 만료 날짜입니다.

```tsx
cookieStore.set('theme', 'dark', {
  expires: new Date('2025-12-31')
})
```

### maxAge

쿠키의 유효 시간(초 단위)입니다.

```tsx
cookieStore.set('theme', 'dark', {
  maxAge: 60 * 60 * 24 * 7 // 7일
})
```

### domain

쿠키를 사용할 도메인입니다.

```tsx
cookieStore.set('theme', 'dark', {
  domain: '.example.com'
})
```

### path

쿠키의 경로입니다. 기본값은 `'/'`입니다.

```tsx
cookieStore.set('theme', 'dark', {
  path: '/dashboard'
})
```

### secure

HTTPS 연결에서만 쿠키를 전송합니다.

```tsx
cookieStore.set('theme', 'dark', {
  secure: true
})
```

### httpOnly

HTTP 요청에서만 쿠키에 접근할 수 있도록 합니다. JavaScript에서 접근 불가능합니다.

```tsx
cookieStore.set('theme', 'dark', {
  httpOnly: true
})
```

### sameSite

크로스사이트 요청 시 쿠키 동작을 제어합니다.

- `'strict'`: 동일한 사이트에서만 쿠키 전송
- `'lax'`: 일부 크로스사이트 요청에서 쿠키 전송 (기본값)
- `'none'`: 모든 크로스사이트 요청에서 쿠키 전송 (secure 필요)

```tsx
cookieStore.set('theme', 'dark', {
  sameSite: 'strict'
})
```

### priority

쿠키의 우선순위를 설정합니다.

- `'low'`
- `'medium'`
- `'high'`

```tsx
cookieStore.set('theme', 'dark', {
  priority: 'high'
})
```

### partitioned

파티션된 쿠키 여부를 설정합니다.

```tsx
cookieStore.set('theme', 'dark', {
  partitioned: true
})
```

---

## 사용 예제

### Server Component에서 쿠키 읽기

```tsx
// app/page.tsx
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = await cookies()
  const theme = cookieStore.get('theme')

  return (
    <div>
      <h1>Current Theme</h1>
      <p>Theme: {theme?.value || 'default'}</p>
    </div>
  )
}
```

### Server Action에서 쿠키 설정

```tsx
// app/actions.ts
'use server'
import { cookies } from 'next/headers'

export async function setTheme(theme: string) {
  const cookieStore = await cookies()

  cookieStore.set('theme', theme, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24 * 30, // 30일
    path: '/',
    sameSite: 'lax',
  })
}
```

```tsx
// app/theme-switcher.tsx
'use client'
import { setTheme } from './actions'

export function ThemeSwitcher() {
  return (
    <form action={async (formData) => {
      await setTheme(formData.get('theme') as string)
    }}>
      <select name="theme">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      <button type="submit">Set Theme</button>
    </form>
  )
}
```

### Route Handler에서 쿠키 설정

```ts
// app/api/theme/route.ts
import { cookies } from 'next/headers'
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const { theme } = await request.json()
  const cookieStore = await cookies()

  cookieStore.set('theme', theme, {
    httpOnly: true,
    secure: true,
    maxAge: 60 * 60 * 24 * 30,
  })

  return NextResponse.json({ success: true })
}
```

### 쿠키 삭제

```tsx
// app/actions.ts
'use server'
import { cookies } from 'next/headers'

export async function logout() {
  const cookieStore = await cookies()

  // 단일 쿠키 삭제
  cookieStore.delete('session')

  // 여러 쿠키 삭제
  cookieStore.delete(['session', 'user', 'preferences'])
}
```

### 모든 쿠키 가져오기

```tsx
// app/page.tsx
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = await cookies()
  const allCookies = cookieStore.getAll()

  return (
    <div>
      <h1>All Cookies</h1>
      <ul>
        {allCookies.map((cookie) => (
          <li key={cookie.name}>
            {cookie.name}: {cookie.value}
          </li>
        ))}
      </ul>
    </div>
  )
}
```

---

## 중요 주의사항

> **Good to know**:
> * ✅ **async/await 필수**: `cookies()`는 Promise를 반환하므로 반드시 `async/await` 사용
> * ✅ **읽기**: Server Components에서 가능
> * ✅ **쓰기/삭제**: Server Actions 또는 Route Handlers에서만 가능
> * ✅ **Dynamic API**: `cookies()`는 동적 렌더링을 트리거합니다
> * ✅ **HTTP 제약**: 스트리밍 시작 후 쿠키 설정 불가능
> * ⚠️ Client Components에서는 직접 사용 불가능 (Server Actions를 통해 간접적으로 사용)

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v15.0.0-RC | `cookies`가 async 함수로 변경 |
| v13.0.0 | `cookies` 도입 |

---

## 관련 문서

- [headers](./headers.md)
- [Server Components](../../getting-started/05-server-and-client-components.md)
- [Server Actions](../../guides/forms.md)
