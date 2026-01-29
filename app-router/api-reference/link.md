---
원문: https://nextjs.org/docs/app/api-reference/components/link
버전: 16.1.6
---

# Link 컴포넌트 API 참조

## 개요
`<Link>`는 HTML `<a>` 요소를 확장하여 Next.js에서 라우트 간 프리페칭 및 클라이언트 사이드 네비게이션을 제공하는 React 컴포넌트입니다.

## Props 참조

| Prop | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| `href` | String or Object | Yes | - | 네비게이트할 경로 또는 URL |
| `replace` | Boolean | No | `false` | 새 히스토리 항목 추가 대신 현재 항목 교체 |
| `scroll` | Boolean | No | `true` | 네비게이션 시 스크롤 위치 유지 |
| `prefetch` | Boolean or null | No | `"auto"` | 프리페칭 동작 제어 |
| `onNavigate` | Function | No | - | 클라이언트 사이드 네비게이션 이벤트 핸들러 |

**추가:** 표준 `<a>` 태그 속성 지원 (예: `className`, `target="_blank"`)

---

## Props 상세

### `href` (필수)
네비게이트할 경로 또는 URL. 문자열 또는 객체 가능.

```tsx
// 문자열
<Link href="/dashboard">대시보드</Link>

// 쿼리 매개변수가 있는 객체
<Link href={{ pathname: '/about', query: { name: 'test' } }}>
  소개
</Link>
```

### `replace`
`true`일 때 새 히스토리 항목을 추가하는 대신 현재 항목을 교체합니다.

```tsx
<Link href="/dashboard" replace>
  대시보드
</Link>
```

### `scroll`
스크롤 동작을 제어합니다. 기본값은 `true` (스크롤 위치 유지).

```tsx
<Link href="/dashboard" scroll={false}>
  대시보드
</Link>
```

**동작**:
- `true`: 스크롤 위치 유지; 첫 번째 Page 요소가 보이지 않으면 맨 위로 스크롤
- `false`: 자동 스크롤 없음

### `prefetch`
링크된 라우트의 프리페칭을 제어합니다. 프로덕션에서만 활성화됩니다.

**값**:
- `"auto"` 또는 `null` (기본값):
  - 정적 라우트: 전체 라우트 프리페칭
  - 동적 라우트: 가장 가까운 `loading.js` 경계까지 부분 프리페칭
- `true`: 정적 및 동적 모두 전체 라우트 프리페칭
- `false`: 프리페칭 없음

```tsx
<Link href="/dashboard" prefetch={false}>
  대시보드
</Link>
```

### `onNavigate`
클라이언트 사이드 네비게이션 중 호출되는 이벤트 핸들러. `preventDefault()` 메서드가 있는 이벤트 객체를 받습니다.

```tsx
<Link
  href="/dashboard"
  onNavigate={(e) => {
    console.log('네비게이팅...')
    // e.preventDefault() // 선택사항: 네비게이션 취소
  }}
>
  대시보드
</Link>
```

**`onClick`과의 주요 차이점**:
- 클라이언트 사이드 네비게이션 중에만 실행
- 수정 키와 함께 클릭 시 실행 안 됨 (Ctrl/Cmd + Click)
- 외부 URL에서 트리거되지 않음
- `download` 속성이 있는 링크에서 트리거되지 않음

---

## 일반적인 예시

### 동적 라우트 링크
```tsx
import Link from 'next/link'

export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

### 활성 링크 감지
```tsx
'use client'

import { usePathname } from 'next/navigation'
import Link from 'next/link'

export function Links() {
  const pathname = usePathname()

  return (
    <nav>
      <Link className={`link ${pathname === '/' ? 'active' : ''}`} href="/">
        홈
      </Link>
      <Link className={`link ${pathname === '/about' ? 'active' : ''}`} href="/about">
        소개
      </Link>
    </nav>
  )
}
```

### 해시 링크
```jsx
<Link href="/dashboard#settings">설정</Link>
// 렌더링: <a href="/dashboard#settings">설정</a>
```

### 저장되지 않은 변경사항에 대한 네비게이션 차단
```tsx
'use client'

import Link from 'next/link'
import { useNavigationBlocker } from '../contexts/navigation-blocker'

export function CustomLink({ children, ...props }) {
  const { isBlocked } = useNavigationBlocker()

  return (
    <Link
      onNavigate={(e) => {
        if (
          isBlocked &&
          !window.confirm('저장되지 않은 변경사항이 있습니다. 그래도 나가시겠습니까?')
        ) {
          e.preventDefault()
        }
      }}
      {...props}
    >
      {children}
    </Link>
  )
}
```

---

## 버전 히스토리

| 버전 | 변경사항 |
|---------|---------|
| v15.4.0 | 기본 `prefetch` 동작의 별칭으로 `auto` 추가 |
| v15.3.0 | `onNavigate` API 추가 |
| v13.0.0 | 자식 `<a>` 태그 더 이상 필요 없음 |
| v10.0.0 | 동적 라우트 자동 해석; `as` prop 더 이상 필요 없음 |
| v8.0.0 | 프리페칭 성능 개선 |
| v1.0.0 | 초기 릴리스 |
