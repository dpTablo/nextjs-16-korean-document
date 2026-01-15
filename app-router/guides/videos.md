# Videos

Next.js 애플리케이션에서 비디오를 사용하고 최적화하는 방법을 알아봅니다.

## 개요

이 문서는 성능을 유지하면서 Next.js 애플리케이션에 비디오를 임베드하고 최적화하는 방법을 다룹니다. 셀프 호스팅 비디오, 외부 플랫폼 비디오 및 모범 사례를 포함합니다.

---

## `<video>` 및 `<iframe>` 사용

### HTML `<video>` 태그

`<video>` 태그는 셀프 호스팅 또는 직접 제공되는 비디오 콘텐츠를 임베드하며, 재생 및 외관에 대한 완전한 제어를 제공합니다.

**기본 예시:**
```tsx
export function Video() {
  return (
    <video width="320" height="240" controls preload="none">
      <source src="/path/to/video.mp4" type="video/mp4" />
      <track
        src="/path/to/captions.vtt"
        kind="subtitles"
        srcLang="ko"
        label="한국어"
      />
      브라우저가 video 태그를 지원하지 않습니다.
    </video>
  )
}
```

### 일반적인 `<video>` 속성

| 속성 | 설명 | 예시 |
|------|------|------|
| `src` | 비디오 파일 소스 | `<video src="/path/to/video.mp4" />` |
| `width` | 비디오 플레이어 너비 | `<video width="320" />` |
| `height` | 비디오 플레이어 높이 | `<video height="240" />` |
| `controls` | 기본 재생 컨트롤 표시 | `<video controls />` |
| `autoPlay` | 자동 재생 시작 | `<video autoPlay />` |
| `loop` | 비디오 반복 재생 | `<video loop />` |
| `muted` | 기본적으로 음소거 | `<video muted />` |
| `preload` | 미리 로드 동작 (`none`, `metadata`, `auto`) | `<video preload="none" />` |
| `playsInline` | iOS에서 인라인 재생 활성화 | `<video playsInline />` |
| `poster` | 재생 전 표시할 이미지 | `<video poster="/thumbnail.jpg" />` |

### 모범 사례

1. **autoPlay 사용 시 주의:**
   ```tsx
   // ✅ 브라우저 호환성을 위해 muted와 playsInline 포함
   <video autoPlay muted playsInline>
     <source src="/video.mp4" type="video/mp4" />
   </video>
   ```

2. **폴백 콘텐츠 포함:**
   ```tsx
   <video controls>
     <source src="/video.webm" type="video/webm" />
     <source src="/video.mp4" type="video/mp4" />
     브라우저가 video 태그를 지원하지 않습니다.
   </video>
   ```

3. **접근성을 위한 자막 추가:**
   ```tsx
   <video controls>
     <source src="/video.mp4" type="video/mp4" />
     <track
       src="/captions-ko.vtt"
       kind="subtitles"
       srcLang="ko"
       label="한국어"
       default
     />
     <track
       src="/captions-en.vtt"
       kind="subtitles"
       srcLang="en"
       label="English"
     />
   </video>
   ```

