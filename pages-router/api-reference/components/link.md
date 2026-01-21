# <Link>

`<Link>`는 HTML `<a>` 요소를 확장한 React 컴포넌트로, [프리페칭](/docs/app/getting-started/linking-and-navigating#prefetching)과 클라이언트 측 라우팅을 제공합니다. Next.js에서 라우트 간 네비게이션의 기본 방법입니다.

## 기본 사용법

```tsx
import Link from 'next/link'

export default function Home() {
  return <Link href="/dashboard">Dashboard</Link>
}
```

## Props 참조

| Prop | 예제 | 타입 | 필수 |
|------|------|------|------|
| [`href`](#href-필수) | `href="/dashboard"` | String 또는 Object | 예 |
| [`as`](#as) | `as="/post/abc"` | String 또는 Object | - |
| [`replace`](#replace) | `replace={false}` | Boolean | - |
| [`scroll`](#scroll) | `scroll={false}` | Boolean | - |
| [`prefetch`](#prefetch) | `prefetch={false}` | Boolean | - |
| [`shallow`](#shallow) | `shallow={false}` | Boolean | - |
| [`locale`](#locale) | `locale="fr"` | String 또는 Boolean | - |
| [`onNavigate`](#onnavigate) | `onNavigate={(e) => {}}` | Function | - |

> **참고**: `className`, `target="_blank"` 같은 `<a>` 태그 속성도 `<Link>`에 props로 추가할 수 있으며, 기본 `<a>` 요소로 전달됩니다.

---

## Props 상세

### `href` (필수)

네비게이션할 경로 또는 URL입니다.

```tsx
// 기본 문자열 사용
<Link href="/about">About</Link>

// URL 객체 사용
<Link
  href={{
    pathname: '/about',
    query: { name: 'test' },
  }}
>
  About
</Link>
```

### `replace`

**기본값: `false`**

`true`일 때, 브라우저 히스토리 스택에 새 URL을 추가하지 않고 현재 히스토리 상태를 대체합니다.

```tsx
<Link href="/dashboard" replace>
  Dashboard
</Link>
```

### `scroll`

**기본값: `true`**

`<Link>`의 기본 스크롤 동작은 스크롤 위치를 유지합니다 (브라우저의 뒤로/앞으로 네비게이션과 유사).

`scroll={false}`일 때, Next.js는 첫 번째 Page 요소로 스크롤하지 않습니다.

```tsx
<Link href="/dashboard" scroll={false}>
  Dashboard
</Link>
```

### `prefetch`

`<Link />` 컴포넌트가 뷰포트에 들어올 때 프리페칭이 발생합니다. Next.js는 링크된 라우트와 데이터를 백그라운드에서 로드하여 클라이언트 측 네비게이션의 성능을 향상시킵니다. **프리페칭은 프로덕션에서만 활성화됩니다.**

값:
- **`true` (기본값)**: 전체 라우트와 데이터가 프리페칭됩니다.
- `false`: 뷰포트 진입 시 프리페칭하지 않고, 호버 시에만 프리페칭합니다. 호버 시에도 완전히 비활성화하려면 `<a>` 태그 사용을 고려하세요.

```tsx
<Link href="/dashboard" prefetch={false}>
  Dashboard
</Link>
```

### `shallow`

현재 페이지 경로를 업데이트하되 `getStaticProps`, `getServerSideProps`, `getInitialProps`를 재실행하지 않습니다.

**기본값: `false`**

```tsx
<Link href="/dashboard" shallow={false}>
  Dashboard
</Link>
```

### `locale`

활성 로케일이 자동으로 앞에 붙습니다. `locale`로 다른 로케일을 지정할 수 있습니다. `false`일 때, `href`에 로케일을 직접 포함해야 합니다.

```tsx
import Link from 'next/link'

export default function Home() {
  return (
    <>
      {/* 기본 동작: 로케일이 앞에 붙음 */}
      <Link href="/dashboard">Dashboard (with locale)</Link>

      {/* 로케일 앞붙기 비활성화 */}
      <Link href="/dashboard" locale={false}>
        Dashboard (without locale)
      </Link>

      {/* 다른 로케일 지정 */}
      <Link href="/dashboard" locale="fr">
        Dashboard (French)
      </Link>
    </>
  )
}
```

### `as`

브라우저 URL 바에 표시될 경로의 선택적 데코레이터입니다. Next.js 9.5.3 이전에는 동적 라우트에 사용되었습니다.

### `onNavigate`

클라이언트 측 네비게이션 중에 호출되는 이벤트 핸들러입니다. 이벤트 객체에는 `preventDefault()` 메서드가 포함되어 필요하면 네비게이션을 취소할 수 있습니다.

```tsx
import Link from 'next/link'

export default function Page() {
  return (
    <Link
      href="/dashboard"
      onNavigate={(e) => {
        // SPA 네비게이션 중에만 실행
        console.log('Navigating...')

        // 선택적으로 네비게이션 방지
        // e.preventDefault()
      }}
    >
      Dashboard
    </Link>
  )
}
```

> **참고**: `onClick`과 `onNavigate`의 차이점
> - `onClick`: 모든 클릭 이벤트에서 실행됩니다.
> - `onNavigate`: 클라이언트 측 네비게이션 중에만 실행됩니다.
> - 수정자 키(`Ctrl`/`Cmd` + Click): `onClick`은 실행되지만 `onNavigate`는 실행되지 않습니다.
> - 외부 URL: `onNavigate`는 실행되지 않습니다 (동일 출처 네비게이션만 해당).
> - `download` 속성: `onClick`은 작동하지만 `onNavigate`는 작동하지 않습니다.

---

## 예제

### 동적 라우트 세그먼트로 링크하기

```tsx
import Link from 'next/link'

function Posts({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

### ID로 스크롤하기

```jsx
<Link href="/dashboard#settings">Settings</Link>

// 출력
<a href="/dashboard#settings">Settings</a>
```

### URL 객체 전달

```tsx
import Link from 'next/link'

function Home() {
  return (
    <ul>
      <li>
        <Link
          href={{
            pathname: '/about',
            query: { name: 'test' },
          }}
        >
          About us
        </Link>
      </li>
      <li>
        <Link
          href={{
            pathname: '/blog/[slug]',
            query: { slug: 'my-post' },
          }}
        >
          Blog Post
        </Link>
      </li>
    </ul>
  )
}

export default Home
```

결과:
- 정의된 라우트: `/about?name=test`
- 동적 라우트: `/blog/my-post`

### URL을 히스토리에 추가하지 않고 대체하기

```tsx
import Link from 'next/link'

export default function Home() {
  return (
    <Link href="/about" replace>
      About us
    </Link>
  )
}
```

### 페이지 상단으로 스크롤하지 않기

```jsx
import Link from 'next/link'

export default function Home() {
  return (
    <Link href="/#hashid" scroll={false}>
      Disables scrolling to the top
    </Link>
  )
}
```

### 미들웨어에서 프리페칭

인증 또는 다른 목적으로 미들웨어를 사용하여 사용자를 다른 페이지로 재작성할 때, `<Link />`가 재작성된 링크를 올바르게 프리페칭하려면 표시할 URL과 프리페칭할 URL을 모두 지정해야 합니다. 이렇게 하면 인증된 경로를 노출하지 않으면서 올바른 라우트를 프리페칭할 수 있습니다.

```ts filename="middleware.ts"
import { NextResponse } from 'next/server'

export function middleware(request: Request) {
  const nextUrl = request.nextUrl
  if (nextUrl.pathname === '/dashboard') {
    if (request.cookies.authToken) {
      return NextResponse.rewrite(new URL('/auth/dashboard', request.url))
    } else {
      return NextResponse.rewrite(new URL('/public/dashboard', request.url))
    }
  }
}
```

```tsx filename="pages/index.tsx"
'use client'

import Link from 'next/link'
import useIsAuthed from './hooks/useIsAuthed'

export default function Home() {
  const isAuthed = useIsAuthed()
  const path = isAuthed ? '/auth/dashboard' : '/public/dashboard'
  return (
    <Link as="/dashboard" href={path}>
      Dashboard
    </Link>
  )
}
```

---

## 버전 이력

| 버전 | 변경사항 |
|------|---------|
| `v15.4.0` | `auto`를 기본 `prefetch` 동작의 별칭으로 추가 |
| `v15.3.0` | `onNavigate` API 추가 |
| `v13.0.0` | 자식 `<a>` 태그가 더 이상 필요하지 않음. 자동 업데이트용 codemod 제공 |
| `v10.0.0` | 동적 라우트를 가리키는 `href` props 자동 해석, `as` prop이 더 이상 필요하지 않음 |
| `v8.0.0` | 프리페칭 성능 개선 |
| `v1.0.0` | `next/link` 도입 |
