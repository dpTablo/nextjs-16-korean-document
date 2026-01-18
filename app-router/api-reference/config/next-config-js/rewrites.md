# rewrites

rewrites를 사용하면 들어오는 요청 경로를 다른 대상 경로로 매핑할 수 있습니다. rewrites는 URL 프록시 역할을 하며 대상 경로를 숨겨 사용자가 사이트에서 위치를 변경하지 않은 것처럼 보이게 합니다. 반면에 redirects는 새로운 페이지로 리라우팅하여 URL 변경을 표시합니다.

rewrites를 사용하려면 `next.config.js`에서 `rewrites` 키를 사용하면 됩니다:

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/about',
        destination: '/',
      },
    ]
  },
}
```

rewrites는 클라이언트 사이드 라우팅에 적용되며, 위의 예제에서 `<Link href="/about">`은 rewrite가 적용됩니다.

`rewrites`는 `source`와 `destination` 속성을 가진 객체를 포함하는 배열이나 객체의 배열(아래 참조)을 반환하는 비동기 함수입니다:

- `source`: `String` - 들어오는 요청 경로 패턴입니다.
- `destination`: `String` - 라우팅할 경로입니다.
- `basePath`: `false` 또는 `undefined` - false인 경우 매칭 시 basePath가 포함되지 않으며, 외부 rewrites에만 사용할 수 있습니다.
- `locale`: `false` 또는 `undefined` - 매칭 시 로케일을 포함하지 않을지 여부입니다.
- `has`: [has 객체](#헤더-쿠키-및-쿼리-매칭)의 배열로, `type`, `key`, `value` 속성을 갖습니다.
- `missing`: [missing 객체](#헤더-쿠키-및-쿼리-매칭)의 배열로, `type`, `key`, `value` 속성을 갖습니다.

`rewrites` 함수가 배열을 반환하면, rewrites는 파일 시스템(페이지 및 `/public` 파일) 확인 후, 동적 라우트 전에 적용됩니다. `rewrites` 함수가 특정 형태의 객체 배열을 반환하면, 이 동작을 변경하고 Next.js `v10.1`부터 더 세밀하게 제어할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return {
      beforeFiles: [
        // 이것들은 headers/redirects 후에 확인되며
        // _next/public 파일을 포함한 모든 파일 전에 확인되어
        // 페이지 파일을 덮어쓸 수 있습니다
        {
          source: '/some-page',
          destination: '/somewhere-else',
          has: [{ type: 'query', key: 'overrideMe' }],
        },
      ],
      afterFiles: [
        // 이것들은 pages/public 파일 후에 확인되며
        // 동적 라우트 전에 확인됩니다
        {
          source: '/non-existent',
          destination: '/somewhere-else',
        },
      ],
      fallback: [
        // 이것들은 pages/public 파일과
        // 동적 라우트 모두 확인된 후에 확인됩니다
        {
          source: '/:path*',
          destination: `https://my-old-site.com/:path*`,
        },
      ],
    }
  },
}
```

> **알아두면 좋은 점**: `beforeFiles`의 rewrites는 source 매칭 직후 파일 시스템/동적 라우트를 확인하지 않고, 모든 `beforeFiles`가 확인될 때까지 계속합니다.

Next.js 라우트가 확인되는 순서는 다음과 같습니다:

1. [headers](/docs/app/api-reference/config/next-config-js/headers)가 확인/적용됨
2. [redirects](/docs/app/api-reference/config/next-config-js/redirects)가 확인/적용됨
3. `beforeFiles` rewrites가 확인/적용됨
4. [public 디렉토리](/docs/app/building-your-application/optimizing/static-assets)의 정적 파일, `_next/static` 파일, 비동적 페이지가 확인/제공됨
5. `afterFiles` rewrites가 확인/적용되며, 이 중 하나가 매칭되면 각 매칭 후 동적 라우트/정적 파일을 확인함
6. `fallback` rewrites가 확인/적용되며, 404 페이지 렌더링 전과 동적 라우트/모든 정적 자산이 확인된 후에 적용됩니다. `getStaticPaths`에서 [fallback: true/'blocking'](/docs/pages/api-reference/functions/get-static-paths#fallback-true)을 사용하면, `next.config.js`에 정의된 fallback `rewrites`가 실행되지 _않습니다_.

## Rewrite 매개변수

rewrite에서 매개변수를 사용할 때, `destination`에서 매개변수가 하나도 사용되지 않으면 매개변수는 기본적으로 쿼리에 전달됩니다.

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/old-about/:path*',
        destination: '/about', // :path 매개변수가 여기서 사용되지 않으므로 쿼리에 자동으로 전달됩니다
      },
    ]
  },
}
```

