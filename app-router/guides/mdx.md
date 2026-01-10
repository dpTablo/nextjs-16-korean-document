# MDX

Next.js에서 MDX를 사용하여 마크다운에 React 컴포넌트를 포함시키는 방법을 알아봅니다.

## 개요

**Markdown**은 텍스트를 HTML로 포맷팅하기 위한 경량 마크업 언어입니다.

**MDX**는 마크다운 파일에서 직접 JSX를 작성할 수 있게 해주는 Markdown의 상위 집합입니다. 동적 상호작용과 React 컴포넌트 임베딩을 가능하게 합니다. Next.js는 로컬 및 원격 MDX 콘텐츠를 모두 지원하며, Server Components를 완전히 지원합니다.

---

## 설치 및 설정

### 1. 의존성 설치

```bash
npm install @next/mdx @mdx-js/loader @mdx-js/react @types/mdx
```

### 2. `next.config.mjs` 구성

```js
import createMDX from '@next/mdx'

/** @type {import('next').NextConfig} */
const nextConfig = {
  // .mdx 파일을 페이지로 사용할 수 있도록 확장자 추가
  pageExtensions: ['js', 'jsx', 'md', 'mdx', 'ts', 'tsx'],
}

const withMDX = createMDX({
  // 여기에 마크다운 플러그인 추가
})

export default withMDX(nextConfig)
```

**`.md` 파일도 지원하려면:**
```js
const withMDX = createMDX({
  extension: /\.(md|mdx)$/,
})
```

### 3. `mdx-components.tsx` 생성

**App Router에서 필수** - 프로젝트 루트(`app` 또는 `pages`와 동일한 레벨)에 생성:

```tsx
import type { MDXComponents } from 'mdx/types'

const components: MDXComponents = {}

export function useMDXComponents(): MDXComponents {
  return components
}
```

> **알아두면 좋은 점:**
> - 이 파일은 App Router를 사용할 때 **필수**입니다.
> - 여기서 전역 MDX 컴포넌트 스타일을 정의할 수 있습니다.

---

## MDX 렌더링 방법

### 방법 1: 파일 기반 라우팅

MDX 파일을 페이지로 직접 사용:

```
my-project/
├── app/
│   └── mdx-page/
│       └── page.mdx
├── mdx-components.tsx
└── package.json
```

**app/mdx-page/page.mdx**
```mdx
import { MyComponent } from '@/components/my-component'

# MDX 페이지에 오신 것을 환영합니다!

이것은 **굵은** 텍스트와 _기울임_ 텍스트입니다.

## 목록

- 하나
- 둘
- 셋

## 컴포넌트

<MyComponent />
```

### 방법 2: MDX 파일 가져오기

페이지를 생성하고 MDX를 별도로 가져오기:

```
app/
├── mdx-page/
│   └── page.tsx
markdown/
└── welcome.mdx
```

**app/mdx-page/page.tsx**
```tsx
import Welcome from '@/markdown/welcome.mdx'

export default function Page() {
  return <Welcome />
}
```

**markdown/welcome.mdx**
```mdx
# 환영합니다!

Next.js와 MDX로 만든 페이지입니다.
```

### 방법 3: 동적 가져오기

동적 라우트 세그먼트를 위한 방법:

**app/blog/[slug]/page.tsx**
```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params

  // 동적으로 MDX 파일 가져오기
  const { default: Post } = await import(`@/content/${slug}.mdx`)

  return <Post />
}

// 정적 파라미터 생성
export function generateStaticParams() {
  return [
    { slug: 'welcome' },
    { slug: 'about' },
    { slug: 'getting-started' },
  ]
}

// 정의되지 않은 슬러그 방지
export const dynamicParams = false
```

---

## 커스텀 스타일 및 컴포넌트

### 전역 스타일 (mdx-components.tsx)

모든 MDX 페이지에서 사용할 전역 컴포넌트 스타일을 정의합니다:

```tsx
import type { MDXComponents } from 'mdx/types'
import Image from 'next/image'

const components = {
  // 제목 커스터마이징
  h1: ({ children }) => (
    <h1 style={{ color: 'red', fontSize: '48px' }}>{children}</h1>
  ),
  h2: ({ children }) => (
    <h2 style={{ color: 'blue', fontSize: '36px' }}>{children}</h2>
  ),
  // 이미지 최적화
  img: (props) => (
    <Image
      sizes="100vw"
      style={{ width: '100%', height: 'auto' }}
      {...(props as any)}
    />
  ),
  // 링크 커스터마이징
  a: ({ href, children }) => (
    <a href={href} style={{ color: 'blue', textDecoration: 'underline' }}>
      {children}
    </a>
  ),
} satisfies MDXComponents

export function useMDXComponents(): MDXComponents {
  return components
}
```

### 로컬 스타일 (페이지별 오버라이드)

