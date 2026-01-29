---
원문: https://nextjs.org/docs/app/api-reference/functions/userAgent
버전: 16.1.6
---

# userAgent

## 개요

`userAgent` 헬퍼는 Next.js의 Web Request API를 확장하여 들어오는 요청에서 사용자 에이전트 정보를 파싱하고 상호작용할 수 있게 해줍니다.

---

## 기본 사용법

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const url = request.nextUrl
  const { device } = userAgent(request)

  const viewport = device.type || 'desktop'
  url.searchParams.set('viewport', viewport)
  return NextResponse.rewrite(url)
}
```

---

## API 레퍼런스

### 함수 시그니처

```ts
import { userAgent } from 'next/server'

userAgent(request: NextRequest): UserAgent
```

### 매개변수

- **`request`**: `NextRequest` 객체

### 반환 값

사용자 에이전트 정보를 포함하는 `UserAgent` 객체

---

## 사용 가능한 속성

### 1. `isBot`

요청이 알려진 봇에서 왔는지 나타내는 Boolean 값

```ts
const { isBot } = userAgent(request)

if (isBot) {
  // 봇 요청 처리
}
```

### 2. `browser`

브라우저 정보

```ts
interface Browser {
  name: string | undefined  // 브라우저 이름 (예: Chrome, Firefox, Safari)
  version: string | undefined  // 브라우저 버전 (예: 120.0.0)
}
```

**예제:**
```ts
const { browser } = userAgent(request)
console.log(browser.name)    // "Chrome"
console.log(browser.version) // "120.0.0"
```

### 3. `device`

디바이스 정보

```ts
interface Device {
  model: string | undefined   // 디바이스 모델 (예: iPhone, iPad)
  type: 'mobile' | 'tablet' | 'console' | 'smarttv' | 'wearable' | 'embedded' | undefined  // 데스크탑인 경우 undefined
  vendor: string | undefined  // 디바이스 제조사 (예: Apple, Samsung)
}
```

**예제:**
```ts
const { device } = userAgent(request)
console.log(device.type)   // "mobile"
console.log(device.vendor) // "Apple"
console.log(device.model)  // "iPhone"
```

### 4. `engine`

브라우저 엔진 정보

```ts
interface Engine {
  name: string | undefined    // 엔진 이름 (예: Blink, Gecko, WebKit, Trident)
  version: string | undefined // 엔진 버전
}
```

**예제:**
```ts
const { engine } = userAgent(request)
console.log(engine.name)    // "Blink"
console.log(engine.version) // "120.0.0.0"
```

### 5. `os`

운영체제 정보

```ts
interface OS {
  name: string | undefined    // OS 이름 (예: Windows, macOS, iOS, Android)
  version: string | undefined // OS 버전
}
```

**예제:**
```ts
const { os } = userAgent(request)
console.log(os.name)    // "iOS"
console.log(os.version) // "17.0"
```

### 6. `cpu`

CPU 아키텍처 정보

```ts
interface CPU {
  architecture: '68k' | 'amd64' | 'arm' | 'arm64' | 'ia32' | 'x86_64' | undefined
}
```

**예제:**
```ts
const { cpu } = userAgent(request)
console.log(cpu.architecture) // "arm64"
```

---

## 실용적인 예제

### 1. 디바이스별 리다이렉트

```ts
// middleware.ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const { device } = userAgent(request)

  if (device.type === 'mobile') {
    return NextResponse.redirect(new URL('/mobile', request.url))
  }

  if (device.type === 'tablet') {
    return NextResponse.redirect(new URL('/tablet', request.url))
  }

  return NextResponse.next()
}
```

### 2. 봇 감지 및 처리

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const { isBot } = userAgent(request)

  if (isBot) {
    // 봇에게는 간단한 HTML 제공
    return NextResponse.rewrite(new URL('/bot-optimized', request.url))
  }

  return NextResponse.next()
}
```

### 3. 브라우저별 최적화

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const { browser } = userAgent(request)

  // 구버전 브라우저 감지
  if (browser.name === 'IE' ||
      (browser.name === 'Chrome' && parseInt(browser.version || '0') < 90)) {
    return NextResponse.redirect(new URL('/legacy-browser', request.url))
  }

  return NextResponse.next()
}
```

### 4. OS별 다운로드 페이지

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const { os } = userAgent(request)
  const url = request.nextUrl.clone()

  if (request.nextUrl.pathname === '/download') {
    switch (os.name) {
      case 'Windows':
        url.pathname = '/download/windows'
        break
      case 'Mac OS':
        url.pathname = '/download/macos'
        break
      case 'Linux':
        url.pathname = '/download/linux'
        break
      default:
        url.pathname = '/download/other'
    }

    return NextResponse.rewrite(url)
  }

  return NextResponse.next()
}
```

