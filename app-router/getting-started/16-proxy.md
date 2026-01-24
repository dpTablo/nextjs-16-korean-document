# 프록시 (Proxy)

> Next.js 16부터 Middleware는 목적을 더 잘 반영하기 위해 Proxy로 이름이 변경되었습니다. 기능은 동일하게 유지됩니다.

Proxy를 사용하면 요청이 완료되기 전에 코드를 실행할 수 있습니다. 들어오는 요청에 따라 다음을 수행할 수 있습니다:

- 요청 재작성(Rewrite)
- 요청 리디렉션(Redirect)
- 요청 또는 응답 헤더 수정
- 직접 응답

## 사용 사례

Proxy가 효과적인 일반적인 시나리오:

- 모든 페이지 또는 페이지의 일부에 대한 헤더 수정
- A/B 테스트 또는 실험에 따라 다른 페이지로 재작성
- 들어오는 요청 속성에 따른 프로그래밍 방식 리디렉션

### Proxy vs 다른 옵션 사용 시기

- 간단한 리디렉션의 경우 먼저 `next.config.ts`의 `redirects` 설정을 사용하세요
- 요청 데이터에 대한 접근이나 복잡한 로직이 필요할 때 Proxy를 사용하세요
- 느린 데이터 가져오기나 전체 세션 관리/권한 부여에는 **적합하지 않습니다**
- 참고: Proxy에서 `options.cache`, `options.next.revalidate`, `options.next.tags`는 효과가 없습니다

## 규칙

프로젝트 루트 또는 `src` 내부에 `pages` 또는 `app`과 같은 수준에서 `proxy.ts` (또는 `.js`) 파일을 생성합니다.

> **참고:** 프로젝트당 하나의 `proxy.ts` 파일만 지원됩니다. 로직을 별도의 모듈로 구성하고 메인 `proxy.ts` 파일에 가져와서 더 깔끔하게 관리하세요.

## 구현 예제

### TypeScript

```ts filename="proxy.ts"
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

// 또는 기본 내보내기 사용:
// export default function proxy(request: NextRequest) { ... }

export const config = {
  matcher: '/about/:path*',
}
```

### JavaScript

```js filename="proxy.js"
import { NextResponse } from 'next/server'

export function proxy(request) {
  return NextResponse.redirect(new URL('/home', request.url))
}

// 또는 기본 내보내기 사용:
// export default function proxy(request) { ... }

export const config = {
  matcher: '/about/:path*',
}
```

## Matcher 설정

`matcher` 설정은 Proxy를 트리거하는 경로를 필터링합니다. 자세한 경로 매칭 옵션은 Matcher 문서를 참조하세요.

## 관련 리소스

- [proxy.js API 참조](/app-router/api-reference/file-conventions/proxy.md) - 전체 API 문서
- [Backend for Frontend 가이드](/app-router/guides/backend-for-frontend.md) - Next.js를 백엔드 프레임워크로 사용하는 방법
