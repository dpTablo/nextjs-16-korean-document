# Single-Page Applications (SPAs)

Next.js에서 Single-Page Application을 구축하는 방법을 알아봅니다.

## 개요

Next.js는 Single-Page Application(SPA) 구축을 완벽하게 지원합니다:

- **빠른 라우트 전환** - 페이지 간 즉각적인 전환
- **프리페칭** - 자동으로 링크된 페이지 미리 로드
- **클라이언트 사이드 데이터 페칭** - 브라우저에서 데이터 가져오기
- **브라우저 API 접근** - `window`, `document` 등 사용 가능
- **서드파티 라이브러리 통합** - 기존 SPA 라이브러리와 호환

기존 SPA를 Next.js로 마이그레이션하고 점진적으로 서버 기능을 도입할 수 있습니다.

---

## Single-Page Application이란?

"순수 SPA"는 다음과 같이 정의됩니다:

- **클라이언트 사이드 렌더링 (CSR)**: 하나의 HTML 파일로 제공되며, 모든 라우팅과 데이터 페칭이 브라우저 JavaScript에서 처리됩니다
- **전체 페이지 새로고침 없음**: 클라이언트 사이드 JavaScript가 DOM을 조작하고 필요에 따라 데이터를 가져옵니다

---

## Next.js를 SPA에 사용하는 이유

### 1. 자동 코드 분할

여러 HTML 진입점을 생성하여 번들 크기를 줄입니다. 사용자가 방문하는 페이지에 필요한 코드만 로드됩니다.

### 2. `next/link` 프리페칭

빠른 페이지 전환을 제공하면서 라우팅 상태를 URL에 유지합니다.

### 3. 점진적 향상

정적 사이트나 순수 SPA로 시작한 후, 필요에 따라 서버 기능(React Server Components, Server Actions)을 추가할 수 있습니다.

---

## 핵심 패턴

### 1. React의 `use`와 Context Provider 사용

루트 레이아웃에서 데이터 페칭을 시작하고, 이를 기다리지(await) 않습니다.

**app/layout.tsx**
```tsx
import { UserProvider } from './user-provider'
import { getUser } from './user'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  let userPromise = getUser() // await 하지 않음

  return (
    <html lang="ko">
      <body>
        <UserProvider userPromise={userPromise}>{children}</UserProvider>
      </body>
    </html>
  )
}
```

**app/user-provider.tsx**
```tsx
'use client'

import { createContext, useContext, ReactNode } from 'react'

type User = {
  id: string
  name: string
  email: string
}

type UserContextType = {
  userPromise: Promise<User | null>
}

const UserContext = createContext<UserContextType | null>(null)

export function useUser(): UserContextType {
  let context = useContext(UserContext)
  if (context === null) {
    throw new Error('useUser must be used within a UserProvider')
  }
  return context
}

export function UserProvider({
  children,
  userPromise,
}: {
  children: ReactNode
  userPromise: Promise<User | null>
}) {
  return (
    <UserContext.Provider value={{ userPromise }}>
      {children}
    </UserContext.Provider>
  )
}
```

**app/profile.tsx**
```tsx
'use client'

import { use } from 'react'
import { useUser } from './user-provider'

export function Profile() {
  const { userPromise } = useUser()
  const user = use(userPromise)

  if (!user) {
    return <div>로그인이 필요합니다</div>
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  )
}
```

**장점:**
- 서버에서 요청을 일찍 시작하여 클라이언트 워터폴 제거
- 성능 향상
- `<Suspense>`와 함께 사용하여 로딩 상태 처리

---

### 2. SWR을 사용한 SPA

SWR 2.3.0 이상과 React 19 이상에서 세 가지 모드를 지원합니다:

| 모드 | 설명 |
|------|------|
| **클라이언트 전용** | `useSWR(key, fetcher)` |
| **서버 전용** | `useSWR(key)` + RSC 제공 데이터 |
| **혼합** | 두 접근 방식 결합 |

**app/layout.tsx**
```tsx
import { SWRConfig } from 'swr'
import { getUser } from './user'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <SWRConfig
          value={{
            fallback: {
              '/api/user': getUser(), // Promise를 전달
            },
          }}
        >
          {children}
        </SWRConfig>
      </body>
    </html>
  )
}
```

**app/profile.tsx**
```tsx
'use client'

import useSWR from 'swr'

const fetcher = (url: string) => fetch(url).then((res) => res.json())

export function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher)

  if (isLoading) return <div>로딩 중...</div>
  if (error) return <div>오류가 발생했습니다</div>

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  )
}
```

**SWR 설치:**
```bash
npm install swr
```

