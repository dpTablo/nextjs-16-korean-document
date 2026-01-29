---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/metadata/manifest
버전: 16.1.6
---

# manifest.json / manifest.webmanifest

## 개요

`manifest.json` 또는 `manifest.webmanifest` 파일은 브라우저에 웹 애플리케이션에 대한 정보를 제공하며, [Web Manifest 사양](https://developer.mozilla.org/docs/Web/Manifest)을 따릅니다. **`app` 디렉토리의 루트**에 배치해야 합니다.

---

## 두 가지 구현 방법

### 1. 정적 Manifest 파일

정적 JSON 파일을 생성합니다:

**파일 위치:**
```
app/
└── manifest.json
```

**내용:**
```json
{
  "name": "My Next.js Application",
  "short_name": "Next.js App",
  "description": "An application built with Next.js",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### 2. 동적 Manifest (권장)

`manifest.ts` 또는 `manifest.js`를 생성하여 Manifest 객체를 반환합니다:

**TypeScript 예제:**
```typescript
// app/manifest.ts
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'Next.js App',
    short_name: 'Next.js App',
    description: 'Next.js App',
    start_url: '/',
    display: 'standalone',
    background_color: '#fff',
    theme_color: '#fff',
    icons: [
      {
        src: '/favicon.ico',
        sizes: 'any',
        type: 'image/x-icon',
      },
    ],
  }
}
```

**JavaScript 예제:**
```javascript
// app/manifest.js
export default function manifest() {
  return {
    name: 'My Next.js Application',
    short_name: 'Next.js App',
    description: 'An application built with Next.js',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    icons: [
      {
        src: '/icon-192.png',
        sizes: '192x192',
        type: 'image/png',
      },
      {
        src: '/icon-512.png',
        sizes: '512x512',
        type: 'image/png',
      },
    ],
  }
}
```

---

## 주요 속성

### 필수 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| `name` | `string` | 앱의 전체 이름 |
| `short_name` | `string` | 앱의 짧은 이름 (홈 화면 표시용) |
| `start_url` | `string` | 앱 시작 URL |

### 선택적 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| `description` | `string` | 앱 설명 |
| `display` | `string` | 표시 모드 (fullscreen, standalone, minimal-ui, browser) |
| `background_color` | `string` | 배경 색상 (스플래시 화면) |
| `theme_color` | `string` | 테마 색상 (브라우저 UI) |
| `icons` | `array` | 앱 아이콘 배열 |
| `orientation` | `string` | 기본 화면 방향 |
| `scope` | `string` | 앱의 범위 |
| `categories` | `array` | 앱 카테고리 |
| `screenshots` | `array` | 앱 스크린샷 |

---

## 실용적인 예제

### 1. 완전한 PWA Manifest

```typescript
// app/manifest.ts
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'My Awesome PWA',
    short_name: 'PWA',
    description: 'A Progressive Web App built with Next.js',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    orientation: 'portrait',
    icons: [
      {
        src: '/icon-192.png',
        sizes: '192x192',
        type: 'image/png',
        purpose: 'any',
      },
      {
        src: '/icon-512.png',
        sizes: '512x512',
        type: 'image/png',
        purpose: 'any',
      },
      {
        src: '/icon-maskable-192.png',
        sizes: '192x192',
        type: 'image/png',
        purpose: 'maskable',
      },
      {
        src: '/icon-maskable-512.png',
        sizes: '512x512',
        type: 'image/png',
        purpose: 'maskable',
      },
    ],
  }
}
```

### 2. 다크 모드 지원

```typescript
// app/manifest.ts
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'My App',
    short_name: 'App',
    description: 'My awesome application',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    icons: [
      {
        src: '/icon-light.png',
        sizes: '192x192',
        type: 'image/png',
        // 라이트 모드 아이콘
      },
      {
        src: '/icon-dark.png',
        sizes: '192x192',
        type: 'image/png',
        // 다크 모드 아이콘 (브라우저가 지원하는 경우)
      },
    ],
  }
}
```

### 3. 스크린샷 포함

```typescript
// app/manifest.ts
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'My App',
    short_name: 'App',
    description: 'My awesome application',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    icons: [
      {
        src: '/icon-512.png',
        sizes: '512x512',
        type: 'image/png',
      },
    ],
    screenshots: [
      {
        src: '/screenshot1.png',
        sizes: '540x720',
        type: 'image/png',
        form_factor: 'narrow',
      },
      {
        src: '/screenshot2.png',
        sizes: '1280x720',
        type: 'image/png',
        form_factor: 'wide',
      },
    ],
  }
}
```

### 4. 동적 환경 기반 Manifest

```typescript
// app/manifest.ts
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'http://localhost:3000'

  return {
    name: 'My Dynamic App',
    short_name: 'App',
    description: 'Dynamically configured PWA',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: process.env.NEXT_PUBLIC_THEME_COLOR || '#000000',
    icons: [
      {
        src: `${baseUrl}/icon-192.png`,
        sizes: '192x192',
        type: 'image/png',
      },
      {
        src: `${baseUrl}/icon-512.png`,
        sizes: '512x512',
        type: 'image/png',
      },
    ],
  }
}
```

### 5. 카테고리 및 언어 설정

```typescript
// app/manifest.ts
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'My Business App',
    short_name: 'Business',
    description: 'A professional business application',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#1e40af',
    lang: 'ko-KR',
    dir: 'ltr',
    categories: ['business', 'productivity', 'finance'],
    icons: [
      {
        src: '/icon-512.png',
        sizes: '512x512',
        type: 'image/png',
      },
    ],
  }
}
```

---

## Display 모드

| 모드 | 설명 | 사용 사례 |
|------|------|----------|
| `fullscreen` | 전체 화면, 브라우저 UI 없음 | 게임, 미디어 플레이어 |
| `standalone` | 독립 앱처럼, 브라우저 UI 최소화 | 대부분의 PWA (권장) |
| `minimal-ui` | 최소한의 브라우저 UI | 웹 앱 |
| `browser` | 일반 브라우저 탭 | 기본 웹사이트 |

---

## 아이콘 크기 권장사항

| 크기 | 용도 | Purpose |
|------|------|---------|
| **192x192** | 홈 화면, 스플래시 | `any`, `maskable` |
| **512x512** | 스플래시 화면 | `any`, `maskable` |
| **144x144** | Windows 타일 | `any` |
| **96x96** | 작은 아이콘 | `any` |

**Maskable 아이콘:**
- Adaptive Icons를 위해 투명한 여백 포함
- Safe zone: 중앙 80% 영역에 중요 콘텐츠 배치

---

## 주요 특징

### 1. 캐싱

- 동적 Manifest는 기본적으로 캐시됩니다
- Dynamic API를 사용하지 않는 한 정적으로 생성됩니다

```typescript
// 정적 - 캐시됨
export default function manifest() {
  return { /* ... */ }
}

