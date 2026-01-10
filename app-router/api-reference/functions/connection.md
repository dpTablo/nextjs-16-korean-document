# connection

## 개요

`connection()` 함수는 들어오는 사용자 요청을 기다린 후 계속 진행하여 컴포넌트의 동적 렌더링을 강제합니다. 이를 통해 빌드 시 정적 사전 렌더링을 방지합니다.

---

## 언제 사용하나요?

다음과 같은 경우에 `connection()`을 사용하세요:

- Dynamic API를 사용하지 않지만 런타임 렌더링이 필요한 컴포넌트
- 요청마다 변경되는 값에 접근하고 싶을 때 (예: `Math.random()`, `new Date()`)
- 각 렌더링마다 의도적으로 다른 결과가 필요한 경우

---

## 기본 사용법

```ts
import { connection } from 'next/server'

export default async function Page() {
  await connection()
  // 이 아래의 모든 것이 사전 렌더링에서 제외됩니다
  const rand = Math.random()
  return <span>{rand}</span>
}
```

---

## API 레퍼런스

### 함수 시그니처

```ts
function connection(): Promise<void>
```

### 매개변수

- 없음

### 반환 값

- `Promise<void>`: 소비용이 아닌 await만을 위한 Promise

---

## 상세 설명

### 1. 정적 렌더링 방지

```tsx
import { connection } from 'next/server'

export default async function RandomNumber() {
  await connection()

  // 이제 이 컴포넌트는 매 요청마다 동적으로 렌더링됩니다
  const random = Math.random()

  return (
    <div>
      <h1>랜덤 숫자</h1>
      <p>{random}</p>
    </div>
  )
}
```

**동작:**
- `connection()` 없이는 빌드 시 한 번만 `Math.random()`이 호출되어 정적 HTML이 생성됩니다
- `connection()` 사용 시 매 요청마다 새로운 랜덤 숫자가 생성됩니다

### 2. 요청별 타임스탬프

```tsx
import { connection } from 'next/server'

export default async function CurrentTime() {
  await connection()

  const now = new Date().toISOString()

  return <p>현재 시간: {now}</p>
}
```

### 3. Dynamic API와의 차이

```tsx
// Dynamic API 사용 (자동으로 동적 렌더링)
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = await cookies()
  // cookies() 사용으로 인해 자동으로 동적 렌더링
  return <div>Content</div>
}
```

```tsx
// connection() 사용 (명시적 동적 렌더링)
import { connection } from 'next/server'

export default async function Page() {
  await connection()
  // Dynamic API 없이도 동적 렌더링 강제
  const random = Math.random()
  return <div>{random}</div>
}
```

---

## 주요 포인트

### 1. `unstable_noStore()` 대체

✅ **권장 (connection):**
```tsx
import { connection } from 'next/server'

export default async function Component() {
  await connection()
  // 동적 렌더링
}
```

❌ **Deprecated (unstable_noStore):**
```tsx
import { unstable_noStore as noStore } from 'next/cache'

export default async function Component() {
  noStore() // v15+에서 deprecated
  // 동적 렌더링
}
```

### 2. Dynamic API 없이도 동적 렌더링

- Dynamic API (`cookies()`, `headers()`, `searchParams` 등)를 사용하면 자동으로 동적 렌더링됩니다
- **`connection()`은 Dynamic API를 사용하지 않을 때만 필요합니다**

### 3. 안정 버전

- **v15.0.0**부터 stable API
- `unstable_noStore()`의 공식 대체 함수

---

## 사용 사례

### 1. 매 요청마다 다른 콘텐츠

```tsx
import { connection } from 'next/server'

const greetings = ['안녕하세요', 'Hello', 'Hola', 'Bonjour', 'こんにちは']

export default async function RandomGreeting() {
  await connection()

  const greeting = greetings[Math.floor(Math.random() * greetings.length)]

  return <h1>{greeting}!</h1>
}
```

### 2. 서버 시간 표시

```tsx
import { connection } from 'next/server'

export default async function ServerTime() {
  await connection()

  const serverTime = new Date().toLocaleString('ko-KR', {
    timeZone: 'Asia/Seoul',
  })

  return (
    <div>
      <h2>서버 시간</h2>
      <time>{serverTime}</time>
    </div>
  )
}
```

### 3. A/B 테스트

