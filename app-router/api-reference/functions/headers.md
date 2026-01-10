# headers

`headers`는 **비동기(async)** 함수로, Server Component에서 HTTP 요청 헤더를 읽을 수 있게 해줍니다.

---

## 함수 시그니처

```typescript
headers(): Promise<ReadonlyHeaders>
```

### 매개변수

없음

### 반환값

**읽기 전용** Web Headers 객체를 반환합니다.

---

## 기본 사용법

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const userAgent = headersList.get('user-agent')

  return <div>User Agent: {userAgent}</div>
}
```

---

## 사용 가능한 메서드

`headers()` 함수는 Web [Headers API](https://developer.mozilla.org/en-US/docs/Web/API/Headers)의 읽기 전용 버전을 반환합니다.

### get(name)

특정 헤더 값을 조회합니다.

```tsx
const headersList = await headers()
const authorization = headersList.get('authorization')
// "Bearer token123" 또는 null
```

### has(name)

헤더 존재 여부를 확인합니다.

```tsx
const headersList = await headers()
const hasAuth = headersList.has('authorization')
// true 또는 false
```

### entries()

모든 헤더의 키/값 쌍을 반복합니다.

```tsx
const headersList = await headers()

for (const [key, value] of headersList.entries()) {
  console.log(`${key}: ${value}`)
}
```

### forEach(callback)

각 헤더의 키/값 쌍에 함수를 실행합니다.

```tsx
const headersList = await headers()

headersList.forEach((value, key) => {
  console.log(`${key}: ${value}`)
})
```

### keys()

모든 헤더 키를 반복합니다.

```tsx
const headersList = await headers()

for (const key of headersList.keys()) {
  console.log(key)
}
```

### values()

모든 헤더 값을 반복합니다.

```tsx
const headersList = await headers()

for (const value of headersList.values()) {
  console.log(value)
}
```

---

## 사용 예제

### 인증 헤더 전달

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const authorization = headersList.get('authorization')

  const res = await fetch('https://api.example.com/user', {
    headers: { authorization }, // 인증 헤더 전달
  })
  const user = await res.json()

  return <h1>{user.name}</h1>
}
```

### User Agent 확인

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const userAgent = headersList.get('user-agent')

  const isMobile = /mobile/i.test(userAgent || '')

  return (
    <div>
      <h1>Device Type</h1>
      <p>{isMobile ? 'Mobile' : 'Desktop'}</p>
    </div>
  )
}
```

### 모든 헤더 출력

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const allHeaders: Record<string, string> = {}

  headersList.forEach((value, key) => {
    allHeaders[key] = value
  })

  return (
    <div>
      <h1>Request Headers</h1>
      <pre>{JSON.stringify(allHeaders, null, 2)}</pre>
    </div>
  )
}
```

### Referer 확인

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const referer = headersList.get('referer')

  return (
    <div>
      <h1>Referer</h1>
      <p>{referer || 'Direct access'}</p>
    </div>
  )
}
```

### Custom 헤더 확인

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const apiKey = headersList.get('x-api-key')

  if (!apiKey) {
    return <div>API Key required</div>
  }

  // API 키 검증 로직
  return <div>Welcome</div>
}
```

### Accept-Language 확인

```tsx
import { headers } from 'next/headers'

export default async function Page() {
  const headersList = await headers()
  const acceptLanguage = headersList.get('accept-language')

  // 언어 파싱
  const languages = acceptLanguage?.split(',').map(lang => lang.split(';')[0].trim())

  return (
    <div>
      <h1>Preferred Languages</h1>
      <ul>
        {languages?.map((lang, index) => (
          <li key={index}>{lang}</li>
        ))}
      </ul>
    </div>
  )
}
```

### Route Handler에서 사용

```ts
// app/api/user/route.ts
import { headers } from 'next/headers'
import { NextResponse } from 'next/server'

export async function GET() {
  const headersList = await headers()
  const authorization = headersList.get('authorization')

  if (!authorization) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    )
  }

  // 인증 로직
  return NextResponse.json({ success: true })
}
```

---

## 중요 주의사항

> **Good to know**:
> * ✅ **반드시 async/await 사용**: `headers()`는 Promise를 반환하는 비동기 함수입니다
> * ✅ **읽기 전용**: 헤더를 읽을 수만 있고 설정/삭제는 불가능합니다
> * ✅ **동적 렌더링**: `headers()`는 Dynamic API로 작동하여 자동으로 동적 렌더링을 활성화합니다
> * ⚠️ **Server Components 전용**: Client Components에서는 직접 사용할 수 없습니다
> * ⚠️ **헤더 설정**: 응답 헤더를 설정하려면 `NextResponse` 또는 middleware를 사용해야 합니다

---

## Web Headers API 메서드 요약

| 메서드 | 설명 | 반환값 |
|--------|------|--------|
| `get(name)` | 특정 헤더 값 조회 | `string \| null` |
| `has(name)` | 헤더 존재 여부 확인 | `boolean` |
| `entries()` | 모든 키/값 쌍 반복 | `Iterator<[string, string]>` |
| `forEach(callback)` | 각 키/값 쌍에 함수 실행 | `void` |
| `keys()` | 모든 키 반복 | `Iterator<string>` |
| `values()` | 모든 값 반복 | `Iterator<string>` |

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v15.0.0-RC | 동기 함수에서 비동기 함수로 변경 (코드모드 지원) |
| v13.0.0 | `headers` 도입 |

---

## 관련 문서

- [cookies](./cookies.md)
- [Server Components](../../getting-started/05-server-and-client-components.md)
- [Web Headers API](https://developer.mozilla.org/en-US/docs/Web/API/Headers)
