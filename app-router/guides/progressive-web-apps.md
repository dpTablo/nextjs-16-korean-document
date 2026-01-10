# Progressive Web Apps (PWAs)

Next.js에서 Progressive Web Application을 구축하는 방법을 알아봅니다.

## 개요

Progressive Web Application(PWA)은 웹의 접근성과 네이티브 앱의 기능을 결합합니다:

- **즉시 업데이트** - 앱 스토어 승인 없이 즉시 배포
- **크로스 플랫폼** - 단일 코드베이스로 모든 플랫폼 지원
- **네이티브 기능** - 홈 화면 설치, 푸시 알림 등

---

## Web App Manifest 생성

웹 앱 매니페스트는 PWA의 핵심 구성 요소입니다.

### TypeScript로 생성

**app/manifest.ts**
```tsx
import type { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'Next.js PWA',
    short_name: 'NextPWA',
    description: 'Next.js로 구축된 Progressive Web App',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    icons: [
      {
        src: '/icon-192x192.png',
        sizes: '192x192',
        type: 'image/png',
      },
      {
        src: '/icon-512x512.png',
        sizes: '512x512',
        type: 'image/png',
      },
    ],
  }
}
```

### JSON으로 생성

**app/manifest.json**
```json
{
  "name": "Next.js PWA",
  "short_name": "NextPWA",
  "description": "Next.js로 구축된 Progressive Web App",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### 아이콘 생성

1. **Favicon 생성기 사용**
   - [Favicon.io](https://favicon.io/)
   - [RealFaviconGenerator](https://realfavicongenerator.net/)

2. **생성된 파일을 `public/` 폴더에 배치**
   ```
   public/
   ├── icon-192x192.png
   ├── icon-512x512.png
   ├── apple-touch-icon.png
   └── favicon.ico
   ```

3. **홈 화면 설치 활성화**

---

## 매니페스트 속성

### 필수 속성

```tsx
{
  name: 'My App',              // 전체 앱 이름
  short_name: 'App',           // 홈 화면에 표시될 짧은 이름
  start_url: '/',              // 앱 시작 URL
  display: 'standalone',       // 디스플레이 모드
  icons: [...]                 // 앱 아이콘
}
```

### Display 모드

| 모드 | 설명 |
|------|------|
| `fullscreen` | 전체 화면 (브라우저 UI 없음) |
| `standalone` | 독립 실행형 (네이티브 앱처럼) |
| `minimal-ui` | 최소 브라우저 UI |
| `browser` | 일반 브라우저 탭 |

### 선택 속성

```tsx
{
  description: 'My awesome PWA',     // 앱 설명
  orientation: 'portrait',           // 화면 방향
  theme_color: '#000000',            // 테마 색상
  background_color: '#ffffff',       // 배경색
  scope: '/',                        // 앱 범위
  categories: ['productivity'],      // 앱 카테고리
  screenshots: [...]                 // 스크린샷
}
```

---

## Web Push Notifications

브라우저 푸시 알림을 구현합니다.

### 브라우저 지원

- **iOS** 16.4+ (홈 화면 앱)
- **Safari** 16+ (macOS 13+)
- **Chrome/Edge** - 완전 지원
- **Firefox** - 완전 지원

### 1. VAPID 키 생성

```bash
npm install -g web-push
web-push generate-vapid-keys
```

**.env.local**
```env
NEXT_PUBLIC_VAPID_PUBLIC_KEY=your_public_key_here
VAPID_PRIVATE_KEY=your_private_key_here
```

### 2. 메인 컴포넌트

**app/page.tsx**
```tsx
'use client'

import { useState, useEffect } from 'react'
import { subscribeUser, unsubscribeUser, sendNotification } from './actions'

function urlBase64ToUint8Array(base64String: string) {
  const padding = '='.repeat((4 - (base64String.length % 4)) % 4)
  const base64 = (base64String + padding).replace(/-/g, '+').replace(/_/g, '/')

  const rawData = window.atob(base64)
  const outputArray = new Uint8Array(rawData.length)

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i)
  }
  return outputArray
}

