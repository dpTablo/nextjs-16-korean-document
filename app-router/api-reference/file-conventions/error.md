---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/error
버전: 16.1.6
---

# error.js

`error.js` 파일은 예상치 못한 런타임 오류를 처리하고 폴백 UI를 표시하는 역할을 합니다. React Error Boundary로 라우트 세그먼트와 중첩된 자식 요소를 감싸서 오류 발생 시 폴백 UI를 표시합니다.

---

## 필수 요구사항

⚠️ **'use client' 필수**: Error boundaries는 반드시 Client Components여야 합니다.

---

## 기본 사용법

```tsx
// app/dashboard/error.tsx
'use client'

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // 오류를 로깅 서비스에 기록
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

---

## Props

### error

Error 객체의 인스턴스입니다.

#### error.message

- **Client Component 오류**: 원본 오류 메시지 표시
- **Server Component 오류**: 민감한 정보 보호를 위해 일반 메시지 + 식별자 표시

```tsx
'use client'

export default function Error({ error }: { error: Error }) {
  return (
    <div>
      <h2>오류 발생</h2>
      <p>{error.message}</p>
    </div>
  )
}
```

#### error.digest

자동 생성되는 오류의 해시값입니다. 서버 측 로그의 대응하는 오류와 매칭하는 데 사용됩니다.

```tsx
'use client'

export default function Error({
  error,
}: {
  error: Error & { digest?: string }
}) {
  return (
    <div>
      <h2>오류 발생</h2>
      <p>오류 ID: {error.digest}</p>
    </div>
  )
}
```

### reset()

오류 경계 콘텐츠를 재렌더링하려고 시도하는 함수입니다.

- 성공하면 폴백 오류 컴포넌트가 재렌더링 결과로 대체됩니다
- 임시 오류 복구에 유용합니다

```tsx
'use client'

export default function Error({ reset }: { reset: () => void }) {
  return (
    <div>
      <h2>오류가 발생했습니다</h2>
      <button onClick={() => reset()}>다시 시도</button>
    </div>
  )
}
```

---

## 사용 예제

### 기본 에러 UI

```tsx
// app/posts/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">게시물을 불러오지 못했습니다</h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <button
        onClick={() => reset()}
        className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        다시 시도
      </button>
    </div>
  )
}
```

### 오류 로깅

```tsx
// app/dashboard/error.tsx
'use client'

import { useEffect } from 'react'
import * as Sentry from '@sentry/nextjs'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Sentry 또는 다른 오류 추적 서비스에 로그
    Sentry.captureException(error)
  }, [error])

  return (
    <div>
      <h2>문제가 발생했습니다</h2>
      <p>오류 ID: {error.digest}</p>
      <button onClick={() => reset()}>다시 시도</button>
    </div>
  )
}
```

### 조건부 메시지

```tsx
// app/api-data/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  const isNetworkError = error.message.includes('fetch')

  return (
    <div>
      <h2>
        {isNetworkError
          ? '네트워크 연결을 확인해주세요'
          : '오류가 발생했습니다'}
      </h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>
        {isNetworkError ? '재연결' : '다시 시도'}
      </button>
    </div>
  )
}
```

---

## Global Error (전역 오류 처리)

루트 레이아웃의 오류를 처리하려면 `app/global-error.js`를 사용하세요.

```tsx
// app/global-error.tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <h2>전역 오류 발생!</h2>
        <p>{error.message}</p>
        <button onClick={() => reset()}>다시 시도</button>
      </body>
    </html>
  )
}
```

**주의사항:**
- ✅ `<html>`, `<body>` 태그 필수
- ❌ `metadata`, `generateMetadata` 미지원
- ✅ React `<title>` 컴포넌트로 타이틀 설정 가능

```tsx
// app/global-error.tsx
'use client'

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <head>
        <title>오류 발생</title>
      </head>
      <body>
        <h2>전역 오류 발생!</h2>
        <button onClick={() => reset()}>다시 시도</button>
      </body>
    </html>
  )
}
```

---

## 파일 구조 예제

```
app/
├── layout.tsx
├── page.tsx
├── error.tsx              # 루트 에러 처리
├── global-error.tsx       # 전역 에러 처리
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    └── error.tsx          # 대시보드 에러 처리
```

**작동 방식:**

```tsx
// 내부적으로 다음과 같이 작동
<Layout>
  <ErrorBoundary fallback={<Error />}>
    <Page />
  </ErrorBoundary>
