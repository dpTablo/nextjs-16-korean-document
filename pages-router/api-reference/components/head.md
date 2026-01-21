# <Head>

Next.js는 페이지의 `<head>` 요소에 내용을 추가하기 위한 내장 `Head` 컴포넌트를 제공합니다.

## 기본 사용법

```jsx
import Head from 'next/head'

function IndexPage() {
  return (
    <div>
      <Head>
        <title>My page title</title>
      </Head>
      <p>Hello world!</p>
    </div>
  )
}

export default IndexPage
```

## 중복 태그 방지

`key` 속성을 사용하여 태그가 한 번만 렌더링되도록 보장할 수 있습니다:

```jsx
import Head from 'next/head'

function IndexPage() {
  return (
    <div>
      <Head>
        <title>My page title</title>
        <meta property="og:title" content="My page title" key="title" />
      </Head>
      <Head>
        <meta property="og:title" content="My new title" key="title" />
      </Head>
      <p>Hello world!</p>
    </div>
  )
}

export default IndexPage
```

**결과**: 중복된 `key` 속성을 가진 메타 태그는 자동으로 처리되어, 두 번째 `<meta property="og:title" />` 태그만 렌더링됩니다.

> **참고**: `<title>`과 `<base>` 태그는 Next.js에서 자동으로 중복 검사되므로 `key`가 필수가 아닙니다.

## 중요 제약사항

### 컴포넌트 언마운트 시 초기화

Head의 내용은 컴포넌트가 언마운트될 때 초기화됩니다. 따라서 각 페이지에서 필요한 모든 요소를 명시적으로 정의해야 합니다. 다른 페이지에서 정의한 태그가 남아있다고 가정하지 마세요.

### 최소한의 중첩 구조 필요

`title`, `meta`, `script` 등의 요소는 `Head`의 **직접 자식**이어야 합니다. 최대 1단계의 `<React.Fragment>` 또는 배열로만 감싸질 수 있습니다. 그렇지 않으면 클라이언트 사이드 네비게이션에서 태그가 올바르게 감지되지 않습니다.

```jsx
// 올바른 사용법
<Head>
  <title>Page Title</title>
  <meta name="description" content="Page description" />
</Head>

// 올바른 사용법 - Fragment 사용
<Head>
  <>
    <title>Page Title</title>
    <meta name="description" content="Page description" />
  </>
</Head>

// 잘못된 사용법 - 중첩된 div
<Head>
  <div>
    <title>Page Title</title>
  </div>
</Head>
```

### 스크립트는 `next/script` 사용

`next/head` 내에서 직접 `<script>`를 작성하는 대신 [`next/script`](/docs/pages/api-reference/components/script) 컴포넌트 사용을 권장합니다.

```jsx
// 권장하지 않음
<Head>
  <script src="https://example.com/script.js" />
</Head>

// 권장
import Script from 'next/script'

<Script src="https://example.com/script.js" />
```

### `html` 또는 `body` 태그 불가

`Head`로 `<html>` 또는 `<body>` 태그의 속성을 설정할 수 없습니다. 이 경우 `next-head-count is missing` 에러가 발생합니다. `next/head`는 HTML `<head>` 태그 내부의 태그만 처리할 수 있습니다.

`<html>` 또는 `<body>` 태그를 커스터마이징하려면 [커스텀 `_document.js`](/docs/pages/api-reference/file-conventions/_document)를 사용하세요.

## 사용 예제

### 페이지별 메타 태그 설정

```jsx
import Head from 'next/head'

export default function About() {
  return (
    <>
      <Head>
        <title>About Us | My Website</title>
        <meta name="description" content="Learn more about our company" />
        <meta property="og:title" content="About Us" />
        <meta property="og:description" content="Learn more about our company" />
        <meta property="og:image" content="/images/about-og.png" />
        <link rel="canonical" href="https://example.com/about" />
      </Head>
      <main>
        <h1>About Us</h1>
        <p>Welcome to our about page.</p>
      </main>
    </>
  )
}
```

### 파비콘 및 아이콘 설정

```jsx
import Head from 'next/head'

export default function Home() {
  return (
    <>
      <Head>
        <link rel="icon" href="/favicon.ico" />
        <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
        <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
        <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
      </Head>
      <main>
        <h1>Home</h1>
      </main>
    </>
  )
}
```

### 동적 타이틀 설정

```jsx
import Head from 'next/head'

export default function Product({ product }) {
  return (
    <>
      <Head>
        <title>{product.name} | My Store</title>
        <meta name="description" content={product.description} />
      </Head>
      <main>
        <h1>{product.name}</h1>
        <p>{product.description}</p>
      </main>
    </>
  )
}

export async function getServerSideProps({ params }) {
  const product = await fetchProduct(params.id)
  return { props: { product } }
}
```

### 공통 Head 컴포넌트 만들기

여러 페이지에서 공통으로 사용하는 메타 태그가 있다면 재사용 가능한 컴포넌트를 만들 수 있습니다:

```jsx
// components/SEO.jsx
import Head from 'next/head'

export default function SEO({ title, description, image }) {
  const siteName = 'My Website'
  const fullTitle = title ? `${title} | ${siteName}` : siteName

  return (
    <Head>
      <title>{fullTitle}</title>
      <meta name="description" content={description} />
      <meta property="og:title" content={fullTitle} />
      <meta property="og:description" content={description} />
      {image && <meta property="og:image" content={image} />}
      <meta name="twitter:card" content="summary_large_image" />
    </Head>
  )
}
```

```jsx
// pages/about.jsx
import SEO from '../components/SEO'

export default function About() {
  return (
    <>
      <SEO
        title="About Us"
        description="Learn more about our company"
        image="/images/about-og.png"
      />
      <main>
        <h1>About Us</h1>
      </main>
    </>
  )
}
```
