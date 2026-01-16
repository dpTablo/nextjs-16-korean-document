# Pages에서 App Router로 마이그레이션하는 방법

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-10-22

## 개요

이 가이드는 다음을 도와줍니다:
- Next.js 애플리케이션을 버전 12에서 13으로 업그레이드
- `pages`와 `app` 디렉토리 모두에서 작동하는 기능 업그레이드
- 기존 애플리케이션을 `pages`에서 `app`으로 점진적으로 마이그레이션

---

## 업그레이딩

### Node.js 버전

최소 Node.js 버전: **v18.17**

### Next.js 버전

Next.js 13으로 업데이트하려면:

```bash
npm install next@latest react@latest react-dom@latest
```

### ESLint 버전

```bash
npm install -D eslint-config-next@latest
```

> **참고**: VS Code에서 ESLint 서버를 재시작해야 할 수 있습니다. Command Palette (`cmd+shift+p` Mac; `ctrl+shift+p` Windows)를 열고 `ESLint: Restart ESLint Server`를 검색하세요.

---

## 새로운 기능 업그레이드

### `<Image/>` 컴포넌트

Next.js 12의 `next/future/image`는 이제 버전 13의 기본값입니다.

마이그레이션을 위한 두 가지 코드모드:
- **`next-image-to-legacy-image`**: `next/image`를 `next/legacy/image`로 안전하게 이름 변경
- **`next-image-experimental`**: 인라인 스타일 추가 및 미사용 props 제거

### `<Link>` 컴포넌트

더 이상 자식으로 `<a>` 태그를 수동으로 추가할 필요가 없습니다.

**Before (Next.js 12):**
```jsx
<Link href="/about">
  <a>About</a>
</Link>
```

**After (Next.js 13):**
```jsx
<Link href="/about">
  About
</Link>
```

업그레이드하려면 `new-link` 코드모드를 사용하세요.

### `<Script>` 컴포넌트

변경 사항:
- `_document.js`의 `beforeInteractive` 스크립트를 루트 레이아웃(`app/layout.tsx`)으로 이동
- 실험적 `worker` 전략은 `app`에서 작동하지 않음 (다른 전략으로 수정 필요)
- `onLoad`, `onReady`, `onError` 핸들러는 Server Components에서 작동하지 않으므로 Client Component로 이동 필요

### 폰트 최적화

새로운 [`next/font`](/app-router/api-reference/components/font.md) 모듈을 사용하세요 (`pages`와 `app` 모두에서 지원).

---

## `pages`에서 `app`으로 마이그레이션

`app` 디렉토리는 `pages` 디렉토리와 동시에 작동하도록 설계되어 페이지별 점진적 마이그레이션을 지원합니다.

### 주요 개념

| 기능 | 설명 |
|-----|------|
| **중첩 라우트 및 레이아웃** | `app` 디렉토리는 중첩된 경로와 레이아웃 지원 |
| **특수 파일** | `page.js`, `layout.js` 등의 특수 파일 규칙 |
| **코로케이션** | 컴포넌트, 스타일, 테스트 등을 `app`에 함께 배치 |
| **데이터 페칭** | `getServerSideProps`, `getStaticProps` → 새로운 API |
| **에러 처리** | `_error.js` → `error.js` |
| **찾을 수 없음** | `404.js` → `not-found.js` |
| **API 라우트** | `pages/api/*` → `route.js` |

### Step 1: `app` 디렉토리 생성

```bash
npm install next@latest
```

프로젝트 루트 (또는 `src/`)에 새로운 `app` 디렉토리 생성

### Step 2: 루트 레이아웃 생성

`app/layout.tsx` 파일 생성:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

```jsx
export default function RootLayout({ children }) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

**주의 사항:**
- `app` 디렉토리는 반드시 루트 레이아웃 포함
- 루트 레이아웃은 `<html>`, `<body>` 태그 정의 필수
- `.js`, `.jsx`, `.tsx` 확장자 사용 가능

#### 메타데이터 추가

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Home',
  description: 'Welcome to Next.js',
}
```

#### `_document.js`와 `_app.js` 마이그레이션

전역 스타일 등의 내용을 루트 레이아웃(`app/layout.tsx`)으로 복사하세요. `app/layout.tsx`의 스타일은 `pages/*`에 적용되지 않으므로 마이그레이션 완료 전까지 `_app`/`_document`를 유지하세요.

React Context providers는 [Client Component](/app-router/getting-started/server-and-client-components.md)로 이동 필요

#### `getLayout()` 패턴을 Layouts으로 마이그레이션 (선택)

