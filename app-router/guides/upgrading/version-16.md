# Next.js 16 업그레이드 가이드

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-12-09

---

## 15에서 16으로 업그레이드

### AI 에이전트와 Next.js DevTools MCP 사용

[Model Context Protocol (MCP)](https://modelcontextprotocol.io)를 지원하는 AI 코딩 어시스턴트를 사용 중이라면, **Next.js DevTools MCP**를 통해 업그레이드 프로세스를 자동화할 수 있습니다.

#### 설정

MCP 클라이언트에 다음 구성을 추가하세요:

```json
{
  "mcpServers": {
    "next-devtools": {
      "command": "npx",
      "args": ["-y", "next-devtools-mcp@latest"]
    }
  }
}
```

### Codemod 사용

Next.js 16으로 업데이트하려면 `upgrade` [codemod](./codemods.md)를 사용하세요:

```bash
npx @next/codemod@canary upgrade latest
```

수동으로 진행하려면, 최신 Next.js 및 React 버전을 설치하세요:

```bash
npm install next@latest react@latest react-dom@latest
```

---

## Node.js 런타임 및 브라우저 지원

| 요구사항 | 변경사항 / 상세정보 |
|---------|-------------------|
| Node.js 20.9+ | 최소 버전 `20.9.0` (LTS); Node.js 18 더 이상 지원 안 함 |
| TypeScript 5+ | 최소 버전 `5.1.0` |
| 브라우저 | Chrome 111+, Edge 111+, Firefox 111+, Safari 16.4+ |

---

## 기본적으로 Turbopack 사용

**Next.js 16**부터 Turbopack은 안정적이며 `next dev`와 `next build`에서 기본적으로 사용됩니다.

이전에는 `--turbopack` 또는 `--turbo` 플래그로 활성화해야 했지만, 이제 필요하지 않습니다:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

### Webpack 구성이 있는 경우

프로젝트에 커스텀 `webpack` 구성이 있고 `next build`를 실행하면 **빌드가 실패**합니다.

해결 방법:
* **Turbopack 계속 사용**: `next build --turbopack`으로 실행하여 webpack 구성을 무시
* **완전히 Turbopack으로 마이그레이션**: webpack 구성을 Turbopack 호환 옵션으로 마이그레이션
* **Webpack 계속 사용**: `--webpack` 플래그로 Turbopack을 비활성화

### Turbopack 비활성화

Webpack을 계속 사용해야 하면 `--webpack` 플래그로 비활성화:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build --webpack",
    "start": "next start"
  }
}
```

### Turbopack 구성 위치

`experimental.turbopack` 구성이 이제 실험적이 아닙니다.

**Next.js 15:**
```ts
const nextConfig: NextConfig = {
  experimental: {
    turbopack: {
      // options
    },
  },
}
```

**Next.js 16:**
```ts
const nextConfig: NextConfig = {
  turbopack: {
    // options
  },
}
```

---

## 비동기 요청 API (Breaking 변경)

버전 15는 비동기 요청 API를 임시 동기 호환성과 함께 소개했습니다.

**Next.js 16**부터는 동기 접근이 완전히 제거되었습니다. 이 API는 비동기로만 접근 가능합니다:

* `cookies`
* `headers`
* `draftMode`
* `params` (layout.js, page.js, route.js 등에서)
* `searchParams` (page.js에서)

[codemod](./codemods.md#비동기-dynamic-api로-마이그레이션)를 사용하여 비동기 동적 API로 마이그레이션하세요.

---

## React 19.2

**Next.js 16**의 App Router는 최신 React Canary 릴리스를 사용하며, 새로 릴리스된 React 19.2 기능을 포함합니다:

* **View Transitions**: Transition 또는 네비게이션 내에서 업데이트되는 엘리먼트 애니메이션
* **`useEffectEvent`**: Effect에서 비반응적 로직을 재사용 가능한 Effect Event 함수로 추출
* **Activity**: `display: none`으로 UI를 숨기면서 상태 유지 및 Effect 정리

---

## React 컴파일러 지원

React 컴파일러 1.0 릴리스 이후 **Next.js 16**에서 React 컴파일러에 대한 기본 제공 지원이 이제 안정적입니다.

`reactCompiler` 구성 옵션이 `experimental`에서 안정적으로 승격되었습니다:

```ts
const nextConfig: NextConfig = {
  reactCompiler: true,
}
```

최신 버전의 React 컴파일러 플러그인을 설치하세요:

```bash
npm install -D babel-plugin-react-compiler
```

---

## 캐싱 API

### revalidateTag

`revalidateTag`는 새로운 함수 시그니처를 가지고 있습니다. 두 번째 인자로 `cacheLife` 프로필을 전달할 수 있습니다:

```ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updateArticle(articleId: string) {
  revalidateTag(`article-${articleId}`, 'max')
}
```

### updateTag

`updateTag`는 새로운 Server Actions 전용 API로 **read-your-writes** 의미론을 제공합니다:

```ts
'use server'