특정 페이지에서만 다른 스타일을 적용:

```tsx
import Welcome from '@/markdown/welcome.mdx'

function CustomH1({ children }: { children: React.ReactNode }) {
  return <h1 style={{ color: 'blue', fontSize: '100px' }}>{children}</h1>
}

export default function Page() {
  return <Welcome components={{ h1: CustomH1 }} />
}
```

### 공유 레이아웃

모든 MDX 페이지에 공통 레이아웃 적용:

**app/mdx-page/layout.tsx**
```tsx
export default function MdxLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <div style={{ color: 'blue' }}>
        {children}
      </div>
    </div>
  )
}
```

### Tailwind Typography 플러그인 사용

아름다운 타이포그래피를 위한 Tailwind CSS 플러그인:

**설치:**
```bash
npm install @tailwindcss/typography
```

**tailwind.config.js**
```js
module.exports = {
  plugins: [
    require('@tailwindcss/typography'),
  ],
}
```

**레이아웃 적용:**
```tsx
export default function MdxLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="prose prose-lg prose-headings:mt-8 prose-headings:font-semibold prose-h1:text-5xl prose-h2:text-4xl prose-a:text-blue-600 dark:prose-invert dark:prose-headings:text-white">
      {children}
    </div>
  )
}
```

---

## Frontmatter와 메타데이터

MDX는 기본적으로 frontmatter를 지원하지 않지만, JavaScript export를 사용할 수 있습니다:

**content/blog-post.mdx**
```mdx
export const metadata = {
  title: '나의 블로그 포스트',
  author: 'John Doe',
  date: '2025-01-01',
  tags: ['nextjs', 'mdx', 'react'],
}

# {metadata.title}

작성자: {metadata.author}
날짜: {metadata.date}

블로그 포스트 내용...
```

**메타데이터 접근:**
```tsx
import BlogPost, { metadata } from '@/content/blog-post.mdx'

export default function Page() {
  console.log(metadata)
  // => { title: '나의 블로그 포스트', author: 'John Doe', ... }

  return (
    <article>
      <h1>{metadata.title}</h1>
      <p>작성자: {metadata.author}</p>
      <p>날짜: {metadata.date}</p>
      <BlogPost />
    </article>
  )
}
```

### generateMetadata와 함께 사용

```tsx
import { Metadata } from 'next'
import BlogPost, { metadata as postMetadata } from '@/content/blog-post.mdx'

export const metadata: Metadata = {
  title: postMetadata.title,
  description: postMetadata.description,
}

export default function Page() {
  return <BlogPost />
}
```

---

## 플러그인: Remark 및 Rehype

### 기본 플러그인 설정

**Remark**: Markdown AST를 처리하는 플러그인
**Rehype**: HTML AST를 처리하는 플러그인

```bash
npm install remark-gfm rehype-slug rehype-autolink-headings
```

**next.config.mjs**
```js
import remarkGfm from 'remark-gfm'
import rehypeSlug from 'rehype-slug'
import rehypeAutolinkHeadings from 'rehype-autolink-headings'
import createMDX from '@next/mdx'

const withMDX = createMDX({
  options: {
    remarkPlugins: [remarkGfm], // GitHub Flavored Markdown 지원
    rehypePlugins: [
      rehypeSlug, // 제목에 ID 추가
      rehypeAutolinkHeadings, // 제목에 링크 추가
    ],
  },
})

export default withMDX(nextConfig)
```

### Turbopack과 함께 사용 (문자열 기반)

```js
const withMDX = createMDX({
  options: {
    remarkPlugins: [
      'remark-gfm',
      ['remark-toc', { heading: '목차' }], // 목차 자동 생성
    ],
    rehypePlugins: [
      'rehype-slug',
      ['rehype-katex', { strict: true }], // 수학 수식 지원
    ],
  },
})
```

### 인기 있는 플러그인

**Remark 플러그인:**
- `remark-gfm`: GitHub Flavored Markdown (테이블, 체크리스트 등)
- `remark-toc`: 목차 자동 생성
- `remark-math`: 수학 표기법 지원
- `remark-emoji`: 이모지 지원

**Rehype 플러그인:**
- `rehype-slug`: 제목에 ID 자동 추가
- `rehype-autolink-headings`: 제목에 링크 추가
- `rehype-katex`: 수학 수식 렌더링
- `rehype-prism`: 코드 구문 강조

---

## 원격 MDX

CMS, 데이터베이스 또는 원격 소스에 저장된 MDX:

```bash
npm install next-mdx-remote-client
```

**app/remote/page.tsx**
```tsx
import { MDXRemote } from 'next-mdx-remote-client/rsc'

export default async function RemoteMdxPage() {
  // API나 CMS에서 MDX 가져오기
  const res = await fetch('https://api.example.com/content')
  const markdown = await res.text()

  return <MDXRemote source={markdown} />
}
```

