# sitemap.xml

## 개요

`sitemap.xml`은 검색 엔진이 사이트를 더 효율적으로 크롤링할 수 있도록 도와주는 **특수 파일 규칙**입니다. Next.js는 정적 또는 동적 방식으로 사이트맵을 생성할 수 있습니다.

---

## 파일 위치

```
app/
├── sitemap.xml          # 정적 파일
├── sitemap.ts          # 동적 생성
└── sitemap.js          # 동적 생성
```

---

## 정적 Sitemap (sitemap.xml)

### 기본 예제

```xml
<!-- app/sitemap.xml -->
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com</loc>
    <lastmod>2024-01-01</lastmod>
    <changefreq>yearly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/about</loc>
    <lastmod>2024-01-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

---

## 동적 Sitemap (sitemap.ts/js)

### 기본 구조

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://example.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
  ]
}
```

### 반환 타입

```tsx
type Sitemap = Array<{
  url: string
  lastModified?: string | Date
  changeFrequency?: 'always' | 'hourly' | 'daily' | 'weekly' | 'monthly' | 'yearly' | 'never'
  priority?: number
  alternates?: {
    languages?: {
      [locale: string]: string
    }
  }
}>
```

---

## 실전 예제

### 1. 정적 페이지 + 동적 콘텐츠

```tsx
// app/sitemap.ts
import { db } from '@/lib/db'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  // 정적 페이지
  const staticPages = [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly' as const,
      priority: 1,
    },
    {
      url: 'https://example.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly' as const,
      priority: 0.8,
    },
  ]

  // 동적 블로그 포스트
  const posts = await db.post.findMany({
    select: {
      slug: true,
      updatedAt: true,
    },
  })

  const postUrls = posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  }))

  return [...staticPages, ...postUrls]
}
```

### 2. 다국어 사이트맵

```tsx
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await db.post.findMany()

  return posts.map((post) => ({
    url: `https://example.com/ko/blog/${post.slug}`,
    lastModified: post.updatedAt,
    alternates: {
      languages: {
        en: `https://example.com/en/blog/${post.slug}`,
        ko: `https://example.com/ko/blog/${post.slug}`,
        ja: `https://example.com/ja/blog/${post.slug}`,
      },
    },
  }))
}
```

### 3. 이미지 사이트맵

```tsx
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const products = await db.product.findMany({
    include: { images: true },
  })

  return products.map((product) => ({
    url: `https://example.com/products/${product.slug}`,
    lastModified: product.updatedAt,
    images: product.images.map((image) => ({
      url: image.url,
      title: image.alt,
      caption: image.caption,
    })),
  }))
}
```

### 4. 비디오 사이트맵

```tsx
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const videos = await db.video.findMany()

  return videos.map((video) => ({
    url: `https://example.com/videos/${video.slug}`,
    lastModified: video.updatedAt,
    videos: [
      {
        title: video.title,
        thumbnail_loc: video.thumbnailUrl,
        description: video.description,
        content_loc: video.videoUrl,
        duration: video.durationSeconds,
        rating: video.rating,
        view_count: video.views,
      },
    ],
  }))
}
```

### 5. 우선순위 전략

```tsx
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const pages = await db.page.findMany()

  return pages.map((page) => {
    // 페이지 유형에 따른 우선순위 설정
    let priority = 0.5
    let changeFrequency: 'always' | 'hourly' | 'daily' | 'weekly' | 'monthly' | 'yearly' | 'never' = 'monthly'

    if (page.slug === '') {
      // 홈페이지
      priority = 1.0
      changeFrequency = 'daily'
    } else if (page.type === 'product') {
      // 제품 페이지
      priority = 0.9
      changeFrequency = 'daily'
    } else if (page.type === 'blog') {
      // 블로그 포스트
      priority = 0.7
      changeFrequency = 'weekly'
    } else if (page.type === 'legal') {
      // 법적 문서
      priority = 0.3
      changeFrequency = 'yearly'
    }

    return {
      url: `https://example.com/${page.slug}`,
      lastModified: page.updatedAt,
      changeFrequency,
      priority,
    }
  })
}
```

### 6. 조건부 URL 포함

```tsx
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await db.post.findMany()

  return posts
    .filter((post) => {
      // 공개된 포스트만 포함
      return post.published && !post.noindex
    })
    .map((post) => ({
      url: `https://example.com/blog/${post.slug}`,
      lastModified: post.updatedAt,
      changeFrequency: 'weekly' as const,
      priority: post.featured ? 0.9 : 0.7,
    }))
}
```

---

## 여러 사이트맵 생성

### 50,000 URL 이상인 경우

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next'

export async function generateSitemaps() {
  const postCount = await db.post.count()
  const sitemaps = Math.ceil(postCount / 50000)

  return Array.from({ length: sitemaps }, (_, i) => ({
    id: String(i),
  }))
}

export default async function sitemap({
  id,
}: {
  id: string
}): Promise<MetadataRoute.Sitemap> {
  const start = parseInt(id) * 50000
  const end = start + 50000

  const posts = await db.post.findMany({
    skip: start,
    take: 50000,
  })

  return posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}
```

