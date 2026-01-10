# fetch

## 개요

Next.js는 Web `fetch()` API를 확장하여 서버 측 요청이 프레임워크의 [Data Cache](../../guides/caching.md#data-cache)를 통해 영구적인 캐싱 및 재검증 의미론을 설정할 수 있도록 합니다.

## 기본 사용법

Server Components에서 `async/await`와 함께 `fetch`를 직접 호출할 수 있습니다:

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

---

## 캐싱 옵션

### `options.cache`

요청이 Next.js Data Cache와 상호작용하는 방식을 제어합니다:

```ts
fetch(`https://...`, { cache: 'force-cache' | 'no-store' })
```

**값:**

- **`auto`** (기본값):
  - 개발 환경: 모든 요청마다 원격 서버에서 가져옴
  - 빌드 시: 정적 사전 렌더링을 위해 `next build` 중에 한 번 가져옴
  - Dynamic API가 감지된 경우: 모든 요청마다 가져옴

- **`no-store`**: Dynamic API와 관계없이 모든 요청마다 원격 서버에서 가져옴

- **`force-cache`**: Data Cache에서 일치하는 요청을 찾음
  - 신선한 일치 항목이 있으면 캐시된 결과 반환
  - 일치 항목이 없거나 stale한 경우 원격 서버에서 가져온 후 캐시 업데이트

### `options.next.revalidate`

리소스의 캐시 수명을 설정합니다 (초 단위):

```ts
fetch(`https://...`, { next: { revalidate: false | 0 | number } })
```

**값:**

- **`false`**: 무기한 캐시 (`revalidate: Infinity`와 의미적으로 동일). HTTP 캐시는 시간이 지남에 따라 오래된 리소스를 제거할 수 있습니다.
- **`0`**: 캐싱 방지
- **`number`**: 캐시 수명(초) (예: `3600`은 1시간)

### `options.next.tags`

온디맨드 재검증을 위한 캐시 태그를 설정합니다:

```ts
fetch(`https://...`, { next: { tags: ['collection'] } })
```

- 최대 태그 길이: 256자
- 최대 태그 항목: 128개
- [`revalidateTag()`](./revalidateTag.md)와 함께 사용

---

## 예제

### 기본 데이터 페칭

```tsx
// app/blog/page.tsx
export default async function BlogPage() {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

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

### 캐싱 강제 (무기한)

```tsx
// app/products/page.tsx
export default async function ProductsPage() {
  // 무기한 캐시 - 수동 재검증까지 캐시됨
  const res = await fetch('https://api.example.com/products', {
    cache: 'force-cache'
  })
  const products = await res.json()

  return <ProductList products={products} />
}
```

### 캐싱 비활성화

```tsx
// app/realtime/page.tsx
export default async function RealtimePage() {
  // 캐시하지 않음 - 항상 최신 데이터
  const res = await fetch('https://api.example.com/realtime-data', {
    cache: 'no-store'
  })
  const data = await res.json()

  return <RealtimeDisplay data={data} />
}
```

### 시간 기반 재검증

```tsx
// app/news/page.tsx
export default async function NewsPage() {
  // 60초마다 재검증
  const res = await fetch('https://api.example.com/news', {
    next: { revalidate: 60 }
  })
  const news = await res.json()

  return <NewsList news={news} />
}
```

### 태그 기반 재검증

```tsx
// app/posts/page.tsx
export default async function PostsPage() {
  // 'posts' 태그로 캐시 - revalidateTag('posts')로 재검증 가능
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  })
  const posts = await res.json()

  return <PostsList posts={posts} />
}
```

```tsx
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function createPost(formData: FormData) {
  // 새 포스트 생성 후 캐시 재검증
  await savePost(formData)
  revalidateTag('posts')
}
```

### 여러 태그 사용

```tsx
export default async function Page() {
  const res = await fetch('https://api.example.com/data', {
    next: {
      tags: ['collection', 'featured', 'recent'],
      revalidate: 3600
    }
  })
  const data = await res.json()

  return <DataDisplay data={data} />
}
```

### POST 요청

```tsx
// app/api/submit/route.ts
export async function POST(request: Request) {
  const body = await request.json()

  const res = await fetch('https://api.example.com/submit', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(body),
    cache: 'no-store' // POST 요청은 일반적으로 캐시하지 않음
  })

  const data = await res.json()
  return Response.json(data)
}
```

### 헤더와 함께 사용

```tsx
export default async function Page() {
  const res = await fetch('https://api.example.com/data', {
    headers: {
      'Authorization': `Bearer ${process.env.API_TOKEN}`,
      'Content-Type': 'application/json',
    },
    next: { revalidate: 3600 }
  })

  const data = await res.json()
  return <div>{data.title}</div>
}
```

---

## 중요 참고사항

> **충돌하는 옵션**: `{ revalidate: 3600, cache: 'no-store' }`와 같은 옵션은 허용되지 않으며 무시됩니다. 개발 모드에서 경고가 표시됩니다.

> **라우트 레벨 재검증**: 개별 `fetch()`가 라우트의 기본값보다 낮은 `revalidate`를 설정하면 라우트의 재검증 간격이 감소합니다.

> **동일한 URL의 여러 요청**: 두 개의 fetch가 다른 `revalidate` 값으로 동일한 URL을 사용하는 경우 더 낮은 값이 사용됩니다.

---

## 문제 해결

### 개발 환경에서 HMR 캐시

Next.js는 더 빠른 응답을 위해 Hot Module Replacement (HMR) 전반에 걸쳐 `fetch` 응답을 캐시합니다. 기본적으로 이는 `auto` 및 `cache: 'no-store'`를 포함한 모든 fetch에 적용됩니다. HMR 새로고침 사이에 새 데이터가 표시되지 않지만 전체 페이지 다시 로드 및 네비게이션 시 캐시가 지워집니다.

[`serverComponentsHmrCache`](https://nextjs.org/docs/app/api-reference/config/next-config-js/serverComponentsHmrCache)로 구성할 수 있습니다.

### 강제 새로고침 동작

개발 환경에서 요청에 `cache-control: no-cache` 헤더가 포함된 경우 (강제 새로고침 중 일반적), `options.cache`, `options.next.revalidate` 및 `options.next.tags`가 무시되고 fetch가 소스에서 제공됩니다.

---

## 요청 중복 제거

Next.js는 동일한 URL과 옵션으로 여러 `fetch` 요청을 자동으로 중복 제거합니다:

```tsx
// 이 두 fetch는 자동으로 중복 제거됨
async function getData() {
  const res = await fetch('https://api.example.com/data')
  return res.json()
}

export default async function Page() {
  // 동일한 요청이 여러 번 호출되어도 한 번만 실행됨
  const data1 = await getData()
  const data2 = await getData()

  return <div>...</div>
}
```

---

## 라우트 세그먼트 옵션과의 관계

라우트 세그먼트 설정 옵션과 `fetch` 옵션의 관계:

```tsx
// app/posts/page.tsx

// 라우트 레벨 설정
export const revalidate = 3600

export default async function PostsPage() {
  // 이 fetch는 라우트의 revalidate 설정을 상속
  const res1 = await fetch('https://api.example.com/posts')

  // 개별 fetch가 더 낮은 값을 설정하면 우선
  const res2 = await fetch('https://api.example.com/featured', {
    next: { revalidate: 60 } // 라우트 전체가 60초로 재검증됨
  })

  // ...
}
```

---

## 모범 사례

### 1. 오류 처리

```tsx
export default async function Page() {
  try {
    const res = await fetch('https://api.example.com/data')

    if (!res.ok) {
      throw new Error('Failed to fetch data')
    }

    const data = await res.json()
    return <DataDisplay data={data} />
  } catch (error) {
    console.error('Fetch error:', error)
    return <ErrorDisplay />
  }
}
```

### 2. 타입 안정성

```tsx
interface Post {
  id: number
  title: string
  content: string
}

export default async function Page() {
  const res = await fetch('https://api.example.com/posts')
  const posts: Post[] = await res.json()

  return <PostList posts={posts} />
}
```

### 3. 환경 변수 사용

```tsx
export default async function Page() {
  const res = await fetch(`${process.env.API_URL}/data`, {
    headers: {
      'Authorization': `Bearer ${process.env.API_TOKEN}`,
    },
    next: { revalidate: 3600 }
  })

  const data = await res.json()
  return <div>{data.title}</div>
}
```

### 4. 조건부 캐싱

```tsx
export default async function Page({ searchParams }: { searchParams: { query?: string } }) {
  const shouldCache = !searchParams.query

  const res = await fetch('https://api.example.com/data', {
    cache: shouldCache ? 'force-cache' : 'no-store'
  })

  const data = await res.json()
  return <div>...</div>
}
```

---

## 버전 히스토리

| 버전   | 변경사항             |
|-----------|-------------------|
| `v13.0.0` | `fetch` 도입 |

---

## 관련 문서

- [캐싱 및 재검증](../../getting-started/06-caching-and-revalidating.md)
- [캐싱 가이드](../../guides/caching.md)
- [revalidateTag](./revalidateTag.md)
- [revalidatePath](./revalidatePath.md)
- [Data Cache](../../guides/caching.md#data-cache)
