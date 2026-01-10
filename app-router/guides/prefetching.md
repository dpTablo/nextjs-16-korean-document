# Prefetching (프리페칭)

Next.js에서 라우트 간 탐색을 즉각적으로 느껴지게 만드는 프리페칭 전략을 알아봅니다.

## 개요

프리페칭(Prefetching)은 네비게이션 전에 리소스를 미리 가져와서 라우트 간 탐색을 즉각적으로 느껴지게 만듭니다. Next.js는 애플리케이션 코드의 링크를 기반으로 기본적으로 지능적으로 프리페치합니다.

---

## 프리페칭 작동 방식

Next.js는 자동으로 애플리케이션을 라우트별로 더 작은 JavaScript 청크로 분할합니다. 모든 코드를 미리 로드하는 대신:

- 현재 라우트에 필요한 코드만 로드됩니다
- 다른 부분은 백그라운드에서 로드됩니다
- 다음 라우트의 리소스는 링크를 클릭하기 전에 캐시됩니다
- 클라이언트 사이드 전환으로 전체 페이지 새로고침을 제거합니다

---

## 정적 vs 동적 라우트

프리페칭 동작은 라우트 유형에 따라 다릅니다:

| 항목 | 정적 페이지 | 동적 페이지 |
|------|-------------|-------------|
| **프리페치 여부** | 예, 전체 라우트 | 아니요, `loading.js`가 있을 때만 |
| **클라이언트 캐시 TTL** | 5분 (기본값) | 꺼짐, 활성화하지 않는 한 |
| **클릭 시 서버 왕복** | 아니요 | 예, 셸 후 스트리밍 |

### 정적 페이지
- **전체 라우트가 프리페치됩니다**
- 클라이언트 캐시에 5분간 저장됩니다
- 클릭 시 즉시 렌더링됩니다

### 동적 페이지
- **기본적으로 프리페치되지 않습니다**
- `loading.js`가 있으면 로딩 상태가 프리페치됩니다
- 클릭 시 서버에 요청하여 데이터를 가져옵니다

---

## 프리페칭 방법

### 1. 자동 프리페치 (기본값)

`Link` 컴포넌트는 기본적으로 프리페칭을 활성화합니다:

```tsx
import Link from 'next/link'

export default function NavLink() {
  return <Link href="/about">About</Link>
}
```

**특징:**
- 프로덕션 환경에서만 실행됩니다
- `prefetch={false}`로 비활성화할 수 있습니다
- 캐시 TTL: `loading.js`가 있으면 30초, 없으면 앱 리로드까지

**예시 - 프리페치 비활성화:**
```tsx
<Link href="/dashboard" prefetch={false}>
  Dashboard
</Link>
```

### 2. 수동 프리페치

`router.prefetch()`를 사용하여 수동으로 프리페치:

```tsx
'use client'

import { useRouter } from 'next/navigation'

export function PricingCard() {
  const router = useRouter()

  return (
    <div
      onMouseEnter={() => router.prefetch('/pricing')}
      className="card"
    >
      <h2>프리미엄 플랜</h2>
      <button onClick={() => router.push('/pricing')}>
        자세히 보기
      </button>
    </div>
  )
}
```

**사용 사례:**
- 뷰포트 밖의 라우트
- 분석 데이터 기반 프리페치
- 호버/스크롤 이벤트 기반

### 3. 호버 기반 프리페치

마우스 호버 시에만 프리페치:

```tsx
'use client'

import Link from 'next/link'
import { useState } from 'react'

export function HoverPrefetchLink({
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

**작동 방식:**
- 초기 상태: `prefetch={false}` (프리페치 꺼짐)
- 호버 시: `prefetch={null}` (기본 프리페치 동작 복원)
- 사용자 의도를 보일 때만 리소스 사용

**사용 예시:**
```tsx
<HoverPrefetchLink href="/products/123">
  제품 상세보기
</HoverPrefetchLink>
```

### 4. 커스텀 전략으로 Link 확장

지속적으로 재검증하는 커스텀 프리페치 전략:

```tsx
'use client'