function PushNotificationManager() {
  const [isSupported, setIsSupported] = useState(false)
  const [subscription, setSubscription] = useState<PushSubscription | null>(null)
  const [message, setMessage] = useState('')

  useEffect(() => {
    if ('serviceWorker' in navigator && 'PushManager' in window) {
      setIsSupported(true)
      registerServiceWorker()
    }
  }, [])

  async function registerServiceWorker() {
    const registration = await navigator.serviceWorker.register('/sw.js', {
      scope: '/',
      updateViaCache: 'none',
    })
    const sub = await registration.pushManager.getSubscription()
    setSubscription(sub)
  }

  async function subscribeToPush() {
    const registration = await navigator.serviceWorker.ready
    const sub = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(
        process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!
      ),
    })
    setSubscription(sub)
    const serializedSub = JSON.parse(JSON.stringify(sub))
    await subscribeUser(serializedSub)
  }

  async function unsubscribeFromPush() {
    await subscription?.unsubscribe()
    setSubscription(null)
    await unsubscribeUser()
  }

  async function sendTestNotification() {
    if (subscription) {
      await sendNotification(message)
      setMessage('')
    }
  }

  if (!isSupported) {
    return <p>이 브라우저는 푸시 알림을 지원하지 않습니다.</p>
  }

  return (
    <div>
      <h3>푸시 알림</h3>
      {subscription ? (
        <>
          <p>푸시 알림을 구독 중입니다.</p>
          <button onClick={unsubscribeFromPush}>구독 취소</button>
          <input
            type="text"
            placeholder="알림 메시지 입력"
            value={message}
            onChange={(e) => setMessage(e.target.value)}
          />
          <button onClick={sendTestNotification}>테스트 전송</button>
        </>
      ) : (
        <>
          <p>푸시 알림을 구독하지 않았습니다.</p>
          <button onClick={subscribeToPush}>구독</button>
        </>
      )}
    </div>
  )
}

export default function Page() {
  return (
    <div>
      <PushNotificationManager />
    </div>
  )
}
```

### 3. Server Actions

**app/actions.ts**
```tsx
'use server'

import webpush from 'web-push'

webpush.setVapidDetails(
  'mailto:your-email@example.com',
  process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
)

let subscription: PushSubscription | null = null

export async function subscribeUser(sub: PushSubscription) {
  subscription = sub
  // 프로덕션: 데이터베이스에 저장
  // await db.subscriptions.create({ data: sub })
  return { success: true }
}

export async function unsubscribeUser() {
  subscription = null
  // 프로덕션: 데이터베이스에서 제거
  // await db.subscriptions.delete({ where: { ... } })
  return { success: true }
}

export async function sendNotification(message: string) {
  if (!subscription) {
    throw new Error('구독 정보가 없습니다')
  }

  try {
    await webpush.sendNotification(
      subscription,
      JSON.stringify({
        title: '테스트 알림',
        body: message,
        icon: '/icon.png',
      })
    )
    return { success: true }
  } catch (error) {
    console.error('푸시 알림 전송 오류:', error)
    return { success: false, error: '알림 전송 실패' }
  }
}
```

### 4. 의존성 설치

```bash
npm install web-push
```

---

## Service Worker

Service Worker는 PWA의 핵심 기능을 처리합니다.

### 기본 Service Worker

**public/sw.js**
```js
self.addEventListener('push', function (event) {
  if (event.data) {
    const data = event.data.json()
    const options = {
      body: data.body,
      icon: data.icon || '/icon.png',
      badge: '/badge.png',
      vibrate: [100, 50, 100],
      data: {
        dateOfArrival: Date.now(),
        primaryKey: '2',
      },
    }
    event.waitUntil(self.registration.showNotification(data.title, options))
  }
})

self.addEventListener('notificationclick', function (event) {
  console.log('알림 클릭 수신')
  event.notification.close()
  event.waitUntil(clients.openWindow('https://your-website.com'))
})
```

### Service Worker 등록

클라이언트 컴포넌트에서 등록:

```tsx
'use client'

import { useEffect } from 'react'

