# after

## 개요

`after`는 **응답이 완료된 후** 작업을 실행하도록 예약하는 Next.js 함수입니다. 응답을 차단하지 않고 로깅, 분석, 기타 사이드 이펙트와 같은 비차단 작업에 이상적입니다.

---

## 주요 특징

- **비차단 (Non-blocking)**: 작업이 사용자에게 응답을 보낸 후 실행됩니다
- **광범위한 호환성**: Server Components, Server Actions, Route Handlers, Proxy에서 작동합니다
- **동적 라우트로 만들지 않음**: 라우트를 동적으로 만들지 않으며, 정적 페이지의 경우 빌드 시 실행됩니다

---

## 기본 사용법

```tsx
import { after } from 'next/server'
import { log } from '@/app/utils'

export default function Layout({ children }) {
  after(() => {
    log() // 레이아웃이 렌더링되고 전송된 후 실행
  })
  return <>{children}</>
}
```

---

## 함수 시그니처

```tsx
import { after } from 'next/server'

after(callback: () => void | Promise<void>): void
```

### 매개변수

- **`callback`**: 응답이 완료된 후 실행할 함수
  - 동기 또는 비동기 함수 모두 가능
  - Promise를 반환할 수 있음

---

## 사용 사례

### 1. 로깅

```tsx
import { after } from 'next/server'
import { log } from '@/app/utils'

export default function Layout({ children }) {
  after(() => {
    // 사용자에게 응답을 보낸 후 로깅
    log()
  })
  return <>{children}</>
}
```

### 2. Server Actions에서 분석

```ts
import { after } from 'next/server'
import { cookies, headers } from 'next/headers'

export async function POST(request: Request) {
  after(async () => {
    // 요청 컨텍스트에 접근하여 분석 로그 기록
    const userAgent = (await headers()).get('user-agent') || 'unknown'
    const sessionId = (await cookies()).get('session-id')?.value || 'anonymous'

    // 분석 시스템에 로그 전송
    logUserAction({ sessionId, userAgent })
  })

  return new Response(JSON.stringify({ status: 'success' }))
}
```

### 3. Route Handlers에서 요청 추적

```ts
import { after } from 'next/server'
import { headers } from 'next/headers'

export async function GET(request: Request) {
  const data = await fetchData()

  after(async () => {
    // 응답 후 요청 메트릭 기록
    const userAgent = (await headers()).get('user-agent')
    trackRequest({ path: request.url, userAgent })
  })

  return Response.json(data)
}
```

### 4. 에러 발생 시에도 실행

```tsx
import { after } from 'next/server'
import { notFound } from 'next/navigation'

export default async function Page({ params }) {
  const data = await getData(params.id)

  if (!data) {
    after(() => {
      // notFound()가 호출되어도 실행됨
      logNotFound(params.id)
    })
    notFound()
  }

  return <div>{data.title}</div>
}
```

---

## 중요한 제약사항

### 1. Server Components에서 요청 API 사용 제한

❌ **불가능:**
```tsx
import { after } from 'next/server'
import { cookies } from 'next/headers'

export default function ServerComponent() {
  after(async () => {
    const session = (await cookies()).get('session')
    // ⚠️ Server Component에서는 작동하지 않음
    // React 렌더링 라이프사이클 이후에 실행되기 때문
  })
  return <div>Content</div>
}
```

✅ **가능 (Server Actions / Route Handlers):**
```ts
import { after } from 'next/server'
import { cookies } from 'next/headers'

export async function POST(request: Request) {
  after(async () => {
    const session = (await cookies()).get('session')
    // ✅ Route Handler에서는 작동함
  })
  return new Response('OK')
}
```

### 2. 에러 발생 시에도 실행

```tsx
import { after } from 'next/server'

export async function action() {
  after(() => {
    // 에러가 발생해도 실행됨
    logAction()
  })

  throw new Error('Something went wrong')
  // after 콜백은 여전히 실행됨
}
```

### 3. 타임아웃 제한

```tsx
// next.config.js
export default {
  experimental: {
    maxDuration: 60, // after 콜백도 이 제한을 따름
  },
}
```