import { useRouter } from 'next/navigation'
import { useEffect } from 'react'

function ManualPrefetchLink({
  href,
  children,
}: {
  href: string
  children: React.ReactNode
}) {
  const router = useRouter()

  useEffect(() => {
    let cancelled = false

    const poll = () => {
      if (!cancelled) {
        router.prefetch(href, {
          onInvalidate: poll, // 무효화 시 재프리페치
        })
      }
    }

    poll()

    return () => {
      cancelled = true
    }
  }, [href, router])

  return (
    <a
      href={href}
      onClick={(event) => {
        event.preventDefault()
        router.push(href)
      }}
    >
      {children}
    </a>
  )
}
```

**특징:**
- 데이터 무효화 시 자동 재프리페치
- 실시간 데이터가 필요한 라우트에 적합

### 5. 프리페치 비활성화

프리페치를 완전히 비활성화한 Link:

```tsx
'use client'

import Link from 'next/link'

function NoPrefetchLink({
  prefetch,
  ...rest
}: React.ComponentProps<typeof Link>) {
  return <Link {...rest} prefetch={false} />
}

export default function Page() {
  return (
    <NoPrefetchLink href="/heavy-page">
      무거운 페이지 (프리페치 안 함)
    </NoPrefetchLink>
  )
}
```

---

## 프리페칭 최적화

### 클라이언트 캐시

Next.js는 프리페치된 React Server Component 페이로드를 라우트 세그먼트별로 메모리에 저장합니다.

**캐시 재사용:**
```
/dashboard/settings → /dashboard/analytics
```
- `/dashboard` 레이아웃은 재사용됩니다
- `/analytics` 리프 페이지만 새로 가져옵니다

**이점:**
- 형제 라우트 간 빠른 네비게이션
- 공유 레이아웃 재렌더링 방지
- 메모리 효율적인 캐싱

### 프리페치 스케줄링

Next.js는 다음 우선순위로 프리페치를 스케줄링합니다:

1. **뷰포트 내 링크** - 화면에 보이는 링크 우선
2. **사용자 의도가 있는 링크** - 호버/터치된 링크
3. **최신 링크 우선** - 새로운 링크가 오래된 것을 대체
4. **스크롤 아웃된 링크 제거** - 화면 밖으로 나간 링크는 폐기

### Partial Prerendering (PPR)

부분 사전 렌더링을 사용하는 경우:

- **정적 셸** - 즉시 프리페치 및 스트리밍
- **동적 데이터** - 준비되면 스트리밍
- **데이터 무효화** - 연결된 프리페치를 자동으로 새로고침

---

## 실전 예시

### 제품 목록 페이지

```tsx
'use client'

import Link from 'next/link'
import { useState } from 'react'

export function ProductCard({ product }) {
  const [shouldPrefetch, setShouldPrefetch] = useState(false)

  return (
    <div
      onMouseEnter={() => setShouldPrefetch(true)}
      className="product-card"
    >
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>{product.price}</p>
      <Link
        href={`/products/${product.id}`}
        prefetch={shouldPrefetch ? null : false}
      >
        상세보기
      </Link>
    </div>
  )
}
```

### 조건부 프리페치

```tsx
'use client'

import { useRouter } from 'next/navigation'
import { useEffect } from 'react'

export function ConditionalPrefetch({
  href,
  shouldPrefetch,
}: {
  href: string
  shouldPrefetch: boolean
}) {
  const router = useRouter()

  useEffect(() => {
    if (shouldPrefetch) {
      router.prefetch(href)
    }
  }, [href, shouldPrefetch, router])

  return null
}

// 사용 예시
export default function Page() {
  const isPremiumUser = true // 사용자 상태

  return (
    <>
      {isPremiumUser && (
        <ConditionalPrefetch
          href="/premium-dashboard"
          shouldPrefetch={true}
        />
      )}
      {/* 페이지 콘텐츠 */}
    </>
  )
}
```

### 분석 기반 프리페치

```tsx
'use client'

