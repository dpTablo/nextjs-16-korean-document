---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/assetPrefix
버전: 16.1.6
---

# assetPrefix

> **주의**: [Vercel에 배포](/docs/app/building-your-application/deploying)하면 Next.js 프로젝트에 대해 글로벌 CDN이 자동으로 구성됩니다. `assetPrefix`를 수동으로 설정할 필요가 없습니다.

> **알아두면 좋은 점**: Next.js 9.5+에서는 커스터마이징 가능한 [Base Path](/docs/app/api-reference/config/next-config-js/basePath)에 대한 지원이 추가되었으며, 이는 `/docs`와 같은 하위 경로에서 애플리케이션을 호스팅하는 데 더 적합합니다. 이 사용 사례에서는 커스텀 Asset Prefix를 사용하지 않는 것이 좋습니다.

## CDN 설정하기

[CDN](https://en.wikipedia.org/wiki/Content_delivery_network)을 설정하려면 asset prefix를 설정하고 CDN의 원본이 Next.js가 호스팅되는 도메인으로 확인되도록 구성하면 됩니다.

`next.config.mjs`를 열고 [phase](/docs/app/api-reference/config/next-config-js#async-configuration)에 따라 `assetPrefix` 구성을 추가하세요:

```js filename="next.config.mjs"
// @ts-check
import { PHASE_DEVELOPMENT_SERVER } from 'next/constants'

export default (phase) => {
  const isDev = phase === PHASE_DEVELOPMENT_SERVER
  /**
   * @type {import('next').NextConfig}
   */
  const nextConfig = {
    assetPrefix: isDev ? undefined : 'https://cdn.mydomain.com',
  }
  return nextConfig
}
```

Next.js는 `/_next/` 경로(`.next/static/` 폴더)에서 로드하는 JavaScript 및 CSS 파일에 asset prefix를 자동으로 사용합니다.

예를 들어, 위 구성에서 다음 JavaScript 청크 요청은:

```
/_next/static/chunks/4b9b41aaa062cbbfeff4add70f256968c51ece5d.4d708494b3aed70c04f0.js
```

다음으로 변경됩니다:

```
https://cdn.mydomain.com/_next/static/chunks/4b9b41aaa062cbbfeff4add70f256968c51ece5d.4d708494b3aed70c04f0.js
```

## CDN에 업로드할 항목

파일을 CDN에 업로드할 때 `.next/static/`의 내용만 업로드해야 하며, 위 URL 요청이 나타내는 대로 `_next/static/`으로 업로드해야 합니다. **`.next/` 폴더의 나머지 부분은 업로드하지 마세요**, 서버 코드 및 기타 설정을 공개적으로 노출해서는 안 됩니다.

## 적용되지 않는 항목

`assetPrefix`가 가져오기 요청을 다루더라도, 다음 경로에 대한 요청은 prefixing되지 않습니다:

- [`public`](/docs/app/building-your-application/optimizing/static-assets) 폴더의 파일; 해당 자산을 CDN을 통해 제공하려면 직접 prefix를 추가해야 합니다
