# CSS-in-JS

Next.js에서 CSS-in-JS 라이브러리를 사용하는 방법을 알아봅니다.

## 개요

이 가이드는 CSS-in-JS 라이브러리를 Next.js의 `app` 디렉토리와 클라이언트 컴포넌트에 통합하는 방법을 다룹니다.

> **중요:** 라이브러리는 React 18+의 동시 렌더링 및 Server Components를 포함한 기능을 지원해야 합니다.

---

## 지원되는 라이브러리

### 완전 지원 (알파벳 순)

| 라이브러리 | 설명 |
|------------|------|
| `ant-design` | Ant Design 컴포넌트 라이브러리 |
| `chakra-ui` | 간단하고 모듈식 접근성 있는 컴포넌트 라이브러리 |
| `@fluentui/react-components` | Microsoft Fluent UI |
| `kuma-ui` | Zero-runtime CSS-in-JS |
| `@mui/material` & `@mui/joy` | Material-UI |
| `pandacss` | CSS-in-JS with build-time optimization |
| `styled-jsx` | Next.js 기본 제공 |
| `styled-components` | 가장 인기 있는 CSS-in-JS 라이브러리 |
| `stylex` | Facebook의 CSS-in-JS |
| `tamagui` | Universal UI kit |
| `tss-react` | TypeScript Style Sheets |
| `vanilla-extract` | Zero-runtime Stylesheets-in-TypeScript |

### 개발 중

- **`emotion`** - [GitHub 이슈](https://github.com/emotion-js/emotion/issues/2928) 참조

---

## 구성 프로세스

CSS-in-JS 설정은 세 단계로 이루어집니다:

1. **스타일 레지스트리** - 렌더링 중 CSS 규칙 수집
2. **useServerInsertedHTML Hook** - 해당 콘텐츠보다 먼저 규칙 주입
3. **클라이언트 컴포넌트 래퍼** - 서버 사이드 렌더링 중 스타일 레지스트리로 앱 래핑

---

## 구현 예시

### styled-jsx (v5.1.0+)

Next.js에 기본 제공되는 CSS-in-JS 솔루션입니다.

#### 1. 레지스트리 생성

**app/registry.tsx**
```tsx
'use client'

import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { StyleRegistry, createStyleRegistry } from 'styled-jsx'

export default function StyledJsxRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  // 지연 초기 상태 - 한 번만 생성
  const [jsxStyleRegistry] = useState(() => createStyleRegistry())

  useServerInsertedHTML(() => {
    const styles = jsxStyleRegistry.styles()
    jsxStyleRegistry.flush()
    return <>{styles}</>
  })

  return <StyleRegistry registry={jsxStyleRegistry}>{children}</StyleRegistry>
}
```

#### 2. 루트 레이아웃에 래퍼 추가

**app/layout.tsx**
```tsx
import StyledJsxRegistry from './registry'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <StyledJsxRegistry>{children}</StyledJsxRegistry>
      </body>
    </html>
  )
}
```

#### 3. 사용 예시

**app/page.tsx**
```tsx
export default function Page() {
  return (
    <div>
      <h1 className="title">Hello World</h1>
      <style jsx>{`
        .title {
          color: blue;
          font-size: 2rem;
        }
      `}</style>
    </div>
  )
}
```

---

### Styled-Components (v6+)

가장 인기 있는 CSS-in-JS 라이브러리입니다.

#### 1. 설치

```bash
npm install styled-components
```

#### 2. Next.js 구성 활성화

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  compiler: {
    styledComponents: true,
  },
}

module.exports = nextConfig
```

#### 3. 레지스트리 생성

**lib/registry.tsx**
```tsx
'use client'

import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'

