# JSON-LD

검색 엔진이 페이지 콘텐츠를 이해할 수 있도록 구조화된 데이터를 추가하는 방법을 알아봅니다.

## 개요

[JSON-LD](https://json-ld.org/)는 검색 엔진과 AI가 콘텐츠를 넘어 페이지의 구조를 이해하는 데 사용하는 구조화된 데이터 형식입니다. 사람, 이벤트, 조직, 영화, 책, 레시피 등의 엔티티를 설명할 수 있습니다.

**JSON-LD의 장점:**
- 검색 엔진 최적화 (SEO) 개선
- 리치 결과 (Rich Results) 표시 가능
- AI 시스템의 콘텐츠 이해도 향상

---

## 권장 접근 방식

`layout.js` 또는 `page.js` 컴포넌트에서 `<script>` 태그로 구조화된 데이터를 렌더링합니다.

```tsx
export default async function Page({ params }) {
  const { id } = await params
  const product = await getProduct(id)

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    image: product.image,
    description: product.description,
  }

  return (
    <section>
      {/* 구조화된 데이터 추가 */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
      {/* 페이지 콘텐츠 */}
    </section>
  )
}
```

---

## 보안 고려사항

`JSON.stringify()`는 XSS 인젝션에 대한 악성 문자열을 자동으로 살균(sanitize)하지 않습니다.

### 권장 솔루션

`<` 문자를 유니코드 동등값으로 대체합니다:

```tsx
JSON.stringify(jsonLd).replace(/</g, '\\u003c')
```

### 대안

커뮤니티에서 관리하는 패키지를 사용할 수 있습니다:

```bash
npm install serialize-javascript
```

```tsx
import serialize from 'serialize-javascript'

<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: serialize(jsonLd, { isJSON: true }),
  }}
/>
```

---

## 구현 예시

### 제품 페이지

**app/products/[id]/page.tsx**
```tsx
import { getProduct } from '@/lib/products'

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await getProduct(id)

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    image: product.image,
    description: product.description,
    brand: {
      '@type': 'Brand',
      name: product.brand,
    },
    offers: {
      '@type': 'Offer',
      price: product.price,
      priceCurrency: 'KRW',
      availability: product.inStock
        ? 'https://schema.org/InStock'
        : 'https://schema.org/OutOfStock',
    },
  }

  return (
    <section>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>가격: {product.price.toLocaleString()}원</p>
    </section>
  )
}
```

### 블로그 게시물

**app/blog/[slug]/page.tsx**
```tsx
import { getPost } from '@/lib/blog'

export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await getPost(slug)

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    headline: post.title,
    description: post.excerpt,
    author: {
      '@type': 'Person',
      name: post.author.name,
    },
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    image: post.coverImage,
    publisher: {
      '@type': 'Organization',
      name: '회사명',
      logo: {
        '@type': 'ImageObject',
        url: 'https://example.com/logo.png',
      },
    },
  }

  return (
    <article>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  )
}
```

### 조직 정보 (사이트 전체)

**app/layout.tsx**
```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const organizationJsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: '회사명',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    sameAs: [
      'https://twitter.com/company',
      'https://facebook.com/company',
      'https://linkedin.com/company/company',
    ],
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+82-2-1234-5678',
      contactType: 'customer service',
      areaServed: 'KR',
      availableLanguage: ['Korean', 'English'],
    },
  }

  return (
    <html lang="ko">
      <body>
        <script
          type="application/ld+json"
          dangerouslySetInnerHTML={{
            __html: JSON.stringify(organizationJsonLd).replace(/</g, '\\u003c'),
          }}
        />
        {children}
      </body>
    </html>
  )
}
```

### FAQ 페이지

**app/faq/page.tsx**
```tsx
const faqs = [
  {
    question: '배송은 얼마나 걸리나요?',
    answer: '일반적으로 2-3 영업일 내에 배송됩니다.',
  },
  {
    question: '반품은 어떻게 하나요?',
    answer: '구매일로부터 14일 이내에 반품 신청이 가능합니다.',
  },
]

export default function FAQPage() {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map((faq) => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  }

  return (
    <section>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
      <h1>자주 묻는 질문</h1>
      {faqs.map((faq, index) => (
        <div key={index}>
          <h2>{faq.question}</h2>
          <p>{faq.answer}</p>
        </div>
      ))}
    </section>
  )
}
```

### 이벤트 정보

**app/events/[id]/page.tsx**
```tsx
export default async function EventPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const event = await getEvent(id)

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Event',
    name: event.name,
    description: event.description,
    startDate: event.startDate,
    endDate: event.endDate,
    location: {
      '@type': 'Place',
      name: event.venue,
      address: {
        '@type': 'PostalAddress',
        streetAddress: event.address,
        addressLocality: event.city,
        addressCountry: 'KR',
      },
    },
    image: event.image,
    offers: {
      '@type': 'Offer',
      price: event.price,
      priceCurrency: 'KRW',
      availability: 'https://schema.org/InStock',
      url: `https://example.com/events/${id}/tickets`,
    },
    organizer: {
      '@type': 'Organization',
      name: event.organizerName,
      url: event.organizerUrl,
    },
  }

  return (
    <section>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
      <h1>{event.name}</h1>
      <p>{event.description}</p>
    </section>
  )
}
```

---

## TypeScript 타이핑

`schema-dts` 패키지를 사용하면 타입 안전성을 확보할 수 있습니다.

### 설치

```bash
npm install schema-dts
```

### 사용 예시

```tsx
import { Product, WithContext } from 'schema-dts'

