# Lazy Loading (지연 로딩)

Next.js에서 컴포넌트와 라이브러리를 필요할 때만 로드하여 성능을 최적화하는 방법을 알아봅니다.

## 개요

Next.js의 지연 로딩(Lazy Loading)은 **클라이언트 컴포넌트**와 가져온 라이브러리의 로딩을 필요할 때까지 연기하여 초기 애플리케이션 성능을 향상시킵니다. 이는 라우트를 렌더링하는 데 필요한 JavaScript 번들 크기를 줄입니다.

---

## 두 가지 지연 로딩 방식

### 1. **`next/dynamic`을 사용한 동적 가져오기**
- `React.lazy()`와 `Suspense`의 조합
- `app`과 `pages` 디렉토리에서 일관되게 작동
- 점진적 마이그레이션 지원

### 2. **`React.lazy()`와 Suspense**
- 네이티브 React 지연 로딩 메커니즘
- `next/dynamic`의 기반으로 사용됨

> **권장:** 대부분의 경우 `next/dynamic`을 사용하는 것이 좋습니다.

---

## 클라이언트 컴포넌트 가져오기

### 기본 사용법

```jsx
// app/page.js
'use client'

import { useState } from 'react'
import dynamic from 'next/dynamic'

// 동적으로 컴포넌트 가져오기
const ComponentA = dynamic(() => import('../components/A'))
const ComponentB = dynamic(() => import('../components/B'))
const ComponentC = dynamic(() => import('../components/C'), { ssr: false })

export default function ClientComponentExample() {
  const [showMore, setShowMore] = useState(false)

  return (
    <div>
      {/* 즉시 로드됨 (별도 번들) */}
      <ComponentA />

      {/* 조건부로 필요할 때만 로드됨 */}
      {showMore && <ComponentB />}
      <button onClick={() => setShowMore(!showMore)}>
        더 보기
      </button>

      {/* 클라이언트에서만 로드됨 (SSR 건너뛰기) */}
      <ComponentC />
    </div>
  )
}
```

### 작동 방식

1. **ComponentA**: 페이지가 로드될 때 즉시 로드되지만, 별도의 JavaScript 번들로 분리됩니다.
2. **ComponentB**: `showMore`가 `true`가 될 때만 로드됩니다 (온디맨드).
3. **ComponentC**: 클라이언트 사이드에서만 로드됩니다 (서버에서 렌더링되지 않음).

---

## SSR (서버 사이드 렌더링) 건너뛰기

브라우저에서만 작동하는 컴포넌트의 경우 `ssr: false` 옵션을 사용하세요:

```jsx
'use client'

import dynamic from 'next/dynamic'

const NoSSR = dynamic(() => import('../components/NoSSR'), {
  ssr: false,
})

export default function Page() {
  return (
    <div>
      <NoSSR />
    </div>
  )
}
```

### 사용 사례

```jsx
'use client'

import dynamic from 'next/dynamic'

// window 객체를 사용하는 컴포넌트
const BrowserOnlyComponent = dynamic(
  () => import('../components/BrowserOnly'),
  { ssr: false }
)

// 차트 라이브러리 등
const ChartComponent = dynamic(
  () => import('../components/Chart'),
  {
    ssr: false,
    loading: () => <div>차트 로딩 중...</div>,
  }
)
```

> **중요:**
> - `ssr: false`는 **클라이언트 컴포넌트**에서만 작동합니다.
> - 적절한 코드 분할을 위해 클라이언트 컴포넌트로 이동하세요.

---

## 서버 컴포넌트 가져오기

서버 컴포넌트도 동적으로 가져올 수 있습니다:

```jsx
// app/page.js
import dynamic from 'next/dynamic'

// 서버 컴포넌트 동적 가져오기
const ServerComponent = dynamic(() => import('../components/ServerComponent'))

export default function ServerComponentExample() {
  return (
    <div>
      <ServerComponent />
    </div>
  )
}
```

> **중요:**
> - 서버 컴포넌트에서는 `ssr: false`를 **지원하지 않습니다**.
> - 서버 컴포넌트는 자동으로 코드 분할됩니다.

---

## 외부 라이브러리 가져오기

필요할 때만 외부 라이브러리를 로드하여 초기 번들 크기를 줄일 수 있습니다:

### 예시: Fuse.js 검색 라이브러리

```jsx
// app/page.js
'use client'

import { useState } from 'react'

const names = ['Tim', 'Joe', 'Bel', 'Lee', 'Max', 'Anna']

export default function Page() {
  const [results, setResults] = useState([])

  return (
    <div>
      <input
        type="text"
        placeholder="이름 검색..."
        onChange={async (e) => {
          const { value } = e.currentTarget

          // Fuse.js를 사용할 때만 동적으로 가져오기
          const Fuse = (await import('fuse.js')).default
          const fuse = new Fuse(names)

          setResults(fuse.search(value))
        }}
      />
      <pre>결과: {JSON.stringify(results, null, 2)}</pre>
    </div>
  )
}
```

### 예시: 날짜 포맷 라이브러리

```jsx
'use client'

import { useState } from 'react'

export default function DatePicker() {
  const [date, setDate] = useState(null)

  const handleDateSelect = async (selectedDate) => {
    // date-fns를 사용할 때만 가져오기
    const { format } = await import('date-fns')
    setDate(format(selectedDate, 'yyyy-MM-dd'))
  }

  return <div>{date}</div>
}
```

---

## 커스텀 로딩 컴포넌트

로딩 중에 표시할 커스텀 UI를 제공할 수 있습니다:

### 기본 로딩 표시

