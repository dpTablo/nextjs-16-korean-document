# 국제화 (i18n)

**버전:** 16.1.1

## 개요
Next.js는 로케일 기반 라우팅 및 현지화 패턴을 통해 여러 언어에 대한 라우팅 및 렌더링 설정을 지원합니다.

## 주요 용어
- **로케일(Locale)**: 언어 및 형식 지정 기본 설정을 위한 식별자
  - 예시: `en-US`, `nl-NL`, `nl`

---

## 라우팅 전략

### 1. **브라우저 기본 설정에서 사용자 로케일 감지**
`Accept-Language` 헤더를 사용하여 사용자의 선호 로케일을 결정:

```js filename="proxy.js"
import { match } from '@formatjs/intl-localematcher'
import Negotiator from 'negotiator'

let headers = { 'accept-language': 'en-US,en;q=0.5' }
let languages = new Negotiator({ headers }).languages()
let locales = ['en-US', 'nl-NL', 'nl']
let defaultLocale = 'en-US'

match(languages, locales, defaultLocale) // -> 'en-US'
```

### 2. **라우팅 접근 방식**
- **하위 경로 라우팅**: `/fr/products`
- **도메인 라우팅**: `my-site.fr/products`

### 3. **Middleware Proxy 설정**
```js filename="proxy.js"
import { NextResponse } from "next/server"

let locales = ['en-US', 'nl-NL', 'nl']

function getLocale(request) { ... }

export function proxy(request) {
  const { pathname } = request.nextUrl
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  )

  if (pathnameHasLocale) return

  // 로케일이 없으면 리다이렉트
  const locale = getLocale(request)
  request.nextUrl.pathname = `/${locale}${pathname}`
  return NextResponse.redirect(request.nextUrl)
}

export const config = {
  matcher: ['/((?!_next).*)', '/' ]
}
```

### 4. **동적 라우트 설정**
모든 파일을 `app/[lang]` 아래에 중첩:

```tsx filename="app/[lang]/page.tsx"
export default async function Page({ params }: PageProps<'/[lang]'>) {
  const { lang } = await params
  return ...
}
```

---

## 현지화

### **사전 기반 번역 패턴**

**1단계: 번역 사전 생성**

```json filename="dictionaries/en.json"
{
  "products": {
    "cart": "Add to Cart"
  }
}
```

```json filename="dictionaries/nl.json"
{
  "products": {
    "cart": "Toevoegen aan Winkelwagen"
  }
}
```

**2단계: 사전 로더 생성**

```ts filename="app/[lang]/dictionaries.ts"
import 'server-only'

const dictionaries = {
  en: () => import('./dictionaries/en.json').then((module) => module.default),
  nl: () => import('./dictionaries/nl.json').then((module) => module.default),
}

export type Locale = keyof typeof dictionaries

export const hasLocale = (locale: string): locale is Locale =>
  locale in dictionaries

export const getDictionary = async (locale: Locale) => dictionaries[locale]()
```

**3단계: 페이지/레이아웃에서 사용**

```tsx filename="app/[lang]/page.tsx"
import { notFound } from 'next/navigation'
import { getDictionary, hasLocale } from './dictionaries'

export default async function Page({ params }: PageProps<'/[lang]'>) {
  const { lang } = await params

  if (!hasLocale(lang)) notFound()

  const dict = await getDictionary(lang)
  return <button>{dict.products.cart}</button>
}
```

**이점:**
- 번역 파일은 서버에서만 실행됨
- 클라이언트 사이드 번들 크기에 영향 없음
- 타입 안전한 로케일 처리

---

## 정적 렌더링

`generateStaticParams`를 사용하여 특정 로케일에 대한 정적 라우트 생성:

```tsx filename="app/[lang]/layout.tsx"
export async function generateStaticParams() {
  return [{ lang: 'en-US' }, { lang: 'de' }]
}

export default async function RootLayout({
  children,
  params,
}: LayoutProps<'/[lang]'>) {
  return (
    <html lang={(await params).lang}>
      <body>{children}</body>
    </html>
  )
}
```

---

## 권장 i18n 라이브러리

- `next-intl`
- `next-international`
- `next-i18n-router`
- `paraglide-next`
- `lingui`
- `tolgee`
- `next-intlayer`
- `gt-next`

**예시 저장소**: [Minimal i18n routing and translations](https://github.com/vercel/next.js/tree/canary/examples/i18n-routing)
