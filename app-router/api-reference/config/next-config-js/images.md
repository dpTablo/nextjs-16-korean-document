---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/images
버전: 16.1.6
---

# images

## 개요

`next.config.js`의 `images` 설정을 통해 이미지 최적화 동작을 커스터마이징할 수 있습니다. Next.js의 내장 이미지 최적화 API 대신 커스텀 클라우드 프로바이더를 사용하는 것도 가능합니다.

---

## 기본 설정

```js
// next.config.js
module.exports = {
  images: {
    // 이미지 설정 옵션
  },
}
```

---

## 설정 옵션

### remotePatterns ⭐ (권장)

특정 외부 경로의 이미지를 허용합니다. 더 나은 보안을 위해 deprecated된 `domains` 대신 사용하세요.

```js
// next.config.js
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

**속성:**
- `protocol` - `'http'` 또는 `'https'`
- `hostname` - 호스트명 (와일드카드 지원)
- `port` - 포트 번호 (선택사항)
- `pathname` - 경로 패턴 (와일드카드 지원)
- `search` - 쿼리 문자열 (선택사항)

**와일드카드 패턴:**

```js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.example.com', // 모든 서브도메인 허용
      },
    ],
  },
}
```

- `*` - 단일 경로 세그먼트 또는 서브도메인과 일치
- `**` - 끝에서 여러 경로 세그먼트 또는 시작에서 여러 서브도메인과 일치

**예제:**

```js
module.exports = {
  images: {
    remotePatterns: [
      // CDN 이미지
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        pathname: '/images/**',
      },
      // 사용자 업로드 이미지
      {
        protocol: 'https',
        hostname: 'user-content.example.com',
        pathname: '/uploads/**',
      },
      // 모든 example.com 서브도메인
      {
        protocol: 'https',
        hostname: '**.example.com',
      },
    ],
  },
}
```

---

### localPatterns

특정 로컬 경로의 이미지만 최적화를 허용하고 나머지는 차단합니다.

```js
// next.config.js
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

**사용 예제:**

```js
module.exports = {
  images: {
    localPatterns: [
      {
        pathname: '/public/images/**',
      },
      {
        pathname: '/static/photos/**',
      },
    ],
  },
}
```

---

### domains ⚠️ (Deprecated)

> **경고:** `domains`는 deprecated되었습니다. 더 나은 보안을 위해 `remotePatterns`를 사용하세요.

허용된 호스트명 목록을 설정하는 레거시 구성입니다.

```js
// next.config.js
module.exports = {
  images: {
    domains: ['assets.acme.com', 'cdn.example.com'],
  },
}
```

---

### deviceSizes

반응형 이미지를 위한 디바이스 너비 중단점 목록입니다. `sizes` prop과 함께 사용됩니다.

```js
// next.config.js
module.exports = {
  images: {
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },
}
```

**기본값:** `[640, 750, 828, 1080, 1200, 1920, 2048, 3840]`

**사용 시나리오:**
- 모바일 우선 사이트의 경우 작은 크기 추가
- 대형 디스플레이가 많은 경우 큰 크기 추가

```js
// 모바일 우선 예제
module.exports = {
  images: {
    deviceSizes: [320, 420, 768, 1024, 1200],
  },
}
```

---

### imageSizes

이미지 `srcset`을 생성하는 데 사용되는 이미지 너비 목록입니다. `deviceSizes`와 연결됩니다.

```js
// next.config.js
module.exports = {
  images: {
    imageSizes: [32, 48, 64, 96, 128, 256, 384],
  },
}
```

**기본값:** `[16, 32, 48, 64, 96, 128, 256, 384]`

> **중요:** `imageSizes`의 모든 값은 `deviceSizes`의 가장 작은 값보다 작아야 합니다.

**사용 예제:**

```js
// 아이콘 및 썸네일용
module.exports = {
  images: {
    imageSizes: [16, 32, 48, 64, 96, 128],
    deviceSizes: [640, 750, 828, 1080, 1200],
  },
}
```

---

### qualities

허용되는 이미지 품질 값 목록입니다 (Next.js 16부터 필수).

