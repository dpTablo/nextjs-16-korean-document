# 병렬 라우트 (Parallel Routes)

병렬 라우트를 사용하면 동일한 레이아웃 내에서 하나 이상의 페이지를 동시에 또는 조건부로 렌더링할 수 있습니다. 대시보드나 소셜 사이트의 피드와 같이 앱의 매우 동적인 섹션에 유용합니다.

예를 들어, 대시보드를 고려하면 병렬 라우트를 사용하여 `team`과 `analytics` 페이지를 동시에 렌더링할 수 있습니다.

## 규칙

### 슬롯(Slots)

병렬 라우트는 명명된 **슬롯**을 사용하여 생성됩니다. 슬롯은 `@folder` 규칙으로 정의됩니다. 예를 들어, 다음 파일 구조는 `@analytics`와 `@team`이라는 두 개의 슬롯을 정의합니다.

슬롯은 공유 부모 레이아웃에 props로 전달됩니다. 위 예제에서 `app/layout.js`의 컴포넌트는 이제 `@analytics`와 `@team` 슬롯 props를 받고, `children` prop과 함께 병렬로 렌더링할 수 있습니다:

```tsx filename="app/layout.tsx"
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```

```jsx filename="app/layout.js"
export default function Layout({ children, team, analytics }) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```

그러나 슬롯은 라우트 세그먼트가 **아니며** URL 구조에 영향을 주지 않습니다. 예를 들어, `/@analytics/views`의 경우 `@analytics`가 슬롯이므로 URL은 `/views`가 됩니다. 슬롯은 일반 Page 컴포넌트와 결합되어 라우트 세그먼트와 연관된 최종 페이지를 형성합니다. 이 때문에 동일한 라우트 세그먼트 수준에서 별도의 정적 및 동적 슬롯을 가질 수 없습니다. 하나의 슬롯이 동적이면 해당 수준의 모든 슬롯이 동적이어야 합니다.

> **알아두면 좋은 점:**
> - `children` prop은 폴더에 매핑할 필요가 없는 암시적 슬롯입니다. 이는 `app/page.js`가 `app/@children/page.js`와 동등하다는 것을 의미합니다.

### `default.js`

초기 로드 또는 전체 페이지 새로고침 시 일치하지 않는 슬롯에 대한 폴백으로 렌더링할 `default.js` 파일을 정의할 수 있습니다.

다음 폴더 구조를 고려하세요. `@team` 슬롯에는 `/settings` 페이지가 있지만 `@analytics`에는 없습니다.

`/settings`로 이동할 때, `@team` 슬롯은 `/settings` 페이지를 렌더링하면서 `@analytics` 슬롯에 대해 현재 활성화된 페이지를 유지합니다.

새로고침 시, Next.js는 `@analytics`에 대해 `default.js`를 렌더링합니다. `default.js`가 없으면 대신 `404`가 렌더링됩니다.

또한 `children`은 암시적 슬롯이므로 Next.js가 부모 페이지의 활성 상태를 복구할 수 없을 때 `children`에 대한 폴백을 렌더링하기 위해 `default.js` 파일도 생성해야 합니다.

## 동작

기본적으로 Next.js는 각 슬롯의 활성 *상태*(또는 하위 페이지)를 추적합니다. 그러나 슬롯 내에서 렌더링되는 콘텐츠는 탐색 유형에 따라 달라집니다:

- **소프트 탐색**: 클라이언트 측 탐색 중에 Next.js는 부분 렌더링을 수행하여 슬롯 내의 하위 페이지를 변경하면서 다른 슬롯의 활성 하위 페이지를 유지합니다. 현재 URL과 일치하지 않더라도 마찬가지입니다.
- **하드 탐색**: 전체 페이지 로드(브라우저 새로고침) 후 Next.js는 현재 URL과 일치하지 않는 슬롯의 활성 상태를 확인할 수 없습니다. 대신 일치하지 않는 슬롯에 대해 `default.js` 파일을 렌더링하거나, `default.js`가 없으면 `404`를 렌더링합니다.

> **알아두면 좋은 점:**
> - 일치하지 않는 라우트에 대한 `404`는 의도하지 않은 페이지에서 병렬 라우트를 실수로 렌더링하지 않도록 보장합니다.

## 예제

### `useSelectedLayoutSegment(s)` 사용

`useSelectedLayoutSegment`와 `useSelectedLayoutSegments`는 모두 슬롯 내에서 활성 라우트 세그먼트를 읽을 수 있는 `parallelRoutesKey` 매개변수를 받습니다.

```tsx filename="app/layout.tsx"
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'

export default function Layout({ auth }: { auth: React.ReactNode }) {
  const loginSegment = useSelectedLayoutSegment('auth')
  // ...
}
```

```jsx filename="app/layout.js"
'use client'

import { useSelectedLayoutSegment } from 'next/navigation'

export default function Layout({ auth }) {
  const loginSegment = useSelectedLayoutSegment('auth')
  // ...
}
```

사용자가 `app/@auth/login`(또는 URL 바에서 `/login`)으로 이동하면 `loginSegment`는 문자열 `"login"`과 같아집니다.

### 조건부 라우트

병렬 라우트를 사용하여 사용자 역할과 같은 특정 조건에 따라 라우트를 조건부로 렌더링할 수 있습니다. 예를 들어, `/admin` 또는 `/user` 역할에 대해 다른 대시보드 페이지를 렌더링하려면:

