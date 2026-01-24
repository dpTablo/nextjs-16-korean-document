# CSS-in-JS

Next.js에서 다양한 CSS-in-JS 라이브러리를 사용할 수 있습니다. JavaScript 코드 내에서 직접 CSS를 작성할 수 있어 컴포넌트와 스타일을 함께 관리할 수 있습니다.

## 지원되는 라이브러리

- styled-jsx (기본 내장)
- styled-components
- Emotion
- Linaria
- Styletron
- Cxs
- Fela
- Stitches

## 인라인 스타일

가장 간단한 방법은 React의 인라인 스타일을 사용하는 것입니다:

```jsx
function HiThere() {
  return <p style={{ color: 'red' }}>안녕하세요</p>
}

export default HiThere
```

## styled-jsx

Next.js는 **styled-jsx**를 기본으로 번들하여 별도의 설치 없이 사용할 수 있습니다. styled-jsx는 Web Components의 "Shadow CSS"와 유사한 격리된 스코프 CSS를 제공합니다.

### 기본 사용법

```jsx
function HelloWorld() {
  return (
    <div>
      Hello world
      <p>스코프가 적용됩니다!</p>
      <style jsx>{`
        p {
          color: blue;
        }
        div {
          background: red;
        }
        @media (max-width: 600px) {
          div {
            background: blue;
          }
        }
      `}</style>
    </div>
  )
}

export default HelloWorld
```

### 전역 스타일

`global` 속성을 추가하여 전역 스타일을 정의할 수 있습니다:

```jsx
function GlobalStyles() {
  return (
    <div>
      <style jsx global>{`
        body {
          background: black;
          margin: 0;
          padding: 0;
        }

        * {
          box-sizing: border-box;
        }
      `}</style>
    </div>
  )
}
```

### 동적 스타일

JavaScript 변수를 사용하여 동적 스타일을 적용할 수 있습니다:

```jsx
function Button({ color }) {
  return (
    <button>
      클릭하세요
      <style jsx>{`
        button {
          background-color: ${color};
          padding: 10px 20px;
          border: none;
          border-radius: 4px;
          color: white;
          cursor: pointer;
        }
        button:hover {
          opacity: 0.8;
        }
      `}</style>
    </button>
  )
}

export default function Page() {
  return (
    <div>
      <Button color="blue" />
      <Button color="green" />
    </div>
  )
}
```

## styled-components

styled-components를 사용하려면 추가 설정이 필요합니다.

### 설치

```bash
npm install styled-components
npm install -D babel-plugin-styled-components
```

### 설정

```js
// next.config.js
module.exports = {
  compiler: {
    styledComponents: true,
  },
}
```

### _document.js 설정 (SSR 지원)

```jsx
// pages/_document.js
import Document from 'next/document'
import { ServerStyleSheet } from 'styled-components'

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
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
```

### 사용 예제

```jsx
import styled from 'styled-components'

const Container = styled.div`
  padding: 20px;
  background-color: #f0f0f0;
`

const Title = styled.h1`
  color: #333;
  font-size: 2rem;
`

const Button = styled.button`
  background-color: ${(props) => (props.primary ? 'blue' : 'gray')};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    opacity: 0.8;
  }
`

export default function Page() {
  return (
    <Container>
      <Title>styled-components 예제</Title>
      <Button primary>Primary 버튼</Button>
      <Button>Secondary 버튼</Button>
    </Container>
  )
}
```

## Emotion

Emotion도 널리 사용되는 CSS-in-JS 라이브러리입니다.

### 설치

```bash
npm install @emotion/react @emotion/styled
```

### 설정

```js
// next.config.js
module.exports = {
  compiler: {
    emotion: true,
  },
}
```

### 사용 예제

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react'
import styled from '@emotion/styled'

const buttonStyle = css`
  background-color: hotpink;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  color: white;
  cursor: pointer;
`

const Container = styled.div`
  padding: 20px;
`

export default function Page() {
  return (
    <Container>
      <button css={buttonStyle}>Emotion 버튼</button>
    </Container>
  )
}
```

## JavaScript 비활성화 시

- **프로덕션 빌드** (`next start`): CSS가 정상적으로 로드됩니다.
- **개발 모드**: Fast Refresh 기능을 위해 JavaScript가 필요합니다.

## 장단점

### 장점

1. **컴포넌트 단위 스타일**: 스타일이 컴포넌트와 함께 있어 관리가 쉽습니다.
2. **동적 스타일링**: props나 상태에 따라 스타일을 동적으로 변경할 수 있습니다.
3. **자동 벤더 프리픽스**: 대부분의 라이브러리가 자동으로 처리합니다.
4. **타입 안전성**: TypeScript와 함께 사용하면 타입 지원을 받을 수 있습니다.

### 단점

1. **런타임 오버헤드**: JavaScript에서 CSS를 생성하므로 약간의 성능 비용이 있습니다.
2. **번들 크기 증가**: 라이브러리 자체의 크기가 추가됩니다.
3. **SSR 설정 필요**: 서버 사이드 렌더링을 위한 추가 설정이 필요할 수 있습니다.

## 권장 사항

- 간단한 프로젝트나 빠른 프로토타이핑에는 **styled-jsx** (기본 내장)를 사용하세요.
- 대규모 프로젝트에서는 **styled-components** 또는 **Emotion**을 고려하세요.
- 성능이 중요한 경우 **Tailwind CSS** 또는 **CSS Modules**를 우선 고려하세요.
