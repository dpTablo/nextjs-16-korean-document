---
원문: https://nextjs.org/docs/app/guides/rendering
버전: 16.1.6
---

# 렌더링 전략

## 개요
Next.js의 React Server Components는 서버에서 UI를 렌더링하고 선택적으로 캐싱할 수 있으며, 라우트 세그먼트별로 렌더링 작업을 분할하여 스트리밍 및 부분 렌더링을 가능하게 합니다.

## 세 가지 서버 렌더링 전략
1. **정적 렌더링 (Static Rendering)** (기본값)
2. **동적 렌더링 (Dynamic Rendering)**
3. **스트리밍 (Streaming)**

---

## 서버 렌더링의 이점

### 데이터 페칭
- 데이터 소스에 더 가깝게 데이터 페칭 이동
- 페칭 시간과 클라이언트 요청 수 감소

### 보안
- 민감한 데이터, 토큰 및 API 키를 서버에 유지
- 클라이언트 측 코드에 노출 방지

### 캐싱
- 렌더링된 결과를 캐시하여 요청 및 사용자 간 재사용
- 성능 향상 및 비용 절감

### 번들 크기
- 큰 의존성을 서버에 유지
- 클라이언트 JavaScript 번들 크기 감소
- 느린 인터넷이나 덜 강력한 기기를 사용하는 사용자에게 유리

### 초기 페이지 로드 및 FCP
- 서버에서 HTML 생성하여 즉시 볼 수 있음
- JavaScript 다운로드/파싱/실행을 기다리지 않고 페이지 확인

### SEO 및 소셜 공유성
- 검색 엔진 인덱싱을 위해 렌더링된 HTML 제공
- 소셜 네트워크 봇이 미리보기 카드 생성 가능

### 스트리밍
- 렌더링을 청크로 분할하여 준비되는 대로 전달
- 전체 서버 렌더링을 기다리지 않고 페이지 일부를 먼저 확인

---

## Server Components 렌더링 방식

### 서버에서 (2단계):
1. React가 Server Components를 **React Server Component Payload (RSC Payload)**로 렌더링
2. Next.js가 RSC Payload + Client Component JavaScript를 사용하여 **HTML** 렌더링

### 클라이언트에서 (3단계):
1. HTML이 빠른 비대화형 미리보기 표시 (초기 페이지 로드만)
2. RSC Payload가 Client 및 Server Component 트리를 조정하고 DOM 업데이트
3. JavaScript가 Client Components를 하이드레이트하여 상호작용 가능하게 함

### React Server Component Payload (RSC)
다음을 포함하는 컴팩트한 바이너리 표현:
- Server Components의 렌더링된 결과
- Client Components용 자리 표시자 및 JavaScript 파일 참조
- Server에서 Client Components로 전달되는 Props

---

## 정적 렌더링 (기본값)

**시기**: **빌드 타임** 또는 데이터 재검증 후 라우트 렌더링
**캐싱**: 결과가 캐시되고 CDN으로 푸시됨
**최적**: 빌드 시점에 알려진 개인화되지 않은 데이터 (블로그 포스트, 제품 페이지)

### 예제

```tsx
// app/blog/page.tsx
export default async function BlogPage() {
  // 빌드 시 한 번 페칭되고 캐시됨
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache'
  }).then(res => res.json())

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  )
}
```

---

## 동적 렌더링

**시기**: 각 사용자에 대해 **요청 타임**에 라우트 렌더링
**최적**: 개인화된 사용자 데이터 또는 요청 시점 정보 (쿠키, 검색 매개변수)

### 캐시된 데이터가 있는 동적 라우트
라우트는 **캐시된 데이터와 캐시되지 않은 데이터**를 동시에 사용할 수 있습니다:
- RSC Payload와 데이터가 별도로 캐시됨
- 요청 시점 성능 페널티 없이 동적 렌더링 가능

### 동적 렌더링으로 전환

Next.js는 다음 경우 자동으로 동적 렌더링으로 전환합니다:

| 동적 함수 | 데이터       | 결과               |
|-----------|------------|-------------------|
| 아니오    | 캐시됨      | 정적 렌더링        |
| 예        | 캐시됨      | 동적 렌더링        |
| 아니오    | 캐시 안됨   | 동적 렌더링        |
| 예        | 캐시 안됨   | 동적 렌더링        |

### 동적 함수
동적 렌더링을 요구하는 요청 시점 정보:
- **`cookies()`** 및 **`headers()`**: 전체 라우트를 동적 렌더링으로 전환
- **`searchParams`**: 페이지 prop이 페이지를 동적 렌더링으로 전환

### 예제

```tsx
// app/dashboard/page.tsx
import { cookies } from 'next/headers'

export default async function Dashboard() {
  // 쿠키 사용 → 동적 렌더링
  const cookieStore = await cookies()
  const theme = cookieStore.get('theme')

  return (
    <div data-theme={theme?.value}>
      <h1>대시보드</h1>
      {/* 사용자별 콘텐츠 */}
    </div>
  )
}
```

```tsx
// app/search/page.tsx
export default function SearchPage({
  searchParams,
}: {
  searchParams: { query?: string }
}) {
  // searchParams 사용 → 동적 렌더링
  return (
    <div>
      <h1>검색 결과: {searchParams.query}</h1>
    </div>
  )
}
```

---

## 스트리밍

**목적**: 청크가 준비되는 대로 서버에서 UI를 점진적으로 렌더링

**이점**:
- 사용자가 페이지 일부를 즉시 확인
- 초기 페이지 로드 성능 향상
- 전체 라우트 렌더링을 차단하는 느린 데이터 페칭 처리
- Next.js App Router에 기본으로 내장

### `loading.js`로 구현

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>대시보드 로딩 중...</div>
}
```

```tsx
// app/dashboard/page.tsx
export default async function Dashboard() {
  const data = await fetchDashboardData() // 느린 쿼리
  return <DashboardContent data={data} />
}
```

### `<Suspense>`로 구현

```tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      {/* 즉시 표시 */}
      <Header />

      {/* 스트리밍됨 */}
      <Suspense fallback={<div>통계 로딩 중...</div>}>
        <Stats />
      </Suspense>

      <Suspense fallback={<div>차트 로딩 중...</div>}>
        <Charts />
      </Suspense>
    </div>
  )
}
```

---

## 렌더링 전략 선택

### 정적 렌더링 사용 시기
- ✅ 모든 사용자에게 동일한 콘텐츠
- ✅ 빌드 시점에 데이터를 알 수 있음
- ✅ 데이터가 자주 변경되지 않음
- ✅ SEO가 중요함

**예제:**
- 블로그 포스트
- 제품 페이지
- 마케팅 페이지
- 문서

```tsx
// app/products/[id]/page.tsx
export async function generateStaticParams() {
  const products = await getProducts()
  return products.map(product => ({ id: product.id }))
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id)
  return <ProductDetails product={product} />
}
```

### 동적 렌더링 사용 시기
- ✅ 사용자별 개인화된 콘텐츠
- ✅ 요청 시점에만 알 수 있는 데이터
- ✅ 쿠키나 헤더에 의존
- ✅ 실시간 데이터

**예제:**
- 사용자 대시보드
- 쇼핑 카트
- 검색 결과
- 개인화된 추천

```tsx
// app/cart/page.tsx
import { cookies } from 'next/headers'

export default async function CartPage() {
  const cookieStore = await cookies()
  const userId = cookieStore.get('userId')?.value

  const cart = await getCart(userId)

  return <Cart items={cart.items} />
}
```

### 스트리밍 사용 시기
- ✅ 일부 데이터 페칭이 느림
- ✅ 빠른 초기 렌더링이 중요함
- ✅ UI를 부분적으로 표시 가능
- ✅ 사용자 경험 개선이 우선

**예제:**
- 복잡한 대시보드
- 여러 데이터 소스가 있는 페이지
- 느린 API를 포함한 페이지

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react'

export default function Dashboard() {
  return (
    <>
      {/* 빠른 데이터 - 즉시 */}
      <QuickStats />

      {/* 느린 데이터 - 스트리밍 */}
      <Suspense fallback={<ChartsSkeleton />}>
        <AnalyticsCharts />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <RecentActivities />
      </Suspense>
    </>
  )
}
```

