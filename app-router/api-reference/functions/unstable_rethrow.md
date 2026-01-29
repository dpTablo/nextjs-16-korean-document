---
원문: https://nextjs.org/docs/app/api-reference/functions/unstable_rethrow
버전: 16.1.6
---

# unstable_rethrow

## 개요

`unstable_rethrow` 함수는 try/catch 블록에서 잡힌 **Next.js 내부 에러를 다시 발생**시켜, 프레임워크가 제어하는 예외를 애플리케이션 에러 핸들링에 의해 억제되지 않고 Next.js가 적절히 처리할 수 있도록 합니다.

> **⚠️ 불안정 API (Unstable)**
>
> 이 함수는 불안정하며 변경될 수 있습니다. 프로덕션 환경에서는 권장되지 않습니다.

---

## 문제 상황

Next.js 함수들(예: `notFound()`, `redirect()`)은 프레임워크가 처리하기 위해 에러를 발생시킵니다. 그러나 이러한 에러가 try/catch 블록에 잡히면 억제되어 의도한 동작이 실행되지 않습니다.

**문제 코드:**

```tsx
import { notFound } from 'next/navigation'

export default async function Page() {
  try {
    const post = await fetch('https://.../posts/1').then((res) => {
      if (res.status === 404) notFound() // ❌ catch에 잡혀서 404 페이지가 표시되지 않음
      if (!res.ok) throw new Error(res.statusText)
      return res.json()
    })
  } catch (err) {
    console.error(err) // notFound()도 여기서 잡힘
  }
}
```

---

## 해결 방법

`unstable_rethrow`를 사용하여 Next.js 내부 에러를 다시 발생시킵니다:

```tsx
import { notFound, unstable_rethrow } from 'next/navigation'

export default async function Page() {
  try {
    const post = await fetch('https://.../posts/1').then((res) => {
      if (res.status === 404) notFound()
      if (!res.ok) throw new Error(res.statusText)
      return res.json()
    })
  } catch (err) {
    unstable_rethrow(err) // ✅ Next.js 에러를 다시 발생
    console.error(err) // 일반 에러만 처리
  }
}
```

---

## 사용 가능한 위치

| 위치 | 사용 가능 |
|------|----------|
| **Server Components** | ✅ 예 |
| **Server Actions** | ✅ 예 |
| **Route Handlers** | ✅ 예 |
| **Client Components** | ✅ 예 |

---

## API 레퍼런스

### 함수 시그니처

```typescript
unstable_rethrow(error: unknown): void
```

### 매개변수

- `error` (unknown): catch 블록에서 잡힌 에러 객체

### 반환 값

- `void` - Next.js 내부 에러인 경우 다시 발생시키고, 일반 에러인 경우 아무 작업도 하지 않습니다

---

## unstable_rethrow가 필요한 API

### 네비게이션 함수

- `notFound()` - 404 페이지 표시
- `redirect()` - 리디렉션 수행
- `permanentRedirect()` - 영구 리디렉션

### 동적 API (정적 컨텍스트에서 사용 시)

- `cookies()` - 쿠키 읽기
- `headers()` - 헤더 읽기
- `searchParams` - 검색 파라미터
- `fetch(..., { cache: 'no-store' })` - 캐시 비활성화
- `fetch(..., { next: { revalidate: 0 } })` - 재검증 시간 0

---

## 실용적인 예제

### 1. notFound()와 함께 사용

