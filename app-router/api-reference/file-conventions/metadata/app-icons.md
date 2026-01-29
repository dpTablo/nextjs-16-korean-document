---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons
버전: 16.1.6
---

# App Icons (favicon, icon, apple-icon)

## 개요

Next.js는 `favicon`, `icon`, `apple-icon` 파일 규칙을 사용하여 앱 아이콘을 설정할 수 있습니다. 이미지 파일을 배치하거나 코드를 사용하여 동적으로 아이콘을 생성할 수 있습니다.

---

## 두 가지 구현 방법

### 1. 이미지 파일 사용 (.ico, .jpg, .png)

이미지 파일을 `/app` 디렉토리에 배치합니다:

| 규칙 | 파일 타입 | 위치 |
|------|----------|------|
| `favicon` | `.ico` | `app/` (루트만) |
| `icon` | `.ico`, `.jpg`, `.jpeg`, `.png`, `.svg` | `app/**/*` |
| `apple-icon` | `.jpg`, `.jpeg`, `.png` | `app/**/*` |

**예제:**
```
app/
├── favicon.ico          ← 파비콘
├── icon.png            ← 일반 아이콘
└── apple-icon.png      ← Apple 기기용 아이콘
```

Next.js는 파일 타입과 메타데이터를 기반으로 자동으로 적절한 `<head>` 태그를 생성합니다.

### 2. 코드로 아이콘 생성 (.js, .ts, .tsx)

`next/og`의 `ImageResponse` API를 사용하여 동적 아이콘을 생성합니다:

```tsx
// app/icon.tsx
import { ImageResponse } from 'next/og'

export const size = { width: 32, height: 32 }
export const contentType = 'image/png'

export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 24,
          background: 'black',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          color: 'white',
        }}
      >
        A
      </div>
    ),
    {
      ...size,
    }
  )
}
```

---

## 주요 특징

### 1. 여러 아이콘 지원

번호 접미사를 사용하여 여러 아이콘을 추가할 수 있습니다:

```
app/
├── icon1.png    ← 첫 번째 아이콘
├── icon2.png    ← 두 번째 아이콘
└── icon3.png    ← 세 번째 아이콘
```

### 2. 동적 파라미터

라우트 파라미터에 `params` prop을 통해 접근할 수 있습니다:

```tsx
// app/products/[id]/icon.tsx
export default async function Icon({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  // id를 사용하여 동적 아이콘 생성
}
```

### 3. 정적 최적화

Dynamic API를 사용하지 않는 한 아이콘은 기본적으로 캐시됩니다.

### 4. 메타데이터 설정

`size`와 `contentType`을 export하여 아이콘 속성을 커스터마이즈할 수 있습니다.

---

## 파일 규칙 상세

### favicon

**위치:** `app/favicon.ico` (루트만)

```
app/
└── favicon.ico
```

**생성되는 HTML:**
```html
<link rel="icon" href="/favicon.ico" sizes="any" />
```

**특징:**
- 브라우저 탭 아이콘
- 북마크 아이콘
- `.ico` 형식만 지원
- 루트 레벨에서만 작동

---

### icon

**위치:** `app/**/*` (모든 레벨)

**지원 형식:** `.ico`, `.jpg`, `.jpeg`, `.png`, `.svg`

#### 정적 이미지 파일

```
app/
├── icon.png
└── about/
    └── icon.png
```

**생성되는 HTML:**
```html
<link rel="icon" href="/icon.png" type="image/png" sizes="<computed>" />
```

#### 동적 아이콘 생성

```tsx
// app/icon.tsx
import { ImageResponse } from 'next/og'

export const size = {
  width: 32,
  height: 32,
}

export const contentType = 'image/png'

export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          background: '#1a1a1a',
          color: '#ffffff',
          fontSize: 24,
          fontWeight: 'bold',
        }}
      >
        My App
      </div>
    ),
    {
      ...size,
    }
  )
}
```

---

### apple-icon

**위치:** `app/**/*` (모든 레벨)

**지원 형식:** `.jpg`, `.jpeg`, `.png`

#### 정적 이미지 파일

```
app/
└── apple-icon.png
```

**생성되는 HTML:**
```html
<link rel="apple-touch-icon" href="/apple-icon.png" type="image/png" sizes="<computed>" />
```

#### 동적 Apple 아이콘

