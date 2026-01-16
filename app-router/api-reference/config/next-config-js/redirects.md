# redirects

## 개요

`redirects`를 사용하면 들어오는 요청 경로를 다른 목적지로 리디렉트할 수 있습니다. `next.config.js`의 `redirects` 키를 사용하여 구성합니다.

---

## 기본 설정

```js
// next.config.js
module.exports = {
  async redirects() {
    return [
      {
        source: '/about',
        destination: '/',
        permanent: true,
      },
    ]
  },
}
```

`redirects`는 비동기 함수이며 다음 속성을 가진 객체 배열을 반환합니다:

---

## 속성 (Properties)

### 필수 속성

#### source

- **타입:** `string`
- **설명:** 들어오는 요청 경로 패턴

```js
{
  source: '/old-path',
  destination: '/new-path',
  permanent: true,
}
```

#### destination

- **타입:** `string`
- **설명:** 라우팅할 경로

```js
{
  source: '/old',
  destination: '/new',
  permanent: true,
}
```

#### permanent

- **타입:** `boolean`
- **설명:**
  - `true` → HTTP 308 (클라이언트/검색엔진에서 영구적으로 캐시)
  - `false` → HTTP 307 (임시, 캐시되지 않음)

```js
// 영구 리디렉트 (308)
{
  source: '/old-blog/:slug',
  destination: '/news/:slug',
  permanent: true,
}

// 임시 리디렉트 (307)
{
  source: '/maintenance',
  destination: '/under-construction',
  permanent: false,
}
```

> **참고:** Next.js는 원본 요청 메서드(GET, POST 등)를 보존하기 위해 307/308을 사용합니다. 레거시 301/302는 브라우저가 종종 GET으로 변환했습니다.

### 선택적 속성

#### statusCode

- **타입:** `number`
- **설명:** 커스텀 HTTP 상태 코드 (`permanent` 대신 사용, 둘 다 사용 불가)

```js
{
  source: '/old-path',
  destination: '/new-path',
  statusCode: 301, // 영구 리디렉트 (SEO 전송)
}

{
  source: '/temp-redirect',
  destination: '/temporary',
  statusCode: 302, // 임시 리디렉트
}
```

#### basePath

- **타입:** `false | undefined`
- **설명:** 매칭에서 `basePath` 접두사 제외 (외부 리디렉트만)

```js
module.exports = {
  basePath: '/docs',
  async redirects() {
    return [
      {
        source: '/with-basePath',
        destination: '/another',
        permanent: false,
        // /docs/with-basePath → /docs/another
      },
      {
        source: '/without-basePath',
        destination: 'https://example.com',
        basePath: false, // basePath 스킵
        permanent: false,
        // /without-basePath → https://example.com
      },
    ]
  },
}
```

#### locale

- **타입:** `false | undefined`
- **설명:** 매칭에서 로케일 제외

```js
{
  source: '/nl/with-locale-handling',
  destination: '/nl/another',
  locale: false, // 로케일 자동 처리 비활성화
  permanent: false,
}
```

#### has

- **타입:** `array`
- **설명:** 리디렉트가 적용되기 위해 일치해야 하는 조건

```js
{
  source: '/specific/:path*',
  has: [
    {
      type: 'header' | 'cookie' | 'host' | 'query',
      key: 'key-name',
      value: 'optional-regex-pattern', // undefined면 모든 값 일치
    },
  ],
  destination: '/another/:path*',
  permanent: false,
}
```

#### missing

- **타입:** `array`
- **설명:** 리디렉트가 적용되기 위해 일치하지 **않아야** 하는 조건

```js
{
  source: '/specific/:path*',
  missing: [
    {
      type: 'header',
      key: 'x-do-not-redirect',
    },
  ],
  destination: '/another/:path*',
  permanent: false,
}
```

---

## 경로 매칭 패턴

### 1. 단순 경로 매칭

```js
{
  source: '/old-blog/:slug',
  destination: '/news/:slug',
  permanent: true,
}
```