```tsx filename="app/dashboard/layout.tsx"
import { checkUserRole } from '@/lib/auth'

export default function Layout({
  user,
  admin,
}: {
  user: React.ReactNode
  admin: React.ReactNode
}) {
  const role = checkUserRole()
  return role === 'admin' ? admin : user
}
```

```jsx filename="app/dashboard/layout.js"
import { checkUserRole } from '@/lib/auth'

export default function Layout({ user, admin }) {
  const role = checkUserRole()
  return role === 'admin' ? admin : user
}
```

### 탭 그룹

슬롯 내에 `layout`을 추가하여 사용자가 슬롯을 독립적으로 탐색할 수 있도록 할 수 있습니다. 이것은 탭을 만드는 데 유용합니다.

예를 들어, `@analytics` 슬롯에는 `/page-views`와 `/visitors`라는 두 개의 하위 페이지가 있습니다.

`@analytics` 내에서 두 페이지 간에 탭을 공유하는 `layout` 파일을 생성합니다:

```tsx filename="app/@analytics/layout.tsx"
import Link from 'next/link'

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Link href="/page-views">페이지 뷰</Link>
        <Link href="/visitors">방문자</Link>
      </nav>
      <div>{children}</div>
    </>
  )
}
```

```jsx filename="app/@analytics/layout.js"
import Link from 'next/link'

export default function Layout({ children }) {
  return (
    <>
      <nav>
        <Link href="/page-views">페이지 뷰</Link>
        <Link href="/visitors">방문자</Link>
      </nav>
      <div>{children}</div>
    </>
  )
}
```

### 모달

병렬 라우트를 인터셉팅 라우트와 함께 사용하여 딥 링킹을 지원하는 모달을 만들 수 있습니다. 이를 통해 모달을 빌드할 때 일반적인 문제를 해결할 수 있습니다:

- **URL을 통해 모달 콘텐츠를 공유**할 수 있게 함
- 페이지를 새로고침할 때 모달을 닫는 대신 **컨텍스트를 유지**
- 이전 라우트로 이동하는 대신 **뒤로 탐색 시 모달 닫기**
- **앞으로 탐색 시 모달 다시 열기**

다음 UI 패턴을 고려하세요. 사용자가 클라이언트 측 탐색을 사용하여 레이아웃에서 로그인 모달을 열거나 별도의 `/login` 페이지에 액세스할 수 있습니다.

이 패턴을 구현하려면 먼저 **메인** 로그인 페이지를 렌더링하는 `/login` 라우트를 생성합니다.

```tsx filename="app/login/page.tsx"
import { Login } from '@/app/ui/login'

export default function Page() {
  return <Login />
}
```

```jsx filename="app/login/page.js"
import { Login } from '@/app/ui/login'

export default function Page() {
  return <Login />
}
```

그런 다음 `@auth` 슬롯 내에 `null`을 반환하는 `default.js` 파일을 추가합니다. 이렇게 하면 모달이 활성화되지 않을 때 렌더링되지 않습니다.

```tsx filename="app/@auth/default.tsx"
export default function Default() {
  return null
}
```

```jsx filename="app/@auth/default.js"
export default function Default() {
  return null
}
```

`@auth` 슬롯 내에서 `<Modal>` 컴포넌트와 그 자식을 `@auth/(.)login/page.tsx` 파일에 가져와서 `/login` 라우트를 인터셉트합니다.

```tsx filename="app/@auth/(.)login/page.tsx"
import { Modal } from '@/app/ui/modal'
import { Login } from '@/app/ui/login'

export default function Page() {
  return (
    <Modal>
      <Login />
    </Modal>
  )
}
```

#### 모달 열기

이제 Next.js 라우터를 활용하여 모달을 열고 닫을 수 있습니다. 이렇게 하면 모달이 열릴 때와 앞뒤로 탐색할 때 URL이 올바르게 업데이트됩니다.

모달을 열려면 `@auth` 슬롯을 부모 레이아웃에 prop으로 전달하고 `children` prop과 함께 렌더링합니다.

```tsx filename="app/layout.tsx"
import Link from 'next/link'

export default function Layout({
  auth,
  children,
}: {
  auth: React.ReactNode
  children: React.ReactNode
}) {
  return (
    <>
      <nav>
        <Link href="/login">모달 열기</Link>
      </nav>
      <div>{auth}</div>
      <div>{children}</div>
    </>
  )
}
```

사용자가 `<Link>`를 클릭하면 `/login` 페이지로 이동하는 대신 모달이 열립니다. 그러나 새로고침 또는 초기 로드 시 `/login`으로 이동하면 사용자가 메인 로그인 페이지로 이동합니다.

#### 모달 닫기

`router.back()`을 호출하거나 `Link` 컴포넌트를 사용하여 모달을 닫을 수 있습니다.

```tsx filename="app/ui/modal.tsx"
'use client'

import { useRouter } from 'next/navigation'

export function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter()

  return (
    <>
      <button
        onClick={() => {
          router.back()
        }}
      >
        모달 닫기
      </button>
      <div>{children}</div>
    </>
  )
}
```

### 로딩 및 에러 UI

병렬 라우트는 독립적으로 스트리밍될 수 있어 각 라우트에 대해 독립적인 에러 및 로딩 상태를 정의할 수 있습니다.

자세한 정보는 로딩 UI 및 에러 처리 문서를 참조하세요.

---

**관련 문서:**
- [default.js](/app-router/api-reference/file-conventions/default.md) - default.js 파일의 API 참조
