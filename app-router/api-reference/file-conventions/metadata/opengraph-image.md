---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image
버전: 16.1.6
---

# opengraph-image 및 twitter-image

## 개요

`opengraph-image`와 `twitter-image` 파일 규칙을 사용하면 라우트 세그먼트의 **Open Graph 및 Twitter 소셜 미디어 이미지**를 설정할 수 있습니다.

소셜 플랫폼과 메신저 앱에서 사이트 링크를 공유할 때 표시되는 이미지를 설정하는 데 유용합니다.

---

## 파일 형식

### 이미지 파일 (`.jpg`, `.jpeg`, `.png`, `.gif`)

```
app/
├── opengraph-image.jpg          # Open Graph 이미지
├── twitter-image.jpg            # Twitter 이미지
└── blog/
    ├── opengraph-image.png      # 블로그 섹션 OG 이미지
    └── [slug]/
        └── opengraph-image.jpg  # 개별 포스트 OG 이미지
```

### 코드 기반 생성 (`.js`, `.ts`, `.tsx`)

```
app/
├── opengraph-image.tsx
├── twitter-image.tsx
└── blog/
    └── [slug]/
        └── opengraph-image.tsx
```

---

## 이미지 파일 사용

### 정적 이미지

```
app/
├── opengraph-image.jpg          # 1200x630 권장
└── twitter-image.jpg            # 1200x630 권장
```

생성되는 HTML:

```html
<meta property="og:image" content="<generated-url>" />
<meta property="og:image:type" content="image/jpeg" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />

<meta name="twitter:image" content="<generated-url>" />
<meta name="twitter:image:type" content="image/jpeg" />
<meta name="twitter:image:width" content="1200" />
<meta name="twitter:image:height" content="1200" />
```

---

## 코드 기반 생성

### 기본 구조 (.tsx/.jsx)

```tsx
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const runtime = 'edge' // 선택사항

export const alt = '내 사이트'
export const size = {
  width: 1200,
  height: 630,
}
export const contentType = 'image/png'

export default async function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        내 사이트
      </div>
    ),
    {
      ...size,
    }
  )
}
```

### Props

```tsx
export default async function Image({
  params,
}: {
  params: Promise<{ slug: string }>
})
```

| Props | 타입 | 설명 |
|-------|------|------|
| `params` | `Promise<object>` | 루트 세그먼트부터 현재 세그먼트까지의 동적 경로 매개변수 객체 |

### 반환값

`Image` 함수는 `Blob`, `ArrayBuffer`, `TypedArray`, `DataView`, `ReadableStream`, 또는 `Response`를 반환해야 합니다.

**알아두기:** `ImageResponse`는 이러한 반환 타입을 만족합니다.

---

## 설정 내보내기

### `alt`

```tsx
export const alt = '내 사이트 소개'
```

생성되는 HTML:

```html
<meta property="og:image:alt" content="내 사이트 소개" />
```

### `size`

```tsx
export const size = {
  width: 1200,
  height: 630,
}
```

생성되는 HTML:

```html
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
```

### `contentType`

```tsx
export const contentType = 'image/png'
```

생성되는 HTML:

```html
<meta property="og:image:type" content="image/png" />
```

---

## 라우트 세그먼트 설정

### `runtime`

```tsx
export const runtime = 'edge' // 'nodejs' (기본값) | 'edge'
```

### `dynamic`

```tsx
export const dynamic = 'force-static' // 'auto' | 'force-static' | 'force-dynamic'
```

### `revalidate`

```tsx
export const revalidate = 3600 // 1시간마다 재생성
```

---

## 실전 예제

### 1. 정적 이미지 파일

```
app/
├── opengraph-image.jpg          # 홈 OG 이미지
├── blog/
│   └── opengraph-image.jpg     # 블로그 섹션 OG 이미지
└── products/
    └── opengraph-image.png     # 제품 섹션 OG 이미지
```

**권장 사이즈:**
- **Open Graph**: 1200 x 630px
- **Twitter**: 1200 x 675px (또는 1:1 비율의 경우 1200 x 1200px)

### 2. 동적 블로그 포스트 OG 이미지

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const runtime = 'edge'

export const alt = '블로그 포스트'
export const size = {
  width: 1200,
  height: 630,
}
export const contentType = 'image/png'

