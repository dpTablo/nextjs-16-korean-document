# 링크 및 네비게이션

Next.js에서 라우트 간 이동하는 방법에는 두 가지가 있습니다:

- [`<Link>` 컴포넌트](#link-컴포넌트) 사용
- [`useRouter` 훅](#userouter-훅) 사용

이 페이지에서는 `<Link>`, `useRouter()`의 사용 방법과 네비게이션 작동 방식에 대해 자세히 알아봅니다.

## Link 컴포넌트

`<Link>`는 HTML `<a>` 요소를 확장한 내장 컴포넌트로, [프리페칭](#프리페칭)과 클라이언트 사이드 네비게이션을 제공합니다. Next.js에서 라우트 간 이동의 기본적이고 권장되는 방법입니다.

```jsx
import Link from 'next/link'

function Home() {
  return (
    <ul>
      <li>
        <Link href="/">홈</Link>
      </li>
      <li>
        <Link href="/about">소개</Link>
      </li>
      <li>
        <Link href="/blog/hello-world">블로그 포스트</Link>
      </li>
    </ul>
  )
}

export default Home
```

**경로 매핑:**

| `href` | 파일 |
|--------|------|
| `/` | `pages/index.js` |
| `/about` | `pages/about.js` |
| `/blog/hello-world` | `pages/blog/[slug].js` |

## 동적 경로로 링크

동적 세그먼트가 있는 라우트에 링크할 때 문자열 보간이나 URL 객체를 사용할 수 있습니다.

### 문자열 보간

```jsx
import Link from 'next/link'

function Posts({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${encodeURIComponent(post.slug)}`}>
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  )
}

export default Posts
```

> UTF-8 호환성을 위해 `encodeURIComponent`를 사용하세요.

### URL 객체

```jsx
import Link from 'next/link'

function Posts({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link
            href={{
              pathname: '/blog/[slug]',
              query: { slug: post.slug },
            }}
          >
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  )
}

export default Posts
```

**URL 객체 속성:**

| 속성 | 설명 | 예시 |
|------|------|------|
| `pathname` | `pages` 디렉토리의 페이지 경로 | `/blog/[slug]` |
| `query` | 동적 세그먼트 객체 | `{ slug: 'my-post' }` |

## useRouter 훅

함수형 컴포넌트 내에서 [`router` 객체](/docs/pages/api-reference/functions/use-router)에 접근하려면 `useRouter` 훅을 사용할 수 있습니다.

```jsx
import { useRouter } from 'next/router'

export default function ReadMore() {
  const router = useRouter()

  return (
    <button onClick={() => router.push('/about')}>
      더 알아보기
    </button>
  )
}
```

## 명령형 네비게이션

`<Link>`로 대부분의 라우팅 요구사항을 충족할 수 있지만, `useRouter`를 사용한 클라이언트 사이드 네비게이션도 가능합니다.

### router.push

새 URL을 히스토리 스택에 추가합니다:

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  const handleClick = () => {
    router.push('/dashboard')
  }

  return <button onClick={handleClick}>대시보드로 이동</button>
}
```

### router.replace

히스토리 스택에 새 항목을 추가하지 않고 현재 URL을 교체합니다:

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  const handleClick = () => {
    router.replace('/login')
  }

  return <button onClick={handleClick}>로그인 페이지로</button>
}
```

### 동적 경로로 이동

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  const navigateToPost = (slug) => {
    router.push({
      pathname: '/blog/[slug]',
      query: { slug },
    })
  }

  return (
    <button onClick={() => navigateToPost('hello-world')}>
      포스트 보기
    </button>
  )
}
```

## Shallow Routing

Shallow routing을 사용하면 [`getServerSideProps`](/docs/pages/api-reference/functions/getServerSideProps), [`getStaticProps`](/docs/pages/api-reference/functions/getStaticProps), [`getInitialProps`](/docs/pages/api-reference/functions/getInitialProps)를 포함한 데이터 페칭 메서드를 다시 실행하지 않고 URL을 변경할 수 있습니다.

업데이트된 `pathname`과 `query`는 상태를 잃지 않고 [`router` 객체](/docs/pages/api-reference/functions/use-router) (`useRouter` 또는 `withRouter`로 추가됨)를 통해 받을 수 있습니다.

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'

// 현재 URL: '/'
function Page() {
  const router = useRouter()

  useEffect(() => {
    // 첫 렌더링 후에 네비게이션 수행
    router.push('/?counter=10', undefined, { shallow: true })
  }, [])

  useEffect(() => {
    // counter가 변경되었음!
  }, [router.query.counter])
}

export default Page
```

### Shallow Routing 주의사항

Shallow routing은 **현재 페이지**의 URL 변경에만 작동합니다. 예를 들어, `pages/about.js`라는 다른 페이지가 있고 다음을 실행하면:

```jsx
router.push('/?counter=10', '/about?counter=10', { shallow: true })
```

이것은 새 페이지이므로, 현재 페이지를 언로드하고 새 페이지를 로드하며 데이터 페칭을 기다립니다. shallow routing을 요청했더라도 마찬가지입니다.

## 프리페칭

`<Link>` 컴포넌트가 뷰포트에 들어올 때 Next.js는 자동으로 링크된 라우트를 프리페칭합니다.

- **정적 생성 페이지**: JSON 데이터를 포함하여 프리페칭됩니다.
- **서버 렌더링 라우트**: `<Link>`가 클릭될 때만 데이터를 페칭합니다.

프리페칭을 비활성화하려면:

```jsx
<Link href="/about" prefetch={false}>
  소개
</Link>
```

## 예제

### 조건부 네비게이션

```jsx
import { useRouter } from 'next/router'

export default function LoginButton() {
  const router = useRouter()

  const handleLogin = async () => {
    const success = await login()
    if (success) {
      router.push('/dashboard')
    } else {
      router.push('/login?error=failed')
    }
  }

  return <button onClick={handleLogin}>로그인</button>
}
```

### 쿼리 파라미터 업데이트

```jsx
import { useRouter } from 'next/router'

export default function FilterProducts() {
  const router = useRouter()

  const updateFilter = (category) => {
    router.push(
      {
        pathname: router.pathname,
        query: { ...router.query, category },
      },
      undefined,
      { shallow: true }
    )
  }

  return (
    <div>
      <button onClick={() => updateFilter('electronics')}>전자제품</button>
      <button onClick={() => updateFilter('clothing')}>의류</button>
    </div>
  )
}
```

### 뒤로가기/앞으로가기 감지

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  useEffect(() => {
    const handleRouteChange = (url) => {
      console.log('라우트가 변경됨:', url)
    }

    router.events.on('routeChangeStart', handleRouteChange)

    return () => {
      router.events.off('routeChangeStart', handleRouteChange)
    }
  }, [router])

  return <div>페이지 컨텐츠</div>
}
```

## 관련 문서

- [useRouter](/docs/pages/api-reference/functions/use-router) - 라우터 API 레퍼런스
- [Link](/docs/pages/api-reference/components/link) - Link 컴포넌트 API
- [동적 라우트](/docs/pages/building-your-application/routing/dynamic-routes)
