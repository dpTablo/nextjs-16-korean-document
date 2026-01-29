---
원문: https://nextjs.org/docs/app/api-reference/functions/generate-image-metadata
버전: 16.1.6
---

# generateImageMetadata

## 개요

`generateImageMetadata`는 단일 라우트 세그먼트에서 **여러 이미지 버전 또는 변형**을 생성하는 Next.js 함수입니다. 아이콘과 같은 하드코딩된 메타데이터 값을 피하는 데 유용합니다.

---

## 주요 특징

- 단일 소스에서 여러 이미지 크기/버전 생성
- 동적 라우트 파라미터 지원
- `id`, `alt`, `size`, `contentType`을 포함한 메타데이터 반환

---

## 함수 시그니처

```tsx
export function generateImageMetadata({
  params,
}: {
  params: Promise<{ slug: string }>
}): Array<ImageMetadata>
```

### 매개변수

- **`params`**: 동적 라우트 파라미터를 포함하는 Promise 객체

### 반환 값

이미지 메타데이터 객체의 배열을 반환합니다.

---

## 반환 객체 속성

| 속성 | 타입 | 필수 여부 | 설명 |
|------|------|----------|------|
| `id` | `string \| number` | ✓ 필수 | 고유 식별자 |
| `alt` | `string` | 선택 | 대체 텍스트 |
| `size` | `{ width: number; height: number }` | 선택 | 이미지 크기 |
| `contentType` | `string` | 선택 | MIME 타입 (예: `image/png`) |

---

## 이미지 생성 함수

default export는 다음 props를 받습니다:

```tsx
export default async function Icon({
  id,
  params
}: {
  id: Promise<string | number>
  params?: Promise<{ slug: string }>
}) {
  const iconId = await id
  const routeParams = await params
  // 이미지 생성 로직
}
```

> **중요:** v16.0.0부터 `id`와 `params`는 Promise이므로 사용 전에 `await`해야 합니다.

---

## 기본 사용 예제

### 1. 여러 아이콘 크기 생성

```tsx
// app/icon.tsx
import { ImageResponse } from 'next/og'

export function generateImageMetadata() {
  return [
    {
      id: 'small',
      size: { width: 48, height: 48 },
      contentType: 'image/png',
    },
    {
      id: 'medium',
      size: { width: 72, height: 72 },
      contentType: 'image/png',
    },
    {
      id: 'large',
      size: { width: 144, height: 144 },
      contentType: 'image/png',
    },
  ]
}

export default async function Icon({
  id
}: {
  id: Promise<string>
}) {
  const iconId = await id

  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: iconId === 'small' ? 32 : iconId === 'medium' ? 48 : 96,
          background: '#000',
          color: '#fff',
        }}
      >
        Icon {iconId}
      </div>
    )
  )
}
```

**생성되는 경로:**
- `/icon?id=small` → 48x48 아이콘
- `/icon?id=medium` → 72x72 아이콘
- `/icon?id=large` → 144x144 아이콘

---

## 실용적인 예제

### 2. 동적 라우트와 함께 사용

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export function generateImageMetadata({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  return [
    {
      id: 'og',
      alt: 'Open Graph Image',
      size: { width: 1200, height: 630 },
      contentType: 'image/png',
    },
    {
      id: 'twitter',
      alt: 'Twitter Card Image',
      size: { width: 1200, height: 600 },
      contentType: 'image/png',
    },
  ]
}

export default async function Image({
  id,
  params,
}: {
  id: Promise<string>
  params: Promise<{ slug: string }>
}) {
  const imageId = await id
  const { slug } = await params

  const post = await getPost(slug)

  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
          justifyContent: 'center',
          background: 'linear-gradient(to bottom, #1e3a8a, #312e81)',
          color: 'white',
        }}
      >
        <h1 style={{ fontSize: 60 }}>{post.title}</h1>
        <p style={{ fontSize: 30 }}>Type: {imageId}</p>
      </div>
    ),
    {
      width: imageId === 'og' ? 1200 : 1200,
      height: imageId === 'og' ? 630 : 600,
    }
  )
}
```

### 3. 여러 Open Graph 이미지

```tsx
// app/opengraph-image.tsx
export function generateImageMetadata() {
  return [
    {
      id: 'default',
      alt: 'Default OG Image',
      size: { width: 1200, height: 630 },
    },
    {
      id: 'dark',
      alt: 'Dark Mode OG Image',
      size: { width: 1200, height: 630 },
    },
  ]
}