4. **접근 가능한 컨트롤 사용:**
   - 기본 컨트롤 또는 서드파티 플레이어 사용
   - [react-player](https://github.com/cookpete/react-player)
   - [video.js](https://videojs.com/)

---

### HTML `<iframe>` 태그

`<iframe>` 태그는 YouTube나 Vimeo 같은 외부 플랫폼의 비디오를 임베드합니다.

**기본 예시:**
```tsx
export default function Page() {
  return (
    <iframe
      src="https://www.youtube.com/embed/VIDEO_ID"
      width="560"
      height="315"
      allowFullScreen
      title="비디오 제목"
    />
  )
}
```

### 일반적인 `<iframe>` 속성

| 속성 | 설명 | 예시 |
|------|------|------|
| `src` | 임베드할 페이지 URL | `<iframe src="https://example.com" />` |
| `width` | iframe 너비 | `<iframe width="500" />` |
| `height` | iframe 높이 | `<iframe height="300" />` |
| `allowFullScreen` | 전체 화면 모드 허용 | `<iframe allowFullScreen />` |
| `sandbox` | 추가 제한 활성화 | `<iframe sandbox />` |
| `loading` | 로딩 최적화 (예: 지연 로딩) | `<iframe loading="lazy" />` |
| `title` | 접근성을 위한 제목 | `<iframe title="설명" />` |

---

### 임베딩 방식 선택

| 방식 | 사용 사례 | 장점 |
|------|----------|------|
| `<video>` | 셀프 호스팅/직접 파일 | 플레이어 기능과 외관에 대한 세부 제어 |
| `<iframe>` | 외부 호스팅 서비스 | 플랫폼 기능 포함, 설정 용이 |

---

## 외부 호스팅 비디오 임베드

### Server Component로 비디오 로드

**app/ui/video-component.tsx**
```tsx
export default async function VideoComponent() {
  const src = await getVideoSrc()

  return (
    <iframe
      src={src}
      width="560"
      height="315"
      allowFullScreen
      title="비디오 플레이어"
    />
  )
}
```

### React Suspense로 스트리밍

**app/page.tsx**
```tsx
import { Suspense } from 'react'
import VideoComponent from './ui/video-component'

export default function Page() {
  return (
    <section>
      <h1>비디오 섹션</h1>
      <Suspense fallback={<p>비디오 로딩 중...</p>}>
        <VideoComponent />
      </Suspense>
    </section>
  )
}
```

### 로딩 스켈레톤 사용

**app/ui/video-skeleton.tsx**
```tsx
export default function VideoSkeleton() {
  return (
    <div className="video-skeleton">
      <div className="skeleton-animation" />
      <div className="skeleton-controls">
        <div className="skeleton-button" />
        <div className="skeleton-progress" />
      </div>
    </div>
  )
}
```

**app/page.tsx**
```tsx
import { Suspense } from 'react'
import VideoComponent from './ui/video-component'
import VideoSkeleton from './ui/video-skeleton'

export default function Page() {
  return (
    <section>
      <Suspense fallback={<VideoSkeleton />}>
        <VideoComponent />
      </Suspense>
    </section>
  )
}
```

### YouTube 임베드 최적화

```tsx
'use client'

import { useState } from 'react'
import Image from 'next/image'

interface YouTubeEmbedProps {
  videoId: string
  title: string
}

export function YouTubeEmbed({ videoId, title }: YouTubeEmbedProps) {
  const [isLoaded, setIsLoaded] = useState(false)

  if (!isLoaded) {
    return (
      <button
        onClick={() => setIsLoaded(true)}
        className="youtube-placeholder"
        aria-label={`${title} 재생`}
      >
        <Image
          src={`https://img.youtube.com/vi/${videoId}/maxresdefault.jpg`}
          alt={title}
          fill
          sizes="(max-width: 768px) 100vw, 560px"
        />
        <span className="play-button">▶</span>
      </button>
    )
  }

  return (
    <iframe
      src={`https://www.youtube.com/embed/${videoId}?autoplay=1`}
      width="560"
      height="315"
      allowFullScreen
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
      title={title}
    />
  )
}
```

---

## 셀프 호스팅 비디오

셀프 호스팅은 다음과 같은 이점을 제공합니다:

- **완전한 제어** - 콘텐츠에 대한 완전한 제어와 독립성
- **맞춤화** - 배경 비디오 등 특정 요구에 맞는 사용자 정의
- **성능 최적화** - 성능과 확장성 최적화
- **비용 효율성** - 쉬운 통합으로 비용 균형

### Vercel Blob으로 비디오 호스팅

#### 1단계: 비디오 업로드

Vercel 대시보드의 Storage 탭에서 업로드하거나 서버/클라이언트 사이드 업로드를 사용합니다.

#### 2단계: Next.js에서 표시

**app/page.tsx**
```tsx
import { Suspense } from 'react'
import { list } from '@vercel/blob'

export default function Page() {
  return (
    <Suspense fallback={<p>비디오 로딩 중...</p>}>
      <VideoComponent fileName="my-video.mp4" />
    </Suspense>
  )
}

async function VideoComponent({ fileName }: { fileName: string }) {
  const { blobs } = await list({
    prefix: fileName,
    limit: 1,
  })
  const { url } = blobs[0]

  return (
    <video controls preload="none" aria-label="비디오 플레이어">
      <source src={url} type="video/mp4" />
      브라우저가 video 태그를 지원하지 않습니다.
    </video>
  )
}
```

### 자막 추가

```tsx
async function VideoComponent({ fileName }: { fileName: string }) {
  const { blobs } = await list({
    prefix: fileName,
    limit: 2,
  })
  const { url } = blobs[0]
  const { url: captionsUrl } = blobs[1]

  return (
    <video controls preload="none" aria-label="비디오 플레이어">
      <source src={url} type="video/mp4" />
      <track
        src={captionsUrl}
        kind="subtitles"
        srcLang="ko"
        label="한국어"
        default
      />
      브라우저가 video 태그를 지원하지 않습니다.
    </video>
  )
}
```

---

## 반응형 비디오 플레이어

### CSS를 사용한 반응형 처리

```tsx
export function ResponsiveVideo() {
  return (
    <div className="video-container">
      <video controls preload="metadata">
        <source src="/video.mp4" type="video/mp4" />
      </video>
      <style jsx>{`
        .video-container {
          position: relative;
          width: 100%;
          padding-bottom: 56.25%; /* 16:9 비율 */
        }
        .video-container video {
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          object-fit: cover;
        }
      `}</style>
    </div>
  )
}
```

### Tailwind CSS 사용

```tsx
export function ResponsiveVideo() {
  return (
    <div className="relative w-full aspect-video">
      <video
        className="absolute inset-0 w-full h-full object-cover"
        controls
        preload="metadata"
      >
        <source src="/video.mp4" type="video/mp4" />
      </video>
    </div>
  )
}
```

---

## 배경 비디오

배경으로 비디오를 사용하는 예시입니다.

```tsx
export function BackgroundVideo() {
  return (
    <div className="relative h-screen w-full overflow-hidden">
      <video
        autoPlay
        muted
        loop
        playsInline
        className="absolute inset-0 h-full w-full object-cover"
      >
        <source src="/background.mp4" type="video/mp4" />
      </video>
      <div className="relative z-10 flex h-full items-center justify-center">
        <h1 className="text-4xl font-bold text-white">환영합니다</h1>
      </div>
    </div>
  )
}
```

---

## 비디오 최적화

### 비디오 포맷 및 코덱

| 포맷 | 코덱 | 브라우저 지원 | 사용 사례 |
|------|------|--------------|----------|
| MP4 | H.264 | 모든 브라우저 | 범용 호환성 |
| WebM | VP9 | 최신 브라우저 | 더 나은 압축 |
| HLS | H.264/H.265 | Safari, 일부 | 적응형 스트리밍 |

### FFmpeg를 사용한 비디오 압축

```bash
# MP4 압축
ffmpeg -i input.mp4 -vcodec h264 -acodec aac -crf 23 output.mp4

