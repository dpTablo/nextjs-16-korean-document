# 서버 및 클라이언트 컴포넌트

**버전:** 16.1.1
**최종 업데이트:** 2025-12-19

## 개요

기본적으로 레이아웃과 페이지는 **서버 컴포넌트**로, 선택적 캐싱 및 클라이언트로의 스트리밍과 함께 데이터 페칭 및 서버 사이드 렌더링을 가능하게 합니다. **클라이언트 컴포넌트**는 필요할 때 인터랙티비티 및 브라우저 API 접근을 추가합니다.

---

## 서버 및 클라이언트 컴포넌트 사용 시기

### 다음이 필요한 경우 클라이언트 컴포넌트 사용:
- **상태 및 이벤트 핸들러** (예: `onClick`, `onChange`)
- **라이프사이클 로직** (예: `useEffect`)
- **브라우저 전용 API** (예: `localStorage`, `window`, `Navigator.geolocation`)
- **커스텀 훅**

### 다음이 필요한 경우 서버 컴포넌트 사용:
- 소스에 가까운 데이터베이스 또는 API에서 데이터 페칭
- 클라이언트 노출 없이 API 키, 토큰 및 시크릿 사용
- 브라우저로 전송되는 JavaScript 감소
- First Contentful Paint (FCP) 개선 및 콘텐츠 점진적 스트리밍

### 예시 패턴

**서버 컴포넌트 (app/[id]/page.tsx)**:
```tsx
import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post = await getPost(id)

  return (
    <div>
      <main>
        <h1>{post.title}</h1>
        <LikeButton likes={post.likes} />
      </main>
    </div>
  )
}
```

**클라이언트 컴포넌트 (app/ui/like-button.tsx)**:
```tsx
'use client'

import { useState } from 'react'

export default function LikeButton({ likes }: { likes: number }) {
  // ...
}
```

---

## Next.js에서 서버 및 클라이언트 컴포넌트의 작동 방식

### 서버에서
- **서버 컴포넌트**는 React 서버 컴포넌트 페이로드(RSC 페이로드)로 렌더링됨
- **클라이언트 컴포넌트** 및 RSC 페이로드는 HTML을 사전 렌더링하는 데 사용됨
- 렌더링 작업은 라우트 세그먼트(레이아웃 및 페이지)별로 분할됨

### React 서버 컴포넌트 페이로드 (RSC)
다음을 포함하는 컴팩트한 바이너리 표현:
- 서버 컴포넌트의 렌더링 결과
- JavaScript 파일 참조를 포함한 클라이언트 컴포넌트 위치의 플레이스홀더
- 서버에서 클라이언트 컴포넌트로 전달되는 Props

### 클라이언트에서 (첫 로드)
1. **HTML**은 빠른 비인터랙티브 미리보기를 표시
2. **RSC 페이로드**는 클라이언트 및 서버 컴포넌트 트리를 조정
3. **JavaScript**는 클라이언트 컴포넌트를 하이드레이트하여 인터랙티비티 추가

### 하이드레이션
정적 HTML에 이벤트 핸들러를 연결하여 인터랙티브하게 만드는 React의 프로세스.

### 후속 네비게이션
- **RSC 페이로드**는 즉각적인 네비게이션을 위해 프리페칭 및 캐싱됨
- **클라이언트 컴포넌트**는 서버 HTML 없이 전적으로 클라이언트에서 렌더링됨

---

## 예제

### 클라이언트 컴포넌트 사용하기

```tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count} 좋아요</p>
      <button onClick={() => setCount(count + 1)}>클릭하세요</button>
    </div>
  )
}
```

`"use client"` 지시어는 서버와 클라이언트 모듈 그래프 사이의 경계를 선언합니다. **모든 임포트와 자식 컴포넌트는 클라이언트 번들의 일부가 됩니다**.

### JS 번들 크기 줄이기

큰 UI 섹션 대신 특정 인터랙티브 컴포넌트만 `'use client'`로 표시하세요.

```tsx
// app/layout.tsx
import Search from './search'  // 클라이언트 컴포넌트
import Logo from './logo'       // 서버 컴포넌트

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <Search />
      </nav>
      <main>{children}</main>
    </>
  )
}
```

