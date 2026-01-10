# generateSitemaps

## 개요

`generateSitemaps`는 대규모 애플리케이션에서 **여러 사이트맵을 동적으로 생성**하는 함수입니다. Google의 50,000 URL 제한을 초과하는 경우 사이트맵을 여러 파일로 분할할 수 있습니다.

---

## 기본 사용법

```tsx
// app/sitemap.ts
export async function generateSitemaps() {
  return [
    { id: '0' },
    { id: '1' },
    { id: '2' },
  ]
}

export default async function sitemap({ id }: { id: string }) {
  // 각 사이트맵에 대한 URL 생성
  const start = parseInt(id) * 50000
  const end = start + 50000
  const posts = await getPosts(start, end)

  return posts.map((post) => ({
    url: `https://example.com/posts/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}
```

---

## 함수 시그니처

```tsx
export async function generateSitemaps(): Promise<Array<{ id: string | number }>>
```

### 반환 값
- `id`를 포함하는 객체 배열을 반환합니다
- 각 `id`는 개별 사이트맵을 나타냅니다

---

## URL 생성 패턴

### 단일 사이트맵 (기본)
```
/sitemap.xml
```

### 여러 사이트맵 (generateSitemaps 사용)
```
/sitemap/0.xml
/sitemap/1.xml
/sitemap/2.xml
```

---

## 실전 예제

### 1. 대량 블로그 포스트

```tsx
// app/sitemap.ts
import { db } from '@/lib/db'

// 사이트맵 ID 생성
export async function generateSitemaps() {
  const totalPosts = await db.post.count()
  const sitemapsNeeded = Math.ceil(totalPosts / 50000)

  return Array.from({ length: sitemapsNeeded }, (_, i) => ({
    id: String(i),
  }))
}

// 각 사이트맵의 URL 생성
export default async function sitemap({ id }: { id: string }) {
  const start = parseInt(id) * 50000
  const end = start + 50000

  const posts = await db.post.findMany({
    skip: start,
    take: 50000,
    orderBy: { createdAt: 'desc' },
  })

  return posts.map((post) => ({
    url: `https://blog.example.com/posts/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }))
}
```

### 2. 다국어 사이트맵

```tsx
// app/sitemap.ts
export async function generateSitemaps() {
  const locales = ['en', 'ko', 'ja', 'zh']

  return locales.map((locale) => ({
    id: locale,
  }))
}

export default async function sitemap({ id }: { id: string }) {
  const locale = id
  const posts = await getPostsByLocale(locale)

  return posts.map((post) => ({
    url: `https://example.com/${locale}/posts/${post.slug}`,
    lastModified: post.updatedAt,
    alternates: {
      languages: {
        en: `https://example.com/en/posts/${post.slug}`,
        ko: `https://example.com/ko/posts/${post.slug}`,
        ja: `https://example.com/ja/posts/${post.slug}`,
        zh: `https://example.com/zh/posts/${post.slug}`,
      },
    },
  }))
}
```

### 3. 카테고리별 사이트맵

```tsx
// app/sitemap.ts
export async function generateSitemaps() {
  const categories = await db.category.findMany({
    select: { slug: true },
  })

  return categories.map((category) => ({
    id: category.slug,
  }))
}

export default async function sitemap({ id }: { id: string }) {
  const categorySlug = id
  const products = await db.product.findMany({
    where: { category: { slug: categorySlug } },
  })

  return products.map((product) => ({
    url: `https://shop.example.com/${categorySlug}/${product.slug}`,
    lastModified: product.updatedAt,
    changeFrequency: 'daily' as const,
    priority: 0.9,
  }))
}
```

### 4. 날짜 기반 분할

```tsx
// app/sitemap.ts
export async function generateSitemaps() {
  const currentYear = new Date().getFullYear()
  const years = Array.from({ length: 5 }, (_, i) => currentYear - i)

  return years.map((year) => ({
    id: String(year),
  }))
}