### 5. 뷰포트 설정

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const url = request.nextUrl.clone()
  const { device } = userAgent(request)

  const viewport = device.type || 'desktop'
  url.searchParams.set('viewport', viewport)

  return NextResponse.rewrite(url)
}
```

### 6. 분석 및 로깅

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  const ua = userAgent(request)

  // 사용자 에이전트 정보 로깅
  console.log({
    path: request.nextUrl.pathname,
    browser: ua.browser.name,
    version: ua.browser.version,
    device: ua.device.type,
    os: ua.os.name,
    isBot: ua.isBot,
  })

  return NextResponse.next()
}
```

### 7. 반응형 이미지 제공

```ts
import { NextRequest, NextResponse, userAgent } from 'next/server'

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/images/')) {
    const { device } = userAgent(request)
    const url = request.nextUrl.clone()

    if (device.type === 'mobile') {
      // 모바일에는 작은 이미지 제공
      url.searchParams.set('size', 'small')
    } else if (device.type === 'tablet') {
      url.searchParams.set('size', 'medium')
    } else {
      url.searchParams.set('size', 'large')
    }

    return NextResponse.rewrite(url)
  }

  return NextResponse.next()
}
```

---

## 주요 포인트

### 1. Middleware에서 주로 사용

`userAgent`는 주로 **middleware**에서 사용되어 요청을 라우팅하기 전에 사용자 에이전트 정보를 파싱합니다.

### 2. Server Components에서도 사용 가능

```ts
import { headers } from 'next/headers'
import { userAgent } from 'next/server'

export default async function Page() {
  const headersList = await headers()
  const ua = userAgent({ headers: headersList })

  return (
    <div>
      <p>브라우저: {ua.browser.name}</p>
      <p>디바이스: {ua.device.type || 'desktop'}</p>
    </div>
  )
}
```

### 3. Route Handlers에서 사용

```ts
import { NextRequest, userAgent } from 'next/server'

export async function GET(request: NextRequest) {
  const ua = userAgent(request)

  return Response.json({
    browser: ua.browser.name,
    device: ua.device.type,
    os: ua.os.name,
  })
}
```

---

## 디바이스 타입 정의

| 타입 | 설명 | 예시 |
|------|------|------|
| `mobile` | 스마트폰 | iPhone, Android Phone |
| `tablet` | 태블릿 | iPad, Galaxy Tab |
| `console` | 게임 콘솔 | PlayStation, Xbox |
| `smarttv` | 스마트 TV | Samsung Smart TV |
| `wearable` | 웨어러블 기기 | Apple Watch |
| `embedded` | 임베디드 디바이스 | 자동차 시스템 |
| `undefined` | 데스크탑 | PC, Mac |

---

## 베스트 프랙티스

### ✅ 권장사항

1. **디바이스별 최적화**
   ```ts
   const { device } = userAgent(request)
   if (device.type === 'mobile') {
     // 모바일 최적화 콘텐츠 제공
   }
   ```

2. **봇 트래픽 처리**
   ```ts
   const { isBot } = userAgent(request)
   if (isBot) {
     // 봇에게 최적화된 응답 제공
   }
   ```

3. **점진적 개선**
   ```ts
   const { browser } = userAgent(request)
   if (browser.name && parseInt(browser.version || '0') >= 90) {
     // 최신 기능 제공
   }
   ```

### ❌ 피해야 할 사항

1. **User-Agent에 과도하게 의존**
   ```ts
   // ❌ User-Agent는 조작될 수 있음
   // 보안 결정에 사용하지 말 것
   const { isBot } = userAgent(request)
   if (isBot) {
     // return blocked // 보안 결정으로 사용하지 말 것
   }
   ```

2. **모든 요청에서 파싱**
   ```ts
   // ❌ 필요하지 않을 때는 파싱하지 않기
   // 성능 영향 고려
   ```

---

## 타입 정의

```ts
interface UserAgent {
  isBot: boolean
  browser: {
    name: string | undefined
    version: string | undefined
  }
  device: {
    model: string | undefined
    type: 'mobile' | 'tablet' | 'console' | 'smarttv' | 'wearable' | 'embedded' | undefined
    vendor: string | undefined
  }
  engine: {
    name: string | undefined
    version: string | undefined
  }
  os: {
    name: string | undefined
    version: string | undefined
  }
  cpu: {
    architecture: '68k' | 'amd64' | 'arm' | 'arm64' | 'ia32' | 'x86_64' | undefined
  }
}
```

---

## 관련 API

- [Middleware](../file-conventions/middleware.md) - Middleware 설정
- [`headers()`](./headers.md) - 요청 헤더 접근
- [`NextRequest`](./next-request-response.md) - Next.js Request 객체
- [`NextResponse`](./next-request-response.md) - Next.js Response 객체

---

## 버전 정보

- **도입 버전:** Next.js 12.2.0
- **현재 상태:** Stable

---

## 요약

- **용도:** 사용자 에이전트 정보 파싱 및 분석
- **주 사용처:** Middleware, Route Handlers, Server Components
- **주요 속성:** `isBot`, `browser`, `device`, `os`, `engine`, `cpu`
- **사용 사례:** 디바이스별 리다이렉트, 봇 감지, 브라우저별 최적화
- **주의사항:** 보안 결정에 사용하지 말 것, User-Agent는 조작 가능
