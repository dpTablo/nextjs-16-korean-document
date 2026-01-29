---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/basePath
버전: 16.1.6
---

# basePath

도메인의 하위 경로에 Next.js 애플리케이션을 배포하려면 `basePath` 구성 옵션을 사용하면 됩니다.

`basePath`는 애플리케이션의 경로 접두사를 설정할 수 있게 해줍니다. 예를 들어, `/docs`를 사용하려면 (기본값인 `/` 대신) `next.config.js`를 열고 `basePath` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  basePath: '/docs',
}
```

> **알아두면 좋은 점**: 이 값은 빌드 시점에 설정되어야 하며 클라이언트 사이드 번들에 값이 인라인되므로 다시 빌드하지 않고는 변경할 수 없습니다.

## 링크

`next/link`와 `next/router`를 사용하여 다른 페이지로 링크할 때 `basePath`가 자동으로 적용됩니다.

예를 들어, `/about`을 사용하면 `basePath`가 `/docs`로 설정된 경우 자동으로 `/docs/about`이 됩니다.

```js
export default function HomePage() {
  return (
    <>
      <Link href="/about">About Page</Link>
    </>
  )
}
```

출력 html:

```html
<a href="/docs/about">About Page</a>
```

이렇게 하면 `basePath` 값을 변경할 때 애플리케이션의 모든 링크를 변경할 필요가 없습니다.

## 이미지

[`next/image`](/docs/app/api-reference/components/image) 컴포넌트를 사용할 때는 `src` 앞에 `basePath`를 추가해야 합니다.

예를 들어, `basePath`가 `/docs`로 설정된 경우 `/docs/me.png`를 사용하면 이미지가 올바르게 제공됩니다.

```jsx
import Image from 'next/image'

function Home() {
  return (
    <>
      <h1>My Homepage</h1>
      <Image
        src="/docs/me.png"
        alt="Picture of the author"
        width={500}
        height={500}
      />
      <p>Welcome to my homepage!</p>
    </>
  )
}

export default Home
```

> **알아두면 좋은 점**: 링크와 달리 이미지는 `basePath` 접두사가 포함된 전체 경로를 명시적으로 포함해야 합니다.
