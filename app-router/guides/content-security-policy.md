---
원문: https://nextjs.org/docs/app/guides/content-security-policy
버전: 16.1.6
---

# Content Security Policy (콘텐츠 보안 정책)

Next.js 애플리케이션을 XSS, 클릭재킹 및 코드 삽입 공격으로부터 보호하는 방법을 알아봅니다.

## 개요

Content Security Policy(CSP)는 스크립트, 스타일시트, 이미지, 폰트 및 기타 리소스에 대해 어떤 출처가 허용되는지 지정하여 Next.js 애플리케이션을 XSS(Cross-Site Scripting), 클릭재킹 및 코드 삽입 공격으로부터 보호하는 보안 표준입니다.

---

## 핵심 개념

### Nonce (논스)

**Nonce**(number used once)는 특정 인라인 스크립트/스타일만 실행되도록 선택적으로 허용하면서도 엄격한 CSP 지시문을 유지하는 고유한 무작위 문자열입니다.

**작동 원리:**
- 각 요청마다 고유한 무작위 값 생성
- 공격자는 nonce 값을 추측해야 정책을 우회할 수 있음
- 예측 불가능성이 보안의 핵심

---

## 구현 방법

### 1. Nonce 사용 (동적 렌더링)

페이지 렌더링 전에 nonce를 생성하는 **프록시** 파일 사용:

**middleware.ts**
```ts
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  // 고유한 nonce 생성
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64')

  // CSP 헤더 정의
  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
    style-src 'self' 'nonce-${nonce}';
    img-src 'self' blob: data:;
    font-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
  `

  // 공백 제거 및 정리
  const contentSecurityPolicyHeaderValue = cspHeader
    .replace(/\s{2,}/g, ' ')
    .trim()

  // 요청 헤더에 nonce 추가
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-nonce', nonce)
  requestHeaders.set(
    'Content-Security-Policy',
    contentSecurityPolicyHeaderValue
  )

  // 응답 헤더에 CSP 추가
  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  })
  response.headers.set(
    'Content-Security-Policy',
    contentSecurityPolicyHeaderValue
  )

  return response
}

// Middleware 적용 범위 설정
export const config = {
  matcher: [
    {
      source: '/((?!api|_next/static|_next/image|favicon.ico).*)',
      missing: [
        { type: 'header', key: 'next-router-prefetch' },
        { type: 'header', key: 'purpose', value: 'prefetch' },
      ],
    },
  ],
}
```

### 페이지에서 Nonce 사용

**app/page.tsx**
```tsx
import { headers } from 'next/headers'
import Script from 'next/script'
import { connection } from 'next/server'

export default async function Page() {
  // 동적 렌더링 강제
  await connection()

  // 헤더에서 nonce 추출
  const headersList = await headers()
  const nonce = headersList.get('x-nonce')

  return (
    <div>
      <h1>CSP with Nonce</h1>
      <Script
        src="https://www.googletagmanager.com/gtag/js"
        strategy="afterInteractive"
        nonce={nonce ?? undefined}
      />
    </div>
  )
}
```

**주요 요구사항:**
- 페이지는 **동적으로 렌더링**되어야 함 (`await connection()` 사용)
- Server Components에서 `headers()`를 사용하여 nonce 추출
- `<Script>` 컴포넌트에 nonce 전달

---

### 2. Nonce 없이 (정적 렌더링)

`next.config.js`에서 직접 CSP 구성:

**next.config.js**
```js
const cspHeader = `
  default-src 'self';
  script-src 'self' 'unsafe-eval' 'unsafe-inline';
  style-src 'self' 'unsafe-inline';
  img-src 'self' blob: data:;
  font-src 'self';
  object-src 'none';
  base-uri 'self';
  form-action 'self';
  frame-ancestors 'none';
  upgrade-insecure-requests;
`

module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: cspHeader.replace(/\n/g, ''),
          },
        ],
      },
    ]
  },
}
```

> **참고:** `'unsafe-eval'`과 `'unsafe-inline'`은 보안을 약화시키므로 프로덕션에서는 권장하지 않습니다.

---

### 3. Subresource Integrity (SRI) - 실험적

정적 생성과 엄격한 정책을 위한 해시 기반 CSP:

**next.config.js**
```js
const cspHeader = `
  default-src 'self';
  script-src 'self';
  style-src 'self';
  img-src 'self' blob: data:;
  font-src 'self';
  object-src 'none';
  base-uri 'self';
  form-action 'self';
  frame-ancestors 'none';
  upgrade-insecure-requests;
`

