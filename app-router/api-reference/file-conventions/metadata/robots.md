# robots.txt

## 개요

`robots.txt`는 검색 엔진 크롤러가 사이트의 어떤 URL에 액세스할 수 있는지 제어하는 **표준 파일**입니다. Next.js는 정적 또는 동적 방식으로 robots.txt를 생성할 수 있습니다.

---

## 파일 위치

```
app/
├── robots.txt          # 정적 파일
├── robots.ts           # 동적 생성
└── robots.js           # 동적 생성
```

---

## 정적 Robots (robots.txt)

### 기본 예제

```txt
# app/robots.txt
User-agent: *
Allow: /
Disallow: /admin/

Sitemap: https://example.com/sitemap.xml
```

---

## 동적 Robots (robots.ts/js)

### 기본 구조

```tsx
// app/robots.ts
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/private/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

### 반환 타입

```tsx
type Robots = {
  rules:
    | {
        userAgent?: string | string[]
        allow?: string | string[]
        disallow?: string | string[]
        crawlDelay?: number
      }
    | Array<{
        userAgent: string | string[]
        allow?: string | string[]
        disallow?: string | string[]
        crawlDelay?: number
      }>
  sitemap?: string | string[]
  host?: string
}
```

---

## 실전 예제

### 1. 기본 설정

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

생성되는 robots.txt:

```txt
User-agent: *
Allow: /
Disallow: /admin/

Sitemap: https://example.com/sitemap.xml
```

### 2. 여러 규칙 정의

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: [
      {
        userAgent: 'Googlebot',
        allow: '/',
        disallow: ['/api/', '/admin/'],
        crawlDelay: 2,
      },
      {
        userAgent: 'Bingbot',
        allow: '/',
        disallow: '/api/',
      },
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/api/', '/admin/', '/private/'],
      },
    ],
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

생성되는 robots.txt:

```txt
User-agent: Googlebot
Allow: /
Disallow: /api/
Disallow: /admin/
Crawl-delay: 2

User-agent: Bingbot
Allow: /
Disallow: /api/

User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /private/

Sitemap: https://example.com/sitemap.xml
```

### 3. 여러 사이트맵

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: [
      'https://example.com/sitemap.xml',
      'https://example.com/sitemap/0.xml',
      'https://example.com/sitemap/1.xml',
    ],
  }
}
```

생성되는 robots.txt:

```txt
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
Sitemap: https://example.com/sitemap/0.xml
Sitemap: https://example.com/sitemap/1.xml
```

### 4. 환경별 설정

```tsx
// app/robots.ts
export default function robots() {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://example.com'
  const isProduction = process.env.NODE_ENV === 'production'

  if (!isProduction) {
    // 개발/스테이징 환경: 모든 크롤링 차단
    return {
      rules: {
        userAgent: '*',
        disallow: '/',
      },
    }
  }

  // 프로덕션 환경: 정상 크롤링 허용
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/admin/', '/api/'],
    },
    sitemap: `${baseUrl}/sitemap.xml`,
  }
}
```

### 5. 특정 봇 차단

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: [
      {
        // 주요 검색 엔진 허용
        userAgent: ['Googlebot', 'Bingbot', 'DuckDuckBot'],
        allow: '/',
        disallow: '/admin/',
      },
      {
        // 나쁜 봇 차단
        userAgent: [
          'AhrefsBot',
          'SemrushBot',
          'MJ12bot',
          'DotBot',
        ],
        disallow: '/',
      },
      {
        // 기본 규칙
        userAgent: '*',
        allow: '/',
        disallow: ['/admin/', '/api/'],
      },
    ],
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

### 6. 동적 사이트맵 목록

```tsx
// app/robots.ts
import { db } from '@/lib/db'

export default async function robots() {
  const baseUrl = 'https://example.com'

  // 사이트맵 개수 계산
  const postCount = await db.post.count()
  const sitemapCount = Math.ceil(postCount / 50000)

  const sitemaps = Array.from(
    { length: sitemapCount },
    (_, i) => `${baseUrl}/sitemap/${i}.xml`
  )

  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
    },
    sitemap: [`${baseUrl}/sitemap.xml`, ...sitemaps],
  }
}
```

### 7. Host 지시자 사용

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: 'https://example.com/sitemap.xml',
    host: 'https://example.com', // Yandex가 선호 도메인으로 사용
  }
}
```

### 8. 크롤 지연 설정

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
      crawlDelay: 10, // 요청 사이에 10초 대기
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

---

## 일반적인 패턴

### 전체 사이트 차단 (개발/스테이징)

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      disallow: '/',
    },
  }
}
```

```txt
User-agent: *
Disallow: /
```

### 모든 크롤러 허용

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

```txt
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

### 특정 디렉토리 차단

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: [
        '/admin/',
        '/api/',
        '/private/',
        '/*.json$', // JSON 파일 차단
        '/search?*', // 쿼리 파라미터 있는 검색 페이지
      ],
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

---

## 모범 사례

### 1. 환경별 분리

```tsx
// ✅ 좋은 예 - 환경별 다른 설정
export default function robots() {
  if (process.env.NODE_ENV !== 'production') {
    return {
      rules: { userAgent: '*', disallow: '/' },
    }
  }

  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}

