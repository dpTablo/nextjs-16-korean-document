# Next.js 컴파일러

[SWC](https://swc.rs/)를 사용하여 Rust로 작성된 Next.js 컴파일러는 프로덕션을 위해 JavaScript를 변환하고 최소화합니다. 이는 개별 파일을 위한 Babel과 출력 번들 최소화를 위한 Terser를 대체합니다.

Next.js 컴파일러를 사용한 컴파일은 Babel보다 **17배 빠르며** Next.js 버전 12부터 기본적으로 활성화됩니다. 기존 Babel 설정이 있거나 [지원되지 않는 기능](#지원되지-않는-기능)을 사용하는 경우, 애플리케이션은 Next.js 컴파일러를 사용하지 않고 Babel을 계속 사용합니다.

## 왜 SWC인가?

[SWC](https://swc.rs/)는 차세대 빠른 개발자 도구를 위한 확장 가능한 Rust 기반 플랫폼입니다.

SWC는 컴파일, 최소화, 번들링 등에 사용될 수 있으며 확장되도록 설계되었습니다. 코드 변환(내장 또는 커스텀)을 수행하기 위해 호출할 수 있습니다. 이러한 변환을 실행하는 것은 Next.js와 같은 상위 수준 도구를 통해 이루어집니다.

몇 가지 이유로 SWC를 선택했습니다:

- **확장성:** SWC는 라이브러리를 포크하거나 설계 제약을 우회하지 않고도 Next.js 내부에서 Crate로 사용할 수 있습니다.
- **성능:** SWC로 전환한 후 Next.js에서 약 3배 빠른 Fast Refresh와 약 5배 빠른 빌드를 달성할 수 있었으며, 아직 최적화 여지가 더 있습니다.
- **WebAssembly:** Rust의 WASM 지원은 모든 가능한 플랫폼을 지원하고 어디서나 Next.js 개발을 가능하게 하는 데 필수적입니다.
- **커뮤니티:** Rust 커뮤니티와 생태계가 성장하고 있습니다.

## 지원되는 기능

### Styled Components

`babel-plugin-styled-components`를 Next.js 컴파일러로 포팅하고 있습니다.

먼저 Next.js 최신 버전으로 업데이트하세요. 그런 다음 `next.config.js` 파일을 업데이트하세요:

```js filename="next.config.js"
module.exports = {
  compiler: {
    styledComponents: true,
  },
}
```

고급 styled-components 사용의 경우 다음과 같이 구성할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  compiler: {
    styledComponents: {
      displayName: true,
      ssr: true,
      fileName: true,
      topLevelImportPaths: [],
      meaninglessFileNames: [],
      minify: true,
      transpileTemplateLiterals: true,
      namespace: '',
      pure: true,
      cssProp: true,
    },
  },
}
```

### Jest

Next.js 컴파일러는 테스트를 트랜스파일하고 다음을 포함하여 Next.js와 함께 Jest 구성을 간소화합니다:

- `.css`, `.module.css`(및 `.scss` 변형), 이미지 가져오기 자동 모킹
- `transform`을 SWC로 자동 설정
- `.env` 파일 로드
- 테스트 해석 및 변환에서 `node_modules` 제외
- 테스트 해석에서 `.next` 제외
- 실험적 SWC 변환을 활성화하는 플래그를 위해 `next.config.js` 로드

```js filename="jest.config.js"
const nextJest = require('next/jest')

const createJestConfig = nextJest({ dir: './' })

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
}

module.exports = createJestConfig(customJestConfig)
```

### Relay

[Relay](https://relay.dev/) 지원을 활성화하려면:

```js filename="next.config.js"
module.exports = {
  compiler: {
    relay: {
      src: './',
      artifactDirectory: './__generated__',
      language: 'typescript',
      eagerEsModules: false,
    },
  },
}
```

### React 속성 제거

JSX 속성을 제거할 수 있습니다. 테스트에 자주 사용됩니다. `babel-plugin-react-remove-properties`와 유사합니다.

기본 정규식 `^data-test`와 일치하는 속성을 제거하려면:

```js filename="next.config.js"
module.exports = {
  compiler: {
    reactRemoveProperties: true,
  },
}
```

커스텀 속성을 제거하려면:

```js filename="next.config.js"
module.exports = {
  compiler: {
    reactRemoveProperties: { properties: ['^data-custom$'] },
  },
}
```

### Console 제거

이 변환은 애플리케이션 코드에서 모든 `console.*` 호출을 제거합니다(`node_modules` 제외).

모든 `console.*` 호출을 제거하려면:

```js filename="next.config.js"
module.exports = {
  compiler: {
    removeConsole: true,
  },
}
```

`console.error`를 제외한 `console.*` 출력을 제거하려면:

```js filename="next.config.js"
module.exports = {
  compiler: {
    removeConsole: {
      exclude: ['error'],
    },
  },
}
```

### 레거시 데코레이터

Next.js는 `jsconfig.json` 또는 `tsconfig.json`에서 `experimentalDecorators`를 자동으로 감지합니다. 레거시 데코레이터는 `mobx`와 같은 오래된 버전의 라이브러리에서 일반적으로 사용됩니다.

### Emotion

```js filename="next.config.js"
module.exports = {
  compiler: {
    emotion: {
      sourceMap: true,
      autoLabel: 'dev-only',
      labelFormat: '[local]',
    },
  },
}
```

### 최소화

Next.js의 SWC 컴파일러는 v13부터 기본적으로 최소화에 사용됩니다. 이는 Terser보다 7배 빠릅니다.

### 모듈 트랜스파일

Next.js는 로컬 패키지(예: 모노레포) 또는 외부 종속성(`node_modules`)의 종속성을 자동으로 트랜스파일하고 번들할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  transpilePackages: ['@acme/ui', 'lodash-es'],
}
```

### Define (빌드 타임 변수 교체)

```js filename="next.config.js"
module.exports = {
  compiler: {
    define: {
      MY_VARIABLE: 'my-string',
      'process.env.MY_ENV_VAR': 'my-env-var',
    },
  },
}
```

## 실험적 기능

### SWC 트레이스 프로파일링

```js filename="next.config.js"
module.exports = {
  experimental: {
    swcTraceProfiling: true,
  },
}
```

Chrome DevTools 또는 Perfetto에서 볼 수 있는 `swc-trace-profile-${timestamp}.json`을 생성합니다.

### SWC 플러그인

```js filename="next.config.js"
module.exports = {
  experimental: {
    swcPlugins: [
      [
        'plugin-name',
        {
          // 플러그인 옵션
        },
      ],
    ],
  },
}
```

## 지원되지 않는 기능

애플리케이션에 `.babelrc` 파일이 있으면 자동으로 Babel로 대체됩니다. 이는 커스텀 Babel 플러그인과의 하위 호환성을 보장합니다.

## 버전 기록

| 버전      | 변경 사항                                    |
| --------- | -------------------------------------------- |
| `v13.1.0` | 모듈 트랜스파일 안정화                       |
| `v13.0.0` | SWC 최소화기 기본 활성화                     |
| `v12.3.0` | SWC 최소화기 안정화                          |
| `v12.2.0` | SWC 플러그인 실험적 지원                     |
| `v12.1.0` | Styled Components, Jest, Relay 등 지원 추가 |
| `v12.0.0` | Next.js 컴파일러 도입                        |
