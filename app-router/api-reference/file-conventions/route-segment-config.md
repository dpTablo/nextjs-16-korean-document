---
원문: https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config
버전: 16.1.6
---

# Route Segment Config

## 개요

Route Segment Config는 특정 변수를 export하여 **Pages, Layouts, Route Handlers**의 동작을 설정할 수 있게 해줍니다.

---

## 사용 가능한 옵션

| 옵션 | 타입 | 기본값 |
|------|------|--------|
| `dynamic` | `'auto' \| 'force-dynamic' \| 'error' \| 'force-static'` | `'auto'` |
| `dynamicParams` | `boolean` | `true` |
| `revalidate` | `false \| 0 \| number` | `false` |
| `fetchCache` | `'auto' \| 'default-cache' \| 'only-cache' \| 'force-cache' \| 'force-no-store' \| 'default-no-store' \| 'only-no-store'` | `'auto'` |
| `runtime` | `'nodejs' \| 'edge'` | `'nodejs'` |
| `preferredRegion` | `'auto' \| 'global' \| 'home' \| string \| string[]` | `'auto'` |
| `maxDuration` | `number` | 배포 플랫폼에 따라 다름 |

---

## 기본 사용법

```tsx
// app/page.tsx
export const dynamic = 'force-dynamic'
export const revalidate = 60
export const runtime = 'nodejs'

export default function Page() {
  return <div>My Page</div>
}
```

---

## 주요 옵션 상세 설명

### 1. dynamic

라우트가 정적으로 렌더링될지 동적으로 렌더링될지 제어합니다.

```tsx
export const dynamic = 'auto' | 'force-dynamic' | 'error' | 'force-static'
```

#### 값 설명

| 값 | 설명 | 사용 사례 |
|------|------|----------|
| **`'auto'`** (기본값) | 동적 동작을 방지하지 않으면서 가능한 한 캐시 | 대부분의 경우 |
| **`'force-dynamic'`** | 요청 시마다 각 사용자에게 렌더링 | 사용자별 개인화 콘텐츠 |
| **`'error'`** | 정적 렌더링 강제, Dynamic API 사용 시 에러 | 정적 사이트 생성 보장 |
| **`'force-static'`** | cookie/header가 빈 값으로 정적 렌더링 강제 | 정적 최적화 |

#### 예제

**자동 (기본):**
```tsx
// app/page.tsx
export const dynamic = 'auto' // 기본값, 생략 가능

export default function Page() {
  return <div>자동 동작</div>
}
```

**동적 렌더링 강제:**
```tsx
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic'

export default async function Dashboard() {
  const session = await getSession() // 매 요청마다 실행
  return <div>안녕하세요, {session.user.name}님</div>
}
```

**정적 렌더링 강제 (에러 모드):**
```tsx
// app/blog/page.tsx
export const dynamic = 'error'

export default async function Blog() {
  // const data = await cookies() // 에러 발생!
  return <div>정적 블로그</div>
}
```

---

### 2. dynamicParams

`generateStaticParams`에 없는 동적 세그먼트의 동작을 제어합니다.

```tsx
export const dynamicParams = true | false
```

| 값 | 설명 |
|------|------|
| **`true`** (기본값) | 요청 시 생성 (온디맨드) |
| **`false`** | 404 반환 |

#### 예제

**온디맨드 생성 허용:**
```tsx
// app/blog/[slug]/page.tsx
export const dynamicParams = true // 기본값

export async function generateStaticParams() {
  return [
    { slug: 'post-1' },
    { slug: 'post-2' },
  ]
}

export default function Post({ params }: { params: { slug: string } }) {
  // post-1, post-2는 빌드 시 생성
  // post-3, post-4 등은 요청 시 생성됨
  return <div>Post: {params.slug}</div>
}
```

**404 반환:**
```tsx
// app/blog/[slug]/page.tsx
export const dynamicParams = false

export async function generateStaticParams() {
  return [
    { slug: 'post-1' },
    { slug: 'post-2' },
  ]
}

export default function Post({ params }: { params: { slug: string } }) {
  // post-1, post-2만 허용
  // post-3 접근 시 404 반환
  return <div>Post: {params.slug}</div>
}
```