**매칭:**
- ✅ `/old-blog/first-post` → `/news/first-post`
- ✅ `/old-blog/hello-world` → `/news/hello-world`

**매칭 안 됨:**
- ❌ `/old-blog/a/b` (중첩 경로)
- ❌ `/old-blog` (슬러그 없음)

### 2. 와일드카드 경로 매칭

#### `*` (0개 이상)

```js
{
  source: '/blog/:slug*',
  destination: '/news/:slug*',
  permanent: true,
}
```

**매칭:**
- ✅ `/blog` → `/news`
- ✅ `/blog/a` → `/news/a`
- ✅ `/blog/a/b` → `/news/a/b`
- ✅ `/blog/a/b/c/d` → `/news/a/b/c/d`

#### `+` (1개 이상)

```js
{
  source: '/blog/:slug+',
  destination: '/news/:slug+',
  permanent: true,
}
```

**매칭:**
- ✅ `/blog/a` → `/news/a`
- ✅ `/blog/a/b` → `/news/a/b`

**매칭 안 됨:**
- ❌ `/blog` (최소 1개 필요)

#### `?` (0개 또는 1개)

```js
{
  source: '/category/:slug?',
  destination: '/categories/:slug?',
  permanent: true,
}
```

**매칭:**
- ✅ `/category` → `/categories`
- ✅ `/category/tech` → `/categories/tech`

**매칭 안 됨:**
- ❌ `/category/tech/news` (1개 초과)

### 3. 정규식 경로 매칭

#### 숫자만 매칭

```js
{
  source: '/post/:id(\\d{1,})',
  destination: '/news/:id',
  permanent: false,
}
```

**매칭:**
- ✅ `/post/123` → `/news/123`
- ✅ `/post/456789` → `/news/456789`

**매칭 안 됨:**
- ❌ `/post/abc` (문자)
- ❌ `/post/12abc` (혼합)

#### 특정 패턴 매칭

```js
{
  source: '/user/:name([a-z]{3,10})',
  destination: '/profile/:name',
  permanent: false,
}
```

**매칭:**
- ✅ `/user/john` → `/profile/john`
- ✅ `/user/alice` → `/profile/alice`

**매칭 안 됨:**
- ❌ `/user/ab` (너무 짧음)
- ❌ `/user/JOHN` (대문자)
- ❌ `/user/john123` (숫자 포함)

#### 특수 문자 이스케이프

정규식 특수 문자 `(`, `)`, `{`, `}`, `:`, `*`, `+`, `?`는 이스케이프해야 합니다:

```js
{
  source: '/english\\(default\\)/:slug',
  destination: '/en-us/:slug',
  permanent: false,
}
```

**매칭:**
- ✅ `/english(default)/hello` → `/en-us/hello`

---

## 헤더, 쿠키, 쿼리 매칭

### has 배열 사용

특정 헤더, 쿠키, 쿼리가 있을 때만 리디렉트를 적용합니다.

```js
{
  source: '/specific/:path*',
  has: [
    {
      type: 'header',
      key: 'x-redirect-me',
      value: 'yes', // 선택적 정규식 패턴
    },
  ],
  destination: '/another/:path*',
  permanent: false,
}
```

#### 헤더 매칭

```js
{
  source: '/admin/:path*',
  has: [
    {
      type: 'header',
      key: 'x-admin-token',
      value: '(?<token>[^/]+)', // 정규식으로 값 캡처
    },
  ],
  destination: '/admin-panel/:path*?token=:token',
  permanent: false,
}
```

#### 쿠키 매칭

```js
{
  source: '/dashboard',
  has: [
    {
      type: 'cookie',
      key: 'authorized',
      value: 'true',
    },
  ],
  destination: '/user/dashboard',
  permanent: false,
}
```

#### 호스트 매칭

```js
{
  source: '/:path*',
  has: [
    {
      type: 'host',
      value: 'old-domain.com',
    },
  ],
  destination: 'https://new-domain.com/:path*',
  permanent: true,
}
```

