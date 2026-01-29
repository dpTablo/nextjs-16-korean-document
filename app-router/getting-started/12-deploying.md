---
원문: https://nextjs.org/docs/app/getting-started/deploying
버전: 16.1.6
---

# 배포

## 개요

Next.js는 Node.js 서버, Docker 컨테이너, 정적 내보내기로 배포하거나 다양한 플랫폼에서 실행되도록 적응될 수 있습니다.

| 배포 옵션 | 기능 지원 |
|---|---|
| Node.js 서버 | 모두 |
| Docker 컨테이너 | 모두 |
| 정적 내보내기 | 제한적 |
| 어댑터 | 플랫폼별 |

---

## Node.js 서버

Next.js는 Node.js를 지원하는 모든 프로바이더에 배포할 수 있습니다. `package.json`에 `"build"` 및 `"start"` 스크립트가 포함되어야 합니다:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

**배포 단계:**
1. `npm run build`를 실행하여 애플리케이션 빌드
2. `npm run start`를 실행하여 Node.js 서버 시작

이 서버는 모든 Next.js 기능을 지원합니다. 필요한 경우 [커스텀 서버로 이젝트](/docs/app/guides/custom-server.md)할 수도 있습니다.

**템플릿:**
- [Flightcontrol](https://github.com/nextjs/deploy-flightcontrol)
- [Railway](https://github.com/nextjs/deploy-railway)
- [Replit](https://github.com/nextjs/deploy-replit)

---

## Docker

Next.js는 Kubernetes 및 클라우드 프로바이더를 포함하여 Docker 컨테이너를 지원하는 모든 프로바이더에 배포할 수 있습니다.

> **개발 참고:** Mac 및 Windows의 경우 개발 중 더 나은 성능을 위해 Docker 대신 로컬 개발(`npm run dev`)을 사용하세요.

**템플릿:**
- [Docker](https://github.com/vercel/next.js/tree/canary/examples/with-docker)
- [Docker Multi-Environment](https://github.com/vercel/next.js/tree/canary/examples/with-docker-multi-env)
- [DigitalOcean](https://github.com/nextjs/deploy-digitalocean)
- [Fly.io](https://github.com/nextjs/deploy-fly)
- [Google Cloud Run](https://github.com/nextjs/deploy-google-cloud-run)
- [Render](https://github.com/nextjs/deploy-render)
- [SST](https://github.com/nextjs/deploy-sst)

---

## 정적 내보내기

Next.js는 HTML/CSS/JS 에셋을 제공하는 모든 웹 서버(AWS S3, Nginx, Apache 등)에 정적 사이트 또는 SPA(Single-Page Application)로 배포할 수 있습니다.

**제한사항:** 정적 내보내기는 서버가 필요한 Next.js 기능을 지원하지 않습니다.

**템플릿:**
- [GitHub Pages](https://github.com/nextjs/deploy-github-pages)

---

## 어댑터

Next.js는 다양한 플랫폼에서 실행되도록 적응될 수 있습니다. 지원되는 기능에 대해서는 각 프로바이더의 문서를 확인하세요:

- [Appwrite Sites](https://appwrite.io/docs/products/sites/quick-start/nextjs)
- [AWS Amplify Hosting](https://docs.amplify.aws/nextjs/start/quickstart/nextjs-app-router-client-components)
- [Cloudflare](https://developers.cloudflare.com/workers/frameworks/framework-guides/nextjs)
- [Deno Deploy](https://docs.deno.com/examples/next_tutorial)
- [Firebase App Hosting](https://firebase.google.com/docs/app-hosting/get-started)
- [Netlify](https://docs.netlify.com/frameworks/next-js/overview/#next-js-support-on-netlify)
- [Vercel](https://vercel.com/docs/frameworks/nextjs)

> **참고:** 모든 플랫폼이 채택할 수 있도록 Deployment Adapters API가 개발 중입니다.