import { useRouter } from 'next/navigation'
import { useEffect } from 'react'

export function AnalyticsPrefetch() {
  const router = useRouter()

  useEffect(() => {
    // 분석 데이터에서 가장 많이 방문하는 페이지 가져오기
    const popularPages = ['/products', '/pricing', '/about']

    popularPages.forEach((page) => {
      router.prefetch(page)
    })
  }, [router])

  return null
}
```

---

## 문제 해결

### 의도하지 않은 부작용 방지

**문제:** 레이아웃/페이지의 부작용이 실제 네비게이션이 아닌 프리페치 중에 트리거됩니다.

**해결책:** 부작용을 `useEffect` 훅이나 Server Actions로 이동하세요:

```tsx
'use client'

import { useEffect } from 'react'
import { trackPageView } from '@/lib/analytics'

export function AnalyticsTracker() {
  useEffect(() => {
    // 실제 페이지 마운트 시에만 실행
    trackPageView()
  }, [])

  return null
}
```

**나쁜 예시 (프리페치 시 실행됨):**
```tsx
// ❌ 프리페치할 때마다 실행됨
const AnalyticsPage = () => {
  trackPageView() // 부작용이 즉시 실행
  return <div>Page</div>
}
```

**좋은 예시 (마운트 시에만 실행):**
```tsx
// ✅ 실제 네비게이션 시에만 실행
const AnalyticsPage = () => {
  useEffect(() => {
    trackPageView() // useEffect 내에서 실행
  }, [])
  return <div>Page</div>
}
```

### 과도한 프리페치 방지

대량의 링크 목록이 있는 경우, 비용이 많이 드는 라우트에서 프리페치를 비활성화하세요:

```tsx
export function BlogPostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link
            prefetch={false} // 수백 개의 포스트 프리페치 방지
            href={`/blog/${post.id}`}
          >
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  )
}
```

**또는 호버까지 연기하여 리소스 사용 줄이기:**

```tsx
export function BlogPostList({ posts }) {
  const [activePosts, setActivePosts] = useState(new Set())

  return (
    <ul>
      {posts.map((post) => (
        <li
          key={post.id}
          onMouseEnter={() => {
            setActivePosts(new Set(activePosts).add(post.id))
          }}
        >
          <Link
            href={`/blog/${post.id}`}
            prefetch={activePosts.has(post.id) ? null : false}
          >
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  )
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **중요한 라우트 프리페치** - 사용자가 자주 방문하는 페이지
2. **호버 기반 프리페치 사용** - 리소스 사용 최적화
3. **정적 라우트 우선** - 동적 라우트보다 빠름
4. **분석 데이터 활용** - 인기 페이지 우선 프리페치

### ❌ 피해야 할 것

1. **모든 링크 프리페치** - 불필요한 네트워크 사용
2. **동적 라우트 과도한 프리페치** - 서버 부하 증가
3. **프리페치 중 부작용 실행** - useEffect 사용
4. **대량 목록 자동 프리페치** - 메모리 낭비

---

## 성능 모니터링

### 프리페치 효과 측정

```tsx
'use client'

import { useEffect } from 'react'

export function PrefetchMonitor() {
  useEffect(() => {
    if (typeof window !== 'undefined' && 'performance' in window) {
      const navigation = performance.getEntriesByType('navigation')[0]
      console.log('Navigation timing:', navigation)
    }
  }, [])

  return null
}
```

### 캐시 히트율 추적

```tsx
'use client'

import { useEffect } from 'react'

export function CacheTracker() {
  useEffect(() => {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.transferSize === 0) {
          console.log('Cache hit:', entry.name)
        }
      }
    })

    observer.observe({ type: 'resource', buffered: true })

    return () => observer.disconnect()
  }, [])

  return null
}
```

---

## 다음 단계

- [Lazy Loading](./lazy-loading.md) - 컴포넌트 지연 로딩
- [Linking and Navigating](../getting-started/04-linking-and-navigating.md) - 네비게이션 기본
- [useRouter](../api-reference/functions/useRouter.md) - Router API 레퍼런스

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
