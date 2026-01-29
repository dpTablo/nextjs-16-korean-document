---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/template
버전: 16.1.6
---

# template.js

## 개요
**template** 파일은 layout과 유사하지만 layout 또는 page를 감싸는 방식에서 주요 차이점이 있습니다:
- **Layouts**: 라우트 간에 지속되고 상태를 유지합니다
- **Templates**: 고유한 키가 부여되어 네비게이션 시 하위 클라이언트 컴포넌트의 상태가 재설정됩니다

## Template 사용 시기

다음과 같은 경우 Template이 유용합니다:
- 네비게이션 시 `useEffect`를 재동기화해야 할 때
- 네비게이션 시 하위 클라이언트 컴포넌트의 상태를 재설정해야 할 때 (예: 입력 필드)
- 기본 프레임워크 동작을 변경해야 할 때 (예: Suspense 경계가 첫 로드뿐만 아니라 모든 네비게이션에서 fallback을 표시)

## 규칙

`template.js` 파일에서 기본 React 컴포넌트를 내보내 template를 정의합니다:

```tsx
// app/template.tsx
export default function Template({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
```

```jsx
// app/template.js
export default function Template({ children }) {
  return <div>{children}</div>
}
```

**중첩 구조:**
```jsx
<Layout>
  {/* Template에 고유한 키가 부여됨 */}
  <Template key={routeParam}>{children}</Template>
</Layout>
```

## Props

### `children` (필수)
Template는 중첩된 page 또는 layout 콘텐츠를 포함하는 `children` prop을 받습니다.

## 주요 동작

| 동작 | 세부사항 |
|----------|---------|
| **Server Components** | Template는 기본적으로 Server Component입니다 |
| **네비게이션** | Template는 세그먼트당 고유한 키를 받으며, 세그먼트 또는 동적 파라미터가 변경되면 다시 마운트됩니다 |
| **상태 재설정** | 내부의 모든 클라이언트 컴포넌트는 네비게이션 시 상태가 재설정됩니다 |
| **Effects** | 컴포넌트가 다시 마운트되면서 `useEffect`가 재동기화됩니다 |
| **DOM** | DOM 요소가 완전히 재생성됩니다 |
| **Search Params** | 재마운트를 트리거하지 않습니다 |

## 네비게이션 예제

다음 프로젝트 구조를 가정합니다:
```
app
├── about/page.tsx
├── blog/
│   ├── [slug]/page.tsx
│   ├── page.tsx
│   └── template.tsx
├── layout.tsx
├── page.tsx
└── template.tsx
```

**`/blog/first-post`로 네비게이션:** blog 레벨 template만 다시 마운트됩니다 (키 변경), 루트 template는 유지됩니다 (동일한 첫 번째 세그먼트).

**`/blog/second-post`로 네비게이션:** blog 레벨 template가 다시 마운트됩니다 (하위 세그먼트 변경), 루트 template는 안정적으로 유지됩니다.

## Layout vs Template 비교

| 특성 | Layout | Template |
|------|--------|----------|
| **상태 유지** | ✅ 네비게이션 간 유지 | ❌ 매번 재설정 |
| **재마운트** | ❌ 재마운트되지 않음 | ✅ 매 네비게이션마다 재마운트 |
| **사용 사례** | 공통 UI, 상태 유지가 필요한 경우 | 상태 재설정이 필요한 경우 |

## 예제: 입력 필드 재설정

```tsx
// app/search/template.tsx
'use client'

import { useState } from 'react'

export default function SearchTemplate({ children }: { children: React.ReactNode }) {
  const [query, setQuery] = useState('')

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="검색..."
      />
      {children}
    </div>
  )
}
```

위 예제에서 `/search/products`에서 `/search/users`로 네비게이션하면 검색 입력 필드가 자동으로 재설정됩니다.

## 버전 히스토리

| 버전   | 변경사항 |
|-----------|---------|
| `v13.0.0` | `template` 도입 |

## 관련 문서

- [layout.js](./layout.md)
- [레이아웃과 페이지](../../getting-started/03-layouts-and-pages.md)
