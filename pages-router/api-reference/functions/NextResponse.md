# NextResponse

NextResponse는 [Web Response API](https://developer.mozilla.org/docs/Web/API/Response)를 확장하여 추가 편의 메서드를 제공합니다.

## cookies

응답의 [Set-Cookie](https://developer.mozilla.org/docs/Web/HTTP/Headers/Set-Cookie) 헤더를 읽거나 변경합니다.

### `set(name, value)`

주어진 이름으로 응답에 쿠키를 설정합니다.

```ts
// /home 요청에 대해
let response = NextResponse.next()
// 배너 숨기기 쿠키 설정
response.cookies.set('show-banner', 'false')
// 응답에 `Set-Cookie:show-banner=false;path=/home` 헤더가 설정됩니다
return response
```

### `get(name)`

쿠키 이름으로 쿠키 값을 반환합니다. 쿠키를 찾지 못하면 `undefined`를 반환합니다. 여러 쿠키가 발견되면 첫 번째 것을 반환합니다.

```ts
// /home 요청에 대해
let response = NextResponse.next()
// { name: 'show-banner', value: 'false', Path: '/home' }
response.cookies.get('show-banner')
```

### `getAll(name?)`

쿠키 이름으로 모든 쿠키 값을 반환합니다. 이름이 주어지지 않으면 응답의 모든 쿠키를 반환합니다.

```ts
// 특정 쿠키 이름으로 조회
response.cookies.getAll('experiments')
// [
//   { name: 'experiments', value: 'new-pricing-page', Path: '/home' },
//   { name: 'experiments', value: 'winter-launch', Path: '/home' },
// ]

// 모든 쿠키 조회
response.cookies.getAll()
```

### `has(name)`

쿠키가 존재하는지 여부를 반환합니다.

```ts
// /home 요청에 대해
let response = NextResponse.next()
// true 또는 false 반환
response.cookies.has('experiments')
```

### `delete(name)`

응답에서 쿠키를 삭제합니다.

```ts
// /home 요청에 대해
let response = NextResponse.next()
// 삭제되면 true, 그렇지 않으면 false를 반환
response.cookies.delete('experiments')
```

## json()

주어진 JSON 본문으로 응답을 생성합니다.

```ts filename="app/api/route.ts" switcher
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 })
}
```

```js filename="app/api/route.js" switcher
import { NextResponse } from 'next/server'

export async function GET(request) {
  return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 })
}
```

## redirect()

[URL](https://developer.mozilla.org/docs/Web/API/URL)로 리다이렉트하는 응답을 생성합니다.

```ts
import { NextResponse } from 'next/server'

return NextResponse.redirect(new URL('/new', request.url))
```

[URL](https://developer.mozilla.org/docs/Web/API/URL)은 `NextResponse.redirect()` 메서드에서 사용하기 전에 생성하고 수정할 수 있습니다. 예를 들어, `request.nextUrl` 속성을 사용하여 현재 URL을 가져온 다음 다른 URL로 리다이렉트하도록 수정할 수 있습니다.

```ts
import { NextResponse } from 'next/server'

// 요청이 주어지면...
const loginUrl = new URL('/login', request.url)
// /login URL에 ?from=/incoming-url을 추가합니다
loginUrl.searchParams.set('from', request.nextUrl.pathname)
// 새 URL로 리다이렉트합니다
return NextResponse.redirect(loginUrl)
```

## rewrite()

원래 URL을 유지하면서 주어진 [URL](https://developer.mozilla.org/docs/Web/API/URL)을 재작성(프록시)하는 응답을 생성합니다.

```ts
import { NextResponse } from 'next/server'

// 들어오는 요청: /about, 브라우저에 표시: /about
// 재작성된 요청: /proxy, 브라우저에 표시: /about
return NextResponse.rewrite(new URL('/proxy', request.url))
```

## next()

`next()` 메서드는 라우팅을 계속 진행할 수 있도록 조기에 반환하는 Proxy에 유용합니다.

```ts
import { NextResponse } from 'next/server'

return NextResponse.next()
```

응답을 생성할 때 헤더를 전달할 수도 있습니다:

```ts
import { NextResponse } from 'next/server'

// 요청이 주어지면...
const newHeaders = new Headers(request.headers)
// 새 헤더 추가
newHeaders.set('x-version', '123')
// 새 요청 헤더로 응답 생성
return NextResponse.next({
  request: {
    // 새 요청 헤더
    headers: newHeaders,
  },
})
```

## 버전 정보

| 버전 | 변경사항 |
|------|---------|
| `v16.0.0` | `NextResponse` 안정화 |
