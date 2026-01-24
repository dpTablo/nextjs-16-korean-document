# typedRoutes

> **참고**: 이 옵션은 안정화되었으므로 `experimental.typedRoutes` 대신 `typedRoutes`를 사용해야 합니다.

## 개요

`typedRoutes` 옵션은 Next.js 프로젝트에서 [정적으로 타입된 링크](/app-router/api-reference/config/typescript.md#statically-typed-links)를 지원합니다.

이 기능을 사용하려면 프로젝트에서 TypeScript를 사용해야 합니다.

## 설정

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  typedRoutes: true,
}

module.exports = nextConfig
```

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typedRoutes: true,
}

export default nextConfig
```

## 기능

`typedRoutes`가 활성화되면:

1. Next.js가 `.next/types` 폴더에 라우트 정의 파일을 생성합니다
2. `Link` 컴포넌트의 `href` prop에 타입 안전성이 적용됩니다
3. 존재하지 않는 라우트로의 링크는 TypeScript 오류를 발생시킵니다

## 예제

```tsx
import Link from 'next/link'

export default function Page() {
  return (
    <>
      {/* 유효한 라우트 - 타입 체크 통과 */}
      <Link href="/about">About</Link>

      {/* 유효하지 않은 라우트 - TypeScript 오류 발생 */}
      <Link href="/invalid-route">Invalid</Link>
    </>
  )
}
```

## 장점

- **개발 시 오류 감지**: 잘못된 라우트 링크를 빌드 전에 발견
- **자동 완성**: IDE에서 라우트 경로 자동 완성 지원
- **리팩토링 안전성**: 라우트 변경 시 관련 링크를 쉽게 찾아 수정