# WebM 변환
ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 output.webm

# 해상도 조정
ffmpeg -i input.mp4 -vf scale=1280:720 -c:a copy output.mp4

# 썸네일 생성
ffmpeg -i input.mp4 -ss 00:00:01 -vframes 1 thumbnail.jpg
```

### 다중 소스 제공

```tsx
export function OptimizedVideo() {
  return (
    <video controls preload="metadata" poster="/thumbnail.jpg">
      {/* 최신 브라우저용 WebM */}
      <source src="/video.webm" type="video/webm" />
      {/* 폴백용 MP4 */}
      <source src="/video.mp4" type="video/mp4" />
      브라우저가 video 태그를 지원하지 않습니다.
    </video>
  )
}
```

---

## 비디오 플랫폼 및 라이브러리

### next-video (오픈 소스)

Vercel Blob, S3, Backblaze, Mux와 호환됩니다.

```bash
npm install next-video
```

```tsx
import Video from 'next-video'
import myVideo from '/videos/my-video.mp4'

export default function Page() {
  return <Video src={myVideo} />
}
```

[next-video 문서](https://next-video.dev/docs)

### Cloudinary

```bash
npm install next-cloudinary
```

```tsx
import { CldVideoPlayer } from 'next-cloudinary'

export default function Page() {
  return (
    <CldVideoPlayer
      width="1920"
      height="1080"
      src="videos/my-video"
    />
  )
}
```

적응형 비트레이트 스트리밍을 지원합니다.

### Mux

고성능 비디오 API를 제공합니다.

```tsx
import MuxPlayer from '@mux/mux-player-react'

export default function Page() {
  return (
    <MuxPlayer
      playbackId="YOUR_PLAYBACK_ID"
      metadata={{
        video_title: '비디오 제목',
      }}
    />
  )
}
```

### ImageKit.io

```tsx
import { IKVideo } from 'imagekitio-react'

export default function Page() {
  return (
    <IKVideo
      path="/videos/my-video.mp4"
      transformation={[{ height: 480, width: 854 }]}
      controls
    />
  )
}
```

---

## 성능 최적화 팁

### 1. 지연 로딩

```tsx
<video controls preload="none" loading="lazy">
  <source src="/video.mp4" type="video/mp4" />
</video>
```

### 2. CDN 사용

- 전 세계적으로 빠른 콘텐츠 제공
- 서버 부하 감소
- 캐싱을 통한 성능 향상

### 3. 적응형 비트레이트 스트리밍

HLS나 DASH를 사용하여 네트워크 조건에 맞는 품질을 제공합니다.

### 4. 썸네일 최적화

```tsx
<video
  controls
  preload="metadata"
  poster="/optimized-thumbnail.webp"
>
  <source src="/video.mp4" type="video/mp4" />
</video>
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **접근성 고려**
   - 자막 제공
   - 키보드 탐색 지원
   - 적절한 `aria-label` 사용

2. **성능 최적화**
   - `preload="none"` 또는 `preload="metadata"` 사용
   - 지연 로딩 구현
   - 압축된 비디오 제공

3. **반응형 디자인**
   - 다양한 화면 크기에 맞게 조정
   - `aspect-ratio` 유지

4. **폴백 제공**
   - 여러 포맷 제공
   - 지원하지 않는 브라우저를 위한 대체 콘텐츠

### ❌ 피해야 할 것

1. **자동 재생 남용**
   ```tsx
   // ❌ 음소거 없는 자동 재생
   <video autoPlay />

   // ✅ 음소거와 함께 자동 재생
   <video autoPlay muted playsInline />
   ```

2. **대용량 비디오 직접 로드**
   - 압축 및 최적화 없이 원본 파일 사용

3. **preload="auto" 과다 사용**
   - 불필요한 대역폭 소비

---

## 다음 단계

- [Image Optimization](../getting-started/image-optimization.md) - 이미지 최적화
- [Static Assets](../getting-started/installation.md#public-폴더) - 정적 자산 관리
- [Lazy Loading](./lazy-loading.md) - 지연 로딩

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-15

**참고 자료:**
- [MDN Video Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [Web Video Text Tracks (WebVTT)](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API)
