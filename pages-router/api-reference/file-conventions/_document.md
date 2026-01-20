# Custom Document (_document)

Custom `Document`는 모든 페이지를 렌더링하는 `<html>`과 `<body>` 태그를 커스터마이징합니다. 서버에서만 렌더링되며, 모든 페이지에 공통적으로 적용되는 HTML 구조를 정의합니다.

---

## 기본 사용법

`pages/_document.tsx` (또는 `pages/_document.js`) 파일을 생성합니다.

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="ko">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

```jsx
// pages/_document.js
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="ko">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

---

## 필수 컴포넌트

| 컴포넌트 | 설명 |
|----------|------|
| `<Html>` | HTML 문서의 루트 요소 |
| `<Head />` | 모든 페이지에 공통인 `<head>` 내용 |
| `<Main />` | 페이지 컴포넌트가 렌더링되는 위치 |
| `<NextScript />` | Next.js가 필요로 하는 스크립트 |

> **주의**: 이 네 가지 컴포넌트는 페이지가 올바르게 렌더링되기 위해 필수입니다.

---

## 사용 사례

### 언어 설정

```tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="ko">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

### 글로벌 메타 태그

```tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
      <Head>
        <meta charSet="utf-8" />
        <meta name="theme-color" content="#000000" />
        <link rel="icon" href="/favicon.ico" />
        <link
          href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR&display=swap"
          rel="stylesheet"
        />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

### body 클래스 추가

```tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
      <Head />
      <body className="antialiased">
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

### 다크 모드 깜빡임 방지

```tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
      <Head />
      <body>
        {/* 페이지 로드 전 테마 적용 스크립트 */}
        <script
          dangerouslySetInnerHTML={{
            __html: `
              (function() {
                const theme = localStorage.getItem('theme') || 'light';
                document.documentElement.classList.add(theme);
              })();
            `,
          }}
        />
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

### 포털용 컨테이너

```tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
      <Head />
      <body>
        <Main />
        <div id="modal-root" />
        <div id="tooltip-root" />
        <NextScript />
      </body>
    </Html>
  )
}
```

---

## 주의사항

### Head 컴포넌트 차이

`_document`의 `<Head />`와 `next/head`의 `<Head>`는 다릅니다:

| 위치 | 용도 |
|------|------|
| `_document`의 `<Head />` | 모든 페이지에 공통인 정적 요소 |
| `next/head`의 `<Head>` | 페이지별 동적 `<head>` 요소 (예: `<title>`) |

```tsx
// pages/about.tsx
import Head from 'next/head'

export default function AboutPage() {
  return (
    <>
      <Head>
        <title>소개 - 내 사이트</title>
        <meta name="description" content="소개 페이지입니다" />
      </Head>
      <main>
        <h1>소개</h1>
      </main>
    </>
  )
}
```

### Main 외부 컴포넌트

`<Main />` 외부에 배치된 React 컴포넌트는:

- 브라우저에서 초기화되지 않습니다
- 이벤트 핸들러가 작동하지 않습니다
- `styled-jsx` 등 CSS-in-JS가 작동하지 않습니다

공유 컴포넌트가 필요하면 `_app.js`의 Layout 패턴을 사용하세요.

### 이벤트 핸들러 미지원

`_document`는 서버에서만 렌더링되므로:

```tsx
// ❌ 작동하지 않음
<body onClick={() => console.log('clicked')}>
```

---

## 고급: renderPage 커스터마이징

CSS-in-JS 라이브러리 등 특수한 경우에만 사용합니다.

```tsx
import Document, {
  Html,
  Head,
  Main,
  NextScript,
  DocumentContext,
  DocumentInitialProps,
} from 'next/document'

class MyDocument extends Document {
  static async getInitialProps(
    ctx: DocumentContext
  ): Promise<DocumentInitialProps> {
    const originalRenderPage = ctx.renderPage

    // React 트리를 동기적으로 렌더링
    ctx.renderPage = () =>
      originalRenderPage({
        // 전체 React 트리 래핑
        enhanceApp: (App) => App,
        // 페이지별 컴포넌트 래핑
        enhanceComponent: (Component) => Component,
      })

    // 기본 Document getInitialProps 실행
    const initialProps = await Document.getInitialProps(ctx)

    return initialProps
  }

  render() {
    return (
      <Html lang="ko">
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

export default MyDocument
```

### styled-components 예제

```tsx
import Document, { DocumentContext, DocumentInitialProps } from 'next/document'
import { ServerStyleSheet } from 'styled-components'

class MyDocument extends Document {
  static async getInitialProps(
    ctx: DocumentContext
  ): Promise<DocumentInitialProps> {
    const sheet = new ServerStyleSheet()
    const originalRenderPage = ctx.renderPage

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: (App) => (props) =>
            sheet.collectStyles(<App {...props} />),
        })

      const initialProps = await Document.getInitialProps(ctx)
      return {
        ...initialProps,
        styles: (
          <>
            {initialProps.styles}
            {sheet.getStyleElement()}
          </>
        ),
      }
    } finally {
      sheet.seal()
    }
  }
}

export default MyDocument
```

---

## 제한사항

- `getStaticProps` 또는 `getServerSideProps`를 지원하지 않습니다
- 클라이언트 측 이벤트 핸들러를 사용할 수 없습니다
- `<Main />` 외부의 React 컴포넌트는 브라우저에서 초기화되지 않습니다

---

## 중요한 주의사항

> **Good to know**:
> - `_document`는 서버에서만 렌더링됩니다
> - `<title>` 태그는 페이지 컴포넌트에서 `next/head`를 사용하세요
> - `getInitialProps`는 클라이언트 측 네비게이션에서 호출되지 않습니다

> **App Router 마이그레이션**:
> - App Router에서는 `app/layout.tsx`가 `_document`의 기능을 포함합니다
> - `<html>`, `<body>` 태그를 Root Layout에서 직접 정의합니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.4.0 | App Router 안정화 |
| v9.0.0 | TypeScript 지원 |

---

## 관련 문서

- [Custom App (_app)](./_app.md)
- [App Router - Layout](../../../app-router/api-reference/file-conventions/layout.md)
- [CSS-in-JS](../../../app-router/guides/css-in-js.md)