---

### 3. React Query를 사용한 SPA

Next.js는 React Query와 클라이언트 및 서버 모두에서 작동합니다.

**app/providers.tsx**
```tsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useState } from 'react'

export default function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000,
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  )
}
```

**app/layout.tsx**
```tsx
import Providers from './providers'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

**app/profile.tsx**
```tsx
'use client'

import { useQuery } from '@tanstack/react-query'

export function Profile() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user'],
    queryFn: () => fetch('/api/user').then((res) => res.json()),
  })

  if (isLoading) return <div>로딩 중...</div>
  if (error) return <div>오류가 발생했습니다</div>

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  )
}
```

**React Query 설치:**
```bash
npm install @tanstack/react-query
```

---

### 4. 브라우저 전용 렌더링

`next/dynamic`을 사용하여 클라이언트 컴포넌트의 사전 렌더링을 비활성화합니다.

```tsx
import dynamic from 'next/dynamic'

const ClientOnlyComponent = dynamic(() => import('./component'), {
  ssr: false,
})

export default function Page() {
  return (
    <div>
      <h1>페이지 제목</h1>
      <ClientOnlyComponent />
    </div>
  )
}
```

**사용 사례:**
- `window` 또는 `document`가 필요한 서드파티 라이브러리
- 브라우저 API에 의존하는 컴포넌트
- 서버 사이드 렌더링과 호환되지 않는 라이브러리

**로딩 상태 추가:**
```tsx
const ClientOnlyComponent = dynamic(() => import('./component'), {
  ssr: false,
  loading: () => <div>로딩 중...</div>,
})
```

---

### 5. Shallow 라우팅

네이티브 브라우저 API를 사용하여 전체 페이지 새로고침 없이 URL 상태를 업데이트합니다.

**app/ui/sort-products.tsx**
```tsx
'use client'

import { useSearchParams } from 'next/navigation'

export default function SortProducts() {
  const searchParams = useSearchParams()

  function updateSorting(sortOrder: string) {
    const urlSearchParams = new URLSearchParams(searchParams.toString())
    urlSearchParams.set('sort', sortOrder)
    window.history.pushState(null, '', `?${urlSearchParams.toString()}`)
  }

  return (
    <div>
      <button onClick={() => updateSorting('asc')}>오름차순</button>
      <button onClick={() => updateSorting('desc')}>내림차순</button>
    </div>
  )
}
```

**필터링 예시:**
```tsx
'use client'

import { useSearchParams, usePathname } from 'next/navigation'

export default function Filters() {
  const searchParams = useSearchParams()
  const pathname = usePathname()

  function updateFilter(key: string, value: string) {
    const params = new URLSearchParams(searchParams.toString())

    if (value) {
      params.set(key, value)
    } else {
      params.delete(key)
    }

    window.history.pushState(null, '', `${pathname}?${params.toString()}`)
  }

  return (
    <div>
      <select onChange={(e) => updateFilter('category', e.target.value)}>
        <option value="">전체</option>
        <option value="electronics">전자제품</option>
        <option value="clothing">의류</option>
      </select>

      <input
        type="text"
        placeholder="검색..."
        onChange={(e) => updateFilter('search', e.target.value)}
      />
    </div>
  )
}
```

---

### 6. 클라이언트 컴포넌트에서 Server Actions

Server Actions를 사용하여 수동 API 라우트 생성 없이 서버 기능을 호출합니다.

**app/actions.ts**
```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  // 데이터베이스에 저장
  await db.post.create({
    data: { title, content },
  })

  revalidatePath('/posts')
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } })
  revalidatePath('/posts')
}
```

**app/button.tsx**
```tsx
'use client'

import { createPost, deletePost } from './actions'
import { useActionState } from 'react'

export function CreatePostForm() {
  const [state, action, isPending] = useActionState(createPost, null)

  return (
    <form action={action}>
      <input name="title" placeholder="제목" required />
      <textarea name="content" placeholder="내용" required />
      <button type="submit" disabled={isPending}>
        {isPending ? '생성 중...' : '게시물 생성'}
      </button>
    </form>
  )
}

export function DeleteButton({ id }: { id: string }) {
  return (
    <button onClick={() => deletePost(id)}>
      삭제
    </button>
  )
}
```

**장점:**
- 수동 API 라우트 생성 불필요
- `useActionState`로 로딩 및 오류 상태 처리
- 자동 캐시 무효화

---

## 정적 내보내기 (선택사항)

완전한 정적 사이트 생성을 활성화합니다.

**next.config.ts**
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  output: 'export',
}

export default nextConfig
```

### 장점

