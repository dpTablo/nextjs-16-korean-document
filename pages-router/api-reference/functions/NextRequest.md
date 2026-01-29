# NextRequest

NextRequest는 [Web Request API](https://developer.mozilla.org/docs/Web/API/Request)를 확장하여 추가적인 편의 메서드를 제공합니다.

## cookies

요청의 [Set-Cookie](https://developer.mozilla.org/docs/Web/HTTP/Headers/Set-Cookie) 헤더를 읽거나 변경합니다.

### `set(name, value)`

주어진 이름으로 요청에 쿠키를 설정합니다.

```ts
// /home 요청에 대해
// 배너 숨기기 쿠키 설정
// request에 `Set-Cookie:show-banner=false;path=/home` 헤더 추가
request.cookies.set('show-banner', 'false')
```

### `get(name)`

쿠키 이름으로 쿠키 값을 반환합니다. 쿠키를 찾지 못하면 `undefined`를 반환합니다. 여러 쿠키가 발견되면 첫 번째 것을 반환합니다.

```ts
// /home 요청에 대해
// { name: 'show-banner', value: 'false', Path: '/home' }
request.cookies.get('show-banner')
```

### `getAll(name?)`

쿠키 이름으로 모든 쿠키 값을 반환합니다. 이름이 주어지지 않으면 요청의 모든 쿠키를 반환합니다.

```ts
// 특정 쿠키 이름으로 모든 값 조회
request.cookies.getAll('experiments')
// [
//   { name: 'experiments', value: 'new-pricing-page', Path: '/home' },
//   { name: 'experiments', value: 'winter-launch', Path: '/home' },
// ]

// 요청의 모든 쿠키 조회
request.cookies.getAll()
```

### `delete(name)`

요청에서 쿠키를 삭제합니다. 삭제에 성공하면 `true`를, 실패하면 `false`를 반환합니다.

```ts
request.cookies.delete('experiments')
```

### `has(name)`

쿠키가 존재하는지 여부를 반환합니다.

```ts
request.cookies.has('experiments')
```

### `clear()`

요청의 모든 쿠키를 제거합니다.

```ts
request.cookies.clear()
```

## nextUrl

Native [`URL`](https://developer.mozilla.org/docs/Web/API/URL) API를 확장하여 Next.js 특화 프로퍼티를 제공합니다.

```ts
// /home 요청의 경우, pathname은 /home입니다
request.nextUrl.pathname
// /home?name=lee 요청의 경우, searchParams는 { 'name': 'lee' }입니다
request.nextUrl.searchParams
```

다음 옵션들을 사용할 수 있습니다:

| 프로퍼티 | 타입 | 설명 |
|---------|------|------|
| `basePath` | `string` | URL의 [base path](/docs/app/api-reference/config/next-config-js/basePath) |
| `buildId` | `string \| undefined` | Next.js 애플리케이션의 빌드 식별자. [사용자 정의](/docs/app/api-reference/config/next-config-js/generateBuildId) 가능 |
| `defaultLocale` | `string \| undefined` | [국제화](/docs/app/guides/internationalization)의 기본 로케일 |
| `domainLocale` | | |
| - `defaultLocale` | `string` | 도메인 내의 기본 로케일 |
| - `domain` | `string` | 특정 로케일과 연결된 도메인 |
| - `http` | `boolean \| undefined` | 도메인이 HTTP를 사용하는지 여부 |
| `locales` | `string[]` | 사용 가능한 로케일의 배열 |
| `locale` | `string \| undefined` | 현재 활성 로케일 |
| `url` | `URL` | URL 객체 |

> **참고**: Pages Router의 국제화 프로퍼티는 App Router에서는 사용할 수 없습니다. [App Router에서 국제화](/docs/app/guides/internationalization)에 대해 자세히 알아보세요.

## 버전 정보

| 버전 | 변경사항 |
|------|---------|
| `v15.0.0` | `ip`와 `geo` 제거됨 |