```js
// next.config.js
module.exports = {
  images: {
    qualities: [75], // 기본값
  },
}
```

**여러 품질 허용:**

```js
module.exports = {
  images: {
    qualities: [25, 50, 75, 100],
  },
}
```

**사용 예제:**

```jsx
// 컴포넌트에서 사용
<Image
  src="/photo.jpg"
  width={800}
  height={600}
  quality={100} // qualities 배열에 포함되어야 함
/>
```

---

### formats

사용할 이미지 형식 목록입니다.

```js
// next.config.js
module.exports = {
  images: {
    formats: ['image/webp'], // 기본값
  },
}
```

**AVIF 활성화:**

```js
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'], // AVIF 우선, WebP 폴백
  },
}
```

**브라우저 지원:**
- WebP: 대부분의 최신 브라우저
- AVIF: 더 나은 압축률, 제한적인 브라우저 지원

---

### minimumCacheTTL

캐시된 최적화 이미지의 TTL(Time to Live)을 초 단위로 설정합니다.

```js
// next.config.js
module.exports = {
  images: {
    minimumCacheTTL: 14400, // 4시간 (기본값)
  },
}
```

**재검증 감소를 위해 증가:**

```js
module.exports = {
  images: {
    minimumCacheTTL: 2678400, // 31일
  },
}
```

**사용 시나리오:**
- 정적 이미지: 높은 TTL (예: 31일)
- 자주 변경되는 이미지: 낮은 TTL (예: 1시간)

---

### loaderFile

Next.js 기본 대신 커스텀 이미지 최적화 서비스를 지정합니다.

```js
// next.config.js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './my/image/loader.js',
  },
}
```

**로더 함수:**

```js
// my/image/loader.js
'use client'

export default function myImageLoader({ src, width, quality }) {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}
```

**매개변수:**
- `src` - 이미지 소스 경로
- `width` - 요청된 이미지 너비
- `quality` - 이미지 품질 (기본값: 75)

**반환값:** 최적화된 이미지 URL 문자열

**클라우드 프로바이더 예제:**

```js
// Cloudinary 로더
'use client'

export default function cloudinaryLoader({ src, width, quality }) {
  const params = ['f_auto', 'c_limit', `w_${width}`, `q_${quality || 'auto'}`]
  return `https://res.cloudinary.com/demo/image/upload/${params.join(',')}${src}`
}
```

```js
// Imgix 로더
'use client'

