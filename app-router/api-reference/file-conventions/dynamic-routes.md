# 동적 라우트 세그먼트 (Dynamic Route Segments)

동적 라우트 세그먼트는 정확한 라우트 이름을 미리 알 수 없을 때, 동적 데이터로부터 라우트를 생성하려고 할 때 사용됩니다. 이를 통해 요청 시점이나 빌드 시점에 라우트를 채울 수 있습니다.

## 규칙

폴더명을 대괄호로 감싸면 동적 세그먼트를 생성할 수 있습니다: `[folderName]`. 예를 들어 블로그는 `app/blog/[slug]/page.js` 라우트를 포함할 수 있으며, `[slug]`는 블로그 게시물의 동적 세그먼트입니다.

```tsx filename="app/blog/[slug]/page.tsx"
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  return <div>내 게시물: {slug}</div>
}
```

```jsx filename="app/blog/[slug]/page.js"
export default async function Page({ params }) {
  const { slug } = await params
  return <div>내 게시물: {slug}</div>
}
```

동적 세그먼트는 `layout`, `page`, `route`, `generateMetadata` 함수에 `params` prop으로 전달됩니다.

| 라우트 | 예시 URL | `params` |
|-------|----------|----------|
| `app/blog/[slug]/page.js` | `/blog/a` | `{ slug: 'a' }` |
| `app/blog/[slug]/page.js` | `/blog/b` | `{ slug: 'b' }` |
| `app/blog/[slug]/page.js` | `/blog/c` | `{ slug: 'c' }` |

### 클라이언트 컴포넌트에서 사용