export default function StyledComponentsRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  // 스타일 시트는 한 번만 생성
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet())

  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement()
    styledComponentsStyleSheet.instance.clearTag()
    return <>{styles}</>
  })

  if (typeof window !== 'undefined') return <>{children}</>

  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  )
}
```

#### 4. 루트 레이아웃에 래퍼 추가

**app/layout.tsx**
```tsx
import StyledComponentsRegistry from './lib/registry'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  )
}
```

#### 5. 사용 예시

**app/page.tsx**
```tsx
'use client'

import styled from 'styled-components'

const Title = styled.h1`
  color: #0070f3;
  font-size: 2rem;
`

const Button = styled.button`
  background: #0070f3;
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background: #0051cc;
  }
`

export default function Page() {
  return (
    <div>
      <Title>Styled Components</Title>
      <Button>Click me</Button>
    </div>
  )
}
```

---

### Material-UI (v5+)

인기 있는 React UI 프레임워크입니다.

#### 1. 설치

```bash
npm install @mui/material @mui/styled-engine-sc @emotion/react @emotion/styled
```

#### 2. 레지스트리 생성

**lib/mui-registry.tsx**
```tsx
'use client'

import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { CacheProvider } from '@emotion/react'
import createCache from '@emotion/cache'

export default function MuiRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  const [cache] = useState(() => {
    const cache = createCache({ key: 'css' })
    cache.compat = true
    return cache
  })

  useServerInsertedHTML(() => {
    return (
      <style
        data-emotion={`${cache.key} ${Object.keys(cache.inserted).join(' ')}`}
        dangerouslySetInnerHTML={{
          __html: Object.values(cache.inserted).join(' '),
        }}
      />
    )
  })

  return <CacheProvider value={cache}>{children}</CacheProvider>
}
```

#### 3. 루트 레이아웃에 래퍼 추가

**app/layout.tsx**
```tsx
import MuiRegistry from './lib/mui-registry'
import { ThemeProvider, createTheme } from '@mui/material/styles'
import CssBaseline from '@mui/material/CssBaseline'

const theme = createTheme({
  palette: {
    primary: {
      main: '#0070f3',
    },
  },
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <MuiRegistry>
          <ThemeProvider theme={theme}>
            <CssBaseline />
            {children}
          </ThemeProvider>
        </MuiRegistry>
      </body>
    </html>
  )
}
```

#### 4. 사용 예시

**app/page.tsx**
```tsx
'use client'

import { Button, Typography } from '@mui/material'

export default function Page() {
  return (
    <div>
      <Typography variant="h1" color="primary">
        Material-UI
      </Typography>
      <Button variant="contained">Click me</Button>
    </div>
  )
}
```

---

### Chakra UI (v2+)

간단하고 모듈식 접근성 있는 컴포넌트 라이브러리입니다.

#### 1. 설치

```bash
npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

#### 2. Provider 생성

**app/providers.tsx**
```tsx
'use client'

import { CacheProvider } from '@chakra-ui/next-js'
import { ChakraProvider } from '@chakra-ui/react'

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <CacheProvider>
      <ChakraProvider>{children}</ChakraProvider>
    </CacheProvider>
  )
}
```

#### 3. 루트 레이아웃에 Provider 추가

**app/layout.tsx**
```tsx
import { Providers } from './providers'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

#### 4. 사용 예시

**app/page.tsx**
```tsx
'use client'

import { Button, Heading } from '@chakra-ui/react'

export default function Page() {
  return (
    <div>
      <Heading>Chakra UI</Heading>
      <Button colorScheme="blue">Click me</Button>
    </div>
  )
}
```

---

## 작동 방식

### 서버 렌더링 플로우

```
1. 컴포넌트 렌더링
    ↓
2. 스타일 레지스트리가 CSS 수집
    ↓
3. useServerInsertedHTML로 <head>에 주입
    ↓
4. HTML 전송
    ↓
5. 클라이언트에서 하이드레이션
    ↓
