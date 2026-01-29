---
원문: https://nextjs.org/docs/app/api-reference/functions/use-selected-layout-segments
버전: 16.1.6
---

# useSelectedLayoutSegments

## 개요

`useSelectedLayoutSegments`는 호출된 Layout의 **한 단계 아래** 활성 라우트 세그먼트들을 읽는 **Client Component hook**입니다. 활성 자식 세그먼트들을 알아야 하는 부모 Layout UI(예: 브레드크럼)를 만드는 데 유용합니다.

---

## 기본 사용법

```tsx
'use client'

import { useSelectedLayoutSegments } from 'next/navigation'

export default function ExampleClientComponent() {
  const segments = useSelectedLayoutSegments()

  return (
    <ul>
      {segments.map((segment, index) => (
        <li key={index}>{segment}</li>
      ))}
    </ul>
  )
}
```

---

## 매개변수

```tsx
const segments = useSelectedLayoutSegments(parallelRoutesKey?: string)
```

- **`parallelRoutesKey`** (선택사항): 특정 parallel route 슬롯 내에서 활성 라우트 세그먼트들을 읽을 수 있습니다.

---

## 반환 값

호출된 Layout에서 한 단계 아래의 활성 세그먼트들을 포함하는 **문자열 배열**을 반환합니다.

### 반환 예제

| Layout | URL | 반환 세그먼트 |
|--------|-----|--------------|
| `app/layout.js` | `/` | `[]` |
| `app/layout.js` | `/dashboard` | `['dashboard']` |
| `app/layout.js` | `/dashboard/settings` | `['dashboard', 'settings']` |
| `app/dashboard/layout.js` | `/dashboard` | `[]` |
| `app/dashboard/layout.js` | `/dashboard/settings` | `['settings']` |

---

## 주요 포인트

### 1. Client Component Hook
- **Client Component hook**이므로 일반적으로 Layout(기본적으로 Server Component)에 import됩니다.

### 2. Route Groups 처리
- 반환된 세그먼트에는 **Route Groups**가 포함됩니다.
- Route Groups를 제외하려면 대괄호로 시작하는 항목을 필터링하세요:

```tsx
const segments = useSelectedLayoutSegments()
  .filter(segment => !segment.startsWith('['))
```

---

## 실용적인 예제

### 브레드크럼 네비게이션

```tsx
'use client'

import { useSelectedLayoutSegments } from 'next/navigation'
import Link from 'next/link'

export default function Breadcrumbs() {
  const segments = useSelectedLayoutSegments()

  return (
    <nav>
      <Link href="/">Home</Link>
      {segments.map((segment, index) => {
        const href = `/${segments.slice(0, index + 1).join('/')}`
        return (
          <span key={segment}>
            {' > '}
            <Link href={href}>{segment}</Link>
          </span>
        )
      })}
    </nav>
  )
}
```

### Route Groups 제외하기

```tsx
'use client'

import { useSelectedLayoutSegments } from 'next/navigation'

export default function FilteredSegments() {
  const segments = useSelectedLayoutSegments()
    // [marketing], [shop] 같은 Route Groups 제외
    .filter(segment => !segment.startsWith('['))

  return (
    <div>
      <h2>활성 경로 (Route Groups 제외):</h2>
      <p>{segments.join(' / ')}</p>
    </div>
  )
}
```

---

## useSelectedLayoutSegment와 비교

| Hook | 반환 타입 | 반환 값 |
|------|----------|---------|
| `useSelectedLayoutSegment` | `string \| null` | 한 단계 아래의 **단일** 세그먼트 |
| `useSelectedLayoutSegments` | `string[]` | 한 단계 아래의 **모든** 세그먼트 배열 |

### 사용 시나리오
- **`useSelectedLayoutSegment`**: 탭, 활성 링크 강조 등 단일 자식 세그먼트 확인
- **`useSelectedLayoutSegments`**: 브레드크럼, 전체 경로 표시 등 여러 세그먼트 필요

---

## 주의사항

1. **Client Component에서만 사용 가능**
   - Server Component에서는 작동하지 않습니다.
   - 컴포넌트 상단에 `'use client'` 지시어가 필요합니다.

2. **상대적 세그먼트**
   - 호출된 Layout을 기준으로 **한 단계 아래** 세그먼트만 반환합니다.
   - 전체 URL 경로를 가져오려면 `usePathname()`을 사용하세요.

3. **Route Groups 주의**
   - 반환값에 Route Groups (`[folder]`)가 포함됩니다.
   - 필요시 `.filter()`로 제거하세요.

---

## 관련 함수

- [`useSelectedLayoutSegment()`](./useSelectedLayoutSegment.md) - 단일 활성 세그먼트 가져오기
- [`usePathname()`](./usePathname.md) - 전체 현재 경로 가져오기
- [`useParams()`](./useParams.md) - 동적 라우트 파라미터 가져오기

---

## 버전 정보

- **도입 버전:** Next.js 13.0.0
- **안정 버전:** Next.js 13.4.0+
