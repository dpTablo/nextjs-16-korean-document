# 정적 자산

Next.js는 루트 디렉토리의 `public` 폴더에서 이미지 같은 정적 파일을 제공할 수 있습니다.

## 기본 사용법

`public` 폴더 내의 파일은 기본 URL(`/`)부터 시작하여 코드에서 참조할 수 있습니다.

예를 들어, `public/avatars/me.png` 파일은 `/avatars/me.png` 경로로 접근할 수 있습니다.

### 예제

```jsx
// components/avatar.js
import Image from 'next/image'

export function Avatar({ id, alt }) {
  return <Image src={`/avatars/${id}.png`} alt={alt} width="64" height="64" />
}

export function AvatarOfMe() {
  return <Avatar id="me" alt="내 프로필 사진" />
}
```

## 디렉토리 구조

일반적인 `public` 폴더 구조:

```
public/
├── images/
│   ├── logo.png
│   └── hero.jpg
├── icons/
│   ├── favicon.ico
│   └── apple-icon.png
├── fonts/
│   └── custom-font.woff2
├── robots.txt
├── sitemap.xml
└── manifest.json
```

## 캐싱 정책

Next.js는 `public` 폴더의 자산을 안전하게 캐시할 수 없습니다. 파일이 언제든지 변경될 수 있기 때문입니다.

기본 캐싱 헤더:

```
Cache-Control: public, max-age=0
```

### 커스텀 캐싱 헤더

`next.config.js`에서 특정 정적 파일에 대해 캐싱 헤더를 설정할 수 있습니다:

```js
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/images/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ]
  },
}
```

## 일반적인 사용 사례

### robots.txt

```txt
# public/robots.txt
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

### favicon.ico

`public` 폴더에 `favicon.ico`를 넣으면 자동으로 사용됩니다.

### Google Site Verification

```html
<!-- public/google1234567890.html -->
google-site-verification: google1234567890.html
```

### 다운로드 파일

```jsx
export default function DownloadPage() {
  return (
    <a href="/documents/guide.pdf" download>
      가이드 다운로드
    </a>
  )
}
```

## next/image와 함께 사용

`next/image` 컴포넌트에서 `public` 폴더의 이미지를 사용할 수 있습니다:

```jsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="/images/hero.jpg"
      alt="히어로 이미지"
      width={1200}
      height={600}
      priority
    />
  )
}
```

## 주의사항

### 파일명 충돌

`pages/` 디렉토리의 파일과 같은 이름의 정적 파일을 `public` 폴더에 두면 오류가 발생합니다.

```
❌ 충돌 예시:
public/about.html
pages/about.js

두 파일 모두 /about 경로에 응답하려고 해서 오류 발생
```

### 빌드 시간에만 처리

`public` 폴더의 자산은 빌드 시간에만 처리됩니다. 런타임에 추가된 파일은 사용할 수 없습니다.

### 대소문자 구분

일부 배포 환경(예: Linux 서버)에서는 파일 경로가 대소문자를 구분합니다.

```jsx
// ✅ 올바른 경로
<Image src="/images/Logo.png" />

// ❌ 파일명이 logo.png인 경우 Linux에서 404 발생
<Image src="/images/Logo.png" />
```

## 권장 사항

1. **최적화된 이미지는 next/image 사용**: 자동 최적화가 필요한 이미지는 `next/image`를 사용하세요.

2. **폰트는 next/font 사용**: 폰트 파일은 `next/font`를 통해 최적화하세요.

3. **아이콘은 SVG 컴포넌트로**: 아이콘은 SVG 파일을 React 컴포넌트로 변환하여 사용하는 것이 좋습니다.

4. **버전 관리**: 캐싱을 활용하려면 파일명에 해시를 포함하거나 적절한 Cache-Control 헤더를 설정하세요.

```jsx
// 해시를 포함한 파일명 예시
<script src="/scripts/app.abc123.js" />
```

5. **큰 파일은 CDN 사용**: 대용량 파일은 전용 CDN을 통해 제공하는 것이 좋습니다.
