# reactMaxHeadersLength

`reactMaxHeadersLength`는 정적 렌더링 중에 React가 생성할 수 있는 헤더의 최대 크기를 제어합니다. 이 헤더들은 폰트, 스크립트, 스타일시트와 같은 리소스의 브라우저 프리로딩을 활성화하여 성능을 최적화합니다.

## 기본값

- **기본값:** `6000` 바이트

## 설정

기본값을 재정의하려면 `next.config.js`를 수정합니다:

```js
// next.config.js
module.exports = {
  reactMaxHeadersLength: 1000,
}
```

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactMaxHeadersLength: 1000,
}

export default nextConfig
```

## 중요 제약사항

- **App Router 전용:** 이 옵션은 Next.js App Router에서만 사용 가능합니다
- **프록시 호환성:** 프록시 설정에 따라 헤더가 잘릴 수 있습니다
- **리버스 프록시 고려사항:** 긴 헤더를 지원하지 않는 리버스 프록시를 사용하는 경우, 헤더 잘림을 방지하기 위해 낮은 값을 설정하세요

## 사용 사례

서버 인프라(특히 리버스 프록시)에 헤더 길이 제한이 있는 경우, 전송 중 헤더가 잘리지 않도록 이 값을 낮추세요.

### 예제: Nginx 프록시 사용 시

```js
// next.config.js
module.exports = {
  // Nginx 기본 헤더 제한에 맞춤
  reactMaxHeadersLength: 4000,
}
```

### 예제: 클라우드 환경

```js
// next.config.js
module.exports = {
  // 클라우드 로드 밸런서 제한에 맞춤
  reactMaxHeadersLength: 2000,
}
```

## 작동 방식

React는 정적 렌더링 중에 `Link` 헤더를 생성하여 브라우저가 필요한 리소스를 미리 로드할 수 있도록 합니다. 이 설정은 해당 헤더의 총 크기를 제한합니다.

헤더가 제한을 초과하면:
- 초과 부분은 잘립니다
- 일부 프리로드 힌트가 손실될 수 있습니다
- 페이지는 정상적으로 로드되지만 최적화가 감소할 수 있습니다

---

## 참고

- [이미지 최적화](/app-router/getting-started/09-image-optimization.md)
- [폰트 최적화](/app-router/getting-started/10-font-optimization.md)
- [프리페칭](/app-router/guides/prefetching.md)
