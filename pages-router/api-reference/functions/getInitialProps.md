# getInitialProps

> **참고**: `getInitialProps`는 레거시 API입니다. 대신 [`getStaticProps`](/docs/pages/api-reference/functions/getStaticProps) 또는 [`getServerSideProps`](/docs/pages/api-reference/functions/getServerSideProps) 사용을 권장합니다.

`getInitialProps`는 페이지의 기본 내보낸 React 컴포넌트에 추가할 수 있는 `async` 함수입니다. 서버 사이드와 클라이언트 사이드(페이지 전환 시) 모두에서 실행되며, 함수의 반환값이 React 컴포넌트에 `props`로 전달됩니다.

## 기본 사용법

```tsx filename="pages/index.tsx"
import { NextPageContext } from 'next'

Page.getInitialProps = async (ctx: NextPageContext) => {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const json = await res.json()
  return { stars: json.stargazers_count }
}

export default function Page({ stars }: { stars: number }) {
  return stars
}
```

```jsx filename="pages/index.js"
Page.getInitialProps = async (ctx) => {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const json = await res.json()
  return { stars: json.stargazers_count }
}

export default function Page({ stars }) {
  return stars
}
```

## Context 객체

`getInitialProps`는 `context`라는 단일 인자를 받습니다. 이 객체에는 다음 속성들이 포함됩니다:

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `pathname` | `String` | 현재 라우트, `/pages`의 페이지 경로 |
| `query` | `Object` | URL의 쿼리 문자열 (객체로 파싱됨) |
| `asPath` | `String` | 브라우저에 표시된 실제 경로 (쿼리 포함) |
| `req` | `HTTP request object` | HTTP 요청 객체 (서버 전용) |
| `res` | `HTTP response object` | HTTP 응답 객체 (서버 전용) |
| `err` | `Error` | 렌더링 중 발생한 에러 객체 |

## 반환 값

`getInitialProps`는 직렬화 가능한 평문 JavaScript 객체를 반환해야 합니다. 반환된 객체의 각 키는 컴포넌트의 `props`로 전달됩니다.

```tsx
Page.getInitialProps = async (ctx) => {
  return {
    title: '페이지 제목',
    data: { id: 1, name: 'Next.js' },
    count: 42,
  }
}

// 컴포넌트에서 props로 받음
export default function Page({ title, data, count }) {
  return (
    <div>
      <h1>{title}</h1>
      <p>{data.name}</p>
      <span>{count}</span>
    </div>
  )
}
```

## 주의사항

### 데이터 직렬화

반환 객체는 JSON으로 직렬화 가능한 **평문 Object**여야 합니다. `Date`, `Map`, `Set` 등의 객체는 사용할 수 없습니다.

```tsx
// 올바른 사용
return { date: new Date().toISOString() }

// 잘못된 사용 - Date 객체 직접 반환
return { date: new Date() }
```

### 실행 타이밍

- **초기 페이지 로드**: 서버에서만 실행됩니다.
- **페이지 전환**: `next/link` 또는 `next/router`를 사용하여 페이지를 전환할 때 클라이언트에서 실행됩니다.
- **Custom `_app.js` 사용 시**: 대상 페이지가 `getServerSideProps`를 사용하면 서버에서만 실행됩니다.

### 사용 제한

- **`pages/` 디렉토리의 최상단 파일에서만 사용 가능**합니다.
- 중첩 컴포넌트에서는 `getInitialProps`를 사용할 수 없습니다.
- 데이터 페칭이 필요한 중첩 컴포넌트가 있다면 App Router 사용을 권장합니다.

### 보안

반환된 props는 초기 HTML에 포함되어 클라이언트 측에서 검사할 수 있습니다. **민감한 정보를 props로 전달하면 안 됩니다**. 페이지 hydration을 위해 필요한 데이터만 반환하세요.

```tsx
// 잘못된 사용 - 민감한 정보 노출
return {
  user: {
    name: 'John',
    password: 'secret123', // 클라이언트에 노출됨!
  },
}

// 올바른 사용 - 필요한 데이터만 반환
return {
  user: {
    name: 'John',
    id: 123,
  },
}
```

## 예제

### 서버/클라이언트 환경 구분

```tsx
Page.getInitialProps = async (ctx) => {
  const { req, query } = ctx

  // req가 있으면 서버 사이드
  if (req) {
    // 서버에서만 실행되는 로직
    console.log('서버에서 실행됨')
  } else {
    // 클라이언트에서만 실행되는 로직
    console.log('클라이언트에서 실행됨')
  }

  return { query }
}
```

### 에러 처리

```tsx
Page.getInitialProps = async (ctx) => {
  const { res, err } = ctx

  if (err) {
    // 에러가 발생한 경우
    if (res) {
      res.statusCode = err.statusCode || 500
    }
    return { error: err.message }
  }

  try {
    const data = await fetchData()
    return { data }
  } catch (error) {
    if (res) {
      res.statusCode = 500
    }
    return { error: '데이터를 불러오는 데 실패했습니다.' }
  }
}
```

### 리다이렉션

```tsx
Page.getInitialProps = async (ctx) => {
  const { res, req } = ctx

  // 서버 사이드 리다이렉션
  if (res) {
    res.writeHead(302, { Location: '/login' })
    res.end()
    return {}
  }

  // 클라이언트 사이드 리다이렉션
  if (typeof window !== 'undefined') {
    window.location.href = '/login'
    return {}
  }

  return {}
}
```

## `getStaticProps` / `getServerSideProps`와의 비교

| 기능 | `getInitialProps` | `getStaticProps` | `getServerSideProps` |
|------|-------------------|------------------|---------------------|
| 실행 환경 | 서버 + 클라이언트 | 서버만 (빌드 시) | 서버만 (요청 시) |
| 정적 생성 | 불가 | 가능 | 불가 |
| ISR 지원 | 불가 | 가능 | 불가 |
| 자동 코드 분할 | 제한적 | 완전 지원 | 완전 지원 |
| 권장 여부 | 레거시 | 권장 | 권장 |

## 마이그레이션 가이드

`getInitialProps`에서 최신 API로 마이그레이션하려면:

1. **정적 콘텐츠**: `getStaticProps` 사용
2. **동적 콘텐츠 (요청별로 달라지는 데이터)**: `getServerSideProps` 사용
3. **클라이언트 사이드 데이터 페칭**: `useEffect` 또는 SWR/React Query 사용

```tsx
// Before: getInitialProps
Page.getInitialProps = async (ctx) => {
  const data = await fetchData()
  return { data }
}

// After: getServerSideProps (요청별 데이터)
export async function getServerSideProps(context) {
  const data = await fetchData()
  return { props: { data } }
}

// After: getStaticProps (정적 데이터)
export async function getStaticProps() {
  const data = await fetchData()
  return { props: { data } }
}
```
