# 정적 내보내기

**버전:** 16.1.1

## 개요
Next.js는 빌드 프로세스 중 라우트당 HTML 파일을 생성하여 정적 내보내기를 생성할 수 있습니다. 이 접근 방식은 번들 크기를 줄이고, 페이지 로드 시간을 개선하며, 정적 에셋을 제공하는 모든 웹 서버에 배포할 수 있습니다.

## 설정

`next.config.js`에서 정적 내보내기 활성화:

```js
const nextConfig = {
  output: 'export',

  // 선택사항: 후행 슬래시 추가
  // trailingSlash: true,

  // 선택사항: 출력 디렉토리 변경
  // distDir: 'dist',
}

module.exports = nextConfig
```

`next build`를 실행하면 모든 HTML/CSS/JS 에셋을 포함하는 `out` 폴더가 생성됩니다.

## 지원되는 기능

### 서버 컴포넌트
- 빌드 중 실행됨 (전통적인 정적 사이트 생성과 유사)
- 초기 페이지 로드를 위한 정적 HTML로 렌더링
- 클라이언트 사이드 네비게이션을 위한 정적 페이로드 생성
- 동적 서버 함수를 사용하지 않는 한 변경 불필요

### 클라이언트 컴포넌트
- 클라이언트 사이드 데이터 페칭을 위해 SWR 또는 유사 라이브러리 사용
- SPA와 같은 라우트 전환 활성화
- SWR 사용 예시:
```tsx
'use client'
import useSWR from 'swr'

export default function Page() {
  const { data, error } = useSWR(url, fetcher)
  // ...
}
```

### 이미지 최적화
- 커스텀 이미지 로더 사용 (예: Cloudinary)
- `next.config.js`에서 정의:
```js
const nextConfig = {
  output: 'export',
  images: {
    loader: 'custom',
    loaderFile: './my-loader.ts',
  },
}
```

### Route Handlers
- `GET` HTTP 동사만 지원됨
- 정적 JSON, HTML, TXT 파일 생성
- 동적 요청 값을 읽을 수 없음

### 브라우저 API
- `useEffect` 훅에서만 `window`, `localStorage`, `navigator` 접근
- 빌드 중 클라이언트 컴포넌트 사전 렌더링

## 지원되지 않는 기능 ❌

- `dynamicParams: true`를 사용한 동적 라우트
- `generateStaticParams()` 없는 동적 라우트
- Request에 의존하는 Route Handlers
- Cookies, Rewrites, Redirects, Headers
- Proxy, ISR, Draft Mode
- Server Actions
- Intercepting Routes
- 기본 Image Optimization 로더

## 배포

`next build` 후, `out` 폴더에는 다음이 포함됩니다:
- 라우트를 위한 `index.html`
- 에러 페이지를 위한 `404.html`
- 동적 라우트를 위한 중첩 HTML 파일

### Nginx 예시
```nginx
server {
  listen 80;
  server_name example.com;
  root /var/www/out;

  location / {
      try_files $uri $uri.html $uri/ =404;
  }

  error_page 404 /404.html;
}
```

모든 정적 호스팅 서비스에 배포 가능 (Vercel, Netlify, AWS S3, GitHub Pages 등)

## 버전 히스토리
- **v14.0.0**: `next export` 제거; `"output": "export"` 사용
- **v13.4.0**: App Router가 서버 컴포넌트로 향상된 정적 내보내기 지원
- **v13.3.0**: `next export` 더 이상 사용되지 않음

---

## 사용 사례

### 언제 정적 내보내기를 사용할까요?

**✅ 좋은 경우:**
- 블로그, 포트폴리오, 문서 사이트
- 마케팅 랜딩 페이지
- 콘텐츠가 자주 변경되지 않는 사이트
- CDN에서 전 세계로 배포
- 서버리스 비용 절감

**❌ 적합하지 않은 경우:**
- 동적 사용자 인증 필요
- 실시간 데이터 업데이트
- Server Actions 사용
- 요청 시점 렌더링 필요

### 하이브리드 접근 방식

일부 페이지는 정적으로, 일부는 동적으로:
1. 정적 부분을 정적 내보내기로
2. 동적 기능은 별도의 Next.js 앱 또는 API로
3. 클라이언트에서 API 호출로 연결
