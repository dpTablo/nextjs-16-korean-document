# not-found.js

## 개요
Next.js는 not found 케이스를 처리하기 위한 두 가지 규칙을 제공합니다:
- **`not-found.js`**: 라우트 세그먼트 내에서 `notFound()` 함수가 호출될 때 UI를 렌더링합니다
- **`global-not-found.js`**: 전체 앱에서 매칭되지 않는 라우트에 대한 전역 404 페이지를 정의합니다 (실험적 기능)

## `not-found.js`

### 목적
`not-found` 파일은 `notFound()` 함수가 발생할 때 커스텀 UI를 렌더링합니다. HTTP 상태 코드:
- 스트리밍된 응답의 경우 `200`
- 스트리밍되지 않은 응답의 경우 `404`

### 기본 예제
```tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>요청한 리소스를 찾을 수 없습니다</p>
      <Link href="/">홈으로 돌아가기</Link>
    </div>
  )
}
```

모든 라우트 레벨에 배치할 수 있습니다 (예: `app/not-found.tsx` 또는 `app/blog/not-found.js`).

## `global-not-found.js` (실험적 기능)

### 목적
전체 애플리케이션에서 매칭되지 않는 URL에 대한 404 페이지를 처리하며, 일반 렌더링을 우회합니다.

### 사용 시기
- 여러 루트 레이아웃이 존재하는 경우 (예: `app/(admin)/` 및 `app/(shop)/`)
- 루트 레이아웃이 최상위 동적 세그먼트를 사용하는 경우 (예: `app/[country]/`)

### 설정
1. `next.config.ts`에서 활성화:
```tsx
const nextConfig: NextConfig = {
  experimental: {
    globalNotFound: true,
  },
}
```

2. `app/global-not-found.tsx` 생성:
```tsx
import './globals.css'
import { Inter } from 'next/font/google'
import type { Metadata } from 'next'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: '404 - Page Not Found',
  description: 'The page you are looking for does not exist.',
}

export default function GlobalNotFound() {
  return (
    <html lang="ko" className={inter.className}>
      <body>
        <h1>404 - 페이지를 찾을 수 없습니다</h1>
        <p>이 페이지는 존재하지 않습니다.</p>
      </body>
    </html>
  )
}
```

**참고**: `<html>` 및 `<body>` 태그를 포함한 전체 HTML 문서를 반환해야 합니다.

## Props
`not-found.js` 및 `global-not-found.js` 컴포넌트는 어떤 props도 받지 않습니다.

## 고급 기능

### 데이터 페칭
`not-found`는 Server Component이며 `async`로 표시할 수 있습니다:
```tsx
import { headers } from 'next/headers'

export default async function NotFound() {
  const headersList = await headers()
  const domain = headersList.get('host')
  const data = await getSiteData(domain)
  return (
    <div>
      <h2>Not Found: {data.name}</h2>
      <p>요청한 리소스를 찾을 수 없습니다</p>
    </div>
  )
}
```

### 메타데이터
`global-not-found.js`에 대해 `metadata` 객체 또는 `generateMetadata` 함수를 내보낼 수 있습니다:
- Next.js는 404 페이지에 대해 자동으로 `<meta name="robots" content="noindex" />`를 주입합니다

## 버전 히스토리
| 버전 | 변경사항 |
|---------|--------|
| v15.4.0 | `global-not-found.js` 도입 (실험적 기능) |
| v13.3.0 | 루트 `app/not-found`가 전역 매칭되지 않는 URL 처리 |
| v13.0.0 | `not-found` 도입 |

## 관련 문서

- [notFound 함수](../functions/notFound.md)
- [에러 처리](../../getting-started/07-error-handling.md)