---

### 3. revalidate

재검증 시간(초)을 설정합니다.

```tsx
export const revalidate = false | 0 | number
```

| 값 | 설명 | 사용 사례 |
|------|------|----------|
| **`false`** (기본값) | 무기한 캐시 | 변경되지 않는 콘텐츠 |
| **`0`** | 항상 동적 렌더링 | 실시간 데이터 |
| **`number`** | n초마다 재검증 | 주기적으로 업데이트되는 콘텐츠 |

#### 예제

**무기한 캐시:**
```tsx
// app/about/page.tsx
export const revalidate = false // 기본값

export default function About() {
  return <div>회사 소개</div>
}
```

**주기적 재검증:**
```tsx
// app/news/page.tsx
export const revalidate = 60 // 60초마다 재검증

export default async function News() {
  const news = await fetchNews()
  return (
    <div>
      {news.map(item => (
        <article key={item.id}>{item.title}</article>
      ))}
    </div>
  )
}
```

**항상 최신 데이터:**
```tsx
// app/live/page.tsx
export const revalidate = 0

export default async function Live() {
  const data = await fetchLiveData()
  return <div>실시간 데이터: {data.value}</div>
}
```

---

### 4. fetchCache

세그먼트의 모든 `fetch()` 요청에 대한 기본 캐싱 동작을 재정의합니다.

```tsx
export const fetchCache =
  | 'auto'
  | 'default-cache'
  | 'only-cache'
  | 'force-cache'
  | 'force-no-store'
  | 'default-no-store'
  | 'only-no-store'
```

| 값 | 설명 |
|------|------|
| **`'auto'`** (기본값) | 기본 fetch 캐싱 동작 |
| **`'default-cache'`** | cache 옵션 없는 fetch를 캐시 |
| **`'only-cache'`** | 모든 fetch가 캐시 옵션 필요 |
| **`'force-cache'`** | 모든 fetch를 강제 캐시 |
| **`'force-no-store'`** | 모든 fetch를 캐시 안함 |
| **`'default-no-store'`** | cache 옵션 없는 fetch를 캐시 안함 |
| **`'only-no-store'`** | 모든 fetch가 no-store 필요 |

#### 예제

**모든 fetch 캐시:**
```tsx
// app/products/page.tsx
export const fetchCache = 'force-cache'

export default async function Products() {
  // 이 fetch는 강제로 캐시됨
  const products = await fetch('https://api.example.com/products')
  return <div>제품 목록</div>
}
```

**모든 fetch 캐시 안함:**
```tsx
// app/dashboard/page.tsx
export const fetchCache = 'force-no-store'

export default async function Dashboard() {
  // 이 fetch는 캐시되지 않음
  const data = await fetch('https://api.example.com/user-data')
  return <div>대시보드</div>
}
```

---

### 5. runtime

실행 환경을 지정합니다.

```tsx
export const runtime = 'nodejs' | 'edge'
```

| 값 | 설명 | 권장 사용 |
|------|------|----------|
| **`'nodejs'`** (기본값) | Node.js 런타임 | 대부분의 경우, 모든 Node.js API 사용 가능 |
| **`'edge'`** | Edge 런타임 | 빠른 응답 필요, 제한된 API |

#### 예제

**Node.js 런타임 (기본):**
```tsx
// app/api/process/route.ts
export const runtime = 'nodejs'

export async function POST(request: Request) {
  // Node.js API 사용 가능
  const fs = require('fs')
  return Response.json({ success: true })
}
```

**Edge 런타임:**
```tsx
// app/api/geo/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  // 빠른 응답, 제한된 API
  return Response.json({ location: 'edge' })
}
```

---

### 6. preferredRegion

배포 리전 선호도를 지정합니다.

```tsx
export const preferredRegion = 'auto' | 'global' | 'home' | string | string[]
```

| 값 | 설명 |
|------|------|
| **`'auto'`** (기본값) | 자동 선택 |
| **`'global'`** | 모든 리전 |
| **`'home'`** | 홈 리전 |
| **`string`** | 특정 리전 (예: `'iad1'`) |
| **`string[]`** | 여러 리전 |

#### 예제