export default function ServiceWorkerRegistration() {
  useEffect(() => {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker
        .register('/sw.js', {
          scope: '/',
          updateViaCache: 'none',
        })
        .then((registration) => {
          console.log('Service Worker 등록 성공:', registration)
        })
        .catch((error) => {
          console.error('Service Worker 등록 실패:', error)
        })
    }
  }, [])

  return null
}
```

---

## 홈 화면 설치

### 요구사항

1. **유효한 웹 앱 매니페스트**
2. **HTTPS로 제공**
3. **최소 아이콘 요구사항**
   - 192x192px
   - 512x512px

### 설치 프롬프트

**app/components/InstallPrompt.tsx**
```tsx
'use client'

import { useState, useEffect } from 'react'

export default function InstallPrompt() {
  const [isIOS, setIsIOS] = useState(false)
  const [isStandalone, setIsStandalone] = useState(false)

  useEffect(() => {
    setIsIOS(
      /iPad|iPhone|iPod/.test(navigator.userAgent) && !(window as any).MSStream
    )
    setIsStandalone(window.matchMedia('(display-mode: standalone)').matches)
  }, [])

  if (isStandalone) {
    return null // 이미 설치됨
  }

  return (
    <div className="install-prompt">
      <h3>앱 설치</h3>
      {isIOS ? (
        <div>
          <p>iOS 기기에 이 앱을 설치하려면:</p>
          <ol>
            <li>공유 버튼 ⎋을 탭하세요</li>
            <li>"홈 화면에 추가" ➕를 선택하세요</li>
          </ol>
        </div>
      ) : (
        <button onClick={() => {
          // 크롬/엣지에서는 브라우저가 자동으로 프롬프트 표시
          console.log('설치 버튼 클릭')
        }}>
          홈 화면에 추가
        </button>
      )}
    </div>
  )
}
```

### iOS 설치 지침

iOS는 자동 설치 프롬프트를 지원하지 않으므로 수동 지침을 제공해야 합니다.

```tsx
'use client'

import { useState, useEffect } from 'react'

export function IOSInstallInstructions() {
  const [showInstructions, setShowInstructions] = useState(false)

  useEffect(() => {
    const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent)
    const isInStandaloneMode = window.matchMedia('(display-mode: standalone)').matches

    if (isIOS && !isInStandaloneMode) {
      setShowInstructions(true)
    }
  }, [])

  if (!showInstructions) return null

  return (
    <div className="ios-install-banner">
      <p>
        Safari에서 공유 <span role="img" aria-label="공유">⎋</span> 버튼을 눌러
        "홈 화면에 추가" <span role="img" aria-label="추가">➕</span>를 선택하세요
      </p>
    </div>
  )
}
```

---

## 로컬 개발 및 테스트

### HTTPS로 개발 서버 실행

PWA 기능은 HTTPS가 필요합니다 (localhost 제외).

```bash
next dev --experimental-https
```

### 테스트 체크리스트

✅ **확인 사항:**
- [ ] HTTPS로 실행 중
- [ ] 브라우저 알림 활성화됨
- [ ] 권한 요청 시 수락
- [ ] 알림이 전역적으로 비활성화되지 않음
- [ ] Service Worker 등록 확인

### 브라우저 DevTools 확인

**Chrome/Edge:**
1. F12 → Application 탭
2. Manifest 확인
3. Service Workers 상태 확인
4. Push Messaging 테스트

**Safari:**
1. Develop → Web Inspector
2. Storage → Service Workers
3. Console에서 에러 확인

---

## 보안 구성

### Next.js 보안 헤더

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
        ],
      },
      {
        source: '/sw.js',
        headers: [
          {
            key: 'Content-Type',
            value: 'application/javascript; charset=utf-8',
          },
          {
            key: 'Cache-Control',
            value: 'no-cache, no-store, must-revalidate',
          },
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self'",
          },
        ],
      },
    ]
  },
}

module.exports = nextConfig
```

### 보안 헤더 설명

**전역 헤더:**
- `X-Content-Type-Options: nosniff` - MIME 스니핑 방지
- `X-Frame-Options: DENY` - 클릭재킹 방지
- `Referrer-Policy: strict-origin-when-cross-origin` - 리퍼러 정보 제어

**Service Worker 헤더:**
- `Content-Type: application/javascript` - 올바른 해석
- `Cache-Control: no-cache, no-store, must-revalidate` - 항상 최신 버전
- `Content-Security-Policy` - 엄격한 origin-only 스크립트

