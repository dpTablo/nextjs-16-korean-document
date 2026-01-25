# 캐시 컴포넌트 (Cache Components)

캐시 컴포넌트는 단일 라우트 내에서 정적, 캐시된, 동적 콘텐츠를 혼합할 수 있게 해주는 opt-in 기능으로, 정적 사이트의 속도와 동적 렌더링의 유연성을 결합합니다.

다음 설정으로 활성화할 수 있습니다:

```ts
// next.config.ts
const nextConfig: NextConfig = {
  cacheComponents: true,
}
```

## 작동 방식

빌드 시점에 Next.js는 라우트를 **정적 HTML 셸**로 사전 렌더링하여 브라우저에 즉시 전송하며, 동적 콘텐츠는 준비되는 대로 업데이트됩니다. 이것은 **부분 사전 렌더링(Partial Prerendering, PPR)**을 사용합니다.

컴포넌트는 다음에 접근하지 않으면 자동으로 사전 렌더링됩니다:
- 네트워크 리소스
- 특정 시스템 API
- 들어오는 요청 데이터

그렇지 않으면 다음 방법 중 하나로 명시적으로 처리해야 합니다:
1. **`<Suspense>`로 지연** - 콘텐츠가 준비될 때까지 대체 UI 표시
2. **`use cache`로 캐싱** - 정적 셸에 포함 (요청 데이터가 필요하지 않은 경우)

## 자동으로 사전 렌더링되는 콘텐츠

사전 렌더링 중에 완료되는 작업들:
- 동기 I/O (`fs.readFileSync`)
- 모듈 임포트
- 순수 연산

```tsx
export default async function Page() {
  const content = fs.readFileSync('./config.json', 'utf-8')
  const constants = await import('./constants.json')
  const processed = JSON.parse(content).items.map((item) => item.value * 2)

  return (
    <div>
      <h1>{constants.appName}</h1>
      <ul>
        {processed.map((value, i) => (
          <li key={i}>{value}</li>
        ))}
      </ul>
    </div>
  )
}
```

## 요청 시점으로 지연하기

### 동적 콘텐츠

외부 소스에서 오는 콘텐츠에는 `<Suspense>`를 사용합니다:

```tsx
async function DynamicContent() {
  const data = await fetch('https://api.example.com/data')
  const users = await db.query('SELECT * FROM users')
  const file = await fs.readFile('..', 'utf-8')
  await new Promise((resolve) => setTimeout(resolve, 100))
  return <div>정적 셸에 포함되지 않음</div>
}

export default async function Page(props) {
  return (
    <>
      <h1>정적 셸의 일부</h1>
      <Suspense fallback={<p>로딩 중..</p>}>
        <DynamicContent />
      </Suspense>
    </>
  )
}
```

### 런타임 데이터

`<Suspense>`가 필요한 요청 전용 데이터:
- `cookies()` - 사용자의 쿠키 데이터
- `headers()` - 요청 헤더
- `searchParams` - URL 쿼리 파라미터
- `params` - 동적 라우트 파라미터

```tsx
async function RuntimeData({ searchParams }) {
  const cookieStore = await cookies()
  const headerStore = await headers()
  const search = await searchParams
  return <div>정적 셸에 포함되지 않음</div>
}

export default async function Page(props) {
  return (
    <Suspense fallback={<p>로딩 중..</p>}>
      <RuntimeData searchParams={props.searchParams} />
    </Suspense>
  )
}
```

### 비결정적 작업

`Math.random()`, `Date.now()`, `crypto.randomUUID()`와 같은 작업을 지연하려면 `connection()`을 사용합니다:

```tsx
import { connection } from 'next/server'

async function UniqueContent() {
  await connection()

  const random = Math.random()
  const now = Date.now()
  const uuid = crypto.randomUUID()

  return <div>{random} {now} {uuid}</div>
}

export default function Page() {
  return (
    <Suspense fallback={<p>로딩 중..</p>}>
      <UniqueContent />
    </Suspense>
  )
}
```

## `use cache` 사용하기

`use cache` 지시어는 함수/컴포넌트의 반환 값을 캐시합니다. 인수와 클로저로 캡처된 값은 자동으로 캐시 키의 일부가 됩니다.

### 사전 렌더링 중

`cacheLife`를 사용하여 정적 셸에 동적 콘텐츠를 캐시합니다:

```tsx
import { cacheLife } from 'next/cache'

export default async function Page() {
  'use cache'
  cacheLife('hours')

  const users = await db.query('SELECT * FROM users')

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

**사용자 정의 캐시 설정:**

```tsx
export default async function Page() {
  'use cache'
  cacheLife({
    stale: 3600,      // 1시간 후 stale 상태
    revalidate: 7200, // 2시간 후 재검증
    expire: 86400,    // 1일 후 만료
  })
  // ...
}
```

### 런타임 데이터와 함께

런타임 데이터를 추출하여 캐시된 함수에 전달합니다:

```tsx
export function Page() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <ProfileContent />
    </Suspense>
  )
}