export default function imgixLoader({ src, width, quality }) {
  const url = new URL(`https://example.imgix.net${src}`)
  url.searchParams.set('auto', 'format')
  url.searchParams.set('fit', 'max')
  url.searchParams.set('w', width.toString())
  if (quality) {
    url.searchParams.set('q', quality.toString())
  }
  return url.href
}
```

> **참고:** 커스텀 로더 파일은 `'use client'`로 표시된 [클라이언트 컴포넌트](../../getting-started/05-server-and-client-components.md)여야 합니다.

---

### path

기본 이미지 최적화 API 경로를 변경하거나 접두사를 붙입니다.

```js
// next.config.js
module.exports = {
  images: {
    path: '/my-prefix/_next/image', // 기본값: '/_next/image'
  },
}
```

**사용 예제:**

```js
// CDN과 함께 사용
module.exports = {
  images: {
    path: 'https://cdn.example.com/_next/image',
  },
}
```

---

### unoptimized

이미지 최적화를 전역적으로 비활성화합니다.

```js
// next.config.js
module.exports = {
  images: {
    unoptimized: true,
  },
}
```

**사용 시나리오:**
- 작은 이미지 (<1KB)
- SVG 이미지
- 애니메이션 이미지 (GIF)
- 정적 내보내기

```js
// 개발 환경에서만 비활성화
module.exports = {
  images: {
    unoptimized: process.env.NODE_ENV === 'development',
  },
}
```

---

### disableStaticImages

정적 이미지 import를 비활성화합니다 (`import icon from './icon.png'`).

```js
// next.config.js
module.exports = {
  images: {
    disableStaticImages: true,
  },
}
```

**사용 시나리오:**
- 다른 플러그인과의 충돌
- 커스텀 이미지 처리

---

### dangerouslyAllowSVG

SVG 이미지 제공을 활성화합니다.

```js
// next.config.js
module.exports = {
  images: {
    dangerouslyAllowSVG: true,
    contentDispositionType: 'attachment',
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

> **경고:** SVG는 보안 위험이 있을 수 있으므로 적절한 CSP 헤더와 함께 사용하세요.

---

### contentDispositionType

`Content-Disposition` 헤더를 구성합니다.

```js
// next.config.js
module.exports = {
  images: {
    contentDispositionType: 'attachment', // 기본값
    // 또는
    contentDispositionType: 'inline',
  },
}
```

**옵션:**
- `'attachment'` - 다운로드 강제 (더 안전)
- `'inline'` - 브라우저에서 표시

---

### contentSecurityPolicy

`Content-Security-Policy` 헤더를 구성합니다. SVG 보안에 중요합니다.

```js
// next.config.js
module.exports = {
  images: {
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

---

### maximumRedirects

원격 이미지를 가져올 때 따를 HTTP 리디렉션 수입니다.

```js
// next.config.js
module.exports = {
  images: {
    maximumRedirects: 3, // 기본값
  },
}
```

**리디렉션 비활성화:**

```js
module.exports = {
  images: {
    maximumRedirects: 0,
  },
}
```

---

### maximumResponseBody

소스 이미지의 최대 크기를 바이트 단위로 설정합니다.

```js
// next.config.js
module.exports = {
  images: {
    maximumResponseBody: 50_000_000, // 50 MB (기본값)
  },
}
```

**메모리 제약 서버 보호:**

```js
module.exports = {
  images: {
    maximumResponseBody: 5_000_000, // 5 MB
  },
}
```

---

### dangerouslyAllowLocalIP

동일 네트워크의 로컬 IP 주소에서 이미지 최적화를 허용합니다.

```js
// next.config.js
module.exports = {
  images: {
    dangerouslyAllowLocalIP: true, // 기본값: false
  },
}
```

> **경고:** 보안 위험으로 인해 대부분의 사용자에게 권장되지 않습니다.

---

## 완전한 설정 예제

### 프로덕션 설정

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        pathname: '/images/**',
      },
      {
        protocol: 'https',
        hostname: 'user-uploads.example.com',
        pathname: '/uploads/**',
      },
    ],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    qualities: [75, 90, 100],
    formats: ['image/avif', 'image/webp'],
    minimumCacheTTL: 86400, // 1일
    unoptimized: false,
  },
}
```

### SVG 지원 설정

```js
// next.config.js
module.exports = {
  images: {
    dangerouslyAllowSVG: true,
    contentDispositionType: 'attachment',
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

### 커스텀 로더 설정

```js
// next.config.js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './lib/cloudinary-loader.js',
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'res.cloudinary.com',
      },
    ],
  },
}
```

```js
// lib/cloudinary-loader.js
'use client'

export default function cloudinaryLoader({ src, width, quality }) {
  const params = ['f_auto', 'c_limit', `w_${width}`, `q_${quality || 'auto'}`]
  const cloudName = process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME
  return `https://res.cloudinary.com/${cloudName}/image/upload/${params.join(',')}${src}`
}
```

### 개발/프로덕션 분리 설정

```js
// next.config.js
module.exports = {
  images: {
    unoptimized: process.env.NODE_ENV === 'development',
    minimumCacheTTL: process.env.NODE_ENV === 'development' ? 0 : 86400,
    remotePatterns: [
      {
        protocol: 'https',
        hostname: process.env.NODE_ENV === 'development'
          ? 'dev-cdn.example.com'
          : 'cdn.example.com',
      },
    ],
  },
}
```

---

## 베스트 프랙티스

### 1. remotePatterns 사용

```js
// ✅ 권장 - 구체적인 패턴
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        pathname: '/images/**',
      },
    ],
  },
}