// ❌ 나쁜 예 - 모든 환경에서 동일
export default function robots() {
  return {
    rules: { userAgent: '*', allow: '/' },
  }
}
```

### 2. 민감한 경로 보호

```tsx
// ✅ 좋은 예 - 민감한 경로 차단
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: [
        '/admin/',
        '/api/',
        '/private/',
        '/.well-known/',
        '/checkout/',
        '/account/',
      ],
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}

// ❌ 나쁜 예 - 민감한 경로 노출
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/', // 모든 경로 허용
    },
  }
}
```

### 3. 타입 안정성

```tsx
// ✅ 좋은 예 - 타입 사용
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}

// ❌ 나쁜 예 - 타입 없음
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allows: '/', // 오타: allows → allow
    },
  }
}
```

### 4. 사이트맵 항상 포함

```tsx
// ✅ 좋은 예 - 사이트맵 포함
export default function robots() {
  return {
    rules: { userAgent: '*', allow: '/' },
    sitemap: 'https://example.com/sitemap.xml',
  }
}

// ❌ 나쁜 예 - 사이트맵 누락
export default function robots() {
  return {
    rules: { userAgent: '*', allow: '/' },
    // sitemap 누락
  }
}
```

---

## 와일드카드 패턴

### 경로 패턴

```tsx
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: [
        '/admin/*', // /admin/ 하위 모든 경로
        '/*.json$', // 모든 JSON 파일
        '/api/*/secret', // 중간에 * 사용
        '/search?*', // 쿼리 파라미터
      ],
    },
  }
}
```

### 특정 파일 타입 차단

```tsx
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: [
        '/*.pdf$',
        '/*.doc$',
        '/*.xls$',
        '/*.zip$',
      ],
    },
  }
}
```

---

## 보안 고려사항

### ⚠️ robots.txt는 보안 수단이 아닙니다

```tsx
// ❌ 나쁜 예 - robots.txt로 보안 의존
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      disallow: '/admin/', // 보안 의존 ❌
    },
  }
}

// ✅ 올바른 방법 - 인증/권한 검사 필수
// app/admin/layout.tsx
import { auth } from '@/lib/auth'
import { redirect } from 'next/navigation'

export default async function AdminLayout({ children }) {
  const session = await auth()

  if (!session || !session.user.isAdmin) {
    redirect('/login')
  }

  return <>{children}</>
}
```

**중요:** robots.txt는 단지 **권장사항**입니다. 악의적인 크롤러는 무시할 수 있습니다. 실제 보안은 인증/권한 검사로 구현해야 합니다.

---

## 검증 및 테스트

### 로컬에서 확인

```bash
# 개발 서버 실행
npm run dev

# 브라우저에서 확인
http://localhost:3000/robots.txt
```

### Google Search Console 테스트

1. Google Search Console 로그인
2. 속성 선택
3. 설정 → robots.txt 테스터
4. URL 입력하여 차단 여부 확인

### 명령줄 테스트

```bash
# robots.txt 다운로드
curl https://example.com/robots.txt

# 특정 경로 테스트
curl -A "Googlebot" https://example.com/admin/
```

---

## 일반적인 봇 User-Agent

### 주요 검색 엔진

```tsx
export default function robots() {
  return {
    rules: [
      {
        userAgent: [
          'Googlebot',        // Google
          'Bingbot',          // Bing
          'Slurp',            // Yahoo
          'DuckDuckBot',      // DuckDuckGo
          'Baiduspider',      // Baidu
          'YandexBot',        // Yandex
          'Sogou',            // Sogou
        ],
        allow: '/',
      },
    ],
  }
}
```

### 소셜 미디어 크롤러

```tsx
export default function robots() {
  return {
    rules: {
      userAgent: [
        'facebookexternalhit', // Facebook
        'Twitterbot',          // Twitter
        'LinkedInBot',         // LinkedIn
        'Pinterestbot',        // Pinterest
      ],
      allow: '/',
    },
  }
}
```

---

## 디버깅

### 로그 추가

```tsx
// app/robots.ts
export default function robots() {
  const config = {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }

  console.log('robots.txt 설정:', config)

  return config
}
```

### 환경 확인

```tsx
export default function robots() {
  console.log('Environment:', process.env.NODE_ENV)
  console.log('Base URL:', process.env.NEXT_PUBLIC_BASE_URL)

  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
  }
}
```

---

## 캐싱

robots.txt는 기본적으로 **정적으로 생성**되며 빌드 시 한 번 평가됩니다.

### 동적 재검증이 필요한 경우

```tsx
// app/robots.ts
export const revalidate = 3600 // 1시간마다 재생성

export default async function robots() {
  // 동적 데이터 사용
  const config = await getConfigFromDB()

  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: config.blockedPaths,
    },
    sitemap: config.sitemapUrl,
  }
}
```

---

## 다국어 사이트

### 로케일별 사이트맵

```tsx
// app/robots.ts
export default function robots() {
  const locales = ['en', 'ko', 'ja', 'zh']

  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: locales.map(
      (locale) => `https://example.com/${locale}/sitemap.xml`
    ),
  }
}
```

---

## 버전 히스토리

- **v13.3.0**: `robots.js`/`robots.ts` 지원 추가

---

## 관련 문서

- [sitemap.xml](./sitemap.md)
- [generateSitemaps](../../functions/generateSitemaps.md)
- [Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
- [SEO 최적화](https://nextjs.org/learn/seo/introduction-to-seo)
