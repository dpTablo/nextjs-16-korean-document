# Custom App (_app)

Next.js의 `App` 컴포넌트는 모든 페이지를 초기화하는 데 사용됩니다. 이를 오버라이드하여 페이지 간 공유 레이아웃, 글로벌 상태, 글로벌 CSS 등을 설정할 수 있습니다.

---

## 기본 사용법

`pages/_app.js` (또는 `pages/_app.tsx`) 파일을 생성하여 기본 `App`을 오버라이드합니다.

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'

export default function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

```jsx
// pages/_app.js
export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

---

## Props

| Prop | 타입 | 설명 |
|------|------|------|
| `Component` | `NextComponentType` | 활성 페이지 컴포넌트. 라우트 변경 시 새로운 페이지로 변경됨 |
| `pageProps` | `object` | 데이터 페칭 메서드로 미리 로드된 초기 props (없으면 빈 객체) |

---

## 사용 사례

### 글로벌 CSS 추가

```tsx
// pages/_app.tsx
import '@/styles/globals.css'
import type { AppProps } from 'next/app'

export default function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

### 공유 레이아웃

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'
import Layout from '@/components/Layout'

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  )
}
```

### 글로벌 상태 관리 (Context)

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'
import { ThemeProvider } from '@/contexts/ThemeContext'
import { AuthProvider } from '@/contexts/AuthContext'

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <Component {...pageProps} />
      </ThemeProvider>
    </AuthProvider>
  )
}
```

### 페이지별 레이아웃

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'
import type { ReactElement, ReactNode } from 'react'
import type { NextPage } from 'next'

export type NextPageWithLayout<P = {}, IP = P> = NextPage<P, IP> & {
  getLayout?: (page: ReactElement) => ReactNode
}

type AppPropsWithLayout = AppProps & {
  Component: NextPageWithLayout
}

export default function MyApp({ Component, pageProps }: AppPropsWithLayout) {
  // 페이지별 레이아웃이 있으면 사용, 없으면 기본 레이아웃
  const getLayout = Component.getLayout ?? ((page) => page)

  return getLayout(<Component {...pageProps} />)
}
```

```tsx
// pages/dashboard.tsx
import type { ReactElement } from 'react'
import DashboardLayout from '@/components/DashboardLayout'
import type { NextPageWithLayout } from './_app'

const DashboardPage: NextPageWithLayout = () => {
  return <p>대시보드 콘텐츠</p>
}

DashboardPage.getLayout = function getLayout(page: ReactElement) {
  return <DashboardLayout>{page}</DashboardLayout>
}

export default DashboardPage
```

### 에러 바운더리

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'
import { ErrorBoundary } from '@/components/ErrorBoundary'

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ErrorBoundary>
      <Component {...pageProps} />
    </ErrorBoundary>
  )
}
```

### 라우트 변경 감지

```tsx
// pages/_app.tsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'
import type { AppProps } from 'next/app'

export default function MyApp({ Component, pageProps }: AppProps) {
  const router = useRouter()

  useEffect(() => {
    const handleRouteChange = (url: string) => {
      console.log('라우트 변경:', url)
      // 애널리틱스 등 처리
    }

    router.events.on('routeChangeComplete', handleRouteChange)

    return () => {
      router.events.off('routeChangeComplete', handleRouteChange)
    }
  }, [router])

  return <Component {...pageProps} />
}
```

---

## getInitialProps 사용

`getInitialProps`를 `App`에서 사용할 수 있지만, **Automatic Static Optimization**이 비활성화됩니다.

```tsx
// pages/_app.tsx
import App, { AppContext, AppInitialProps, AppProps } from 'next/app'

type AppOwnProps = { example: string }

export default function MyApp({
  Component,
  pageProps,
  example,
}: AppProps & AppOwnProps) {
  return (
    <>
      <p>앱 데이터: {example}</p>
      <Component {...pageProps} />
    </>
  )
}

MyApp.getInitialProps = async (
  context: AppContext
): Promise<AppOwnProps & AppInitialProps> => {
  // 기본 App.getInitialProps 호출 (페이지의 getInitialProps 실행)
  const ctx = await App.getInitialProps(context)

  return { ...ctx, example: '앱 레벨 데이터' }
}
```

> **주의**: `App`에서 `getInitialProps`를 사용하면 모든 페이지가 서버 사이드 렌더링됩니다. 가능하면 이 패턴을 피하세요.

---

## 중요한 주의사항

> **Good to know**:
> - `pages/_app.js`가 없던 상태에서 추가하면 개발 서버를 재시작해야 합니다
> - `App`은 `getStaticProps`, `getServerSideProps`를 지원하지 않습니다
> - `getInitialProps` 사용 시 Automatic Static Optimization이 비활성화됩니다

> **App Router 마이그레이션**:
> - App Router에서는 `app/layout.tsx`가 `_app.js`를 대체합니다
> - 글로벌 CSS와 공유 레이아웃을 Root Layout에서 정의합니다
> - Context Provider는 Client Component로 분리합니다

---

## Data Fetching 미지원

`App` 컴포넌트는 다음 데이터 페칭 메서드를 지원하지 않습니다:

- `getStaticProps`
- `getServerSideProps`

앱 레벨에서 데이터가 필요한 경우:
1. `getInitialProps` 사용 (권장하지 않음)
2. 개별 페이지에서 데이터를 가져와 Context로 공유
3. App Router로 마이그레이션

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.4.0 | App Router 안정화 |
| v9.0.0 | TypeScript 지원 |

---

## 관련 문서

- [Custom Document (_document)](./custom-document.md)
- [getStaticProps](../getStaticProps.md)
- [getServerSideProps](../getServerSideProps.md)
- [App Router - Layout](../../../app-router/api-reference/file-conventions/layout.md)