- **라우트별 자동 코드 분할**: 단일 번들 대신
- **완전히 렌더링된 페이지**: 최소 스켈레톤 대신
- **즉각적인 클라이언트 사이드 전환**

### 제한사항

정적 내보내기에서는 다음 기능을 사용할 수 없습니다:

- Server Components (동적)
- Server Actions
- Route Handlers (동적)
- Middleware
- 동적 라우트 (generateStaticParams 없이)

---

## SPA 아키텍처 패턴

### 상태 관리

```tsx
'use client'

import { create } from 'zustand'

interface AppState {
  count: number
  increment: () => void
  decrement: () => void
}

const useStore = create<AppState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}))

export function Counter() {
  const { count, increment, decrement } = useStore()

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
    </div>
  )
}
```

### 클라이언트 사이드 인증

```tsx
'use client'

import { createContext, useContext, useState, useEffect } from 'react'

interface AuthContextType {
  user: User | null
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  isLoading: boolean
}

const AuthContext = createContext<AuthContextType | null>(null)

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    // 초기 인증 상태 확인
    const token = localStorage.getItem('token')
    if (token) {
      fetchUser(token).then(setUser).finally(() => setIsLoading(false))
    } else {
      setIsLoading(false)
    }
  }, [])

  const login = async (email: string, password: string) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    })
    const { user, token } = await response.json()
    localStorage.setItem('token', token)
    setUser(user)
  }

  const logout = () => {
    localStorage.removeItem('token')
    setUser(null)
  }

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  )
}

export function useAuth() {
  const context = useContext(AuthContext)
  if (!context) throw new Error('useAuth must be used within AuthProvider')
  return context
}
```

---

## 비교: Next.js SPA vs 기존 SPA

| 기능 | 기존 SPA (CRA, Vite) | Next.js SPA |
|------|---------------------|-------------|
| 코드 분할 | 수동 설정 필요 | 자동 |
| 라우팅 | 추가 라이브러리 필요 | 내장 |
| 프리페칭 | 수동 구현 | 자동 |
| SEO | 제한적 | SSG/SSR 옵션 |
| 서버 기능 | 별도 백엔드 필요 | 내장 (선택적) |
| 빌드 최적화 | 수동 설정 | 자동 |

---

## 마이그레이션 가이드

기존 SPA를 Next.js로 마이그레이션할 수 있습니다:

- [Create React App에서 마이그레이션](https://nextjs.org/docs/app/building-your-application/upgrading/from-create-react-app)
- [Vite에서 마이그레이션](https://nextjs.org/docs/app/building-your-application/upgrading/from-vite)
- [App Router로 점진적 전환](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration)

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **서버에서 데이터 페칭 시작**
   ```tsx
   // 루트 레이아웃에서 Promise 전달
   let dataPromise = fetchData() // await 없이
   ```

2. **코드 분할 활용**
   ```tsx
   const HeavyComponent = dynamic(() => import('./heavy'), {
     loading: () => <Skeleton />,
   })
   ```

3. **URL 상태 사용**
   ```tsx
   // 필터, 정렬 등을 URL에 유지
   window.history.pushState(null, '', `?filter=${value}`)
   ```

4. **Server Actions로 API 단순화**
   ```tsx
   // API 라우트 대신 Server Actions 사용
   'use server'
   export async function createItem() {}
   ```

### ❌ 피해야 할 것

1. **클라이언트 워터폴**
   ```tsx
   // ❌ 나쁜 예: 순차적 요청
   const user = await fetchUser()
   const posts = await fetchPosts(user.id)

   // ✅ 좋은 예: 병렬 요청
   const [user, posts] = await Promise.all([
     fetchUser(),
     fetchPosts(userId),
   ])
   ```

2. **과도한 클라이언트 상태**
   ```tsx
   // ❌ 모든 것을 클라이언트 상태로
   // ✅ URL, 서버 상태 활용
   ```

3. **불필요한 `use client`**
   ```tsx
   // ❌ 모든 컴포넌트에 'use client'
   // ✅ 필요한 곳에만 사용
   ```

---

## 다음 단계

- [Static Exports](./static-exports.md) - 정적 내보내기
- [Data Fetching Patterns](./data-fetching-patterns.md) - 데이터 페칭 패턴
- [Lazy Loading](./lazy-loading.md) - 지연 로딩
- [Third Party Libraries](./third-party-libraries.md) - 서드파티 라이브러리

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-15

**참고 자료:**
- [SWR 문서](https://swr.vercel.app/)
- [React Query 문서](https://tanstack.com/query/latest)
- [Zustand 문서](https://zustand-demo.pmnd.rs/)
