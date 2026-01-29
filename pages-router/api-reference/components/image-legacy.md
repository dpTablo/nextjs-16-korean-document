# Image (Legacy)

> **경고**: 이 컴포넌트는 Next.js 13부터 레거시로 분류되었습니다. 새로운 프로젝트에서는 [`next/image`](./image.md)를 사용하세요.

`next/legacy/image` 컴포넌트는 Next.js 12 이전의 Image 최적화 API를 제공합니다. 기존 프로젝트의 하위 호환성을 위해 유지됩니다.

```jsx
import Image from 'next/legacy/image'

export default function Page() {
  return (
    <Image
      src="/profile.png"
      width={500}
      height={500}
      alt="저자 사진"
    />
  )
}
```

---

## Props 참조

### 필수 Props

#### `src`

이미지 소스입니다. 다음 중 하나일 수 있습니다:
- **내부 경로 문자열**: `<Image src="/profile.png" />`
- **절대 외부 URL**: `domains` 설정 필요
- **정적 import**: `import profile from './profile.png'`

#### `width` 및 `height`

픽셀 단위의 이미지 크기입니다. 정적 import 시에는 자동으로 계산됩니다.

```jsx
<Image src="/profile.png" width={500} height={500} />
```

---

### 선택적 Props

#### `layout`

이미지 레이아웃 동작을 지정합니다:

```jsx
<Image src="/profile.png" layout="responsive" />
```

**옵션:**
- `'intrinsic'`: 기본값. 원본 크기까지 축소, 확대 안 됨
- `'fixed'`: 고정 크기 (width/height 필수)
- `'responsive'`: 컨테이너 너비에 맞춰 조정 (width/height 필수)
- `'fill'`: 부모 요소 크기로 확장 (width/height 불필요)

#### `loader`

커스텀 이미지 URL 생성 함수입니다:

```jsx
const imageLoader = ({ src, width, quality }) => {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}

export default function Page() {
  return (
    <Image
      loader={imageLoader}
      src="me.png"
      alt="저자 사진"
      width={500}
      height={500}
    />
  )
}
```

#### `sizes`

다양한 breakpoint에서 이미지 크기를 정의합니다:

```jsx
<Image
  src="/example.png"
  layout="responsive"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

#### `quality`

1~100 사이의 정수입니다. 최적화된 이미지 품질을 설정합니다 (기본값: 75):

```jsx
<Image quality={90} />
```

#### `priority`

이미지를 우선 로드합니다. `loading="eager"`와 preload를 사용합니다:

```jsx
<Image priority />
```

**사용해야 하는 경우:**
- LCP(Largest Contentful Paint) 요소
- 뷰포트 위쪽(above the fold)

#### `placeholder`

로드 중 표시할 플레이스홀더입니다:

```jsx
<Image placeholder="blur" />
```

- `'empty'`: 플레이스홀더 없음 (기본값)
- `'blur'`: 흐린 이미지 (`blurDataURL` 필요)

#### `blurDataURL`

로드 전 플레이스홀더용 Data URL입니다:

```jsx
<Image
  src="/profile.png"
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
  width={500}
  height={500}
/>
```

**자동 설정:**
정적 import한 `jpg`, `png`, `webp`, `avif` 파일은 자동으로 생성됩니다.

#### `lazyBoundary`

지연 로딩 감지 경계를 설정합니다:

```jsx
<Image lazyBoundary="200px" />
```

기본값: `"200px"`

#### `lazyRoot`

스크롤 감지를 위한 루트 요소를 지정합니다:

```jsx
<Image lazyRoot={ref} />
```

#### `unoptimized`

이미지 최적화를 비활성화합니다:

```jsx
<Image unoptimized />
```

#### `onLoadingComplete`

이미지 로드 완료 시 호출됩니다:

```jsx
<Image
  onLoadingComplete={(result) => {
    console.log(result.naturalWidth, result.naturalHeight)
  }}
