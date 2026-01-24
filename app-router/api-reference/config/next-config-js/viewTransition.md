# viewTransition

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

`viewTransition`은 React에서 새로운 [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)를 활성화하는 실험적 플래그입니다. 이 API는 네이티브 View Transitions 브라우저 API를 활용하여 UI 상태 간의 매끄러운 전환을 생성할 수 있습니다.

## 설정

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    viewTransition: true,
  },
}

module.exports = nextConfig
```

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    viewTransition: true,
  },
}

export default nextConfig
```

## 사용 방법

React에서 `<ViewTransition>` 컴포넌트를 import하여 사용할 수 있습니다:

```jsx
import { ViewTransition } from 'react'

export default function Page() {
  return (
    <ViewTransition>
      <div>전환 효과가 적용되는 콘텐츠</div>
    </ViewTransition>
  )
}
```

## 중요 사항

- `<ViewTransition>` 컴포넌트는 이미 React의 Canary 릴리스 채널에서 사용 가능합니다
- `experimental.viewTransition`은 Next.js 기능과의 깊은 통합을 위해서만 필요합니다 (예: 네비게이션을 위한 Transition types 자동 추가)
- Next.js 특정 전환 타입은 아직 구현되지 않았습니다

## 데모

[Next.js View Transition Demo](https://view-transition-example.vercel.app)에서 이 기능을 실제로 확인할 수 있습니다.

## 주의사항

API가 진화함에 따라 문서가 업데이트되고 더 많은 예제가 공유될 예정입니다. 현재로서는 이 기능을 프로덕션에서 사용하지 않을 것을 강력히 권고합니다.
