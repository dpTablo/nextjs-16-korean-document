# webpack

> **알아두면 좋은 점**: webpack 설정 변경은 semver로 커버되지 않으므로 자신의 책임하에 진행하세요

애플리케이션에 커스텀 webpack 설정을 추가하기 전에, Next.js가 이미 사용 사례를 지원하지 않는지 확인하세요:

- [CSS 가져오기](/docs/app/building-your-application/styling/css)
- [CSS 모듈](/docs/app/building-your-application/styling/css#css-modules)
- [Sass/SCSS 가져오기](/docs/app/building-your-application/styling/sass)
- [Sass/SCSS 모듈](/docs/app/building-your-application/styling/sass)

자주 요청되는 일부 기능은 플러그인으로 제공됩니다:

- [@next/mdx](https://github.com/vercel/next.js/tree/canary/packages/next-mdx)
- [@next/bundle-analyzer](https://github.com/vercel/next.js/tree/canary/packages/next-bundle-analyzer)

`webpack` 사용을 확장하려면, `next.config.js` 내부에서 설정을 확장하는 함수를 정의하면 됩니다:

```js filename="next.config.js"
module.exports = {
  webpack: (
    config,
    { buildId, dev, isServer, defaultLoaders, nextRuntime, webpack }
  ) => {
    // 중요: 수정된 config를 반환하세요
    return config
  },
}
```

> `webpack` 함수는 서버(nodejs / edge 런타임)에 대해 두 번, 클라이언트에 대해 한 번, 총 세 번 실행됩니다. 이를 통해 `isServer` 속성을 사용하여 클라이언트와 서버 설정을 구분할 수 있습니다.

`webpack` 함수의 두 번째 인수는 다음 속성을 가진 객체입니다:

- `buildId`: `String` - 빌드 간의 고유 식별자로 사용되는 빌드 ID
- `dev`: `Boolean` - 개발 모드에서 컴파일이 수행될지 여부를 나타냅니다
- `isServer`: `Boolean` - 서버 사이드 컴파일이면 `true`, 클라이언트 사이드 컴파일이면 `false`
- `nextRuntime`: `String | undefined` - 서버 사이드 컴파일의 대상 런타임; `"edge"` 또는 `"nodejs"` 중 하나이며, 클라이언트 사이드 컴파일의 경우 `undefined`
- `defaultLoaders`: `Object` - Next.js가 내부적으로 사용하는 기본 로더들:
  - `babel`: `Object` - 기본 `babel-loader` 설정

`defaultLoaders.babel` 사용 예제:

```js filename="next.config.js"
// @mdx-js/loader에 의존하는 @next/mdx 플러그인 소스에서 가져온 설정 예제입니다.
// https://github.com/vercel/next.js/tree/canary/packages/next-mdx
module.exports = {
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\.mdx/,
      use: [
        options.defaultLoaders.babel,
        {
          loader: '@mdx-js/loader',
          options: pluginOptions.options,
        },
      ],
    })

    return config
  },
}
```

## nextRuntime

`isServer`가 `true`인 경우 `nextRuntime`이 `"edge"` 또는 `"nodejs"`인지 확인하세요. `"edge"` 값의 `nextRuntime`은 현재 edge 런타임의 미들웨어와 서버 컴포넌트에만 해당됩니다.

## 중요 참고 사항

- webpack 설정 변경은 Next.js의 semver 정책에 포함되지 않습니다
- 수정된 config 객체를 항상 반환해야 합니다
- 이 패턴은 `@next/mdx`와 같은 플러그인에서 일반적으로 사용됩니다