---

## 오프라인 지원

### Serwist 통합

[Serwist](https://github.com/serwist/serwist)는 Next.js용 Service Worker 라이브러리입니다.

#### 1. 설치

```bash
npm install @serwist/next
```

#### 2. 구성

**next.config.js**
```js
const withSerwist = require('@serwist/next').default({
  swSrc: 'app/sw.ts',
  swDest: 'public/sw.js',
})

module.exports = withSerwist({
  // Next.js 구성
})
```

#### 3. Service Worker 작성

**app/sw.ts**
```ts
import { defaultCache } from '@serwist/next/worker'
import type { PrecacheEntry, SerwistGlobalConfig } from 'serwist'
import { Serwist } from 'serwist'

declare global {
  interface WorkerGlobalScope extends SerwistGlobalConfig {
    __SW_MANIFEST: (PrecacheEntry | string)[] | undefined
  }
}

declare const self: ServiceWorkerGlobalScope

const serwist = new Serwist({
  precacheEntries: self.__SW_MANIFEST,
  skipWaiting: true,
  clientsClaim: true,
  navigationPreload: true,
  runtimeCaching: defaultCache,
})

serwist.addEventListeners()
```

> **참고:** Serwist는 현재 webpack 구성이 필요합니다.

---

## 고급 기능

### Background Sync

네트워크 연결이 복원되면 요청을 재시도합니다.

**public/sw.js**
```js
self.addEventListener('sync', function(event) {
  if (event.tag === 'sync-messages') {
    event.waitUntil(syncMessages())
  }
})

async function syncMessages() {
  const messages = await getUnsentMessages()
  return Promise.all(
    messages.map(msg =>
      fetch('/api/send', {
        method: 'POST',
        body: JSON.stringify(msg)
      })
    )
  )
}
```

### Periodic Background Sync

주기적으로 콘텐츠를 업데이트합니다.

```js
// Service Worker 등록 시
const registration = await navigator.serviceWorker.ready
await registration.periodicSync.register('content-sync', {
  minInterval: 24 * 60 * 60 * 1000 // 24시간
})

// Service Worker 내
self.addEventListener('periodicsync', (event) => {
  if (event.tag === 'content-sync') {
    event.waitUntil(updateContent())
  }
})
```

### File System Access API

파일 시스템에 접근합니다.

```tsx
async function saveFile() {
  const handle = await window.showSaveFilePicker()
  const writable = await handle.createWritable()
  await writable.write('Hello, World!')
  await writable.close()
}
```

---

## 정적 내보내기와 PWA

서버 없는 배포를 위해 정적 내보내기를 사용할 수 있습니다.

### 1. 정적 내보내기 구성

**next.config.js**
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
}

module.exports = nextConfig
```

### 2. Server Actions 대체

정적 내보내기는 Server Actions를 지원하지 않으므로 외부 API를 사용해야 합니다.

**lib/api.ts**
```ts
export async function subscribeUser(subscription: PushSubscription) {
  const response = await fetch('https://your-api.com/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription),
  })
  return response.json()
}

export async function sendNotification(message: string) {
  const response = await fetch('https://your-api.com/notify', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message }),
  })
  return response.json()
}
```

### 3. 보안 헤더를 프록시로 이동

정적 호스팅의 경우 Cloudflare, Netlify 등에서 헤더를 설정합니다.

**_headers (Netlify)**
```
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin

/sw.js
  Content-Type: application/javascript; charset=utf-8
  Cache-Control: no-cache, no-store, must-revalidate
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **Progressive Enhancement**
   ```tsx
   if ('serviceWorker' in navigator) {
     // PWA 기능 활성화
   }
   ```

2. **오류 처리**
   ```tsx
   try {
     await registration.pushManager.subscribe(...)
   } catch (error) {
     console.error('구독 실패:', error)
     // 사용자에게 피드백 제공
   }
   ```

3. **권한 확인**
   ```tsx
   const permission = await Notification.requestPermission()
   if (permission === 'granted') {
     // 알림 구독
   }
   ```

4. **사용자 경험**
   - 설치 프롬프트를 자연스럽게 통합
   - 오프라인 폴백 제공
   - 로딩 상태 표시

