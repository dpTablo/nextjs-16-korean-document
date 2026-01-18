# sassOptions

`sassOptions`를 사용하면 Sass 컴파일러를 구성할 수 있습니다.

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const sassOptions = {
  additionalData: `
    $var: red;
  `,
}

const nextConfig: NextConfig = {
  sassOptions: {
    ...sassOptions,
    implementation: 'sass-embedded',
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */

const sassOptions = {
  additionalData: `
    $var: red;
  `,
}

const nextConfig = {
  sassOptions: {
    ...sassOptions,
    implementation: 'sass-embedded',
  },
}

module.exports = nextConfig
```

## 일반적인 옵션

- **`additionalData`**: 전역적으로 추가 Sass 변수나 imports를 주입합니다
- **`implementation`**: Sass 구현을 지정합니다 (예: `'sass-embedded'`)

## 중요 참고 사항

> **알아두면 좋은 점**: `sassOptions`는 Next.js가 다른 가능한 속성을 유지하지 않기 때문에 `implementation` 속성 외에는 완전히 타입이 지정되지 않습니다.

> **알아두면 좋은 점**: 커스텀 Sass 함수를 정의하기 위한 `functions` 속성은 webpack에서만 지원됩니다. Turbopack(Rust 기반 아키텍처)을 사용할 때는 이 옵션을 통해 JavaScript 함수를 직접 실행할 수 없기 때문에 커스텀 Sass 함수를 사용할 수 없습니다.
