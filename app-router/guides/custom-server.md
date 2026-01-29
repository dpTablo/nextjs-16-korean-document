---
원문: https://nextjs.org/docs/app/guides/custom-server
버전: 16.1.6
---

# Custom Server

커스텀 서버를 사용하여 Next.js를 프로그래밍 방식으로 시작하는 방법을 알아봅니다.

## 개요

Next.js는 기본적으로 `next start`를 통해 자체 서버를 포함합니다. 커스텀 Next.js 서버를 사용하면 사용자 정의 패턴을 위해 프로그래밍 방식으로 서버를 시작할 수 있습니다. 대부분의 애플리케이션에서는 필요하지 않지만, 표준 설정에서 벗어나야 할 때 사용할 수 있습니다.

---

## 주의사항

커스텀 서버를 사용하기 전에 다음을 고려하세요:

- Next.js의 통합 라우터가 요구사항을 충족할 수 없는 경우에만 사용해야 합니다
- 커스텀 서버는 **자동 정적 최적화(Automatic Static Optimization)** 같은 중요한 성능 최적화를 제거합니다
- `standalone` 출력 모드는 커스텀 서버와 함께 사용할 수 없습니다 (별도의 최소 `server.js` 파일을 출력함)

**권장사항:** 대부분의 경우 기본 `next start`나 Middleware를 사용하는 것이 더 좋습니다.

---

## 기본 커스텀 서버 예시

### TypeScript 버전

**server.ts**
```ts
import { createServer } from 'http'
import { parse } from 'url'
import next from 'next'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    const parsedUrl = parse(req.url!, true)
    handle(req, res, parsedUrl)
  }).listen(port)

  console.log(
    `> 서버가 http://localhost:${port}에서 ${
      dev ? '개발' : '프로덕션'
    } 모드로 실행 중입니다`
  )
})
```

### JavaScript 버전

**server.js**
```js
import { createServer } from 'http'
import { parse } from 'url'
import next from 'next'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    const parsedUrl = parse(req.url, true)
    handle(req, res, parsedUrl)
  }).listen(port)

  console.log(
    `> 서버가 http://localhost:${port}에서 ${
      dev ? '개발' : '프로덕션'
    } 모드로 실행 중입니다`
  )
})
```

---

## package.json 구성

`package.json`의 scripts를 업데이트합니다:

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
```

### TypeScript 사용 시

TypeScript를 사용하는 경우 `tsx` 또는 `ts-node`를 사용합니다:

```json
{
  "scripts": {
    "dev": "tsx server.ts",
    "build": "next build",
    "start": "NODE_ENV=production tsx server.ts"
  }
}
```

**tsx 설치:**
```bash
npm install -D tsx
```

### 개발 중 자동 재시작

`nodemon`을 사용하여 파일 변경 시 자동으로 서버를 재시작할 수 있습니다:

```json
{
  "scripts": {
    "dev": "nodemon server.js"
  }
}
```

**nodemon.json**
```json
{
  "watch": ["server.js", "app/**/*", "lib/**/*"],
  "ext": "js,jsx,ts,tsx,json",
  "exec": "node server.js"
}
```

---

## Next.js 가져오기 및 구성

```ts
import next from 'next'

const app = next({
  dev: process.env.NODE_ENV !== 'production',
  // 추가 옵션
})
```

### 사용 가능한 옵션

| 옵션 | 타입 | 설명 |
|------|------|------|
| `conf` | `Object` | `next.config.js`에서 사용하는 것과 동일한 객체. 기본값: `{}` |
| `dev` | `Boolean` | Next.js를 개발 모드로 시작. 기본값: `false` |
| `dir` | `String` | Next.js 프로젝트 위치. 기본값: `'.'` |
| `quiet` | `Boolean` | 에러 메시지 숨기기. 기본값: `false` |
| `hostname` | `String` | 서버가 실행되는 호스트명 |
| `port` | `Number` | 서버가 실행되는 포트 |
| `httpServer` | `node:http#Server` | Next.js가 실행되는 HTTP 서버 |
| `turbopack` | `Boolean` | Turbopack 활성화. 기본값: `true` |
| `webpack` | `Boolean` | webpack 활성화 |