export default function ProductPage() {
  const jsonLd: WithContext<Product> = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: 'Next.js 스티커',
    image: 'https://nextjs.org/imgs/sticker.png',
    description: '정적 속도로 동적인 경험을 제공합니다.',
  }

  return (
    <section>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
    </section>
  )
}
```

### 다양한 스키마 타입

```tsx
import {
  Article,
  BlogPosting,
  BreadcrumbList,
  Event,
  FAQPage,
  Organization,
  Person,
  Product,
  WebPage,
  WithContext,
} from 'schema-dts'

// 블로그 게시물
const blogPost: WithContext<BlogPosting> = {
  '@context': 'https://schema.org',
  '@type': 'BlogPosting',
  headline: '글 제목',
  // ...
}

// FAQ 페이지
const faq: WithContext<FAQPage> = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  mainEntity: [],
}

// 브레드크럼
const breadcrumb: WithContext<BreadcrumbList> = {
  '@context': 'https://schema.org',
  '@type': 'BreadcrumbList',
  itemListElement: [
    {
      '@type': 'ListItem',
      position: 1,
      name: '홈',
      item: 'https://example.com',
    },
    {
      '@type': 'ListItem',
      position: 2,
      name: '블로그',
      item: 'https://example.com/blog',
    },
  ],
}
```

---

## 유틸리티 함수

재사용 가능한 유틸리티 함수를 만들 수 있습니다.

**lib/json-ld.tsx**
```tsx
import { Thing, WithContext } from 'schema-dts'

interface JsonLdProps<T extends Thing> {
  data: WithContext<T>
}

export function JsonLd<T extends Thing>({ data }: JsonLdProps<T>) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data).replace(/</g, '\\u003c'),
      }}
    />
  )
}
```

**사용:**
```tsx
import { JsonLd } from '@/lib/json-ld'
import { Product, WithContext } from 'schema-dts'