```tsx
// app/apple-icon.tsx
import { ImageResponse } from 'next/og'

export const size = {
  width: 180,
  height: 180,
}

export const contentType = 'image/png'

export default function AppleIcon() {
  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          background: 'linear-gradient(to bottom, #4F46E5, #7C3AED)',
          color: 'white',
          fontSize: 88,
        }}
      >
        📱
      </div>
    ),
    {
      ...size,
    }
  )
}
```

---

## 실용적인 예제

### 1. 완전한 아이콘 세트

```
app/
├── favicon.ico           # 브라우저 파비콘
├── icon.png             # 일반 아이콘 (32x32)
├── icon.svg             # SVG 아이콘
└── apple-icon.png       # Apple Touch Icon (180x180)
```

### 2. 여러 크기의 아이콘

```tsx
// app/icon.tsx
import { ImageResponse } from 'next/og'

// generateImageMetadata를 사용하여 여러 크기 생성
export function generateImageMetadata() {
  return [
    {
      id: 'small',
      size: { width: 16, height: 16 },
      contentType: 'image/png',
    },
    {
      id: 'medium',
      size: { width: 32, height: 32 },
      contentType: 'image/png',
    },
    {
      id: 'large',
      size: { width: 48, height: 48 },
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
  const sizeMap = {
    small: 16,
    medium: 32,
    large: 48,
  }

  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          background: '#000',
          color: '#fff',
          fontSize: sizeMap[iconId as keyof typeof sizeMap],
        }}
      >
        A
      </div>
    ),
    {
      width: sizeMap[iconId as keyof typeof sizeMap],
      height: sizeMap[iconId as keyof typeof sizeMap],
    }
  )
}
```

### 3. 라우트별 다른 아이콘

```
app/
├── icon.png              # 홈 아이콘
├── blog/
│   └── icon.png         # 블로그 아이콘
└── shop/
    └── icon.png         # 쇼핑 아이콘
```

### 4. 동적 라우트의 아이콘

```tsx
// app/team/[slug]/icon.tsx
import { ImageResponse } from 'next/og'

export const size = { width: 32, height: 32 }
export const contentType = 'image/png'

export default async function Icon({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const team = await getTeam(slug)

  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          background: team.color,
          color: 'white',
          fontSize: 20,
          fontWeight: 'bold',
        }}
      >
        {team.name[0]}
      </div>
    ),
    {
      ...size,
    }
  )
}
```

### 5. 테마 기반 아이콘

```tsx
// app/icon.tsx
import { ImageResponse } from 'next/og'

export const size = { width: 32, height: 32 }
export const contentType = 'image/png'

export default function Icon() {
  // 시스템 테마에 따라 다른 아이콘 생성
  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          background: 'transparent',
        }}
      >
        <svg
          width="32"
          height="32"
          viewBox="0 0 32 32"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <circle cx="16" cy="16" r="16" fill="currentColor" />
        </svg>
      </div>
    ),
    {
      ...size,
    }
  )
}
```

---

## 중요한 제약사항

### 1. favicon 동적 생성 불가

❌ **불가능:**
```tsx
// app/favicon.tsx - 작동하지 않음
export default function Favicon() {
  return new ImageResponse(/* ... */)
}
```

✅ **대안:**
```tsx
// app/icon.tsx - 이것을 사용하세요
export default function Icon() {
  return new ImageResponse(/* ... */)
}
```

또는 정적 `favicon.ico` 파일 사용:
```
app/
└── favicon.ico
```

### 2. 파일 위치 제한

- **`favicon.ico`**: 루트 `app/` 디렉토리에만 배치 가능
- **`icon.*`**: 모든 라우트 세그먼트에 배치 가능
- **`apple-icon.*`**: 모든 라우트 세그먼트에 배치 가능

### 3. 캐싱 동작

```tsx
// 정적 - 캐시됨
export default function Icon() {
  return new ImageResponse(/* ... */)
}

// 동적 - 캐시 안됨
export default async function Icon() {
  const data = await fetch('https://api.example.com') // Dynamic API
  return new ImageResponse(/* ... */)
}
```

---

## Props

### Icon 함수 Props

```tsx
export default async function Icon({
  params
}: {
  params: Promise<{ [key: string]: string }>
}) {
  // ...
}
```

| Prop | 타입 | 설명 |
|------|------|------|
| `params` | `Promise<object>` | 동적 라우트 파라미터 |

---

## Exports

### 필수 Export