// 동적 - 캐시 안됨
export default async function manifest() {
  const data = await fetch('https://api.example.com/config')
  return { /* ... */ }
}
```

### 2. 타입 안정성

TypeScript에서 `MetadataRoute.Manifest` 타입 사용:

```typescript
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  // 타입 체크와 자동완성 지원
  return {
    name: 'My App',
    // ...
  }
}
```

### 3. 유연성

환경 변수, 데이터베이스 쿼리 등을 사용하여 동적으로 생성:

```typescript
export default async function manifest(): Promise<MetadataRoute.Manifest> {
  const config = await getAppConfig()

  return {
    name: config.appName,
    theme_color: config.themeColor,
    // ...
  }
}
```

---

## 생성되는 HTML

manifest를 추가하면 다음 태그가 생성됩니다:

```html
<link rel="manifest" href="/manifest.webmanifest" />
```

---

## PWA 체크리스트

완전한 PWA를 만들기 위해 필요한 항목:

- [x] `manifest.json` / `manifest.webmanifest`
- [x] 아이콘 (192x192, 512x512)
- [ ] Service Worker (별도 구현 필요)
- [x] HTTPS (프로덕션 환경)
- [x] `start_url` 설정
- [x] `display: standalone` 설정

---

## 베스트 프랙티스

### ✅ 권장사항

1. **동적 Manifest 사용**
   ```typescript
   // ✅ 권장: manifest.ts
   export default function manifest() {
     return { /* ... */ }
   }
   ```

2. **필수 아이콘 크기 제공**
   ```typescript
   icons: [
     { src: '/icon-192.png', sizes: '192x192', type: 'image/png' },
     { src: '/icon-512.png', sizes: '512x512', type: 'image/png' },
   ]
   ```

3. **Maskable 아이콘 추가**
   ```typescript
   icons: [
     {
       src: '/icon-maskable.png',
       sizes: '512x512',
       type: 'image/png',
       purpose: 'maskable', // Adaptive Icons 지원
     },
   ]
   ```

4. **명확한 이름과 설명**
   ```typescript
   {
     name: 'My Application - Full Name',
     short_name: 'MyApp', // 홈 화면용 짧은 이름
     description: '명확하고 간결한 앱 설명',
   }
   ```

### ❌ 피해야 할 사항

1. **너무 긴 이름**
   ```typescript
   // ❌ 너무 긴 이름 (홈 화면에서 잘림)
   short_name: 'My Super Amazing Application'

   // ✅ 간결한 이름
   short_name: 'MyApp'
   ```

2. **잘못된 아이콘 크기**
   ```typescript
   // ❌ 최소 크기 미달
   icons: [{ src: '/icon.png', sizes: '48x48' }]

   // ✅ 권장 크기
   icons: [
     { src: '/icon-192.png', sizes: '192x192' },
     { src: '/icon-512.png', sizes: '512x512' },
   ]
   ```

3. **manifest 위치 오류**
   ```
   # ❌ 잘못된 위치
   app/
   └── components/
       └── manifest.json

   # ✅ 올바른 위치
   app/
   └── manifest.json
   ```

---

## 테스트 방법

### Chrome DevTools

1. Chrome DevTools 열기 (F12)
2. **Application** 탭 선택
3. **Manifest** 섹션 확인
4. 모든 필드가 올바르게 표시되는지 확인

### Lighthouse

1. Chrome DevTools > **Lighthouse** 탭
2. **Progressive Web App** 카테고리 선택
3. **Generate report** 실행
4. Manifest 관련 체크 항목 확인

---

## 타입 정의

```typescript
type Manifest = {
  name: string
  short_name?: string
  description?: string
  start_url: string
  display?: 'fullscreen' | 'standalone' | 'minimal-ui' | 'browser'
  background_color?: string
  theme_color?: string
  orientation?:
    | 'any'
    | 'natural'
    | 'landscape'
    | 'portrait'
    | 'portrait-primary'
    | 'portrait-secondary'
    | 'landscape-primary'
    | 'landscape-secondary'
  icons?: Array<{
    src: string
    sizes?: string
    type?: string
    purpose?: 'any' | 'maskable' | 'monochrome'
  }>
  screenshots?: Array<{
    src: string
    sizes: string
    type?: string
    form_factor?: 'narrow' | 'wide'
  }>
  categories?: string[]
  scope?: string
  lang?: string
  dir?: 'ltr' | 'rtl' | 'auto'
}
```

---

## 관련 리소스

- [Web Manifest MDN 문서](https://developer.mozilla.org/docs/Web/Manifest)
- [PWA Builder](https://www.pwabuilder.com/)
- [Maskable.app](https://maskable.app/) - Maskable 아이콘 생성 도구
- [App Icon Generator](https://icon.kitchen/) - 앱 아이콘 생성

---

## 관련 API

- [App Icons (favicon, icon, apple-icon)](./app-icons.md) - 앱 아이콘 파일 규칙
- [`generateMetadata`](../../functions/generateMetadata.md) - 동적 메타데이터 생성
- [Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata) - 메타데이터 최적화

---

## 버전 정보

- **도입 버전:** Next.js 13.3.0
- **안정 버전:** Next.js 13.4.0+
- **현재 상태:** Stable

---

## 요약

- **위치:** `app/manifest.json` 또는 `app/manifest.ts`
- **두 가지 방법:** 정적 JSON 파일 또는 동적 함수
- **권장 방법:** TypeScript로 동적 생성 (`manifest.ts`)
- **필수 속성:** `name`, `short_name`, `start_url`
- **PWA 필수:** 192x192, 512x512 아이콘 포함
- **타입:** `MetadataRoute.Manifest` 사용 (TypeScript)
- **용도:** Progressive Web App 지원, 홈 화면 추가