export default function ProductPage() {
  const productData: WithContext<Product> = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: '제품명',
    // ...
  }

  return (
    <section>
      <JsonLd data={productData} />
      {/* 페이지 콘텐츠 */}
    </section>
  )
}
```

---

## 여러 스키마 결합

한 페이지에 여러 JSON-LD 스키마를 포함할 수 있습니다.

```tsx
export default function ProductPage() {
  const productJsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: '제품명',
    // ...
  }

  const breadcrumbJsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: [
      { '@type': 'ListItem', position: 1, name: '홈', item: 'https://example.com' },
      { '@type': 'ListItem', position: 2, name: '제품', item: 'https://example.com/products' },
    ],
  }

  return (
    <section>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(productJsonLd).replace(/</g, '\\u003c'),
        }}
      />
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(breadcrumbJsonLd).replace(/</g, '\\u003c'),
        }}
      />
      {/* 페이지 콘텐츠 */}
    </section>
  )
}
```

---

## 검증 도구

구조화된 데이터가 올바르게 구현되었는지 검증하세요.

### Google Rich Results Test

[Rich Results Test](https://search.google.com/test/rich-results)에서 URL이나 코드를 입력하여 테스트할 수 있습니다.

**확인 사항:**
- 구문 오류 없음
- 필수 속성 포함
- 리치 결과 자격 여부

### Schema Markup Validator

[Schema Markup Validator](https://validator.schema.org/)는 Schema.org 표준 준수 여부를 검증합니다.

### 로컬 테스트

개발 서버에서 페이지 소스를 확인합니다:

1. 브라우저에서 페이지 열기
2. 마우스 우클릭 → "페이지 소스 보기"
3. `application/ld+json` 스크립트 태그 확인
4. JSON이 올바르게 형식화되었는지 확인

---

## 일반적인 스키마 타입

### 웹사이트

```tsx
const websiteJsonLd = {
  '@context': 'https://schema.org',
  '@type': 'WebSite',
  name: '사이트 이름',
  url: 'https://example.com',
  potentialAction: {
    '@type': 'SearchAction',
    target: 'https://example.com/search?q={search_term_string}',
    'query-input': 'required name=search_term_string',
  },
}
```

### 로컬 비즈니스

```tsx
const localBusinessJsonLd = {
  '@context': 'https://schema.org',
  '@type': 'LocalBusiness',
  name: '가게 이름',
  image: 'https://example.com/store.jpg',
  '@id': 'https://example.com',
  url: 'https://example.com',
  telephone: '+82-2-1234-5678',
  address: {
    '@type': 'PostalAddress',
    streetAddress: '강남대로 123',
    addressLocality: '서울',
    postalCode: '06000',
    addressCountry: 'KR',
  },
  geo: {
    '@type': 'GeoCoordinates',
    latitude: 37.5665,
    longitude: 126.978,
  },
  openingHoursSpecification: [
    {
      '@type': 'OpeningHoursSpecification',
      dayOfWeek: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'],
      opens: '09:00',
      closes: '18:00',
    },
  ],
}
```

### 레시피

```tsx
const recipeJsonLd = {
  '@context': 'https://schema.org',
  '@type': 'Recipe',
  name: '김치찌개',
  image: 'https://example.com/kimchi-stew.jpg',
  author: {
    '@type': 'Person',
    name: '요리사 이름',
  },
  datePublished: '2024-01-15',
  description: '맛있는 김치찌개 레시피입니다.',
  prepTime: 'PT15M',
  cookTime: 'PT30M',
  totalTime: 'PT45M',
  recipeYield: '4인분',
  recipeIngredient: [
    '김치 300g',
    '돼지고기 200g',
    '두부 1모',
    '대파 1대',
  ],
  recipeInstructions: [
    {
      '@type': 'HowToStep',
      text: '냄비에 기름을 두르고 돼지고기를 볶습니다.',
    },
    {
      '@type': 'HowToStep',
      text: '김치를 넣고 함께 볶습니다.',
    },
  ],
  nutrition: {
    '@type': 'NutritionInformation',
    calories: '350 calories',
  },
}
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **XSS 방지를 위한 살균 처리**
   ```tsx
   JSON.stringify(jsonLd).replace(/</g, '\\u003c')
   ```

2. **필수 속성 포함**
   - 각 스키마 타입의 필수 속성을 확인하세요

3. **검증 도구로 테스트**
   - 배포 전 Rich Results Test로 확인

4. **동적 데이터 사용**
   - 하드코딩 대신 실제 데이터를 사용

5. **TypeScript 타입 사용**
   - `schema-dts` 패키지로 타입 안전성 확보

### ❌ 피해야 할 것

1. **살균 없이 사용자 입력 포함**
   ```tsx
   // ❌ 위험
   const jsonLd = {
     name: userInput, // XSS 취약점
   }
   ```

2. **불필요한 마크업 추가**
   - 페이지 콘텐츠와 관련 없는 스키마 추가 금지

3. **잘못된 데이터 제공**
   - 실제 콘텐츠와 일치하지 않는 정보는 SEO에 불이익

4. **숨겨진 콘텐츠에 대한 마크업**
   - 사용자에게 보이지 않는 콘텐츠의 마크업은 권장하지 않음

---

## 다음 단계

- [Metadata](../api-reference/functions/generateMetadata.md) - 메타데이터 생성
- [Sitemap](../api-reference/file-conventions/metadata/sitemap.md) - 사이트맵 생성
- [robots.txt](../api-reference/file-conventions/metadata/robots.md) - 로봇 설정

---

**문서 버전:** Next.js 16.1.1
**최종 업데이트:** 2026-01-15

**참고 자료:**
- [Schema.org](https://schema.org/)
- [Google Structured Data](https://developers.google.com/search/docs/advanced/structured-data)
- [schema-dts](https://www.npmjs.com/package/schema-dts)
