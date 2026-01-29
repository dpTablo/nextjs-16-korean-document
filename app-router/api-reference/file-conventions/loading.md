---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/loading
버전: 16.1.6
---

# loading.js

`loading.js`는 [React Suspense](https://react.dev/reference/react/Suspense)를 통해 의미 있는 로딩 UI를 만들기 위한 특수 파일입니다. 라우트 세그먼트의 콘텐츠가 로딩되는 동안 서버에서 즉시 로딩 상태를 표시할 수 있으며, 렌더링이 완료되면 새 콘텐츠가 자동으로 교체됩니다.

---

## 기본 사용법

```tsx
// app/feed/loading.tsx
export default function Loading() {
  return <p>Loading...</p>
}
```

**특징:**
- 기본적으로 **Server Component**
- `"use client"` 지시어로 Client Component로도 사용 가능
- 매개변수를 받지 않음

---

## 주요 동작 방식

### 즉시 로딩 상태 (Instant Loading States)

`loading.js` 파일을 추가하면 네비게이션 시 즉시 로딩 UI가 표시됩니다.

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <LoadingSkeleton />
}
```

**작동 원리:**
1. `loading.js`는 같은 폴더의 `layout.js` 내에 중첩됩니다
2. `page.js` 파일과 그 아래의 모든 자식을 자동으로 `<Suspense>` 경계로 래핑합니다
3. 페이지 콘텐츠가 로딩되는 동안 로딩 UI를 표시합니다

### 네비게이션 특징

- ✅ **프리페치**: 폴백 UI가 미리 가져와져서 즉시 네비게이션 제공
- ✅ **중단 가능**: 새 라우트로 이동할 때 콘텐츠 로딩 완료를 기다릴 필요 없음
- ✅ **상호작용 유지**: 공유 레이아웃은 새 라우트 세그먼트 로딩 중에도 상호작용 가능

---

## 사용 예제

### 기본 로딩 스피너

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="flex items-center justify-center h-screen">
      <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-gray-900" />
    </div>
  )
}
```

### 스켈레톤 UI

```tsx
// app/posts/loading.tsx
export default function Loading() {
  return (
    <div className="space-y-4">
      <div className="h-8 bg-gray-200 rounded animate-pulse" />
      <div className="h-4 bg-gray-200 rounded animate-pulse w-3/4" />
      <div className="h-4 bg-gray-200 rounded animate-pulse w-1/2" />
    </div>
  )
}
```

### 커스텀 로딩 컴포넌트

```tsx
// app/products/loading.tsx
import { Spinner } from '@/components/ui/spinner'

export default function Loading() {
  return (
    <div className="container mx-auto p-4">
      <h2 className="text-xl font-bold mb-4">제품 로딩 중...</h2>
      <Spinner />
    </div>
  )
}
```

---

## 수동 Suspense 경계

더 세밀한 제어가 필요한 경우 `<Suspense>`를 수동으로 사용할 수 있습니다.

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react'
import { PostFeed, Weather } from './components'

export default function Dashboard() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
    </section>
  )
}
```

**장점:**
- 컴포넌트별로 개별 로딩 상태 제공
- 더 나은 사용자 경험
- 부분적인 페이지 렌더링

---

## 파일 구조 예제

```
app/
├── layout.tsx
├── page.tsx
├── loading.tsx          # 루트 로딩
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    └── loading.tsx      # 대시보드 로딩
```

**작동 방식:**

```tsx
// 내부적으로 다음과 같이 작동
<Layout>
  <Suspense fallback={<Loading />}>
    <Page />
  </Suspense>
</Layout>
```

---

## SEO 고려사항

### 메타데이터 처리

`generateMetadata`는 스트리밍 전에 해결되어 초기 HTML의 `<head>`에 포함됩니다.

```tsx
// app/posts/[id]/page.tsx
import { Metadata } from 'next'

export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.id)
  return {
    title: post.title,
    description: post.excerpt,
  }
}

export default async function Post({ params }) {
  const post = await getPost(params.id)
  return <article>{post.content}</article>
}
```

### 봇 크롤링

JavaScript를 실행할 수 없는 봇(예: Twitterbot, 검색 엔진 크롤러)의 경우:
- Next.js는 `generateMetadata`를 먼저 해결
- 메타데이터가 초기 HTML에 포함됨
- 스트리밍 시 `200` 상태 코드 반환 (성공 신호)

---

## 상태 코드 처리

스트리밍이 시작된 후에는 HTTP 상태 코드를 변경할 수 없습니다 (응답 헤더가 이미 전송됨).

**대안:**
- [`redirect()`](../functions/redirect.md) 사용
- [`notFound()`](../functions/notFound.md) 사용

```tsx
// app/posts/[id]/page.tsx
import { notFound } from 'next/navigation'

export default async function Post({ params }) {
  const post = await getPost(params.id)

  if (!post) {
    notFound() // 404 페이지로 리디렉션
  }

  return <article>{post.content}</article>
}
```

---

## 플랫폼 지원

| 배포 옵션 | 지원 여부 |
|---------|---------|
| Node.js 서버 | ✅ 지원 |
| Docker 컨테이너 | ✅ 지원 |
| 정적 내보내기 | ❌ 미지원 |
| Adapters | 플랫폼별로 다름 |

---

## 모범 사례

### 1. 의미 있는 로딩 UI 제공

```tsx
// ❌ 나쁜 예
export default function Loading() {
  return <div>Loading...</div>
}

// ✅ 좋은 예
export default function Loading() {
  return (
    <div>
      <h2>제품 로딩 중</h2>
      <ProductListSkeleton />
    </div>
  )
}
```

### 2. 스켈레톤 UI 사용

실제 콘텐츠 레이아웃과 유사한 스켈레톤을 제공하세요.

```tsx
export default function Loading() {
  return (
    <div className="grid grid-cols-3 gap-4">
      {[...Array(6)].map((_, i) => (
        <div key={i} className="border rounded p-4">
          <div className="h-48 bg-gray-200 rounded animate-pulse mb-4" />
          <div className="h-4 bg-gray-200 rounded animate-pulse mb-2" />
          <div className="h-4 bg-gray-200 rounded animate-pulse w-2/3" />
        </div>
      ))}
    </div>
  )
}
```

### 3. 세밀한 Suspense 경계

페이지 전체보다는 개별 컴포넌트에 Suspense를 사용하세요.

---

## 중요한 주의사항

> **Good to know**:
> * `loading.js`는 자동으로 `<Suspense>` 경계를 생성합니다
> * 더 세밀한 제어가 필요하면 수동으로 `<Suspense>`를 사용하세요
> * 정적 내보내기에서는 작동하지 않습니다
> * 스트리밍 시작 후 HTTP 상태 코드 변경 불가능

> **성능 팁**:
> * 스켈레톤 UI는 실제 콘텐츠와 레이아웃이 유사해야 합니다
> * 로딩 상태는 간결하고 빠르게 렌더링되어야 합니다
> * 너무 많은 애니메이션은 피하세요

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v13.0.0 | `loading.js` 도입 |

---

## 관련 문서

- [error.js](./error.md)
- [layout.js](./layout.md)
- [Suspense (React)](https://react.dev/reference/react/Suspense)
- [Loading UI and Streaming](../../getting-started/03-layouts-and-pages.md)