5. **테스트**
   - 다양한 브라우저에서 테스트
   - 오프라인 시나리오 테스트
   - 설치 흐름 검증

### ❌ 피해야 할 것

1. **권한 요청 남용**
   ```tsx
   // ❌ 페이지 로드 즉시 요청
   useEffect(() => {
     Notification.requestPermission()
   }, [])

   // ✅ 사용자 액션에 따라 요청
   <button onClick={() => Notification.requestPermission()}>
     알림 활성화
   </button>
   ```

2. **Service Worker 캐싱 오용**
   - API 응답을 너무 오래 캐싱하지 않기
   - 사용자 데이터는 신중하게 캐싱

3. **beforeinstallprompt 의존**
   - 크로스 브라우저 지원 부족
   - iOS에서 작동하지 않음

4. **HTTP에서 테스트**
   - PWA 기능은 HTTPS 필요 (localhost 제외)

---

## 문제 해결

### 푸시 알림이 작동하지 않음

**1. 권한 확인:**
```tsx
console.log(Notification.permission) // "granted", "denied", "default"
```

**2. VAPID 키 확인:**
```bash
# 환경 변수가 올바르게 설정되었는지 확인
echo $NEXT_PUBLIC_VAPID_PUBLIC_KEY
```

**3. Service Worker 상태 확인:**
```tsx
const registration = await navigator.serviceWorker.ready
console.log(registration.active) // ServiceWorker 객체여야 함
```

### 홈 화면에 추가되지 않음

**요구사항 확인:**
- [ ] HTTPS로 제공 (또는 localhost)
- [ ] 유효한 manifest.json
- [ ] 192x192, 512x512 아이콘
- [ ] Service Worker 등록됨

**Chrome DevTools:**
1. F12 → Application → Manifest
2. 오류 메시지 확인

### Service Worker가 업데이트되지 않음

**해결책:**
```js
// Service Worker 강제 업데이트
navigator.serviceWorker.register('/sw.js', {
  updateViaCache: 'none'
})

// 또는 수동 업데이트
const registration = await navigator.serviceWorker.ready
await registration.update()
```

---

## 실전 예시

### 완전한 PWA 설정

**app/layout.tsx**
```tsx
import type { Metadata } from 'next'
import './globals.css'

export const metadata: Metadata = {
  title: 'My PWA',
  description: 'A Progressive Web App built with Next.js',
  manifest: '/manifest.json',
  themeColor: '#000000',
  appleWebApp: {
    capable: true,
    statusBarStyle: 'default',
    title: 'My PWA',
  },
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <head>
        <link rel="icon" href="/favicon.ico" />
        <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
      </head>
      <body>{children}</body>
    </html>
  )
}
```

**app/components/PWAProvider.tsx**
```tsx
'use client'

import { useEffect } from 'react'

export function PWAProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker
        .register('/sw.js', {
          scope: '/',
          updateViaCache: 'none',
        })
        .then((registration) => {
          console.log('SW 등록 성공:', registration.scope)
        })
        .catch((error) => {
          console.error('SW 등록 실패:', error)
        })
    }
  }, [])

  return <>{children}</>
}
```

---

## 참고 자료

### 공식 문서

- [Next.js Manifest](../api-reference/file-conventions/metadata/manifest.md)
- [Content Security Policy](./content-security-policy.md)
- [Static Exports](./static-exports.md)

### 외부 리소스

- [Web.dev - PWA](https://web.dev/progressive-web-apps/)
- [MDN - Progressive Web Apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [What PWA Can Do Today](https://whatpwacando.today/)
- [VAPID Keys Generator](https://vapidkeys.com/)
- [Serwist Documentation](https://serwist.pages.dev/)

---

## 다음 단계

- [Manifest 파일](../api-reference/file-conventions/metadata/manifest.md)
- [Static Exports](./static-exports.md) - 정적 내보내기
- [Content Security Policy](./content-security-policy.md) - CSP 구성

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11

**참고:**
- PWA는 HTTPS가 필요합니다 (localhost 제외)
- iOS 지원은 제한적일 수 있습니다
- Service Worker는 신중하게 구현해야 합니다