---

## 하이브리드 렌더링

### 정적 + 동적 조합

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  // 정적: 빌드 시 경로 생성
  const posts = await getPosts()
  return posts.map(post => ({ slug: post.slug }))
}

export default async function BlogPost({ params, searchParams }) {
  // 정적: 포스트 콘텐츠
  const post = await getPost(params.slug)

  // 동적: 댓글 (사용자별로 다를 수 있음)
  const showComments = searchParams.comments === 'true'

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>

      {showComments && (
        <Suspense fallback={<div>댓글 로딩 중...</div>}>
          <Comments postId={post.id} />
        </Suspense>
      )}
    </article>
  )
}
```

### 점진적 정적 재생성 (ISR)

```tsx
// app/news/page.tsx
export default async function NewsPage() {
  // 60초마다 재검증
  const news = await fetch('https://api.example.com/news', {
    next: { revalidate: 60 }
  }).then(res => res.json())

  return <NewsList news={news} />
}
```

---

## 모범 사례

### 1. 가능한 한 정적 렌더링 사용

```tsx
// ✅ 좋은 예 - 정적 렌더링
export default async function Page() {
  const data = await fetch('https://api.example.com/data', {
    cache: 'force-cache'
  })
  return <DataDisplay data={data} />
}
```

### 2. 필요한 부분만 동적으로

```tsx
// ✅ 좋은 예 - 세분화된 동적 렌더링
export default function Page() {
  return (
    <>
      <StaticContent /> {/* 정적 */}
      <Suspense fallback={<Loading />}>
        <DynamicContent /> {/* 동적 */}
      </Suspense>
    </>
  )
}
```

### 3. 스트리밍으로 UX 개선

```tsx
// ✅ 좋은 예 - 점진적 로딩
export default function Page() {
  return (
    <>
      <Header />
      <Suspense fallback={<Skeleton />}>
        <SlowComponent />
      </Suspense>
    </>
  )
}
```

### 4. 적절한 재검증 간격

```tsx
// ✅ 좋은 예 - 데이터 특성에 맞는 재검증
export default async function Page() {
  // 자주 변경되는 데이터 - 짧은 재검증
  const news = await fetch('...', { next: { revalidate: 60 } })

  // 거의 변경되지 않는 데이터 - 긴 재검증
  const settings = await fetch('...', { next: { revalidate: 3600 } })

  return <Content news={news} settings={settings} />
}
```

---

## 성능 최적화

### 1. 병렬 데이터 페칭

```tsx
export default async function Page() {
  // 병렬로 페칭
  const [users, posts] = await Promise.all([
    getUsers(),
    getPosts()
  ])

  return <Dashboard users={users} posts={posts} />
}
```

### 2. 선택적 하이드레이션

```tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <>
      {/* 중요한 콘텐츠 - 먼저 하이드레이트 */}
      <MainContent />

      {/* 덜 중요한 콘텐츠 - 나중에 하이드레이트 */}
      <Suspense fallback={null}>
        <SecondaryContent />
      </Suspense>
    </>
  )
}
```

### 3. 캐싱 전략

```tsx
import { cache } from 'react'

// 요청 내에서 중복 제거
export const getUser = cache(async (id: string) => {
  return await db.user.findUnique({ where: { id } })
})
```

---

## 기본 설정
- Next.js는 기본적으로 Server Components 사용
- 추가 구성 없이 자동 서버 렌더링
- 필요한 경우에만 Client Components 선택

---

## 관련 문서

- [Server and Client Components](../getting-started/05-server-and-client-components.md)
- [Loading UI](../api-reference/file-conventions/loading.md)
- [Data Fetching Patterns](./data-fetching-patterns.md)
- [Caching](./caching.md)

## Sources

- [Rendering: Server Components | Next.js](https://nextjs.org/docs/14/app/building-your-application/rendering/server-components)
- [Getting Started: Server and Client Components | Next.js](https://nextjs.org/docs/app/getting-started/server-and-client-components)
