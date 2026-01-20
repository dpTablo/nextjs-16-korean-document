# useLinkStatus

`useLinkStatus` Hook은 `<Link>` 컴포넌트의 **pending** 상태를 추적합니다. 네비게이션 완료 중에 클릭된 링크에 shimmer 효과 등의 인라인 피드백을 제공할 때 유용합니다.

---

## 사용 시기

`useLinkStatus`는 다음 상황에서 유용합니다:

- 프리페칭이 비활성화되었거나 진행 중일 때 (네비게이션이 차단됨)
- 대상 라우트가 동적이며 `loading.js` 파일이 없을 때

---

## 시그니처

```tsx
const { pending } = useLinkStatus()
```

### 매개변수

없음

### 반환값

| 속성 | 타입 | 설명 |
|------|------|------|
| `pending` | `boolean` | history 업데이트 전 `true`, 후 `false` |

---

## 기본 사용법

```tsx
'use client'

import Link from 'next/link'
import { useLinkStatus } from 'next/link'

function Hint() {
  const { pending } = useLinkStatus()
  return (
    <span
      aria-hidden
      className={`link-hint ${pending ? 'is-pending' : ''}`}
    />
  )
}

export default function Header() {
  return (
    <header>
      <Link href="/dashboard" prefetch={false}>
        <span className="label">대시보드</span> <Hint />
      </Link>
    </header>
  )
}
```

---

## 중요 사항

| 항목 | 설명 |
|------|------|
| ✅ 사용 위치 | `Link` 컴포넌트의 자식 컴포넌트 내에서만 사용 가능 |
| ✅ 권장 설정 | `Link` 컴포넌트에 `prefetch={false}` 설정 시 가장 유용 |
| ✅ 프리페칭된 라우트 | pending 상태를 건너뜀 |
| ✅ 연속 클릭 | 여러 링크를 빠르게 클릭하면 마지막 링크의 pending만 표시 |
| ❌ Pages Router | 미지원 (항상 `{ pending: false }` 반환) |
| ⚠️ 레이아웃 시프트 | 인라인 지표는 레이아웃 시프트를 유발할 수 있으므로 고정 크기 요소 사용 권장 |

---

## 실제 예제

### 로딩 인디케이터 컴포넌트

```tsx
// app/components/loading-indicator.tsx
'use client'

import { useLinkStatus } from 'next/link'

export default function LoadingIndicator() {
  const { pending } = useLinkStatus()

  return (
    <span
      aria-hidden
      className={`link-hint ${pending ? 'is-pending' : ''}`}
    />
  )
}
```

### 네비게이션 바에서 사용

```tsx
// app/layout.tsx
import Link from 'next/link'
import LoadingIndicator from './components/loading-indicator'

const links = [
  { href: '/shop/electronics', label: '전자제품' },
  { href: '/shop/clothing', label: '의류' },
  { href: '/shop/books', label: '도서' },
]

function Menubar() {
  return (
    <nav>
      {links.map((link) => (
        <Link key={link.label} href={link.href} prefetch={false}>
          <span className="label">{link.label}</span>
          <LoadingIndicator />
        </Link>
      ))}
    </nav>
  )
}

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <Menubar />
      <main>{children}</main>
    </div>
  )
}
```

### CSS 애니메이션 스타일

```css
/* globals.css */
.link-hint {
  display: inline-block;
  width: 0.6em;
  height: 0.6em;
  margin-left: 0.25rem;
  border-radius: 9999px;
  background: currentColor;
  opacity: 0;
  visibility: hidden; /* 공간 확보하되 표시 안 함 */
}

.link-hint.is-pending {
  visibility: visible;
  animation-name: fadeIn, pulse;
  animation-duration: 200ms, 1s;
  animation-delay: 100ms, 100ms; /* 네비게이션이 시간이 걸릴 때만 표시 */
  animation-timing-function: ease, ease-in-out;
  animation-iteration-count: 1, infinite;
  animation-fill-mode: forwards, none;
}

@keyframes fadeIn {
  to {
    opacity: 0.35;
  }
}

@keyframes pulse {
  50% {
    opacity: 0.15;
  }
}
```

### 스피너 인디케이터

```tsx
// app/components/link-spinner.tsx
'use client'

import { useLinkStatus } from 'next/link'

export default function LinkSpinner() {
  const { pending } = useLinkStatus()

  if (!pending) return null

  return (
    <span className="spinner" aria-label="로딩 중">
      <svg
        width="16"
        height="16"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
      >
        <circle
          cx="12"
          cy="12"
          r="10"
          strokeOpacity="0.25"
        />
        <path
          d="M12 2a10 10 0 0 1 10 10"
          strokeLinecap="round"
        >
          <animateTransform
            attributeName="transform"
            type="rotate"
            from="0 12 12"
            to="360 12 12"
            dur="1s"
            repeatCount="indefinite"
          />
        </path>
      </svg>
    </span>
  )
}
```

### 조건부 스타일 적용

```tsx
// app/components/nav-link.tsx
'use client'

import Link from 'next/link'
import { useLinkStatus } from 'next/link'

function NavLinkContent({ children }: { children: React.ReactNode }) {
  const { pending } = useLinkStatus()

  return (
    <span
      style={{
        opacity: pending ? 0.7 : 1,
        transition: 'opacity 150ms',
      }}
    >
      {children}
      {pending && <span className="ml-2">...</span>}
    </span>
  )
}

export default function NavLink({
  href,
  children,
}: {
  href: string
  children: React.ReactNode
}) {
  return (
    <Link href={href} prefetch={false}>
      <NavLinkContent>{children}</NavLinkContent>
    </Link>
  )
}
```

---

## 사용하지 않아도 되는 경우

다음 조건이라면 `useLinkStatus`가 불필요할 수 있습니다:

1. **대상이 정적이고 프리페칭됨** - pending 단계가 스킵될 수 있음
2. **`loading.js` 파일이 있음** - 라우트 레벨의 즉시 전환 가능

> **팁**: 네비게이션은 일반적으로 빠릅니다. 느린 전환을 발견했을 때만 사용하고, 프리페칭이나 `loading.js`로 근본 원인을 해결하는 것이 좋습니다.

---

## 중요한 주의사항

> **Good to know**:
> - `useLinkStatus`는 반드시 `Link` 컴포넌트의 자식 내에서 호출해야 합니다
> - 부모 컴포넌트에서 호출하면 항상 `{ pending: false }`를 반환합니다
> - Pages Router에서는 지원되지 않습니다

> **성능 팁**:
> - 인라인 로딩 인디케이터는 레이아웃 시프트를 방지하기 위해 고정 크기를 사용하세요
> - CSS 애니메이션에 약간의 지연을 추가하여 빠른 네비게이션에서 깜박임을 방지하세요

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.3.0 | `useLinkStatus` 도입 |

---

## 관련 문서

- [Link Component](../link.md)
- [loading.js](../file-conventions/loading.md)
- [Prefetching](../../guides/prefetching.md)
