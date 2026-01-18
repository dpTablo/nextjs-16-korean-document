# serverActions

Next.js 애플리케이션에서 Server Actions 동작을 구성하는 옵션입니다.

## 구성 옵션

### allowedOrigins

Server Actions를 호출할 수 있는 추가 안전한 출처 도메인 목록입니다. Next.js는 Server Action 요청의 출처와 호스트 도메인을 비교하여 CSRF 공격을 방지합니다. 일치하지 않으면 요청이 거부됩니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */

module.exports = {
  experimental: {
    serverActions: {
      allowedOrigins: ['my-proxy.com', '*.my-proxy.com'],
    },
  },
}
```

**지원:** 와일드카드 도메인 (예: `*.my-proxy.com`)

### bodySizeLimit

기본적으로 Server Action으로 전송되는 요청 본문의 최대 크기는 1MB입니다. 이는 대량의 데이터를 파싱하는 데 과도한 서버 리소스가 소비되는 것을 방지하고 잠재적인 DDoS 공격을 막기 위함입니다.

그러나 `serverActions.bodySizeLimit` 옵션을 사용하여 이 제한을 구성할 수 있습니다. 바이트 수 또는 바이트에서 지원하는 문자열 형식(예: `1000`, `'500kb'`, `'3mb'`)을 사용할 수 있습니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */

module.exports = {
  experimental: {
    serverActions: {
      bodySizeLimit: '2mb',
    },
  },
}
```

### Server Actions 활성화 (v13)

Server Actions는 Next.js 14에서 안정적인 기능이 되었으며 기본적으로 활성화됩니다. 그러나 이전 버전의 Next.js를 사용하는 경우 `experimental.serverActions`를 `true`로 설정하여 활성화할 수 있습니다.

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const config = {
  experimental: {
    serverActions: true,
  },
}

module.exports = config
```

## 요약 테이블

| 옵션 | 타입 | 기본값 | 목적 |
|------|------|--------|------|
| `allowedOrigins` | Array | 동일 출처만 | CSRF 방지 |
| `bodySizeLimit` | String/Number | `1MB` | 요청 페이로드 크기 제한 |
| `serverActions` | Boolean | `true` (v14+) | 기능 활성화 (v13만) |
