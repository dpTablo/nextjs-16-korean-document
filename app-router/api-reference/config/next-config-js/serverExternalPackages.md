# serverExternalPackages

[서버 컴포넌트](/docs/app/building-your-application/rendering/server-components)와 [라우트 핸들러](/docs/app/building-your-application/routing/route-handlers) 내부에서 사용되는 종속성은 Next.js에 의해 자동으로 번들됩니다.

종속성이 Node.js 특정 기능을 사용하는 경우, 서버 컴포넌트 번들링에서 특정 종속성을 제외하고 네이티브 Node.js `require`를 사용하도록 선택할 수 있습니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  serverExternalPackages: ['@acme/ui'],
}

module.exports = nextConfig
```

Next.js는 현재 호환성 작업 중인 [인기 패키지의 짧은 목록](https://github.com/vercel/next.js/blob/canary/packages/next/src/lib/server-external-packages.json)을 포함하며 자동으로 제외됩니다:

## 자동 제외 패키지

Next.js는 Node.js 종속성이 있는 것으로 알려진 선별된 인기 패키지 목록을 자동으로 제외합니다. 여기에는 다음이 포함됩니다:

**데이터베이스/ORM 패키지:**
- `@prisma/client`
- `mongodb`
- `mongoose`
- `pg`
- `sqlite3`
- `libsql`
- `better-sqlite3`

**AWS/클라우드:**
- `@aws-sdk/client-s3`
- `@aws-sdk/s3-presigned-post`
- `firebase-admin`

**인증:**
- `@node-rs/argon2`
- `@node-rs/bcrypt`
- `argon2`
- `bcrypt`
- `oslo`

**AI/ML:**
- `@huggingface/transformers`
- `@xenova/transformers`
- `onnxruntime-node`

**유틸리티:**
- `sharp`
- `express`
- `node-cron`
- `canvas`
- `playwright`
- `puppeteer`
- `shiki`

**개발 도구:**
- `typescript`
- `eslint`
- `prettier`
- `webpack`
- `jest`
- `cypress`

그 외 70개 이상의 패키지가 포함됩니다.

## 버전 기록

| 버전      | 변경 사항                                                                            |
| --------- | ------------------------------------------------------------------------------------ |
| `v15.0.0` | experimental에서 stable로 이동; `serverComponentsExternalPackages`에서 이름 변경됨   |