```tsx
// app/ui/search.tsx
'use client'

export default function Search() {
  // ...
}
```

### 서버에서 클라이언트 컴포넌트로 데이터 전달하기

Props를 통해 데이터를 전달하세요:

```tsx
// 서버 컴포넌트
import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const post = await getPost(id)

  return <LikeButton likes={post.likes} />
}
```

**참고**: Props는 React에 의해 [직렬화 가능](https://react.dev/reference/react/use-server#serializable-parameters-and-return-values)해야 합니다. 또는 [`use` Hook](https://react.dev/reference/react/use)을 사용하여 데이터를 스트리밍하세요.

### 서버 및 클라이언트 컴포넌트 인터리빙

`children` 패턴을 사용하여 서버 컴포넌트를 클라이언트 컴포넌트에 props로 전달하세요:

```tsx
// app/ui/modal.tsx
'use client'

export default function Modal({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
```

```tsx
// app/page.tsx (서버 컴포넌트)
import Modal from './ui/modal'
import Cart from './ui/cart'

export default function Page() {
  return (
    <Modal>
      <Cart />
    </Modal>
  )
}
```

서버 컴포넌트는 서버에서 먼저 렌더링되며, RSC 페이로드는 클라이언트 컴포넌트 렌더링 위치에 대한 참조를 포함합니다.

### Context Providers

React context는 서버 컴포넌트에서 지원되지 않습니다. 클라이언트 컴포넌트 provider를 생성하세요:

```tsx
// app/theme-provider.tsx
'use client'

import { createContext } from 'react'

export const ThemeContext = createContext({})

export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode
}) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
```

```tsx
// app/layout.tsx
import ThemeProvider from './theme-provider'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  )
}
```

**모범 사례**: 정적 서버 컴포넌트 부분을 최적화하기 위해 트리의 깊은 곳에서 providers를 렌더링하세요.

### 서드파티 컴포넌트

클라이언트 전용 기능에 의존하는 서드파티 컴포넌트를 래핑하세요:

```tsx
// app/carousel.tsx
'use client'

import { Carousel } from 'acme-carousel'

export default Carousel
```

```tsx
// app/page.tsx
import Carousel from './carousel'

export default function Page() {
  return (
    <div>
      <p>사진 보기</p>
      <Carousel />
    </div>
  )
}
```

**라이브러리 작성자를 위한 조언**: 클라이언트 전용 기능을 사용하는 진입점에 `"use client"`를 추가하여 직접적인 서버 컴포넌트 임포트를 가능하게 하세요.

### 환경 오염 방지

[`server-only`](https://www.npmjs.com/package/server-only) 패키지를 사용하여 클라이언트 컴포넌트로 서버 전용 코드가 실수로 임포트되는 것을 방지하세요:

```js
// lib/data.js
import 'server-only'

export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })

  return res.json()
}
```

이는 명확한 오류와 함께 빌드 시점에 클라이언트 컴포넌트 임포트를 방지합니다.

**설치**:
```bash
npm install server-only
```

클라이언트 전용 로직(예: `window` 객체 접근)에는 [`client-only`](https://www.npmjs.com/package/client-only) 패키지를 사용하세요.

**참고**: Next.js는 더 명확한 오류 메시지를 위해 이러한 패키지를 내부적으로 처리합니다. 실제 NPM 패키지 내용은 Next.js에서 사용되지 않습니다.

---

## 핵심 요점

- 서버 컴포넌트가 기본값이며, 인터랙티비티가 필요한 경우에만 클라이언트 컴포넌트를 사용하세요
- 클라이언트 컴포넌트에는 모듈 레벨에서 `"use client"` 지시어를 사용하세요
- 서버에서 클라이언트로 데이터를 props(직렬화 가능)를 통해 전달하세요
- `children` 패턴을 사용하여 컴포넌트를 인터리빙하세요
- 최적화를 위해 트리의 깊은 곳에서 providers를 사용하세요
- `server-only` 패키지로 서버 시크릿을 보호하세요
- 번들 크기를 줄이기 위해 클라이언트 컴포넌트 범위를 최소화하세요
