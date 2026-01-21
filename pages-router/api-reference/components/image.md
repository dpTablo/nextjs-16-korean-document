# <Image>

Next.js Image 컴포넌트는 HTML `<img>` 요소를 확장하여 자동 이미지 최적화를 제공합니다.

```jsx
import Image from 'next/image'

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
- **절대 외부 URL**: `remotePatterns` 설정 필요
- **정적 import**: `import profile from './profile.png'`

> **보안 참고**: 기본 loader는 `src` 이미지를 가져올 때 헤더를 전달하지 않습니다. 인증이 필요한 경우 `unoptimized` 속성 사용을 권장합니다.

#### `alt`

이미지에 대한 접근성 설명입니다. 스크린 리더와 검색 엔진에서 사용됩니다.
- 페이지 의미 변경 없이 이미지를 대체할 수 있는 텍스트
- 장식용 이미지의 경우: `alt=""`
- [WHATWG 가이드라인](https://html.spec.whatwg.org/multipage/images.html#general-guidelines) 참고

#### `width` 및 `height`

픽셀 단위의 고유 이미지 크기입니다. 렌더링 크기가 아닌 종횡비 계산에 사용됩니다.

다음 경우를 제외하고 필수입니다:
- 정적 import 이미지
- `fill` 속성 사용 시

```jsx
<Image src="/profile.png" width={500} height={500} />
```

---

### 선택적 Props

#### `fill`

부모 요소 크기로 이미지를 확장합니다. 부모 요소의 position 지정이 필요합니다:
- 부모: `position: "relative"`, `"fixed"`, `"absolute"`
- 이미지: 기본 `position: "absolute"`

**Object Fit 옵션:**
- `"contain"`: 종횡비를 유지하며 축소
- `"cover"`: 컨테이너를 채우고 자르기

```jsx
<Image src="/profile.png" fill={true} />
```

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

다양한 breakpoint에서 이미지 크기를 정의합니다. 반응형 레이아웃에 사용됩니다:

```jsx
<Image
  fill
  src="/example.png"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

**사용해야 하는 경우:**
- `fill` prop 사용 시
- CSS로 반응형 구성 시

생략 시 브라우저는 `100vw`로 가정하여 불필요하게 큰 이미지를 다운로드할 수 있습니다.

#### `quality`

1~100 사이의 정수입니다. 최적화된 이미지 품질을 설정합니다 (기본값: 75):

```jsx
<Image quality={75} />
```

> 원본이 이미 낮은 품질이면 높은 값을 설정해도 개선 효과가 없습니다.

#### `style`

CSS 스타일을 적용합니다:

```jsx
const imageStyle = {
  borderRadius: '50%',
  border: '1px solid #fff',
  width: '100px',
  height: 'auto',
}

export default function ProfileImage() {
  return <Image src="..." style={imageStyle} />
}
```

> 커스텀 width 설정 시 `height: 'auto'`를 함께 설정하여 종횡비를 유지하세요.

#### `preload`

이미지 사전 로드 여부입니다 (Next.js 16+에서 `priority`를 대체):

```jsx
<Image preload={false} />
```

- `true`: `<head>`에 `<link>`를 삽입하여 사전 로드
- `false`: 사전 로드 안 함

**사용해야 하는 경우:**
- LCP(Largest Contentful Paint) 요소
- 뷰포트 위쪽(fold above)
- `<head>`에서 로드를 시작하길 원할 때

**사용하지 말아야 하는 경우:**
- 뷰포트에 따라 LCP가 달라질 때
- `loading` 또는 `fetchPriority` 사용 시

일반적으로 `loading="eager"` 또는 `fetchPriority="high"`를 선호합니다.

#### `priority` (deprecated)

> Next.js 16부터 deprecated되었습니다. `preload`를 사용하세요.

#### `loading`

이미지 로드 시작 시점을 지정합니다:

```jsx
<Image loading="lazy" />
```

- `lazy`: 뷰포트 근처에 도달할 때까지 로드 연기 (기본값)
- `eager`: 즉시 로드

