# 메타데이터와 OG 이미지 (Metadata and OG Images)

Metadata API는 애플리케이션 메타데이터를 정의하여 SEO 및 웹 공유성을 개선하기 위해 사용됩니다. 다음 세 가지 방법을 포함합니다:

1. 정적 `metadata` 객체
2. 동적 `generateMetadata` 함수
3. 정적 또는 동적으로 생성된 파비콘과 OG 이미지를 추가할 수 있는 특수 파일 규칙

Next.js는 자동으로 해당 `<head>` 태그를 생성하며, 브라우저 개발자 도구에서 검사할 수 있습니다.

> **참고:** `metadata` 객체와 `generateMetadata` 함수 exports는 **서버 컴포넌트**에서만 지원됩니다.

## 기본 필드

라우트가 메타데이터를 정의하지 않아도 항상 추가되는 두 가지 기본 `meta` 태그가 있습니다:

- **charset 태그**: 웹사이트의 문자 인코딩 설정
- **viewport 태그**: 다양한 기기에 맞게 조정되는 뷰포트 너비 및 스케일 설정

```html
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

## 정적 메타데이터 (Static Metadata)

정적 메타데이터를 정의하려면 정적 `layout.js` 또는 `page.js` 파일에서 `Metadata` 객체를 export합니다.

### 예제: 블로그 라우트에 제목 및 설명 추가

```tsx filename="app/blog/layout.tsx"
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: '내 블로그',
  description: '...',
}

export default function Layout() {}
```

```jsx filename="app/blog/layout.js"
export const metadata = {
  title: '내 블로그',
  description: '...',
}

export default function Layout() {}
```

더 많은 옵션은 [`generateMetadata` 문서](/app-router/api-reference/functions/generateMetadata.md#metadata-fields)에서 확인할 수 있습니다.

## 생성된 메타데이터 (Generated Metadata)

`generateMetadata` 함수를 사용하여 데이터에 따라 메타데이터를 `fetch`할 수 있습니다.

### 예제: 특정 블로그 포스트의 제목 및 설명 가져오기

```tsx filename="app/blog/[slug]/page.tsx"
import type { Metadata, ResolvingMetadata } from 'next'

type Props = {
  params: Promise<{ slug: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}

export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  const slug = (await params).slug

  // 포스트 정보 가져오기
  const post = await fetch(`https://api.vercel.app/blog/${slug}`).then((res) =>
    res.json()
  )

  return {
    title: post.title,
    description: post.description,
  }
}

export default function Page({ params, searchParams }: Props) {}
```

```jsx filename="app/blog/[slug]/page.js"
export async function generateMetadata({ params, searchParams }, parent) {
  const slug = (await params).slug

  // 포스트 정보 가져오기
  const post = await fetch(`https://api.vercel.app/blog/${slug}`).then((res) =>
    res.json()
  )

  return {
    title: post.title,
    description: post.description,
  }
}

export default function Page({ params, searchParams }) {}
```

## 스트리밍 메타데이터 (Streaming Metadata)

동적으로 렌더링되는 페이지의 경우, Next.js는 메타데이터를 별도로 스트리밍하고 `generateMetadata`가 해결되면 HTML에 주입합니다. UI 렌더링을 차단하지 않습니다.

**장점**: 스트리밍 메타데이터는 시각적 콘텐츠를 먼저 스트리밍하여 인지된 성능을 개선합니다.

### 봇 및 크롤러

스트리밍 메타데이터는 **봇 및 크롤러에 대해 비활성화됩니다** (예: `Twitterbot`, `Slackbot`, `Bingbot`). 이들은 User Agent 헤더를 통해 감지됩니다.

`next.config.js`의 [`htmlLimitedBots`](/app-router/api-reference/config/next-config-js/htmlLimitedBots.md) 옵션으로 스트리밍 메타데이터를 커스터마이징하거나 비활성화할 수 있습니다.

정적으로 렌더링된 페이지는 스트리밍을 사용하지 않습니다 (메타데이터는 빌드 시간에 해결됨).

## 데이터 요청 메모이제이션 (Memoizing Data Requests)

메타데이터와 페이지 자체에 대해 **동일한** 데이터를 가져와야 하는 경우가 있을 수 있습니다. 중복 요청을 방지하려면 React의 `cache` 함수를 사용하여 반환 값을 메모이제이션하고 데이터를 한 번만 가져올 수 있습니다.

### 예제: 메타데이터와 페이지 모두에서 블로그 포스트 정보 가져오기

```ts filename="app/lib/data.ts"
import { cache } from 'react'
import { db } from '@/app/lib/db'

// getPost는 두 번 사용되지만 한 번만 실행됨
export const getPost = cache(async (slug: string) => {
  const res = await db.query.posts.findFirst({ where: eq(posts.slug, slug) })
  return res
})
```

```tsx filename="app/blog/[slug]/page.tsx"
import { getPost } from '@/app/lib/data'

export async function generateMetadata({
  params,
}: {
  params: { slug: string }
}) {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.description,
  }
}

export default async function Page({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  return <div>{post.title}</div>
}
```

```js filename="app/lib/data.js"
import { cache } from 'react'
import { db } from '@/app/lib/db'

// getPost는 두 번 사용되지만 한 번만 실행됨
export const getPost = cache(async (slug) => {
  const res = await db.query.posts.findFirst({ where: eq(posts.slug, slug) })
  return res
})
```

```jsx filename="app/blog/[slug]/page.js"
import { getPost } from '@/app/lib/data'

export async function generateMetadata({ params }) {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.description,
  }
}

