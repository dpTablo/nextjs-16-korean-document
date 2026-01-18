# transpilePackages

Next.js는 로컬 패키지(예: 모노레포) 또는 외부 종속성(`node_modules`)의 종속성을 자동으로 트랜스파일하고 번들할 수 있습니다. 이것은 `next-transpile-modules` 패키지를 대체합니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  transpilePackages: ['package-name'],
}

module.exports = nextConfig
```

## 주요 기능

- 로컬 패키지 종속성을 자동으로 트랜스파일
- 모노레포 구조 지원
- `node_modules`의 외부 종속성 처리
- `next-transpile-modules`의 내장 대체

## 구성 세부 사항

- **매개변수**: 패키지 이름의 배열 (문자열)
- **타입**: `string[]`
- **예제**: `transpilePackages: ['my-package', 'another-package']`

## 버전 기록

| 버전     | 변경 사항                  |
| -------- | -------------------------- |
| `v13.0.0` | `transpilePackages` 추가됨 |
