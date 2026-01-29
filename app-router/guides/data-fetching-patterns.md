---
원문: https://nextjs.org/docs/app/guides/data-fetching
버전: 16.1.6
---

# 데이터 페칭 패턴

## 개요
이 문서는 Next.js Server 및 Client Components에서 데이터를 페칭하는 방법, 스트리밍, 캐싱 및 최적의 성능을 위한 모범 사례를 다룹니다.

---

## Server Components

### `fetch` API로 데이터 페칭

Server Components는 비동기이므로 직접 데이터를 페칭할 수 있습니다:

```tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

**주요 포인트:**
- `fetch` 응답은 기본적으로 캐시되지 않지만, Next.js는 성능 향상을 위해 라우트를 사전 렌더링합니다
- 동적 렌더링을 선택하려면 `{ cache: 'no-store' }` 사용
- 개발 환경에서 더 나은 가시성을 위한 로깅 제공

### ORM 또는 데이터베이스로 페칭

```tsx
import { db, posts } from '@/lib/db'

export default async function Page() {
  const allPosts = await db.select().from(posts)

  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

---

## Client Components

### React의 `use` Hook을 사용한 스트리밍

Server Component에서 데이터를 페칭하고 프로미스를 Client Component에 전달합니다:

**Server Component (app/blog/page.tsx):**
```tsx
import Posts from '@/app/ui/posts'
import { Suspense } from 'react'

export default function Page() {
  const posts = getPosts() // await하지 않음

  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <Posts posts={posts} />
    </Suspense>
  )
}
```

**Client Component (app/ui/posts.tsx):**
```tsx
'use client'
import { use } from 'react'

export default function Posts({ posts }) {
  const allPosts = use(posts)

  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### 커뮤니티 라이브러리 (SWR/React Query)

```tsx
'use client'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((r) => r.json())

export default function BlogPage() {
  const { data, error, isLoading } = useSWR(
    'https://api.vercel.app/blog',
    fetcher
  )

  if (isLoading) return <div>로딩 중...</div>
  if (error) return <div>오류: {error.message}</div>

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

---

## 중복 제거 및 캐싱

### Request Memoization (요청 메모이제이션)
- 단일 렌더 패스에서 동일한 URL/옵션을 가진 `fetch` 호출에 대해 자동으로 적용
- 요청 수명 범위 내에서만 유효
- Abort signal을 사용하여 옵트아웃 가능

### Data Cache
```tsx
// fetch와 cache 옵션 사용
fetch(url, { cache: 'force-cache' })

// ORM/데이터베이스와 함께 React cache 사용
import { cache } from 'react'

export const getPost = cache(async (id: string) => {
  const post = await db.query.posts.findFirst({
    where: eq(posts.id, parseInt(id)),
  })
})
```

---

## 스트리밍

### `loading.js` 사용

`app/blog/loading.tsx`를 생성하여 전체 페이지를 스트리밍합니다:

```tsx
export default function Loading() {
  return <div>로딩 중...</div>
}
```

- 자동으로 페이지를 `<Suspense>` 경계로 감쌉니다
- 네비게이션 시 즉시 로딩 UI를 표시합니다
- 렌더링이 완료되면 콘텐츠가 자동으로 교체됩니다

### `<Suspense>` 사용

세밀한 스트리밍 제어를 위해:

```tsx
import { Suspense } from 'react'
import BlogList from '@/components/BlogList'
import BlogListSkeleton from '@/components/BlogListSkeleton'

export default function BlogPage() {
  return (
    <div>
      {/* 즉시 전송됨 */}
      <header>
        <h1>블로그에 오신 것을 환영합니다</h1>
      </header>

      {/* 스트리밍됨 */}
      <Suspense fallback={<BlogListSkeleton />}>
        <BlogList />
      </Suspense>
    </div>
  )
}
```

---

## 데이터 페칭 패턴

### 순차 데이터 페칭 (Sequential)

하나의 요청이 다른 요청에 의존하는 경우:

```tsx
export default async function Page({ params }) {
  const { username } = await params
  const artist = await getArtist(username)

  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<div>로딩 중...</div>}>
        <Playlists artistID={artist.id} />
      </Suspense>
    </>
  )
}

async function Playlists({ artistID }) {
  const playlists = await getArtistPlaylists(artistID)
  return (
    <ul>
      {playlists.map((playlist) => (
        <li key={playlist.id}>{playlist.name}</li>
      ))}
    </ul>
  )
}
```

### 병렬 데이터 페칭 (Parallel)

여러 요청을 동시에 시작:

```tsx
export default async function Page({ params }) {
  const { username } = await params

  // 두 요청 모두 시작
  const artistData = getArtist(username)
  const albumsData = getAlbums(username)

  // 함께 await
  const [artist, albums] = await Promise.all([artistData, albumsData])

  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums} />
    </>
  )
}
```

**오류 처리:** 하나의 실패가 전체 작업을 실패시키지 않아야 하는 경우 `Promise.allSettled()`를 사용하세요.

### 데이터 미리 로드 (Preloading)

렌더링 전에 데이터 페칭을 적극적으로 시작:

```tsx
const preload = (id: string) => {
  void getItem(id) // void는 사용하지 않는 프로미스 경고를 방지
}

export default async function Page({ params }) {
  const { id } = await params
  preload(id)
  const isAvailable = await checkIsAvailable()

  return isAvailable ? <Item id={id} /> : null
}

export async function Item({ id }) {
  const result = await getItem(id)
  // ...
}
```

**React `cache`와 함께:**

```ts
import { cache } from 'react'
import 'server-only'

export const preload = (id: string) => {
  void getItem(id)
}

export const getItem = cache(async (id: string) => {
  // 캐시된 데이터 페칭
})
```

---

## 실전 예제

### 사용자 대시보드 (병렬 페칭)

```tsx
// app/dashboard/[userId]/page.tsx
export default async function DashboardPage({ params }) {
  const { userId } = await params

  // 모든 데이터를 병렬로 페칭
  const [user, stats, activities] = await Promise.all([
    getUser(userId),
    getUserStats(userId),
    getRecentActivities(userId)
  ])

  return (
    <div>
      <h1>{user.name}의 대시보드</h1>
      <StatsWidget stats={stats} />
      <ActivitiesList activities={activities} />
    </div>
  )
}
```

### 블로그 포스트 (순차 + 스트리밍)

```tsx
// app/blog/[slug]/page.tsx
import { Suspense } from 'react'

export default async function BlogPost({ params }) {
  const { slug } = await params

  // 포스트 먼저 페칭 (필수)
  const post = await getPost(slug)

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>

      {/* 댓글은 스트리밍 */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={post.id} />
      </Suspense>

      {/* 관련 포스트도 스트리밍 */}
      <Suspense fallback={<RelatedPostsSkeleton />}>
        <RelatedPosts tags={post.tags} />
      </Suspense>
    </article>
  )
}

async function Comments({ postId }) {
  const comments = await getComments(postId)
  return <CommentsList comments={comments} />
}

async function RelatedPosts({ tags }) {
  const posts = await getRelatedPosts(tags)
  return <RelatedPostsList posts={posts} />
}
```

### 검색 페이지 (Client Component)

```tsx
// app/search/page.tsx
'use client'
import { useState, useEffect } from 'react'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then(r => r.json())

export default function SearchPage() {
  const [query, setQuery] = useState('')
  const [debouncedQuery, setDebouncedQuery] = useState('')

  // 디바운싱
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedQuery(query), 300)
    return () => clearTimeout(timer)
  }, [query])

  const { data, error, isLoading } = useSWR(
    debouncedQuery ? `/api/search?q=${debouncedQuery}` : null,
    fetcher
  )

  return (
    <div>
      <input
        type="search"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="검색..."
      />

      {isLoading && <div>검색 중...</div>}
      {error && <div>검색 오류</div>}
      {data && <SearchResults results={data} />}
    </div>
  )
}
```

---

## 모범 사례

### 1. 의미 있는 로딩 상태

```tsx
// ❌ 나쁜 예
<div>Loading...</div>

// ✅ 좋은 예
<div className="animate-pulse">
  <div className="h-4 bg-gray-200 rounded w-3/4 mb-4"></div>
  <div className="h-4 bg-gray-200 rounded w-1/2"></div>
</div>
```

### 2. 첫 번째 요청 최적화

```tsx
// ❌ 나쁜 예 - 순차적 워터폴
export default async function Page() {
  const user = await getUser()
  const posts = await getPosts(user.id)
  const comments = await getComments(posts[0].id)
  // ...
}

// ✅ 좋은 예 - 병렬화
export default async function Page() {
  const [user, posts] = await Promise.all([
    getUser(),
    getPosts()
  ])
  // ...
}
```

### 3. Suspense 전략적 사용

```tsx
export default function Page() {
  return (
    <>
      {/* 중요한 콘텐츠 - 즉시 */}
      <MainContent />

      {/* 덜 중요한 콘텐츠 - 스트리밍 */}
      <Suspense fallback={<Skeleton />}>
        <SecondaryContent />
      </Suspense>
    </>
  )
}
```

### 4. 캐싱 적절히 사용

```tsx
// 자주 접근하는 데이터 캐싱
import { cache } from 'react'

export const getUser = cache(async (id: string) => {
  const user = await db.user.findUnique({ where: { id } })
  return user
})
```

### 5. 오류 처리

```tsx
export default async function Page() {
  try {
    const data = await fetchData()
    return <DataDisplay data={data} />
  } catch (error) {
    console.error('Data fetch failed:', error)
    return <ErrorDisplay />
  }
}
```

---

## 성능 최적화 팁

### 1. 요청 중복 제거

```tsx
// 동일한 데이터를 여러 컴포넌트에서 페칭 가능
// Next.js가 자동으로 중복 제거
async function Header() {
  const user = await getUser() // 캐시됨
  return <div>{user.name}</div>
}

async function Sidebar() {
  const user = await getUser() // 캐시에서 가져옴
  return <div>{user.avatar}</div>
}
```

### 2. Preloading 패턴

```tsx
import { preload } from '@/lib/data'

export default async function Page({ params }) {
  const { id } = await params

  // 조기에 페칭 시작
  preload(id)

  // 다른 작업 수행
  const metadata = await getMetadata()

  // 데이터 사용
  return <Item id={id} metadata={metadata} />
}
```

### 3. 스트리밍으로 TTFB 개선

```tsx
// 빠른 초기 로드, 점진적 향상
export default function Page() {
  return (
    <>
      <Header /> {/* 즉시 */}
      <Suspense fallback={<Skeleton />}>
        <SlowContent /> {/* 스트리밍 */}
      </Suspense>
    </>
  )
}
```

---

## 관련 문서

- [캐싱 및 재검증](../getting-started/06-caching-and-revalidating.md)
- [fetch API](../api-reference/functions/fetch.md)
- [loading.js](../api-reference/file-conventions/loading.md)
- [캐싱 가이드](./caching.md)
- [Server Components](../getting-started/05-server-and-client-components.md)

## Sources

- [Getting Started: Fetching Data | Next.js](https://nextjs.org/docs/app/getting-started/fetching-data)
- [Data Fetching: Data Fetching Patterns and Best Practices | Next.js](https://nextjs.org/docs/14/app/building-your-application/data-fetching/patterns)
