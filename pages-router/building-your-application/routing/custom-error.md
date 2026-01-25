# Custom Errors (커스텀 에러)

Next.js는 모든 방문에서 서버 렌더링 에러를 피하기 위해 정적 에러 페이지와 함께 내장 에러 처리를 제공합니다. 이는 비용을 줄이고 경험을 빠르게 합니다.

## 404 페이지 커스터마이징

Next.js는 기본 정적 404 페이지를 제공하지만, `pages/404.js` 파일을 생성하여 커스터마이징할 수 있습니다. 이 파일은 빌드 시점에 정적으로 생성됩니다:

```jsx
// pages/404.js
export default function Custom404() {
  return <h1>404 - 페이지를 찾을 수 없습니다</h1>
}
```

빌드 시점에 데이터를 가져와야 하는 경우 이 페이지 내에서 [`getStaticProps`](/pages-router/api-reference/getStaticProps.md)를 사용할 수 있습니다.

## 500 페이지 커스터마이징

Next.js는 기본 정적 500 페이지를 제공합니다. `pages/500.js` 파일을 생성하여 커스터마이징하세요:

```jsx
// pages/500.js
export default function Custom500() {
  return <h1>500 - 서버 측 에러가 발생했습니다</h1>
}
```

여기에서도 필요한 경우 [`getStaticProps`](/pages-router/api-reference/getStaticProps.md)를 사용할 수 있습니다.

## 고급 에러 페이지 커스터마이징

클라이언트 및 서버 측 모두에서 더 고급 에러 처리를 위해 `pages/_error.js` 파일을 생성합니다:

```jsx
// pages/_error.js
function Error({ statusCode }) {
  return (
    <p>
      {statusCode
        ? `서버에서 ${statusCode} 에러가 발생했습니다`
        : '클라이언트에서 에러가 발생했습니다'}
    </p>
  )
}

Error.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404
  return { statusCode }
}

export default Error
```

> **참고**: `pages/_error.js`는 프로덕션에서만 사용됩니다. 개발 중에는 에러와 함께 콜 스택을 볼 수 있습니다.

## 내장 에러 페이지 재사용

기본 에러 페이지를 렌더링하려면 `next/error`에서 `Error` 컴포넌트를 가져옵니다:

```jsx
// pages/post/[id].js
import Error from 'next/error'

export async function getServerSideProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`)
  const errorCode = res.ok ? false : res.status
  const post = await res.json()

  return {
    props: { errorCode, post },
  }
}

export default function Post({ errorCode, post }) {
  if (errorCode) {
    return <Error statusCode={errorCode} />
  }

  return <div>{post.title}</div>
}
```

`Error` 컴포넌트는 커스텀 텍스트 메시지를 위한 `title` 속성도 받습니다:

```jsx
<Error statusCode={404} title="게시물을 찾을 수 없습니다" />
```

## 주의사항

- `Error`는 `getStaticProps`나 `getServerSideProps`와 같은 Next.js 데이터 페칭 메서드를 지원하지 않습니다
- `_error`는 `_app`과 같은 예약된 경로명이며 라우팅을 통해 직접 접근할 수 없습니다
- `/_error`에 직접 접근하거나 커스텀 서버에서 접근하면 404가 렌더링됩니다

## 에러 처리 우선순위

1. **404 에러**: `pages/404.js` → `pages/_error.js` → 기본 404 페이지
2. **500 에러**: `pages/500.js` → `pages/_error.js` → 기본 500 페이지
3. **기타 에러**: `pages/_error.js` → 기본 에러 페이지

## 에러 경계와 함께 사용

React 에러 경계와 함께 사용하여 클라이언트 측 에러를 더 세밀하게 처리할 수 있습니다:

```jsx
// components/ErrorBoundary.js
import React from 'react'

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error) {
    return { hasError: true }
  }

  componentDidCatch(error, errorInfo) {
    console.log({ error, errorInfo })
  }

  render() {
    if (this.state.hasError) {
      return <h1>문제가 발생했습니다.</h1>
    }

    return this.props.children
  }
}

export default ErrorBoundary
```

---

## 참고

- [404.js](/pages-router/api-reference/file-conventions/404.md)
- [500.js](/pages-router/api-reference/file-conventions/500.md)
- [_error.js](/pages-router/api-reference/file-conventions/_error.md)
