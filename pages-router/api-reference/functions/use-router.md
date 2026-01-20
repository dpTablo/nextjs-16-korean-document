# useRouter

`useRouter`는 Pages Router에서 라우터 객체에 접근하고 클라이언트 사이드 네비게이션을 수행하는 Hook입니다.

---

## 기본 사용법

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return (
    <button onClick={() => router.push('/about')}>
      더 읽기
    </button>
  )
}
```

---

## Router 객체

`useRouter`가 반환하는 `router` 객체는 다음 속성들을 포함합니다.

### 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| `pathname` | `string` | 현재 라우트의 경로 (`/pages` 이후) |
| `query` | `object` | 쿼리 스트링과 동적 라우트 파라미터 |
| `asPath` | `string` | 브라우저에 표시되는 실제 경로 (쿼리 포함) |
| `basePath` | `string` | 활성화된 basePath |
| `locale` | `string` | 현재 활성 로케일 |
| `locales` | `string[]` | 지원하는 모든 로케일 |
| `defaultLocale` | `string` | 기본 로케일 |
| `domainLocales` | `array` | 설정된 도메인 로케일 |
| `isReady` | `boolean` | 라우터 필드가 클라이언트에서 업데이트되어 사용 가능한지 여부 |
| `isPreview` | `boolean` | Preview Mode 활성화 여부 |
| `isFallback` | `boolean` | Fallback 페이지 렌더링 중인지 여부 |

### pathname과 query 사용

```jsx
import { useRouter } from 'next/router'

export default function BlogPost() {
  const router = useRouter()
  const { pathname, query } = router

  return (
    <div>
      <p>현재 경로: {pathname}</p>
      <p>포스트 ID: {query.id}</p>
    </div>
  )
}
```

---

## Router 메서드

### router.push()

클라이언트 사이드 네비게이션을 수행합니다.

```jsx
router.push(url, as, options)
```

**파라미터:**

| 파라미터 | 타입 | 설명 |
|----------|------|------|
| `url` | `string \| UrlObject` | 이동할 URL |
| `as` | `string \| UrlObject` | 브라우저 URL 바에 표시될 경로 (선택) |
| `options` | `object` | 추가 옵션 |

**옵션:**

| 옵션 | 타입 | 설명 |
|------|------|------|
| `shallow` | `boolean` | 데이터 페칭 메서드 재실행 없이 URL 변경 |
| `locale` | `string` | 새 페이지의 로케일 |
| `scroll` | `boolean` | 네비게이션 후 상단으로 스크롤 (기본: `true`) |

```jsx
import { useRouter } from 'next/router'

export default function Navigation() {
  const router = useRouter()

  return (
    <>
      {/* 기본 네비게이션 */}
      <button onClick={() => router.push('/about')}>
        소개 페이지
      </button>

      {/* URL 객체 사용 */}
      <button onClick={() => router.push({
        pathname: '/post/[id]',
        query: { id: '123' },
      })}>
        포스트 보기
      </button>

      {/* 옵션 사용 */}
      <button onClick={() => router.push('/dashboard', undefined, { shallow: true })}>
        대시보드 (shallow)
      </button>
    </>
  )
}
```

### router.replace()

히스토리 스택에 새 항목을 추가하지 않고 URL을 변경합니다.

```jsx
router.replace(url, as, options)
```

```jsx
import { useRouter } from 'next/router'

export default function LoginPage() {
  const router = useRouter()

  const handleLogin = async () => {
    const success = await login()
    if (success) {
      // 뒤로 가기 시 로그인 페이지로 돌아가지 않음
      router.replace('/dashboard')
    }
  }

  return <button onClick={handleLogin}>로그인</button>
}
```

### router.prefetch()

더 빠른 클라이언트 사이드 전환을 위해 페이지를 프리페치합니다.

```jsx
router.prefetch(url, as, options)
```

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  useEffect(() => {
    // 대시보드 페이지 미리 로드
    router.prefetch('/dashboard')
  }, [router])

  return (
    <button onClick={() => router.push('/dashboard')}>
      대시보드로 이동
    </button>
  )
}
```

### router.back()

히스토리에서 이전 페이지로 이동합니다.

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return <button onClick={() => router.back()}>뒤로 가기</button>
}
```

### router.forward()

히스토리에서 다음 페이지로 이동합니다.

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return <button onClick={() => router.forward()}>앞으로 가기</button>
}
```

### router.reload()

현재 URL을 다시 로드합니다.

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return <button onClick={() => router.reload()}>새로고침</button>
}
```

### router.beforePopState()

브라우저의 뒤로 가기/앞으로 가기 전에 실행할 콜백을 설정합니다.

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  useEffect(() => {
    router.beforePopState(({ url, as, options }) => {
      // 특정 조건에서만 네비게이션 허용
      if (as !== '/' && as !== '/other') {
        window.location.href = as
        return false
      }
      return true
    })

    return () => {
      router.beforePopState(() => true)
    }
  }, [router])

  return <p>beforePopState 예제</p>
}
```