```tsx
export default function Icon() {
  // 아이콘을 반환하는 함수 (필수)
}
```

### 선택적 Exports

```tsx
// 아이콘 크기 지정
export const size = {
  width: 32,
  height: 32,
}

// Content-Type 지정
export const contentType = 'image/png'

// 런타임 지정
export const runtime = 'edge' // 또는 'nodejs'
```

---

## 권장 아이콘 크기

| 타입 | 권장 크기 | 용도 |
|------|----------|------|
| **favicon.ico** | 16x16, 32x32, 48x48 | 브라우저 파비콘 |
| **icon.png** | 32x32 | 일반 아이콘 |
| **apple-icon.png** | 180x180 | Apple Touch Icon |
| **icon.svg** | 벡터 | 확장 가능한 아이콘 |

---

## 베스트 프랙티스

### ✅ 권장사항

1. **모든 형식 제공**
   ```
   app/
   ├── favicon.ico      # IE, 레거시 브라우저
   ├── icon.png        # 현대 브라우저
   ├── icon.svg        # 고해상도 디스플레이
   └── apple-icon.png  # iOS 기기
   ```

2. **적절한 크기 사용**
   - favicon: 16x16, 32x32
   - icon: 32x32
   - apple-icon: 180x180

3. **SVG 우선 사용** (가능한 경우)
   ```
   app/
   ├── icon.svg        # 우선 사용
   └── icon.png        # 폴백
   ```

4. **세그먼트별 아이콘 커스터마이징**
   ```
   app/
   ├── icon.png             # 기본 아이콘
   ├── dashboard/
   │   └── icon.png        # 대시보드 아이콘
   └── settings/
       └── icon.png        # 설정 아이콘
   ```

### ❌ 피해야 할 사항

1. **너무 큰 파일 크기**
   - 아이콘은 가능한 한 작게 유지 (< 100KB)
   - 최적화 도구 사용 권장

2. **favicon 동적 생성 시도**
   ```tsx
   // ❌ 작동하지 않음
   // app/favicon.tsx
   export default function Favicon() { ... }
   ```

3. **잘못된 위치**
   ```
   # ❌ 작동하지 않음
   app/
   └── components/
       └── favicon.ico    # 잘못된 위치
   ```

---

## 생성되는 HTML 예제

### 이미지 파일 사용 시

```
app/
├── favicon.ico
├── icon.png
└── apple-icon.png
```

**생성되는 HTML:**
```html
<link rel="icon" href="/favicon.ico" sizes="any" />
<link rel="icon" href="/icon.png" type="image/png" sizes="32x32" />
<link rel="apple-touch-icon" href="/apple-icon.png" type="image/png" sizes="180x180" />
```

### 코드로 생성 시

```tsx
// app/icon.tsx
export const size = { width: 32, height: 32 }
export const contentType = 'image/png'
```

**생성되는 HTML:**
```html
<link rel="icon" href="/icon?<hash>" type="image/png" sizes="32x32" />
```

---

## 타입 정의

```tsx
// Icon 함수
type IconFunction = (props: {
  params?: Promise<{ [key: string]: string }>
}) => ImageResponse | File | Response

// Export 타입
export const size: {
  width: number
  height: number
}

export const contentType: string

export const runtime: 'edge' | 'nodejs'
```

---

## 관련 API

- [`ImageResponse`](../../functions/ImageResponse.md) - 동적 이미지 생성
- [`generateImageMetadata`](../../functions/generateImageMetadata.md) - 여러 이미지 변형 생성
- [`generateMetadata`](../../functions/generateMetadata.md) - 동적 메타데이터 생성
- [Metadata Files](./opengraph-image.md) - Open Graph 이미지

---

## 버전 정보

- **도입 버전:** Next.js 13.3.0
- **안정 버전:** Next.js 13.4.0+
- **현재 상태:** Stable

---

## 요약

- **세 가지 타입:** `favicon`, `icon`, `apple-icon`
- **두 가지 방법:** 이미지 파일 또는 코드로 생성
- **위치:** favicon은 루트만, icon/apple-icon은 모든 레벨
- **형식:** .ico, .jpg, .jpeg, .png, .svg (타입별 상이)
- **동적 생성:** `ImageResponse` API 사용
- **제약:** favicon은 동적 생성 불가
- **권장 크기:** favicon(16-48px), icon(32px), apple-icon(180px)
