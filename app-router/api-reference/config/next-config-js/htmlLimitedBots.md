# htmlLimitedBots

## 개요

`htmlLimitedBots` 설정은 [스트리밍 메타데이터](/app-router/api-reference/functions/generateMetadata.md#streaming-metadata) 대신 차단 메타데이터를 받아야 하는 사용자 에이전트 목록을 지정합니다.

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const config: NextConfig = {
  htmlLimitedBots: /MySpecialBot|MyAnotherSpecialBot|SimpleCrawler/,
}

export default config
```

```js filename="next.config.js"
module.exports = {
  htmlLimitedBots: /MySpecialBot|MyAnotherSpecialBot|SimpleCrawler/,
}
```

## 기본 봇 목록

Next.js는 다음을 포함한 기본 HTML 제한 봇 목록을 포함합니다:

- Google 크롤러 (Mediapartners-Google, AdsBot-Google, Google-PageRenderer 등)
- Bingbot
- Twitterbot
- Slackbot

전체 목록은 [GitHub](https://github.com/vercel/next.js/blob/canary/packages/next/src/shared/lib/router/utils/html-bots.ts)에서 확인할 수 있습니다.

> **중요**: `htmlLimitedBots` 설정은 Next.js 기본 목록을 **덮어쓰므로** 대부분의 경우 기본값 사용이 권장됩니다.

## 스트리밍 메타데이터 완전 비활성화

모든 사용자 에이전트에 대해 스트리밍 메타데이터를 비활성화하려면:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const config: NextConfig = {
  htmlLimitedBots: /.*/,
}

export default config
```

```js filename="next.config.js"
module.exports = {
  htmlLimitedBots: /.*/,
}
```

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| 15.2.0 | `htmlLimitedBots` 옵션 도입 |
