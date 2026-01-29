---
원문: https://nextjs.org/docs/app/guides/draft-mode
버전: 16.1.6
---

# Draft Mode (초안 모드)

헤드리스 CMS의 초안 콘텐츠를 Next.js 애플리케이션에서 미리보기하는 방법을 알아봅니다.

## 개요

Draft Mode는 헤드리스 CMS의 초안 콘텐츠를 요청 시점에 정적 렌더링에서 동적 렌더링으로 전환하여 미리보기할 수 있게 합니다. 전체 사이트를 다시 빌드하지 않고도 초안 변경 사항을 볼 수 있습니다.

### 주요 특징

- ✅ **실시간 미리보기** - 빌드 없이 초안 콘텐츠 확인
- ✅ **동적 렌더링** - 초안 모드에서만 동적으로 데이터 페칭
- ✅ **보안** - 비밀 토큰으로 무단 액세스 방지
- ✅ **CMS 통합** - 대부분의 헤드리스 CMS와 호환

---

## 작동 방식

```
CMS에서 "미리보기" 클릭
    ↓
/api/draft 라우트 핸들러 호출
    ↓
비밀 토큰 검증
    ↓
Draft Mode 활성화 (쿠키 설정)
    ↓
콘텐츠 페이지로 리디렉션
    ↓
동적 렌더링으로 초안 데이터 페칭
```

---

## 1단계: Route Handler 생성

Draft Mode를 활성화하는 API 엔드포인트를 생성합니다.

**app/api/draft/route.ts**
```ts
import { draftMode } from 'next/headers'

export async function GET(request: Request) {
  // Draft Mode 활성화
  const draft = await draftMode()
  draft.enable()

  return new Response('Draft mode is enabled')
}
```

### 작동 원리

이 코드는 `__prerender_bypass`라는 쿠키를 설정하여 후속 요청에서 Draft Mode를 활성화합니다.

**설정되는 쿠키:**
```
__prerender_bypass=<random-value>; Path=/; HttpOnly; SameSite=None; Secure
```

---

## 2단계: CMS 통합 보안 설정

### CMS 구성

1. **비밀 토큰 생성** - Next.js 앱과 CMS만 아는 토큰
2. **커스텀 초안 URL 설정** - CMS 설정에서 다음과 같이 설정:

```bash
https://<your-site>/api/draft?secret=<token>&slug=<path>
```

### 보안 Route Handler 구현

**app/api/draft/route.ts**
```ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  // 쿼리 파라미터 파싱
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')

  // 1. 비밀 토큰 검증
  if (secret !== process.env.DRAFT_SECRET_TOKEN) {
    return new Response('Invalid token', { status: 401 })
  }

  // 2. slug 존재 확인
  if (!slug) {
    return new Response('Missing slug', { status: 400 })
  }

  // 3. CMS에서 slug 유효성 검증
  const post = await getPostBySlug(slug)

  if (!post) {
    return new Response('Invalid slug', { status: 404 })
  }

  // 4. Draft Mode 활성화
  const draft = await draftMode()
  draft.enable()

  // 5. 실제 포스트 경로로 리디렉션
  redirect(post.slug)
}
```

**환경 변수 설정 (.env.local):**
```bash
DRAFT_SECRET_TOKEN=your-secret-token-here
```

> **보안 주의사항:**
> - 쿼리 파라미터의 `slug`가 아닌 CMS에서 가져온 `post.slug`로 리디렉션하세요
> - 이렇게 하면 오픈 리디렉션 취약점을 방지할 수 있습니다

---

## 3단계: 초안 콘텐츠 미리보기

페이지에서 `draftMode().isEnabled`를 확인하여 초안 데이터를 페칭합니다.

### 기본 예시

**app/posts/[slug]/page.tsx**
```tsx
import { draftMode } from 'next/headers'

async function getData(slug: string) {
  const { isEnabled } = await draftMode()

  // Draft Mode에 따라 엔드포인트 선택
  const url = isEnabled
    ? `https://draft.example.com/posts/${slug}`
    : `https://api.example.com/posts/${slug}`

  const res = await fetch(url)

  if (!res.ok) {
    throw new Error('Failed to fetch data')
  }

  return res.json()
}

