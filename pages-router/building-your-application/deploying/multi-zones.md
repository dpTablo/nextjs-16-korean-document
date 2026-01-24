# Multi-Zones

Multi-Zones는 큰 애플리케이션을 여러 개의 독립적인 Next.js 애플리케이션으로 분리하여 각각 특정 경로 세트를 제공하는 마이크로 프론트엔드 접근 방식입니다.

## 장점

- **빌드 시간 개선**: 애플리케이션 크기 감소로 빌드가 빨라집니다.
- **독립적인 개발 및 배포**: 각 Zone을 개별적으로 개발하고 배포할 수 있습니다.
- **유연한 기술 스택**: 각 Zone에서 다른 프레임워크 버전이나 설정을 사용할 수 있습니다.

## 예시

```
애플리케이션 구조:
- /blog/*        → 블로그 Zone (블로그 포스트)
- /dashboard/*   → 대시보드 Zone (로그인 사용자용)
- /*             → 메인 Zone (나머지 웹사이트)
```

## Zone 정의 방법

### assetPrefix 설정

각 Zone에서 정적 자산에 대한 고유한 접두사를 설정합니다:

```js
// next.config.js (블로그 Zone)
const nextConfig = {
  assetPrefix: '/blog-static',
}

module.exports = nextConfig
```

### Next.js 15 이전 버전

Next.js 15 이전 버전에서는 rewrite 설정이 필요합니다:

```js
// next.config.js
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

## 요청을 올바른 Zone으로 라우팅

### Rewrites를 이용한 라우팅

메인 애플리케이션에서 다른 Zone으로 요청을 라우팅합니다:

```js
// next.config.js (메인 애플리케이션)
const nextConfig = {
  async rewrites() {
    return [
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
    ]
  },
}

module.exports = nextConfig
```

### Middleware를 이용한 라우팅

조건부 라우팅이 필요한 경우 Middleware를 사용합니다:

```js
// middleware.js
import { NextResponse } from 'next/server'

export function middleware(request) {
  const { pathname, search } = request.nextUrl

  // 특정 조건에서 다른 Zone으로 라우팅
  if (pathname.startsWith('/blog') && shouldUseBlogZone()) {
    const rewriteDomain = process.env.BLOG_DOMAIN
    return NextResponse.rewrite(`${rewriteDomain}${pathname}${search}`)
  }

  return NextResponse.next()
}
```

## 네비게이션 유형

### 소프트 네비게이션

같은 Zone 내에서 페이지를 이동할 때 발생합니다:
- 페이지 리로드 없음
- `<Link>` 컴포넌트 사용

### 하드 네비게이션

다른 Zone으로 이동할 때 발생합니다:
- 전체 페이지 리로드
- 리소스 재로드

## Zone 간 링크 설정

다른 Zone의 경로로 이동할 때는 `<Link>` 컴포넌트 대신 `<a>` 태그를 사용해야 합니다:

```jsx
// ❌ 잘못된 방법 - 다른 Zone으로 갈 때
import Link from 'next/link'

function Navigation() {
  return <Link href="/blog/post">블로그로 이동</Link>
}

// ✅ 올바른 방법 - 다른 Zone으로 갈 때
function Navigation() {
  return <a href="/blog/post">블로그로 이동</a>
}
```

> **이유**: `<Link>` 컴포넌트는 클라이언트 사이드 네비게이션을 시도하는데, 다른 Zone의 코드는 현재 Zone에 없기 때문입니다.

## 코드 공유 방법

### 1. Monorepo

모든 Zone을 한 저장소에서 관리합니다:

```
monorepo/
├── apps/
│   ├── main/         # 메인 애플리케이션
│   ├── blog/         # 블로그 Zone
│   └── dashboard/    # 대시보드 Zone
└── packages/
    ├── ui/           # 공유 UI 컴포넌트
    └── utils/        # 공유 유틸리티
```

### 2. NPM 패키지

공유 코드를 NPM 패키지로 배포합니다:

```json
// packages/shared-ui/package.json
{
  "name": "@company/shared-ui",
  "version": "1.0.0",
  "main": "dist/index.js"
}
```

```js
// apps/blog/components/Header.js
import { Button } from '@company/shared-ui'
```

### 3. Feature Flags

Zone 간 기능 통일을 관리합니다:

```js
// 기능 플래그에 따라 동작 변경
if (featureFlags.newDesign) {
  // 새로운 디자인 렌더링
} else {
  // 기존 디자인 렌더링
}
```

## 주의사항

1. **URL 경로 충돌 방지**: 각 Zone의 URL 경로는 고유해야 합니다.

2. **세션 공유**: Zone 간 세션을 공유하려면 공통 세션 저장소(Redis 등)를 사용하세요.

3. **환경 변수**: 각 Zone에서 필요한 환경 변수를 올바르게 설정하세요.

4. **CORS 설정**: Zone 간 API 호출 시 CORS 설정이 필요할 수 있습니다.

## 배포 전략

### 독립 배포

각 Zone을 독립적으로 배포합니다:

```bash
# 블로그 Zone 배포
cd apps/blog && npm run build && npm run deploy

# 메인 애플리케이션 배포
cd apps/main && npm run build && npm run deploy
```

### 프록시/로드 밸런서

Nginx나 클라우드 로드 밸런서를 사용하여 요청을 라우팅합니다:

```nginx
# nginx.conf
upstream main_zone {
    server main.example.com;
}

upstream blog_zone {
    server blog.example.com;
}

server {
    listen 80;
    server_name example.com;

    location /blog {
        proxy_pass http://blog_zone;
    }

    location / {
        proxy_pass http://main_zone;
    }
}
```