export default async function Image({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await getPost(slug)

  return new ImageResponse(
    (
      <div
        style={{
          height: '100%',
          width: '100%',
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'flex-start',
          justifyContent: 'space-between',
          backgroundColor: '#1a1a1a',
          padding: 80,
        }}
      >
        {/* 제목 */}
        <div
          style={{
            fontSize: 60,
            fontWeight: 'bold',
            color: 'white',
            lineHeight: 1.2,
          }}
        >
          {post.title}
        </div>

        {/* 메타 정보 */}
        <div
          style={{
            display: 'flex',
            alignItems: 'center',
            fontSize: 28,
            color: '#888',
          }}
        >
          <div>{post.author}</div>
          <div style={{ margin: '0 20px' }}>•</div>
          <div>{post.date}</div>
        </div>
      </div>
    ),
    {
      ...size,
    }
  )
}
```

### 3. 커스텀 폰트 사용

```tsx
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og'
import { readFile } from 'node:fs/promises'
import { join } from 'node:path'

export const runtime = 'edge'

export default async function Image() {
  // 폰트 로드
  const interSemiBold = await readFile(
    join(process.cwd(), 'assets/Inter-SemiBold.ttf')
  )

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 60,
          fontFamily: 'Inter',
          background: 'linear-gradient(to bottom right, #667eea 0%, #764ba2 100%)',
          color: '#fff',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        커스텀 폰트 적용
      </div>
    ),
    {
      width: 1200,
      height: 630,
      fonts: [
        {
          name: 'Inter',
          data: interSemiBold,
          style: 'normal',
          weight: 400,
        },
      ],
    }
  )
}
```

### 4. 제품 페이지 OG 이미지

```tsx
// app/products/[id]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const alt = '제품 이미지'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

export default async function Image({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  return new ImageResponse(
    (
      <div
        style={{
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'space-between',
          width: '100%',
          height: '100%',
          backgroundColor: 'white',
          padding: 80,
        }}
      >
        {/* 제품 이미지 */}
        <img
          src={product.imageUrl}
          alt={product.name}
          width={400}
          height={400}
          style={{ objectFit: 'cover', borderRadius: 20 }}
        />

        {/* 제품 정보 */}
        <div
          style={{
            display: 'flex',
            flexDirection: 'column',
            flex: 1,
            marginLeft: 60,
          }}
        >
          <div style={{ fontSize: 48, fontWeight: 'bold', color: '#000' }}>
            {product.name}
          </div>
          <div style={{ fontSize: 36, color: '#666', marginTop: 20 }}>
            {product.description}
          </div>
          <div style={{ fontSize: 56, fontWeight: 'bold', color: '#0070f3', marginTop: 30 }}>
            ${product.price}
          </div>
        </div>
      </div>
    ),
    { ...size }
  )
}
```

### 5. 다국어 지원

```tsx
// app/[locale]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const alt = 'My Site'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

const messages = {
  en: 'Welcome to My Site',
  ko: '내 사이트에 오신 것을 환영합니다',
  ja: '私のサイトへようこそ',
}

export default async function Image({
  params,
}: {
  params: Promise<{ locale: string }>
}) {
  const { locale } = await params
  const message = messages[locale as keyof typeof messages] || messages.en

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 80,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          color: '#000',
        }}
      >
        {message}
      </div>
    ),
    { ...size }
  )
}
```

### 6. 동적 배경 색상

```tsx
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export default async function Image() {
  // 테마 또는 카테고리에 따른 색상
  const categories = {
    tech: { bg: '#667eea', fg: '#fff' },
    design: { bg: '#f093fb', fg: '#000' },
    business: { bg: '#4facfe', fg: '#fff' },
  }

  const theme = categories.tech

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 72,
          background: theme.bg,
          color: theme.fg,
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontWeight: 'bold',
        }}
      >
        Tech Blog
      </div>
    ),
    { width: 1200, height: 630 }
  )
}
```

---

## Twitter 전용 이미지

### 기본 사용법

```tsx
// app/twitter-image.tsx
import { ImageResponse } from 'next/og'

export const alt = 'Twitter 이미지'
export const size = {
  width: 1200,
  height: 675, // Twitter는 1.91:1 비율 권장
}
export const contentType = 'image/png'

export default async function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: 'linear-gradient(to right, #1DA1F2, #14171A)',
          color: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        Twitter 공유
      </div>
    ),
    { ...size }
  )
}
```

---

## 여러 이미지 생성

### `generateImageMetadata` 사용

```tsx
// app/products/[id]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export async function generateImageMetadata({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  return [
    {
      id: 'default',
      alt: product.name,
      size: { width: 1200, height: 630 },
      contentType: 'image/png',
    },
    {
      id: 'square',
      alt: product.name,
      size: { width: 1200, height: 1200 },
      contentType: 'image/png',
    },
  ]
}