export default async function PostPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const { title, content } = await getData(slug)

  return (
    <article>
      <h1>{title}</h1>
      <div dangerouslySetInnerHTML={{ __html: content }} />
    </article>
  )
}
```

### 주요 동작

- **Draft 쿠키 있음** → 요청 시점에 데이터 페칭 (동적 렌더링)
- **Draft 쿠키 없음** → 빌드 시점에 데이터 페칭 (정적 렌더링)
- `isEnabled`가 `true`일 때 초안 엔드포인트 사용
- 사이트를 다시 빌드하지 않고도 초안 업데이트 확인 가능

---

## CMS별 통합 예시

### Contentful 통합

**CMS 초안 URL 설정:**
```
https://your-site.com/api/draft?secret=YOUR_TOKEN&slug=/posts/{entry.fields.slug}
```

**app/api/draft/route.ts**
```ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'
import { getPostBySlug } from '@/lib/contentful'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')

  if (secret !== process.env.CONTENTFUL_PREVIEW_SECRET) {
    return new Response('Invalid token', { status: 401 })
  }

  if (!slug) {
    return new Response('Missing slug', { status: 400 })
  }

  // Contentful에서 포스트 확인
  const post = await getPostBySlug(slug, true) // preview 모드

  if (!post) {
    return new Response('Post not found', { status: 404 })
  }

  const draft = await draftMode()
  draft.enable()

  redirect(`/posts/${post.slug}`)
}
```

**lib/contentful.ts**
```ts
import { createClient } from 'contentful'

const client = createClient({
  space: process.env.CONTENTFUL_SPACE_ID!,
  accessToken: process.env.CONTENTFUL_ACCESS_TOKEN!,
})

const previewClient = createClient({
  space: process.env.CONTENTFUL_SPACE_ID!,
  accessToken: process.env.CONTENTFUL_PREVIEW_TOKEN!,
  host: 'preview.contentful.com',
})

export async function getPostBySlug(slug: string, preview = false) {
  const activeClient = preview ? previewClient : client

  const entries = await activeClient.getEntries({
    content_type: 'post',
    'fields.slug': slug,
    limit: 1,
  })

  return entries.items[0] || null
}
```

### Sanity 통합

**CMS 초안 URL 설정:**
```
https://your-site.com/api/draft?secret=YOUR_TOKEN&slug={slug.current}
```

**lib/sanity.ts**
```ts
import { createClient } from '@sanity/client'

const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET!,
  apiVersion: '2023-01-01',
  useCdn: false,
})

export async function getPost(slug: string, preview = false) {
  const query = `*[_type == "post" && slug.current == $slug][0]`

  return await client.fetch(query, { slug }, {
    perspective: preview ? 'previewDrafts' : 'published',
  })
}
```

### Strapi 통합

**app/api/draft/route.ts**
```ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const id = searchParams.get('id')

  if (secret !== process.env.STRAPI_PREVIEW_SECRET) {
    return new Response('Invalid token', { status: 401 })
  }

  // Strapi에서 초안 확인
  const res = await fetch(
    `${process.env.STRAPI_URL}/api/posts/${id}?publicationState=preview`,
    {
      headers: {
        Authorization: `Bearer ${process.env.STRAPI_API_TOKEN}`,
      },
    }
  )

  const post = await res.json()

  if (!post.data) {
    return new Response('Post not found', { status: 404 })
  }

  const draft = await draftMode()
  draft.enable()

  redirect(`/posts/${post.data.attributes.slug}`)
}
```

---

## Draft Mode 비활성화

초안 미리보기를 종료하려면 Draft Mode를 비활성화합니다.

**app/api/disable-draft/route.ts**
```ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  const draft = await draftMode()
  draft.disable()

  redirect('/')
}
```

### 페이지에 종료 버튼 추가

**components/ExitDraftMode.tsx**
```tsx
'use client'

export function ExitDraftMode() {
  return (
    <a
      href="/api/disable-draft"
      className="fixed bottom-4 right-4 bg-red-500 text-white px-4 py-2 rounded"
    >
      Exit Draft Mode
    </a>
  )
}
```

**app/posts/[slug]/page.tsx**
```tsx
import { draftMode } from 'next/headers'
import { ExitDraftMode } from '@/components/ExitDraftMode'

export default async function PostPage({ params }) {
  const { isEnabled } = await draftMode()
  const { slug } = await params
  const post = await getData(slug)

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
      {isEnabled && <ExitDraftMode />}
    </article>
  )
}
```

---

## 고급 사용법

### Draft Mode 상태 표시

**components/DraftBanner.tsx**
```tsx
import { draftMode } from 'next/headers'