destination에서 매개변수가 사용되면 매개변수가 쿼리에 자동으로 전달되지 않습니다.

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/docs/:path*',
        destination: '/:path*', // :path 매개변수가 여기서 사용되므로 쿼리에 자동으로 전달되지 않습니다
      },
    ]
  },
}
```

destination에서 이미 매개변수가 사용되는 경우 destination에 쿼리를 지정하여 쿼리에 매개변수를 수동으로 전달할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/:first/:second',
        destination: '/:first?second=:second',
        // :first 매개변수가 destination에서 사용되므로 :second 매개변수가
        // 쿼리에 자동으로 추가되지 않지만, 위와 같이 수동으로 추가할 수 있습니다
      },
    ]
  },
}
```

> **알아두면 좋은 점**: [정적 생성](/docs/app/building-your-application/data-fetching/incremental-static-regeneration)의 rewrites에서 자동 정적 최적화나 사전 렌더링 매개변수는 클라이언트에서 하이드레이션 후 파싱되어 쿼리에 제공됩니다.

## 경로 매칭

경로 매칭이 허용됩니다. 예를 들어 `/blog/:slug`는 `/blog/hello-world`와 매칭됩니다(중첩 경로 없음):

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/blog/:slug',
        destination: '/news/:slug', // 매칭된 매개변수는 destination에서 사용할 수 있습니다
      },
    ]
  },
}
```

### 와일드카드 경로 매칭

와일드카드 경로를 매칭하려면 매개변수 뒤에 `*`를 사용할 수 있습니다. 예를 들어 `/blog/:slug*`는 `/blog/a/b/c/d/hello-world`와 매칭됩니다:

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/blog/:slug*',
        destination: '/news/:slug*', // 매칭된 매개변수는 destination에서 사용할 수 있습니다
      },
    ]
  },
}
```

### 정규식 경로 매칭

정규식 경로를 매칭하려면 매개변수 뒤에 괄호로 정규식을 감쌀 수 있습니다. 예를 들어 `/blog/:slug(\\d{1,})`는 `/blog/123`과 매칭되지만 `/blog/abc`와는 매칭되지 않습니다:

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/old-blog/:post(\\d{1,})',
        destination: '/blog/:post', // 매칭된 매개변수는 destination에서 사용할 수 있습니다
      },
    ]
  },
}
```

다음 문자들 `(`, `)`, `{`, `}`, `[`, `]`, `|`, `\`, `^`, `.`, `:`, `*`, `+`, `-`, `?`, `$`는 정규식 경로 매칭에 사용되므로, `source`에서 비특수 값으로 사용될 때는 앞에 `\\`를 추가하여 이스케이프해야 합니다:

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        // 이것은 `/english(default)/something` 요청과 매칭됩니다
        source: '/english\\(default\\)/:slug',
        destination: '/en-us/:slug',
      },
    ]
  },
}
```

## 헤더, 쿠키 및 쿼리 매칭

rewrite가 `has` 필드와 매칭되거나 `missing` 필드와 매칭되지 않을 때만 매칭되도록 할 수 있습니다. `source`와 모든 `has` 항목이 매칭되고 모든 `missing` 항목이 매칭되지 않을 때만 rewrite가 적용됩니다.

`has`와 `missing` 항목은 다음 필드를 가질 수 있습니다:

- `type`: `String` - `header`, `cookie`, `host`, 또는 `query` 중 하나여야 합니다.
- `key`: `String` - 선택된 타입에서 매칭할 키입니다.
- `value`: `String` 또는 `undefined` - 확인할 값입니다. undefined인 경우 모든 값이 매칭됩니다. 정규식과 같은 문자열을 사용하여 값의 특정 부분을 캡처할 수 있습니다. 예: 값이 `first-(?<paramName>.*)`이고 `first-second`이면 destination에서 `:paramName`으로 `second`를 사용할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      // 헤더 `x-rewrite-me`가 있으면
      // 이 rewrite가 적용됩니다
      {
        source: '/:path*',
        has: [
          {
            type: 'header',
            key: 'x-rewrite-me',
          },
        ],
        destination: '/another-page',
      },
      // 헤더 `x-rewrite-me`가 없으면
      // 이 rewrite가 적용됩니다
      {
        source: '/:path*',
        missing: [
          {
            type: 'header',
            key: 'x-rewrite-me',
          },
        ],
        destination: '/another-page',
      },
      // source, query, cookie가 매칭되면
      // 이 rewrite가 적용됩니다
      {
        source: '/specific/:path*',
        has: [
          {
            type: 'query',
            key: 'page',
            // page 값은 destination에서 사용할 수 없습니다
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
        destination: '/:path*/home',
      },
      // 헤더 `x-authorized`가 있고
      // 매칭 값이 'yes' 또는 'true'를 포함하면
      // 이 rewrite가 적용됩니다
      {
        source: '/:path*',
        has: [
          {
            type: 'header',
            key: 'x-authorized',
            value: '(?<authorized>yes|true)',
          },
        ],
        destination: '/home?authorized=:authorized',
      },
      // 호스트가 `example.com`이면
      // 이 rewrite가 적용됩니다
      {
        source: '/:path*',
        has: [
          {
            type: 'host',
            value: 'example.com',
          },
        ],
        destination: '/another-page',
      },
    ]
  },
}
```

## 외부 URL로 Rewriting

rewrites를 사용하면 외부 URL로 rewrite할 수 있습니다. 이는 Next.js를 점진적으로 도입하는 데 특히 유용합니다. 다음은 메인 앱의 `/blog` 라우트를 외부 사이트로 리디렉션하는 rewrite 예제입니다.

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return [
      {
        source: '/blog',
        destination: 'https://example.com/blog',
      },
      {
        source: '/blog/:slug',
        destination: 'https://example.com/blog/:slug',
      },
    ]
  },
}
```

`trailingSlash: true`를 사용하는 경우, `source` 매개변수에도 후행 슬래시를 삽입해야 합니다. 대상 서버도 후행 슬래시를 기대하는 경우 `destination` 매개변수에도 포함해야 합니다.

```js filename="next.config.js"
module.exports = {
  trailingSlash: true,
  async rewrites() {
    return [
      {
        source: '/blog/',
        destination: 'https://example.com/blog/',
      },
      {
        source: '/blog/:path*/',
        destination: 'https://example.com/blog/:path*/',
      },
    ]
  },
}
```

### Next.js 점진적 도입

모든 Next.js 라우트를 확인한 후 기존 웹사이트로 프록시하는 폴백을 Next.js에 설정할 수도 있습니다.

이렇게 하면 더 많은 페이지를 Next.js로 마이그레이션할 때 rewrites 구성을 변경할 필요가 없습니다.

```js filename="next.config.js"
module.exports = {
  async rewrites() {
    return {
      fallback: [
        {
          source: '/:path*',
          destination: `https://custom-routes-proxying-endpoint.vercel.app/:path*`,
        },
      ],
    }
  },
}
```

### basePath 지원이 있는 Rewrites

rewrites와 함께 [`basePath` 지원](/docs/app/api-reference/config/next-config-js/basePath)을 활용할 때, 각 `source`와 `destination`은 rewrite에 `basePath: false`를 추가하지 않는 한 자동으로 `basePath`가 접두사로 붙습니다:

```js filename="next.config.js"
module.exports = {
  basePath: '/docs',

  async rewrites() {
    return [
      {
        source: '/with-basePath', // 자동으로 /docs/with-basePath가 됩니다
        destination: '/another', // 자동으로 /docs/another가 됩니다
      },
      {
        // basePath: false가 설정되어 있으므로 /without-basePath에 /docs가 추가되지 않습니다
        // 참고: 이것은 내부 rewrites에는 사용할 수 없습니다. 예: `destination: '/another'`
        source: '/without-basePath',
        destination: 'https://example.com',
        basePath: false,
      },
    ]
  },
}
```

## 버전 기록

| 버전      | 변경 사항       |
| --------- | --------------- |
| `v13.3.0` | `missing` 추가됨 |
| `v10.2.0` | `has` 추가됨     |
| `v9.5.0`  | Headers 추가됨   |