생성되는 URL:
```
/sitemap/0.xml
/sitemap/1.xml
/sitemap/2.xml
```

---

## 캐싱 및 재검증

### 정적 생성 (기본)

```tsx
// app/sitemap.ts
export const revalidate = false // 빌드 시 한 번 생성 (기본값)

export default async function sitemap() {
  const pages = await getPages()
  return pages.map((page) => ({
    url: `https://example.com/${page.slug}`,
    lastModified: page.updatedAt,
  }))
}
```

### 주기적 재검증

```tsx
// app/sitemap.ts
export const revalidate = 3600 // 1시간마다 재생성

export default async function sitemap() {
  const pages = await getPages()
  return pages.map((page) => ({
    url: `https://example.com/${page.slug}`,
    lastModified: page.updatedAt,
  }))
}
```

### 온디맨드 재검증

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost(data: FormData) {
  await db.post.create({ data })

  // 사이트맵 재검증
  revalidatePath('/sitemap.xml')
}
```

---

## 모범 사례

### 1. 타입 안정성

```tsx
// ✅ 좋은 예 - 타입 사용
import type { MetadataRoute } from 'next'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly', // 타입 체크됨
      priority: 1,
    },
  ]
}

// ❌ 나쁜 예 - 타입 없음
export default async function sitemap() {
  return [
    {
      url: 'https://example.com',
      changeFreq: 'yearly', // 오타 감지 안됨
    },
  ]
}
```

### 2. 효율적인 쿼리

```tsx
// ✅ 좋은 예 - 필요한 필드만 선택
export default async function sitemap() {
  const posts = await db.post.findMany({
    select: {
      slug: true,
      updatedAt: true,
    },
    where: {
      published: true,
    },
  })

  return posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}

// ❌ 나쁜 예 - 전체 데이터 조회
export default async function sitemap() {
  const posts = await db.post.findMany() // 모든 필드 가져옴

  return posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
  }))
}
```

### 3. 적절한 우선순위 설정

```tsx
// ✅ 좋은 예 - 페이지 중요도에 따른 우선순위
export default async function sitemap() {
  return [
    { url: 'https://example.com', priority: 1.0 }, // 홈
    { url: 'https://example.com/products', priority: 0.9 }, // 주요 페이지
    { url: 'https://example.com/about', priority: 0.7 }, // 일반 페이지
    { url: 'https://example.com/privacy', priority: 0.3 }, // 법적 문서
  ]
}

// ❌ 나쁜 예 - 모든 페이지 동일 우선순위
export default async function sitemap() {
  return pages.map((page) => ({
    url: page.url,
    priority: 1.0, // 모든 페이지가 최고 우선순위
  }))
}
```

### 4. 변경 빈도 설정

```tsx
// ✅ 좋은 예 - 실제 업데이트 주기 반영
export default async function sitemap() {
  return [
    { url: 'https://example.com', changeFrequency: 'daily' as const }, // 자주 변경
    { url: 'https://example.com/blog', changeFrequency: 'weekly' as const }, // 주간 업데이트
    { url: 'https://example.com/terms', changeFrequency: 'yearly' as const }, // 거의 변경 안됨
  ]
}
```

