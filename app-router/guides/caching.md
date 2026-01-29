---
원문: https://nextjs.org/docs/app/guides/caching
버전: 16.1.6
---

# 캐싱

## 캐싱 메커니즘 개요

Next.js는 4가지 주요 캐싱 메커니즘을 구현합니다:

| 메커니즘 | 대상 | 위치 | 목적 | 지속 시간 |
|-----------|------|-------|---------|----------|
| **Request Memoization** | 함수 반환 값 | 서버 | React 컴포넌트 트리에서 데이터 재사용 | 요청별 생명주기 |
| **Data Cache** | 데이터 | 서버 | 사용자 요청 및 배포 간 데이터 저장 | 영구적 (재검증 가능) |
| **Full Route Cache** | HTML 및 RSC 페이로드 | 서버 | 렌더링 비용 감소 및 성능 향상 | 영구적 (재검증 가능) |
| **Router Cache** | RSC 페이로드 | 클라이언트 | 네비게이션 시 서버 요청 감소 | 사용자 세션 또는 시간 기반 |

---

## 1. Request Memoization

**목적**: 단일 렌더 패스 내에서 동일한 URL과 옵션을 가진 `fetch` 요청을 자동으로 중복 제거합니다.

### 작동 방식

```tsx
async function getItem() {
  // 자동으로 메모이제이션 및 캐시됨
  const res = await fetch('https://.../item/1')
  return res.json()
}

const item = await getItem() // 캐시 MISS
const item = await getItem() // 캐시 HIT (같은 렌더 패스)
```

### 주요 특징

- **지속 시간**: 렌더링이 완료될 때까지 서버 요청의 수명
- **범위**: React 컴포넌트 트리만 (Route Handlers 아님)
- **메서드**: `GET` 메서드만 메모이제이션됨; `POST` 및 `DELETE`는 안 됨
- **재검증 불필요**: 메모이제이션은 요청별이며 요청 간 공유되지 않음
- **React 기능**: Next.js 기능이 아닌 React 최적화

### 비활성화

```js
const { signal } = new AbortController()
fetch(url, { signal })
```

---

## 2. Data Cache

**목적**: 들어오는 서버 요청 및 배포 간 fetch 결과를 유지합니다.

### 기본 동작

기본적으로 `fetch` 데이터는 **자동으로 캐시되지 않습니다**. 캐싱은 렌더링 전략에 따라 다릅니다:

- **정적 렌더링**: 페칭된 데이터는 기본적으로 Data Cache에 캐시됨
- **동적 렌더링**: Fetch는 매 요청마다 실행되어 새 데이터 반환

### 설정

#### 강제 캐싱

```js
fetch(`https://...`, { cache: 'force-cache' })
```

#### 캐싱 비활성화

```js
fetch('https://api.vercel.app/blog', { cache: 'no-store' })
```

### 재검증 전략

#### 시간 기반 재검증

```js
// 최대 1시간마다 재검증 (3600초)
fetch('https://...', { next: { revalidate: 3600 } })
```

**작동 방식**:
1. 첫 요청은 소스에서 가져와 데이터 캐시
2. 기간 내 요청은 캐시된 데이터 반환
3. 기간 후, 다음 요청은 오래된 데이터를 반환하며 백그라운드 재검증 발생
4. 새 데이터가 캐시 업데이트; 재검증 실패 시 이전 데이터 유지

#### 태그를 사용한 온디맨드 재검증

```jsx
// 태그로 데이터 캐시
fetch(`https://...`, { next: { tags: ['a', 'b', 'c'] } })