- `after` 콜백은 플랫폼의 최대 duration 또는 `maxDuration` 설정을 따릅니다
- 시간 초과 시 실행이 중단될 수 있습니다

---

## 플랫폼 지원

| 플랫폼 | 지원 여부 |
|--------|----------|
| Node.js 서버 | ✅ 예 |
| Docker | ✅ 예 |
| 정적 내보내기 (Static export) | ❌ 아니오 |
| Vercel | ✅ 예 |
| Adapters (어댑터) | 🔷 플랫폼별 지원 |

---

## 주요 포인트

### 1. 비차단 작업
- 사용자 응답을 지연시키지 않고 백그라운드 작업 수행
- 응답 속도에 영향을 주지 않음

### 2. 넓은 호환성
```tsx
// Server Components
export default function Page() {
  after(() => log('page viewed'))
  return <div>Content</div>
}

// Server Actions
export async function submitForm(data: FormData) {
  after(() => log('form submitted'))
  await saveData(data)
}

// Route Handlers
export async function GET() {
  after(() => log('api called'))
  return Response.json({ data: 'value' })
}
```

### 3. 정적 페이지에서의 동작
- 정적으로 생성된 페이지의 경우 **빌드 시간**에 실행됩니다
- 라우트를 동적으로 만들지 않습니다

---

## 실용적인 예제

### 분석 추적

```tsx
import { after } from 'next/server'

export default function Page() {
  after(async () => {
    // 페이지 뷰 추적
    await fetch('https://analytics.example.com/track', {
      method: 'POST',
      body: JSON.stringify({
        page: '/products',
        timestamp: new Date().toISOString(),
      }),
    })
  })

  return <div>Products Page</div>
}
```

### 캐시 예열 (Cache Warming)

```tsx
import { after } from 'next/server'

export async function GET(request: Request) {
  const data = await fetchData()

  after(async () => {
    // 응답 후 관련 데이터 미리 캐싱
    await warmupRelatedCache(data.relatedIds)
  })

  return Response.json(data)
}
```

### 정리 작업 (Cleanup)

```tsx
import { after } from 'next/server'

export async function POST(request: Request) {
  const tempFile = await createTempFile()

  after(async () => {
    // 응답 후 임시 파일 정리
    await deleteTempFile(tempFile)
  })

  return new Response('File processed')
}
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **비차단 작업에 사용**
   ```tsx
   after(() => {
     logAnalytics() // 빠른 로깅
     sendMetrics()  // 비차단 메트릭
   })
   ```

2. **에러 처리 포함**
   ```tsx
   after(async () => {
     try {
       await logToExternalService()
     } catch (error) {
       console.error('Logging failed:', error)
     }
   })
   ```

### ❌ 피해야 할 사항

1. **중요한 작업에 사용하지 말 것**
   ```tsx
   // ❌ 잘못된 사용
   after(async () => {
     await sendConfirmationEmail() // 중요! after에 넣으면 안됨
   })

   // ✅ 올바른 사용
   await sendConfirmationEmail() // 응답 전에 실행
   after(() => logEmailSent())   // 로깅만 after에
   ```

2. **Server Component에서 요청 API 사용하지 말 것**
   ```tsx
   // ❌ Server Component에서
   after(async () => {
     await cookies() // 작동하지 않음
   })
   ```

---

## 관련 API

- [`revalidatePath()`](./revalidatePath.md) - 경로 재검증
- [`revalidateTag()`](./revalidateTag.md) - 태그 재검증
- [`cookies()`](./cookies.md) - 쿠키 접근
- [`headers()`](./headers.md) - 헤더 접근

---

## 버전 정보

- **도입 버전:** Next.js 15.0.0 (experimental)
- **안정 버전:** Next.js 15.1.0
- **현재 상태:** Stable

---

## 요약

- **용도:** 응답 완료 후 비차단 작업 실행
- **주 사용처:** 로깅, 분석, 캐시 예열, 정리 작업
- **장점:** 응답 속도에 영향 없음, 넓은 호환성
- **제약:** Server Component에서 요청 API 사용 불가, 타임아웃 제한
- **권장사항:** 중요하지 않은 사이드 이펙트에만 사용
