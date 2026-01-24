# 정적 내보내기 (Static Exports)

Next.js는 정적 사이트 또는 SPA(Single-Page Application)로 시작하여 나중에 서버가 필요한 기능으로 업그레이드할 수 있습니다.

`next build` 실행 시 경로당 HTML 파일을 생성하여 번들 크기를 줄이고 페이지 로드를 빠르게 합니다.

## 설정 방법

`next.config.js`에서 output 모드를 변경합니다:

```js
// next.config.js
const nextConfig = {
  output: 'export',

  // 선택사항: /me -> /me/ 및 /me.html -> /me/index.html로 변경
  // trailingSlash: true,

  // 선택사항: 자동 리다이렉트 방지
  // skipTrailingSlashRedirect: true,

  // 선택사항: 출력 디렉토리 변경 (out -> dist)
  // distDir: 'dist',
}

module.exports = nextConfig
```

`next build` 실행 후 `out` 폴더에 HTML/CSS/JS 자산이 생성됩니다.

## 지원되는 기능

정적 내보내기에서 지원되는 Next.js 기능:

- `getStaticPaths`를 사용한 Dynamic Routes
- `next/link` 프리페칭
- JavaScript 프리로딩
- Dynamic Imports
- 모든 스타일링 옵션 (CSS Modules, styled-jsx 등)
- 클라이언트 측 데이터 페칭
- `getStaticProps`

### 이미지 최적화

정적 내보내기에서 이미지 최적화를 사용하려면 커스텀 로더를 설정해야 합니다:

```js
// next.config.js
const nextConfig = {
  output: 'export',
  images: {
    loader: 'custom',
    loaderFile: './my-loader.ts',
  },
}
```

커스텀 로더 예제 (Cloudinary):

```ts
// my-loader.ts
export default function cloudinaryLoader({
  src,
  width,
  quality,
}: {
  src: string
  width: number
  quality?: number
}) {
  const params = ['f_auto', 'c_limit', `w_${width}`, `q_${quality || 'auto'}`]
  return `https://res.cloudinary.com/demo/image/upload/${params.join(',')}${src}`
}
```

## 지원되지 않는 기능

서버가 필요한 기능은 정적 내보내기에서 지원되지 않습니다:

- Internationalized Routing
- API Routes
- Rewrites
- Redirects
- Headers
- Middleware
- Incremental Static Regeneration (ISR)
- 기본 로더를 사용한 Image Optimization
- Draft Mode
- `getStaticPaths`의 `fallback: true` 또는 `fallback: 'blocking'`
- `getServerSideProps`

## 빌드 출력 예시

빌드 후 생성되는 파일 구조:

```
out/
├── index.html
├── 404.html
├── about.html
├── blog/
│   ├── post-1.html
│   └── post-2.html
├── _next/
│   ├── static/
│   │   ├── chunks/
│   │   └── css/
│   └── data/
└── images/
```

## 배포

### Nginx 설정 예제

```nginx
server {
  listen 80;
  server_name acme.com;
  root /var/www/out;

  location / {
      try_files $uri $uri.html $uri/ =404;
  }

  # 블로그 경로 처리
  location /blog/ {
      rewrite ^/blog/(.*)$ /blog/$1.html break;
  }

  # 404 페이지
  error_page 404 /404.html;
  location = /404.html {
      internal;
  }
}
```

### Apache 설정 예제

```apache
# .htaccess
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /

  # HTML 확장자 없이 접근
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME}.html -f
  RewriteRule ^(.*)$ $1.html [L]
</IfModule>

ErrorDocument 404 /404.html
```

### AWS S3 + CloudFront

1. S3 버킷 생성 및 정적 웹 호스팅 활성화
2. `out` 폴더 내용을 S3에 업로드
3. CloudFront 배포 생성
4. 오류 페이지에서 404를 `/404.html`로 설정

### GitHub Pages

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./out
```

## 마이그레이션

기존 `next export` 명령어는 v14.0.0에서 제거되었습니다. 대신 `next.config.js`에서 `output: 'export'`를 사용하세요.

```js
// 이전 방식 (사용 중지)
// next build && next export

// 새로운 방식
// next.config.js에 output: 'export' 설정 후
// next build
```