import { updateTag } from 'next/cache'

export async function updateUserProfile(userId: string, profile: Profile) {
  await db.users.update(userId, profile)
  updateTag(`user-${userId}`)
}
```

### refresh

`refresh`를 사용하여 Server Action 내에서 클라이언트 라우터를 새로고침할 수 있습니다:

```ts
'use server'

import { refresh } from 'next/cache'

export async function markNotificationAsRead(notificationId: string) {
  await db.notifications.markAsRead(notificationId)
  refresh()
}
```

### cacheLife 및 cacheTag

`cacheLife`와 `cacheTag`는 이제 안정적입니다. `unstable_` 접두사가 더 이상 필요하지 않습니다.

---

## `middleware`에서 `proxy`로

`middleware` 파일명은 더 이상 사용되지 않으며 `proxy`로 이름이 변경되었습니다.

`edge` 런타임은 `proxy`에서 **지원되지 않습니다**. `proxy` 런타임은 `nodejs`입니다.

```bash
# middleware 파일 이름 변경
mv middleware.ts proxy.ts
```

명명된 `middleware` 내보내기도 더 이상 사용되지 않습니다:

```ts
export function proxy(request: Request) {}
```

---

## `next/image` 변경사항

### `minimumCacheTTL` 기본값 (Breaking 변경)

기본값이 `60초`에서 `4시간`(14400초)으로 변경되었습니다.

### `imageSizes` 기본값 (Breaking 변경)

기본 배열에서 값 `16`이 제거되었습니다.

### `qualities` 기본값 (Breaking 변경)

기본값이 모든 품질을 허용하는 것에서 `[75]`로만 변경되었습니다.

### `next/legacy/image` 컴포넌트 (더 이상 사용 안 함)

대신 `next/image`를 사용하세요.

### `images.domains` 구성 (더 이상 사용 안 함)

대신 `images.remotePatterns`를 사용하세요.

---

## 동시 `dev` 및 `build`

`next dev`와 `next build`는 이제 별도의 출력 디렉토리를 사용하여 동시 실행을 가능하게 합니다. `next dev` 명령은 `.next/dev`로 출력합니다.

---

## Parallel Routes `default.js` 요구사항

모든 parallel route 슬롯은 이제 명시적 `default.js` 파일이 필요합니다. 없으면 빌드가 실패합니다.

```tsx
import { notFound } from 'next/navigation'

export default function Default() {
  notFound()
}
```

---

## ESLint Flat Config

`@next/eslint-plugin-next`는 이제 기본적으로 ESLint Flat Config 형식을 사용합니다.

---

## 제거 사항

### AMP 지원

모든 AMP API 및 구성이 제거되었습니다.

### `next lint` 명령

`next lint` 명령이 제거되었습니다. ESLint를 직접 사용하세요.

### 런타임 구성

`serverRuntimeConfig`와 `publicRuntimeConfig`가 제거되었습니다. 대신 환경 변수를 사용하세요.

---

## 다음 단계

전체 변경 사항 및 세부 마이그레이션 가이드는 [Next.js 16 릴리스 노트](https://nextjs.org/blog/next-16)를 참조하세요.
