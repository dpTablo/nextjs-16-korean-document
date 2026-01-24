# 데이터 가져오기 (Fetching Data)

이 페이지에서는 서버 컴포넌트와 클라이언트 컴포넌트에서 데이터를 가져오는 방법, 그리고 데이터에 의존하는 컴포넌트를 스트리밍하는 방법을 설명합니다.

## 데이터 가져오기

### 서버 컴포넌트

서버 컴포넌트에서는 비동기 I/O를 사용하여 데이터를 가져올 수 있습니다:

1. `fetch` API
2. ORM 또는 데이터베이스
3. `fs`와 같은 Node.js API

#### `fetch` API 사용하기

컴포넌트를 비동기 함수로 만들고 `fetch` 호출을 await합니다:

```tsx filename="app/blog/page.tsx"
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

**알아두면 좋은 점:**
- `fetch` 응답은 기본적으로 캐시되지 않지만, Next.js는 라우트를 사전 렌더링하고 출력을 캐시합니다
- 동적 렌더링을 위해 `{ cache: 'no-store' }`를 사용하세요
- 개발 환경에서는 더 나은 디버깅을 위해 `fetch` 호출을 로그로 기록할 수 있습니다

#### ORM 또는 데이터베이스 사용하기

서버 컴포넌트는 서버에서 실행되므로 데이터베이스를 직접 안전하게 쿼리할 수 있습니다:

```tsx filename="app/blog/page.tsx"
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

### 클라이언트 컴포넌트

클라이언트 컴포넌트에서 데이터를 가져오는 두 가지 방법:

1. React의 `use` 훅
2. SWR 또는 React Query와 같은 커뮤니티 라이브러리

#### `use` 훅으로 데이터 스트리밍하기

서버 컴포넌트에서 데이터를 가져오고 Promise를 클라이언트 컴포넌트에 전달합니다:

```tsx filename="app/blog/page.tsx"
import Posts from '@/app/ui/posts'
import { Suspense } from 'react'

export default function Page() {
  const posts = getPosts()

  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <Posts posts={posts} />
    </Suspense>
  )
}
```

클라이언트 컴포넌트에서 `use` 훅을 사용하여 Promise를 읽습니다:

```tsx filename="app/ui/posts.tsx"
'use client'
import { use } from 'react'

export default function Posts({
  posts,
}: {
  posts: Promise<{ id: string; title: string }[]>
}) {
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

#### 커뮤니티 라이브러리

클라이언트 컴포넌트 데이터 가져오기를 위해 SWR 또는 React Query와 같은 라이브러리를 사용합니다:

```tsx filename="app/blog/page.tsx"
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
      {data.map((post: { id: string; title: string }) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

## 요청 중복 제거 및 데이터 캐싱

### 요청 메모이제이션

단일 렌더링 패스에서 동일한 URL과 옵션으로 `GET` 또는 `HEAD`를 사용하는 `fetch` 호출은 자동으로 하나의 요청으로 결합됩니다. Abort 시그널을 전달하여 이 기능을 비활성화할 수 있습니다.

### 데이터 캐시

렌더링 패스와 들어오는 요청 간에 데이터를 공유하려면 `fetch` 옵션에 `cache: 'force-cache'`를 설정합니다.

### React `cache` 사용하기

ORM 또는 직접 데이터베이스 쿼리의 경우, React `cache` 함수로 데이터 액세스를 래핑합니다:

```tsx filename="app/lib/data.ts"
import { cache } from 'react'
import { db, posts, eq } from '@/lib/db'

export const getPost = cache(async (id: string) => {
  const post = await db.query.posts.findFirst({
    where: eq(posts.id, parseInt(id)),
  })
})
```

## 스트리밍

> **참고:** Next.js 15 이상에서 `cacheComponents` 설정 옵션을 활성화해야 합니다.

스트리밍은 페이지 HTML을 더 작은 청크로 분할하고 점진적으로 클라이언트에 전송하여 초기 로드 시간을 개선합니다.

### `loading.js` 사용하기

페이지와 같은 폴더에 `loading.js` 파일을 생성하여 전체 페이지를 스트리밍합니다:

```tsx filename="app/blog/loading.tsx"
export default function Loading() {
  return <div>로딩 중...</div>
}
```

사용자는 페이지가 렌더링되는 동안 즉시 로딩 상태를 볼 수 있습니다. 내부적으로 `loading.js`는 페이지를 `<Suspense>` 경계로 래핑합니다.

### `<Suspense>` 사용하기

스트리밍할 부분을 세밀하게 제어합니다:

```tsx filename="app/blog/page.tsx"
import { Suspense } from 'react'
import BlogList from '@/components/BlogList'
import BlogListSkeleton from '@/components/BlogListSkeleton'

export default function BlogPage() {
  return (
    <div>
      <header>
        <h1>블로그에 오신 것을 환영합니다</h1>
        <p>아래에서 최신 게시물을 읽어보세요.</p>
      </header>
      <main>
        <Suspense fallback={<BlogListSkeleton />}>
          <BlogList />
        </Suspense>
      </main>
    </div>
  )
}
```

### 의미 있는 로딩 상태 만들기

스켈레톤, 스피너, 또는 커버 사진이나 제목과 같은 미래 화면의 의미 있는 부분을 사용하여 의미 있는 로딩 상태를 디자인하세요.

## 예제

### 순차적 데이터 가져오기

하나의 요청이 다른 요청에 의존하는 경우:

```tsx filename="app/artist/[username]/page.tsx"
export default async function Page({
  params,
}: {
  params: Promise<{ username: string }>
}) {
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

async function Playlists({ artistID }: { artistID: string }) {
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

### 병렬 데이터 가져오기

`Promise.all`을 사용하여 여러 요청을 동시에 시작합니다:

```tsx filename="app/artist/[username]/page.tsx"
export default async function Page({
  params,
}: {
  params: Promise<{ username: string }>
}) {
  const { username } = await params

  const artistData = getArtist(username)
  const albumsData = getAlbums(username)

  const [artist, albums] = await Promise.all([artistData, albumsData])

  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums} />
    </>
  )
}
```

> **팁:** 개별 요청 실패를 처리하려면 `Promise.allSettled`를 대신 사용하세요.

### 데이터 미리 로드하기

블로킹 요청 전에 데이터 가져오기를 미리 호출합니다:

```tsx filename="app/item/[id]/page.tsx"
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  preload(id)
  const isAvailable = await checkIsAvailable()

  return isAvailable ? <Item id={id} /> : null
}

const preload = (id: string) => {
  void getItem(id)
}

export async function Item({ id }: { id: string }) {
  const result = await getItem(id)
}
```

React의 `cache` 함수로 재사용 가능한 유틸리티를 만듭니다:

```ts filename="utils/get-item.ts"
import { cache } from 'react'
import 'server-only'
import { getItem } from '@/lib/data'

export const preload = (id: string) => {
  void getItem(id)
}

export const getItem = cache(async (id: string) => {
  // ...
})
```

## API 참조

- [데이터 보안](/app-router/guides/data-security.md)
- [fetch](/app-router/api-reference/functions/fetch.md)
- [loading.js](/app-router/api-reference/file-conventions/loading.md)
- [logging](/app-router/api-reference/config/next-config-js/logging.md)