export async function DraftBanner() {
  const { isEnabled } = await draftMode()

  if (!isEnabled) {
    return null
  }

  return (
    <div className="bg-yellow-100 border-b border-yellow-400 p-4 text-center">
      <p className="text-sm font-semibold">
        🔍 Draft Mode 활성화됨 - 초안 콘텐츠를 보고 있습니다
      </p>
      <a
        href="/api/disable-draft"
        className="text-blue-600 underline ml-4"
      >
        종료
      </a>
    </div>
  )
}
```

**app/layout.tsx**
```tsx
import { DraftBanner } from '@/components/DraftBanner'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <DraftBanner />
        {children}
      </body>
    </html>
  )
}
```

### 쿼리 파라미터 보존

리디렉션 시 추가 쿼리 파라미터 보존:

```ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')
  const locale = searchParams.get('locale') // 추가 파라미터

  // ... 검증 로직 ...

  const draft = await draftMode()
  draft.enable()

  // 쿼리 파라미터 포함하여 리디렉션
  const redirectUrl = locale
    ? `/posts/${post.slug}?locale=${locale}`
    : `/posts/${post.slug}`

  redirect(redirectUrl)
}
```

---

## API 레퍼런스

### `draftMode()`

**가져오기:**
```ts
import { draftMode } from 'next/headers'
```

**메서드:**

| 메서드 | 설명 | 반환 타입 |
|--------|------|----------|
| `enable()` | Draft Mode 활성화 (쿠키 설정) | `void` |
| `disable()` | Draft Mode 비활성화 (쿠키 삭제) | `void` |
| `isEnabled` | 현재 Draft Mode 활성화 상태 | `boolean` |

### 쿠키 세부사항

| 속성 | 값 |
|------|-----|
| **이름** | `__prerender_bypass` |
| **Path** | `/` |
| **HttpOnly** | `true` |
| **SameSite** | `None` |
| **Secure** | `true` (HTTPS 필요) |

---

## 문제 해결

### Draft Mode가 작동하지 않음

**1. HTTPS 확인**
```bash
# Draft Mode는 HTTPS가 필요합니다 (로컬 개발 제외)
# 로컬에서 테스트:
npm run dev
```

**2. 쿠키 확인**
- 브라우저 DevTools → Application → Cookies
- `__prerender_bypass` 쿠키 존재 확인

**3. 환경 변수 확인**
```bash
# .env.local에 비밀 토큰 설정되어 있는지 확인
DRAFT_SECRET_TOKEN=your-secret-token
```

### CMS 변수가 작동하지 않음

대부분의 CMS는 URL에 변수를 삽입하는 구문을 지원합니다:

```bash
# Contentful
/api/draft?secret=TOKEN&slug=/posts/{entry.fields.slug}

# Sanity
/api/draft?secret=TOKEN&slug={slug.current}

# Strapi
/api/draft?secret=TOKEN&id={id}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **비밀 토큰 사용** - 환경 변수에 저장
2. **CMS에서 slug 검증** - 오픈 리디렉션 방지
3. **Draft 배너 표시** - 사용자에게 초안 모드 알림
4. **종료 버튼 제공** - 쉽게 Draft Mode 종료
5. **HTTPS 사용** - 프로덕션 환경 필수

### ❌ 피해야 할 것

1. **쿼리 파라미터로 직접 리디렉션** - 보안 취약점
2. **비밀 토큰을 클라이언트에 노출** - 서버에서만 검증
3. **Draft Mode를 프로덕션 캐시에 의존** - 동적 렌더링 확인
4. **에러 핸들링 생략** - 적절한 상태 코드 반환

---

## 실전 예시

### 완전한 블로그 Draft Mode 구현

**app/api/draft/route.ts**
```ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'
import { getPostBySlug } from '@/lib/cms'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')

  // 검증
  if (secret !== process.env.DRAFT_SECRET_TOKEN) {
    return new Response('Invalid token', { status: 401 })
  }

  if (!slug) {
    return new Response('Missing slug', { status: 400 })
  }

  // CMS에서 포스트 가져오기
  try {
    const post = await getPostBySlug(slug, true)

    if (!post) {
      return new Response('Post not found', { status: 404 })
    }

    // Draft Mode 활성화
    const draft = await draftMode()
    draft.enable()

    // 포스트로 리디렉션
    redirect(`/posts/${post.slug}`)
  } catch (error) {
    return new Response('Internal server error', { status: 500 })
  }
}
```

---

## 다음 단계

- [draftMode API](../api-reference/functions/draftMode.md) - API 레퍼런스
- [Route Handlers](../getting-started/11-route-handlers.md) - Route Handler 가이드
- [Data Fetching](./data-fetching-patterns.md) - 데이터 페칭 패턴

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