6. CSS-in-JS 라이브러리 인계
```

### 스트리밍과 CSS

- 각 청크에서 스타일이 수집됩니다
- 스타일은 해당 콘텐츠보다 먼저 추가됩니다
- 하이드레이션 후 styled-components가 인계합니다

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **지연 초기 상태 사용**
   ```tsx
   const [registry] = useState(() => createStyleRegistry())
   ```

2. **최상위 레벨에 클라이언트 컴포넌트**
   - 더 효율적인 CSS 추출
   - 후속 렌더링에서 스타일 재생성 방지

3. **서버와 클라이언트 분리**
   ```tsx
   if (typeof window !== 'undefined') return <>{children}</>
   ```

4. **플러시 확인**
   ```tsx
   useServerInsertedHTML(() => {
     const styles = registry.styles()
     registry.flush() // ← 중요!
     return <>{styles}</>
   })
   ```

### ❌ 피해야 할 것

1. **매 렌더링마다 레지스트리 생성**
   ```tsx
   // ❌ 나쁜 예
   const registry = createStyleRegistry()

   // ✅ 좋은 예
   const [registry] = useState(() => createStyleRegistry())
   ```

2. **플러시 누락**
   - 스타일 중복 발생
   - 메모리 누수 가능

3. **클라이언트에서 레지스트리 사용**
   - 불필요한 오버헤드
   - 브라우저에서는 네이티브 CSS 사용

---

## TypeScript 타입

### Styled-Components

**styled.d.ts**
```ts
import 'styled-components'

declare module 'styled-components' {
  export interface DefaultTheme {
    colors: {
      primary: string
      secondary: string
    }
    spacing: {
      small: string
      medium: string
      large: string
    }
  }
}
```

**사용:**
```tsx
import styled from 'styled-components'

const Button = styled.button`
  background: ${({ theme }) => theme.colors.primary};
  padding: ${({ theme }) => theme.spacing.medium};
`
```

---

## 성능 최적화

### 1. 코드 스플리팅

```tsx
'use client'

import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
})
```

### 2. 조건부 스타일

```tsx
import styled, { css } from 'styled-components'

const Button = styled.button<{ $variant?: 'primary' | 'secondary' }>`
  ${({ $variant }) =>
    $variant === 'primary' &&
    css`
      background: blue;
      color: white;
    `}
`
```

### 3. 스타일 메모이제이션

```tsx
import { useMemo } from 'react'
import styled from 'styled-components'

function Component({ color }: { color: string }) {
  const StyledDiv = useMemo(
    () => styled.div`
      color: ${color};
    `,
    [color]
  )

  return <StyledDiv>Content</StyledDiv>
}
```

---

## 문제 해결

### 스타일이 적용되지 않음

**1. 레지스트리 확인:**
- `useState(() => ...)` 사용 확인
- `flush()` 호출 확인

**2. 클라이언트 컴포넌트 확인:**
- 레지스트리 파일에 `'use client'` 지시어 있는지 확인

**3. Next.js 구성 확인:**
```js
// styled-components의 경우
module.exports = {
  compiler: {
    styledComponents: true,
  },
}
```

### FOUC (Flash of Unstyled Content)

**해결책:**
```tsx
useServerInsertedHTML(() => {
  const styles = registry.styles()
  registry.flush()
  return <>{styles}</> // 콘텐츠보다 먼저 주입됨
})
```

---

## 다음 단계

- [Tailwind CSS](./tailwind-css.md) - Tailwind CSS 가이드
- [Sass](./sass.md) - Sass 사용 가이드
- [CSS Modules](https://nextjs.org/docs/app/building-your-application/styling/css-modules)

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11

**참고 자료:**
- [styled-jsx 예시](https://github.com/vercel/next.js/tree/canary/examples/with-styled-jsx)
- [styled-components 예시](https://github.com/vercel/next.js/tree/canary/examples/with-styled-components)
- [Next.js Compiler](https://nextjs.org/docs/architecture/nextjs-compiler#styled-components)
