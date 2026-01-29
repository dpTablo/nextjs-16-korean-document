---
원문: https://nextjs.org/docs/app/getting-started/linking-and-navigating
버전: 16.1.6
---

# 링크 및 네비게이션

## 개요

Next.js에서 라우트는 기본적으로 서버에서 렌더링됩니다. Next.js는 내장된 프리페칭, 스트리밍, 클라이언트 사이드 전환을 포함하여 네비게이션이 빠르고 반응적으로 유지되도록 합니다. 이 가이드는 네비게이션 작동 방식과 동적 라우트 및 느린 네트워크를 위한 최적화 방법을 설명합니다.

---

## 네비게이션 작동 방식

### 주요 개념
- 서버 렌더링
- 프리페칭
- 스트리밍
- 클라이언트 사이드 전환

### 서버 렌더링

레이아웃과 페이지는 기본적으로 React 서버 컴포넌트입니다. 서버 컴포넌트 페이로드는 클라이언트로 전송되기 전에 서버에서 생성됩니다.

**두 가지 서버 렌더링 유형:**

1. **정적 렌더링 (프리렌더링)** - 빌드 시점 또는 재검증 중에 발생; 결과가 캐시됨
2. **동적 렌더링** - 클라이언트 요청에 응답하여 요청 시점에 발생

Next.js는 라우트를 프리페칭하고 클라이언트 사이드 전환을 수행하여 서버 응답 대기 지연을 해결합니다.

> **참고:** HTML도 초기 방문을 위해 생성됩니다.

### 프리페칭

프리페칭은 사용자가 네비게이트하기 전에 백그라운드에서 라우트를 로드하여 네비게이션을 즉각적으로 느끼게 합니다.

Next.js는 `<Link>` 컴포넌트로 링크된 라우트가 뷰포트에 들어올 때 자동으로 프리페칭합니다.

```tsx filename="app/layout.tsx"
import Link from 'next/link'

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <nav>
          {/* 링크가 호버되거나 뷰포트에 들어올 때 프리페칭됨 */}
          <Link href="/blog">블로그</Link>
          {/* 프리페칭 없음 */}
          <a href="/contact">연락처</a>
        </nav>
        {children}
      </body>
    </html>
  )
}
```

```jsx filename="app/layout.js"
import Link from 'next/link'

export default function Layout() {
  return (
    <html>
      <body>
        <nav>
          {/* 링크가 호버되거나 뷰포트에 들어올 때 프리페칭됨 */}
          <Link href="/blog">블로그</Link>
          {/* 프리페칭 없음 */}
          <a href="/contact">연락처</a>
        </nav>
        {children}
      </body>
    </html>
  )
}
```

**라우트 유형별 프리페칭 동작:**
- **정적 라우트:** 전체 라우트가 프리페칭됨
- **동적 라우트:** 프리페칭이 건너뛰어지거나 `loading.tsx`가 있는 경우 부분적으로 프리페칭됨

### 스트리밍

스트리밍을 통해 서버는 전체 라우트가 렌더링될 때까지 기다리는 대신 준비되는 즉시 동적 라우트의 부분을 클라이언트에 전송할 수 있습니다.

동적 라우트의 경우, **부분적으로 프리페칭**될 수 있습니다 - 공유 레이아웃과 로딩 스켈레톤을 미리 요청할 수 있습니다.

**스트리밍을 사용하려면 라우트 폴더에 `loading.tsx`를 생성하세요:**

```tsx filename="app/dashboard/loading.tsx"
export default function Loading() {
  // 라우트가 로딩되는 동안 표시될 폴백 UI를 추가합니다.
  return <LoadingSkeleton />
}
```

```jsx filename="app/dashboard/loading.js"
export default function Loading() {
  // 라우트가 로딩되는 동안 표시될 폴백 UI를 추가합니다.
  return <LoadingSkeleton />
}
```

내부적으로 Next.js는 자동으로 `page.tsx` 내용을 `<Suspense>` 경계로 감쌉니다. 프리페칭된 폴백 UI는 라우트가 로드되는 동안 표시되고 실제 콘텐츠로 교체됩니다.

> **참고:** 중첩된 컴포넌트를 위한 로딩 UI를 생성하기 위해 `<Suspense>`를 사용할 수도 있습니다.