#### `placeholder`

로드 중 표시할 플레이스홀더입니다:

```jsx
<Image placeholder="empty" />
```

- `empty`: 플레이스홀더 없음 (기본값)
- `blur`: 흐린 이미지 (`blurDataURL` 필요)
- `data:image/...`: Data URL 사용

#### `blurDataURL`

로드 전 플레이스홀더용 Data URL입니다:

```jsx
<Image placeholder="blur" blurDataURL="..." />
```

**자동 설정:**
정적 import한 `jpg`, `png`, `webp`, `avif` 파일은 자동으로 생성됩니다 (애니메이션 제외).

**수동 설정:**
동적/원격 이미지는 직접 제공해야 합니다. 생성 도구:
- [png-pixel.com](https://png-pixel.com)
- [Plaiceholder](https://github.com/joe-bell/plaiceholder)

> 큰 blurDataURL은 성능을 저해합니다. 작고 간단하게 유지하세요.

#### `onLoad`

이미지가 완전히 로드되고 플레이스홀더가 제거된 후 호출됩니다:

```jsx
<Image onLoad={(e) => console.log(e.target.naturalWidth)} />
```

#### `onError`

이미지 로드 실패 시 호출됩니다:

```jsx
<Image onError={(e) => console.error(e.target.id)} />
```

#### `unoptimized`

이미지 최적화를 비활성화합니다 (작은 이미지, SVG, GIF에 유용):

```jsx
<Image {...props} unoptimized />
```

또는 `next.config.js`에서 전역으로 설정할 수 있습니다:

```js
module.exports = {
  images: {
    unoptimized: true,
  },
}
```

#### `overrideSrc`

생성되는 `src` 속성을 재정의합니다. SEO 목적(기존 이미지 순위 유지)에 유용합니다:

```jsx
<Image src="/profile.jpg" overrideSrc="/override.jpg" />
```

```html
<!-- srcset은 /profile.jpg 사용, src만 /override.jpg -->
<img
  srcset="/_next/image?url=%2Fprofile.jpg&w=640&q=75 1x, ..."
  src="/override.jpg"
/>
```

#### `decoding`

이미지 디코딩 힌트를 지정합니다:

```jsx
<Image decoding="async" />
```

- `async`: 비동기 디코딩 (기본값)
- `sync`: 동기 디코딩
- `auto`: 브라우저 선택

---

## 설정 옵션 (`next.config.js`)

### `localPatterns`

특정 로컬 경로에서만 최적화를 허용합니다:

```js
module.exports = {
  images: {
    localPatterns: [
      {
        pathname: '/assets/images/**',
        search: '',
      },
    ],
  },
}
```

### `remotePatterns`

외부 이미지 URL 패턴을 화이트리스트로 지정합니다 (필수 권장):

```js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
        port: '',
        pathname: '/account123/**',
        search: '',
      },
    ],
  },
}
```

**와일드카드 패턴:**
- `*`: 단일 경로 세그먼트/서브도메인
- `**`: 임의 개수 경로/서브도메인 (끝에만 가능)

```js
// 서브도메인 매칭
remotePatterns: [
  {
    protocol: 'https',
    hostname: '**.example.com',
  },
]
```

### `loaderFile`

커스텀 이미지 최적화 서비스를 사용합니다:

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

기기 너비 breakpoint를 지정합니다 (기본값: `[640, 750, 828, 1080, 1200, 1920, 2048, 3840]`):

```js
module.exports = {
  images: {
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },
}
```

### `imageSizes`

이미지 너비 목록입니다 (기본값: `[32, 48, 64, 96, 128, 256, 384]`):

```js
module.exports = {
  images: {
    imageSizes: [32, 48, 64, 96, 128, 256, 384],
  },
}
```

> `deviceSizes`보다 작아야 합니다.

### `qualities`

허용할 이미지 품질 값입니다 (Next.js 16+에서 필수):

```js
// 기본값
module.exports = {
  images: {
    qualities: [75],
  },
}
```

```js
// 복수 품질 허용
module.exports = {
  images: {
    qualities: [25, 50, 75, 100],
  },
}
```

`quality` prop이 배열 값과 맞지 않으면 가장 가까운 값이 사용됩니다.

### `formats`

지원할 이미지 포맷입니다 (기본값: `['image/webp']`):

```js
module.exports = {
  images: {
    formats: ['image/webp'],
  },
}
```

**AVIF 지원:**

```js
// AVIF 우선, 미지원 시 WebP로 폴백
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
  },
}
```

> AVIF는 인코딩이 50% 느리지만 압축은 20% 작습니다. 캐시 저장소가 증가할 수 있습니다.

### `minimumCacheTTL`

캐시된 최적화 이미지 TTL입니다 (초 단위, 기본값: 14400 = 4시간):

```js
module.exports = {
  images: {
    minimumCacheTTL: 14400,
  },
}
```

```js
// 31일로 증가
module.exports = {
  images: {
    minimumCacheTTL: 2678400,
  },
}
```

> 캐시 무효화 메커니즘이 없습니다. `src` prop을 변경하거나 `<distDir>/cache/images`를 수동으로 삭제해야 합니다.

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

SVG 이미지 지원을 활성화합니다 (기본값: false):

```js
module.exports = {
  images: {
    dangerouslyAllowSVG: true,
    contentDispositionType: 'attachment',
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

> SVG는 HTML/CSS 기능을 포함할 수 있으므로 CSP 헤더가 필요합니다.

### `domains` (deprecated)

> Next.js 14부터 deprecated되었습니다. `remotePatterns` 사용을 권장합니다 (더 안전).

```js
module.exports = {
  images: {
    domains: ['assets.acme.com'],
  },
}
```

---

## 함수

### `getImageProps()`

`<Image>` 생성 props를 다른 컴포넌트/요소에서 사용할 수 있습니다:

```jsx
import { getImageProps } from 'next/image'

const { props } = getImageProps({
  src: 'https://example.com/image.jpg',
  alt: '경치 좋은 산 전경',
  width: 1200,
  height: 800,
})

function ImageWithCaption() {
  return (
    <figure>
      <img {...props} />
      <figcaption>경치 좋은 산 전경</figcaption>
    </figure>
  )
}
```

> React `useState()` 호출이 없어 더 나은 성능을 제공합니다. 단, `placeholder` prop은 사용할 수 없습니다.

---

## 사용 예제

### 스타일링

**CSS Module:**

```jsx
import styles from './styles.module.css'

export default function MyImage() {
  return <Image className={styles.image} src="/my-image.png" alt="My Image" />
}
```

**인라인 스타일:**

```jsx
export default function MyImage() {
  return (
    <Image style={{ borderRadius: '8px' }} src="/my-image.png" alt="My Image" />
  )
}
```

**fill 사용 시 부모 포지셔닝:**

```jsx
<div style={{ position: 'relative' }}>
  <Image fill src="/my-image.png" alt="My Image" />
</div>
```

### 반응형 이미지 (정적 import)

```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'

export default function Responsive() {
  return (
    <div style={{ display: 'flex', flexDirection: 'column' }}>
      <Image
        alt="산"
        src={mountains}
        sizes="100vw"
        style={{
          width: '100%',
          height: 'auto',
        }}
      />
    </div>
  )
}
```

### 반응형 이미지 (원격 URL)

```jsx
import Image from 'next/image'

export default function Page({ photoUrl }) {
  return (
    <Image
      src={photoUrl}
      alt="저자 사진"
      sizes="100vw"
      style={{
        width: '100%',
        height: 'auto',
      }}
      width={500}
      height={300}
    />
  )
}
```

### fill로 반응형 이미지

```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'

export default function Fill() {
  return (
    <div
      style={{
        display: 'grid',
        gridGap: '8px',
        gridTemplateColumns: 'repeat(auto-fit, minmax(400px, auto))',
      }}
    >
      <div style={{ position: 'relative', width: '400px' }}>
        <Image
          alt="산"
          src={mountains}
          fill
          sizes="(min-width: 808px) 50vw, 100vw"
          style={{
            objectFit: 'cover',
          }}
        />
      </div>
    </div>
  )
}
```

### 배경 이미지

```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'

export default function Background() {
  return (
    <Image
      alt="산"
      src={mountains}
      placeholder="blur"
      quality={100}
      fill
      sizes="100vw"
      style={{
        objectFit: 'cover',
      }}
    />
  )
}
```

### 원격 이미지

```jsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="저자 사진"
      width={500}
      height={500}
    />
  )
}
```

**next.config.js 설정:**

```js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
        search: '',
      },
    ],
  },
}
```

### 라이트/다크 모드 테마 감지

```css
/* components/theme-image.module.css */
.imgDark {
  display: none;
}

