# 지원 브라우저

Next.js는 설정 없이 **최신 브라우저**를 지원합니다.

- Chrome 111+
- Edge 111+
- Firefox 111+
- Safari 16.4+

## Browserslist

특정 브라우저나 기능을 대상으로 하려면 Next.js는 `package.json` 파일에서 [Browserslist](https://browsersl.ist) 설정을 지원합니다. Next.js는 기본적으로 다음 Browserslist 설정을 사용합니다:

```json filename="package.json"
{
  "browserslist": ["chrome 111", "edge 111", "firefox 111", "safari 16.4"]
}
```

## 폴리필

Next.js는 널리 사용되는 폴리필을 자동으로 주입합니다:

- [**fetch()**](https://developer.mozilla.org/docs/Web/API/Fetch_API) — `whatwg-fetch`와 `unfetch`를 대체합니다.
- [**URL**](https://developer.mozilla.org/docs/Web/API/URL) — [`url` 패키지 (Node.js API)](https://nodejs.org/api/url.html)를 대체합니다.
- [**Object.assign()**](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) — `object-assign`, `object.assign`, `core-js/object/assign`를 대체합니다.

종속성에 이러한 폴리필이 포함되어 있으면 중복을 피하기 위해 프로덕션 빌드에서 자동으로 제거됩니다.

또한 번들 크기를 줄이기 위해 Next.js는 이러한 폴리필이 필요한 브라우저에만 로드합니다. 전 세계 웹 트래픽의 대부분은 이러한 폴리필을 다운로드하지 않습니다.

### 커스텀 폴리필

자체 코드 또는 외부 npm 종속성에 대상 브라우저(예: IE 11)에서 지원되지 않는 기능이 필요한 경우 직접 폴리필을 추가해야 합니다.

#### App Router

`instrumentation-client.ts`에서 폴리필을 가져옵니다:

```ts filename="instrumentation-client.ts"
import './polyfills'
```

#### Pages Router

커스텀 `<App>`에서 최상위 import를 추가합니다:

```tsx filename="pages/_app.tsx"
import './polyfills'

import type { AppProps } from 'next/app'

export default function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

### 조건부 폴리필 로딩

필요한 경우에만 폴리필을 조건부로 로드합니다:

```ts
export const useAnalytics = () => {
  const tracker = useCallback(async (data: unknown) => {
    if (!('structuredClone' in globalThis)) {
      import('polyfills/structured-clone').then((mod) => {
        globalThis.structuredClone = mod.default
      })
    }
    /* structuredClone을 사용하는 작업 수행 */
  }, [])

  return tracker
}
```

## 지원되는 JavaScript 언어 기능

- Async/await (ES2017)
- Object Rest/Spread Properties (ES2018)
- Dynamic `import()` (ES2020)
- Optional Chaining (ES2020)
- Nullish Coalescing (ES2020)
- Class Fields 및 Static Properties (ES2022)
- 내장 TypeScript 지원
