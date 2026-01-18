# pageExtensions

기본적으로 Next.js는 다음 확장자를 가진 파일을 페이지로 허용합니다: `.tsx`, `.ts`, `.jsx`, `.js`. 이것은 마크다운(`.md`, `.mdx`)과 같은 다른 확장자를 허용하도록 수정할 수 있습니다.

```js filename="next.config.js"
const withMDX = require('@next/mdx')()

/** @type {import('next').NextConfig} */
const nextConfig = {
  pageExtensions: ['js', 'jsx', 'ts', 'tsx', 'md', 'mdx'],
}

module.exports = withMDX(nextConfig)
```

## 사용 사례

- 콘텐츠 기반 애플리케이션을 위한 마크다운(`.md`) 또는 MDX(`.mdx`) 지원 추가
- 다른 커스텀 파일 형식을 페이지 소스로 지원

## 알아두면 좋은 점

- `pageExtensions` 배열은 Next.js가 유효한 페이지 파일로 인식할 모든 파일 확장자를 지정합니다
- MDX 파일을 처리하려면 `withMDX()` 래퍼가 필요합니다
- 기본 확장자(`.js`, `.jsx`, `.ts`, `.tsx`)도 배열에 포함해야 합니다