/>
```

#### `onLoad`

이미지 로드 시 호출됩니다:

```jsx
<Image onLoad={(e) => console.log(e.target.naturalWidth)} />
```

#### `onError`

이미지 로드 실패 시 호출됩니다:

```jsx
<Image onError={(e) => console.error(e.target.id)} />
```

#### `loading`

이미지 로드 동작을 지정합니다:

```jsx
<Image loading="lazy" />
```

- `'lazy'`: 뷰포트 근처에 도달할 때까지 로드 연기 (기본값)
- `'eager'`: 즉시 로드

#### `objectFit`

`layout="fill"` 사용 시 이미지 맞춤 방식을 지정합니다:

```jsx
<Image layout="fill" objectFit="cover" />
```

- `'fill'`: 이미지 늘려서 채우기
- `'contain'`: 종횡비 유지하며 맞추기
- `'cover'`: 종횡비 유지하며 채우고 자르기
- `'none'`: 원본 크기 유지
- `'scale-down'`: none과 contain 중 작은 것

#### `objectPosition`

`layout="fill"` 사용 시 이미지 위치를 지정합니다:

```jsx
<Image layout="fill" objectPosition="center" />
```

기본값: `"50% 50%"`

---

## 설정 옵션 (`next.config.js`)

### `domains`

외부 이미지 URL을 허용합니다:

```js
module.exports = {
  images: {
    domains: ['example.com', 'cdn.example.com'],
  },
}
```

### `loader`

이미지 최적화 서비스를 지정합니다:

```js
module.exports = {
  images: {
    loader: 'cloudinary',
  },
}
```

**옵션:**
- `'default'`: Next.js 내장 Image Optimization API
- `'imgix'`: Imgix
- `'cloudinary'`: Cloudinary
- `'akamai'`: Akamai
- `'custom'`: 커스텀 loader 사용 (`loaderFile` 필요)

### `loaderFile`

커스텀 loader 파일 경로를 지정합니다:

```js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './my/image/loader.js',
  },
}
```

```js
// my/image/loader.js
export default function myImageLoader({ src, width, quality }) {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}
```

### `deviceSizes`

기기 너비 breakpoint를 지정합니다:

```js
module.exports = {
  images: {
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },
}
```

기본값: `[640, 750, 828, 1080, 1200, 1920, 2048, 3840]`

### `imageSizes`

이미지 너비 목록입니다:

```js
module.exports = {
  images: {
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
}
```

기본값: `[16, 32, 48, 64, 96, 128, 256, 384]`

> `deviceSizes`보다 작아야 합니다.

### `formats`

지원할 이미지 포맷입니다:

```js
module.exports = {
  images: {
    formats: ['image/webp'],
  },
}
```

기본값: `['image/webp']`

**AVIF 지원:**

```js
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
  },
}
```

### `minimumCacheTTL`

캐시된 최적화 이미지 TTL입니다 (초 단위):

```js
module.exports = {
  images: {
    minimumCacheTTL: 60,
  },
}
```

기본값: `60`

### `disableStaticImages`

정적 이미지 import를 비활성화합니다:

```js
module.exports = {
  images: {
    disableStaticImages: true,
  },
}
```

### `dangerouslyAllowSVG`

SVG 이미지 지원을 활성화합니다:

```js
module.exports = {
  images: {
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

> SVG는 HTML/CSS 기능을 포함할 수 있으므로 CSP 헤더가 필요합니다.

---

## 사용 예제

### 고정 크기 이미지

```jsx
import Image from 'next/legacy/image'

export default function Page() {
  return (
    <Image
      src="/profile.png"
      width={500}
      height={500}
      alt="저자 사진"
      layout="fixed"
    />
  )
}
```

### 반응형 이미지

```jsx
import Image from 'next/legacy/image'

export default function Page() {
  return (
    <div style={{ width: '100%' }}>
      <Image
        src="/profile.png"
        width={500}
        height={500}
        alt="저자 사진"
        layout="responsive"
      />
    </div>
  )
}
```

### 컨테이너 채우기

```jsx
import Image from 'next/legacy/image'

export default function Page() {
  return (
    <div style={{ position: 'relative', width: '300px', height: '300px' }}>
      <Image
        src="/profile.png"
        alt="저자 사진"
        layout="fill"
        objectFit="cover"
      />
    </div>
  )
}
```

### 정적 Import

```jsx
import Image from 'next/legacy/image'
import profilePic from '../public/me.png'

export default function Page() {
  return (
    <Image
      src={profilePic}
      alt="저자 사진"
      placeholder="blur"
    />
  )
}
```

### 외부 이미지

```jsx
import Image from 'next/legacy/image'

export default function Page() {
  return (
    <Image
      src="https://example.com/profile.png"
      width={500}
      height={500}
      alt="저자 사진"
    />
  )
}
```

**next.config.js 설정:**

```js
module.exports = {
  images: {
    domains: ['example.com'],
  },
}
```

---

## 새 Image 컴포넌트로 마이그레이션

Next.js 13+에서는 새로운 `next/image` API를 사용하는 것이 권장됩니다. 마이그레이션을 돕기 위해 codemod를 제공합니다:

```bash
npx @next/codemod@latest next-image-to-legacy-image .
```

이 명령은 기존 `next/image` import를 `next/legacy/image`로 자동 변경합니다.

### 주요 차이점

**새 `next/image`의 변경사항:**
- `layout` prop 제거 → `fill`, `style`, `sizes` 사용
- `objectFit` prop 제거 → `style` 사용
- `objectPosition` prop 제거 → `style` 사용
- `lazyBoundary` prop 제거
- `lazyRoot` prop 제거
- `loader` 서명 변경
- `alt` prop 필수화

**마이그레이션 예제:**

```jsx
// Before (Legacy)
<Image
  src="/profile.png"
  width={500}
  height={500}
  layout="responsive"
  objectFit="cover"
/>

// After (New)
<Image
  src="/profile.png"
  width={500}
  height={500}
  sizes="100vw"
  style={{
    width: '100%',
    height: 'auto',
    objectFit: 'cover',
  }}
/>
```

---

## 알려진 제한사항

- `layout="fill"`과 함께 `position: static`인 부모 사용 불가
- 정적 import 시 `width`/`height` 재정의 불가
- `priority` 속성과 `loading="lazy"` 동시 사용 불가

---

## 버전 이력

| 버전 | 변경사항 |
|------|---------|
| `v13.0.0` | `next/image`가 재설계되고, 이전 구현은 `next/legacy/image`로 이동 |
| `v12.3.0` | `remotePatterns`, `unoptimized` 설정 추가 |
| `v12.0.0` | AVIF 지원 추가 |
| `v11.1.0` | `onLoadingComplete` 및 `lazyBoundary` 추가 |
| `v11.0.0` | 정적 이미지 import 지원 |
| `v10.0.5` | `loader` prop 추가 |
| `v10.0.0` | `next/image` 도입 |

---

## 관련 문서

- [새 Image 컴포넌트](./image.md)
- [Image Optimization Guide](../../getting-started/09-image-optimization.md)
- [Upgrade Guide](../../upgrading/codemods.md)
