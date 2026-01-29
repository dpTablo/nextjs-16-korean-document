---
원문: https://nextjs.org/docs/app/getting-started/layouts-and-pages
버전: 16.1.6
---

# 레이아웃과 페이지

## 개요

Next.js는 **파일 시스템 기반 라우팅**을 사용하여 폴더와 파일로 라우트를 정의할 수 있습니다. 이 가이드는 레이아웃과 페이지를 생성하고 이들 간에 링크하는 방법을 다룹니다.

---

## 페이지 생성하기

**페이지**는 특정 라우트에서 렌더링되는 UI입니다. `app` 디렉토리 내에 `page` 파일을 추가하고 React 컴포넌트를 export하여 페이지를 생성합니다.

**예시 - 인덱스 페이지 (`/`):**

```tsx
// app/page.tsx
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```

---

## 레이아웃 생성하기

**레이아웃**은 여러 페이지 간에 공유되는 UI입니다. 레이아웃은 상태를 보존하고 인터랙티브를 유지하며 네비게이션 시 재렌더링되지 않습니다.

`layout` 파일에서 React 컴포넌트를 export하여 레이아웃을 생성합니다. 컴포넌트는 `children` prop (페이지 또는 다른 레이아웃)을 받아야 합니다.

**예시 - 루트 레이아웃:**

```tsx
// app/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <main>{children}</main>
      </body>
    </html>
  )
}
```

**중요:** 루트 레이아웃은 **필수**이며 `html`과 `body` 태그를 포함해야 합니다.

---

## 중첩 라우트 생성하기

중첩 라우트는 여러 URL 세그먼트로 구성됩니다. 예를 들어, `/blog/[slug]`는 세 개의 세그먼트를 가집니다:
- `/` (루트 세그먼트)
- `blog` (세그먼트)
- `[slug]` (리프 세그먼트)

**폴더**는 라우트 세그먼트를 정의하고, **파일** (page, layout)은 세그먼트를 위한 UI를 생성합니다.

**예시 - 블로그 페이지:**

```tsx
// app/blog/page.tsx
import { getPosts } from '@/lib/posts'
import { Post } from '@/ui/post'

export default async function Page() {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <Post key={post.id} post={post} />
      ))}
    </ul>
  )
}
```

**예시 - 블로그 포스트 페이지:**

```tsx
// app/blog/[slug]/page.tsx
function generateStaticParams() {}

export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

---

## 레이아웃 중첩하기

폴더 계층 구조의 레이아웃은 자동으로 중첩되어 `children` prop을 통해 자식 레이아웃을 감쌉니다.

**예시 - 블로그 레이아웃:**

```tsx
// app/blog/layout.tsx
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return <section>{children}</section>
}
```

**중첩 계층:** 루트 레이아웃 → 블로그 레이아웃 → 블로그/포스트 페이지

---

## 동적 세그먼트 생성하기

**동적 세그먼트**는 데이터로부터 라우트를 생성합니다. 폴더 이름을 대괄호로 감쌉니다: `[segmentName]`.

**예시 - 동적 블로그 포스트 라우트:**

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPostPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await getPost(slug)

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  )
}
```

---

## 검색 매개변수로 렌더링하기

**서버 컴포넌트**는 `searchParams` prop을 사용하여 검색 매개변수에 접근할 수 있습니다:

```tsx
// app/page.tsx
export default async function Page({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const filters = (await searchParams).filters
}
```

**클라이언트 컴포넌트**는 `useSearchParams` 훅을 사용합니다.

### 사용 시기

- **`searchParams` prop:** 페이지를 위한 데이터 로드 (페이지네이션, 필터링)
- **`useSearchParams` 훅:** 클라이언트 사이드 필터링만
- **`URLSearchParams`:** 재렌더링 없이 콜백에서 매개변수 읽기

---

## 페이지 간 링크하기

네비게이션을 위해 `<Link>` 컴포넌트를 사용합니다. HTML `<a>` 태그를 프리페칭 및 클라이언트 사이드 네비게이션으로 확장합니다.

**예시 - 블로그 포스트 링크:**

```tsx
// app/ui/post.tsx
import Link from 'next/link'

export default async function Post({ post }) {
  const posts = await getPosts()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

**참고:** `<Link>`는 주요 네비게이션 방법입니다. 고급 네비게이션을 위해서는 [`useRouter`](/docs/app/api-reference/functions/use-router.md) 훅을 사용하세요.

---

## 라우트 Props 헬퍼

Next.js는 라우트로부터 `params`와 slots를 추론하기 위한 유틸리티 타입을 제공합니다:

- **PageProps:** 페이지 컴포넌트의 Props (`params`와 `searchParams` 포함)
- **LayoutProps:** 레이아웃 컴포넌트의 Props (`children`과 명명된 슬롯 포함)

이러한 헬퍼는 `next dev`, `next build`, 또는 `next typegen`에 의해 생성되어 전역적으로 사용 가능합니다.

**예시 - PageProps 사용:**

```tsx
// app/blog/[slug]/page.tsx
export default async function Page(props: PageProps<'/blog/[slug]'>) {
  const { slug } = await props.params
  return <h1>블로그 포스트: {slug}</h1>
}
```

**예시 - LayoutProps 사용:**

```tsx
// app/dashboard/layout.tsx
export default function Layout(props: LayoutProps<'/dashboard'>) {
  return (
    <section>
      {props.children}
      {/* {props.analytics} */}
    </section>
  )
}
```

**참고:** 정적 라우트는 `params`를 `{}`로 해석합니다. 헬퍼는 import가 필요 없습니다.

---

## API 참조

- [링크 및 네비게이션](/docs/app/getting-started/linking-and-navigating.md)
- [layout.js](/docs/app/api-reference/file-conventions/layout.md)
- [page.js](/docs/app/api-reference/file-conventions/page.md)
- [Link 컴포넌트](/docs/app/api-reference/components/link.md)
- [동적 세그먼트](/docs/app/api-reference/file-conventions/dynamic-routes.md)