**Before:**
```jsx
// components/DashboardLayout.js
export default function DashboardLayout({ children }) {
  return (
    <div>
      <h2>My Dashboard</h2>
      {children}
    </div>
  )
}

// pages/dashboard/index.js
import DashboardLayout from '../components/DashboardLayout'

export default function Page() {
  return <p>My Page</p>
}

Page.getLayout = function getLayout(page) {
  return <DashboardLayout>{page}</DashboardLayout>
}
```

**After:**
```jsx
// app/dashboard/page.js
export default function Page() {
  return <p>My Page</p>
}

// app/dashboard/DashboardLayout.js
'use client'

export default function DashboardLayout({ children }) {
  return (
    <div>
      <h2>My Dashboard</h2>
      {children}
    </div>
  )
}

// app/dashboard/layout.js
import DashboardLayout from './DashboardLayout'

export default function Layout({ children }) {
  return <DashboardLayout>{children}</DashboardLayout>
}
```

### Step 3: `next/head` 마이그레이션

**Before:**
```tsx
import Head from 'next/head'

export default function Page() {
  return (
    <>
      <Head>
        <title>My page title</title>
      </Head>
    </>
  )
}
```

**After:**
```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'My Page Title',
}

export default function Page() {
  return '...'
}
```

[모든 메타데이터 옵션 보기](/app-router/api-reference/functions/generate-metadata.md)

### Step 4: 페이지 마이그레이션

#### 라우팅 매핑

| `pages` 디렉토리 | `app` 디렉토리 | 라우트 |
|-----------------|--------------|--------|
| `index.js` | `page.js` | `/` |
| `about.js` | `about/page.js` | `/about` |
| `blog/[slug].js` | `blog/[slug]/page.js` | `/blog/post-1` |

#### Step 4.1: Client Component 생성

```tsx
// app/home-page.tsx
'use client'

export default function HomePage({ recentPosts }) {
  return (
    <div>
      {recentPosts.map((post) => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  )
}
```

#### Step 4.2: 새로운 페이지 생성

```tsx
// app/page.tsx
import HomePage from './home-page'

async function getPosts() {
  const res = await fetch('https://...')
  const posts = await res.json()
  return posts
}

export default async function Page() {
  const recentPosts = await getPosts()
  return <HomePage recentPosts={recentPosts} />
}
```

### Step 5: 라우팅 훅 마이그레이션

새로운 훅은 `next/navigation`에서 import합니다:

```tsx
'use client'

import { useRouter, usePathname, useSearchParams } from 'next/navigation'

export default function ExampleClientComponent() {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()
  // ...
}
```

#### 주요 변경 사항

- `next/router`의 `useRouter` → `next/navigation`의 `useRouter` (다른 동작)
- `pathname` 반환 없음 → `usePathname()` 사용
- `query` 객체 없음 → `useSearchParams()`, `useParams()` 사용
- `isFallback`, `locale`, `basePath`, `asPath`, `isReady`, `route` 제거

> **참고**: 이 훅들은 Client Components에서만 사용 가능 (Server Components 불가)

#### `pages`와 `app` 간 컴포넌트 공유

`next/compat/router`의 `useRouter` 훅을 사용하세요:
```jsx
import { useRouter } from 'next/compat/router'
```

### Step 6: 데이터 페칭 메서드 마이그레이션

```tsx
export default async function Page() {
  // 캐시된 데이터 (getStaticProps와 유사)
  const staticData = await fetch(`https://...`, { cache: 'force-cache' })

  // 매 요청마다 리페치 (getServerSideProps와 유사)
  const dynamicData = await fetch(`https://...`, { cache: 'no-store' })

  // 10초 캐시 (revalidate와 유사)
  const revalidatedData = await fetch(`https://...`, {
    next: { revalidate: 10 },
  })

  return <div>...</div>
}
```

#### Server-Side Rendering (`getServerSideProps`)

**Before (pages):**
```jsx
export async function getServerSideProps() {
  const res = await fetch(`https://...`)
  const projects = await res.json()
  return { props: { projects } }
}

export default function Dashboard({ projects }) {
  return (
    <ul>
      {projects.map((project) => (
        <li key={project.id}>{project.name}</li>
      ))}
    </ul>
  )
}
```

**After (app):**
```jsx
async function getProjects() {
  const res = await fetch(`https://...`, { cache: 'no-store' })
  const projects = await res.json()
  return projects
}

export default async function Dashboard() {
  const projects = await getProjects()
  return (
    <ul>
      {projects.map((project) => (
        <li key={project.id}>{project.name}</li>
      ))}
    </ul>
  )
}
```

#### 요청 객체 접근

**Before (pages):**
```jsx
export async function getServerSideProps({ req, query }) {
  const authHeader = req.getHeaders()['authorization']
  const theme = req.cookies['theme']
  return { props: { ... } }
}
```

**After (app):**
```tsx
import { cookies, headers } from 'next/headers'