**특정 리전:**
```tsx
// app/api/route.ts
export const runtime = 'edge'
export const preferredRegion = 'iad1' // 버지니아

export async function GET() {
  return Response.json({ region: 'iad1' })
}
```

**여러 리전:**
```tsx
// app/api/global/route.ts
export const runtime = 'edge'
export const preferredRegion = ['iad1', 'sfo1', 'hnd1']

export async function GET() {
  return Response.json({ regions: 'multiple' })
}
```

---

### 7. maxDuration

서버 사이드 로직의 최대 실행 시간(초)을 설정합니다.

```tsx
export const maxDuration = number // 초 단위
```

#### 예제

**긴 작업:**
```tsx
// app/api/process/route.ts
export const maxDuration = 300 // 5분

export async function POST(request: Request) {
  // 긴 처리 작업
  await longRunningTask()
  return Response.json({ done: true })
}
```

**빠른 응답:**
```tsx
// app/api/quick/route.ts
export const maxDuration = 5 // 5초

export async function GET() {
  const data = await quickFetch()
  return Response.json(data)
}
```

---

## 실용적인 조합 예제

### 1. 정적 블로그

```tsx
// app/blog/[slug]/page.tsx
export const dynamic = 'error' // 정적만 허용
export const revalidate = false // 무기한 캐시
export const runtime = 'nodejs'

export async function generateStaticParams() {
  const posts = await getAllPosts()
  return posts.map(post => ({ slug: post.slug }))
}

export default function Post({ params }: { params: { slug: string } }) {
  return <article>블로그 포스트</article>
}
```

### 2. ISR (Incremental Static Regeneration)

```tsx
// app/products/page.tsx
export const revalidate = 3600 // 1시간마다 재검증
export const fetchCache = 'force-cache' // fetch 결과 캐시

export default async function Products() {
  const products = await fetch('https://api.example.com/products')
  return <div>제품 목록</div>
}
```

### 3. 완전 동적 대시보드

```tsx
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic' // 항상 동적
export const revalidate = 0 // 캐시 안함
export const fetchCache = 'force-no-store' // fetch도 캐시 안함

export default async function Dashboard() {
  const userData = await fetch('https://api.example.com/user')
  return <div>대시보드</div>
}
```

### 4. Edge 함수 API

```tsx
// app/api/edge/route.ts
export const runtime = 'edge'
export const preferredRegion = ['iad1', 'sfo1']
export const maxDuration = 10

export async function GET(request: Request) {
  const geo = request.headers.get('x-vercel-ip-country')
  return Response.json({ country: geo })
}
```

### 5. 보호된 정적 페이지

```tsx
// app/docs/page.tsx
export const dynamic = 'error' // Dynamic API 사용 시 에러
export const dynamicParams = false // 정의된 경로만 허용
export const revalidate = false // 캐시

export default function Docs() {
  return <div>문서</div>
}
```

---

## 중요한 제약사항

### ⚠️ Server Component에서만 작동

```tsx
// ✅ 작동함 (Server Component)
export const dynamic = 'force-dynamic'

export default function Page() {
  return <div>Server Component</div>
}
```

```tsx
// ❌ 작동하지 않음 (Client Component)
'use client'

export const dynamic = 'force-dynamic' // 무시됨

export default function Page() {
  return <div>Client Component</div>
}
```

### ⚠️ 개발 환경에서의 동작

개발 환경(`npm run dev`)에서는:
- 페이지가 항상 온디맨드로 렌더링됩니다
- 캐시되지 않습니다
- `revalidate` 설정이 무시됩니다

프로덕션 빌드(`npm run build`)에서만 정확한 동작을 확인할 수 있습니다.

### ⚠️ generateStaticParams 제약

```tsx
// ❌ Client Component에서 사용 불가
'use client'

export async function generateStaticParams() { // 에러!
  return []
}
```

---

## 옵션 조합 가이드

### 완전 정적 (SSG)

```tsx
export const dynamic = 'error'
export const revalidate = false
```

### ISR (Incremental Static Regeneration)

```tsx
export const revalidate = 60 // 60초마다
```

### 완전 동적 (SSR)

```tsx
export const dynamic = 'force-dynamic'
export const revalidate = 0
```

