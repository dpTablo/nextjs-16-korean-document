# headers

headers를 사용하면 주어진 경로에 대한 들어오는 요청의 응답에 커스텀 HTTP 헤더를 설정할 수 있습니다.

커스텀 HTTP 헤더를 설정하려면 `next.config.js`에서 `headers` 키를 사용하면 됩니다:

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: '/about',
        headers: [
          {
            key: 'x-custom-header',
            value: 'my custom header value',
          },
          {
            key: 'x-another-custom-header',
            value: 'my other custom header value',
          },
        ],
      },
    ]
  },
}
```

`headers`는 `source`와 `headers` 속성을 가진 객체를 포함하는 배열을 반환하는 비동기 함수입니다:

- `source`: 들어오는 요청 경로 패턴입니다.
- `headers`: `key`와 `value` 속성을 가진 응답 헤더 객체의 배열입니다.
- `basePath`: `false` 또는 `undefined` - false인 경우 매칭 시 basePath가 포함되지 않습니다.
- `locale`: `false` 또는 `undefined` - 매칭 시 로케일을 포함하지 않을지 여부입니다.
- `has`: [has 객체](#헤더-쿠키-및-쿼리-매칭)의 배열로, `type`, `key`, `value` 속성을 갖습니다.
- `missing`: [missing 객체](#헤더-쿠키-및-쿼리-매칭)의 배열로, `type`, `key`, `value` 속성을 갖습니다.

헤더는 페이지 및 `/public` 파일을 포함한 파일 시스템보다 먼저 확인됩니다.

## 헤더 덮어쓰기 동작

두 개의 헤더가 같은 경로와 매칭되고 같은 헤더 키를 설정하면, 마지막 헤더 키가 첫 번째를 덮어씁니다. 아래 헤더를 사용하면 경로 `/hello`는 마지막으로 설정된 헤더 값이 `world`이므로 헤더 `x-hello`가 `world`가 됩니다.

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'x-hello',
            value: 'there',
          },
        ],
      },
      {
        source: '/hello',
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
      },
    ]
  },
}
```

## 경로 매칭

경로 매칭이 허용됩니다. 예를 들어 `/blog/:slug`는 `/blog/hello-world`와 매칭됩니다(중첩 경로 없음):

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: '/blog/:slug',
        headers: [
          {
            key: 'x-slug',
            value: ':slug', // 매칭된 매개변수는 값에서 사용할 수 있습니다
          },
          {
            key: 'x-slug-:slug', // 매칭된 매개변수는 키에서도 사용할 수 있습니다
            value: 'my other custom header value',
          },
        ],
      },
    ]
  },
}
```

### 와일드카드 경로 매칭

와일드카드 경로를 매칭하려면 매개변수 뒤에 `*`를 사용할 수 있습니다. 예를 들어 `/blog/:slug*`는 `/blog/a/b/c/d/hello-world`와 매칭됩니다:

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: '/blog/:slug*',
        headers: [
          {
            key: 'x-slug',
            value: ':slug*', // 매칭된 매개변수는 값에서 사용할 수 있습니다
          },
          {
            key: 'x-slug-:slug*', // 매칭된 매개변수는 키에서도 사용할 수 있습니다
            value: 'my other custom header value',
          },
        ],
      },
    ]
  },
}
```

### 정규식 경로 매칭

정규식 경로를 매칭하려면 매개변수 뒤에 괄호로 정규식을 감쌀 수 있습니다. 예를 들어 `/blog/:post(\\d{1,})`는 `/blog/123`과 매칭되지만 `/blog/abc`와는 매칭되지 않습니다:

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        source: '/blog/:post(\\d{1,})',
        headers: [
          {
            key: 'x-post',
            value: ':post',
          },
        ],
      },
    ]
  },
}
```

다음 문자들 `(`, `)`, `{`, `}`, `:`, `*`, `+`, `?`는 정규식 경로 매칭에 사용되므로, `source`에서 비특수 값으로 사용될 때는 앞에 `\\`를 추가하여 이스케이프해야 합니다:

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      {
        // 이것은 `/english(default)/something` 요청과 매칭됩니다
        source: '/english\\(default\\)/:slug',
        headers: [
          {
            key: 'x-header',
            value: 'value',
          },
        ],
      },
    ]
  },
}
```