// 특정 태그로 항목 재검증
revalidateTag('a')
```

사용처:
- **Route Handlers**: 서드파티 이벤트(웹훅)에 대한 응답으로 재검증
- **Server Actions**: 사용자 액션(폼 제출) 후 재검증

#### 경로를 사용한 온디맨드 재검증

```jsx
revalidatePath('/')
```

Route Handlers 또는 Server Actions에서 Data Cache 및 Full Route Cache 재검증에 사용.

### 주요 차이점: Data Cache vs Request Memoization

| 측면 | Request Memoization | Data Cache |
|--------|-------------------|-----------|
| 지속성 | 요청별 수명 | 요청 및 배포 간 |
| 범위 | React 컴포넌트 트리 | 서버 전체 |
| 재검증 | 해당 없음 | 시간 기반 또는 온디맨드 |

---

## 3. Full Route Cache

**목적**: 빌드 시점 또는 재검증 후 렌더링된 결과(HTML 및 React Server Component Payload)를 캐시합니다.

### 작동 방식

1. **React 서버 렌더링**: 서버 컴포넌트를 RSC Payload(컴팩트 바이너리 형식)로 렌더링
2. **HTML 생성**: RSC Payload 및 클라이언트 컴포넌트 지시사항에서 HTML 생성
3. **서버 캐싱**: RSC Payload 및 HTML 모두 캐시
4. **클라이언트 하이드레이션**: 브라우저가 초기 뷰를 위해 HTML 사용, 그 다음 조정을 위해 RSC Payload 사용

### 정적 vs 동적 렌더링

**정적 렌더링**:
- 빌드 시점 또는 데이터 재검증 후 라우트 렌더링
- 결과가 캐시되어 요청 간 재사용됨
- Full Route Cache에 완전히 캐시됨

**동적 렌더링**:
- 요청 시점에 라우트 렌더링
- 트리거: `cookies`, `headers`, `connection`, `draftMode`, `searchParams`, `unstable_noStore`, `cache: 'no-store'`를 사용한 `fetch`
- Full Route Cache에 캐시되지 않음
- 개별 요청에 Data Cache 여전히 사용 가능

### 지속 시간

- **기본값**: 사용자 요청 간 영구적
- **제거**: 새 배포 시

### 무효화

**옵션 1: Data Cache 재검증**
```jsx
revalidatePath('/') // Full Route Cache 무효화
revalidateTag('tag') // Full Route Cache 무효화
```

**옵션 2: 재배포**
- 배포 시 Full Route Cache 제거 (Data Cache와 달리)

### 비활성화

**동적 API 사용**:
```tsx
import { cookies } from 'next/headers'

export default function Page() {
  const cookieStore = cookies() // Full Route Cache 비활성화
  // 이제 라우트가 동적으로 렌더링됨
}
```

**Route Segment Config 사용**:
```jsx
export const dynamic = 'force-dynamic' // Full Route Cache 및 Data Cache 건너뛰기
export const revalidate = 0 // 동일한 효과
```

**하이브리드 캐싱** (동적 라우트 + 캐시된 데이터):
```jsx
export const dynamic = 'force-dynamic' // 동적 라우트

// 이 fetch는 여전히 캐시됨
fetch('https://...', { cache: 'force-cache' })

// 이 fetch는 캐시되지 않음
fetch('https://...', { cache: 'no-store' })
```

---

## 4. 클라이언트 사이드 Router Cache

**목적**: 라우트 세그먼트(레이아웃, 로딩 상태, 페이지)별로 분할된 RSC 페이로드를 저장하는 인메모리 캐시.

### 캐시되는 것

- **레이아웃**: 캐시되어 네비게이션 시 재사용됨 (부분 렌더링)
- **로딩 상태**: 즉각적인 네비게이션을 위해 캐시됨
- **페이지**: 기본적으로 캐시되지 않음 (브라우저 뒤로/앞으로에서 재사용됨)

### 지속 시간

**세션**: 캐시는 네비게이션 간 유지되지만 페이지 새로고침 시 제거됨

**자동 무효화 기간**:
- **기본 프리페칭** (`prefetch={null}` 또는 미지정):
  - 동적 페이지는 캐시되지 않음
  - 정적 페이지는 5분
- **전체 프리페칭** (`prefetch={true}`): 정적 및 동적 모두 5분

실험적 `staleTimes`로 설정:
```js
// next.config.js
export default {
  experimental: {
    staleTimes: {
      dynamic: 0,
      static: 60,
    },
  },
}
```

### 무효화

**Server Actions에서**:
```jsx
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'
import { cookies } from 'next/headers'