// ❌ 피하기 - 너무 광범위
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**', // 모든 호스트 허용
      },
    ],
  },
}
```

### 2. 적절한 이미지 크기 설정

```js
// ✅ 권장 - 실제 사용 패턴에 맞춤
module.exports = {
  images: {
    deviceSizes: [640, 750, 1080, 1200, 1920],
    imageSizes: [32, 64, 96, 128, 256],
  },
}
```

### 3. AVIF 형식 고려

```js
// ✅ 권장 - 최신 형식 우선
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
  },
}
```

### 4. 적절한 캐시 TTL

```js
// ✅ 권장 - 이미지 변경 빈도에 따라 조정
module.exports = {
  images: {
    minimumCacheTTL: 86400, // 정적 이미지: 1일
    // minimumCacheTTL: 3600, // 동적 이미지: 1시간
  },
}
```

### 5. SVG 보안

```js
// ✅ 권장 - 적절한 보안 헤더와 함께
module.exports = {
  images: {
    dangerouslyAllowSVG: true,
    contentDispositionType: 'attachment',
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
}
```

---

## 자주 하는 실수

### 1. imageSizes > deviceSizes

```js
// ❌ 잘못된 설정
module.exports = {
  images: {
    imageSizes: [1024, 2048], // 너무 큼
    deviceSizes: [640, 750, 828],
  },
}

// ✅ 올바른 설정
module.exports = {
  images: {
    imageSizes: [16, 32, 48, 64, 96], // deviceSizes보다 작음
    deviceSizes: [640, 750, 828, 1080],
  },
}
```

### 2. domains와 remotePatterns 혼용

```js
// ❌ 잘못된 설정 - domains는 deprecated
module.exports = {
  images: {
    domains: ['example.com'],
    remotePatterns: [/* ... */],
  },
}

// ✅ 올바른 설정 - remotePatterns만 사용
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
      },
    ],
  },
}
```

### 3. qualities 누락 (Next.js 16+)

```js
// ❌ Next.js 16에서 에러
module.exports = {
  images: {
    // qualities 없음
  },
}

// ✅ 올바른 설정
module.exports = {
  images: {
    qualities: [75, 90, 100],
  },
}
```

---

## 타입 정의

```typescript
type ImagesConfig = {
  remotePatterns?: RemotePattern[]
  localPatterns?: LocalPattern[]
  domains?: string[]
  deviceSizes?: number[]
  imageSizes?: number[]
  qualities?: number[]
  formats?: ImageFormat[]
  minimumCacheTTL?: number
  loader?: 'default' | 'imgix' | 'cloudinary' | 'akamai' | 'custom'
  loaderFile?: string
  path?: string
  unoptimized?: boolean
  disableStaticImages?: boolean
  dangerouslyAllowSVG?: boolean
  contentDispositionType?: 'inline' | 'attachment'
  contentSecurityPolicy?: string
  maximumRedirects?: number
  maximumResponseBody?: number
  dangerouslyAllowLocalIP?: boolean
}

type RemotePattern = {
  protocol?: 'http' | 'https'
  hostname: string
  port?: string
  pathname?: string
  search?: string
}

type LocalPattern = {
  pathname: string
  search?: string
}

type ImageFormat = 'image/webp' | 'image/avif'
```

---

## 관련 문서

- [Image 컴포넌트](../../components/image.md)
- [이미지 최적화 가이드](../../../guides/09-image-optimization.md)
- [성능 최적화](../../../guides/lazy-loading.md)

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| `v16.0.0` | `qualities` 필수 옵션으로 변경 |
| `v15.0.0` | `remotePatterns` 추가, `domains` deprecated |
| `v14.0.0` | `formats`에 AVIF 지원 추가 |
| `v13.0.0` | 이미지 최적화 안정화 |

---

## 요약

- **remotePatterns**: 외부 이미지 소스 제어 (권장)
- **deviceSizes/imageSizes**: 반응형 이미지 크기 정의
- **qualities**: 허용되는 품질 값 (Next.js 16+)
- **formats**: WebP/AVIF 지원 설정
- **minimumCacheTTL**: 캐시 지속 시간
- **loaderFile**: 커스텀 이미지 로더
- **unoptimized**: 최적화 비활성화
- **dangerouslyAllowSVG**: SVG 지원 (보안 주의)
