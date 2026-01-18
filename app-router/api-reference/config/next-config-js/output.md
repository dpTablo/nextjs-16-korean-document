# output

빌드하는 동안 Next.js는 각 페이지와 그 종속성을 자동으로 추적하여 애플리케이션의 프로덕션 버전을 배포하는 데 필요한 모든 파일을 결정합니다.

이 기능은 배포 크기를 대폭 줄이는 데 도움이 됩니다. 이전에는 Docker로 배포할 때 패키지의 `dependencies`에 있는 모든 파일을 설치해야 `next start`를 실행할 수 있었습니다. Next.js 12부터는 `.next/` 디렉토리의 출력 파일 추적을 활용하여 필요한 파일만 포함할 수 있습니다.

또한, 더 이상 사용되지 않는 `serverless` 대상이 필요 없어집니다. 이 대상은 다양한 문제를 일으킬 수 있고 불필요한 중복을 생성합니다.

## 작동 방식

`next build` 중에 Next.js는 [`@vercel/nft`](https://github.com/vercel/nft)를 사용하여 `import`, `require`, `fs` 사용을 정적으로 분석하여 페이지가 로드할 수 있는 모든 파일을 결정합니다.

Next.js의 프로덕션 서버도 필요한 파일을 추적하고 `.next/next-server.js.nft.json`에 출력하여 프로덕션에서 활용할 수 있습니다.

`.next` 출력 디렉토리에 출력된 `.nft.json` 파일을 활용하려면 각 추적에서 `.nft.json` 파일에 상대적인 파일 목록을 읽고 배포 위치에 복사하면 됩니다.

## 필요한 파일 자동 복사

Next.js는 프로덕션 배포에 필요한 파일만 복사하는 `standalone` 폴더를 자동으로 생성할 수 있으며, 여기에는 `node_modules`의 선택된 파일도 포함됩니다.

이 자동 복사를 활용하려면 `next.config.js`에서 활성화할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  output: 'standalone',
}
```

이렇게 하면 `.next/standalone`에 `node_modules`를 설치하지 않고도 자체적으로 배포할 수 있는 폴더가 생성됩니다.

또한, `next start` 대신 사용할 수 있는 최소한의 `server.js` 파일도 출력됩니다. 이 최소 서버는 기본적으로 `public`이나 `.next/static` 폴더를 복사하지 않습니다. 이러한 폴더는 CDN에서 처리하는 것이 이상적이기 때문입니다. 하지만 이 폴더들을 `standalone/public`과 `standalone/.next/static` 폴더에 수동으로 복사한 후 `server.js` 파일이 자동으로 제공하도록 할 수 있습니다.

수동으로 복사하려면:

```bash
cp -r public .next/standalone/ && cp -r .next/static .next/standalone/.next/
```

서버를 시작하려면:

```bash
node .next/standalone/server.js
```

커스텀 포트와 호스트네임으로:

```bash
PORT=8080 HOSTNAME=0.0.0.0 node server.js
```

> **알아두면 좋은 점**:
>
> - `next.config.js`는 `next build` 중에 읽히고 `server.js` 출력 파일에 직렬화됩니다. 레거시 [`serverRuntimeConfig` 또는 `publicRuntimeConfig` 옵션](/docs/pages/api-reference/config/next-config-js/runtime-configuration)을 사용하는 경우 값은 빌드 시점의 값이 적용됩니다.
> - 프로젝트가 기본 `loader`로 이미지 최적화를 사용하는 경우 `sharp`를 종속성으로 설치해야 합니다:

```bash
npm i sharp
```

```bash
yarn add sharp
```

```bash
pnpm add sharp
```

```bash
bun add sharp
```

## 구성 옵션

### outputFileTracingRoot

모노레포와 같이 Next.js 프로젝트 외부의 파일을 추적해야 하는 경우가 있습니다.

`outputFileTracingRoot`를 사용하여 추적을 시작하는 데 사용할 디렉토리를 지정할 수 있습니다.

```js filename="packages/web-app/next.config.js"
const path = require('path')

module.exports = {
  outputFileTracingRoot: path.join(__dirname, '../../'),
}
```

### outputFileTracingExcludes

라우트 glob을 키로 사용하고 glob 배열을 값으로 사용하여 특정 파일을 추적에서 제외합니다.

```js filename="next.config.js"
module.exports = {
  outputFileTracingExcludes: {
    '/api/hello': ['./un-necessary-folder/**/*'],
    '/api/*': ['src/temp/**/*', 'public/large-logs/**/*'],
  },
}
```

### outputFileTracingIncludes

추적에서 놓친 파일을 포함합니다.

```js filename="next.config.js"
module.exports = {
  outputFileTracingIncludes: {
    '/api/another': ['./necessary-folder/**/*'],
    '/api/login/\\[\\[\\.\\.\\.slug\\]\\]': [
      './node_modules/aws-crt/dist/bin/**/*',
    ],
    '/products/*': ['src/lib/payments/**/*'],
    '/*': ['src/config/runtime/**/*.json'],
  },
}
```

### 글로벌 라우트

`'/*'`를 사용하여 모든 라우트를 대상으로 합니다:

```js filename="next.config.js"
module.exports = {
  outputFileTracingIncludes: {
    '/*': ['src/i18n/locales/**/*.json'],
  },
}
```

## 중요 참고 사항

- include/exclude 옵션의 **키**는 라우트 경로와 매칭됩니다 (`'/api/hello'`, `'/products/[id]'` 등)
- **값**은 프로젝트 루트에서 해석되는 glob 패턴입니다 (`src/` 디렉토리와 함께 작동)
- 라우트 glob은 [picomatch](https://www.npmjs.com/package/picomatch#basic-globbing) 매칭을 사용합니다
- 이 옵션은 **서버 추적**에만 적용됩니다:
  - Edge Runtime 라우트에는 적용되지 않음
  - 완전히 정적인 페이지에는 적용되지 않음
- 크로스 플랫폼 호환성을 위해 패턴에 슬래시(`/`)를 사용하세요
- 과도하게 큰 추적을 피하기 위해 패턴을 좁게 유지하세요 (저장소 루트에서 `**/*` 사용 피하기)