module.exports = {
  experimental: {
    sri: {
      algorithm: 'sha256', // sha384, sha512도 지원
    },
  },
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: cspHeader.replace(/\n/g, ''),
          },
        ],
      },
    ]
  },
}
```

**SRI 이점:**
- ✅ 정적 생성 가능
- ✅ CDN 캐싱 지원
- ✅ 더 나은 성능
- ✅ 빌드 시간 보안 검증

---

## 개발 vs 프로덕션

개발 환경에서는 완화된 정책 활성화:

**middleware.ts**
```ts
export function middleware(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64')
  const isDev = process.env.NODE_ENV === 'development'

  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic' ${
      isDev ? "'unsafe-eval'" : ''
    };
    style-src 'self' ${isDev ? "'unsafe-inline'" : `'nonce-${nonce}'`};
    img-src 'self' blob: data:;
    font-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
  `

  // ... 나머지 구현
}
```

**개발 환경에서만:**
- `'unsafe-eval'` - Next.js Fast Refresh를 위해 필요
- `'unsafe-inline'` - 스타일 디버깅 편의성

---

## 서드파티 스크립트

필요한 도메인을 CSP에 추가하고 nonce 전달:

### Google Tag Manager 예시

**app/layout.tsx**
```tsx
import { GoogleTagManager } from '@next/third-parties/google'
import { headers } from 'next/headers'
import { connection } from 'next/server'

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  await connection()
  const nonce = (await headers()).get('x-nonce')

  return (
    <html lang="ko">
      <body>
        {children}
        <GoogleTagManager gtmId="GTM-XYZ" nonce={nonce ?? undefined} />
      </body>
    </html>
  )
}
```

**CSP 헤더 업데이트:**
```ts
const cspHeader = `
  script-src 'self' 'nonce-${nonce}' 'strict-dynamic' https://www.googletagmanager.com;
  connect-src 'self' https://www.google-analytics.com;
  img-src 'self' https://www.google-analytics.com;
`
```

### 다양한 서드파티 서비스

```ts
const cspHeader = `
  script-src 'self' 'nonce-${nonce}' 'strict-dynamic'
    https://www.googletagmanager.com
    https://cdn.segment.com
    https://connect.facebook.net;
  connect-src 'self'
    https://www.google-analytics.com
    https://api.segment.io
    https://graph.facebook.com;
  img-src 'self' blob: data:
    https://www.google-analytics.com
    https://www.facebook.com;
  frame-src https://www.youtube.com;
`
```

---

## CSP 지시문 설명

### 주요 지시문

| 지시문 | 설명 | 예시 |
|--------|------|------|
| `default-src` | 다른 지시문의 기본값 | `'self'` |
| `script-src` | JavaScript 소스 | `'self' 'nonce-xyz'` |
| `style-src` | CSS 소스 | `'self' 'nonce-xyz'` |
| `img-src` | 이미지 소스 | `'self' data: blob:` |
| `font-src` | 폰트 소스 | `'self'` |
| `connect-src` | fetch, XHR, WebSocket | `'self' https://api.example.com` |
| `frame-src` | iframe 소스 | `https://www.youtube.com` |
| `object-src` | Flash 등 플러그인 | `'none'` |
| `base-uri` | `<base>` 태그 제한 | `'self'` |
| `form-action` | 폼 제출 대상 | `'self'` |
| `frame-ancestors` | 임베드 가능 출처 | `'none'` |

### 특수 키워드

- `'self'` - 현재 도메인
- `'none'` - 아무것도 허용하지 않음
- `'unsafe-inline'` - 인라인 스크립트/스타일 허용 (보안 취약)
- `'unsafe-eval'` - `eval()` 허용 (보안 취약)
- `'strict-dynamic'` - nonce/hash 스크립트가 로드한 스크립트 신뢰

---