### 커스텀 컴포넌트와 함께 사용

```tsx
import { MDXRemote } from 'next-mdx-remote-client/rsc'
import { CustomComponent } from '@/components/custom'

const components = {
  CustomComponent,
}

export default async function RemoteMdxPage() {
  const res = await fetch('https://api.example.com/content')
  const markdown = await res.text()

  return <MDXRemote source={markdown} components={components} />
}
```

> **⚠️ 보안 주의사항:**
> - 신뢰할 수 있는 소스에서만 MDX를 가져오세요.
> - 원격 코드 실행(RCE) 위험을 방지하기 위해 사용자 입력을 직접 MDX로 렌더링하지 마세요.

---

## 실험적 Rust 기반 컴파일러

더 빠른 MDX 컴파일을 위한 Rust 기반 컴파일러:

**next.config.js**
```js
module.exports = withMDX({
  experimental: {
    mdxRs: {
      jsxRuntime: 'automatic',
      mdxType: 'gfm', // 'gfm' | 'commonmark'
    },
  },
})
```

> **참고:** 실험적 기능이므로 프로덕션에서는 주의해서 사용하세요.

---

## Markdown이 HTML로 변환되는 방법

`@next/mdx`는 내부적으로 **remark**와 **rehype**를 사용합니다:

```js
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'

async function transformMarkdown(markdown) {
  const file = await unified()
    .use(remarkParse)        // Markdown → Markdown AST
    .use(remarkRehype)       // Markdown AST → HTML AST
    .use(rehypeStringify)    // HTML AST → HTML 문자열
    .process(markdown)

  return String(file)
}
```

### 처리 파이프라인

```
Markdown 텍스트
    ↓ remarkParse
Markdown AST (mdast)
    ↓ remarkPlugins
수정된 Markdown AST
    ↓ remarkRehype
HTML AST (hast)
    ↓ rehypePlugins
수정된 HTML AST
    ↓ rehypeStringify
HTML 문자열
```

---

## 실전 예시

### 블로그 시스템 구축

**content/posts/first-post.mdx**
```mdx
export const metadata = {
  title: '첫 번째 포스트',
  date: '2025-01-11',
  author: 'John Doe',
}

# {metadata.title}

이것은 MDX로 작성된 블로그 포스트입니다.

<Callout type="info">
  MDX에서는 이렇게 React 컴포넌트를 사용할 수 있습니다!
</Callout>
```

**app/blog/[slug]/page.tsx**
```tsx
import { notFound } from 'next/navigation'

export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params

  try {
    const { default: Post, metadata } = await import(
      `@/content/posts/${slug}.mdx`
    )

    return (
      <article className="prose lg:prose-xl">
        <h1>{metadata.title}</h1>
        <p>작성자: {metadata.author} | {metadata.date}</p>
        <Post />
      </article>
    )
  } catch {
    notFound()
  }
}
```

---

## 베스트 프랙티스

### 1. **타입 안전성 확보**

```tsx
// types/mdx.ts
export interface BlogMetadata {
  title: string
  date: string
  author: string
  tags: string[]
  description: string
}

// content/post.mdx
import type { BlogMetadata } from '@/types/mdx'

export const metadata: BlogMetadata = {
  title: '...',
  date: '...',
  author: '...',
  tags: [],
  description: '...',
}
```

### 2. **재사용 가능한 컴포넌트 생성**

```tsx
// components/mdx/Callout.tsx
export function Callout({
  type = 'info',
  children,
}: {
  type?: 'info' | 'warning' | 'error'
  children: React.ReactNode
}) {
  const styles = {
    info: 'bg-blue-100 border-blue-500',
    warning: 'bg-yellow-100 border-yellow-500',
    error: 'bg-red-100 border-red-500',
  }

  return (
    <div className={`border-l-4 p-4 ${styles[type]}`}>
      {children}
    </div>
  )
}

// mdx-components.tsx
import { Callout } from '@/components/mdx/Callout'

export function useMDXComponents(): MDXComponents {
  return {
    Callout,
  }
}
```

### 3. **코드 구문 강조**

```bash
npm install rehype-prism-plus
```

```js
import rehypePrism from 'rehype-prism-plus'

const withMDX = createMDX({
  options: {
    rehypePlugins: [rehypePrism],
  },
})
```

---

## 다음 단계

- [MDX 공식 문서](https://mdxjs.com/)
- [@next/mdx 패키지](https://www.npmjs.com/package/@next/mdx)
- [Remark 플러그인 생태계](https://github.com/remarkjs/remark)
- [Rehype 플러그인 생태계](https://github.com/rehypejs/rehype)
- [Portfolio Starter Kit](https://vercel.com/templates/next.js/portfolio-starter-kit) - MDX를 사용한 예시

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-11
