---
원문: https://nextjs.org/docs/app/api-reference/functions/use-selected-layout-segment
버전: 16.1.6
---

# useSelectedLayoutSegment

## 개요

`useSelectedLayoutSegment`는 호출된 Layout의 **한 단계 아래** 활성 라우트 세그먼트를 읽는 **Client Component hook**입니다. 활성 자식 세그먼트에 따라 스타일이 변경되는 탭과 같은 동적 네비게이션 UI를 만드는 데 유용합니다.

---

## 기본 사용법

```tsx
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'

export default function ExampleClientComponent() {
  const segment = useSelectedLayoutSegment()
  return <p>활성 세그먼트: {segment}</p>
}
```

---

## 매개변수

```tsx
const segment = useSelectedLayoutSegment(parallelRoutesKey?: string)
```

- **`parallelRoutesKey`** (선택사항): 특정 parallel route 슬롯 내에서 활성 라우트 세그먼트를 읽을 수 있습니다.

---

## 반환 값

활성 세그먼트의 **문자열** 또는 세그먼트가 없으면 **`null`**을 반환합니다.

### 반환 예제

| Layout | URL | 반환 값 |
|--------|-----|--------|
| `app/layout.js` | `/` | `null` |
| `app/layout.js` | `/dashboard` | `'dashboard'` |
| `app/dashboard/layout.js` | `/dashboard` | `null` |
| `app/dashboard/layout.js` | `/dashboard/settings` | `'settings'` |
| `app/dashboard/layout.js` | `/dashboard/analytics` | `'analytics'` |

---

## 주요 포인트

- **Client Component hook**입니다 (`'use client'` 필수)
- Layouts는 기본적으로 **Server Components**이므로 일반적으로 Client Component를 통해 Layout으로 hook을 가져옵니다
- **한 단계 아래**의 세그먼트만 반환합니다 (모든 활성 세그먼트는 `useSelectedLayoutSegments` 사용)

---

## 실전 예제

### 활성 링크 컴포넌트

```tsx
// components/blog-nav-link.tsx
'use client'

import Link from 'next/link'
import { useSelectedLayoutSegment } from 'next/navigation'

export default function BlogNavLink({
  slug,
  children,
}: {
  slug: string
  children: React.ReactNode
}) {
  const segment = useSelectedLayoutSegment()
  const isActive = slug === segment

  return (
    <Link
      href={`/blog/${slug}`}
      style={{ fontWeight: isActive ? 'bold' : 'normal' }}
    >
      {children}
    </Link>
  )
}
```

```tsx
// app/blog/layout.tsx - Server Component
import { BlogNavLink } from './blog-nav-link'
import getFeaturedPosts from './get-featured-posts'

export default async function Layout({ children }: { children: React.ReactNode }) {
  const featuredPosts = await getFeaturedPosts()

  return (
    <div>
      {featuredPosts.map((post) => (
        <BlogNavLink key={post.id} slug={post.slug}>
          {post.title}
        </BlogNavLink>
      ))}
      <div>{children}</div>
    </div>
  )
}
```

### 탭 네비게이션

```tsx
// components/dashboard-tabs.tsx
'use client'

import Link from 'next/link'
import { useSelectedLayoutSegment } from 'next/navigation'

export default function DashboardTabs() {
  const segment = useSelectedLayoutSegment()

  const tabs = [
    { name: '개요', href: '/dashboard', segment: null },
    { name: '분석', href: '/dashboard/analytics', segment: 'analytics' },
    { name: '설정', href: '/dashboard/settings', segment: 'settings' },
  ]

  return (
    <nav className="flex space-x-4">
      {tabs.map((tab) => {
        const isActive = segment === tab.segment

        return (
          <Link
            key={tab.name}
            href={tab.href}
            className={`px-3 py-2 rounded-md ${
              isActive
                ? 'bg-blue-500 text-white'
                : 'bg-gray-200 text-gray-700'
            }`}
          >
            {tab.name}
          </Link>
        )
      })}
    </nav>
  )
}
```

```tsx
// app/dashboard/layout.tsx
import DashboardTabs from '@/components/dashboard-tabs'

export default function DashboardLayout({ children }) {
  return (
    <div>
      <DashboardTabs />
      <main>{children}</main>
    </div>
  )
}
```

### Parallel Routes와 함께 사용

