# use cache: private

`'use cache: private'` 지시어는 캐시된 범위 내에서 `cookies()`, `headers()`, `searchParams`와 같은 런타임 요청 API에 접근할 수 있게 합니다. 그러나 결과는 **서버에 저장되지 않으며**, 브라우저 메모리에만 캐시되고 페이지 새로고침 시 유지되지 않습니다.

> **주의**: 현재 실험적 기능이며 프로덕션 환경에는 권장되지 않습니다.

---

## 사용 시점

`'use cache: private'`는 다음 상황에서 유용합니다:

- 런타임 데이터에 접근하는 함수를 캐시하고 싶지만, 값을 인자로 전달하도록 리팩토링하기 실용적이지 않을 때
- 컴플라이언스 요구사항이 특정 데이터의 서버 저장을 금지할 때

---

## 주요 특징

| 특징 | 설명 |
|------|------|
| **서버 저장 없음** | 결과가 서버에 저장되지 않음 |
| **브라우저 메모리 캐시** | 브라우저 메모리에만 캐시됨 |
| **페이지 새로고침 시 소멸** | 페이지 새로고침 시 캐시가 유지되지 않음 |
| **런타임 실행** | 런타임 데이터 접근으로 인해 모든 서버 렌더링에서 실행됨 |
| **정적 셸 제외** | 정적 셸(static shell) 생성 중에는 실행되지 않음 |

---

## 설정 방법

### 1단계: cacheComponents 플래그 활성화

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

### 2단계: 함수에 지시어 추가

함수에 `'use cache: private'`와 `cacheLife` 구성을 추가합니다.

```tsx
async function getPersonalizedData() {
  'use cache: private'
  cacheLife({ stale: 60 })

  // 런타임 API 접근 가능
  const sessionId = (await cookies()).get('session-id')?.value
  // ...
}
```

---

## 기본 사용법

```tsx
// app/product/[id]/page.tsx
import { Suspense } from 'react'
import { cookies } from 'next/headers'
import { cacheLife, cacheTag } from 'next/cache'

export async function generateStaticParams() {
  return [{ id: '1' }]
}

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params

  return (
    <div>
      <ProductDetails id={id} />
      <Suspense fallback={<div>추천 상품 로딩 중...</div>}>
        <Recommendations productId={id} />
      </Suspense>
    </div>
  )
}

async function Recommendations({ productId }: { productId: string }) {
  const recommendations = await getRecommendations(productId)

  return (
    <div>
      {recommendations.map((rec) => (
        <ProductCard key={rec.id} product={rec} />
      ))}
    </div>
  )
}

async function getRecommendations(productId: string) {
  'use cache: private'
  cacheTag(`recommendations-${productId}`)
  cacheLife({ stale: 60 })

  // 프라이빗 캐시 함수 내에서 cookies 접근
  const sessionId = (await cookies()).get('session-id')?.value || 'guest'

  return getPersonalizedRecommendations(productId, sessionId)
}
```

> **중요**: `stale` 시간은 런타임 프리페칭이 작동하려면 최소 30초 이상이어야 합니다.

---

## 프라이빗 캐시에서 허용되는 요청 API

| API | `use cache`에서 허용 | `'use cache: private'`에서 허용 |
|-----|---------------------|-------------------------------|
| `cookies()` | ❌ | ✅ |
| `headers()` | ❌ | ✅ |
| `searchParams` | ❌ | ✅ |
| `connection()` | ❌ | ❌ |

> **참고**: `connection()` API는 연결 특정 정보를 제공하므로 두 지시어 모두에서 금지됩니다.

---

## 실제 예제

### 사용자 맞춤 콘텐츠 캐싱

```tsx
// lib/personalized.ts
import { cookies } from 'next/headers'
import { cacheLife, cacheTag } from 'next/cache'

export async function getPersonalizedFeed() {
  'use cache: private'
  cacheTag('personalized-feed')
  cacheLife({ stale: 60 })

  const userId = (await cookies()).get('user-id')?.value

  if (!userId) {
    return getDefaultFeed()
  }

  return getPersonalizedContent(userId)
}
```

### 헤더 기반 지역화 콘텐츠

```tsx
// lib/localized.ts
import { headers } from 'next/headers'
import { cacheLife, cacheTag } from 'next/cache'

export async function getLocalizedContent(pageId: string) {
  'use cache: private'
  cacheTag(`page-${pageId}`)
  cacheLife({ stale: 120 })

  const headersList = await headers()
  const locale = headersList.get('accept-language')?.split(',')[0] || 'en'

  return fetchLocalizedPage(pageId, locale)
}
```

### 검색 파라미터 기반 필터링

```tsx
// app/products/page.tsx
import { cacheLife, cacheTag } from 'next/cache'

async function getFilteredProducts(searchParams: { category?: string; sort?: string }) {
  'use cache: private'
  cacheTag('products')
  cacheLife({ stale: 60 })

  const category = searchParams.category || 'all'
  const sort = searchParams.sort || 'newest'

  return fetchProducts({ category, sort })
}

export default async function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ category?: string; sort?: string }>
}) {
  const params = await searchParams
  const products = await getFilteredProducts(params)

  return (
    <div>
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

---

## use cache vs use cache: private 비교

| 특성 | `use cache` | `use cache: private` |
|------|------------|---------------------|
| **저장 위치** | 서버 캐시 | 브라우저 메모리 |
| **지속성** | 재시작까지 유지 | 페이지 새로고침 시 소멸 |
| **런타임 API** | ❌ 접근 불가 | ✅ 접근 가능 |
| **공유 가능** | ✅ 모든 사용자 | ❌ 개별 사용자 |
| **사용 사례** | 공개 데이터 | 개인화된 데이터 |

---

## 주의사항

- **Route Handlers에서 사용 불가**
- 런타임 프리페칭이 안정화되지 않았으므로 실험적 기능입니다
- 커스텀 캐시 핸들러 구성이 불가능합니다

---

## 중요한 주의사항

> **Good to know**:
> - `'use cache: private'`는 현재 실험적 기능입니다
> - 프로덕션 환경에서는 신중하게 사용하세요
> - `stale` 시간은 최소 30초 이상으로 설정해야 런타임 프리페칭이 작동합니다

> **언제 사용해야 하나요?**:
> - 사용자별 맞춤 콘텐츠를 캐시할 때
> - 쿠키나 헤더 기반의 개인화된 데이터가 필요할 때
> - 서버에 민감한 데이터를 저장하면 안 될 때

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v16.0.0 | Cache Components 기능과 함께 `"use cache: private"` 활성화 |

---

## 관련 문서

- [use cache](./use-cache.md)
- [use cache: remote](./use-cache-remote.md)
- [cacheLife](./functions/cacheLife.md)
- [cacheTag](./functions/cacheTag.md)
