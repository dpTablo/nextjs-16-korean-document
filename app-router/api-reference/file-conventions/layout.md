---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/layout
버전: 16.1.6
---

# layout.js

`layout` 파일은 Next.js 애플리케이션에서 레이아웃을 정의합니다. 이는 라우트 세그먼트를 감싸는 공유 UI를 생성하는 데 사용되는 특수 파일입니다.

## 기본 구조

### 일반 레이아웃
```tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return <section>{children}</section>
}
```

### 루트 레이아웃
루트 레이아웃은 `app` 디렉토리의 최상위 레이아웃이며 **반드시** `<html>` 및 `<body>` 태그를 정의해야 합니다:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

## Props 참조

### `children` (필수)
- 레이아웃이 감싸는 라우트 세그먼트로 채워집니다
- 하위 레이아웃, 페이지 또는 특수 파일(Loading, Error)이 될 수 있습니다

### `params` (선택사항)
동적 라우트 매개변수를 포함하는 객체로 해석되는 **프로미스**입니다:

```tsx
export default async function Layout({
  children,
  params,
}: {
  children: React.ReactNode
  params: Promise<{ team: string }>
}) {
  const { team } = await params
}
```

**참고**: `params`는 v15.0.0+에서 프로미스입니다. 값에 접근하려면 `async/await` 또는 React의 `use()` 함수를 사용하세요.

| 라우트 예제 | URL | params |
|---|---|---|
| `app/dashboard/[team]/layout.js` | `/dashboard/1` | `Promise<{ team: '1' }>` |
| `app/shop/[tag]/[item]/layout.js` | `/shop/1/2` | `Promise<{ tag: '1', item: '2' }>` |
| `app/blog/[...slug]/layout.js` | `/blog/1/2` | `Promise<{ slug: ['1', '2'] }>` |

## LayoutProps 헬퍼

강력하게 타입이 지정된 props를 위해 `LayoutProps`로 레이아웃을 타이핑하세요:

```tsx
export default function Layout(props: LayoutProps<'/dashboard'>) {
  return (
    <section>
      {props.children}
      {/* {props.analytics} - 존재하는 경우 타입이 지정된 슬롯 */}
    </section>
  )
}
```

타입은 `next dev`, `next build` 또는 `next typegen` 중에 생성됩니다.

## 주요 제약 사항 및 주의사항

### Request 객체
레이아웃은 성능을 유지하기 위해 원시 request 객체에 접근할 수 없습니다. 대신 다음을 사용하세요:
- `headers()` API
- `cookies()` API

```tsx
import { cookies } from 'next/headers'

export default async function Layout({ children }) {
  const cookieStore = await cookies()
  const theme = cookieStore.get('theme')
  return '...'
}
```

### 쿼리 매개변수
레이아웃은 네비게이션 시 다시 렌더링되지 않으므로 검색 매개변수에 접근할 수 없습니다. 해결 방법:
- Page의 `searchParams` prop 사용
- 클라이언트 컴포넌트에서 `useSearchParams()` 훅 사용

```tsx
'use client'
import { useSearchParams } from 'next/navigation'

export default function Search() {
  const searchParams = useSearchParams()
  const search = searchParams.get('search')
  return '...'
}
```

### 경로명 (Pathname)
레이아웃은 네비게이션 시 다시 렌더링되지 않습니다. 클라이언트 컴포넌트에서 `usePathname()` 훅을 사용하세요:

```tsx
'use client'
import { usePathname } from 'next/navigation'

export default function Breadcrumbs() {
  const pathname = usePathname()
  const segments = pathname.split('/')
  return '...'
}
```

### 데이터 페칭
- 레이아웃은 하위 요소에 데이터를 전달할 수 없습니다
- 여러 위치에서 동일한 데이터를 가져오세요. Next.js `fetch()`는 자동으로 중복 제거합니다
- React `cache()`를 사용하여 요청을 중복 제거하세요

### 하위 세그먼트 접근
클라이언트 컴포넌트에서 사용:
- `useSelectedLayoutSegment()`
- `useSelectedLayoutSegments()`

## 루트 레이아웃 요구사항

- **반드시** `<html>` 및 `<body>` 태그를 포함해야 합니다
- **수동으로** `<head>` 태그(`<title>`, `<meta>`)를 추가하면 안 됩니다 - 대신 Metadata API를 사용하세요
- 다음을 사용하여 **여러 루트 레이아웃**을 생성할 수 있습니다:
  - 라우트 그룹: `app/(shop)/layout.js`, `app/(marketing)/layout.js`
  - `app/layout.js`를 생략하여 하위 디렉토리가 루트 레이아웃이 되도록 함
- 여러 루트 레이아웃 간 네비게이션은 **전체 페이지 로드**를 유발합니다
- 동적 세그먼트 아래에 위치할 수 있습니다 (예: 국제화: `app/[lang]/layout.js`)

## 일반적인 예제

### 메타데이터
```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Next.js',
}

export default function Layout({ children }: { children: React.ReactNode }) {
  return '...'
}
```

### 활성 네비게이션 링크
```tsx
'use client'
import { usePathname } from 'next/navigation'

export function NavLinks() {
  const pathname = usePathname()
  return (
    <nav>
      <Link className={`link ${pathname === '/' ? 'active' : ''}`} href="/">
        Home
      </Link>
    </nav>
  )
}
```

### `params` 기반 콘텐츠 표시
```tsx
export default async function DashboardLayout({
  children,
  params,
}: {
  children: React.ReactNode
  params: Promise<{ team: string }>
}) {
  const { team } = await params
  return (
    <section>
      <header><h1>Welcome to {team}'s Dashboard</h1></header>
      <main>{children}</main>
    </section>
  )
}
```

### 클라이언트 컴포넌트에서 `params` 사용
```tsx
'use client'
import { use } from 'react'

export default function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = use(params)
}
```

## 버전 히스토리

| 버전 | 변경사항 |
|---|---|
| v15.0.0-RC | `params`가 이제 프로미스입니다 (codemod 사용 가능) |
| v13.0.0 | `layout` 도입 |

## 관련 문서

- [레이아웃과 페이지](../../getting-started/03-layouts-and-pages.md)
- [generateMetadata](../functions/generateMetadata.md)
- [usePathname](../functions/usePathname.md)
- [useSelectedLayoutSegment](../functions/useSelectedLayoutSegment.md)
