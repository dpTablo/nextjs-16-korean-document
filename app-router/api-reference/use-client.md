---
원문: https://nextjs.org/docs/app/api-reference/directives/use-client
버전: 16.1.6
---

# 'use client' 지시어

## 개요
`'use client'` 지시어는 컴포넌트가 **클라이언트 사이드**에서 렌더링되도록 진입점을 선언합니다. 다음이 필요한 인터랙티브 UI를 만들 때 사용합니다:
- 상태 관리
- 이벤트 처리
- 브라우저 API 접근

## 주요 사항

> **중요:** 서버 컴포넌트 내에서 직접 렌더링하려는 컴포넌트가 있는 파일에만 `'use client'`를 추가하면 됩니다. 클라이언트 컴포넌트가 있는 모든 파일이 아니라 클라이언트-서버 경계를 정의하는 것입니다.

## 사용법

파일 **최상단**, import 전에 지시어를 추가하세요:

```tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  )
}
```

## Props 직렬화 요구사항

Props는 **직렬화 가능**해야 합니다 (React가 서버에서 클라이언트로 데이터를 전송할 수 있어야 함):

```tsx
// ❌ 유효하지 않음 - 함수는 직렬화할 수 없음
'use client'

export default function Counter({ onClick }) {
  return <button onClick={onClick}>증가</button>
}
```

## 서버 컴포넌트에 클라이언트 컴포넌트 중첩

**모범 사례 패턴:**

```tsx
// app/page.tsx (서버 컴포넌트)
import Header from './header'        // 서버 컴포넌트
import Counter from './counter'      // 클라이언트 컴포넌트

export default function Page() {
  return (
    <div>
      <Header />
      <Counter />
    </div>
  )
}
```

**전략:**
1. **서버 컴포넌트** → 정적 콘텐츠, 데이터 페칭, SEO
2. **클라이언트 컴포넌트** → 인터랙티브 요소, 상태, effects, 브라우저 API
3. **구성** → 필요에 따라 서버 컴포넌트 내에 클라이언트 컴포넌트를 중첩

## 참조
- [React 'use client' 문서](https://react.dev/reference/rsc/use-client)