export async function updateAction() {
  // 특정 경로 재검증
  revalidatePath('/dashboard')

  // 태그로 재검증
  revalidateTag('posts')

  // 인증 변경을 위한 Router Cache 무효화
  cookies().set('token', newToken)
}
```

**클라이언트 사이드**:
```jsx
'use client'

import { useRouter } from 'next/navigation'

export function RefreshButton() {
  const router = useRouter()

  return (
    <button onClick={() => router.refresh()}>
      새로고침
    </button>
  )
}
```

**참고**: `router.refresh()`는 Router Cache만 제거하며 Data 또는 Full Route Cache에 영향을 주지 않습니다.

### 비활성화

```jsx
// 프리페칭 비활성화
<Link href="/page" prefetch={false}>링크</Link>

// Next.js 15부터 페이지는 기본적으로 비활성화됨
```

---

## API 참조 요약

### `fetch()` API

**기본 동작** (렌더링 전략에 따라 다름):
```jsx
// 정적 렌더링: Data Cache에 캐시됨
// 동적 렌더링: 캐시되지 않음, 매 요청마다 새 데이터
fetch('https://...')
```

**강제 캐싱**:
```jsx
fetch('https://...', { cache: 'force-cache' })
```

**캐싱 비활성화**:
```jsx
fetch('https://...', { cache: 'no-store' })
```

**시간 기반 재검증**:
```jsx
fetch('https://...', { next: { revalidate: 3600 } })
```

**태그 기반 재검증**:
```jsx
fetch('https://...', { next: { tags: ['posts', 'blog'] } })
```

### 세그먼트 설정 옵션

```jsx
// app/page.jsx
export const dynamic = 'force-dynamic' // Full Route Cache 비활성화
export const revalidate = 3600 // 매 시간 재검증
export const fetchCache = 'default-no-store' // Data Cache 비활성화

export default function Page() {
  // ...
}
```

### React `cache()` 함수

```tsx
import { cache } from 'react'
import db from '@/lib/db'

// 비 fetch 함수 메모이제이션
export const getItem = cache(async (id: string) => {
  const item = await db.item.findUnique({ id })
  return item
})

// 여러 번 호출되지만 한 번만 실행됨
const item1 = await getItem('1')
const item2 = await getItem('1') // 메모이제이션된 결과 반환
```

---

## 모범 사례

1. **기본적으로 정적 렌더링 활용**: 가장 성능이 좋은 옵션
2. **자주 업데이트되지 않는 경우 `next.revalidate` 사용**: 서버 부하 감소
3. **관련 데이터에 태그 사용**: 더 세밀한 재검증 제어
4. **정적과 동적 혼합**: 최적 성능을 위해 캐시 컴포넌트 사용
5. **필요하지 않은 경우 `cache: 'no-store'` 피하기**: 성능에 영향
6. **`router.refresh()` 신중하게 사용**: 클라이언트 사이드만, 상태 유지
7. **변경에는 Server Actions 선호**: 더 나은 캐시 무효화 제어
8. **캐싱 동작 테스트**: 적절한 캐시 히트/미스 확인

---

## 예시: 완전한 캐싱 설정

```tsx
// app/blog/[slug]/page.tsx
import { revalidateTag } from 'next/cache'

// 빌드 시점에 이 경로들 렌더링
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  }).then(res => res.json())

  return posts.map(post => ({ slug: post.slug }))
}

// 매 시간 재검증
export const revalidate = 3600

export default async function Page({ params }) {
  // 'posts' 태그로 캐시됨
  const post = await fetch(`https://api.example.com/posts/${params.slug}`, {
    next: { tags: ['posts'] }
  }).then(res => res.json())

  return <article>{post.content}</article>
}

// Server Action에서
export async function updatePost(slug: string) {
  'use server'

  // 모든 'posts' 태그된 데이터 재검증
  // Full Route Cache 및 Router Cache 무효화
  revalidateTag('posts')
}
```
