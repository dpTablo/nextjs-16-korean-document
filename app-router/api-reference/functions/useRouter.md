---
원문: https://nextjs.org/docs/app/api-reference/functions/use-router
버전: 16.1.6
---

# useRouter

`useRouter` hook은 Next.js App Router의 Client Components에서 프로그래밍 방식으로 라우팅을 변경할 수 있게 해줍니다.

> **권장사항**: 특정 요구사항이 없다면 [`<Link>` 컴포넌트](../link.md)를 사용하세요.

---

## 기본 사용법

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  )
}
```

---

## useRouter() 메서드

### router.push(href, options)

클라이언트 측 네비게이션을 수행합니다. 브라우저 히스토리 스택에 새 항목을 추가합니다.

```tsx
router.push(href: string, { scroll?: boolean })
```

**매개변수:**
- `href`: 이동할 URL
- `options.scroll`: 네비게이션 후 페이지 상단으로 스크롤할지 여부 (기본값: `true`)

**예제:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  )
}
```

### router.replace(href, options)

클라이언트 측 네비게이션을 수행합니다. 브라우저 히스토리 스택에 항목을 추가하지 않습니다.

```tsx
router.replace(href: string, { scroll?: boolean })
```

**매개변수:**
- `href`: 이동할 URL
- `options.scroll`: 네비게이션 후 페이지 상단으로 스크롤할지 여부 (기본값: `true`)

**예제:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.replace('/dashboard')}>
      Dashboard
    </button>
  )
}
```

### router.refresh()

현재 라우트를 새로고침합니다. 서버에 새로운 요청을 보내고, 데이터 요청을 재조회하며, Server Components를 재렌더링합니다.

클라이언트는 영향을 받지 않는 클라이언트 측 상태(`useState`, 스크롤 위치 등)를 잃지 않고 업데이트된 React Server Component payload를 병합합니다.

```tsx
router.refresh(): void
```

**예제:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.refresh()}>
      Refresh
    </button>
  )
}
```

### router.prefetch(href, options)

제공된 라우트를 미리 불러와서 더 빠른 클라이언트 측 전환을 제공합니다.

```tsx
router.prefetch(href: string, options?: { onInvalidate?: () => void })
```

**매개변수:**
- `href`: 미리 불러올 URL
- `options.onInvalidate`: 미리 불러온 데이터가 stale해질 때 호출되는 콜백 (최대 1회 호출)

**예제:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button
      onClick={() => router.prefetch('/dashboard')}
      onMouseEnter={() => router.prefetch('/dashboard')}
    >
      Dashboard
    </button>
  )
}
```

**onInvalidate 콜백 사용:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  const handlePrefetch = () => {
    router.prefetch('/dashboard', {
      onInvalidate: () => {
        console.log('Prefetched data is now stale')
      }
    })
  }

  return (
    <button onClick={handlePrefetch}>
      Prefetch Dashboard
    </button>
  )
}
```

### router.back()

브라우저 히스토리 스택의 이전 라우트로 돌아갑니다.

```tsx
router.back(): void
```

**예제:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.back()}>
      Back
    </button>
  )
}
```

### router.forward()

브라우저 히스토리 스택의 다음 라우트로 이동합니다.

```tsx
router.forward(): void
```

**예제:**

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.forward()}>
      Forward
    </button>
  )
}
```

---

## 사용 예제

### 스크롤 비활성화

네비게이션 후 페이지 상단으로 스크롤하지 않으려면 `scroll: false` 옵션을 사용합니다.

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  return (
    <button
      type="button"
      onClick={() => router.push('/dashboard', { scroll: false })}
    >
      Dashboard
    </button>
  )
}
```

### 라우터 이벤트 감지

`usePathname`과 `useSearchParams`를 조합하여 라우팅 변경을 감지할 수 있습니다.

```tsx
'use client'

import { useEffect } from 'react'
import { usePathname, useSearchParams } from 'next/navigation'

export function NavigationEvents() {
  const pathname = usePathname()
  const searchParams = useSearchParams()

  useEffect(() => {
    const url = `${pathname}?${searchParams}`
    console.log('URL changed to:', url)

    // 여기서 분석 이벤트 전송 등 가능
  }, [pathname, searchParams])

  return null
}
```

Layout에서 사용:

```tsx
// app/layout.tsx
import { Suspense } from 'react'
import { NavigationEvents } from './components/navigation-events'

export default function Layout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Suspense fallback={null}>
          <NavigationEvents />
        </Suspense>
      </body>
    </html>
  )
}
```

### 조건부 네비게이션

```tsx
'use client'

import { useRouter } from 'next/navigation'

export default function Page() {
  const router = useRouter()

  const handleSubmit = async (formData: FormData) => {
    const res = await fetch('/api/submit', {
      method: 'POST',
      body: formData,
    })

    if (res.ok) {
      router.push('/success')
    } else {
      router.push('/error')
    }
  }

  return (
    <form action={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  )
}
```

---

## next/router에서 마이그레이션

Pages Router의 `next/router`에서 App Router의 `next/navigation`으로 마이그레이션할 때 참고하세요.

| 이전 (`next/router`) | 현재 (`next/navigation`) |
|---------------------|-------------------------|
| `import { useRouter } from 'next/router'` | `import { useRouter } from 'next/navigation'` |
| `router.pathname` | [`usePathname()`](./usePathname.md) |
| `router.query` | [`useSearchParams()`](./useSearchParams.md) |
| `router.asPath` | [`usePathname()`](./usePathname.md) + [`useSearchParams()`](./useSearchParams.md) |
| `router.events` | `usePathname()` + `useSearchParams()` 조합 |
| `router.push('/path', undefined, { shallow: true })` | `router.push('/path')` (기본적으로 shallow) |
| `router.isFallback` | 제거됨 |
| `router.basePath` | 제거됨 |
| `router.locale` | 제거됨 (i18n은 middleware에서 처리) |
| `router.locales` | 제거됨 |
| `router.defaultLocale` | 제거됨 |
| `router.domainLocales` | 제거됨 |
| `router.isReady` | 제거됨 (Server Components는 항상 준비됨) |
| `router.isPreview` | [`draftMode()`](./draftMode.md)로 대체 |

---

## 중요한 주의사항

> **보안 경고**:
> * 신뢰할 수 없거나 살균되지 않은 URL을 `router.push()` 또는 `router.replace()`에 전송하지 마세요
> * `javascript:` URL 같은 것은 XSS(Cross-Site Scripting) 취약점을 일으킬 수 있습니다
> * 사용자 입력을 라우팅에 사용하기 전에 항상 검증하고 살균하세요

> **Good to know**:
> * `useRouter`는 Client Components에서만 사용 가능합니다
> * `<Link>` 컴포넌트가 대부분의 사용 사례에 더 적합합니다
> * `router.refresh()`는 Server Components를 재렌더링하지만 클라이언트 상태는 보존합니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.4.0 | `router.prefetch`에 선택적 `onInvalidate` 콜백 추가 |
| v13.0.0 | `next/navigation`의 `useRouter` 도입 |

---

## 관련 문서

- [usePathname](./usePathname.md)
- [useSearchParams](./useSearchParams.md)
- [useParams](./useParams.md)
- [Link 컴포넌트](../link.md)
- [Linking and Navigating](../../getting-started/04-linking-and-navigating.md)
