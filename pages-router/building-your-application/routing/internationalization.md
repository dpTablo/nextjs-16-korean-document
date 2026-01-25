# Internationalized Routing (국제화 라우팅)

Next.js Pages Router는 `next.config.js`에서 i18n 설정을 통해 국제화된 라우팅을 내장 지원합니다.

## 설정

```js
// next.config.js
module.exports = {
  i18n: {
    // 지원하는 모든 로케일 목록
    locales: ['en', 'ko', 'ja', 'zh'],
    // 로케일 접두사가 없을 때 사용할 기본 로케일
    defaultLocale: 'ko',
    // Accept-Language 헤더 기반 자동 로케일 감지 비활성화 (선택사항)
    localeDetection: false,
  },
}
```

## 라우팅 전략

### 서브 경로 라우팅 (기본값)

로케일이 URL 경로의 일부로 포함됩니다:

- `/` → 기본 로케일 (ko)
- `/en` → 영어
- `/ja` → 일본어
- `/zh` → 중국어

```js
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'ko', 'ja'],
    defaultLocale: 'ko',
  },
}
```

### 도메인 라우팅

다른 도메인에서 다른 로케일을 제공합니다:

```js
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'ko', 'ja'],
    defaultLocale: 'ko',
    domains: [
      {
        domain: 'example.com',
        defaultLocale: 'en',
      },
      {
        domain: 'example.ko',
        defaultLocale: 'ko',
      },
      {
        domain: 'example.jp',
        defaultLocale: 'ja',
        // 선택적으로 HTTP 전용 설정
        http: true,
      },
    ],
  },
}
```

## 자동 로케일 감지

Next.js는 `Accept-Language` 헤더와 현재 도메인을 기반으로 사용자가 선호하는 로케일을 자동으로 감지하려고 시도합니다.

```js
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'ko'],
    defaultLocale: 'ko',
    // 자동 감지 비활성화
    localeDetection: false,
  },
}
```

## 로케일 정보 접근

### getStaticProps에서

```tsx
// pages/index.tsx
import { GetStaticProps } from 'next'

export const getStaticProps: GetStaticProps = async ({ locale, locales, defaultLocale }) => {
  console.log('현재 로케일:', locale)
  console.log('사용 가능한 로케일:', locales)
  console.log('기본 로케일:', defaultLocale)

  return {
    props: {
      locale,
    },
  }
}
```

### getServerSideProps에서

```tsx
// pages/profile.tsx
import { GetServerSideProps } from 'next'

export const getServerSideProps: GetServerSideProps = async ({ locale, req }) => {
  // Accept-Language 헤더 접근
  const acceptLanguage = req.headers['accept-language']

  return {
    props: {
      locale,
      acceptLanguage,
    },
  }
}
```

### useRouter에서

```tsx
// pages/about.tsx
import { useRouter } from 'next/router'

export default function AboutPage() {
  const router = useRouter()
  const { locale, locales, defaultLocale, asPath } = router

  return (
    <div>
      <p>현재 로케일: {locale}</p>
      <p>사용 가능한 로케일: {locales?.join(', ')}</p>
      <p>기본 로케일: {defaultLocale}</p>
    </div>
  )
}
```

## 로케일 전환

### Link 컴포넌트 사용

```tsx
import Link from 'next/link'

export default function LocaleSwitcher() {
  return (
    <nav>
      <Link href="/" locale="ko">한국어</Link>
      <Link href="/" locale="en">English</Link>
      <Link href="/" locale="ja">日本語</Link>
    </nav>
  )
}
```

### 프로그래밍 방식

```tsx
import { useRouter } from 'next/router'

export default function LocaleSwitcher() {
  const router = useRouter()

  const changeLocale = (locale: string) => {
    router.push(router.pathname, router.asPath, { locale })
  }

  return (
    <div>
      <button onClick={() => changeLocale('ko')}>한국어</button>
      <button onClick={() => changeLocale('en')}>English</button>
      <button onClick={() => changeLocale('ja')}>日本語</button>
    </div>
  )
}
```

## 정적 생성 (SSG)

동적 라우트의 경우 `getStaticPaths`에서 모든 로케일에 대한 경로를 반환해야 합니다:

```tsx
// pages/posts/[slug].tsx
import { GetStaticPaths, GetStaticProps } from 'next'

export const getStaticPaths: GetStaticPaths = async ({ locales }) => {
  const posts = await getPosts()

  const paths = []
  for (const locale of locales || []) {
    for (const post of posts) {
      paths.push({
        params: { slug: post.slug },
        locale,
      })
    }
  }

  return {
    paths,
    fallback: false,
  }
}

export const getStaticProps: GetStaticProps = async ({ params, locale }) => {
  const post = await getPost(params?.slug as string, locale)

  return {
    props: {
      post,
    },
  }
}
```

## 번역 관리

### 간단한 딕셔너리 접근법

```tsx
// lib/dictionaries.ts
const dictionaries = {
  ko: () => import('../dictionaries/ko.json').then((m) => m.default),
  en: () => import('../dictionaries/en.json').then((m) => m.default),
  ja: () => import('../dictionaries/ja.json').then((m) => m.default),
}

export const getDictionary = async (locale: string) => {
  return dictionaries[locale as keyof typeof dictionaries]?.() || dictionaries.ko()
}
```

```json
// dictionaries/ko.json
{
  "greeting": "안녕하세요",
  "welcome": "환영합니다"
}
```

```json
// dictionaries/en.json
{
  "greeting": "Hello",
  "welcome": "Welcome"
}
```

### 페이지에서 사용

```tsx
// pages/index.tsx
import { GetStaticProps } from 'next'
import { getDictionary } from '../lib/dictionaries'

export const getStaticProps: GetStaticProps = async ({ locale }) => {
  const dict = await getDictionary(locale || 'ko')

  return {
    props: {
      dict,
    },
  }
}

export default function HomePage({ dict }) {
  return (
    <div>
      <h1>{dict.greeting}</h1>
      <p>{dict.welcome}</p>
    </div>
  )
}
```

## 권장 i18n 라이브러리

더 고급 기능이 필요한 경우:

- [next-intl](https://github.com/amannn/next-intl) - Pages Router와 App Router 모두 지원
- [next-i18next](https://github.com/i18next/next-i18next) - Pages Router 전용
- [react-intl](https://formatjs.io/docs/react-intl/) - 범용 React 국제화

---

## 참고

- [Link 컴포넌트](/pages-router/api-reference/components/link.md)
- [useRouter](/pages-router/api-reference/functions/use-router.md)
- [getStaticProps](/pages-router/api-reference/getStaticProps.md)
- [getStaticPaths](/pages-router/api-reference/getStaticPaths.md)