async function ProfileContent() {
  const session = (await cookies()).get('session')?.value
  return <CachedContent sessionId={session} />
}

async function CachedContent({ sessionId }: { sessionId: string }) {
  'use cache'
  // sessionId가 캐시 키의 일부가 됨
  const data = await fetchUserData(sessionId)
  return <div>{data}</div>
}
```

### 비결정적 작업과 함께

`use cache` 내에서 비결정적 작업은 사전 렌더링 중 한 번만 실행됩니다:

```tsx
export default async function Page() {
  'use cache'

  const random = Math.random()
  const uuid = crypto.randomUUID()
  const now = Date.now()

  return (
    <div>
      <p>{random} {uuid} {now}</p>
    </div>
  )
}
```

모든 요청은 재검증될 때까지 동일한 캐시된 값을 제공합니다.

### 태깅과 재검증

**`updateTag` 사용 (즉시 새로고침):**

```tsx
import { cacheTag, updateTag } from 'next/cache'

export async function getCart() {
  'use cache'
  cacheTag('cart')
  // 데이터 가져오기
}

export async function updateCart(itemId: string) {
  'use server'
  // 데이터 쓰기
  updateTag('cart')
}
```

**`revalidateTag` 사용 (stale-while-revalidate):**

```tsx
import { cacheTag, revalidateTag } from 'next/cache'

export async function getPosts() {
  'use cache'
  cacheTag('posts')
  // 데이터 가져오기
}

export async function createPost(post: FormData) {
  'use server'
  // 데이터 쓰기
  revalidateTag('posts', 'max')
}
```

## 전체 예제

```tsx
import { Suspense } from 'react'
import { cookies } from 'next/headers'
import { cacheLife } from 'next/cache'
import Link from 'next/link'

export default function BlogPage() {
  return (
    <>
      {/* 정적 콘텐츠 - 자동으로 사전 렌더링 */}
      <header>
        <h1>우리 블로그</h1>
        <nav>
          <Link href="/">홈</Link> | <Link href="/about">소개</Link>
        </nav>
      </header>

      {/* 캐시된 동적 콘텐츠 - 정적 셸에 포함 */}
      <BlogPosts />

      {/* 런타임 동적 콘텐츠 - 요청 시 스트리밍 */}
      <Suspense fallback={<p>환경설정 로딩 중...</p>}>
        <UserPreferences />
      </Suspense>
    </>
  )
}

async function BlogPosts() {
  'use cache'
  cacheLife('hours')

  const res = await fetch('https://api.vercel.app/blog')
  const posts = await res.json()

  return (
    <section>
      <h2>최신 게시물</h2>
      <ul>
        {posts.slice(0, 5).map((post: any) => (
          <li key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.author} 작성, {post.date}</p>
          </li>
        ))}
      </ul>
    </section>
  )
}

async function UserPreferences() {
  const theme = (await cookies()).get('theme')?.value || 'light'
  const favoriteCategory = (await cookies()).get('category')?.value

  return (
    <aside>
      <p>테마: {theme}</p>
      {favoriteCategory && <p>선호 카테고리: {favoriteCategory}</p>}
    </aside>
  )
}
```

## Route Segment Config에서 마이그레이션

### `dynamic = "force-dynamic"` → 제거

모든 페이지가 기본적으로 동적이므로 필요하지 않습니다.

### `dynamic = "force-static"` → `use cache` 사용

```tsx
// 이전
export const dynamic = 'force-static'
export default async function Page() {
  const data = await fetch('https://api.example.com/data')
  return <div>...</div>
}

// 이후
import { cacheLife } from 'next/cache'
export default async function Page() {
  'use cache'
  cacheLife('max')
  const data = await fetch('https://api.example.com/data')
  return <div>...</div>
}
```

### `revalidate` → `cacheLife` 사용

```tsx
// 이전
export const revalidate = 3600

// 이후
import { cacheLife } from 'next/cache'
export default async function Page() {
  'use cache'
  cacheLife('hours')
  return <div>...</div>
}
```

### `fetchCache` → 필요 없음

`use cache`는 범위 내의 모든 데이터 페칭을 자동으로 캐시합니다.

### `runtime = 'edge'` → 지원되지 않음

캐시 컴포넌트는 Node.js 런타임이 필요합니다.

## 주요 기능

- **Activity 컴포넌트**: React의 `<Activity>`와 `"hidden"` 모드를 사용하여 클라이언트 사이드 네비게이션 중 컴포넌트 상태 보존
- **메타데이터/뷰포트**: 별도로 처리됨; 런타임 데이터 처리에 대한 문서 참조
- **GET Route Handlers**: `cacheComponents` 활성화 시 페이지와 동일한 사전 렌더링 모델을 따름

---

## 참고

- [use cache 지시어](/app-router/api-reference/use-cache.md)
- [cacheLife 함수](/app-router/api-reference/functions/cacheLife.md)
- [cacheTag 함수](/app-router/api-reference/functions/cacheTag.md)
- [부분 사전 렌더링](/app-router/guides/rendering.md)
