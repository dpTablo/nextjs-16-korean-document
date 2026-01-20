# use cache: remote

`'use cache: remote'` 지시어는 캐시된 출력을 메모리 대신 **원격 캐시**에 저장하도록 선언적으로 지정합니다. 이는 특정 작업에 대해 더 지속적인 캐싱을 제공하지만 인프라 비용과 캐시 조회 시 네트워크 지연이 발생합니다.

---

## 기본 설정

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

---

## 언제 사용해야 하나요?

### 원격 캐싱을 피해야 할 때

| 상황 | 이유 |
|------|------|
| 데이터 계층에 이미 서버 측 캐시가 있음 | 중복 캐싱 |
| 작업이 이미 빠름 (< 50ms) | 오버헤드가 이점보다 큼 |
| 캐시 키가 요청마다 대부분 고유함 | 캐시 적중률 저하 |
| 데이터가 자주 변경됨 (초~분 단위) | 캐시 효과 미미 |

### 원격 캐싱이 유용한 경우

| 상황 | 이점 |
|------|------|
| **요청 시점에 지연된 콘텐츠** | `cookies()`, `headers()`, `searchParams` 접근 시 |
| **속도 제한 API** | 업스트림 서비스의 속도 제한이나 요청 할당량 보호 |
| **느린 백엔드 보호** | 높은 트래픽에서 데이터베이스/API 병목 방지 |
| **비용이 큰 작업** | 반복 실행 시 비용이 많이 드는 쿼리/계산 |
| **불안정한 서비스** | 외부 서비스의 간헐적 장애 대응 |

---

## 캐싱 지시어 비교

| 기능 | `use cache` | `'use cache: remote'` | `'use cache: private'` |
|------|-------------|----------------------|------------------------|
| 서버 측 캐싱 | 메모리 또는 캐시 핸들러 | 원격 캐시 핸들러 | 없음 |
| 캐시 범위 | 모든 사용자 공유 | 모든 사용자 공유 | 클라이언트별 |
| 쿠키/헤더 직접 접근 | ❌ 불가 | ❌ 불가 | ✅ 가능 |
| 서버 캐시 활용률 | 낮음 (정적 셸 외) | 높음 (인스턴스 간 공유) | N/A |
| 추가 비용 | 없음 | 있음 | 없음 |
| 지연 시간 | 없음 | 캐시 핸들러 조회 | 없음 |

---

## 기본 사용법

```tsx
import { cacheLife, cacheTag } from 'next/cache'

async function getGlobalData() {
  'use cache: remote'
  cacheTag('global-data')
  cacheLife({ expire: 3600 })

  const response = await fetch('https://api.example.com/data')
  return response.json()
}
```

---

## 캐시 키 최적화

각 고유한 값은 별도의 캐시 항목을 생성하므로, 캐시 활용률을 높이려면 고유 값이 적은 차원으로 캐시해야 합니다.

### 카테고리별 상품 캐싱

```tsx
// app/products/[category]/page.tsx
async function ProductList({
  params,
  searchParams,
}: {
  params: Promise<{ category: string }>
  searchParams: Promise<{ minPrice?: string }>
}) {
  const { category } = await params
  const { minPrice } = await searchParams

  // 카테고리만 캐시 (가격 필터는 메모리에서 필터링)
  const products = await getProductsByCategory(category)
  const filtered = minPrice
    ? products.filter((p) => p.price >= parseFloat(minPrice))
    : products

  return (
    <div>
      {filtered.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}

async function getProductsByCategory(category: string) {
  'use cache: remote'
  cacheTag(`category-${category}`)
  cacheLife({ expire: 300 })

  return db.products.findByCategory(category)
}
```

### 언어별 콘텐츠 캐싱

```tsx
// lib/cms.ts
import { cookies } from 'next/headers'
import { cacheLife } from 'next/cache'

async function WelcomeMessage() {
  const language = (await cookies()).get('language')?.value || 'en'
  const content = await getCMSContent(language)
  return <div>{content.welcomeMessage}</div>
}

async function getCMSContent(language: string) {
  'use cache: remote'
  cacheLife({ expire: 3600 })

  // 사용자별(수천)이 아닌 언어별(~50)로 캐시 항목 생성
  return cms.getHomeContent(language)
}
```

---

## 중첩 규칙

| 중첩 조합 | 허용 |
|----------|------|
| 원격 캐시 내부에 원격 캐시 | ✅ 가능 |
| 일반 캐시(`use cache`) 내부에 원격 캐시 | ✅ 가능 |
| 프라이빗 캐시 내부에 원격 캐시 | ❌ 불가 |
| 원격 캐시 내부에 프라이빗 캐시 | ❌ 불가 |