---

## Router Events

라우터 이벤트를 구독하여 네비게이션 상태를 추적할 수 있습니다.

### 사용 가능한 이벤트

| 이벤트 | 설명 |
|--------|------|
| `routeChangeStart(url, { shallow })` | 라우트 변경 시작 |
| `routeChangeComplete(url, { shallow })` | 라우트 변경 완료 |
| `routeChangeError(err, url, { shallow })` | 라우트 변경 에러 또는 취소 |
| `beforeHistoryChange(url, { shallow })` | 브라우저 히스토리 변경 전 |
| `hashChangeStart(url, { shallow })` | 해시 변경 시작 |
| `hashChangeComplete(url, { shallow })` | 해시 변경 완료 |

### 이벤트 구독

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'

export default function MyApp({ Component, pageProps }) {
  const router = useRouter()

  useEffect(() => {
    const handleRouteChangeStart = (url, { shallow }) => {
      console.log(`라우트 변경 시작: ${url}, shallow: ${shallow}`)
    }

    const handleRouteChangeComplete = (url, { shallow }) => {
      console.log(`라우트 변경 완료: ${url}`)
    }

    const handleRouteChangeError = (err, url) => {
      if (err.cancelled) {
        console.log(`${url}로의 라우트 변경이 취소되었습니다`)
      } else {
        console.error(`라우트 변경 에러:`, err)
      }
    }

    router.events.on('routeChangeStart', handleRouteChangeStart)
    router.events.on('routeChangeComplete', handleRouteChangeComplete)
    router.events.on('routeChangeError', handleRouteChangeError)

    return () => {
      router.events.off('routeChangeStart', handleRouteChangeStart)
      router.events.off('routeChangeComplete', handleRouteChangeComplete)
      router.events.off('routeChangeError', handleRouteChangeError)
    }
  }, [router])

  return <Component {...pageProps} />
}
```

### 로딩 인디케이터 예제

```jsx
import { useState, useEffect } from 'react'
import { useRouter } from 'next/router'

export default function MyApp({ Component, pageProps }) {
  const router = useRouter()
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    const handleStart = () => setLoading(true)
    const handleComplete = () => setLoading(false)

    router.events.on('routeChangeStart', handleStart)
    router.events.on('routeChangeComplete', handleComplete)
    router.events.on('routeChangeError', handleComplete)

    return () => {
      router.events.off('routeChangeStart', handleStart)
      router.events.off('routeChangeComplete', handleComplete)
      router.events.off('routeChangeError', handleComplete)
    }
  }, [router])

  return (
    <>
      {loading && <div className="loading-spinner">로딩 중...</div>}
      <Component {...pageProps} />
    </>
  )
}
```

---

## Shallow Routing

Shallow 라우팅을 사용하면 `getServerSideProps`, `getStaticProps`, `getInitialProps`를 재실행하지 않고 URL을 변경할 수 있습니다.

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  useEffect(() => {
    // URL을 /?counter=10 으로 변경하지만 데이터 페칭 재실행 안 함
    router.push('/?counter=10', undefined, { shallow: true })
  }, [])

  useEffect(() => {
    // 쿼리 변경에 반응
    console.log('counter:', router.query.counter)
  }, [router.query.counter])

  return <p>Shallow Routing 예제</p>
}
```

> **주의**: Shallow 라우팅은 현재 페이지 내에서의 URL 변경에만 작동합니다. 다른 페이지로 이동하면 정상적으로 데이터 페칭이 실행됩니다.

---

## withRouter

클래스 컴포넌트에서 `router` 객체에 접근하려면 `withRouter` HOC를 사용합니다.

```jsx
import { withRouter } from 'next/router'

class Page extends React.Component {
  render() {
    return <p>현재 경로: {this.props.router.pathname}</p>
  }
}

export default withRouter(Page)
```

---

## TypeScript

```tsx
import { useRouter, NextRouter } from 'next/router'

export default function Page() {
  const router: NextRouter = useRouter()

  return <p>현재 경로: {router.pathname}</p>
}
```

---

## 중요한 주의사항

> **Good to know**:
> - `useRouter`는 React Hook이므로 클래스 컴포넌트에서 사용할 수 없습니다
> - `router.isReady`가 `true`가 되기 전에는 `query`가 빈 객체일 수 있습니다
> - Shallow 라우팅은 같은 페이지 내에서만 동작합니다

> **App Router 마이그레이션**:
> - App Router에서는 `next/navigation`의 `useRouter`를 사용합니다
> - `usePathname`과 `useSearchParams`로 기능이 분리되었습니다
> - 라우터 이벤트는 다른 방식으로 처리합니다

---

## 관련 문서

- [Link 컴포넌트](../components/link.md)
- [getStaticProps](../getStaticProps.md)
- [getServerSideProps](../getServerSideProps.md)