```tsx
// app/posts/[id]/page.tsx
import { notFound, unstable_rethrow } from 'next/navigation'

interface Post {
  id: string
  title: string
  content: string
}

export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  let post: Post | null = null

  try {
    const res = await fetch(`https://api.example.com/posts/${id}`)

    if (res.status === 404) {
      notFound() // 404 페이지 표시
    }

    if (!res.ok) {
      throw new Error(`API 에러: ${res.statusText}`)
    }

    post = await res.json()
  } catch (err) {
    unstable_rethrow(err) // Next.js 에러 재발생
    console.error('포스트 로딩 실패:', err)
    // 일반 에러 처리
    throw err
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

### 2. redirect()와 함께 사용

```tsx
// app/dashboard/page.tsx
import { redirect, unstable_rethrow } from 'next/navigation'
import { verifySession } from '@/app/lib/dal'

export default async function DashboardPage() {
  let session = null

  try {
    session = await verifySession()

    if (!session) {
      redirect('/login') // 로그인 페이지로 리디렉트
    }
  } catch (err) {
    unstable_rethrow(err) // redirect 에러 재발생
    console.error('세션 검증 실패:', err)
    throw new Error('인증 에러')
  }

  return <main>대시보드</main>
}
```

### 3. Server Actions에서 사용

```ts
// app/actions.ts
'use server'

import { redirect, unstable_rethrow } from 'next/navigation'
import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  let postId: string | null = null

  try {
    const title = formData.get('title')
    const content = formData.get('content')

    const res = await fetch('https://api.example.com/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, content }),
    })

    if (!res.ok) {
      throw new Error('포스트 생성 실패')
    }

    const post = await res.json()
    postId = post.id

    revalidatePath('/posts')
    redirect(`/posts/${postId}`) // 새 포스트로 리디렉트
  } catch (err) {
    unstable_rethrow(err) // redirect 에러 재발생
    console.error('포스트 생성 에러:', err)
    return { error: '포스트를 생성할 수 없습니다' }
  }
}
```

### 4. Promise 체인에서 사용

```tsx
// app/products/[id]/page.tsx
import { notFound, unstable_rethrow } from 'next/navigation'

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params

  const product = await fetch(`https://api.example.com/products/${id}`)
    .then((res) => {
      if (res.status === 404) notFound()
      if (!res.ok) throw new Error('API 에러')
      return res.json()
    })
    .catch((err) => {
      unstable_rethrow(err) // Next.js 에러 재발생
      console.error('제품 로딩 실패:', err)
      throw err
    })

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  )
}
```

### 5. 리소스 정리와 함께 사용

```tsx
// app/streaming/page.tsx
import { unstable_rethrow } from 'next/navigation'

export default async function StreamingPage() {
  const connection = await createConnection()

  try {
    const data = await connection.fetchData()
    return <main>{data}</main>
  } catch (err) {
    // 리소스 정리 먼저
    await connection.close()

    // 그 다음 Next.js 에러 재발생
    unstable_rethrow(err)

    // 일반 에러 처리
    console.error('데이터 페칭 실패:', err)
    throw err
  }
}
```

### 6. finally 블록 활용

```tsx
// app/reports/page.tsx
import { notFound, unstable_rethrow } from 'next/navigation'

export default async function ReportPage() {
  const db = await connectToDatabase()

  try {
    const report = await db.reports.findFirst()

    if (!report) {
      notFound()
    }

    return <main>{report.data}</main>
  } catch (err) {
    unstable_rethrow(err) // Next.js 에러 재발생
    console.error('리포트 로딩 실패:', err)
    throw err
  } finally {
    // finally 블록에서 정리 작업
    await db.disconnect()
  }
}
```

### 7. 동적 API와 함께 사용

```tsx
// app/profile/page.tsx
import { cookies } from 'next/headers'
import { unstable_rethrow } from 'next/navigation'

