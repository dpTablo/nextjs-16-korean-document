# ImageResponse

## 개요
`ImageResponse`는 JSX와 CSS를 사용하여 동적 이미지를 생성하는 Next.js API로, 주로 소셜 미디어 콘텐츠(Open Graph 이미지, Twitter 카드 등)를 위해 사용됩니다.

---

## 생성자 구문

```jsx
import { ImageResponse } from 'next/og'

new ImageResponse(
  element: ReactElement,
  options: {
    width?: number = 1200
    height?: number = 630
    emoji?: 'twemoji' | 'blobmoji' | 'noto' | 'openmoji' = 'twemoji',
    fonts?: Array<{
      name: string,
      data: ArrayBuffer,
      weight: number,
      style: 'normal' | 'italic'
    }>
    debug?: boolean = false
    status?: number = 200
    statusText?: string
    headers?: Record<string, string>
  },
)
```

---

## 지원 기능

### CSS 속성
- ✅ Flexbox 레이아웃
- ✅ Absolute positioning
- ✅ 커스텀 폰트
- ✅ 텍스트 wrapping 및 centering
- ✅ 중첩 이미지

### 제한사항
- ❌ Grid 레이아웃 (`display: grid`) 미지원
- ❌ Flexbox 및 일부 CSS 속성만 작동
- ⚠️ 최대 번들 크기: **500KB** (JSX, CSS, 폰트, 이미지, 에셋 포함)
- ⚠️ 지원 폰트 형식: `ttf`, `otf`, `woff` (ttf/otf 권장)

---

## 사용 예제

### 1. Route Handler로 사용

```js
// app/api/og/route.js
import { ImageResponse } from 'next/og'

export async function GET() {
  try {
    return new ImageResponse(
      (
        <div style={{
          height: '100%',
          width: '100%',
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
          justifyContent: 'center',
          backgroundColor: 'white',
          padding: '40px',
        }}>
          <div style={{ fontSize: 60, fontWeight: 'bold', color: 'black' }}>
            내 사이트에 오신 것을 환영합니다
          </div>
          <div style={{ fontSize: 30, color: '#666', marginTop: '20px' }}>
            Next.js ImageResponse로 생성됨
          </div>
        </div>
      ),
      { width: 1200, height: 630 }
    )
  } catch (e) {
    return new Response(`이미지 생성 실패`, { status: 500 })
  }
}
```

### 2. 파일 기반 메타데이터 (`opengraph-image.tsx`)

```tsx
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const alt = '내 사이트'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

export default async function Image() {
  return new ImageResponse(
    (
      <div style={{
        fontSize: 128,
        background: 'white',
        width: '100%',
        height: '100%',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
      }}>
        내 사이트
      </div>
    ),
    { ...size }
  )
}
```

### 3. 커스텀 폰트 사용

```tsx
// app/api/og/route.tsx
import { ImageResponse } from 'next/og'
import { readFile } from 'node:fs/promises'
import { join } from 'node:path'

export async function GET() {
  const interSemiBold = await readFile(
    join(process.cwd(), 'assets/Inter-SemiBold.ttf')
  )

  return new ImageResponse(
    (
      <div style={{
        fontSize: 60,
        fontFamily: 'Inter',
        background: '#000',
        color: '#fff',
        width: '100%',
        height: '100%',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
      }}>
        커스텀 폰트 사용
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

---

## 실전 예제

### 블로그 포스트 OG 이미지

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const alt = '블로그 포스트'
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

export default async function Image({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)

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
        <div style={{
          fontSize: 60,
          fontWeight: 'bold',
          color: 'white',
          lineHeight: 1.2,
        }}>
          {post.title}
        </div>

        {/* 하단 정보 */}
        <div style={{
          display: 'flex',
          alignItems: 'center',
          fontSize: 28,
          color: '#888',
        }}>
          <div>{post.author}</div>
          <div style={{ margin: '0 20px' }}>•</div>
          <div>{post.date}</div>
        </div>
      </div>
    ),
    { ...size }
  )
}
```

### 동적 쿼리 매개변수

```tsx
// app/api/og/route.tsx
import { ImageResponse } from 'next/og'
import { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  const { searchParams } = request.nextUrl
  const title = searchParams.get('title') || '기본 제목'
  const theme = searchParams.get('theme') || 'light'

  const isDark = theme === 'dark'

  return new ImageResponse(
    (
      <div style={{
        height: '100%',
        width: '100%',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        backgroundColor: isDark ? '#000' : '#fff',
        color: isDark ? '#fff' : '#000',
        fontSize: 60,
        fontWeight: 'bold',
      }}>
        {title}
      </div>
    ),
    { width: 1200, height: 630 }
  )
}

// 사용: /api/og?title=안녕하세요&theme=dark
```