### Edge 최적화

```tsx
export const runtime = 'edge'
export const preferredRegion = 'auto'
```

---

## 옵션 우선순위

1. **`dynamic = 'error'` 또는 `'force-static'`** → 정적 렌더링 강제
2. **`dynamic = 'force-dynamic'` 또는 `revalidate = 0`** → 동적 렌더링
3. **`revalidate`** → ISR 설정
4. **`fetchCache`** → fetch 동작 재정의
5. **개별 `fetch()` 옵션** → 가장 높은 우선순위

---

## 디버깅 팁

### 렌더링 모드 확인

```tsx
// app/page.tsx
export const dynamic = 'force-dynamic'

export default function Page() {
  console.log('렌더링 시간:', new Date().toISOString())
  return <div>페이지</div>
}
```

프로덕션에서:
- 정적: 빌드 시 한 번만 로그
- 동적: 매 요청마다 로그

### Next.js 빌드 출력 확인

```bash
npm run build
```

출력 예제:
```
○ (Static)   - 정적 페이지
λ (Dynamic)  - 동적 페이지 (SSR)
◐ (ISR)      - ISR 페이지
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **기본값 사용**
   ```tsx
   // 특별한 이유가 없다면 기본값 사용
   export default function Page() {
     return <div>콘텐츠</div>
   }
   ```

2. **명시적 설정**
   ```tsx
   // 의도가 명확하면 명시적으로 설정
   export const dynamic = 'force-dynamic'
   export const revalidate = 3600
   ```

3. **ISR 활용**
   ```tsx
   // 자주 변경되지 않는 데이터는 ISR
   export const revalidate = 3600 // 1시간
   ```

### ❌ 피해야 할 사항

1. **불필요한 동적 렌더링**
   ```tsx
   // ❌ 정적 콘텐츠인데 동적 설정
   export const dynamic = 'force-dynamic'

   export default function Static() {
     return <div>정적 콘텐츠</div>
   }
   ```

2. **과도한 재검증**
   ```tsx
   // ❌ 너무 자주 재검증
   export const revalidate = 1 // 1초마다
   ```

3. **Client Component에서 설정**
   ```tsx
   // ❌ 작동하지 않음
   'use client'
   export const dynamic = 'force-dynamic'
   ```

---

## 타입 정의

```typescript
type RouteSegmentConfig = {
  dynamic?: 'auto' | 'force-dynamic' | 'error' | 'force-static'
  dynamicParams?: boolean
  revalidate?: false | 0 | number
  fetchCache?:
    | 'auto'
    | 'default-cache'
    | 'only-cache'
    | 'force-cache'
    | 'force-no-store'
    | 'default-no-store'
    | 'only-no-store'
  runtime?: 'nodejs' | 'edge'
  preferredRegion?: 'auto' | 'global' | 'home' | string | string[]
  maxDuration?: number
}
```

---

## 관련 API

- [`generateStaticParams`](../functions/generateStaticParams.md) - 정적 경로 생성
- [`revalidatePath`](../functions/revalidatePath.md) - 경로 재검증
- [`revalidateTag`](../functions/revalidateTag.md) - 태그 재검증
- [`fetch`](../functions/fetch.md) - Next.js 확장 fetch API

---

## 참고 자료

- [Rendering Strategies](../../guides/rendering.md) - 렌더링 전략 가이드
- [Data Fetching](../../guides/data-fetching-patterns.md) - 데이터 페칭 패턴
- [Caching](../../guides/caching.md) - 캐싱 가이드

---

## 버전 정보

- **도입 버전:** Next.js 13.0.0
- **maxDuration 추가:** Next.js 13.4.10
- **현재 상태:** Stable

---

## 요약

- **위치:** Pages, Layouts, Route Handlers에서 export
- **주요 옵션:** `dynamic`, `revalidate`, `runtime`
- **렌더링 제어:** 정적/동적/ISR 선택
- **캐싱 제어:** `revalidate`, `fetchCache`
- **런타임 선택:** Node.js vs Edge
- **제약:** Server Component에서만 작동
- **개발 환경:** 설정이 무시됨 (프로덕션에서만 적용)
