# _error.js

`_error.js` 파일은 클라이언트와 서버 양쪽에서 발생하는 에러를 처리하기 위한 커스텀 에러 페이지입니다.

## 개요

500 에러는 클라이언트와 서버 양쪽에서 `Error` 컴포넌트로 처리됩니다. 이를 오버라이드하려면 `pages/_error.js` 파일을 정의하세요.

```jsx filename="pages/_error.js"
function Error({ statusCode }) {
  return (
    <p>
      {statusCode
        ? `서버에서 ${statusCode} 에러가 발생했습니다`
        : '클라이언트에서 에러가 발생했습니다'}
    </p>
  )
}

Error.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404
  return { statusCode }
}

export default Error
```

## Props

### `statusCode`

에러 상태 코드입니다.
- `res` 객체가 있으면 `res.statusCode` 사용
- `err` 객체가 있으면 `err.statusCode` 사용
- 둘 다 없으면 404 반환

---

## 404 페이지

Next.js는 기본적으로 정적 404 페이지를 제공합니다. 매 방문 시마다 서버 렌더링하면 서버 부하 증가로 비용 상승 및 성능 저하가 발생할 수 있습니다.

### 커스터마이징

`pages/404.js` 파일을 생성하세요. 이 파일은 빌드 시 정적으로 생성됩니다.

```jsx filename="pages/404.js"
export default function Custom404() {
  return <h1>404 - 페이지를 찾을 수 없습니다</h1>
}
```

> **참고**: `getStaticProps`를 사용하여 빌드 시 데이터를 페칭할 수 있습니다.

---

## 500 페이지

Next.js는 기본적으로 정적 500 페이지를 제공합니다. 매 방문 시 서버 렌더링하면 에러 대응의 복잡도가 증가합니다.

### 커스터마이징

`pages/500.js` 파일을 생성하세요. 이 파일은 빌드 시 정적으로 생성됩니다.

```jsx filename="pages/500.js"
export default function Custom500() {
  return <h1>500 - 서버 에러가 발생했습니다</h1>
}
```

> **참고**: `getStaticProps`를 사용하여 빌드 시 데이터를 페칭할 수 있습니다.

---

## 기본 에러 페이지 재사용

Next.js의 기본 에러 페이지를 렌더링하려면 `next/error`에서 `Error` 컴포넌트를 import하세요.

```jsx
import Error from 'next/error'

export async function getServerSideProps() {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const errorCode = res.ok ? false : res.status
  const json = await res.json()

  return {
    props: { errorCode, stars: json.stargazers_count },
  }
}

export default function Page({ errorCode, stars }) {
  if (errorCode) {
    return <Error statusCode={errorCode} />
  }

  return <div>Next.js 스타 수: {stars}</div>
}
```

### Error 컴포넌트의 Props

| Prop | 설명 |
|------|------|
| `statusCode` | 에러 상태 코드 |
| `title` | (선택사항) 상태 코드와 함께 표시할 텍스트 메시지 |

---

## 주의사항

### 개발 환경 vs 프로덕션 환경

- **개발 환경**: `_error.js`는 사용되지 않으며, 콜 스택과 함께 에러가 표시됩니다.
- **프로덕션 환경**: `_error.js`만 사용됩니다.

### Data Fetching 미지원

`Error` 컴포넌트는 `getStaticProps`, `getServerSideProps` 등의 Data Fetching 메서드를 지원하지 않습니다.

### 예약된 경로명

`_error`는 `_app`처럼 예약된 경로명입니다:
- 정의된 커스텀 에러 페이지 레이아웃과 동작을 제어합니다.
- `/_error` 경로로 직접 접근하면 라우팅에서 404를 렌더링합니다.

---

## 예제

### 에러 상태에 따른 다른 UI 표시

```jsx filename="pages/_error.js"
function Error({ statusCode }) {
  if (statusCode === 404) {
    return (
      <div>
        <h1>404</h1>
        <p>요청하신 페이지를 찾을 수 없습니다.</p>
      </div>
    )
  }

  if (statusCode === 500) {
    return (
      <div>
        <h1>500</h1>
        <p>서버에서 문제가 발생했습니다. 잠시 후 다시 시도해주세요.</p>
      </div>
    )
  }

  return (
    <div>
      <h1>에러</h1>
      <p>
        {statusCode
          ? `서버에서 ${statusCode} 에러가 발생했습니다`
          : '클라이언트에서 에러가 발생했습니다'}
      </p>
    </div>
  )
}

Error.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404
  return { statusCode }
}

export default Error
```

### 에러 로깅 서비스와 통합

```jsx filename="pages/_error.js"
import * as Sentry from '@sentry/nextjs'

function Error({ statusCode, hasGetInitialPropsRun, err }) {
  if (!hasGetInitialPropsRun && err) {
    // getInitialProps가 실행되지 않은 경우
    // 에러를 Sentry에 보고
    Sentry.captureException(err)
  }

  return (
    <p>
      {statusCode
        ? `서버에서 ${statusCode} 에러가 발생했습니다`
        : '클라이언트에서 에러가 발생했습니다'}
    </p>
  )
}

Error.getInitialProps = async (context) => {
  const { res, err, asPath } = context

  // 서버에서 에러 발생 시 Sentry에 보고
  if (err) {
    Sentry.captureException(err)
    await Sentry.flush(2000)
  }

  const statusCode = res ? res.statusCode : err ? err.statusCode : 404

  return { statusCode, hasGetInitialPropsRun: true, err }
}

export default Error
```