클라이언트 컴포넌트 **page**에서는 [`use`](https://react.dev/reference/react/use) 훅을 사용하여 동적 세그먼트에 접근할 수 있습니다.

```tsx filename="app/blog/[slug]/page.tsx"
'use client'
import { use } from 'react'

export default function BlogPostPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = use(params)

  return (
    <div>
      <p>{slug}</p>
    </div>
  )
}
```

또는 [`useParams`](/app-router/api-reference/functions/useParams.md) 훅을 사용하여 클라이언트 컴포넌트 트리 어디서든 `params`에 접근할 수 있습니다.

### Catch-all 세그먼트

대괄호 안에 줄임표(`...`)를 추가하여 동적 세그먼트를 **catch-all**로 확장할 수 있습니다: `[...folderName]`.

예를 들어, `app/shop/[...slug]/page.js`는 `/shop/clothes`뿐만 아니라 `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts` 등과도 매칭됩니다.

| 라우트 | 예시 URL | `params` |
|-------|----------|----------|
| `app/shop/[...slug]/page.js` | `/shop/a` | `{ slug: ['a'] }` |
| `app/shop/[...slug]/page.js` | `/shop/a/b` | `{ slug: ['a', 'b'] }` |
| `app/shop/[...slug]/page.js` | `/shop/a/b/c` | `{ slug: ['a', 'b', 'c'] }` |

### Optional Catch-all 세그먼트

Catch-all 세그먼트를 **optional**로 만들려면 이중 대괄호로 감싸면 됩니다: `[[...folderName]]`.

예를 들어, `app/shop/[[...slug]]/page.js`는 `/shop/clothes`, `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts`뿐만 아니라 **`/shop`도 매칭**됩니다.

**Catch-all**과 **Optional catch-all** 세그먼트의 차이는 optional의 경우 파라미터 없는 라우트도 매칭된다는 점입니다.

| 라우트 | 예시 URL | `params` |
|-------|----------|----------|
| `app/shop/[[...slug]]/page.js` | `/shop` | `{ slug: undefined }` |
| `app/shop/[[...slug]]/page.js` | `/shop/a` | `{ slug: ['a'] }` |
| `app/shop/[[...slug]]/page.js` | `/shop/a/b` | `{ slug: ['a', 'b'] }` |
| `app/shop/[[...slug]]/page.js` | `/shop/a/b/c` | `{ slug: ['a', 'b', 'c'] }` |

### TypeScript

TypeScript를 사용할 때, 설정된 라우트 세그먼트에 따라 `params`에 타입을 추가할 수 있습니다. `PageProps<'/route'>`, `LayoutProps<'/route'>`, 또는 `RouteContext<'/route'>`를 사용하여 각각 `page`, `layout`, `route`에서 `params`를 타입할 수 있습니다.

라우트 `params` 값은 `string`, `string[]`, 또는 `undefined`(optional catch-all 세그먼트의 경우)로 타입됩니다. 런타임까지 값을 알 수 없기 때문에 이러한 광범위한 타입은 애플리케이션 코드가 모든 경우를 처리하도록 돕습니다.

| 라우트 | `params` 타입 정의 |
|-------|-------------------|
| `app/blog/[slug]/page.js` | `{ slug: string }` |
| `app/shop/[...slug]/page.js` | `{ slug: string[] }` |
| `app/shop/[[...slug]]/page.js` | `{ slug?: string[] }` |
| `app/[categoryId]/[itemId]/page.js` | `{ categoryId: string, itemId: string }` |

`[locale]` 파라미터처럼 고정된 수의 유효한 값만 가질 수 있는 라우트에서는 런타임 검증을 사용하여 사용자가 입력한 잘못된 파라미터를 처리하고, 알려진 집합의 좁은 타입으로 나머지 애플리케이션을 작동시킬 수 있습니다.

```tsx filename="/app/[locale]/page.tsx"
import { notFound } from 'next/navigation'
import type { Locale } from '@i18n/types'
import { isValidLocale } from '@i18n/utils'

function assertValidLocale(value: string): asserts value is Locale {
  if (!isValidLocale(value)) notFound()
}

export default async function Page(props: PageProps<'/[locale]'>) {
  const { locale } = await props.params // locale은 string으로 타입됨
  assertValidLocale(locale)
  // locale은 이제 Locale로 타입됨
}
```

## 동작

- `params` prop은 Promise입니다. 값에 접근하려면 `async`/`await` 또는 React의 `use` 함수를 사용해야 합니다.
  - 버전 14 이전에는 `params`가 동기식 prop이었습니다. 하위 호환성을 위해 Next.js 15에서는 여전히 동기식으로 접근할 수 있지만, 이 동작은 향후 사용 중단될 예정입니다.

## 예제

### `generateStaticParams` 사용

[`generateStaticParams`](/app-router/api-reference/functions/generateStaticParams.md) 함수는 요청 시점의 온디맨드 방식 대신 빌드 시점에 라우트를 [정적으로 생성](/app-router/guides/caching.md#static-rendering)하는 데 사용할 수 있습니다.

```tsx filename="app/blog/[slug]/page.tsx"
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

```jsx filename="app/blog/[slug]/page.js"
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())

  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

`generateStaticParams` 함수 내에서 `fetch`를 사용할 때, 요청은 [자동으로 중복 제거](/app-router/guides/caching.md#request-memoization)됩니다. 이는 동일한 데이터에 대한 여러 네트워크 호출을 피하므로 빌드 시간을 단축시킵니다.

### `generateStaticParams`를 사용한 동적 GET Route Handlers

`generateStaticParams`는 동적 [Route Handlers](/app-router/api-reference/file-conventions/route.md)에서도 작동하여 빌드 시점에 API 응답을 정적으로 생성합니다.

```ts filename="app/api/posts/[id]/route.ts"
export async function generateStaticParams() {
  const posts: { id: number }[] = await fetch(
    'https://api.vercel.app/blog'
  ).then((res) => res.json())

  return posts.map((post) => ({
    id: `${post.id}`,
  }))
}

export async function GET(
  request: Request,
  { params }: RouteContext<'/api/posts/[id]'>
) {
  const { id } = await params
  const res = await fetch(`https://api.vercel.app/blog/${id}`)

  if (!res.ok) {
    return Response.json({ error: 'Post not found' }, { status: 404 })
  }

  const post = await res.json()
  return Response.json(post)
}
```

```js filename="app/api/posts/[id]/route.js"
export async function generateStaticParams() {
  const posts = await fetch('https://api.vercel.app/blog').then((res) =>
    res.json()
  )

  return posts.map((post) => ({
    id: `${post.id}`,
  }))
}

export async function GET(request, { params }) {
  const { id } = await params
  const res = await fetch(`https://api.vercel.app/blog/${id}`)

  if (!res.ok) {
    return Response.json({ error: 'Post not found' }, { status: 404 })
  }

  const post = await res.json()
  return Response.json(post)
}
```

이 예제에서, `generateStaticParams`에서 반환된 모든 블로그 게시물 ID에 대한 라우트 핸들러는 빌드 시점에 정적으로 생성됩니다. 다른 ID에 대한 요청은 요청 시점에 동적으로 처리됩니다.

## 다음 단계

다음에 수행할 작업에 대한 자세한 정보는 다음 섹션을 참조하세요:

- [generateStaticParams](/app-router/api-reference/functions/generateStaticParams.md) - generateStaticParams 함수에 대한 API 참조