```tsx
// components/team-nav.tsx
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'

export default function TeamNav() {
  // 기본 슬롯의 세그먼트
  const segment = useSelectedLayoutSegment()

  // @team 슬롯의 세그먼트
  const teamSegment = useSelectedLayoutSegment('team')

  return (
    <div>
      <p>주요 세그먼트: {segment}</p>
      <p>팀 세그먼트: {teamSegment}</p>
    </div>
  )
}
```

### 조건부 렌더링

```tsx
// components/sidebar.tsx
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'

export default function Sidebar() {
  const segment = useSelectedLayoutSegment()

  // 특정 세그먼트에서만 사이드바 표시
  if (segment === 'settings' || segment === 'profile') {
    return (
      <aside>
        <SettingsMenu />
      </aside>
    )
  }

  return null
}
```

---

## useSelectedLayoutSegments

모든 활성 세그먼트를 배열로 가져오려면 `useSelectedLayoutSegments`를 사용하세요:

```tsx
'use client'

import { useSelectedLayoutSegments } from 'next/navigation'

export default function Breadcrumbs() {
  const segments = useSelectedLayoutSegments()

  return (
    <nav>
      <ol>
        <li>
          <a href="/">홈</a>
        </li>
        {segments.map((segment, index) => (
          <li key={segment}>
            <a href={`/${segments.slice(0, index + 1).join('/')}`}>
              {segment}
            </a>
          </li>
        ))}
      </ol>
    </nav>
  )
}
```

### 반환 예제

| Layout | URL | 반환 값 |
|--------|-----|--------|
| `app/layout.js` | `/` | `[]` |
| `app/layout.js` | `/dashboard` | `['dashboard']` |
| `app/layout.js` | `/dashboard/settings` | `['dashboard', 'settings']` |
| `app/dashboard/layout.js` | `/dashboard` | `[]` |
| `app/dashboard/layout.js` | `/dashboard/settings` | `['settings']` |

---

## 모범 사례

### 1. Client Component로 분리

```tsx
// ✅ 좋은 예 - Client Component로 분리
// components/nav.tsx
'use client'

export function Nav() {
  const segment = useSelectedLayoutSegment()
  // ...
}

// app/layout.tsx (Server Component)
import { Nav } from '@/components/nav'

export default function Layout({ children }) {
  return (
    <>
      <Nav />
      {children}
    </>
  )
}
```

### 2. 타입 안정성

```tsx
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'

type Segment = 'analytics' | 'settings' | 'profile' | null

export default function TypedNav() {
  const segment = useSelectedLayoutSegment() as Segment

  // 타입 안전한 조건문
  if (segment === 'analytics') {
    return <AnalyticsNav />
  }

  return <DefaultNav />
}
```

### 3. 동적 스타일링

```tsx
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'
import clsx from 'clsx'

export function NavItem({ href, segment, children }) {
  const activeSegment = useSelectedLayoutSegment()

  return (
    <a
      href={href}
      className={clsx(
        'nav-item',
        activeSegment === segment && 'active'
      )}
    >
      {children}
    </a>
  )
}
```

---

## 제한사항

### Server Components에서 사용 불가

```tsx
// ❌ 오류 - Server Component에서 사용
export default function Layout() {
  const segment = useSelectedLayoutSegment() // 오류!
  return <div>{segment}</div>
}

// ✅ 올바른 방법 - Client Component 사용
'use client'

export default function LayoutNav() {
  const segment = useSelectedLayoutSegment()
  return <div>{segment}</div>
}
```

### 한 단계만 읽음

```tsx
// app/dashboard/layout.js에서
const segment = useSelectedLayoutSegment()

// /dashboard/analytics/overview 방문 시
// segment는 'analytics'만 반환 ('overview'는 포함 안됨)

// 모든 세그먼트를 원하면 useSelectedLayoutSegments 사용
const segments = useSelectedLayoutSegments()
// ['analytics', 'overview'] 반환
```

---

## 버전 히스토리

- **v13.0.0**: `useSelectedLayoutSegment` 도입

---

## 관련 문서

- [usePathname](./usePathname.md)
- [useParams](./useParams.md)
- [Linking and Navigating](../../getting-started/04-linking-and-navigating.md)
- [Layouts and Pages](../../getting-started/03-layouts-and-pages.md)