```tsx
import { connection } from 'next/server'

export default async function ABTestComponent() {
  await connection()

  // 매 요청마다 랜덤하게 버전 선택
  const showVersionA = Math.random() > 0.5

  return showVersionA ? <VersionA /> : <VersionB />
}
```

---

## Dynamic API와의 비교

| 상황 | 사용할 API | 이유 |
|------|-----------|------|
| 쿠키/헤더 접근 | Dynamic API (`cookies()`, `headers()`) | 자동으로 동적 렌더링 |
| 검색 파라미터 | `searchParams` prop | 자동으로 동적 렌더링 |
| 랜덤 값/시간 | `connection()` | Dynamic API 없이 동적 렌더링 필요 |
| 사용자별 다른 콘텐츠 | `connection()` 또는 Dynamic API | 요구사항에 따라 선택 |

---

## 베스트 프랙티스

### ✅ 권장사항

1. **필요할 때만 사용**
   ```tsx
   // ✅ 필요한 경우에만
   export default async function Dynamic() {
     await connection()
     const random = Math.random()
     return <div>{random}</div>
   }

   // ✅ 정적 렌더링 가능하면 사용하지 않기
   export default function Static() {
     return <div>정적 콘텐츠</div>
   }
   ```

2. **컴포넌트 최상단에 배치**
   ```tsx
   export default async function Page() {
     await connection() // 최상단에 배치

     const data = await fetchData()
     return <div>{data}</div>
   }
   ```

### ❌ 피해야 할 사항

1. **불필요한 사용**
   ```tsx
   // ❌ 이미 Dynamic API 사용 중
   import { cookies } from 'next/headers'
   import { connection } from 'next/server'

   export default async function Page() {
     await connection() // 불필요! cookies()가 이미 동적 렌더링
     const cookieStore = await cookies()
   }
   ```

2. **모든 컴포넌트에 적용**
   ```tsx
   // ❌ 정적 렌더링 가능한데도 사용
   export default async function StaticContent() {
     await connection() // 불필요!
     return <div>항상 같은 콘텐츠</div>
   }
   ```

---

## 마이그레이션 가이드

### `unstable_noStore()`에서 마이그레이션

**Before (v14):**
```tsx
import { unstable_noStore as noStore } from 'next/cache'

export default async function Component() {
  noStore()
  const data = await getData()
  return <div>{data}</div>
}
```

**After (v15+):**
```tsx
import { connection } from 'next/server'

export default async function Component() {
  await connection()
  const data = await getData()
  return <div>{data}</div>
}
```

---

## 성능 고려사항

### 정적 vs 동적 렌더링

```tsx
// 정적 렌더링 (빠름, 캐시 가능)
export default function StaticPage() {
  return <div>정적 콘텐츠</div>
}
// 빌드 시 한 번만 렌더링, CDN 캐싱 가능
```

```tsx
// 동적 렌더링 (느림, 매 요청마다 렌더링)
import { connection } from 'next/server'

export default async function DynamicPage() {
  await connection()
  return <div>{Math.random()}</div>
}
// 매 요청마다 서버에서 렌더링
```

**권장사항:** 가능한 한 정적 렌더링을 사용하고, 꼭 필요한 경우에만 `connection()`을 사용하세요.

---

## 관련 API

- [`unstable_noStore()`](./unstable_noStore.md) - Deprecated, `connection()`으로 대체
- [`cookies()`](./cookies.md) - 쿠키 접근 (자동으로 동적 렌더링)
- [`headers()`](./headers.md) - 헤더 접근 (자동으로 동적 렌더링)
- [Dynamic APIs](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-apis) - 동적 렌더링을 트리거하는 API들

---

## 버전 정보

- **도입 버전:** Next.js 15.0.0-RC (experimental)
- **안정 버전:** Next.js 15.0.0
- **현재 상태:** Stable
- **대체 API:** `unstable_noStore()` (deprecated)

---

## 요약

- **용도:** Dynamic API 없이 동적 렌더링 강제
- **주 사용처:** 랜덤 값, 현재 시간, 요청별 다른 콘텐츠
- **장점:** `unstable_noStore()`보다 명확한 의도, stable API
- **제약:** 성능 영향 (정적 렌더링 불가)
- **권장사항:** 꼭 필요한 경우에만 사용, 가능하면 정적 렌더링 우선
