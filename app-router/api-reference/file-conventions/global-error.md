---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/global-error
버전: 16.1.6
---

# global-error.js

## 개요

`global-error.js`는 루트 레이아웃이나 템플릿에서 발생하는 오류를 처리하는 특수 파일입니다. 일반 `error.js`와 달리 전체 애플리케이션을 감싸는 최상위 오류 처리기입니다.

---

## 목적

- 루트 레이아웃 또는 템플릿의 오류 처리
- 활성화될 때 루트 레이아웃이나 템플릿을 대체
- 자체 `<html>` 및 `<body>` 태그 정의 필요

---

## 파일 위치

```
app/global-error.tsx
app/global-error.js
```

---

## error.js와의 주요 차이점

| 특성 | error.js | global-error.js |
|------|----------|-----------------|
| **위치** | 모든 라우트 세그먼트 | 루트 app 디렉토리만 |
| **HTML 태그** | 불필요 | `<html>`, `<body>` 필수 |
| **스타일/폰트** | 레이아웃에서 상속 | 자체 정의 필요 |
| **메타데이터** | 지원 | 미지원 (React `<title>` 사용) |
| **클라이언트 컴포넌트** | 필수 | 필수 |

---

## 필수 요구사항

⚠️ **'use client' 필수**: Global error는 반드시 Client Component여야 합니다.

---

## 기본 사용법

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
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
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

#### error.digest
- 자동 생성되는 오류의 해시값
- 서버 측 로그의 대응하는 오류와 매칭하는 데 사용

### reset()

오류 경계 콘텐츠를 재렌더링하려고 시도하는 함수입니다.
- 성공하면 폴백 오류 컴포넌트가 재렌더링 결과로 대체됩니다
- 임시 오류 복구에 유용합니다

---

## 완전한 예제

```tsx
// app/global-error.tsx
'use client'

import { useEffect } from 'react'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // 오류를 로깅 서비스에 기록
    console.error('Global error:', error)
  }, [error])

  return (
    <html lang="ko">
      <head>
        <title>오류 발생</title>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
      </head>
      <body style={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center',
        minHeight: '100vh',
        fontFamily: 'system-ui, sans-serif',
        backgroundColor: '#f5f5f5',
        padding: '20px',
      }}>
        <div style={{
          textAlign: 'center',
          backgroundColor: 'white',
          padding: '40px',
          borderRadius: '8px',
          boxShadow: '0 2px 10px rgba(0,0,0,0.1)',
        }}>
          <h1 style={{ fontSize: '48px', marginBottom: '20px' }}>😞</h1>
          <h2 style={{ fontSize: '24px', marginBottom: '10px' }}>
            전역 오류가 발생했습니다
          </h2>
          <p style={{ color: '#666', marginBottom: '20px' }}>
            애플리케이션에 문제가 발생했습니다.
          </p>
          {error.digest && (
            <p style={{ fontSize: '12px', color: '#999', marginBottom: '20px' }}>
              오류 ID: {error.digest}
            </p>
          )}
          <button
            onClick={() => reset()}
            style={{
              padding: '10px 20px',
              fontSize: '16px',
              backgroundColor: '#0070f3',
              color: 'white',
              border: 'none',
              borderRadius: '5px',
              cursor: 'pointer',
            }}
          >
            다시 시도
          </button>
        </div>
      </body>
    </html>
  )
}
```

---

## 스타일링

`global-error.js`는 자체 스타일을 정의해야 합니다:

### 1. 인라인 스타일 (위 예제 참조)

### 2. CSS-in-JS
```tsx
'use client'

import { styled } from 'styled-components'

const Container = styled.div`
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
`

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <Container>
          <h2>전역 오류 발생!</h2>
          <button onClick={() => reset()}>다시 시도</button>
        </Container>
      </body>
    </html>
  )
}
```

### 3. 글로벌 CSS 직접 포함
```tsx
'use client'

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <head>
        <style>{`
          body {
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            font-family: system-ui, sans-serif;
          }
        `}</style>
      </head>
      <body>
        <div>
          <h2>전역 오류 발생!</h2>
          <button onClick={() => reset()}>다시 시도</button>
        </div>
      </body>
    </html>
  )
}
```

---

## 주의사항

### ✅ 해야 할 것
- `<html>` 및 `<body>` 태그 포함
- 자체 스타일 정의
- React `<title>` 컴포넌트로 타이틀 설정
- 'use client' 지시어 사용

### ❌ 하지 말아야 할 것
- `metadata` 또는 `generateMetadata` 내보내기
- 레이아웃 스타일 상속 기대
- Server Component로 작성

---

## 오류 계층 구조

```
app/
├── global-error.tsx       # 최상위 오류 처리기
├── error.tsx              # 루트 레벨 오류 처리
├── layout.tsx
└── dashboard/
    ├── error.tsx          # 대시보드 오류 처리
    └── page.tsx
```

**오류 발생 시 우선순위:**
1. 가장 가까운 `error.js` 확인
2. 상위로 버블링
3. 루트 `error.js` 확인
4. 마지막으로 `global-error.js` 표시

---

## 개발 vs 프로덕션

### 개발 환경 (v15.2.0+)
- `global-error`가 개발 모드에서도 표시됨
- 상세한 오류 스택 트레이스 제공
- 디버깅 정보 포함

### 프로덕션 환경
- 사용자에게는 간단한 오류 메시지만 표시
- 민감한 정보 보호
- `error.digest`로 서버 로그와 연결

---

## 모범 사례

### 1. 포괄적인 오류 처리

```tsx
'use client'

import { useEffect } from 'react'
import * as Sentry from '@sentry/nextjs'

export default function GlobalError({ error, reset }) {
  useEffect(() => {
    // 오류 추적 서비스에 로그
    Sentry.captureException(error, {
      tags: { errorBoundary: 'global' }
    })
  }, [error])

  return (
    <html>
      <body>
        {/* 오류 UI */}
      </body>
    </html>
  )
}
```

### 2. 사용자 친화적 메시지

```tsx
export default function GlobalError({ error, reset }) {
  return (
    <html lang="ko">
      <body>
        <div>
          <h2>죄송합니다, 예기치 않은 문제가 발생했습니다</h2>
          <p>
            잠시 후 다시 시도하거나, 문제가 지속되면 고객 지원팀에 문의해주세요.
          </p>
          <button onClick={() => reset()}>다시 시도</button>
          <a href="/">홈으로 돌아가기</a>
        </div>
      </body>
    </html>
  )
}
```

### 3. 접근성 고려

```tsx
export default function GlobalError({ error, reset }) {
  return (
    <html lang="ko">
      <body>
        <div role="alert" aria-live="assertive">
          <h2>오류가 발생했습니다</h2>
          <p>{error.message}</p>
          <button onClick={() => reset()} aria-label="페이지 다시 로드">
            다시 시도
          </button>
        </div>
      </body>
    </html>
  )
}
```

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.2.0 | 개발 환경에서도 `global-error` 표시 |
| v13.1.0 | `global-error` 도입 |

---

## 관련 문서

- [error.js](./error.md)
- [not-found.js](./not-found.md)
- [에러 처리](../../getting-started/07-error-handling.md)
- [React Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