export default async function Page({ params }) {
  const post = await getPost(params.slug)
  return <div>{post.title}</div>
}
```

## 파일 기반 메타데이터 (File-Based Metadata)

다음 특수 파일들을 메타데이터로 사용할 수 있습니다:

- `favicon.ico`, `apple-icon.jpg`, `icon.jpg`
- `opengraph-image.jpg`, `twitter-image.jpg`
- `robots.txt`
- `sitemap.xml`

정적 메타데이터로 사용하거나 코드로 프로그래밍 방식으로 생성할 수 있습니다.

## 파비콘 (Favicons)

파비콘은 북마크와 검색 결과에서 사이트를 나타내는 작은 아이콘입니다. 애플리케이션에 파비콘을 추가하려면 `favicon.ico`를 생성하고 앱 폴더의 루트에 배치합니다.

```
app/
├── favicon.ico
├── layout.tsx
└── page.tsx
```

> **참고**: 코드로 프로그래밍 방식으로 파비콘을 생성할 수도 있습니다. 자세한 내용은 [파비콘 문서](/app-router/api-reference/file-conventions/metadata/app-icons.md)를 참조하세요.

## 정적 Open Graph 이미지 (Static Open Graph Images)

Open Graph (OG) 이미지는 소셜 미디어에서 사이트를 나타내는 이미지입니다. 애플리케이션에 정적 OG 이미지를 추가하려면 앱 폴더의 루트에 `opengraph-image.jpg` 파일을 생성합니다.

```
app/
├── opengraph-image.jpg
├── layout.tsx
└── page.tsx
```

### 특정 라우트에 OG 이미지 추가

폴더 구조 내에 `opengraph-image.jpg`를 배치하여 특정 라우트에 OG 이미지를 추가할 수 있습니다. 예를 들어, `/blog` 라우트에 특정 OG 이미지를 생성하려면 `blog` 폴더 내에 `opengraph-image.jpg` 파일을 추가합니다.

```
app/
├── blog/
│   └── opengraph-image.jpg
├── opengraph-image.jpg
├── layout.tsx
└── page.tsx
```

더 구체적인 이미지는 폴더 구조에서 위의 OG 이미지보다 우선됩니다.

### 지원되는 형식

`jpeg`, `png`, `gif` 등 다른 이미지 형식도 지원됩니다. 자세한 내용은 [Open Graph Image 문서](/app-router/api-reference/file-conventions/metadata/opengraph-image.md)를 참조하세요.

## 생성된 Open Graph 이미지 (Generated Open Graph Images)

`ImageResponse` 생성자를 사용하면 JSX 및 CSS를 사용하여 동적 이미지를 생성할 수 있습니다. 이는 데이터에 따라 달라지는 OG 이미지에 유용합니다.

### 예제: 각 블로그 포스트에 대한 고유한 OG 이미지 생성

```tsx filename="app/blog/[slug]/opengraph-image.tsx"
import { ImageResponse } from 'next/og'
import { getPost } from '@/app/lib/data'

// 이미지 메타데이터
export const size = {
  width: 1200,
  height: 630,
}

export const contentType = 'image/png'

// 이미지 생성
export default async function Image({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)

  return new ImageResponse(
    (
      // ImageResponse JSX 요소
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        {post.title}
      </div>
    )
  )
}
```

```jsx filename="app/blog/[slug]/opengraph-image.js"
import { ImageResponse } from 'next/og'
import { getPost } from '@/app/lib/data'

// 이미지 메타데이터
export const size = {
  width: 1200,
  height: 630,
}

export const contentType = 'image/png'

// 이미지 생성
export default async function Image({ params }) {
  const post = await getPost(params.slug)

  return new ImageResponse(
    (
      // ImageResponse JSX 요소
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        {post.title}
      </div>
    )
  )
}
```

### ImageResponse의 CSS 지원

`ImageResponse`는 flexbox 및 절대 위치 지정, 사용자 정의 글꼴, 텍스트 줄바꿈, 중앙 정렬, 중첩 이미지 등 일반적인 CSS 속성을 지원합니다.

> **알아두면 좋은 점:**
>
> - 예제는 [Vercel OG Playground](https://og-playground.vercel.app/)에서 확인할 수 있습니다.
> - `ImageResponse`는 [`@vercel/og`](https://vercel.com/docs/og-image-generation), [`satori`](https://github.com/vercel/satori), `resvg`를 사용하여 HTML과 CSS를 PNG로 변환합니다.
> - Flexbox와 CSS 속성의 일부만 지원됩니다. 고급 레이아웃 (예: `display: grid`)은 작동하지 않습니다.

## API 참조

이 페이지에서 언급된 Metadata API에 대해 자세히 알아보세요:

- [generateMetadata](/app-router/api-reference/functions/generateMetadata.md) - Next.js 애플리케이션에 메타데이터를 추가하여 검색 엔진 최적화(SEO) 및 웹 공유성을 개선하는 방법
- [generateViewport](/app-router/api-reference/functions/generateViewport.md) - generateViewport 함수의 API 참조
- [ImageResponse](/app-router/api-reference/functions/ImageResponse.md) - ImageResponse 생성자의 API 참조
- [favicon, icon, and apple-icon](/app-router/api-reference/file-conventions/metadata/app-icons.md) - Favicon, Icon, Apple Icon 파일 규칙의 API 참조
- [opengraph-image and twitter-image](/app-router/api-reference/file-conventions/metadata/opengraph-image.md) - Open Graph Image 및 Twitter Image 파일 규칙의 API 참조
- [robots.txt](/app-router/api-reference/file-conventions/metadata/robots.md) - robots.txt 파일의 API 참조
- [sitemap.xml](/app-router/api-reference/file-conventions/metadata/sitemap.md) - sitemap.xml 파일의 API 참조
