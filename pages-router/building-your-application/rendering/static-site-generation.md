# 정적 사이트 생성 (SSG)

정적 생성(Static Generation)을 사용하면 페이지 HTML이 **빌드 시간**에 생성됩니다. 프로덕션에서 `next build`를 실행할 때 페이지 HTML이 생성됩니다. 이 HTML은 각 요청에서 재사용되며 CDN에서 캐싱할 수 있습니다.

Next.js에서는 **데이터가 있는** 페이지와 **데이터가 없는** 페이지를 정적으로 생성할 수 있습니다.

## 데이터 없는 정적 생성

기본적으로 Next.js는 데이터를 가져오지 않고 정적 생성을 사용하여 페이지를 사전 렌더링합니다:

```jsx
function About() {
  return <div>About</div>
}

export default About
```

이 페이지는 외부 데이터를 가져올 필요가 없습니다. 이 경우 Next.js는 빌드 시간에 페이지당 하나의 HTML 파일을 생성합니다.

## 데이터 있는 정적 생성

일부 페이지는 사전 렌더링을 위해 외부 데이터를 가져와야 합니다. 두 가지 시나리오가 있으며, 하나 또는 둘 다 적용될 수 있습니다. 각 경우에 Next.js가 제공하는 함수를 사용할 수 있습니다:

### 시나리오 1: 페이지 콘텐츠가 외부 데이터에 의존하는 경우

`getStaticProps`를 사용합니다.

**예시:** 블로그 페이지가 CMS에서 블로그 게시물 목록을 가져와야 할 수 있습니다.

```jsx
export default function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}

// 이 함수는 빌드 시간에 서버에서 호출됩니다
export async function getStaticProps() {
  // 외부 API 엔드포인트를 호출하여 게시물을 가져옵니다
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  // { props: { posts } }를 반환하면
  // Blog 컴포넌트는 빌드 시간에 `posts`를 prop으로 받습니다
  return {
    props: {
      posts,
    },
  }
}
```

### 시나리오 2: 페이지 경로가 외부 데이터에 의존하는 경우

`getStaticPaths`를 사용합니다 (보통 `getStaticProps`와 함께).

**예시:** 동적 라우트를 사용하여 `id`를 기반으로 하나의 블로그 게시물을 보여주는 페이지 (`pages/posts/[id].js`)

```jsx
// pages/posts/[id].js
export default function Post({ post }) {
  // 게시물 렌더링...
}

// 이 함수는 빌드 시간에 호출됩니다
export async function getStaticPaths() {
  // 외부 API 엔드포인트를 호출하여 게시물을 가져옵니다
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  // 게시물을 기반으로 사전 렌더링할 경로를 가져옵니다
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))

  // 빌드 시간에 이 경로들만 사전 렌더링합니다
  // { fallback: false }는 다른 라우트는 404가 됨을 의미합니다
  return { paths, fallback: false }
}

// 빌드 시간에도 호출됩니다
export async function getStaticProps({ params }) {
  // params에는 게시물 `id`가 포함됩니다
  // 라우트가 /posts/1이면 params.id는 1입니다
  const res = await fetch(`https://.../posts/${params.id}`)
  const post = await res.json()

  // props를 통해 페이지에 게시물 데이터 전달
  return { props: { post } }
}
```

## 정적 생성을 사용해야 하는 경우

가능하면 **정적 생성**을 사용하는 것을 권장합니다. 페이지를 한 번 빌드하고 CDN에서 제공할 수 있기 때문입니다. 이는 서버가 모든 요청마다 페이지를 렌더링하는 것보다 훨씬 빠릅니다.

다음과 같은 페이지에 정적 생성을 사용할 수 있습니다:

- 마케팅 페이지
- 블로그 게시물 및 포트폴리오
- E-commerce 제품 목록
- 도움말 및 문서

스스로에게 물어보세요: "사용자 요청 **이전에** 이 페이지를 미리 렌더링할 수 있는가?" 답이 예라면 정적 생성을 선택해야 합니다.

## 정적 생성이 적합하지 않은 경우

반면에 사용자 요청 이전에 페이지를 미리 렌더링할 수 없다면 정적 생성은 좋은 아이디어가 아닙니다. 페이지가 자주 업데이트되는 데이터를 보여주고 페이지 콘텐츠가 모든 요청마다 변경될 수 있습니다.

이런 경우 다음 중 하나를 수행할 수 있습니다:

### 1. 클라이언트 사이드 데이터 페칭과 함께 정적 생성 사용

페이지의 일부를 미리 렌더링하지 않고 클라이언트 사이드 JavaScript를 사용하여 채울 수 있습니다. 이 접근 방식에 대해 더 알아보려면 [클라이언트 사이드 데이터 페칭](/pages-router/building-your-application/data-fetching/client-side.md) 문서를 확인하세요.

### 2. 서버 사이드 렌더링 사용

Next.js는 모든 요청마다 페이지를 사전 렌더링합니다. CDN에서 캐싱할 수 없기 때문에 더 느리지만 사전 렌더링된 페이지는 항상 최신 상태입니다. 이 접근 방식에 대해서는 [서버 사이드 렌더링](/pages-router/building-your-application/rendering/server-side-rendering.md) 문서를 확인하세요.