---

## 고급 예시

### 커스텀 라우팅

특정 경로에 대한 커스텀 처리를 추가합니다.

```ts
import { createServer } from 'http'
import { parse } from 'url'
import next from 'next'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer(async (req, res) => {
    const parsedUrl = parse(req.url!, true)
    const { pathname, query } = parsedUrl

    // 커스텀 라우팅
    if (pathname === '/a') {
      await app.render(req, res, '/a', query)
    } else if (pathname === '/b') {
      await app.render(req, res, '/b', query)
    } else if (pathname?.startsWith('/api/custom')) {
      // 커스텀 API 처리
      res.writeHead(200, { 'Content-Type': 'application/json' })
      res.end(JSON.stringify({ message: '커스텀 API' }))
    } else {
      handle(req, res, parsedUrl)
    }
  }).listen(port)

  console.log(`> 서버가 http://localhost:${port}에서 실행 중입니다`)
})
```

### Express와 통합

Express를 사용하여 더 복잡한 서버 로직을 처리합니다.

**server.ts**
```ts
import express from 'express'
import next from 'next'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  const server = express()

  // Express 미들웨어
  server.use(express.json())

  // 커스텀 API 라우트
  server.get('/api/custom', (req, res) => {
    res.json({ message: '커스텀 Express API' })
  })

  // 프록시 설정
  server.use('/proxy', (req, res) => {
    // 프록시 로직
  })

  // Next.js 처리
  server.all('*', (req, res) => {
    return handle(req, res)
  })

  server.listen(port, () => {
    console.log(`> 서버가 http://localhost:${port}에서 실행 중입니다`)
  })
})
```

**Express 설치:**
```bash
npm install express
npm install -D @types/express
```

### Fastify와 통합

고성능이 필요한 경우 Fastify를 사용합니다.

**server.ts**
```ts
import Fastify from 'fastify'
import next from 'next'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

const fastify = Fastify({ logger: true })

app.prepare().then(() => {
  // 커스텀 라우트
  fastify.get('/api/custom', async (request, reply) => {
    return { message: '커스텀 Fastify API' }
  })

  // Next.js 처리
  fastify.all('*', async (request, reply) => {
    await handle(request.raw, reply.raw)
    reply.sent = true
  })

  fastify.listen({ port }, (err) => {
    if (err) throw err
    console.log(`> 서버가 http://localhost:${port}에서 실행 중입니다`)
  })
})
```

**Fastify 설치:**
```bash
npm install fastify
```

### WebSocket 지원

WebSocket을 커스텀 서버와 함께 사용합니다.

**server.ts**
```ts
import { createServer } from 'http'
import { parse } from 'url'
import next from 'next'
import { WebSocketServer } from 'ws'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  const server = createServer((req, res) => {
    const parsedUrl = parse(req.url!, true)
    handle(req, res, parsedUrl)
  })

  // WebSocket 서버 설정
  const wss = new WebSocketServer({ server, path: '/ws' })

  wss.on('connection', (ws) => {
    console.log('클라이언트 연결됨')

    ws.on('message', (message) => {
      console.log('수신된 메시지:', message.toString())
      ws.send(`에코: ${message}`)
    })

    ws.on('close', () => {
      console.log('클라이언트 연결 해제됨')
    })
  })

  server.listen(port)
  console.log(`> 서버가 http://localhost:${port}에서 실행 중입니다`)
  console.log(`> WebSocket 서버가 ws://localhost:${port}/ws에서 실행 중입니다`)
})
```

**ws 설치:**
```bash
npm install ws
npm install -D @types/ws
```

---

## 프로덕션 배포

### 프로세스 관리자 사용

PM2를 사용하여 프로덕션에서 서버를 관리합니다.

**ecosystem.config.js**
```js
module.exports = {
  apps: [
    {
      name: 'next-app',
      script: 'server.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'development',
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
    },
  ],
}
```

**PM2 명령어:**
```bash
# 설치
npm install -g pm2