export default async function ProfilePage() {
  let userId: string | null = null

  try {
    const cookieStore = await cookies()
    const sessionCookie = cookieStore.get('session')

    if (!sessionCookie) {
      throw new Error('세션이 없습니다')
    }

    userId = await verifySession(sessionCookie.value)
  } catch (err) {
    unstable_rethrow(err) // cookies() 에러 재발생
    console.error('세션 검증 실패:', err)
    return <div>로그인이 필요합니다</div>
  }

  const user = await fetchUser(userId)
  return <main>{user.name}</main>
}
```

---

## 사용 규칙

### 1. catch 블록 상단에서 호출

```tsx
// ✅ 올바른 사용
try {
  // code
} catch (err) {
  unstable_rethrow(err) // 첫 번째로 호출
  console.error(err)
}
```

```tsx
// ❌ 잘못된 사용
try {
  // code
} catch (err) {
  console.error(err)
  unstable_rethrow(err) // 너무 늦음
}
```

### 2. 에러 객체를 유일한 인자로 전달

```tsx
// ✅ 올바른 사용
catch (err) {
  unstable_rethrow(err)
}
```

```tsx
// ❌ 잘못된 사용
catch (err) {
  unstable_rethrow() // 인자 누락
}
```

### 3. 리소스 정리는 unstable_rethrow 전에 수행

```tsx
// ✅ 올바른 사용
catch (err) {
  clearInterval(timer) // 정리 먼저
  unstable_rethrow(err) // 그 다음 재발생
  console.error(err)
}
```

```tsx
// ✅ 또는 finally 블록 사용
try {
  // code
} catch (err) {
  unstable_rethrow(err)
  console.error(err)
} finally {
  clearInterval(timer) // 정리는 finally에서
}
```

---

## 언제 사용해야 하는가?

### ✅ 사용해야 할 때

1. **Next.js API와 일반 에러를 모두 처리해야 할 때**

   ```tsx
   try {
     const res = await fetch(url)
     if (res.status === 404) notFound() // Next.js API
     if (!res.ok) throw new Error('API 에러') // 일반 에러
     return res.json()
   } catch (err) {
     unstable_rethrow(err) // 둘 다 처리
     console.error(err)
   }
   ```

2. **try/catch 블록 안에서 Next.js 네비게이션을 사용할 때**

   ```tsx
   try {
     const session = await verifySession()
     if (!session) redirect('/login')
   } catch (err) {
     unstable_rethrow(err)
     console.error(err)
   }
   ```

3. **리소스 정리가 필요한 에러 처리**

   ```tsx
   try {
     // 데이터베이스 작업
   } catch (err) {
     await cleanup()
     unstable_rethrow(err)
   }
   ```

### ❌ 사용하지 않아야 할 때

1. **Next.js API를 호출자가 처리할 수 있을 때**

   ```tsx
   // ❌ 불필요한 사용
   async function getPost(id: string) {
     try {
       const res = await fetch(`/api/posts/${id}`)
       if (res.status === 404) notFound()
       return res.json()
     } catch (err) {
       unstable_rethrow(err)
       throw err
     }
   }

   // ✅ 리팩토링
   async function getPost(id: string) {
     const res = await fetch(`/api/posts/${id}`)
     if (res.status === 404) notFound() // 호출자가 처리
     return res.json()
   }
   ```

2. **일반 애플리케이션 에러만 처리할 때**

   ```tsx
   // ❌ 불필요한 사용
   try {
     const data = await processData()
     return data
   } catch (err) {
     unstable_rethrow(err) // Next.js API가 없으므로 불필요
     console.error(err)
     throw err
   }
   ```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **필요한 경우에만 사용**
   - Next.js API와 일반 에러를 모두 처리해야 할 때만 사용

2. **catch 블록 상단에 배치**
   ```tsx
   catch (err) {
     unstable_rethrow(err) // 첫 번째
     // 나머지 에러 처리
   }
   ```

3. **리소스 정리는 먼저 또는 finally에서**
   ```tsx
   catch (err) {
     cleanup() // 먼저
     unstable_rethrow(err)
   }
   // 또는
   finally {
     cleanup() // finally에서
   }
   ```

4. **가능하면 리팩토링 고려**
   - 혼합 에러 처리를 피할 수 있다면 코드 구조 개선

5. **타입 안전성 유지**
   ```tsx
   catch (err: unknown) {
     unstable_rethrow(err)
     if (err instanceof Error) {
       console.error(err.message)
     }
   }
   ```

### ❌ 피해야 할 사항

1. **무분별한 사용**
   ```tsx
   // ❌ 필요하지 않은데 사용
   catch (err) {
     unstable_rethrow(err) // Next.js API가 없음
   }
   ```

2. **정리 작업 후에 호출**
   ```tsx
   // ❌ 정리 작업이 실행 안 될 수 있음
   catch (err) {
     unstable_rethrow(err)
     cleanup() // Next.js 에러면 실행 안 됨
   }
   ```

3. **조건부 호출**
   ```tsx
   // ❌ 조건부로 호출하지 말 것
   catch (err) {
     if (someCondition) {
       unstable_rethrow(err)
     }
   }
   ```

---

## 내부 동작 원리

`unstable_rethrow`는 에러가 Next.js 내부 에러인지 확인하고, 맞다면 다시 발생시킵니다:

```typescript
// 간소화된 내부 구현 개념
function unstable_rethrow(error: unknown): void {
  if (isNextJsInternalError(error)) {
    throw error // Next.js 프레임워크가 처리
  }
  // 일반 에러는 그냥 반환 (catch 블록 계속 실행)
}
```

**Next.js 내부 에러 예:**
- `NEXT_NOT_FOUND` (notFound()에서 발생)
- `NEXT_REDIRECT` (redirect()에서 발생)
- 동적 API 에러 (정적 컨텍스트에서 cookies(), headers() 등)

---

## 실제 사용 패턴

### 패턴 1: API 응답 처리

```tsx
async function fetchData() {
  try {
    const res = await fetch(url)

    if (res.status === 404) notFound()
    if (res.status === 401) redirect('/login')
    if (!res.ok) throw new Error('API 에러')

    return res.json()
  } catch (err) {
    unstable_rethrow(err) // notFound, redirect 재발생

    // 일반 에러만 여기서 처리
    logError(err)
    throw new Error('데이터를 불러올 수 없습니다')
  }
}
```

### 패턴 2: 조건부 네비게이션

```tsx
async function checkAccess() {
  try {
    const session = await verifySession()

    if (!session) redirect('/login')
    if (!session.isAdmin) redirect('/403')

    return session
  } catch (err) {
    unstable_rethrow(err) // redirect 재발생

    // 세션 검증 에러 처리
    console.error('액세스 확인 실패:', err)
    throw err
  }
}
```

### 패턴 3: 데이터베이스 작업

```tsx
async function saveData(data: any) {
  const connection = await db.connect()

  try {
    const result = await connection.insert(data)

    if (!result.success) {
      throw new Error('저장 실패')
    }

    revalidatePath('/data')
    redirect('/data')
  } catch (err) {
    unstable_rethrow(err) // redirect 재발생

    // DB 에러 처리
    console.error('데이터 저장 에러:', err)
    throw err
  } finally {
    await connection.close()
  }
}
```

---

## 마이그레이션 가이드

### 기존 코드 (문제)

```tsx
// ❌ notFound()가 catch에 잡혀서 404 페이지가 표시되지 않음
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params

  try {
    const post = await fetchPost(id)
    return <article>{post.content}</article>
  } catch (err) {
    console.error(err)
    return <div>에러가 발생했습니다</div>
  }
}