```jsx
'use client'

import dynamic from 'next/dynamic'

const WithCustomLoading = dynamic(
  () => import('../components/WithCustomLoading'),
  {
    loading: () => <p>로딩 중...</p>,
  }
)

export default function Page() {
  return (
    <div>
      <WithCustomLoading />
    </div>
  )
}
```

### 스켈레톤 UI

```jsx
'use client'

import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(
  () => import('../components/HeavyComponent'),
  {
    loading: () => (
      <div className="animate-pulse">
        <div className="h-4 bg-gray-200 rounded w-3/4 mb-4"></div>
        <div className="h-4 bg-gray-200 rounded w-1/2"></div>
      </div>
    ),
  }
)

export default function Page() {
  return <HeavyComponent />
}
```

### 스피너 컴포넌트

```jsx
'use client'

import dynamic from 'next/dynamic'

function LoadingSpinner() {
  return (
    <div className="flex justify-center items-center">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-gray-900"></div>
    </div>
  )
}

const AsyncComponent = dynamic(
  () => import('../components/AsyncComponent'),
  {
    loading: LoadingSpinner,
  }
)
```

---

## Named Export 가져오기

Named export된 컴포넌트를 동적으로 가져오는 방법:

**components/hello.js**
```jsx
'use client'

export function Hello() {
  return <p>안녕하세요!</p>
}

export function Goodbye() {
  return <p>안녕히 가세요!</p>
}
```

**app/page.js**
```jsx
import dynamic from 'next/dynamic'

// Named export 가져오기
const Hello = dynamic(() =>
  import('../components/hello').then((mod) => mod.Hello)
)

const Goodbye = dynamic(() =>
  import('../components/hello').then((mod) => mod.Goodbye)
)

export default function Page() {
  return (
    <div>
      <Hello />
      <Goodbye />
    </div>
  )
}
```

---

## React.lazy()와 Suspense 사용

Next.js 외부에서 표준 React 방식을 사용할 수도 있습니다:

```jsx
'use client'

import { lazy, Suspense } from 'react'

const LazyComponent = lazy(() => import('../components/LazyComponent'))

export default function Page() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <LazyComponent />
    </Suspense>
  )
}
```

> **참고:** `next/dynamic`이 더 많은 기능을 제공하므로 권장됩니다.

---

## 고급 사용 패턴

### 모달 지연 로딩

```jsx
'use client'

import { useState } from 'react'
import dynamic from 'next/dynamic'

const Modal = dynamic(() => import('../components/Modal'), {
  ssr: false,
})

export default function Page() {
  const [showModal, setShowModal] = useState(false)

  return (
    <div>
      <button onClick={() => setShowModal(true)}>
        모달 열기
      </button>

      {/* 모달이 필요할 때만 로드 */}
      {showModal && (
        <Modal onClose={() => setShowModal(false)} />
      )}
    </div>
  )
}
```

### 탭 컴포넌트 지연 로딩

```jsx
'use client'

import { useState } from 'react'
import dynamic from 'next/dynamic'

const TabA = dynamic(() => import('../components/TabA'))
const TabB = dynamic(() => import('../components/TabB'))
const TabC = dynamic(() => import('../components/TabC'))

export default function Tabs() {
  const [activeTab, setActiveTab] = useState('a')

  return (
    <div>
      <nav>
        <button onClick={() => setActiveTab('a')}>Tab A</button>
        <button onClick={() => setActiveTab('b')}>Tab B</button>
        <button onClick={() => setActiveTab('c')}>Tab C</button>
      </nav>

      {activeTab === 'a' && <TabA />}
      {activeTab === 'b' && <TabB />}
      {activeTab === 'c' && <TabC />}
    </div>
  )
}
```

### 조건부 렌더링

```jsx
'use client'

import dynamic from 'next/dynamic'

const AdminPanel = dynamic(() => import('../components/AdminPanel'))
const UserDashboard = dynamic(() => import('../components/UserDashboard'))

export default function Dashboard({ isAdmin }) {
  return (
    <div>
      {isAdmin ? <AdminPanel /> : <UserDashboard />}
    </div>
  )
}
```

---

## 주요 포인트

### ✅ 해야 할 것

1. **클라이언트 컴포넌트에 적용**: 서버 컴포넌트는 자동으로 코드 분할됩니다.
2. **조건부 렌더링 최적화**: 사용자 액션에 따라 로드되는 컴포넌트에 적합합니다.
3. **무거운 라이브러리 지연 로딩**: 초기 번들 크기를 줄입니다.
4. **로딩 상태 제공**: 더 나은 UX를 위해 커스텀 로딩 컴포넌트를 사용하세요.

### ❌ 피해야 할 것

1. **모든 컴포넌트에 적용**: 즉시 필요한 컴포넌트는 정적으로 가져오세요.
2. **작은 컴포넌트 분할**: 오버헤드가 이점보다 클 수 있습니다.
3. **서버 컴포넌트에 `ssr: false` 사용**: 에러가 발생합니다.

---

## 성능 고려사항

### 번들 크기 분석

```bash
# 번들 분석기 설치
npm install @next/bundle-analyzer

# next.config.js에 추가
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})

module.exports = withBundleAnalyzer({})

# 분석 실행
ANALYZE=true npm run build
```

### 최적화 팁

1. **라우트 기반 분할**: 각 라우트가 독립적인 번들을 가집니다.
2. **컴포넌트 기반 분할**: `next/dynamic`으로 큰 컴포넌트를 분할합니다.
3. **라이브러리 분할**: 큰 외부 라이브러리를 필요할 때만 로드합니다.

---

## 다음 단계

- [Optimizing](./optimizing.md) - 성능 최적화 가이드
- [Server and Client Components](../getting-started/05-server-and-client-components.md) - 컴포넌트 이해
- [Code Splitting](https://react.dev/reference/react/lazy) - React 공식 문서

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