---

## 환경별 URL

### 환경 변수 사용

```tsx
// app/sitemap.ts
const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://example.com'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await db.post.findMany()

  return posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}
```

### 조건부 기본 URL

```tsx
// app/sitemap.ts
function getBaseUrl() {
  if (process.env.VERCEL_ENV === 'production') {
    return 'https://example.com'
  }
  if (process.env.VERCEL_ENV === 'preview') {
    return `https://${process.env.VERCEL_URL}`
  }
  return 'http://localhost:3000'
}

export default async function sitemap() {
  const baseUrl = getBaseUrl()

  return [
    {
      url: `${baseUrl}`,
      lastModified: new Date(),
    },
  ]
}
```

---

## robots.txt와 통합

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

여러 사이트맵이 있는 경우:

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: [
      'https://example.com/sitemap/0.xml',
      'https://example.com/sitemap/1.xml',
      'https://example.com/sitemap/2.xml',
    ],
  }
}
```

---

## 생성되는 XML 형식

### 기본 사이트맵

```tsx
// app/sitemap.ts
export default function sitemap() {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date('2024-01-01'),
      changeFrequency: 'yearly',
      priority: 1,
    },
  ]
}
```

생성되는 XML:

```xml
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com</loc>
    <lastmod>2024-01-01T00:00:00.000Z</lastmod>
    <changefreq>yearly</changefreq>
    <priority>1</priority>
  </url>
</urlset>
```

### 다국어 사이트맵

```tsx
export default function sitemap() {
  return [
    {
      url: 'https://example.com/ko/blog/post',
      lastModified: new Date(),
      alternates: {
        languages: {
          en: 'https://example.com/en/blog/post',
          ko: 'https://example.com/ko/blog/post',
        },
      },
    },
  ]
}
```

생성되는 XML:

```xml
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://example.com/ko/blog/post</loc>
    <xhtml:link rel="alternate" hreflang="en" href="https://example.com/en/blog/post" />
    <xhtml:link rel="alternate" hreflang="ko" href="https://example.com/ko/blog/post" />
    <lastmod>2024-01-01T00:00:00.000Z</lastmod>
  </url>
</urlset>
```

---

## 제한사항

### URL 개수
- **단일 사이트맵**: 최대 50,000 URL
- **파일 크기**: 최대 50MB (압축 전)
- 초과 시 `generateSitemaps` 사용

### 지원되는 필드
- `url` (필수)
- `lastModified`
- `changeFrequency`
- `priority`
- `alternates.languages`
- `images` (이미지 사이트맵)
- `videos` (비디오 사이트맵)

---

## 디버깅

### 로컬에서 확인

```bash
# 개발 서버 실행
npm run dev

# 브라우저에서 확인
http://localhost:3000/sitemap.xml
```

### 로그 추가

```tsx
// app/sitemap.ts
export default async function sitemap() {
  const posts = await db.post.findMany()
  console.log(`사이트맵에 ${posts.length}개 URL 생성`)

  return posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}
```

### 프로덕션 빌드 확인

```bash
# 빌드
npm run build

# .next/server/app 폴더에서 생성된 사이트맵 확인
```

---

## 검색 엔진 제출

### Google Search Console

1. Search Console에 로그인
2. 속성 선택
3. 사이드바에서 "Sitemaps" 선택
4. 사이트맵 URL 입력: `https://example.com/sitemap.xml`
5. "제출" 클릭

### 수동 크롤링 요청

```bash
# Google
curl https://www.google.com/ping?sitemap=https://example.com/sitemap.xml

# Bing
curl https://www.bing.com/ping?sitemap=https://example.com/sitemap.xml
```

---

## 버전 히스토리

- **v13.3.0**: `sitemap.js`/`sitemap.ts` 지원 추가
- **v13.4.5**: `generateSitemaps` 추가

---

## 관련 문서

- [generateSitemaps](../../functions/generateSitemaps.md)
- [robots.txt](./robots.md)
- [Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
- [SEO 최적화](https://nextjs.org/learn/seo/introduction-to-seo)