#### 쿼리 매칭

```js
{
  source: '/products',
  has: [
    {
      type: 'query',
      key: 'page',
      value: 'home',
    },
  ],
  destination: '/',
  permanent: false,
}
```

### missing 배열 사용

특정 조건이 **없을 때만** 리디렉트를 적용합니다.

```js
{
  source: '/public/:path*',
  missing: [
    {
      type: 'header',
      key: 'x-do-not-redirect',
    },
  ],
  destination: '/protected/:path*',
  permanent: false,
}
```

### 복합 조건

```js
{
  source: '/feature',
  has: [
    {
      type: 'cookie',
      key: 'beta-user',
      value: 'true',
    },
    {
      type: 'header',
      key: 'user-agent',
      value: '.*Mobile.*', // 모바일 디바이스
    },
  ],
  missing: [
    {
      type: 'query',
      key: 'disable-redirect',
    },
  ],
  destination: '/mobile-beta-feature',
  permanent: false,
}
```

### 정규식으로 값 캡처

```js
{
  source: '/',
  has: [
    {
      type: 'header',
      key: 'x-authorized',
      value: '(?<authorized>yes|true)',
    },
  ],
  destination: '/home?authorized=:authorized',
  permanent: false,
}
```

**요청:**
```
GET / HTTP/1.1
x-authorized: yes
```

**결과:**
```
→ /home?authorized=yes
```

---

## 실용적인 예제

### 1. 구 블로그에서 새 블로그로 이동

```js
{
  source: '/old-blog/:year(\\d{4})/:month(\\d{2})/:slug',
  destination: '/blog/:slug',
  permanent: true, // SEO 가치 전송
}
```

**예:**
- `/old-blog/2023/12/my-post` → `/blog/my-post`

### 2. 다중 구 경로를 하나로

```js
async redirects() {
  return [
    {
      source: '/(old|previous|legacy)-page',
      destination: '/new-page',
      permanent: true,
    },
  ]
}
```

**예:**
- `/old-page` → `/new-page`
- `/previous-page` → `/new-page`
- `/legacy-page` → `/new-page`

### 3. 로케일 기반 리디렉트

```js
{
  source: '/en/old-path',
  destination: '/en/new-path',
  permanent: false,
}

{
  source: '/:locale(en|fr|de)/products',
  destination: '/:locale/shop',
  permanent: true,
}
```

### 4. 인증 기반 리디렉트

```js
{
  source: '/dashboard',
  missing: [
    {
      type: 'cookie',
      key: 'session',
    },
  ],
  destination: '/login?redirect=/dashboard',
  permanent: false,
}
```

### 5. A/B 테스트 리디렉트

```js
{
  source: '/experiment',
  has: [
    {
      type: 'cookie',
      key: 'ab-test',
      value: 'variant-b',
    },
  ],
  destination: '/experiment-b',
  permanent: false,
}
```

### 6. 외부 URL로 리디렉트

```js
{
  source: '/docs/:path*',
  destination: 'https://docs.example.com/:path*',
  permanent: true,
}
```

### 7. 쿼리 파라미터 보존

```js
{
  source: '/search',
  destination: '/search-results',
  permanent: false,
}
```

**요청:** `/search?q=next.js&sort=date`
**결과:** `/search-results?q=next.js&sort=date`

> 쿼리 값은 자동으로 목적지로 전달됩니다.

### 8. 도메인 마이그레이션

```js
{
  source: '/:path*',
  has: [
    {
      type: 'host',
      value: 'old-domain.com',
    },
  ],
  destination: 'https://new-domain.com/:path*',
  permanent: true,
  basePath: false,
}
```

### 9. 특정 파일 타입 리디렉트

```js
{
  source: '/files/:path*\\.pdf',
  destination: '/documents/:path*.pdf',
  permanent: true,
}
```

### 10. 유지보수 모드