**`loading.tsx`의 이점:**
- 사용자를 위한 즉각적인 네비게이션 및 시각적 피드백
- 공유 레이아웃이 인터랙티브를 유지하고 네비게이션이 중단 가능
- Core Web Vitals 개선: TTFB, FCP, TTI

### 클라이언트 사이드 전환

전통적으로 네비게이션은 전체 페이지 로드를 트리거하여 상태를 지우고, 스크롤 위치를 리셋하며, 인터랙티브를 차단합니다.

Next.js는 `<Link>` 컴포넌트를 사용하여 클라이언트 사이드 전환으로 이를 방지합니다. 리로드하는 대신:
- 공유 레이아웃과 UI를 유지
- 프리페칭된 로딩 상태 또는 새 페이지(사용 가능한 경우)로 현재 페이지를 교체

클라이언트 사이드 전환은 서버 렌더링된 앱이 클라이언트 렌더링된 앱처럼 *느껴지게* 하여 동적 라우트에서도 빠른 전환을 가능하게 합니다.

---

## 전환을 느리게 만들 수 있는 것

### `loading.tsx` 없는 동적 라우트

동적 라우트로 네비게이트할 때, 클라이언트는 결과를 표시하기 전에 서버 응답을 기다려야 하므로 앱이 응답하지 않는 것처럼 보입니다.

**권장사항:** 동적 라우트에 `loading.tsx`를 추가하여 부분 프리페칭을 활성화하고 즉각적인 네비게이션을 트리거하며 로딩 UI를 표시하세요.

```tsx filename="app/blog/[slug]/loading.tsx"
export default function Loading() {
  return <LoadingSkeleton />
}
```

```jsx filename="app/blog/[slug]/loading.js"
export default function Loading() {
  return <LoadingSkeleton />
}
```

> **참고:** 개발 모드에서는 Next.js Devtools를 사용하여 라우트가 정적인지 동적인지 확인하세요.

### `generateStaticParams` 없는 동적 세그먼트

동적 세그먼트가 프리렌더링될 수 있지만 `generateStaticParams`가 없어서 프리렌더링되지 않는다면, 라우트는 요청 시점에 동적 렌더링으로 폴백됩니다.

**해결책:** `generateStaticParams`를 추가하여 빌드 시점에 정적 생성을 보장하세요:

```tsx filename="app/blog/[slug]/page.tsx"
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}

export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  // ...
}
```

```jsx filename="app/blog/[slug]/page.js"
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}

export default async function Page({ params }) {
  const { slug } = await params
  // ...
}
```

### 느린 네트워크

느리거나 불안정한 네트워크에서는 사용자가 링크를 클릭하기 전에 프리페칭이 완료되지 않을 수 있습니다. 전환이 진행 중일 때 즉각적인 피드백을 표시하려면 `useLinkStatus` 훅을 사용하세요.

```tsx filename="app/ui/loading-indicator.tsx"
'use client'

import { useLinkStatus } from 'next/link'

export default function LoadingIndicator() {
  const { pending } = useLinkStatus()
  return (
    <span aria-hidden className={`link-hint ${pending ? 'is-pending' : ''}`} />
  )
}
```

```jsx filename="app/ui/loading-indicator.js"
'use client'

import { useLinkStatus } from 'next/link'

export default function LoadingIndicator() {
  const { pending } = useLinkStatus()
  return (
    <span aria-hidden className={`link-hint ${pending ? 'is-pending' : ''}`} />
  )
}
```

**힌트 디바운스**하기 위해 초기 애니메이션 지연(예: 100ms)을 추가하고 보이지 않게 시작하세요(예: `opacity: 0`). 네비게이션이 지연보다 오래 걸리는 경우에만 인디케이터가 표시됩니다.

### 프리페칭 비활성화

`<Link>` 컴포넌트에서 `prefetch={false}`를 설정하여 프리페칭을 해제합니다. 큰 링크 목록으로 불필요한 리소스 사용을 피하는 데 유용합니다.

```tsx
<Link prefetch={false} href="/blog">
  블로그
</Link>
```

**트레이드오프:**
- **정적 라우트**는 사용자가 클릭할 때만 페치됨
- **동적 라우트**는 클라이언트 네비게이션 전에 서버 렌더링 필요

**대안:** 가능성 있는 라우트로 프리페칭을 제한하기 위해 호버 시에만 프리페칭:

```tsx filename="app/ui/hover-prefetch-link.tsx"
'use client'

import Link from 'next/link'
import { useState } from 'react'

function HoverPrefetchLink({
  href,
  children,
}: {
  href: string
  children: React.ReactNode
}) {
  const [active, setActive] = useState(false)

  return (
    <Link
      href={href}
      prefetch={active ? null : false}
      onMouseEnter={() => setActive(true)}
    >
      {children}
    </Link>
  )
}
```

```jsx filename="app/ui/hover-prefetch-link.js"
'use client'

import Link from 'next/link'
import { useState } = require('react')

function HoverPrefetchLink({ href, children }) {
  const [active, setActive] = useState(false)

  return (
    <Link
      href={href}
      prefetch={active ? null : false}
      onMouseEnter={() => setActive(true)}
    >
      {children}
    </Link>
  )
}
```

### 하이드레이션이 완료되지 않음

`<Link>`는 클라이언트 컴포넌트이며 라우트를 프리페칭하기 전에 하이드레이트되어야 합니다. 큰 JavaScript 번들은 하이드레이션을 지연시킬 수 있습니다.

**개선사항:**
- `@next/bundle-analyzer` 플러그인을 사용하여 번들 크기를 식별하고 줄이기
- 가능한 경우 클라이언트에서 서버로 로직 이동

---

## 예제

### 네이티브 History API

Next.js는 네이티브 `window.history.pushState` 및 `window.history.replaceState` 메서드 사용을 허용하여 페이지를 리로드하지 않고 브라우저의 히스토리 스택을 업데이트합니다.

이러한 호출은 Next.js Router에 통합되어 `usePathname` 및 `useSearchParams`와 동기화됩니다.

#### `window.history.pushState`

브라우저의 히스토리 스택에 새 항목을 추가합니다. 사용자는 이전 상태로 다시 네비게이트할 수 있습니다. 예: 제품 정렬.

```tsx fileName="app/ui/sort-products.tsx"
'use client'

import { useSearchParams } from 'next/navigation'

export default function SortProducts() {
  const searchParams = useSearchParams()

  function updateSorting(sortOrder: string) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }

  return (
    <>
      <button onClick={() => updateSorting('asc')}>오름차순 정렬</button>
      <button onClick={() => updateSorting('desc')}>내림차순 정렬</button>
    </>
  )
}
```

```jsx fileName="app/ui/sort-products.js"
'use client'

import { useSearchParams } from 'next/navigation'

export default function SortProducts() {
  const searchParams = useSearchParams()

  function updateSorting(sortOrder) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }

  return (
    <>
      <button onClick={() => updateSorting('asc')}>오름차순 정렬</button>
      <button onClick={() => updateSorting('desc')}>내림차순 정렬</button>
    </>
  )
}
```

#### `window.history.replaceState`

브라우저의 히스토리 스택에서 현재 항목을 대체합니다. 사용자는 이전 상태로 다시 네비게이트할 수 없습니다. 예: 애플리케이션 로케일 전환.

```tsx fileName="app/ui/locale-switcher.tsx"
'use client'

import { usePathname } from 'next/navigation'

export function LocaleSwitcher() {
  const pathname = usePathname()

  function switchLocale(locale: string) {
    // 예: '/en/about' 또는 '/fr/contact'
    const newPath = `/${locale}${pathname}`
    window.history.replaceState(null, '', newPath)
  }

  return (
    <>
      <button onClick={() => switchLocale('en')}>English</button>
      <button onClick={() => switchLocale('ko')}>한국어</button>
    </>
  )
}
```

```jsx fileName="app/ui/locale-switcher.js"
'use client'

import { usePathname } from 'next/navigation'

export function LocaleSwitcher() {
  const pathname = usePathname()

  function switchLocale(locale) {
    // 예: '/en/about' 또는 '/fr/contact'
    const newPath = `/${locale}${pathname}`
    window.history.replaceState(null, '', newPath)
  }

  return (
    <>
      <button onClick={() => switchLocale('en')}>English</button>
      <button onClick={() => switchLocale('ko')}>한국어</button>
    </>
  )
}
```

---

## 관련 리소스

- **[Link 컴포넌트](/docs/app/api-reference/components/link.md)** - 내장 `next/link` 컴포넌트로 빠른 클라이언트 사이드 네비게이션 활성화
- **[loading.js](/docs/app/api-reference/file-conventions/loading.md)** - loading.js 파일의 API 참조
- **[프리페칭](/docs/app/guides/prefetching.md)** - Next.js에서 프리페칭을 설정하는 방법 알아보기
