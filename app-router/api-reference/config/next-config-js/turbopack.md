# turbopack

`turbopack` 옵션을 사용하면 다양한 파일을 변환하고 모듈 해석 방식을 변경하기 위해 Turbopack을 커스터마이징할 수 있습니다.

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  turbopack: {
    // ...
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  turbopack: {
    // ...
  },
}

module.exports = nextConfig
```

> **알아두면 좋은 점**:
>
> - Next.js용 Turbopack은 내장 기능을 위한 로더나 로더 설정이 필요하지 않습니다. Turbopack은 CSS와 최신 JavaScript 컴파일을 위한 내장 지원이 있으므로, `@babel/preset-env`를 사용한다면 `css-loader`, `postcss-loader`, 또는 `babel-loader`가 필요하지 않습니다.

## 참조

다음 옵션은 `turbopack` 설정에 사용할 수 있습니다:

| 옵션                | 설명                                                          |
| ------------------- | ------------------------------------------------------------- |
| `root`              | 애플리케이션의 루트 디렉토리를 설정합니다. 절대 경로여야 합니다 |
| `rules`             | Turbopack에 적용할 지원되는 webpack 로더 목록                  |
| `resolveAlias`      | 별칭 임포트를 모듈에 매핑합니다                                |
| `resolveExtensions` | 파일 임포트 시 해석할 확장자 목록                              |
| `debugIds`          | JavaScript 번들 및 소스맵에 디버그 ID 생성 활성화              |

### `root`

Turbopack이 파일을 해석하는 데 사용하는 애플리케이션의 루트 디렉토리입니다. 절대 경로여야 합니다.

```js filename="next.config.js"
const path = require('path')

module.exports = {
  turbopack: {
    root: path.join(__dirname, '..'),
  },
}
```

Next.js는 잠금 파일(`pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, `bun.lock`, `bun.lockb`)의 위치를 기반으로 루트를 자동 감지합니다.

`npm link`, `yarn link`, `pnpm link`, 또는 유사한 것과 연결된 종속성이 있는 경우 잠금 파일이 없으면 감지되지 않을 수 있습니다. 이 경우 프로젝트와 연결된 패키지의 공통 부모 디렉토리로 `root`를 설정해야 합니다.

### `rules`

`rules`를 사용하면 파일 변환을 위해 Turbopack과 함께 webpack 로더를 사용할 수 있습니다.

> **알아두면 좋은 점**: 기본적으로 프로젝트에 `babel.config.js`나 `.babelrc` 파일이 있으면 Turbopack은 이를 감지하고 `babel-loader`를 자동으로 설정합니다.

#### 지원되는 로더

다음 로더는 Turbopack의 webpack 로더 구현으로 작동하도록 테스트되었습니다:

- [`babel-loader`](https://www.npmjs.com/package/babel-loader)
- [`@svgr/webpack`](https://www.npmjs.com/package/@svgr/webpack)
- [`svg-inline-loader`](https://www.npmjs.com/package/svg-inline-loader)
- [`yaml-loader`](https://www.npmjs.com/package/yaml-loader)
- [`string-replace-loader`](https://www.npmjs.com/package/string-replace-loader)
- [`raw-loader`](https://www.npmjs.com/package/raw-loader)
- [`sass-loader`](https://www.npmjs.com/package/sass-loader)
- [`graphql-tag/loader`](https://www.npmjs.com/package/graphql-tag)

#### 설정

로더를 사용하도록 설정하려면, 설치한 로더의 이름과 옵션을 `next.config.js`에 추가하여 파일 확장자를 로더 목록에 매핑합니다:

```js filename="next.config.js"
module.exports = {
  turbopack: {
    rules: {
      '*.svg': {
        loaders: ['@svgr/webpack'],
        as: '*.js',
      },
    },
  },
}
```

위 예제의 경우 `loaders`는 로더 이름이나 로더 이름과 옵션을 함께 지정할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  turbopack: {
    rules: {
      '*.svg': {
        loaders: [
          {
            loader: '@svgr/webpack',
            options: {
              icon: true,
            },
          },
        ],
        as: '*.js',
      },
    },
  },
}
```

> **알아두면 좋은 점**: Next.js 버전 13.4.4 이전에는 `turbo.rules`가 `turbo.loaders`로 명명되었으며 `*.mdx`와 같은 파일 확장자만 허용했습니다.

#### 조건부 로더

로더가 적용되어야 하는 시점을 더 세밀하게 제어하려면 `condition` 속성을 사용하세요. 이를 통해 경로 및 콘텐츠별로 파일 필터링을 수행하고, 일치하는 환경에서 로더를 적용하지 않도록 브라우저/노드/엣지 환경별로 선택할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  turbopack: {
    rules: {
      '*': {
        condition: {
          all: [
            // 'foreign'은 node_modules의 코드를 일치시킵니다
            { not: 'foreign' },
            { path: /^img\/[0-9]{3}\// },
            {
              // any는 OR이고, all은 AND입니다
              any: [
                { path: '*.svg' },
                // content는 파일 내용을 일치시킵니다
                { content: /\<svg\W/ },
              ],
            },
          ],
        },
        loaders: ['@svgr/webpack'],
        as: '*.js',
      },
    },
  },
}
```

##### 조건 연산자

조건은 두 가지 유형으로 나뉩니다:

- **불리언**: 이 구조를 사용하여 조건을 결합합니다. `{all: [조건, ...]}` - AND, `{any: [조건, ...]}` - OR, `{not: 조건}` - NOT.
- **커스터마이징 가능**: 문자열(glob으로 처리) 또는 정규식을 값으로 지정할 수 있습니다. `{path: 문자열 | RegExp}`, `{content: RegExp}`, `{query: 문자열 | RegExp}`.

##### 내장 조건

일부 조건은 문자열로 사용 가능합니다:

- `browser` - 클라이언트 사이드 코드에서 일치합니다.
- `foreign` - `node_modules`의 코드에서 일치합니다.
- `development` - `next dev`를 사용할 때 일치합니다.
- `production` - `next build`를 사용할 때 일치합니다.
- `node` - 기본 Node.js 런타임 코드에서 일치합니다.
- `edge-light` - Edge 런타임 코드에서 일치합니다.

#### 분리된 조건이 있는 다중 규칙

패턴에 대해 여러 규칙이 필요한 경우 규칙 배열을 지정할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  turbopack: {
    rules: {
      '*.svg': [
        {
          condition: 'browser',
          loaders: ['@svgr/webpack'],
          as: '*.js',
        },
        {
          condition: { not: 'browser' },
          loaders: [require.resolve('./custom-svg-loader.js')],
          as: '*.js',
        },
      ],
    },
  },
}
```

### `resolveAlias`

`resolveAlias`를 통해 Turbopack을 설정하여 별칭 임포트를 다른 모듈로 로드하도록 모듈 해석을 수정할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  turbopack: {
    resolveAlias: {
      underscore: 'lodash',
      mocha: { browser: 'mocha/browser-entry.js' },
    },
  },
}
```

이 별칭은 `underscore` 패키지 임포트를 `lodash` 패키지로 대체합니다. 다시 말해, `import underscore from 'underscore'`는 `underscore` 대신 `lodash` 모듈을 로드합니다.

Turbopack은 Node.js의 [조건부 내보내기](https://nodejs.org/docs/latest-v18.x/api/packages.html#conditional-exports)와 유사하게 이 필드를 통한 조건부 별칭 지정도 지원합니다. 현재는 `browser` 조건만 지원됩니다. 위의 경우 Turbopack이 브라우저 환경을 타겟팅할 때 `mocha` 모듈의 임포트는 `mocha/browser-entry.js`로 별칭됩니다.

### `resolveExtensions`

`resolveExtensions`를 통해 Turbopack을 설정하여 커스텀 확장자로 모듈을 해석할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  turbopack: {
    resolveExtensions: ['.mdx', '.tsx', '.ts', '.jsx', '.js', '.mjs', '.json'],
  },
}
```

이것은 원래 해석 확장자를 제공된 목록으로 덮어씁니다. 기본 확장자를 포함하도록 하세요.

webpack에서 Turbopack으로 앱을 마이그레이션하는 방법에 대한 자세한 정보와 지침은 [webpack에서 Turbopack으로 마이그레이션 문서](https://turbo.build/pack/docs/migrating-from-webpack)를 참조하세요.

### `debugIds`

JavaScript 번들 및 소스맵에 디버그 ID 생성을 활성화하려면 `debugIds` 옵션을 `true`로 설정하세요.

```js filename="next.config.js"
module.exports = {
  turbopack: {
    debugIds: true,
  },
}
```

이것은 Sentry와 같은 외부 도구가 소스맵과 스택 트레이스를 상관시키는 데 도움이 됩니다. 디버그 ID는 브라우저에서 `globalThis._debugIds`에서 찾을 수 있습니다.

## 버전 기록

| 버전     | 변경 사항                                                   |
| -------- | ----------------------------------------------------------- |
| `16.2.0` | `turbopack.rules.*.condition.query` 추가                    |
| `16.0.0` | `turbopack.debugIds`와 `turbopack.rules.*.condition` 추가   |
| `15.3.0` | `experimental.turbo`가 `turbopack`으로 변경됨               |
| `13.0.0` | `experimental.turbo` 도입                                   |