---

## 실제 예제

### 사용자 선호도 기반 가격 캐싱

```tsx
// app/product/[id]/page.tsx
import { Suspense } from 'react'
import { cookies } from 'next/headers'
import { cacheLife, cacheTag } from 'next/cache'

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params

  return (
    <div>
      <ProductDetails id={id} />
      <Suspense fallback={<div>가격 로딩 중...</div>}>
        <ProductPrice productId={id} />
      </Suspense>
    </div>
  )
}

async function ProductPrice({ productId }: { productId: string }) {
  const currency = (await cookies()).get('currency')?.value ?? 'USD'
  const price = await getProductPrice(productId, currency)
  return <div>가격: {price} {currency}</div>
}

async function getProductPrice(productId: string, currency: string) {
  'use cache: remote'
  cacheTag(`product-price-${productId}`)
  cacheLife({ expire: 3600 })

  return db.products.getPrice(productId, currency)
}
```

### 데이터베이스 부하 감소

```tsx
// lib/analytics.ts
import { connection } from 'next/server'
import { cacheLife, cacheTag } from 'next/cache'

async function DashboardStats() {
  await connection()
  const stats = await getGlobalStats()
  return <StatsDisplay stats={stats} />
}

async function getGlobalStats() {
  'use cache: remote'
  cacheTag('global-stats')
  cacheLife({ expire: 60 })

  const stats = await db.analytics.aggregate({
    total_users: 'count',
    active_sessions: 'count',
    revenue: 'sum',
  })

  return stats
}
```

### 외부 API 응답 캐싱

```tsx
// lib/feed.ts
import { cacheLife, cacheTag } from 'next/cache'

async function getFeedItems() {
  'use cache: remote'
  cacheTag('feed-items')
  cacheLife({ expire: 120 })

  const response = await fetch('https://api.example.com/feed')
  return response.json()
}
```

### 복잡한 계산 캐싱

```tsx
// lib/reports.ts
import { cacheLife } from 'next/cache'

async function generateReport() {
  'use cache: remote'
  cacheLife({ expire: 3600 })

  const data = await db.transactions.findMany()

  return {
    totalRevenue: calculateRevenue(data),
    topProducts: analyzeProducts(data),
    trends: calculateTrends(data),
  }
}
```

### 혼합 캐싱 전략

```tsx
// lib/products.ts
import { cookies } from 'next/headers'
import { cacheLife, cacheTag } from 'next/cache'

// 정적 캐시 (빌드 시)
async function getProduct(id: string) {
  'use cache'
  cacheTag(`product-${id}`)
  return db.products.find({ where: { id } })
}

// 원격 캐시 (런타임, 공유)
async function getProductPrice(id: string) {
  'use cache: remote'
  cacheTag(`product-price-${id}`)
  cacheLife({ expire: 300 })
  return db.products.getPrice({ where: { id } })
}

// 프라이빗 캐시 (사용자별)
async function getRecommendations(productId: string) {
  'use cache: private'
  cacheLife({ expire: 60 })

  const sessionId = (await cookies()).get('session-id')?.value
  return db.recommendations.findMany({
    where: { productId, sessionId },
  })
}
```

---

## 플랫폼 지원

| 배포 옵션 | 지원 |
|----------|------|
| Node.js 서버 | ✅ |
| Docker 컨테이너 | ✅ |
| 정적 내보내기 | ❌ |
| 어댑터 | ✅ |

---

## 중요한 주의사항

> **Good to know**:
> - 원격 캐시는 인프라 비용과 네트워크 지연이 발생합니다
> - 캐시 키가 자주 변경되는 경우 효과가 떨어집니다
> - 프라이빗 캐시와 함께 중첩 사용할 수 없습니다

> **성능 팁**:
> - 고유 값이 적은 차원으로 캐시하세요 (사용자별이 아닌 카테고리별, 언어별 등)
> - 빠른 작업(< 50ms)에는 원격 캐시를 사용하지 마세요
> - 캐시 적중률을 모니터링하세요

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v16.0.0 | Cache Components 기능으로 `'use cache: remote'` 활성화 |

---

## 관련 문서

- [use cache](./use-cache.md)
- [use cache: private](./use-cache-private.md)
- [cacheLife](./functions/cacheLife.md)
- [cacheTag](./functions/cacheTag.md)