```js
async redirects() {
  const isMaintenanceMode = process.env.MAINTENANCE_MODE === 'true'

  if (isMaintenanceMode) {
    return [
      {
        source: '/:path((?!maintenance).*)', // /maintenance 제외
        destination: '/maintenance',
        permanent: false,
      },
    ]
  }

  return []
}
```

---

## basePath 지원

`basePath` 설정을 사용할 때 `source`와 `destination`은 자동으로 접두사가 붙습니다:

```js
// next.config.js
module.exports = {
  basePath: '/docs',

  async redirects() {
    return [
      {
        source: '/with-basePath',
        destination: '/another',
        permanent: false,
      },
      // /docs/with-basePath → /docs/another

      {
        source: '/without-basePath',
        destination: 'https://example.com',
        basePath: false, // basePath 접두사 스킵
        permanent: false,
      },
      // /without-basePath → https://example.com
    ]
  },
}
```

---

## i18n 지원

```js
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'fr', 'de'],
    defaultLocale: 'en',
  },

  async redirects() {
    return [
      // 특정 로케일
      {
        source: '/en/old-path',
        destination: '/en/new-path',
        permanent: false,
      },

      // 모든 로케일
      {
        source: '/:locale/old-path',
        destination: '/:locale/new-path',
        permanent: false,
      },

      // 특정 로케일만
      {
        source: '/:locale(en|fr|de)/:path*',
        destination: '/:locale/new-section/:path*',
        permanent: false,
      },

      // 로케일 처리 비활성화
      {
        source: '/raw-redirect',
        destination: 'https://example.com',
        locale: false,
        permanent: false,
      },
    ]
  },
}
```

---

## 주의사항

### 1. 클라이언트 사이드 라우팅

**Pages Router:**
- `Link` 또는 `router.push`를 통한 클라이언트 사이드 라우팅에는 적용되지 않음
- Middleware가 있고 경로가 일치하면 적용됨

**App Router:**
- 클라이언트 사이드 네비게이션에도 적용됨

### 2. 파일시스템 우선순위

리디렉트는 파일시스템(`pages/`, `/public` 등)보다 먼저 확인됩니다.

```js
// /about 페이지가 있어도 리디렉트가 우선
{
  source: '/about',
  destination: '/about-us',
  permanent: true,
}
```

### 3. 쿼리 자동 전달

쿼리 파라미터는 자동으로 목적지로 전달됩니다:

```js
{
  source: '/old',
  destination: '/new',
  permanent: true,
}
```

**요청:** `/old?name=value&foo=bar`
**결과:** `/new?name=value&foo=bar`

### 4. 콜론 앞에 슬래시 필수

```js
// ✅ 올바른
{ source: '/old-blog/:slug' }

// ❌ 잘못된
{ source: '/old-blog:slug' }
```

---

## 베스트 프랙티스

### 1. 영구 vs 임시 리디렉트

```js
// ✅ 콘텐츠가 영구적으로 이동 - SEO 가치 전송
{
  source: '/old-product',
  destination: '/new-product',
  permanent: true, // 308
}

// ✅ 일시적인 리디렉트 - SEO 가치 유지
{
  source: '/under-construction',
  destination: '/maintenance',
  permanent: false, // 307
}
```

### 2. 구체적인 패턴 우선

```js
// ✅ 권장 - 구체적인 것부터
async redirects() {
  return [
    {
      source: '/blog/special-post',
      destination: '/featured/special-post',
      permanent: true,
    },
    {
      source: '/blog/:slug',
      destination: '/posts/:slug',
      permanent: true,
    },
  ]
}
```

### 3. 정규식 최소화

```js
// ✅ 단순한 패턴
{
  source: '/blog/:slug',
  destination: '/posts/:slug',
  permanent: true,
}

// ❌ 불필요하게 복잡
{
  source: '/blog/:slug([a-zA-Z0-9-_]+)',
  destination: '/posts/:slug',
  permanent: true,
}
```

### 4. has/missing 조건 문서화