async function fetchPost(id: string) {
  const res = await fetch(`/api/posts/${id}`)
  if (res.status === 404) notFound() // ❌ 상위 catch에 잡힘
  return res.json()
}
```

### 수정된 코드 (해결)

```tsx
// ✅ unstable_rethrow로 notFound() 재발생
import { unstable_rethrow } from 'next/navigation'

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params

  try {
    const post = await fetchPost(id)
    return <article>{post.content}</article>
  } catch (err) {
    unstable_rethrow(err) // ✅ notFound() 재발생
    console.error(err)
    return <div>에러가 발생했습니다</div>
  }
}

async function fetchPost(id: string) {
  const res = await fetch(`/api/posts/${id}`)
  if (res.status === 404) notFound()
  return res.json()
}
```

---

## 타입 정의

```typescript
function unstable_rethrow(error: unknown): void
```

---

## 관련 API

- [`notFound()`](./notFound.md) - 404 페이지 렌더링
- [`redirect()`](./redirect.md) - 리디렉션 수행
- [`permanentRedirect()`](./permanentRedirect.md) - 영구 리디렉션
- [`cookies()`](./cookies.md) - 쿠키 읽기
- [`headers()`](./headers.md) - 헤더 읽기

---

## 버전 정보

- **도입 버전:** Next.js 16.0.0 (추정)
- **현재 상태:** Unstable
- **안정화 예정:** 미정

---

## 요약

- **용도:** Next.js 내부 에러를 catch 블록에서 다시 발생시킴
- **사용 이유:** `notFound()`, `redirect()` 등이 try/catch에 잡혀도 정상 동작하도록
- **호출 위치:** catch 블록 상단
- **필요한 API:** `notFound()`, `redirect()`, `permanentRedirect()`, 동적 API 등
- **리소스 정리:** `unstable_rethrow()` 전에 수행하거나 finally 블록 사용
- **베스트 프랙티스:** 필요한 경우에만 사용, 가능하면 리팩토링 고려
- **주의:** 불안정 API, 변경될 수 있음
