---
원문: https://nextjs.org/docs/app/api-reference/components/image
버전: 16.1.6
---

# Image 컴포넌트 API 참조

## 개요
Next.js `<Image>` 컴포넌트는 HTML `<img>` 요소를 확장하여 자동 이미지 최적화를 제공합니다.

## 핵심 Props

### `src` (필수)
이미지 소스. 다음을 허용:
- 내부 경로 문자열: `src="/profile.png"`
- 절대 외부 URL: `src="https://example.com/profile.png"` (`remotePatterns` 설정 필요)
- 정적 import: `import profile from './profile.png'`

### `alt` (필수)
접근성 및 SEO를 위한 설명 텍스트. 순수 장식용 이미지는 빈 문자열 (`alt=""`) 사용.

### `width` 및 `height`
픽셀 단위의 내재적 이미지 크기. 종횡비 추론 및 레이아웃 시프트 방지에 사용.

**다음의 경우 필수가 아님:**
- 이미지를 정적으로 import한 경우
- `fill` 속성 사용 시

```jsx
<Image src="/profile.png" width={500} height={500} alt="작성자" />
```

### `fill`
이미지를 부모 요소 크기로 확장하는 Boolean.

**요구사항:**
- 부모는 `position: "relative"`, `"fixed"`, 또는 `"absolute"` 필요
- `objectFit` prop으로 크롭 제어:
  - `"contain"`: 축소, 종횡비 유지
  - `"cover"`: 컨테이너 채우기, 필요 시 크롭

```jsx
<div style={{ position: 'relative' }}>
  <Image src="/profile.png" fill objectFit="cover" alt="작성자" />
</div>
```

---

## 로딩 및 성능

### `loading`
이미지 로딩 시작 시점 제어 (기본값: `"lazy"`).
- `"lazy"`: 뷰포트 근처까지 로딩 연기
- `"eager"`: 즉시 로드

### `preload`
이미지를 미리 로드할지 표시 (기본값: `false`).
- `true`: `<head>`에 `<link>` 삽입하여 프리로드
- `false`: 프리로드 안 함

**사용 시기:**
- 이미지가 LCP(Largest Contentful Paint) 요소일 때
- 폴드 위의 히어로 이미지

### `quality`
최적화된 이미지 품질을 설정하는 정수 (1-100, 기본값: 75).

```jsx
<Image quality={75} />
```

---

## 반응형 이미지

### `sizes`
다양한 중단점에서 이미지 크기 정의.

**필수 조건:**
- `fill` prop 사용 시
- CSS로 반응형 이미지 사용 시

```jsx
<Image
  fill
  src="/example.png"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

### `loader`
이미지 URL을 생성하는 커스텀 함수.

```jsx
const imageLoader = ({ src, width, quality }) => {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}

<Image loader={imageLoader} src="me.png" width={500} height={500} alt="작성자" />
```

---

## 시각 효과 및 스타일링

### `placeholder`
로딩 중 플레이스홀더 (기본값: `"empty"`).
- `"empty"`: 플레이스홀더 없음
- `"blur"`: 블러 이미지 (`blurDataURL` 필요)
- `"data:image/..."`: 데이터 URL 플레이스홀더

### `blurDataURL`
블러 플레이스홀더를 위한 데이터 URL. 매우 작은 이미지(≤10px) 권장.

### `style`
이미지 요소의 인라인 CSS 스타일.

```jsx
const imageStyle = {
  borderRadius: '50%',
  border: '1px solid #fff',
}
<Image style={imageStyle} src="..." />
```

---

## 설정 옵션 (`next.config.js`)

### `remotePatterns`
외부 이미지 URL 허용 목록.

```js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
        port: '',
        pathname: '/account123/**',
      },
    ],
  },
}
```

### `deviceSizes`
디바이스 너비 중단점 (기본값: `[640, 750, 828, 1080, 1200, 1920, 2048, 3840]`).

### `imageSizes`
반응형 이미지의 이미지 너비 (기본값: `[32, 48, 64, 96, 128, 256, 384]`).

### `qualities`
허용되는 이미지 품질 값 (기본값: `[75]`). **Next.js 16+에서 필수**.

```js
module.exports = {
  images: {
    qualities: [25, 50, 75, 100],
  },
}
```

### `formats`
지원되는 이미지 형식 (기본값: `['image/webp']`).

```js
// AVIF 및 WebP 폴백
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
  },
}
```

### `minimumCacheTTL`
최적화된 이미지의 캐시 지속 시간(초) (기본값: `14400` = 4시간).

### `unoptimized`
전역적으로 최적화 비활성화.

```js
module.exports = {
  images: {
    unoptimized: true,
  },
}
```

---

## 일반적인 사용 예시

### 기본 이미지
```jsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="/profile.png"
      width={500}
      height={500}
      alt="작성자 사진"
    />
  )
}
```

### 반응형 이미지 (정적 Import)
```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'

export default function Responsive() {
  return (
    <Image
      alt="산"
      src={mountains}
      sizes="100vw"
      style={{
        width: '100%',
        height: 'auto',
      }}
    />
  )
}
```

### 원격 이미지
```jsx
<Image
  src="https://s3.amazonaws.com/my-bucket/profile.png"
  alt="작성자 사진"
  width={500}
  height={500}
/>
```

### 컨테이너 채우기
```jsx
<div style={{ position: 'relative', width: '400px' }}>
  <Image
    alt="산"
    src={mountains}
    fill
    sizes="(min-width: 808px) 50vw, 100vw"
    style={{ objectFit: 'cover' }}
  />
</div>
```

### 블러 플레이스홀더
```jsx
<Image
  alt="산"
  src={mountains}
  placeholder="blur"
  quality={100}
  fill
  sizes="100vw"
  style={{ objectFit: 'cover' }}
/>
```

---

## 보안 설정

### `dangerouslyAllowSVG`
SVG 지원 활성화 (보안상 기본적으로 비활성화).

```js
module.exports = {
  images: {
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

---

## 유틸리티 함수

### `getImageProps()`
플레이스홀더 지원 없이 기본 `<img>` 요소의 props 생성.

```jsx
import { getImageProps } from 'next/image'

const { props } = getImageProps({
  src: 'https://example.com/image.jpg',
  alt: '경치 좋은 산 전망',
  width: 1200,
  height: 800,
})

function ImageWithCaption() {
  return (
    <figure>
      <img {...props} />
      <figcaption>경치 좋은 산 전망</figcaption>
    </figure>
  )
}
```
