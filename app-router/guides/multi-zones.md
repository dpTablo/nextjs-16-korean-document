# Multi-Zones

대규모 Next.js 애플리케이션을 여러 개의 독립적인 애플리케이션으로 분리하는 방법을 알아봅니다.

## 개요

Multi-Zones는 대규모 애플리케이션을 더 작은 독립적인 Next.js 애플리케이션으로 분리하는 마이크로 프론트엔드 접근 방식입니다. 각 애플리케이션은 동일한 도메인에서 서로 다른 경로 집합을 제공합니다.

**장점:**
- 애플리케이션 크기 및 빌드 시간 감소
- 독립적인 개발 및 배포 가능
- 존 내에서 다른 프레임워크 사용 가능
- 존 내에서는 소프트 네비게이션, 존 간에는 하드 네비게이션

---

## 사용 사례

예를 들어, 다음과 같이 경로를 분리할 수 있습니다:

| 경로 | 존 |
|------|-----|
| `/blog/*` | 블로그 존 |
| `/dashboard/*` | 대시보드 존 |
| `/*` | 기본 존 |

각 존은 독립적으로 개발, 배포, 확장할 수 있습니다.

---

## 존 정의

### 기본 구성

각 존은 `assetPrefix`가 설정된 일반 Next.js 앱입니다. 이를 통해 정적 자산 충돌을 방지합니다.

**블로그 존 - next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  assetPrefix: '/blog-static',
}

module.exports = nextConfig
```

**대시보드 존 - next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  assetPrefix: '/dashboard-static',
}

module.exports = nextConfig
```

자산은 `/assetPrefix/_next/...` 경로에서 제공됩니다.

---

### 레거시 구성 (Next.js 15 이전)

Next.js 15.0.0 이전 버전에서는 정적 자산에 대한 rewrites를 추가해야 합니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  assetPrefix: '/blog-static',
  async rewrites() {
    return {
      beforeFiles: [
        {
          source: '/blog-static/_next/:path+',
          destination: '/_next/:path+',
        },
      ],
    }
  },
}

module.exports = nextConfig
```

---

## 요청을 존으로 라우팅

### Rewrites 사용 (권장)

메인 애플리케이션에서 경로를 올바른 존으로 라우팅하도록 구성합니다.

**메인 앱 - next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async rewrites() {
    return [
      // 블로그 존
      {
        source: '/blog',
        destination: `${process.env.BLOG_DOMAIN}/blog`,
      },
      {
        source: '/blog/:path+',
        destination: `${process.env.BLOG_DOMAIN}/blog/:path+`,
      },
      {
        source: '/blog-static/:path+',
        destination: `${process.env.BLOG_DOMAIN}/blog-static/:path+`,
      },
      // 대시보드 존
      {
        source: '/dashboard',
        destination: `${process.env.DASHBOARD_DOMAIN}/dashboard`,
      },
      {
        source: '/dashboard/:path+',
        destination: `${process.env.DASHBOARD_DOMAIN}/dashboard/:path+`,
      },
      {
        source: '/dashboard-static/:path+',
        destination: `${process.env.DASHBOARD_DOMAIN}/dashboard-static/:path+`,
      },
    ]
  },
}

module.exports = nextConfig
```

**환경 변수 설정 (.env.local):**
```env
BLOG_DOMAIN=https://blog.example.com
DASHBOARD_DOMAIN=https://dashboard.example.com
```

> **중요:** URL 경로는 모든 존에서 고유해야 합니다. 예를 들어, 두 존이 `/blog/post`를 제공하면 라우팅 충돌이 발생합니다.

---

### 프록시 사용

동적 라우팅 결정(예: 마이그레이션 중 기능 플래그)이 필요한 경우 미들웨어를 사용할 수 있습니다.

**middleware.ts**
```ts
import { NextResponse, NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  const { pathname, search } = request.nextUrl

  // 기능 플래그에 따라 라우팅
  if (pathname.startsWith('/new-feature') && isFeatureEnabled()) {
    return NextResponse.rewrite(
      new URL(`${process.env.NEW_FEATURE_DOMAIN}${pathname}${search}`)
    )
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/new-feature/:path*'],
}
```

---

## 존 간 링크

존 간 링크에는 Next.js `<Link>` 컴포넌트 대신 표준 `<a>` 태그를 사용합니다.

```tsx
// ❌ 다른 존으로의 링크에 <Link> 사용하지 않기
import Link from 'next/link'

export function Navigation() {
  return (
    <nav>
      <Link href="/blog/post">블로그 글</Link> {/* 작동하지 않음 */}
    </nav>
  )
}

// ✅ <a> 태그 사용
export function Navigation() {
  return (
    <nav>
      <a href="/blog/post">블로그 글</a>
    </nav>
  )
}
```

**이유:** `<Link>` 컴포넌트는 소프트 네비게이션을 시도하지만, 이는 존 간에 작동하지 않습니다. 다른 존으로 이동할 때는 전체 페이지 새로고침(하드 네비게이션)이 필요합니다.

---

## 코드 공유

존 간에 코드를 공유하는 여러 방법이 있습니다.

### 1. 모노레포

모든 존을 하나의 저장소에 저장하여 코드 공유를 용이하게 합니다.