export default async function sitemap({ id }: { id: string }) {
  const year = parseInt(id)
  const startDate = new Date(year, 0, 1)
  const endDate = new Date(year, 11, 31)

  const articles = await db.article.findMany({
    where: {
      publishedAt: {
        gte: startDate,
        lte: endDate,
      },
    },
  })

  return articles.map((article) => ({
    url: `https://news.example.com/articles/${article.slug}`,
    lastModified: article.publishedAt,
    changeFrequency: 'never' as const,
    priority: 0.7,
  }))
}
```

### 5. 이커머스 제품 사이트맵

```tsx
// app/sitemap.ts
export async function generateSitemaps() {
  const productCount = await db.product.count()
  const pageSize = 50000
  const pages = Math.ceil(productCount / pageSize)

  return Array.from({ length: pages }, (_, i) => ({
    id: String(i),
  }))
}

export default async function sitemap({ id }: { id: string }) {
  const page = parseInt(id)
  const skip = page * 50000

  const products = await db.product.findMany({
    skip,
    take: 50000,
    where: { published: true },
    include: { images: true },
  })

  return products.map((product) => ({
    url: `https://store.example.com/products/${product.slug}`,
    lastModified: product.updatedAt,
    changeFrequency: 'daily' as const,
    priority: product.featured ? 1.0 : 0.8,
    images: product.images.map((img) => ({
      url: img.url,
      title: img.alt,
    })),
  }))
}
```

---

## Sitemap 반환 타입

```tsx
type SitemapEntry = {
  url: string
  lastModified?: string | Date
  changeFrequency?: 'always' | 'hourly' | 'daily' | 'weekly' | 'monthly' | 'yearly' | 'never'
  priority?: number
  alternates?: {
    languages?: {
      [locale: string]: string
    }
  }
  images?: Array<{
    url: string
    title?: string
    caption?: string
  }>
  videos?: Array<{
    title: string
    thumbnail_loc: string
    description: string
  }>
}

export default async function sitemap({ id }): Promise<SitemapEntry[]>
```

---

## 캐싱 및 재검증

### 기본 캐싱

```tsx
// app/sitemap.ts
export const revalidate = 3600 // 1시간마다 재검증

export async function generateSitemaps() {
  return [{ id: '0' }, { id: '1' }]
}

export default async function sitemap({ id }: { id: string }) {
  const posts = await getPosts(id)
  return posts.map((post) => ({
    url: `https://example.com/posts/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}
```

### 동적 재검증

```tsx
// Server Action에서 재검증
import { revalidatePath } from 'next/cache'

export async function createPost(data: FormData) {
  // 포스트 생성
  await db.post.create({ data })

  // 사이트맵 재검증
  revalidatePath('/sitemap/[id].xml', 'page')
}
```

---

## 모범 사례

### 1. 페이지네이션 크기 최적화

```tsx
// ✅ 좋은 예 - 50,000 URL 이하
export async function generateSitemaps() {
  const totalItems = await db.item.count()
  const pageSize = 50000
  const pages = Math.ceil(totalItems / pageSize)

  return Array.from({ length: pages }, (_, i) => ({ id: String(i) }))
}

// ❌ 나쁜 예 - 제한 초과
export async function generateSitemaps() {
  return [{ id: 'all' }] // 100,000개 URL 반환
}
```

### 2. 효율적인 데이터베이스 쿼리

```tsx
// ✅ 좋은 예 - 필요한 필드만 선택
export default async function sitemap({ id }: { id: string }) {
  const posts = await db.post.findMany({
    select: {
      slug: true,
      updatedAt: true,
    },
    skip: parseInt(id) * 50000,
    take: 50000,
  })

  return posts.map((post) => ({
    url: `https://example.com/posts/${post.slug}`,
    lastModified: post.updatedAt,
  }))
}

// ❌ 나쁜 예 - 모든 필드 가져옴
export default async function sitemap({ id }: { id: string }) {
  const posts = await db.post.findMany() // 전체 데이터 조회
  return posts.map((post) => ({ url: post.url }))
}
```

### 3. 적절한 재검증 주기

```tsx
// ✅ 좋은 예 - 콘텐츠 업데이트 주기에 맞춤
export const revalidate = 86400 // 24시간 (하루에 한 번 업데이트되는 콘텐츠)