## 헤더, 쿠키 및 쿼리 매칭

헤더, 쿠키 또는 쿼리 값도 `has` 필드와 매칭되거나 `missing` 필드와 매칭되지 않을 때만 헤더를 적용할 수 있습니다. `source`와 모든 `has` 항목이 매칭되고 모든 `missing` 항목이 매칭되지 않아야만 헤더가 적용됩니다.

`has`와 `missing` 항목은 다음 필드를 가질 수 있습니다:

- `type`: `String` - `header`, `cookie`, `host`, 또는 `query` 중 하나여야 합니다.
- `key`: `String` - 선택된 타입에서 매칭할 키입니다.
- `value`: `String` 또는 `undefined` - 확인할 값입니다. undefined인 경우 모든 값이 매칭됩니다. 정규식과 같은 문자열을 사용하여 값의 특정 부분을 캡처할 수 있습니다. 예: 값이 `first-(?<paramName>.*)`이고 `first-second`이면 destination에서 `:paramName`으로 `second`를 사용할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  async headers() {
    return [
      // 헤더 `x-add-header`가 있으면
      // `x-another-header` 헤더가 적용됩니다
      {
        source: '/:path*',
        has: [
          {
            type: 'header',
            key: 'x-add-header',
          },
        ],
        headers: [
          {
            key: 'x-another-header',
            value: 'hello',
          },
        ],
      },
      // 헤더 `x-no-header`가 없으면
      // `x-another-header` 헤더가 적용됩니다
      {
        source: '/:path*',
        missing: [
          {
            type: 'header',
            key: 'x-no-header',
          },
        ],
        headers: [
          {
            key: 'x-another-header',
            value: 'hello',
          },
        ],
      },
      // source, query, cookie가 매칭되면
      // `x-authorized` 헤더가 적용됩니다
      {
        source: '/specific/:path*',
        has: [
          {
            type: 'query',
            key: 'page',
            // page 값은 헤더 key/values에서 사용할 수 없습니다
            // 명명된 캡처 그룹을 사용하지 않았으므로
            // 예: (?<page>home)
            value: 'home',
          },
          {
            type: 'cookie',
            key: 'authorized',
            value: 'true',
          },
        ],
        headers: [
          {
            key: 'x-authorized',
            value: ':authorized',
          },
        ],
      },
      // 헤더 `x-authorized`가 있고
      // 매칭 값이 'yes' 또는 'true'를 포함하면
      // `x-another-header` 헤더가 적용됩니다
      {
        source: '/:path*',
        has: [
          {
            type: 'header',
            key: 'x-authorized',
            value: '(?<authorized>yes|true)',
          },
        ],
        headers: [
          {
            key: 'x-another-header',
            value: ':authorized',
          },
        ],
      },
      // 호스트가 `example.com`이면
      // 이 헤더가 적용됩니다
      {
        source: '/:path*',
        has: [
          {
            type: 'host',
            value: 'example.com',
          },
        ],
        headers: [
          {
            key: 'x-another-header',
            value: ':authorized',
          },
        ],
      },
    ]
  },
}
```

## basePath 지원이 있는 Headers

headers와 함께 [`basePath` 지원](/docs/app/api-reference/config/next-config-js/basePath)을 활용할 때, 각 `source`는 headers에 `basePath: false`를 추가하지 않는 한 자동으로 `basePath`가 접두사로 붙습니다:

```js filename="next.config.js"
module.exports = {
  basePath: '/docs',

  async headers() {
    return [
      {
        source: '/with-basePath', // /docs/with-basePath가 됩니다
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
      },
      {
        source: '/without-basePath', // basePath: false가 설정되어 있으므로 수정되지 않습니다
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
        basePath: false,
      },
    ]
  },
}
```

## i18n 지원이 있는 Headers

headers와 함께 [`i18n` 지원](/docs/app/building-your-application/routing/internationalization)을 활용할 때, 각 `source`는 headers에 `locale: false`를 추가하지 않는 한 구성된 `locales`를 처리하도록 자동으로 접두사가 붙습니다. `locale: false`를 사용하는 경우 `source`에 로케일을 접두사로 붙여야 올바르게 매칭됩니다.

```js filename="next.config.js"
module.exports = {
  i18n: {
    locales: ['en', 'fr', 'de'],
    defaultLocale: 'en',
  },

  async headers() {
    return [
      {
        source: '/with-locale', // 모든 로케일을 자동으로 처리합니다
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
      },
      {
        // locale: false가 설정되어 있으므로 로케일을 자동으로 처리하지 않습니다
        source: '/nl/with-locale-manual',
        locale: false,
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
      },
      {
        // 'en'이 defaultLocale이므로 '/'와 매칭됩니다
        source: '/en',
        locale: false,
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
      },
      {
        // 이것은 /(en|fr|de)/(.*)로 변환되므로
        // /:path*처럼 최상위 `/` 또는 `/fr` 라우트와 매칭되지 않습니다
        source: '/(.*)',
        headers: [
          {
            key: 'x-hello',
            value: 'world',
          },
        ],
      },
    ]
  },
}
```

## Cache-Control

Next.js는 진정으로 불변인 자산에 대해 `Cache-Control` 헤더를 `public, max-age=31536000, immutable`로 설정합니다. 이것은 덮어쓸 수 없습니다. 이러한 불변 파일은 파일명에 SHA-해시를 포함하므로 무기한으로 안전하게 캐시할 수 있습니다. 예를 들어, [정적 이미지 가져오기](/docs/app/building-your-application/optimizing/images#local-images)가 있습니다. `next.config.js`에서 이러한 자산에 대한 `Cache-Control` 헤더를 설정할 수 없습니다.

그러나 다른 응답이나 데이터에 대한 `Cache-Control` 헤더는 설정할 수 있습니다.

App Router를 사용하여 [캐싱](/docs/app/building-your-application/caching)에 대해 자세히 알아보세요.

## 옵션

### CORS

[교차 출처 리소스 공유(CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS)는 어떤 사이트가 리소스에 접근할 수 있는지 제어하는 보안 기능입니다. 특정 출처가 라우트 핸들러에 접근할 수 있도록 `Access-Control-Allow-Origin` 헤더를 설정할 수 있습니다.

```js filename="next.config.js"
async headers() {
  return [
    {
      source: "/api/:path*",
      headers: [
        {
          key: "Access-Control-Allow-Origin",
          value: "*", // 출처를 설정합니다
        },
        {
          key: "Access-Control-Allow-Methods",
          value: "GET, POST, PUT, DELETE, OPTIONS",
        },
        {
          key: "Access-Control-Allow-Headers",
          value: "Content-Type, Authorization",
        },
      ],
    },
  ];
}
```

### X-DNS-Prefetch-Control

[이 헤더](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-DNS-Prefetch-Control)는 DNS 프리페칭을 제어하여 브라우저가 외부 링크, 이미지, CSS, JavaScript 등에 대한 도메인 이름 확인을 사전에 수행할 수 있게 합니다. 이 프리페칭은 백그라운드에서 수행되므로 참조된 항목이 필요할 때 [DNS](https://developer.mozilla.org/docs/Glossary/DNS)가 이미 확인되어 있을 가능성이 높습니다. 이는 사용자가 링크를 클릭할 때 지연 시간을 줄입니다.

```js filename="next.config.js"
{
  key: 'X-DNS-Prefetch-Control',
  value: 'on'
}
```

### Strict-Transport-Security

[이 헤더](https://developer.mozilla.org/docs/Web/HTTP/Headers/Strict-Transport-Security)는 브라우저에 HTTP 대신 HTTPS만 사용하여 접근해야 함을 알립니다. 아래 구성을 사용하면 현재 및 미래의 모든 서브도메인은 최대 2년 동안 HTTPS를 사용합니다. 이는 HTTP로만 제공될 수 있는 페이지나 서브도메인에 대한 접근을 차단합니다.

[Vercel](https://vercel.com/docs/concepts/edge-network/headers#strict-transport-security?utm_source=next-site&utm_medium=docs&utm_campaign=next-website)에 배포하는 경우, `next.config.js`에서 `headers`를 선언하지 않으면 모든 배포에 자동으로 추가되므로 이 헤더가 필요하지 않습니다.

```js filename="next.config.js"
{
  key: 'Strict-Transport-Security',
  value: 'max-age=63072000; includeSubDomains; preload'
}
```

### X-Frame-Options

[이 헤더](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options)는 사이트가 `iframe` 내에서 표시될 수 있는지 여부를 나타냅니다. 이는 클릭재킹 공격을 방지할 수 있습니다.

**이 헤더는 CSP의 `frame-ancestors` 옵션으로 대체되었습니다**, 최신 브라우저에서 더 나은 지원을 제공합니다(구성 세부 정보는 [콘텐츠 보안 정책](/docs/app/building-your-application/configuring/content-security-policy) 참조).

```js filename="next.config.js"
{
  key: 'X-Frame-Options',
  value: 'SAMEORIGIN'
}
```

### Permissions-Policy

[이 헤더](https://developer.mozilla.org/docs/Web/HTTP/Headers/Permissions-Policy)를 사용하면 브라우저에서 어떤 기능과 API를 사용할 수 있는지 제어할 수 있습니다. 이전에는 `Feature-Policy`로 명명되었습니다.

```js filename="next.config.js"
{
  key: 'Permissions-Policy',
  value: 'camera=(), microphone=(), geolocation=(), browsing-topics=()'
}
```

### X-Content-Type-Options

[이 헤더](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Content-Type-Options)는 `Content-Type` 헤더가 명시적으로 설정되지 않은 경우 브라우저가 콘텐츠 유형을 추측하려고 하는 것을 방지합니다. 이는 사용자가 파일을 업로드하고 공유할 수 있는 웹사이트에 대한 XSS 익스플로잇을 방지할 수 있습니다.

예를 들어, 사용자가 이미지를 다운로드하려고 하지만 실행 파일과 같은 다른 `Content-Type`으로 처리되어 악의적일 수 있습니다. 이 헤더는 브라우저 확장 프로그램 다운로드에도 적용됩니다. 이 헤더의 유일한 유효한 값은 `nosniff`입니다.

```js filename="next.config.js"
{
  key: 'X-Content-Type-Options',
  value: 'nosniff'
}
```

### Referrer-Policy

[이 헤더](https://developer.mozilla.org/docs/Web/HTTP/Headers/Referrer-Policy)는 현재 웹사이트(출처)에서 다른 웹사이트로 이동할 때 브라우저가 포함하는 정보의 양을 제어합니다.

```js filename="next.config.js"
{
  key: 'Referrer-Policy',
  value: 'origin-when-cross-origin'
}
```

### Content-Security-Policy

애플리케이션에 [콘텐츠 보안 정책](/docs/app/building-your-application/configuring/content-security-policy)을 추가하는 방법에 대해 자세히 알아보세요.

## 버전 기록

| 버전      | 변경 사항        |
| --------- | ---------------- |
| `v13.3.0` | `missing` 추가됨 |
| `v10.2.0` | `has` 추가됨     |
| `v9.5.0`  | Headers 추가됨   |