## 성능 영향

| 접근 방식 | 정적? | CDN 캐싱 | 서버 부하 | 사용 시기 |
|-----------|-------|----------|-----------|----------|
| **Nonces** | ❌ 아니요 | ❌ 불가 | 높음 | 엄격한 보안 필요 |
| **정적 CSP** | ✅ 예 | ✅ 가능 | 낮음 | 낮은 보안 요구사항 |
| **SRI** | ✅ 예 | ✅ 가능 | 낮음 | 엄격한 CSP + 정적 생성 |

---

## 실전 예시

### 완전한 CSP 구현

**middleware.ts**
```ts
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64')
  const isDev = process.env.NODE_ENV === 'development'

  // 프로덕션용 엄격한 CSP
  const productionCSP = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
    style-src 'self' 'nonce-${nonce}';
    img-src 'self' blob: data: https:;
    font-src 'self';
    connect-src 'self' https://api.example.com;
    frame-src https://www.youtube.com;
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
  `

  // 개발용 완화된 CSP
  const developmentCSP = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic' 'unsafe-eval';
    style-src 'self' 'unsafe-inline';
    img-src 'self' blob: data: https:;
    font-src 'self';
    connect-src 'self' https://api.example.com;
    frame-src https://www.youtube.com;
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
  `

  const cspHeader = isDev ? developmentCSP : productionCSP
  const contentSecurityPolicyHeaderValue = cspHeader
    .replace(/\s{2,}/g, ' ')
    .trim()

  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-nonce', nonce)
  requestHeaders.set(
    'Content-Security-Policy',
    contentSecurityPolicyHeaderValue
  )

  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  })

  response.headers.set(
    'Content-Security-Policy',
    contentSecurityPolicyHeaderValue
  )

  return response
}

export const config = {
  matcher: [
    {
      source: '/((?!api|_next/static|_next/image|favicon.ico).*)',
      missing: [
        { type: 'header', key: 'next-router-prefetch' },
        { type: 'header', key: 'purpose', value: 'prefetch' },
      ],
    },
  ],
}
```

---

## 문제 해결

### CSP 위반 보고

CSP 위반을 모니터링하고 보고:

**middleware.ts에 추가:**
```ts
const cspHeader = `
  ...기존 CSP...
  report-uri /api/csp-report;
  report-to csp-endpoint;
`
```

**app/api/csp-report/route.ts**
```ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const body = await request.json()

  console.log('CSP Violation:', {
    documentURI: body['document-uri'],
    violatedDirective: body['violated-directive'],
    blockedURI: body['blocked-uri'],
  })

  // 분석 서비스로 전송
  // await analytics.track('csp-violation', body)

  return NextResponse.json({ received: true })
}
```

### 브라우저 DevTools에서 확인

1. 브라우저 개발자 도구 열기 (F12)
2. Console 탭 확인
3. CSP 위반 시 오류 메시지 표시됨

**예시 오류:**
```
Refused to execute inline script because it violates the following
Content Security Policy directive: "script-src 'self' 'nonce-xyz'"
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **엄격한 CSP 사용** - `'strict-dynamic'`과 nonce 사용
2. **HTTPS 강제** - `upgrade-insecure-requests` 지시문 추가
3. **프레임 보호** - `frame-ancestors 'none'` 설정
4. **CSP 위반 모니터링** - 보고 엔드포인트 설정
5. **환경별 정책** - 개발/프로덕션 분리

### ❌ 피해야 할 것

1. **`'unsafe-inline'` 사용** - XSS 공격에 취약
2. **`'unsafe-eval'` 프로덕션 사용** - 코드 삽입 위험
3. **와일드카드 사용** - `https://*` 같은 느슨한 정책
4. **CSP 비활성화** - 보안 취약점 발생

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|-----------|
| v14.0.0 | 실험적 SRI 지원 추가 |
| v13.4.20+ | 적절한 nonce 처리 및 CSP 헤더 파싱 |

---

## 다음 단계

- [Authentication](./authentication.md) - 인증 보안
- [Self-Hosting](./self-hosting.md) - 셀프 호스팅 보안
- [Middleware](../api-reference/file-conventions/middleware.md) - Middleware API

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