**프로젝트 구조:**
```
my-app/
├── apps/
│   ├── main/          # 메인 앱
│   ├── blog/          # 블로그 존
│   └── dashboard/     # 대시보드 존
├── packages/
│   ├── ui/            # 공유 UI 컴포넌트
│   ├── utils/         # 공유 유틸리티
│   └── config/        # 공유 설정
├── package.json
└── turbo.json
```

**packages/ui/index.ts:**
```ts
export { Button } from './button'
export { Card } from './card'
export { Input } from './input'
```

**apps/blog/page.tsx:**
```tsx
import { Button, Card } from '@repo/ui'

export default function BlogPage() {
  return (
    <Card>
      <h1>블로그</h1>
      <Button>더 보기</Button>
    </Card>
  )
}
```

### 2. NPM 패키지

별도의 저장소 간에 공개 또는 비공개 패키지를 통해 코드를 공유합니다.

```bash
# 공유 패키지 설치
npm install @my-org/shared-components
```

### 3. 기능 플래그

서로 다른 시점에 릴리스되는 존 간에 기능 출시를 조정합니다.

```ts
// lib/feature-flags.ts
export const features = {
  newCheckout: process.env.ENABLE_NEW_CHECKOUT === 'true',
  betaDashboard: process.env.ENABLE_BETA_DASHBOARD === 'true',
}
```

---

## Server Actions 구성

Multi-Zones에서 Server Actions를 사용할 때는 사용자가 접근하는 원본을 명시적으로 허용해야 합니다.

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverActions: {
      allowedOrigins: ['your-production-domain.com', 'www.your-production-domain.com'],
    },
  },
}

module.exports = nextConfig
```

---

## 네비게이션 동작

### 소프트 네비게이션 (존 내)

동일한 존 내에서 페이지 간 이동 시 페이지 새로고침이 발생하지 않습니다.

```
/products → /products/123  (소프트 네비게이션)
```

- 빠른 전환
- 상태 유지
- `<Link>` 컴포넌트 사용 가능

### 하드 네비게이션 (존 간)

다른 존으로 이동 시 전체 페이지 새로고침이 발생합니다.

```
/products → /dashboard  (하드 네비게이션)
```

- 전체 페이지 로드
- 상태 초기화
- `<a>` 태그 사용 필요

### 최적화 팁

자주 방문하는 페이지들을 동일한 존에 그룹화하여 하드 네비게이션을 최소화하세요.

```
// ✅ 좋은 예: 관련 페이지를 같은 존에
블로그 존: /blog, /blog/[slug], /blog/categories, /blog/tags

// ❌ 나쁜 예: 관련 페이지가 다른 존에
존 1: /blog
존 2: /blog/[slug]  // 하드 네비게이션 발생
```

---

## 배포 전략

### Vercel에서 배포

각 존을 별도의 Vercel 프로젝트로 배포하고, 메인 앱에서 rewrites로 연결합니다.

**메인 앱 vercel.json:**
```json
{
  "rewrites": [
    {
      "source": "/blog/:path*",
      "destination": "https://blog-zone.vercel.app/blog/:path*"
    },
    {
      "source": "/dashboard/:path*",
      "destination": "https://dashboard-zone.vercel.app/dashboard/:path*"
    }
  ]
}
```

### 셀프 호스팅

Nginx나 다른 리버스 프록시를 사용하여 요청을 라우팅합니다.

**nginx.conf:**
```nginx
upstream main_app {
    server localhost:3000;
}

upstream blog_zone {
    server localhost:3001;
}

upstream dashboard_zone {
    server localhost:3002;
}

server {
    listen 80;
    server_name example.com;

    location /blog {
        proxy_pass http://blog_zone;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /dashboard {
        proxy_pass http://dashboard_zone;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        proxy_pass http://main_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **고유한 경로 사용**
   - 각 존의 경로가 충돌하지 않도록 설계

2. **assetPrefix 설정**
   - 정적 자산 충돌 방지

3. **관련 페이지 그룹화**
   - 자주 함께 방문하는 페이지를 같은 존에 배치

4. **공유 코드 패키지화**
   - UI 컴포넌트, 유틸리티 등을 패키지로 분리

### ❌ 피해야 할 것

1. **존 간에 `<Link>` 사용**
   ```tsx
   // ❌ 작동하지 않음
   <Link href="/blog">블로그</Link>

   // ✅ 올바른 방법
   <a href="/blog">블로그</a>
   ```

2. **경로 충돌**
   - 두 존이 동일한 경로를 제공하지 않도록 주의

3. **과도한 존 분리**
   - 너무 많은 존은 복잡성을 증가시킴

---

## 문제 해결

### 자산이 로드되지 않음

**확인 사항:**
- `assetPrefix`가 올바르게 설정되었는지 확인
- rewrites가 정적 자산 경로를 포함하는지 확인

### 존 간 네비게이션이 작동하지 않음

**확인 사항:**
- `<a>` 태그를 사용하고 있는지 확인
- rewrites 구성이 올바른지 확인
- 환경 변수가 설정되었는지 확인

---

## 다음 단계

- [Rewrites](../api-reference/file-conventions/route-segment-config.md) - rewrites 구성
- [Middleware](../api-reference/file-conventions/middleware.md) - 미들웨어 가이드
- [Self-Hosting](./self-hosting.md) - 셀프 호스팅

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-15

**참고 자료:**
- [공식 예제](https://github.com/vercel/next.js/tree/canary/examples/with-zones)
- [마이크로 프론트엔드](https://micro-frontends.org/)