export default async function Image({
  params,
  id,
}: {
  params: Promise<{ id: string }>
  id: string
}) {
  const { id: productId } = await params
  const product = await getProduct(productId)

  const isSquare = id === 'square'

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: isSquare ? 100 : 60,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        {product.name}
      </div>
    ),
    {
      width: isSquare ? 1200 : 1200,
      height: isSquare ? 1200 : 630,
    }
  )
}
```

---

## 캐싱 및 재검증

### 정적 생성 (기본값)

```tsx
// app/opengraph-image.tsx
export const dynamic = 'force-static' // 빌드 시 생성

export default async function Image() {
  return new ImageResponse(/* ... */)
}
```

### 동적 생성

```tsx
// app/blog/[slug]/opengraph-image.tsx
export const dynamic = 'force-dynamic' // 매 요청마다 생성

export default async function Image({ params }) {
  const { slug } = await params
  const post = await getPost(slug)

  return new ImageResponse(/* post 데이터 사용 */)
}
```

### 재검증 주기 설정

```tsx
// app/opengraph-image.tsx
export const revalidate = 3600 // 1시간마다 재생성

export default async function Image() {
  const data = await fetchDynamicData()

  return new ImageResponse(/* 동적 데이터 사용 */)
}
```

---

## 모범 사례

### 1. 적절한 이미지 크기

```tsx
// ✅ 좋은 예 - 표준 OG 이미지 크기
export const size = {
  width: 1200,
  height: 630, // 1.91:1 비율
}

// ❌ 나쁜 예 - 비표준 크기
export const size = {
  width: 800,
  height: 400, // 너무 작음
}
```

**권장 크기:**
- **Open Graph**: 1200 x 630px
- **Twitter Summary Large**: 1200 x 675px
- **Twitter Summary**: 1200 x 1200px (정사각형)

### 2. Alt 텍스트 포함

```tsx
// ✅ 좋은 예 - 설명적인 alt 텍스트
export const alt = '블로그 포스트: Next.js 15 새로운 기능'

// ❌ 나쁜 예 - 일반적인 alt 텍스트
export const alt = 'Image'
```

### 3. 성능 최적화

```tsx
// ✅ 좋은 예 - Edge Runtime 사용
export const runtime = 'edge'

export default async function Image() {
  // Edge에서 빠른 이미지 생성
  return new ImageResponse(/* ... */)
}

// ❌ 나쁜 예 - 불필요하게 복잡한 로직
export default async function Image() {
  // 여러 데이터베이스 쿼리 및 복잡한 연산
  const data1 = await query1()
  const data2 = await query2()
  const processed = await heavyProcessing(data1, data2)

  return new ImageResponse(/* ... */)
}
```

### 4. 폰트 크기 및 가독성

```tsx
// ✅ 좋은 예 - 읽기 쉬운 폰트 크기
<div style={{
  fontSize: 60, // 충분히 큼
  lineHeight: 1.2,
}}>
  {title}
</div>

// ❌ 나쁜 예 - 너무 작은 폰트
<div style={{
  fontSize: 20, // 너무 작음
}}>
  {title}
</div>
```

---

## 생성되는 메타 태그

### Open Graph 이미지

```html
<meta property="og:image" content="<generated-url>" />
<meta property="og:image:alt" content="내 사이트" />
<meta property="og:image:type" content="image/png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
```

### Twitter 이미지

```html
<meta name="twitter:image" content="<generated-url>" />
<meta name="twitter:image:alt" content="내 사이트" />
<meta name="twitter:card" content="summary_large_image" />
```

---

## 파일 우선순위

같은 라우트 세그먼트에 여러 OG 이미지 파일이 있는 경우:

1. `opengraph-image.tsx` / `twitter-image.tsx` (동적 생성)
2. `opengraph-image.jpg` / `twitter-image.jpg` (정적 이미지)
3. `opengraph-image.png` / `twitter-image.png`
4. 부모 세그먼트의 이미지 (없는 경우)

---

## 버전 히스토리

- **v13.3.0**: `opengraph-image` 및 `twitter-image` 도입

---

## 관련 문서

- [ImageResponse](../../functions/ImageResponse.md)
- [generateImageMetadata](../../functions/generateImageMetadata.md)
- [generateMetadata](../../functions/generateMetadata.md)
- [Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
