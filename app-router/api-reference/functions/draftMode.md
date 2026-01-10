# draftMode

`draftMode`는 **비동기 함수**로, [Draft Mode](../../guides/draft-mode.md)를 활성화/비활성화하고 Server Component에서 Draft Mode 상태를 확인할 수 있습니다.

---

## 기본 사용법

```tsx
import { draftMode } from 'next/headers'

export default async function Page() {
  const { isEnabled } = await draftMode()

  return (
    <div>
      <p>Draft Mode: {isEnabled ? 'Enabled' : 'Disabled'}</p>
    </div>
  )
}
```

---

## API 참조

### isEnabled

Draft Mode 활성화 여부를 나타내는 boolean 값입니다.

```tsx
const { isEnabled } = await draftMode()
```

### enable()

Route Handler에서 Draft Mode를 활성화합니다 (쿠키 설정).

```tsx
const draft = await draftMode()
draft.enable()
```

### disable()

Route Handler에서 Draft Mode를 비활성화합니다 (쿠키 삭제).

```tsx
const draft = await draftMode()
draft.disable()
```

---

## 사용 예제

### Draft Mode 활성화

```tsx
// app/api/draft/route.ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  // 쿼리 문자열 파라미터에서 secret과 slug 파싱
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')

  // secret 및 다음 파라미터 확인
  // 이 secret은 이 Route Handler와 CMS에만 알려져야 함
  if (secret !== process.env.DRAFT_SECRET_TOKEN) {
    return new Response('Invalid token', { status: 401 })
  }

  // Draft Mode 활성화
  const draft = await draftMode()
  draft.enable()

  // 지정된 경로로 리디렉션
  redirect(slug || '/')
}
```

### Draft Mode 비활성화

```tsx
// app/api/disable-draft/route.ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  const draft = await draftMode()
  draft.disable()

  redirect('/')
}
```

> **중요**: `<Link>` 컴포넌트로 호출 시 `prefetch={false}` 필수

```tsx
// app/components/exit-draft-mode.tsx
import Link from 'next/link'

export function ExitDraftMode() {
  return (
    <Link href="/api/disable-draft" prefetch={false}>
      Exit Draft Mode
    </Link>
  )
}
```

### Draft Mode 상태 확인

```tsx
// app/page.tsx
import { draftMode } from 'next/headers'

export default async function Page() {
  const { isEnabled } = await draftMode()

  return (
    <main>
      <h1>My Blog Post</h1>
      <p>Draft Mode is currently {isEnabled ? 'Enabled' : 'Disabled'}</p>
    </main>
  )
}
```

### CMS에서 Draft 콘텐츠 가져오기

```tsx
// app/blog/[slug]/page.tsx
import { draftMode } from 'next/headers'

async function getPost(slug: string, isDraft: boolean) {
  const url = isDraft
    ? `https://cms.example.com/api/draft/${slug}`
    : `https://cms.example.com/api/posts/${slug}`

  const res = await fetch(url)
  return res.json()
}

export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const { isEnabled } = await draftMode()
  const post = await getPost(slug, isEnabled)

  return (
    <article>
      <h1>{post.title}</h1>
      {isEnabled && <p className="draft-badge">Draft Preview</p>}
      <div>{post.content}</div>
    </article>
  )
}
```

### Draft Mode 토글 버튼

```tsx
// app/components/draft-mode-toggle.tsx
import { draftMode } from 'next/headers'
import Link from 'next/link'

export async function DraftModeToggle() {
  const { isEnabled } = await draftMode()

  if (isEnabled) {
    return (
      <Link href="/api/disable-draft" prefetch={false}>
        Exit Draft Mode
      </Link>
    )
  }

  return (
    <Link href="/api/draft?secret=MY_SECRET">
      Enable Draft Mode
    </Link>
  )
}
```

---

## Draft Mode 작동 원리

1. **쿠키 기반**: Draft Mode는 쿠키를 사용하여 상태를 저장합니다
2. **페이지별**: 각 페이지에서 개별적으로 Draft Mode 확인 가능
3. **보안**: 매번 `next build` 실행 시 새로운 bypass 쿠키값 생성
4. **프리렌더링 방지**: Draft Mode가 활성화되면 페이지가 동적으로 렌더링됨

---

## 중요한 주의사항

> **Good to know**:
> * **비동기 함수**: `async/await` 또는 React의 [`use`](https://react.dev/reference/react/use) 함수 필수
> * **보안**: CMS와 공유하는 secret 토큰을 사용하여 무단 접근 방지
> * **쿠키 생성**: 매 빌드마다 새로운 bypass 쿠키값이 생성됩니다
> * **프리페치 비활성화**: Draft Mode 비활성화 링크는 `prefetch={false}` 설정 필요
> * **로컬 테스트**: HTTP 환경에서는 브라우저 설정에서 써드파티 쿠키 및 로컬 스토리지 접근 허용 필요

> **호환성**:
> * v15.0.0-RC 이전: 동기 함수였으나 비동기 함수로 변경
> * 코드모드(codemod) 제공으로 마이그레이션 지원
> * v14 이전 버전에서는 여전히 동기식 접근 가능 (향후 deprecated 예정)

---

## 보안 모범 사례

### 1. 강력한 Secret 사용

```tsx
// ❌ 나쁜 예
const secret = 'my-secret'

// ✅ 좋은 예
const secret = process.env.DRAFT_SECRET_TOKEN // 복잡한 랜덤 문자열
```

### 2. Secret 검증

```tsx
// app/api/draft/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')

  // Secret 검증
  if (!secret || secret !== process.env.DRAFT_SECRET_TOKEN) {
    return new Response('Invalid token', { status: 401 })
  }

  // Draft Mode 활성화
  const draft = await draftMode()
  draft.enable()

  redirect(searchParams.get('slug') || '/')
}
```

### 3. 타임아웃 설정

```tsx
// Draft Mode 세션에 타임아웃 추가 (선택사항)
const DRAFT_TIMEOUT = 60 * 60 * 1000 // 1시간

export async function GET(request: Request) {
  const draft = await draftMode()
  draft.enable()

  // 타임아웃 후 자동 비활성화
  setTimeout(async () => {
    draft.disable()
  }, DRAFT_TIMEOUT)

  redirect('/')
}
```

---

## CMS와 통합

### Contentful 예제

```tsx
// app/api/draft/route.ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)

  // Contentful의 secret과 slug
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')

  if (secret !== process.env.CONTENTFUL_PREVIEW_SECRET) {
    return new Response('Invalid token', { status: 401 })
  }

  const draft = await draftMode()
  draft.enable()

  redirect(`/blog/${slug}`)
}
```

### Sanity CMS 예제

```tsx
// app/api/draft/route.ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'
import { validatePreviewUrl } from '@sanity/preview-url-secret'
import { client } from '@/sanity/lib/client'

export async function GET(request: Request) {
  const { isValid, redirectTo = '/' } = await validatePreviewUrl(
    client.withConfig({ useCdn: false }),
    request.url
  )

  if (!isValid) {
    return new Response('Invalid secret', { status: 401 })
  }

  const draft = await draftMode()
  draft.enable()

  redirect(redirectTo)
}
```

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.0.0-RC | 비동기 함수로 변경 (codemod 제공) |
| v13.4.0 | `draftMode` 도입 |

---

## 관련 문서

- [Draft Mode Guide](../../guides/draft-mode.md)
- [Server Components](../../getting-started/05-server-and-client-components.md)
- [cookies](./cookies.md)
- [headers](./headers.md)