# 시작
pm2 start ecosystem.config.js --env production

# 상태 확인
pm2 status

# 로그 확인
pm2 logs

# 재시작
pm2 reload next-app

# 중지
pm2 stop next-app
```

### Docker 배포

**Dockerfile**
```dockerfile
FROM node:20-alpine

WORKDIR /app

# 의존성 설치
COPY package*.json ./
RUN npm ci

# 소스 복사 및 빌드
COPY . .
RUN npm run build

# 프로덕션 모드 실행
ENV NODE_ENV=production
EXPOSE 3000

CMD ["node", "server.js"]
```

---

## 커스텀 서버 vs 대안

커스텀 서버 대신 고려할 수 있는 대안들:

### 1. Middleware

대부분의 커스텀 로직은 Middleware로 처리할 수 있습니다.

**middleware.ts**
```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // 인증 체크
  const token = request.cookies.get('token')
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  // 커스텀 헤더 추가
  const response = NextResponse.next()
  response.headers.set('x-custom-header', 'value')

  return response
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
}
```

### 2. Route Handlers

API 라우트는 Route Handlers로 처리합니다.

**app/api/custom/route.ts**
```ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  return NextResponse.json({ message: '커스텀 API' })
}
```

### 3. Instrumentation

서버 시작 시 초기화 로직은 Instrumentation을 사용합니다.

**instrumentation.ts**
```ts
export async function register() {
  // 서버 시작 시 실행
  console.log('서버 초기화')

  // 데이터베이스 연결, 모니터링 설정 등
}
```

---

## 주의사항

### 컴파일 관련

`server.js`는 Next.js 컴파일러나 번들링 프로세스를 거치지 않습니다. 구문과 소스 코드가 현재 Node.js 버전과 호환되는지 확인하세요.

### ESM vs CommonJS

```js
// ESM 사용 시 (package.json에 "type": "module" 필요)
import next from 'next'

// CommonJS 사용 시
const next = require('next')
```

### 성능 고려사항

- 자동 정적 최적화가 비활성화됨
- ISR이 제한될 수 있음
- 클러스터링을 직접 구현해야 함

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **정말 필요한 경우에만 사용**
   - Middleware나 Route Handlers로 해결할 수 없는 경우에만

2. **적절한 에러 처리**
   ```ts
   createServer((req, res) => {
     handle(req, res).catch((err) => {
       console.error(err)
       res.statusCode = 500
       res.end('Internal Server Error')
     })
   })
   ```

3. **프로세스 관리자 사용**
   - PM2, forever 등 사용

4. **헬스 체크 엔드포인트 추가**
   ```ts
   if (pathname === '/health') {
     res.writeHead(200)
     res.end('OK')
     return
   }
   ```

### ❌ 피해야 할 것

1. **불필요한 복잡성 추가**
   - 기본 기능으로 충분한 경우 커스텀 서버 사용 금지

2. **standalone 모드와 혼용**
   - `output: 'standalone'`과 커스텀 서버는 호환되지 않음

3. **개발 모드에서 프로덕션 설정**
   - `dev` 플래그를 올바르게 설정

---

## 다음 단계

- [Middleware](../api-reference/file-conventions/middleware.md) - 미들웨어 가이드
- [Route Handlers](../getting-started/route-handlers.md) - API 라우트
- [Instrumentation](../api-reference/file-conventions/instrumentation.md) - 초기화 로직
- [Self-Hosting](./self-hosting.md) - 셀프 호스팅 가이드

---
