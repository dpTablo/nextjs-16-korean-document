# 배포

Next.js는 Node.js 서버, Docker 컨테이너, 정적 내보내기(Static Export) 또는 다양한 플랫폼에 맞게 적응하여 배포할 수 있습니다.

## 배포 옵션 비교

| 배포 옵션 | 기능 지원 |
|---------|---------|
| Node.js 서버 | 모두 지원 |
| Docker 컨테이너 | 모두 지원 |
| 정적 내보내기 | 제한적 |
| 어댑터 | 플랫폼별 |

## Node.js 서버 배포

### package.json 설정

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

### 배포 절차

1. `npm run build`로 애플리케이션 빌드
2. `npm run start`로 Node.js 서버 시작

**특징:**
- 모든 Next.js 기능 지원
- 필요시 커스텀 서버로 확장 가능

**지원 플랫폼:**
- Flightcontrol
- Railway
- Replit

## Docker 배포

Docker를 지원하는 모든 제공자에서 배포할 수 있습니다:

- Kubernetes 같은 컨테이너 오케스트레이터
- 자체 호스팅 서버
- 클라우드 제공자

**특징:**
- 모든 Next.js 기능 지원
- 일관된 환경 제공

**Dockerfile 예시:**

```dockerfile
FROM node:20-alpine AS base

# 의존성 설치
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# 빌드
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# 프로덕션
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT=3000

CMD ["node", "server.js"]
```

**지원 플랫폼:**
- DigitalOcean
- Fly.io
- Google Cloud Run
- Render
- SST

> **참고:** Mac/Windows에서 개발 시 Docker 대신 로컬 개발(`npm run dev`)을 권장합니다.

## 정적 내보내기 (Static Export)

### 특징

- 정적 사이트 또는 SPA로 시작 가능
- 나중에 서버 기능으로 업그레이드 가능
- AWS S3, Nginx, Apache 등 정적 자산을 서빙할 수 있는 모든 웹 서버에 배포 가능

### 제한사항

서버가 필요한 Next.js 기능은 지원되지 않습니다:
- API Routes
- 서버 사이드 렌더링 (SSR)
- Incremental Static Regeneration (ISR)
- Internationalized Routing

**지원 플랫폼:**
- GitHub Pages
- AWS S3
- Nginx
- Apache

## 어댑터 (Adapters)

Next.js를 다양한 플랫폼에 맞춰 실행할 수 있는 어댑터를 제공합니다:

| 플랫폼 | 설명 |
|--------|------|
| **Vercel** | Next.js 공식 플랫폼 |
| **AWS Amplify Hosting** | AWS 기반 호스팅 |
| **Cloudflare** | 엣지 컴퓨팅 플랫폼 |
| **Netlify** | JAMstack 호스팅 |
| **Deno Deploy** | Deno 런타임 기반 |
| **Firebase App Hosting** | Google Firebase 기반 |
| **Appwrite Sites** | 오픈소스 백엔드 플랫폼 |

## 프로덕션 체크리스트

배포 전 확인사항:

1. **환경 변수 설정**
   - 프로덕션 환경 변수가 올바르게 설정되었는지 확인

2. **빌드 최적화**
   - `next build`가 오류 없이 완료되는지 확인
   - 번들 크기 분석 (`@next/bundle-analyzer`)

3. **보안**
   - HTTPS 사용
   - 보안 헤더 설정
   - 환경 변수에 민감한 정보 저장

4. **성능**
   - 이미지 최적화
   - 코드 분할
   - 캐싱 전략

5. **모니터링**
   - 에러 추적 설정
   - 성능 모니터링

## 다음 단계

- [정적 내보내기](/pages-router/building-your-application/deploying/static-exports.md)
- [Multi-Zones](/pages-router/building-your-application/deploying/multi-zones.md)
- [CI 빌드 캐싱](/pages-router/building-your-application/deploying/ci-build-caching.md)