export default async function OGImage({
  id
}: {
  id: Promise<string>
}) {
  const theme = await id

  return new ImageResponse(
    (
      <div
        style={{
          background: theme === 'dark' ? '#000' : '#fff',
          color: theme === 'dark' ? '#fff' : '#000',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: 60,
        }}
      >
        My Website - {theme === 'dark' ? 'Dark' : 'Light'}
      </div>
    )
  )
}
```

### 4. 제품별 이미지 생성

```tsx
// app/products/[id]/icon.tsx
export async function generateImageMetadata({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  return product.variants.map((variant, index) => ({
    id: variant.id,
    alt: `${product.name} - ${variant.name}`,
    size: { width: 48, height: 48 },
    contentType: 'image/png',
  }))
}

export default async function ProductIcon({
  id,
  params,
}: {
  id: Promise<string>
  params: Promise<{ id: string }>
}) {
  const variantId = await id
  const { id: productId } = await params

  const variant = await getProductVariant(productId, variantId)

  return new ImageResponse(
    (
      <div
        style={{
          background: variant.color,
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          color: '#fff',
          fontSize: 24,
        }}
      >
        {variant.name[0]}
      </div>
    )
  )
}
```

---

## 주요 포인트

### 1. Promise 처리 (v16+)

```tsx
// ✅ 올바른 사용 (v16+)
export default async function Icon({
  id,
  params
}: {
  id: Promise<string>
  params?: Promise<{ slug: string }>
}) {
  const iconId = await id  // Promise를 await
  const routeParams = await params

  return new ImageResponse(/* ... */)
}
```

```tsx
// ❌ 잘못된 사용 (v16+에서 에러)
export default function Icon({ id }: { id: string }) {
  // id는 이제 Promise입니다
  return new ImageResponse(/* ... */)
}
```

### 2. 동적 메타데이터 생성

```tsx
export async function generateImageMetadata({ params }: Props) {
  const { slug } = await params
  const data = await fetchData(slug)

  // 데이터에 기반한 동적 메타데이터
  return data.images.map((img, i) => ({
    id: img.id,
    alt: img.alt,
    size: { width: img.width, height: img.height },
  }))
}
```

### 3. 고유 ID 필수

```tsx
// ✅ 올바른 - 모든 항목이 고유한 id
export function generateImageMetadata() {
  return [
    { id: 'icon-1', size: { width: 32, height: 32 } },
    { id: 'icon-2', size: { width: 64, height: 64 } },
  ]
}

// ❌ 잘못된 - 중복된 id
export function generateImageMetadata() {
  return [
    { id: 'icon', size: { width: 32, height: 32 } },
    { id: 'icon', size: { width: 64, height: 64 } }, // 중복!
  ]
}
```

---

## 사용 사례

| 사용 사례 | 설명 |
|----------|------|
| **여러 아이콘 크기** | 다양한 디바이스/플랫폼용 파비콘 크기 생성 |
| **테마 변형** | 라이트/다크 모드 이미지 변형 |
| **소셜 미디어 최적화** | Open Graph, Twitter Card 등 여러 포맷 |
| **제품 변형** | 제품의 여러 색상/스타일 아이콘 |
| **국제화** | 언어별 이미지 변형 |

---

## 파일 규칙과 함께 사용

`generateImageMetadata`는 다음 파일 규칙과 함께 사용할 수 있습니다:

- `icon.tsx` / `apple-icon.tsx`
- `opengraph-image.tsx`
- `twitter-image.tsx`

---

## 베스트 프랙티스

### ✅ 권장사항

1. **의미 있는 ID 사용**
   ```tsx
   return [
     { id: 'icon-16x16', size: { width: 16, height: 16 } },
     { id: 'icon-32x32', size: { width: 32, height: 32 } },
   ]
   ```

2. **적절한 contentType 지정**
   ```tsx
   return [
     { id: 'png-version', contentType: 'image/png' },
     { id: 'webp-version', contentType: 'image/webp' },
   ]
   ```

3. **alt 텍스트 제공**
   ```tsx
   return [
     { id: 'logo', alt: '회사 로고', size: { width: 48, height: 48 } },
   ]
   ```

### ❌ 피해야 할 사항

1. **너무 많은 변형 생성**
   ```tsx
   // ❌ 과도한 변형은 빌드 시간 증가
   return Array.from({ length: 100 }, (_, i) => ({
     id: `icon-${i}`,
     size: { width: 16 + i, height: 16 + i },
   }))
   ```

2. **Promise await 누락 (v16+)**
   ```tsx
   // ❌ v16+에서 에러
   export default function Icon({ id }) {
     return new ImageResponse(<div>{id}</div>) // id는 Promise
   }
   ```

---

## 타입 정의

```tsx
interface ImageMetadata {
  id: string | number
  alt?: string
  size?: {
    width: number
    height: number
  }
  contentType?: string
}

type GenerateImageMetadata = (props: {
  params: Promise<Record<string, string>>
}) => Array<ImageMetadata> | Promise<Array<ImageMetadata>>
```

---

## 관련 API

- [`ImageResponse`](./ImageResponse.md) - 동적 이미지 생성
- [`generateSitemaps`](./generateSitemaps.md) - 여러 사이트맵 생성
- [Metadata Files](../file-conventions/metadata/opengraph-image.md) - Open Graph 이미지 파일 규칙
- [`generateMetadata`](./generateMetadata.md) - 동적 메타데이터 생성

---

## 버전 정보

- **도입 버전:** Next.js 13.3.0
- **Promise 반환:** Next.js 16.0.0 (Breaking Change)
- **현재 상태:** Stable

---

## 마이그레이션 가이드 (v15 → v16)

### Before (v15)

```tsx
export default function Icon({ id }: { id: string }) {
  return new ImageResponse(
    <div>{id}</div>
  )
}
```

### After (v16)

```tsx
export default async function Icon({ id }: { id: Promise<string> }) {
  const iconId = await id  // Promise를 await
  return new ImageResponse(
    <div>{iconId}</div>
  )
}
```

---

## 요약

- **용도:** 단일 경로에서 여러 이미지 변형 생성
- **주 사용처:** 아이콘, Open Graph 이미지, 소셜 미디어 카드
- **필수 반환 속성:** `id`
- **선택 반환 속성:** `alt`, `size`, `contentType`
- **중요 변경사항 (v16):** `id`와 `params`가 Promise로 변경
- **권장사항:** 의미 있는 ID 사용, 적절한 변형 개수 유지
