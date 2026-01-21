# 페이지와 레이아웃

Pages Router는 `pages` 디렉토리의 **파일 시스템 기반 라우터**입니다. `pages` 디렉토리에 파일을 추가하면 자동으로 라우트로 사용할 수 있습니다.

## 페이지

페이지는 `pages` 디렉토리의 `.js`, `.jsx`, `.ts`, `.tsx` 파일에서 내보낸 React 컴포넌트입니다. 각 페이지는 파일 이름을 기반으로 라우트와 연결됩니다.

### Index 라우트

라우터는 `index`라는 이름의 파일을 디렉토리의 루트로 자동 라우팅합니다:

```
pages/index.js → /
pages/blog/index.js → /blog
```

### 중첩 라우트

라우터는 중첩된 파일을 지원합니다. 중첩 폴더 구조를 만들면 파일이 동일한 방식으로 라우팅됩니다:

```
pages/blog/first-post.js → /blog/first-post
pages/dashboard/settings/username.js → /dashboard/settings/username
```

### 동적 라우트

대괄호를 사용하여 동적 세그먼트를 만들 수 있습니다:

```
pages/posts/[id].js → /posts/1, /posts/2 등
```

[동적 라우트](/docs/pages/building-your-application/routing/dynamic-routes)에서 자세히 알아보세요.

## 레이아웃

레이아웃 패턴을 사용하면 여러 페이지에서 UI를 공유할 수 있습니다. 페이지 간 이동 시 레이아웃은 상태를 유지하고, 상호작용을 보존하며, 다시 렌더링되지 않습니다.

### 기본 레이아웃 컴포넌트

```jsx filename="components/layout.js"
import Navbar from './navbar'
import Footer from './footer'

export default function Layout({ children }) {
  return (
    <>
      <Navbar />
      <main>{children}</main>
      <Footer />
    </>
  )
}
```

## 단일 공유 레이아웃

전체 애플리케이션에서 하나의 레이아웃만 사용하려면 [Custom App](/docs/pages/api-reference/file-conventions/_app)을 사용합니다. 레이아웃 컴포넌트를 만들고 `pages/_app.js`에서 페이지를 감싸세요:

```jsx filename="pages/_app.js"
import Layout from '../components/layout'

export default function MyApp({ Component, pageProps }) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  )
}
```

## 페이지별 레이아웃

여러 레이아웃이 필요한 경우 페이지에 `getLayout` 속성을 추가하여 React 컴포넌트를 반환할 수 있습니다. 이를 통해 페이지별로 레이아웃을 정의할 수 있습니다.

```jsx filename="pages/index.js"
import Layout from '../components/layout'
import NestedLayout from '../components/nested-layout'

export default function Page() {
  return (
    /** 컨텐츠 */
  )
}

Page.getLayout = function getLayout(page) {
  return (
    <Layout>
      <NestedLayout>{page}</NestedLayout>
    </Layout>
  )
}
```

```jsx filename="pages/_app.js"
export default function MyApp({ Component, pageProps }) {
  // 페이지 레벨에서 정의된 레이아웃 사용
  const getLayout = Component.getLayout ?? ((page) => page)

  return getLayout(<Component {...pageProps} />)
}
```

페이지 간 이동 시 Single-Page Application(SPA) 경험을 위해 페이지 상태(입력값, 스크롤 위치 등)가 유지됩니다. 이 레이아웃 패턴은 페이지 전환 간에 React 컴포넌트 트리가 유지되기 때문에 상태 유지가 가능합니다. 컴포넌트 트리를 통해 React는 어떤 요소가 변경되었는지 이해하여 상태를 보존할 수 있습니다.

## TypeScript

TypeScript를 사용할 때는 먼저 `getLayout` 함수를 포함하는 페이지의 새 타입을 만들어야 합니다. 그런 다음 이전에 만든 타입을 사용하도록 `Component` 속성을 재정의하는 `AppProps`의 새 타입을 만들어야 합니다.

```tsx filename="pages/_app.tsx"
import type { ReactElement, ReactNode } from 'react'
import type { NextPage } from 'next'
import type { AppProps } from 'next/app'

export type NextPageWithLayout<P = {}, IP = P> = NextPage<P, IP> & {
  getLayout?: (page: ReactElement) => ReactNode
}

type AppPropsWithLayout = AppProps & {
  Component: NextPageWithLayout
}

export default function MyApp({ Component, pageProps }: AppPropsWithLayout) {
  // 페이지 레벨에서 정의된 레이아웃 사용
  const getLayout = Component.getLayout ?? ((page) => page)

  return getLayout(<Component {...pageProps} />)
}
```

```tsx filename="pages/index.tsx"
import type { ReactElement } from 'react'
import type { NextPageWithLayout } from './_app'
import Layout from '../components/layout'
import NestedLayout from '../components/nested-layout'

const Page: NextPageWithLayout = () => {
  return <p>hello world</p>
}

Page.getLayout = function getLayout(page: ReactElement) {
  return (
    <Layout>
      <NestedLayout>{page}</NestedLayout>
    </Layout>
  )
}

export default Page
```

## 레이아웃에서 데이터 페칭

레이아웃 내부에서는 `useEffect`나 [SWR](https://swr.vercel.app/)과 같은 라이브러리를 사용하여 클라이언트 측에서 데이터를 가져올 수 있습니다. 이 파일이 [페이지](/docs/pages/building-your-application/routing/pages-and-layouts)가 아니기 때문에 `getStaticProps` 또는 `getServerSideProps`를 사용할 수 없습니다.

```jsx filename="components/layout.js"
import useSWR from 'swr'
import Navbar from './navbar'
import Footer from './footer'

export default function Layout({ children }) {
  const { data, error } = useSWR('/api/navigation', fetcher)

  if (error) return <div>불러오기 실패</div>
  if (!data) return <div>로딩 중...</div>

  return (
    <>
      <Navbar links={data.links} />
      <main>{children}</main>
      <Footer />
    </>
  )
}
```

## 예제

### 대시보드 레이아웃

```jsx filename="components/dashboard-layout.js"
import Sidebar from './sidebar'
import Header from './header'

export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <div className="main-content">
        <Header />
        <main>{children}</main>
      </div>
    </div>
  )
}
```

```jsx filename="pages/dashboard/index.js"
import DashboardLayout from '../../components/dashboard-layout'

export default function DashboardHome() {
  return <h1>대시보드 홈</h1>
}

DashboardHome.getLayout = function getLayout(page) {
  return <DashboardLayout>{page}</DashboardLayout>
}
```

### 인증 레이아웃

```jsx filename="components/auth-layout.js"
export default function AuthLayout({ children }) {
  return (
    <div className="auth-container">
      <div className="auth-box">
        <h1>로고</h1>
        {children}
      </div>
    </div>
  )
}
```

```jsx filename="pages/login.js"
import AuthLayout from '../components/auth-layout'

export default function Login() {
  return (
    <form>
      <input type="email" placeholder="이메일" />
      <input type="password" placeholder="비밀번호" />
      <button type="submit">로그인</button>
    </form>
  )
}

Login.getLayout = function getLayout(page) {
  return <AuthLayout>{page}</AuthLayout>
}
```

## 관련 문서

- [_app.js](/docs/pages/api-reference/file-conventions/_app) - Custom App
- [_document.js](/docs/pages/api-reference/file-conventions/_document) - Custom Document
- [동적 라우트](/docs/pages/building-your-application/routing/dynamic-routes)
