# usePathname

`usePathname`은 **Client Component** hook으로, 현재 URL의 **pathname**을 읽을 수 있게 해줍니다.

---

## 기본 사용법

```tsx
'use client'

import { usePathname } from 'next/navigation'

export default function ExampleClientComponent() {
  const pathname = usePathname()
  return <p>Current pathname: {pathname}</p>
}
```

---

## 함수 시그니처

```typescript
usePathname(): string
```

### 매개변수

없음

### 반환값

현재 URL의 pathname을 문자열로 반환합니다.

---

## 반환값 예제

| URL | 반환값 |
|-----|--------|
| `/` | `'/'` |
| `/dashboard` | `'/dashboard'` |
| `/dashboard?v=2` | `'/dashboard'` |
| `/blog/hello-world` | `'/blog/hello-world'` |
| `/blog/[slug]` (동적 라우트) | `/blog/hello-world` |

---

## 사용 예제

### 현재 경로 표시

```tsx
'use client'

import { usePathname } from 'next/navigation'

export default function CurrentPath() {
  const pathname = usePathname()

  return (
    <div>
      <h1>현재 페이지</h1>
      <p>경로: {pathname}</p>
    </div>
  )
}
```

### 활성 링크 스타일링

```tsx
'use client'

import Link from 'next/link'
import { usePathname } from 'next/navigation'

export function Navigation() {
  const pathname = usePathname()

  return (
    <nav>
      <Link
        href="/"
        className={pathname === '/' ? 'active' : ''}
      >
        Home
      </Link>
      <Link
        href="/dashboard"
        className={pathname === '/dashboard' ? 'active' : ''}
      >
        Dashboard
      </Link>
      <Link
        href="/blog"
        className={pathname.startsWith('/blog') ? 'active' : ''}
      >
        Blog
      </Link>
    </nav>
  )
}
```

### 경로 변경 감지

```tsx
'use client'

import { useEffect } from 'react'
import { usePathname, useSearchParams } from 'next/navigation'

export function NavigationEvents() {
  const pathname = usePathname()
  const searchParams = useSearchParams()

  useEffect(() => {
    console.log('Path changed to:', pathname)

    // 분석 이벤트 전송
    if (typeof window !== 'undefined') {
      window.gtag?.('config', 'GA_MEASUREMENT_ID', {
        page_path: pathname,
      })
    }
  }, [pathname, searchParams])

  return null
}
```

### 조건부 렌더링

```tsx
'use client'

import { usePathname } from 'next/navigation'

export function ConditionalHeader() {
  const pathname = usePathname()

  // 특정 경로에서만 헤더 표시
  if (pathname === '/login' || pathname === '/signup') {
    return null
  }

  return (
    <header>
      <h1>My App</h1>
    </header>
  )
}
```

### Breadcrumb 네비게이션

```tsx
'use client'

import Link from 'next/link'
import { usePathname } from 'next/navigation'

export function Breadcrumbs() {
  const pathname = usePathname()
  const segments = pathname.split('/').filter(Boolean)

  return (
    <nav>
      <Link href="/">Home</Link>
      {segments.map((segment, index) => {
        const href = `/${segments.slice(0, index + 1).join('/')}`
        const isLast = index === segments.length - 1

        return (
          <span key={href}>
            {' / '}
            {isLast ? (
              <span>{segment}</span>
            ) : (
              <Link href={href}>{segment}</Link>
            )}
          </span>
        )
      })}
    </nav>
  )
}
```

---

## Hydration Mismatch 방지

정적으로 사전 렌더링된 페이지에서 rewrites가 있는 경우 hydration 오류가 발생할 수 있습니다. 이를 방지하려면:

```tsx
'use client'

import { useEffect, useState } from 'react'
import { usePathname } from 'next/navigation'

export default function PathnameBadge() {
  const pathname = usePathname()
  const [clientPathname, setClientPathname] = useState('')

  useEffect(() => {
    setClientPathname(pathname)
  }, [pathname])

  return (
    <p>
      Current pathname: <span>{clientPathname}</span>
    </p>
  )
}
```

---

## 중요한 주의사항

> **Good to know**:
> * `usePathname`은 **Client Component**에서만 사용 가능합니다
> * Server Component에서는 사용할 수 없습니다
> * 쿼리 파라미터는 포함되지 않습니다 (쿼리는 [`useSearchParams()`](./useSearchParams.md) 사용)
> * Pages Router와 호환성을 위해 `usePathname`이 `null`을 반환할 수 있으므로 TypeScript에서 주의가 필요합니다

> **Pages Router 호환성**:
> * App Router와 Pages Router를 모두 사용하는 경우 `usePathname`이 `null`을 반환할 수 있습니다
> * TypeScript를 사용하는 경우 타입을 `string | null`로 처리해야 할 수 있습니다

> **cacheComponents 활성화 시**:
> * 동적 파라미터가 있는 경우 Suspense 경계가 필요할 수 있습니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.0.0 | `usePathname` 도입 |

---

## 관련 문서

- [useRouter](./useRouter.md)
- [useSearchParams](./useSearchParams.md)
- [useParams](./useParams.md)
- [Linking and Navigating](../../getting-started/04-linking-and-navigating.md)
