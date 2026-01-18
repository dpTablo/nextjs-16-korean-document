# mdxRs

`mdxRs`는 기본 JavaScript 컴파일러 대신 새로운 Rust 기반 컴파일러를 사용하여 MDX 파일을 컴파일할 수 있게 해주는 실험적 기능입니다.

## 상태

- **안정성**: 실험적이며 변경될 수 있음
- **프로덕션 준비**: 프로덕션 사용에 권장되지 않음

## 사용법

mdxRs를 활성화하려면 Next.js 구성 파일에 추가하세요:

```js filename="next.config.js"
const withMDX = require('@next/mdx')()

/** @type {import('next').NextConfig} */
const nextConfig = {
  pageExtensions: ['ts', 'tsx', 'mdx'],
  experimental: {
    mdxRs: true,
  },
}

module.exports = withMDX(nextConfig)
```

## 요구 사항

- `@next/mdx` 패키지와 함께 사용해야 합니다
- `.mdx` 파일을 인식하려면 적절한 `pageExtensions` 구성이 필요합니다

## 주요 포인트

- 이 기능은 실험적이며 테스트/개발 목적으로만 사용해야 합니다
- 프로덕션 환경에는 권장되지 않습니다
- 향후 릴리스에서 발전하고 잠재적으로 변경될 것으로 예상됩니다
- MDX Next.js 플러그인과 함께 작동합니다
