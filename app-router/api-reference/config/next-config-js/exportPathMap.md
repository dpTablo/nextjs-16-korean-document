# exportPathMap

> **경고**: `exportPathMap`은 `pages`에서 `getStaticPaths`를 사용하기 위한 레거시 API입니다. `pages/`에서 미리 렌더링할 페이지 목록을 지정하기 위해 `getStaticPaths`를 사용하거나 App Router에서 `generateStaticParams`를 사용하는 것이 좋습니다.

`exportPathMap`을 사용하면 내보내기 중에 사용할 요청 경로와 페이지 대상의 매핑을 지정할 수 있습니다. `exportPathMap`에 정의된 경로는 [`next dev`](/docs/app/api-reference/cli/next#next-dev-options)를 사용할 때도 사용할 수 있습니다.

다음 페이지가 있는 앱에 대해 커스텀 `exportPathMap`을 만드는 예제로 시작하겠습니다:

- `pages/index.js`
- `pages/about.js`
- `pages/post.js`

`next.config.js`를 열고 다음 `exportPathMap` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  exportPathMap: async function (
    defaultPathMap,
    { dev, dir, outDir, distDir, buildId }
  ) {
    return {
      '/': { page: '/' },
      '/about': { page: '/about' },
      '/p/hello-nextjs': { page: '/post', query: { title: 'hello-nextjs' } },
      '/p/learn-nextjs': { page: '/post', query: { title: 'learn-nextjs' } },
      '/p/deploy-nextjs': { page: '/post', query: { title: 'deploy-nextjs' } },
    }
  },
}
```

## 함수 매개변수

`exportPathMap` 함수는 다음을 받습니다:

1. **defaultPathMap** - Next.js가 사용하는 기본 맵
2. **옵션 객체**:
   - `dev` - 개발 모드에서 `true`, `next export` 실행 시 `false`
   - `dir` - 프로젝트 디렉토리의 절대 경로
   - `outDir` - `out/` 디렉토리의 절대 경로 (dev가 true일 때 null)
   - `distDir` - `.next/` 디렉토리의 절대 경로
   - `buildId` - 생성된 빌드 ID

## 반환 객체 구조

반환된 맵은 다음을 사용합니다:

- **키**: `pathname` (URL 경로)
- **값**: 다음을 포함하는 객체:
  - `page`: String - 렌더링할 `pages` 디렉토리 내의 페이지 경로
  - `query`: Object - `getInitialProps`에 전달되는 쿼리 객체 (기본값 `{}`)

## 중요한 제한 사항

> **알아두면 좋은 점**: `query` 필드는 자동으로 정적 최적화된 페이지나 `getStaticProps` 페이지에서 사용할 수 없습니다. 이러한 페이지는 빌드 시점에 HTML로 렌더링되며 추가 쿼리 정보를 제공할 수 없습니다.

## 상태

이 API는 더 이상 사용되지 않습니다. pages router에서는 `getStaticPaths`를, app router에서는 `generateStaticParams`를 대신 사용하세요.
