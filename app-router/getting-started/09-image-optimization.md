---
원문: https://nextjs.org/docs/app/getting-started/image-optimization
버전: 16.1.6
---

# 이미지 최적화

## 개요

Next.js `<Image>` 컴포넌트는 HTML `<img>` 요소를 다음 기능으로 확장합니다:

- **크기 최적화:** WebP와 같은 최신 형식을 사용하여 각 디바이스에 맞는 크기의 이미지를 자동으로 제공
- **시각적 안정성:** 이미지 로딩 중 레이아웃 시프트를 자동으로 방지
- **더 빠른 페이지 로드:** 네이티브 브라우저 지연 로딩을 사용하여 뷰포트에 들어올 때만 이미지를 로드하며 선택적 블러 업 플레이스홀더 제공
- **에셋 유연성:** 원격 이미지를 포함하여 온디맨드로 이미지 크기 조정

## 기본 구현

### TypeScript 예시
```tsx
import Image from 'next/image'

export default function Page() {
  return <Image src="" alt="" />
}
```

### JavaScript 예시
```jsx
import Image from 'next/image'

export default function Page() {
  return <Image src="" alt="" />
}
```

`src` 속성은 로컬 또는 원격일 수 있습니다.

---

## 로컬 이미지

정적 파일(이미지, 폰트)은 루트 디렉토리의 `public` 폴더에 저장되며 기본 URL(`/`)에서 참조됩니다.

### 기본 로컬 이미지
```tsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="/profile.png"
      alt="작성자 사진"
      width={500}
      height={500}
    />
  )
}
```

### 정적 Import (자동 최적화)
```tsx
import Image from 'next/image'
import ProfileImage from './profile.png'

export default function Page() {
  return (
    <Image
      src={ProfileImage}
      alt="작성자 사진"
      // width와 height가 자동으로 제공됨
      // blurDataURL이 자동으로 제공됨
      // placeholder="blur" // 선택사항: 로딩 중 블러 업
    />
  )
}
```

정적으로 import하면 Next.js가 자동으로 `width`와 `height`를 결정하여 Cumulative Layout Shift를 방지합니다.

---

## 원격 이미지

```tsx
import Image from 'next/image'

export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="작성자 사진"
      width={500}
      height={500}
    />
  )
}
```

### 원격 이미지 요구사항
- `width`, `height`, 그리고 선택적으로 `blurDataURL` props를 수동으로 제공해야 함
- Next.js는 빌드 중 원격 파일에 접근할 수 없음
- 또는 부모 상대적 크기 지정을 위해 `fill` 속성 사용
- 보안을 위해 `next.config.js`에서 허용된 URL 패턴을 정의해야 함

### 설정 예시
```ts
import type { NextConfig } from 'next'

const config: NextConfig = {
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

export default config
```

---

## 리소스

- **비디오 튜토리얼:** [YouTube - 9분](https://youtu.be/IU_qq_c_lKA)
- **전체 API 참조:** [Image 컴포넌트 문서](/docs/app/api-reference/components/image.md)
