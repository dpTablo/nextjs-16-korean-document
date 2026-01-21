# 동적 라우트

정확한 세그먼트 이름을 미리 알 수 없을 때 동적 데이터로부터 라우트를 생성하기 위해 **동적 세그먼트**를 사용할 수 있습니다. 동적 세그먼트는 요청 시점에 채워지거나 빌드 시점에 [미리 렌더링](/docs/pages/api-reference/functions/getStaticPaths)될 수 있습니다.

## 규칙

파일이나 폴더 이름을 **대괄호로 감싸서** 동적 세그먼트를 생성합니다: `[segmentName]`

예를 들어, `[id]` 또는 `[slug]`와 같이 사용합니다.

동적 세그먼트는 [`useRouter`](/docs/pages/api-reference/functions/use-router) 훅을 통해 접근할 수 있습니다.

## 기본 예제

**파일 구조:** `pages/blog/[slug].js`

```jsx
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()
  return <p>Post: {router.query.slug}</p>
}
```

**라우팅 테이블:**

| 라우트 | 예시 URL | params |
|--------|---------|--------|
| `pages/blog/[slug].js` | `/blog/a` | `{ slug: 'a' }` |
| `pages/blog/[slug].js` | `/blog/b` | `{ slug: 'b' }` |
| `pages/blog/[slug].js` | `/blog/c` | `{ slug: 'c' }` |

## Catch-all 세그먼트

동적 세그먼트에 줄임표(`...`)를 추가하면 **모든** 후속 세그먼트를 캡처할 수 있습니다.

**형식:** `[...segmentName]`

예를 들어, `pages/shop/[...slug].js`는 `/shop/clothes`뿐만 아니라 `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts` 등도 매칭합니다.

**라우팅 테이블:**

| 라우트 | 예시 URL | params |
|--------|----------|--------|
| `pages/shop/[...slug].js` | `/shop/a` | `{ slug: ['a'] }` |
| `pages/shop/[...slug].js` | `/shop/a/b` | `{ slug: ['a', 'b'] }` |
| `pages/shop/[...slug].js` | `/shop/a/b/c` | `{ slug: ['a', 'b', 'c'] }` |

## Optional Catch-all 세그먼트

Catch-all 세그먼트를 이중 대괄호(`[[...segmentName]]`)로 감싸면 **선택적**으로 만들 수 있습니다.

예를 들어, `pages/shop/[[...slug]].js`는 `/shop`도 매칭하며, `/shop/clothes`, `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts` 등도 모두 매칭합니다.

**라우팅 테이블:**

| 라우트 | 예시 URL | params |
|--------|----------|--------|
| `pages/shop/[[...slug]].js` | `/shop` | `{ slug: undefined }` |
| `pages/shop/[[...slug]].js` | `/shop/a` | `{ slug: ['a'] }` |
| `pages/shop/[[...slug]].js` | `/shop/a/b` | `{ slug: ['a', 'b'] }` |
| `pages/shop/[[...slug]].js` | `/shop/a/b/c` | `{ slug: ['a', 'b', 'c'] }` |

## Catch-all vs Optional Catch-all

| 유형 | 형식 | 최소 세그먼트 | params |
|------|------|-------------|--------|
| Catch-all | `[...slug]` | 1개 이상 | 항상 배열 |
| Optional Catch-all | `[[...slug]]` | 0개 이상 | 배열 또는 `undefined` |

`[...slug]`와 `[[...slug]]`의 차이점은 Optional Catch-all의 경우 파라미터 없이도 라우트가 매칭된다는 것입니다 (위 예제에서 `/shop`).

## 예제

### 블로그 포스트 페이지

```jsx filename="pages/blog/[slug].js"
import { useRouter } from 'next/router'

export default function BlogPost() {
  const router = useRouter()
  const { slug } = router.query

  return (
    <article>
      <h1>블로그 포스트: {slug}</h1>
    </article>
  )
}
```

### 제품 카테고리 페이지

```jsx filename="pages/shop/[...slug].js"
import { useRouter } from 'next/router'

export default function ProductCategory() {
  const router = useRouter()
  const { slug } = router.query

  // slug가 배열이므로 breadcrumb으로 활용 가능
  const breadcrumb = slug ? slug.join(' > ') : ''

  return (
    <div>
      <nav>카테고리: {breadcrumb}</nav>
      {/* 제품 목록 */}
    </div>
  )
}
```

### getStaticPaths와 함께 사용

동적 라우트를 빌드 시점에 미리 렌더링하려면 `getStaticPaths`를 사용합니다:

```jsx filename="pages/blog/[slug].js"
export async function getStaticPaths() {
  const posts = await fetchAllPosts()

  const paths = posts.map((post) => ({
    params: { slug: post.slug },
  }))

  return {
    paths,
    fallback: false, // 또는 'blocking', true
  }
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug)

  return {
    props: { post },
  }
}

export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

### Catch-all 라우트와 getStaticPaths

```jsx filename="pages/docs/[...slug].js"
export async function getStaticPaths() {
  return {
    paths: [
      { params: { slug: ['getting-started'] } },
      { params: { slug: ['guides', 'routing'] } },
      { params: { slug: ['api', 'reference', 'hooks'] } },
    ],
    fallback: false,
  }
}

export async function getStaticProps({ params }) {
  const { slug } = params
  const docPath = slug.join('/')
  const doc = await fetchDoc(docPath)

  return {
    props: { doc },
  }
}

export default function DocPage({ doc }) {
  return (
    <main>
      <h1>{doc.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: doc.content }} />
    </main>
  )
}
```

## 관련 문서

- [useRouter](/docs/pages/api-reference/functions/use-router) - 라우터 API 레퍼런스
- [getStaticPaths](/docs/pages/api-reference/functions/getStaticPaths) - 정적 경로 생성
- [링크 및 네비게이션](/docs/pages/building-your-application/routing/linking-and-navigating) - 페이지 간 네비게이션