### 이미지 포함

```tsx
// app/api/og/route.tsx
import { ImageResponse } from 'next/og'

export async function GET() {
  // 로컬 이미지를 base64로 변환
  const imageData = await fetch(
    new URL('../../../public/logo.png', import.meta.url)
  ).then((res) => res.arrayBuffer())

  return new ImageResponse(
    (
      <div style={{
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        width: '100%',
        height: '100%',
        backgroundColor: 'white',
      }}>
        <img
          src={`data:image/png;base64,${Buffer.from(imageData).toString('base64')}`}
          alt="Logo"
          width={200}
          height={200}
        />
        <div style={{ fontSize: 60, marginLeft: 40 }}>
          내 회사
        </div>
      </div>
    ),
    { width: 1200, height: 630 }
  )
}
```

### 그라데이션 배경

```tsx
import { ImageResponse } from 'next/og'

export default async function Image() {
  return new ImageResponse(
    (
      <div style={{
        height: '100%',
        width: '100%',
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center',
        backgroundImage: 'linear-gradient(to bottom right, #667eea 0%, #764ba2 100%)',
      }}>
        <div style={{
          fontSize: 80,
          fontWeight: 'bold',
          color: 'white',
          textAlign: 'center',
        }}>
          그라데이션 배경
        </div>
      </div>
    ),
    { width: 1200, height: 630 }
  )
}
```

### 이모지 사용

```tsx
import { ImageResponse } from 'next/og'

export async function GET() {
  return new ImageResponse(
    (
      <div style={{
        fontSize: 100,
        background: 'white',
        width: '100%',
        height: '100%',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
      }}>
        안녕하세요 👋 환영합니다 🎉
      </div>
    ),
    {
      width: 1200,
      height: 630,
      emoji: 'twemoji', // 'blobmoji', 'noto', 'openmoji'
    }
  )
}
```

---

## 디버깅

```tsx
import { ImageResponse } from 'next/og'

export async function GET() {
  return new ImageResponse(
    ( /* JSX */ ),
    {
      width: 1200,
      height: 630,
      debug: true, // 개발 환경에서 경고 및 오류 표시
    }
  )
}
```

---

## 모범 사례

### 1. 표준 크기 사용

```tsx
// ✅ 좋은 예 - 표준 OG 이미지 크기
export const size = {
  width: 1200,
  height: 630,
}

// Open Graph 표준 비율 (1.91:1)
```

### 2. 폰트 최적화

```tsx
// ✅ 좋은 예 - 필요한 웨이트만 로드
const fontData = await readFile(
  join(process.cwd(), 'assets/Inter-Regular.ttf')
)

// ❌ 나쁜 예 - 여러 웨이트 로드 (번들 크기 증가)
```

### 3. 에러 처리

```tsx
export async function GET() {
  try {
    return new ImageResponse( /* ... */ )
  } catch (error) {
    console.error('OG 이미지 생성 실패:', error)
    return new Response('이미지 생성 실패', { status: 500 })
  }
}
```

### 4. 캐싱

```tsx
// app/api/og/route.tsx
export async function GET() {
  const response = new ImageResponse( /* ... */ )

  // 1시간 캐싱
  response.headers.set('Cache-Control', 'public, max-age=3600, immutable')

  return response
}
```

---

## 지원되는 CSS 속성

### Layout
- `display: flex`
- `flexDirection`
- `alignItems`, `justifyContent`
- `position: absolute`, `relative`
- `width`, `height`
- `padding`, `margin`

### Typography
- `fontSize`
- `fontWeight`
- `fontFamily`
- `color`
- `lineHeight`
- `textAlign`

### Visual
- `backgroundColor`
- `backgroundImage` (gradient만)
- `border`, `borderRadius`
- `opacity`

---

## 제한사항

### 번들 크기
- 최대 500KB (JSX + CSS + 폰트 + 이미지 포함)
- 큰 폰트나 이미지는 피하세요

### 미지원 기능
- CSS Grid
- CSS 애니메이션
- SVG (이미지로 변환 필요)
- 외부 스타일시트

---

## 기술 스택
- [@vercel/og](https://vercel.com/docs/concepts/functions/edge-functions/og-image-generation) 기반
- [Satori](https://github.com/vercel/satori)를 사용한 HTML/CSS 변환
- Resvg로 PNG 렌더링

---

## 버전 히스토리
- **v14.0.0**: `next/server`에서 `next/og`로 이동
- **v13.3.0**: `next/server`에서 import 가능
- **v13.0.0**: `@vercel/og` 패키지로 최초 도입

---

## 관련 문서

- [generateMetadata](./generateMetadata.md)
- [opengraph-image 파일 규칙](../file-conventions/metadata/opengraph-image.md)
- [Metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