async function getData() {
  const authHeader = (await headers()).get('authorization')
  return '...'
}

export default async function Page() {
  const theme = (await cookies()).get('theme')
  const data = await getData()
  return '...'
}
```

#### Static Site Generation (`getStaticProps`)

**Before (pages):**
```jsx
export async function getStaticProps() {
  const res = await fetch(`https://...`)
  const projects = await res.json()
  return { props: { projects } }
}

export default function Index({ projects }) {
  return projects.map((project) => <div>{project.name}</div>)
}
```

**After (app):**
```jsx
async function getProjects() {
  const res = await fetch(`https://...`)
  const projects = await res.json()
  return projects
}

export default async function Index() {
  const projects = await getProjects()
  return projects.map((project) => <div>{project.name}</div>)
}
```

#### 동적 경로 (`getStaticPaths` → `generateStaticParams`)

**Before (pages):**
```jsx
export async function getStaticPaths() {
  return {
    paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
  }
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://.../posts/${params.id}`)
  const post = await res.json()
  return { props: { post } }
}

export default function Post({ post }) {
  return <PostLayout post={post} />
}
```

**After (app):**
```jsx
export async function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }]
}

async function getPost(params) {
  const res = await fetch(`https://.../posts/${(await params).id}`)
  const post = await res.json()
  return post
}

export default async function Post({ params }) {
  const post = await getPost(params)
  return <PostLayout post={post} />
}
```

#### `fallback` 대체

**Before (pages):**
```jsx
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: 'blocking'
  }
}
```

**After (app):**
```jsx
export const dynamicParams = true

export async function generateStaticParams() {
  return [...]
}
```

`dynamicParams` 옵션:
- **`true`** (기본값): `generateStaticParams`에 없는 동적 세그먼트는 온디맨드 생성
- **`false`**: `generateStaticParams`에 없는 동적 세그먼트는 404 반환

#### Incremental Static Regeneration (`revalidate`)

**Before (pages):**
```jsx
export async function getStaticProps() {
  const res = await fetch(`https://.../posts`)
  const posts = await res.json()
  return {
    props: { posts },
    revalidate: 60,
  }
}
```

**After (app):**
```jsx
async function getPosts() {
  const res = await fetch(`https://.../posts`, { next: { revalidate: 60 } })
  const data = await res.json()
  return data.posts
}

export default async function PostList() {
  const posts = await getPosts()
  return posts.map((post) => <div>{post.name}</div>)
}
```

#### API Routes

**기존 API Routes** (`pages/api/*`)는 계속 작동하지만, **Route Handlers** (`app` 디렉토리)로 대체됩니다.

```ts
// app/api/route.ts
export async function GET(request: Request) {}
```

> **참고**: API를 클라이언트에서 호출하던 경우, [Server Components](/app-router/getting-started/server-and-client-components.md)를 대신 사용하면 안전하게 데이터를 페칭할 수 있습니다.

### Step 7: 스타일링

`pages` 디렉토리에서는 전역 스타일시트가 `pages/_app.js`에만 제한되었지만, `app` 디렉토리에서는 모든 레이아웃, 페이지, 컴포넌트에 추가할 수 있습니다.

지원하는 스타일링 방법:
- [CSS Modules](/app-router/getting-started/css.md#css-modules)
- [Tailwind CSS](/app-router/getting-started/css.md#tailwind-css)
- [전역 스타일](/app-router/getting-started/css.md#global-css)
- [CSS-in-JS](/app-router/guides/css-in-js.md)
- [외부 스타일시트](/app-router/getting-started/css.md#external-stylesheets)
- [Sass](/app-router/guides/sass.md)

#### Tailwind CSS 설정

`tailwind.config.js` 업데이트:

```js
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}', // ← 이 라인 추가
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
}
```

`app/layout.js`에 전역 스타일 import:

```jsx
import '../styles/globals.css'

export default function RootLayout({ children }) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  )
}
```

---

## App Router와 Pages Router 함께 사용

서로 다른 Next.js 라우터로 제공되는 경로 간 네비게이션 시 하드 네비게이션이 발생합니다. `next/link`의 자동 프리페칭은 라우터를 넘어 작동하지 않습니다.

---

## 코드모드

Next.js는 기능이 사용 중단될 때 코드베이스를 업그레이드하는 데 도움이 되는 Codemod 변환을 제공합니다. [Codemods](/app-router/guides/upgrading/codemods.md) 자세히 알아보기
