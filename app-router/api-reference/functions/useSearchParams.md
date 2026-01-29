---
원문: https://nextjs.org/docs/app/api-reference/functions/use-search-params
버전: 16.1.6
---

# useSearchParams

`useSearchParams`는 **Client Component** hook으로, 현재 URL의 **쿼리 문자열**을 읽을 수 있게 해줍니다. [`URLSearchParams`](https://developer.mozilla.org/docs/Web/API/URLSearchParams) 인터페이스의 **읽기 전용** 버전을 반환합니다.

---

## 기본 사용법

```tsx
'use client'

import { useSearchParams } from 'next/navigation'

export default function SearchBar() {
  const searchParams = useSearchParams()
  const search = searchParams.get('search')

  // URL -> `/dashboard?search=my-project`
  // `search` -> 'my-project'
  return <>Search: {search}</>
}
```

---

## 함수 시그니처

```typescript
useSearchParams(): ReadonlyURLSearchParams
```

### 매개변수

없음

### 반환값

**읽기 전용** `URLSearchParams` 인터페이스를 반환합니다.

---

## 사용 가능한 메서드

### get(name)

쿼리 파라미터의 첫 번째 값을 반환합니다.

```tsx
const searchParams = useSearchParams()
const value = searchParams.get('search')
```

**예제:**

| URL | `searchParams.get('a')` |
|-----|------------------------|
| `/dashboard?a=1` | `'1'` |
| `/dashboard?a=` | `''` |
| `/dashboard?b=3` | `null` |
| `/dashboard?a=1&a=2` | `'1'` (모든 값은 `getAll()` 사용) |

### has(name)

파라미터 존재 여부를 boolean으로 반환합니다.

```tsx
const searchParams = useSearchParams()
const hasSearch = searchParams.has('search')
```

**예제:**

| URL | `searchParams.has('a')` |
|-----|------------------------|
| `/dashboard?a=1` | `true` |
| `/dashboard?b=3` | `false` |

### getAll(name)

특정 파라미터의 모든 값을 배열로 반환합니다.

```tsx
const searchParams = useSearchParams()
const all = searchParams.getAll('a')
```

**예제:**

| URL | `searchParams.getAll('a')` |
|-----|---------------------------|
| `/dashboard?a=1&a=2` | `['1', '2']` |
| `/dashboard?a=1` | `['1']` |
| `/dashboard?b=3` | `[]` |

### 기타 읽기 전용 메서드

- `keys()`: 모든 파라미터 이름의 iterator
- `values()`: 모든 파라미터 값의 iterator
- `entries()`: 모든 [name, value] 쌍의 iterator
- `forEach(callback)`: 각 파라미터에 함수 실행
- `toString()`: 쿼리 문자열 반환

---

## 동작 원리

### 정적 렌더링 (Static Rendering)

정적으로 렌더링된 라우트에서 `useSearchParams` 호출 시, 가장 가까운 `Suspense` 경계까지의 클라이언트 컴포넌트 트리가 클라이언트 측에서 렌더링됩니다.

**권장사항**: `Suspense` 경계로 감싸기

```tsx
// app/page.tsx
import { Suspense } from 'react'
import SearchBar from './search-bar'

function SearchBarFallback() {
  return <>placeholder</>
}

export default function Page() {
  return (
    <>
      <nav>
        <Suspense fallback={<SearchBarFallback />}>
          <SearchBar />
        </Suspense>
      </nav>
      <h1>Dashboard</h1>
    </>
  )
}
```

### 동적 렌더링 (Dynamic Rendering)

동적으로 렌더링된 라우트에서는 서버 초기 렌더링 중에 `useSearchParams`를 사용할 수 있습니다.

```tsx
// app/page.tsx
import { connection } from 'next/server'
import SearchBar from './search-bar'

export default async function Page() {
  await connection()
  return (
    <>
      <nav>
        <SearchBar />
      </nav>
      <h1>Dashboard</h1>
    </>
  )
}
```

---

## 사용 예제

### 검색 기능 구현

```tsx
'use client'

import { useSearchParams } from 'next/navigation'

export default function SearchResults() {
  const searchParams = useSearchParams()
  const query = searchParams.get('q')
  const filter = searchParams.get('filter')

  return (
    <div>
      <h1>Search Results</h1>
      {query && <p>Query: {query}</p>}
      {filter && <p>Filter: {filter}</p>}
    </div>
  )
}
```

### searchParams 업데이트

```tsx
'use client'

import { useRouter, usePathname, useSearchParams } from 'next/navigation'
import { useCallback } from 'react'

export default function SortButtons() {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()

  const createQueryString = useCallback(
    (name: string, value: string) => {
      const params = new URLSearchParams(searchParams.toString())
      params.set(name, value)
      return params.toString()
    },
    [searchParams]
  )

  return (
    <>
      <button
        onClick={() => {
          router.push(pathname + '?' + createQueryString('sort', 'asc'))
        }}
      >
        Sort ASC
      </button>
      <button
        onClick={() => {
          router.push(pathname + '?' + createQueryString('sort', 'desc'))
        }}
      >
        Sort DESC
      </button>
    </>
  )
}
```

### Link와 함께 사용

```tsx
'use client'

import Link from 'next/link'
import { usePathname, useSearchParams } from 'next/navigation'

export default function FilterLinks() {
  const pathname = usePathname()
  const searchParams = useSearchParams()

  const createQueryString = (name: string, value: string) => {
    const params = new URLSearchParams(searchParams.toString())
    params.set(name, value)
    return params.toString()
  }

  return (
    <nav>
      <Link href={pathname + '?' + createQueryString('filter', 'all')}>
        All
      </Link>
      <Link href={pathname + '?' + createQueryString('filter', 'active')}>
        Active
      </Link>
      <Link href={pathname + '?' + createQueryString('filter', 'completed')}>
        Completed
      </Link>
    </nav>
  )
}
```

### 여러 파라미터 처리

```tsx
'use client'

import { useSearchParams } from 'next/navigation'

export default function MultipleParams() {
  const searchParams = useSearchParams()

  const tags = searchParams.getAll('tag')
  const sort = searchParams.get('sort') || 'date'
  const page = parseInt(searchParams.get('page') || '1')

  return (
    <div>
      <p>Tags: {tags.join(', ')}</p>
      <p>Sort: {sort}</p>
      <p>Page: {page}</p>
    </div>
  )
}
```

---

## 서버 컴포넌트에서

Server Components에서는 `searchParams` prop을 사용하세요.

```tsx
// app/page.tsx
type Props = {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}

export default async function Page({ searchParams }: Props) {
  const params = await searchParams
  const query = params.search

  return <div>Search: {query}</div>
}
```

**주의**: Layouts는 `searchParams` prop을 받지 않습니다 (네비게이션 중 재렌더링 방지).

---

## 중요한 주의사항

> **Good to know**:
> * `useSearchParams`는 **Client Component** hook이며 Server Components에서는 사용할 수 없습니다
> * 정적 페이지에서 사용 시 `Suspense` 경계가 필수입니다 (없으면 프로덕션 빌드 실패)
> * 개발 환경에서는 `Suspense` 없이도 작동할 수 있지만, 프로덕션에서는 필수입니다
> * Pages Router(`/pages` 디렉토리)와 함께 사용 시 `ReadonlyURLSearchParams | null` 반환

> **Suspense 경계**:
> * 정적으로 렌더링된 페이지에서 `useSearchParams`를 사용하면 가장 가까운 Suspense 경계까지 클라이언트 측 렌더링으로 전환됩니다
> * 성능 최적화를 위해 `useSearchParams`를 사용하는 컴포넌트를 Suspense로 감싸는 것이 좋습니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.0.0 | `useSearchParams` 도입 |

---

## 관련 문서

- [useRouter](./useRouter.md)
- [usePathname](./usePathname.md)
- [useParams](./useParams.md)
- [Linking and Navigating](../../getting-started/04-linking-and-navigating.md)