```js
{
  source: '/feature',
  // 베타 사용자만 새 기능으로 리디렉트
  has: [
    {
      type: 'cookie',
      key: 'beta-user',
      value: 'true',
    },
  ],
  destination: '/new-feature',
  permanent: false,
}
```

---

## 자주 하는 실수

### 1. permanent와 statusCode 동시 사용

```js
// ❌ 잘못된 - 둘 중 하나만
{
  source: '/old',
  destination: '/new',
  permanent: true,
  statusCode: 301, // Error!
}

// ✅ 올바른
{
  source: '/old',
  destination: '/new',
  permanent: true,
}
// 또는
{
  source: '/old',
  destination: '/new',
  statusCode: 308,
}
```

### 2. 와일드카드 미매칭

```js
// ❌ 잘못된 - 중첩 경로 미매칭
{
  source: '/blog/:slug',
  destination: '/posts/:slug',
  permanent: true,
}
// /blog/a/b가 매칭 안 됨

// ✅ 올바른 - 모든 중첩 경로 매칭
{
  source: '/blog/:slug*',
  destination: '/posts/:slug*',
  permanent: true,
}
```

### 3. 특수 문자 미이스케이프

```js
// ❌ 잘못된 - 특수 문자 미이스케이프
{
  source: '/category(old)',
  destination: '/category-old',
  permanent: true,
}

// ✅ 올바른 - 이스케이프
{
  source: '/category\\(old\\)',
  destination: '/category-old',
  permanent: true,
}
```

### 4. 무한 리디렉트 루프

```js
// ❌ 위험 - 무한 루프 가능
{
  source: '/:path*',
  destination: '/new/:path*',
  permanent: true,
}
// /new/something → /new/new/something → ...

// ✅ 올바른 - 특정 패턴
{
  source: '/old/:path*',
  destination: '/new/:path*',
  permanent: true,
}
```

---

## 디버깅

### 1. 리디렉트 로깅

```js
// next.config.js
module.exports = {
  async redirects() {
    const redirects = [
      {
        source: '/old',
        destination: '/new',
        permanent: true,
      },
    ]

    console.log('Configured redirects:', redirects)
    return redirects
  },
}
```

### 2. 조건부 리디렉트 테스트

```js
async redirects() {
  const redirects = [
    {
      source: '/test',
      has: [
        {
          type: 'header',
          key: 'x-test',
          value: 'true',
        },
      ],
      destination: '/test-result',
      permanent: false,
    },
  ]

  return redirects
}
```

**테스트:**
```bash
curl -H "x-test: true" http://localhost:3000/test -L
```

---

## 타입 정의

```typescript
type Redirect = {
  source: string
  destination: string
  permanent?: boolean
  statusCode?: number
  basePath?: false
  locale?: false
  has?: Array<{
    type: 'header' | 'cookie' | 'host' | 'query'
    key: string
    value?: string
  }>
  missing?: Array<{
    type: 'header' | 'cookie' | 'host' | 'query'
    key: string
    value?: string
  }>
}
```

---

## 관련 문서

- [Middleware](../../file-conventions/middleware.md)
- [Rewrites](./rewrites.md)
- [Headers](./headers.md)
- [리디렉션 가이드](../../../guides/redirecting.md)

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| `v13.3.0` | `missing` 추가 |
| `v10.2.0` | `has` 추가 |
| `v9.5.0` | `redirects` 추가 |

---

## 요약

- **용도**: 들어오는 요청을 다른 URL로 리디렉트
- **필수 속성**: `source`, `destination`, `permanent` (또는 `statusCode`)
- **패턴 매칭**: 단순, 와일드카드(`*`, `+`, `?`), 정규식
- **조건부**: `has` (조건 일치), `missing` (조건 불일치)
- **HTTP 상태**: 308 (영구), 307 (임시)
- **우선순위**: 파일시스템보다 먼저 확인
- **자동 전달**: 쿼리 파라미터 자동 전달