</Layout>
```

---

## 중첩된 Error Boundaries

오류는 가장 가까운 상위 error boundary로 버블링됩니다.

```
app/
├── error.tsx              # 전체 앱 에러 처리
└── dashboard/
    ├── error.tsx          # 대시보드 에러 처리
    └── settings/
        ├── error.tsx      # 설정 에러 처리 (가장 가까움)
        └── page.tsx
```

`app/dashboard/settings/page.tsx`에서 오류 발생 시:
1. `app/dashboard/settings/error.tsx` 표시 (있는 경우)
2. 없으면 `app/dashboard/error.tsx` 표시
3. 없으면 `app/error.tsx` 표시
4. 없으면 `app/global-error.tsx` 표시

---

## 개발 vs 프로덕션

### 개발 환경

- 상세한 오류 스택 트레이스 표시
- React DevTools로 error boundary 토글 가능
- `global-error`도 표시됨 (v15.2.0+)

### 프로덕션 환경

- 사용자에게는 간단한 오류 메시지만 표시
- 민감한 정보 보호
- `error.digest`로 서버 로그와 연결

---

## 모범 사례

### 1. 의미 있는 오류 메시지

```tsx
// ❌ 나쁜 예
<h2>Error!</h2>

// ✅ 좋은 예
<h2>게시물을 불러올 수 없습니다</h2>
<p>잠시 후 다시 시도해주세요.</p>
```

### 2. 오류 로깅

```tsx
useEffect(() => {
  // 오류 추적 서비스에 로그
  logErrorToService(error)
}, [error])
```

### 3. 복구 옵션 제공

```tsx
<div>
  <button onClick={() => reset()}>다시 시도</button>
  <Link href="/">홈으로 돌아가기</Link>
</div>
```

### 4. 사용자 친화적 UI

```tsx
export default function Error({ error, reset }) {
  return (
    <div className="text-center p-8">
      <div className="text-6xl mb-4">😕</div>
      <h2 className="text-2xl font-bold mb-2">앗, 문제가 발생했습니다</h2>
      <p className="text-gray-600 mb-4">
        일시적인 문제일 수 있습니다. 다시 시도해주세요.
      </p>
      <button onClick={() => reset()}>다시 시도</button>
    </div>
  )
}
```

---

## Layout 에러는 처리 안됨

`error.js`는 **같은 세그먼트**의 `layout.js`에서 발생한 오류를 처리하지 못합니다.

**이유:**
- error boundary가 해당 레이아웃 컴포넌트 내부에 중첩되기 때문

**해결 방법:**
- 상위 세그먼트의 `error.js` 사용
- `global-error.js` 사용 (루트 레이아웃의 경우)

---

## 중요한 주의사항

> **Good to know**:
> * `error.js`는 반드시 Client Component(`'use client'`)여야 합니다
> * Server Component에서 발생한 오류는 민감한 정보를 숨기기 위해 일반 메시지로 표시됩니다
> * `error.digest`를 사용하여 서버 로그와 클라이언트 오류를 매칭할 수 있습니다
> * `error.js`는 같은 세그먼트의 `layout.js` 오류를 처리하지 못합니다
> * React DevTools로 error boundaries를 테스트할 수 있습니다

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.2.0 | 개발 환경에서도 `global-error` 표시 |
| v13.1.0 | `global-error` 도입 |
| v13.0.0 | `error.js` 도입 |

---

## 관련 문서

- [loading.js](./loading.md)
- [not-found.js](./not-found.md)
- [Error Handling](../../getting-started/07-error-handling.md)
- [React Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