// ❌ 나쁜 예 - 너무 짧은 재검증 주기
export const revalidate = 60 // 1분 (불필요한 서버 부하)
```

### 4. 타입 안정성

```tsx
// ✅ 좋은 예 - TypeScript 타입 사용
import type { MetadataRoute } from 'next'

export default async function sitemap({ id }: { id: string }): Promise<MetadataRoute.Sitemap> {
  // ...
}

// ❌ 나쁜 예 - 타입 없음
export default async function sitemap({ id }) {
  // 타입 안정성 없음
}
```

---

## 성능 최적화

### 병렬 처리

```tsx
// app/sitemap.ts
export default async function sitemap({ id }: { id: string }) {
  const page = parseInt(id)
  const skip = page * 50000

  // 병렬로 여러 쿼리 실행
  const [posts, products, pages] = await Promise.all([
    db.post.findMany({ skip, take: 16666 }),
    db.product.findMany({ skip, take: 16666 }),
    db.page.findMany({ skip, take: 16666 }),
  ])

  return [
    ...posts.map((post) => ({
      url: `https://example.com/blog/${post.slug}`,
      lastModified: post.updatedAt,
    })),
    ...products.map((product) => ({
      url: `https://example.com/shop/${product.slug}`,
      lastModified: product.updatedAt,
    })),
    ...pages.map((page) => ({
      url: `https://example.com/${page.slug}`,
      lastModified: page.updatedAt,
    })),
  ]
}
```

### 데이터베이스 인덱스

```sql
-- 사이트맵 쿼리 성능 향상을 위한 인덱스
CREATE INDEX idx_posts_created_updated ON posts(createdAt, updatedAt);
CREATE INDEX idx_posts_slug ON posts(slug);
```

---

## 제한사항

### URL 제한
- **각 사이트맵 파일**: 최대 50,000 URL
- **파일 크기**: 최대 50MB (압축 전)
- Google 권장사항을 따릅니다

### 동적 경로
```tsx
// ❌ 동적 경로에서 generateSitemaps 사용 불가
// app/[locale]/sitemap.ts
export async function generateSitemaps() {
  // 작동하지 않음
}

// ✅ 루트 레벨에서 사용
// app/sitemap.ts
export async function generateSitemaps() {
  // 올바름
}
```

---

## 디버깅

### 개발 환경에서 확인

```bash
# 사이트맵 URL 확인
http://localhost:3000/sitemap/0.xml
http://localhost:3000/sitemap/1.xml
```

### 로그 추가

```tsx
export async function generateSitemaps() {
  const sitemaps = [{ id: '0' }, { id: '1' }]
  console.log('생성된 사이트맵:', sitemaps)
  return sitemaps
}

export default async function sitemap({ id }: { id: string }) {
  console.log('사이트맵 생성 중:', id)
  const urls = await getUrls(id)
  console.log(`사이트맵 ${id}에 ${urls.length}개 URL`)
  return urls
}
```

---

## robots.txt와 함께 사용

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

또는 동적으로:

```tsx
// app/robots.ts
import { db } from '@/lib/db'

export default async function robots() {
  const sitemapCount = await db.post.count()
  const sitemaps = Math.ceil(sitemapCount / 50000)

  return {
    rules: {
      userAgent: '*',
      allow: '/',
    },
    sitemap: Array.from({ length: sitemaps }, (_, i) =>
      `https://example.com/sitemap/${i}.xml`
    ),
  }
}
```

---

## 버전 히스토리

- **v13.3.0**: `generateSitemaps` 도입
- **v13.4.0**: TypeScript 지원 개선

---

## 관련 문서

- [sitemap.xml 파일 규칙](../file-conventions/metadata/sitemap.md)
- [robots.txt 파일 규칙](../file-conventions/metadata/robots.md)
- [generateMetadata](./generateMetadata.md)
- [Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
