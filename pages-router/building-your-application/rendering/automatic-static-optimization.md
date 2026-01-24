# 자동 정적 최적화

Next.js는 `getServerSideProps`와 `getInitialProps`가 없는 페이지를 자동으로 정적 페이지로 판단합니다. 이 기능을 통해 **서버 렌더링 페이지와 정적 생성 페이지가 모두 포함된 하이브리드 애플리케이션**을 만들 수 있습니다.

## 핵심 특징

- 정적으로 생성된 페이지도 여전히 반응형입니다 - Next.js는 클라이언트 사이드에서 하이드레이션을 수행하여 완전한 상호작용성을 제공합니다.

- 정적 최적화된 페이지는 서버 측 계산이 필요 없습니다.

- CDN 위치에서 최종 사용자에게 즉시 스트리밍될 수 있어 **매우 빠른 로딩 경험**을 제공합니다.

## 작동 방식

Next.js는 페이지에 `getServerSideProps` 또는 `getInitialProps`가 있는지 여부에 따라 자동으로 페이지가 정적인지(빌드 시간에 사전 렌더링 가능한지) 판단합니다.

- **`getServerSideProps` 또는 `getInitialProps` 존재** → 요청 시 온디맨드 렌더링 (SSR)
- **위 함수 없음** → 자동 정적 최적화 (사전 렌더링)

### Query 객체

사전 렌더링 중에는 라우터의 `query` 객체가 비어있습니다. 이 단계에서는 query 정보가 없기 때문입니다.

하이드레이션 후 Next.js는 애플리케이션 업데이트를 트리거하여 `query` 객체에 라우트 파라미터를 제공합니다.

query가 하이드레이션 후 업데이트되어 다른 렌더링을 트리거하는 경우는 다음과 같습니다:

- 페이지가 [동적 라우트](/pages-router/building-your-application/routing/dynamic-routes.md)인 경우
- URL에 query 값이 있는 경우
- `next.config.js`에 [Rewrites](/app-router/api-reference/next-config/rewrites.md) 설정이 있는 경우 - query에 파싱되어 제공되어야 할 파라미터를 가질 수 있습니다

### router.isReady 사용

query가 완전히 업데이트되고 사용할 준비가 되었는지 구분하려면 `next/router`의 `isReady` 필드를 활용할 수 있습니다.

```jsx
import { useRouter } from 'next/router'

function MyComponent() {
  const router = useRouter()

  // router.isReady가 true일 때만 query 사용
  if (router.isReady) {
    // query 객체 사용 가능
    console.log(router.query)
  }

  return <div>...</div>
}
```

## 빌드 결과 확인

### 정적 최적화된 페이지

정적 최적화된 페이지는 `.html` 파일로 생성됩니다:

```bash
.next/server/pages/about.html
```

### getServerSideProps가 있는 페이지

`getServerSideProps`가 있는 페이지는 `.js` 파일로 생성되어 요청 시 실행됩니다:

```bash
.next/server/pages/about.js
```

## 주의사항

### Custom App의 getInitialProps

Custom `App`에 `getInitialProps`가 있으면 [정적 생성](/pages-router/api-reference/getStaticProps.md) 없는 페이지에서 이 최적화가 비활성화됩니다.

### Custom Document의 getInitialProps

Custom `Document`의 `getInitialProps`에서는 서버 사이드 렌더링인지 사전 렌더링인지 확인할 수 있습니다:

```jsx
// pages/_document.js
import Document from 'next/document'

class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx)

    if (ctx.req) {
      // 서버 사이드 렌더링
      console.log('서버에서 렌더링됨')
    } else {
      // 정적 최적화 (사전 렌더링)
      console.log('사전 렌더링됨')
    }

    return initialProps
  }
}

export default MyDocument
```

### asPath 사용 시 주의

정적으로 최적화된 페이지는 라우트 파라미터 없이 하이드레이션됩니다. 즉, `query`는 빈 객체(`{}`)입니다.

하이드레이션 후 Next.js는 애플리케이션 업데이트를 트리거하여 `query` 객체에 라우트 파라미터를 제공합니다.

`asPath`를 사용할 때는 반드시 `isReady`를 확인한 후 사용해야 합니다:

```jsx
import { useRouter } from 'next/router'

function MyComponent() {
  const router = useRouter()

  // ❌ 위험: isReady 확인 전 사용
  if (router.asPath === '/about') {
    // ...
  }

  // ✅ 안전: isReady 확인 후 사용
  if (router.isReady && router.asPath === '/about') {
    // ...
  }

  return <div>...</div>
}
```

정적 최적화 페이지에서는 서버가 클라이언트 측 `asPath`를 알지 못하므로, `asPath`를 prop으로 사용하면 하이드레이션 mismatch 에러가 발생할 수 있습니다.

## 정리

| 페이지 특성 | 렌더링 방식 | 결과 파일 |
|------------|------------|----------|
| `getServerSideProps` 없음 | 정적 최적화 | `.html` |
| `getServerSideProps` 있음 | 서버 사이드 렌더링 | `.js` |
| `getInitialProps` 있음 | 서버 사이드 렌더링 | `.js` |
| `getStaticProps` 있음 | 정적 생성 | `.html` + `.json` |