@media (prefers-color-scheme: dark) {
  .imgLight {
    display: none;
  }
  .imgDark {
    display: unset;
  }
}
```

```jsx
/* components/theme-image.jsx */
import styles from './theme-image.module.css'
import Image from 'next/image'

const ThemeImage = (props) => {
  const { srcLight, srcDark, ...rest } = props

  return (
    <>
      <Image {...rest} src={srcLight} className={styles.imgLight} />
      <Image {...rest} src={srcDark} className={styles.imgDark} />
    </>
  )
}
```

> 기본 `loading="lazy"` 동작으로 올바른 이미지만 로드됩니다.

### Art Direction (모바일/데스크톱 다른 이미지)

```jsx
import { getImageProps } from 'next/image'

export default function Home() {
  const common = { alt: 'Art Direction 예제', sizes: '100vw' }
  const {
    props: { srcSet: desktop },
  } = getImageProps({
    ...common,
    width: 1440,
    height: 875,
    quality: 80,
    src: '/desktop.jpg',
  })
  const {
    props: { srcSet: mobile, ...rest },
  } = getImageProps({
    ...common,
    width: 750,
    height: 1334,
    quality: 70,
    src: '/mobile.jpg',
  })

  return (
    <picture>
      <source media="(min-width: 1000px)" srcSet={desktop} />
      <source media="(min-width: 500px)" srcSet={mobile} />
      <img {...rest} style={{ width: '100%', height: 'auto' }} />
    </picture>
  )
}
```

---

## 알려진 브라우저 버그

| 버그 | 해결책 |
|------|--------|
| Safari 15-16.3: 로드 중 회색 테두리 | CSS: `img[loading="lazy"] { clip-path: inset(0.6px) }` 또는 `loading="eager"` |
| Firefox 67+: 로드 중 흰 배경 | AVIF 활성화 또는 `placeholder` 사용 |

---

## 버전 이력

| 버전 | 변경사항 |
|------|---------|
| `v16.1.2` | `maximumResponseBody` 추가 |
| `v16.0.0` | `preload` prop 추가, `priority` deprecated, `qualities` 필수화 |
| `v15.3.0` | `remotePatterns` URL 객체 배열 지원 |
| `v14.2.23` | `qualities` 설정 추가 |
| `v14.2.15` | `decoding` prop, `localPatterns` 추가 |
| `v14.2.14` | `remotePatterns.search` 추가 |
| `v14.2.0` | `overrideSrc` prop 추가 |
| `v14.1.0` | `getImageProps()` 안정화 |
| `v14.0.0` | `onLoadingComplete` 및 `domains` deprecated |
| `v13.4.14` | `placeholder="data:image/..."` 지원 |
| `v13.2.0` | `contentDispositionType` 추가 |
| `v13.0.0` | 주요 재구조화 (`next/legacy/image` 및 `next/image` 분리) |
| `v12.3.0` | `remotePatterns`, `unoptimized` 안정화 |
| `v12.0.0` | AVIF 지원 추가 |
| `v10.0.0` | `next/image` 도입 |
