# use cache

## 개요

`'use cache'` 지시어는 라우트, React 컴포넌트, 또는 함수를 **캐시 가능**하게 표시합니다. 이를 통해 동적 데이터를 포함한 정적 셸 생성과 인메모리 LRU 스토리지를 통한 런타임 캐싱을 활성화합니다.

> **버전:** 16.1.2 | **최종 업데이트:** 2025-12-18

---

## 설정 요구사항

`next.config.ts`에서 캐싱을 활성화해야 합니다:

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

---

## 구현 레벨

### 파일 레벨 (File Level)

```tsx
'use cache'

export default async function Page() {
  // 모든 export는 async여야 함
}
```

### 컴포넌트 레벨 (Component Level)

```tsx
export async function MyComponent() {
  'use cache'
  return <></>
}
```

### 함수 레벨 (Function Level)

```tsx
export async function getData() {
  'use cache'
  const data = await fetch('/api/data')
  return data
}
```

---

## 캐시 키 (Cache Keys)

캐시 키는 다음 요소로부터 생성됩니다:

1. **Build ID** - 빌드마다 고유; 변경 시 모든 캐시 항목 무효화
2. **Function ID** - 함수 위치 및 시그니처의 보안 해시
3. **직렬화 가능한 인자** - Props/arguments
4. **HMR refresh hash** (개발 환경만) - 핫 모듈 교체 시 무효화

**외부 스코프 변수는 자동으로 캡처됩니다:**

```tsx
async function Component({ userId }: { userId: string }) {
  const getData = async (filter: string) => {
    'use cache'
    // 캐시 키에는 userId(클로저)와 filter(인자) 모두 포함됨
    return fetch(`/api/users/${userId}/data?filter=${filter}`)
  }
  return getData('active')
}
```

---

## 직렬화 규칙 (Serialization Rules)

### 지원되는 인자 타입

- **기본 타입:** `string`, `number`, `boolean`, `null`, `undefined`
- **순수 객체:** `{ key: value }`
- **배열:** `[1, 2, 3]`
- **Date, Map, Set, TypedArray, ArrayBuffer**
- **React elements** (pass-through만)

### 지원되는 반환 값

- 인자와 동일한 타입 + JSX 엘리먼트

### 지원되지 않는 타입

- 클래스 인스턴스
- 함수 (pass-through 제외)
- Symbol, WeakMap, WeakSet
- URL 인스턴스

**유효한 예제:**

```tsx
async function UserCard({
  id,
  config,
}: {
  id: string
  config: { theme: string }
}) {
  'use cache'
  return <div>{id}</div>
}
```

**잘못된 예제:**

```tsx
async function UserProfile({ user }: { user: UserClass }) {
  'use cache'
  // Error: 클래스 인스턴스는 직렬화할 수 없음
  return <div>{user.name}</div>
}
```

### Pass-Through 패턴 (비직렬화 가능 값)

직렬화할 수 없는 값을 검사 없이 통과시킵니다:

```tsx
async function CachedWrapper({ children }: { children: ReactNode }) {
  'use cache'
  // children을 읽거나 수정하지 말고 그냥 통과시킴
  return (
    <div className="wrapper">
      <header>캐시된 헤더</header>
      {children}
    </div>
  )
}

export default function Page() {
  return (
    <CachedWrapper>
      <DynamicComponent /> {/* 캐시되지 않음, 통과됨 */}
    </CachedWrapper>
  )
}
```

**Server Actions Pass-Through:**

```tsx
async function CachedForm({ action }: { action: () => Promise<void> }) {
  'use cache'
  // action을 호출하지 말고 그냥 통과시킴
  return <form action={action}>{/* ... */}</form>
}
```

---

## 제약 사항 (Constraints)

**런타임 API에 직접 접근할 수 없습니다:**

- `cookies()`
- `headers()`
- `searchParams`

**해결 방법:** 캐시된 범위 외부에서 읽고 인자로 전달합니다:

```tsx
// ❌ 잘못된 예
async function Page() {
  'use cache'
  const cookieStore = cookies() // Error
}

// ✅ 올바른 예
async function Page() {
  const cookieValue = (await cookies()).get('name')?.value
  return <Cached value={cookieValue} />
}
```

---

## 런타임 캐싱 동작 (Runtime Caching Behavior)

### 환경별 동작

| 환경 | 동작 |
|------|------|
| **Serverless** | 캐시 항목이 요청 간 유지되지 않음; 빌드 타임 캐싱은 정상 동작 |
| **Self-hosted** | 캐시 항목 유지; `cacheMaxMemorySize`로 크기 제어 |

### 캐시 대안

- **`'use cache: remote'`** - 플랫폼 제공 캐시 핸들러 (Redis/KV); 네트워크 왕복 필요; 플랫폼 요금 포함
- **`'use cache: private'`** - 규정 준수 요구사항이 있거나 런타임 데이터를 인자로 리팩토링할 수 없을 때

---

## 캐시 수명 및 재검증 (Cache Lifetime & Revalidation)

### 기본 프로필

```tsx
async function getData() {
  'use cache'
  // 기본값: stale: 5분, revalidate: 15분, 시간 제한 만료 없음
  return fetch('/api/data')
}
```

### cacheLife()로 커스텀 캐시 기간 설정

```tsx
import { cacheLife } from 'next/cache'

async function getData() {
  'use cache'
  cacheLife('hours') // 'hours' 프로필 사용
  return fetch('/api/data')
}
```

### 태그로 온디맨드 재검증

```tsx
// 캐시된 데이터에 태그 추가
import { cacheTag } from 'next/cache'

async function getProducts() {
  'use cache'
  cacheTag('products')
  return fetch('/api/products')
}

// Server Action에서 무효화
import { updateTag } from 'next/cache'

export async function updateProduct() {
  await db.products.update(...)
  updateTag('products') // 모든 'products' 캐시 무효화
}
```

### 클라이언트 vs 서버 캐싱

**서버:**
- `revalidate` 및 `expire` 시간을 존중하는 인메모리 스토리지
- `cacheHandlers`로 커스터마이징 가능

**클라이언트 (브라우저):**
- `stale` 시간 동안 브라우저 메모리에 저장
- 최소 30초 stale 시간 (라우터 강제)
- `x-nextjs-stale-time` 응답 헤더를 통해 수명 수신

---

## 사용 예제

### 전체 라우트 캐싱

```tsx
// app/layout.tsx
'use cache'

export default async function Layout({ children }: { children: ReactNode }) {
  return <div>{children}</div>
}
```

```tsx
// app/page.tsx
'use cache'

async function Users() {
  const users = await fetch('/api/users')
  // users 처리
}

export default async function Page() {
  return (
    <main>
      <Users />
    </main>
  )
}
```

> **참고:** `'use cache'`가 있는 세그먼트만 캐시됩니다. 전체 캐싱을 위해서는 layout과 page 모두에 추가하세요.

### 컴포넌트 출력 캐싱

```tsx
export async function Bookings({ type = 'haircut' }: BookingsProps) {
  'use cache'

  async function getBookingsData() {
    const data = await fetch(`/api/bookings?type=${encodeURIComponent(type)}`)
    return data
  }

  return // JSX
}
```

직렬화된 props가 일치하면 캐시 항목이 재사용됩니다.

### 함수 출력 캐싱

```tsx
// app/actions.ts
export async function getData() {
  'use cache'
  const data = await fetch('/api/data')
  return data
}
```

### 슬롯으로 구성 (Interleaving)

```tsx
export default async function Page() {
  const uncachedData = await getData()
  return (
    <CacheComponent header={<h1>홈</h1>}>
      <DynamicComponent data={uncachedData} />
    </CacheComponent>
  )
}

async function CacheComponent({
  header,
  children,
}: {
  header: ReactNode
  children: ReactNode
}) {
  'use cache'
  const cachedData = await fetch('/api/cached-data')
  return (
    <div>
      {header}
      <PrerenderedComponent data={cachedData} />
      {children}
    </div>
  )
}
```

props로 전달된 컴포지셔널 슬롯은 함수 본문에서 직접 참조되지 않으면 캐시 항목에 영향을 주지 않습니다.

---

## 문제 해결 (Troubleshooting)

### 상세 캐시 로깅

```bash
NEXT_PRIVATE_DEBUG_CACHE=1 npm run dev
# 또는 프로덕션용
NEXT_PRIVATE_DEBUG_CACHE=1 npm run start
```

개발 환경에서 캐시된 함수 로그는 `Cache` 접두사와 함께 표시됩니다.

### 빌드 중단 (50초 타임아웃)

**에러:** "Filling a cache during prerender timed out, likely because request-specific arguments such as params, searchParams, cookies() or dynamic data were used inside 'use cache'."

**원인:** `'use cache'` 경계 내에서 동적/런타임 데이터로 해결되는 Promise에 접근

**해결 - 런타임 데이터 Promise 전달:**

```tsx
// ❌ 문제
async function Cached({ promise }: { promise: Promise<unknown> }) {
  'use cache'
  const data = await promise // 빌드 중단
  return <p>..</p>
}

// ✅ 해결
async function Dynamic() {
  const cookieStore = await cookies()
  const value = cookieStore.get('name')?.value
  return <Cached value={value} /> // 해결된 값 전달
}
```

**해결 - 공유 스토리지:**

```tsx
// ✅ Next.js 내장 fetch() 중복 제거 사용
// 또는 캐시된/캐시되지 않은 컨텍스트용 별도 Map 사용
```

---

## 플랫폼 지원

| 배포 옵션 | 지원 |
|-----------|------|
| Node.js 서버 | ✅ 예 |
| Docker 컨테이너 | ✅ 예 |
| 정적 내보내기 | ❌ 아니오 |
| 어댑터 | 플랫폼별 |

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| `v16.0.0` | Cache Components 기능과 함께 `'use cache'` 활성화 |
| `v15.0.0` | `'use cache'` 실험적 기능으로 도입 |

---

## 핵심 요약

### ✅ 권장사항

- cookies/headers를 캐시된 범위 외부에서 읽고 인자로 전달
- 커스텀 기간을 위해 `cacheLife()` 사용
- 온디맨드 무효화를 위해 `cacheTag()`/`updateTag()` 사용
- 직렬화할 수 없는 값을 검사 없이 통과시킴
- `children` 및 Server Actions와 안전하게 구성

### ❌ 피해야 할 사항

- `'use cache'` 내부에서 `cookies()`, `headers()`, `searchParams` 직접 호출
- 클래스 인스턴스나 지원되지 않는 타입을 인자로 전달
- `children` 또는 Server Action props 검사/수정
- 캐시된 코드에서 접근하는 공유 스토리지에 동적 Promise 저장

---

## 사용 가능한 위치

| 위치 | 사용 가능 |
|------|----------|
| **Server Components** | ✅ 예 |
| **Server Actions** | ✅ 예 |
| **Route Handlers** | ✅ 예 |
| **Client Components** | ❌ 아니오 |

---

## 베스트 프랙티스

### 1. 동적 데이터는 인자로 전달

```tsx
// ✅ 올바른 방법
export default async function Page() {
  const session = await getSession()
  return <CachedComponent userId={session.userId} />
}

async function CachedComponent({ userId }: { userId: string }) {
  'use cache'
  const data = await fetchUserData(userId)
  return <div>{data}</div>
}
```

### 2. 적절한 캐시 프로필 사용

```tsx
import { cacheLife } from 'next/cache'

async function getStaticData() {
  'use cache'
  cacheLife('days') // 거의 변하지 않는 데이터
  return fetch('/api/static-content')
}

async function getFrequentData() {
  'use cache'
  cacheLife('minutes') // 자주 변하는 데이터
  return fetch('/api/frequent-updates')
}
```

### 3. 태그로 선택적 재검증

```tsx
import { cacheTag } from 'next/cache'

async function getProductList() {
  'use cache'
  cacheTag('products', 'catalog')
  return fetch('/api/products')
}

async function getProductDetails(id: string) {
  'use cache'
  cacheTag('products', `product-${id}`)
  return fetch(`/api/products/${id}`)
}

// 특정 제품만 재검증
import { updateTag } from 'next/cache'

export async function updateProductAction(id: string) {
  await updateProduct(id)
  updateTag(`product-${id}`) // 해당 제품만 무효화
}
```

### 4. 세분화된 캐싱

```tsx
// ❌ 모든 것을 함께 캐싱
async function Page() {
  'use cache'
  const user = await getUser()
  const posts = await getPosts()
  return <div>{/* ... */}</div>
}

// ✅ 개별적으로 캐싱
async function Page() {
  return (
    <div>
      <UserSection /> {/* 별도 캐시 */}
      <PostsSection /> {/* 별도 캐시 */}
    </div>
  )
}

async function UserSection() {
  'use cache'
  const user = await getUser()
  return <div>{user.name}</div>
}

async function PostsSection() {
  'use cache'
  const posts = await getPosts()
  return <div>{posts.map(...)}</div>
}
```

---

## 실제 사용 패턴

### 패턴 1: 사용자별 캐싱

```tsx
async function Dashboard({ userId }: { userId: string }) {
  'use cache'
  const stats = await getUserStats(userId)
  return (
    <div>
      <h1>대시보드</h1>
      <Stats data={stats} />
    </div>
  )
}

// 사용자별로 캐시됨
export default async function Page() {
  const session = await getSession()
  return <Dashboard userId={session.userId} />
}
```

### 패턴 2: 다국어 지원

```tsx
async function Content({ locale }: { locale: string }) {
  'use cache'
  const translations = await getTranslations(locale)
  return <div>{translations.welcome}</div>
}

export default async function Page({
  params,
}: {
  params: Promise<{ locale: string }>
}) {
  const { locale } = await params
  return <Content locale={locale} />
}
```

### 패턴 3: 필터링된 목록

```tsx
async function ProductList({
  category,
  sort,
}: {
  category: string
  sort: string
}) {
  'use cache'
  cacheTag('products', `category-${category}`)
  const products = await getProducts({ category, sort })
  return <div>{products.map(...)}</div>
}

export default async function Page({
  searchParams,
}: {
  searchParams: Promise<{ category?: string; sort?: string }>
}) {
  const params = await searchParams
  return (
    <ProductList
      category={params.category || 'all'}
      sort={params.sort || 'name'}
    />
  )
}
```

---

## 고급 사용법

### 조건부 캐싱

```tsx
async function ConditionalCache({ shouldCache }: { shouldCache: boolean }) {
  if (shouldCache) {
    'use cache'
  }

  const data = await fetchData()
  return <div>{data}</div>
}
```

### 중첩된 캐싱

```tsx
async function OuterComponent({ id }: { id: string }) {
  'use cache'
  const outerData = await getOuterData(id)

  return (
    <div>
      <InnerComponent data={outerData} />
    </div>
  )
}

async function InnerComponent({ data }: { data: any }) {
  'use cache'
  const processedData = await processData(data)
  return <div>{processedData}</div>
}
```

### 에러 처리와 함께

```tsx
async function SafeCachedComponent({ id }: { id: string }) {
  'use cache'

  try {
    const data = await fetchData(id)
    return <div>{data}</div>
  } catch (error) {
    console.error('캐시된 컴포넌트 에러:', error)
    return <div>데이터를 불러올 수 없습니다</div>
  }
}
```

---

## 성능 최적화 팁

### 1. 적절한 캐시 세분도 선택

- **너무 거친 캐싱:** 불필요한 데이터 재페칭
- **너무 세밀한 캐싱:** 캐시 오버헤드 증가

```tsx
// ✅ 적절한 균형
async function ProductPage({ id }: { id: string }) {
  return (
    <div>
      <ProductDetails id={id} /> {/* 자주 변경 - 별도 캐시 */}
      <ProductReviews id={id} /> {/* 덜 자주 변경 - 별도 캐시 */}
      <RelatedProducts id={id} /> {/* 거의 정적 - 별도 캐시 */}
    </div>
  )
}
```

### 2. 메모리 사용량 모니터링

```ts
// next.config.ts
const nextConfig: NextConfig = {
  cacheMaxMemorySize: 50 * 1024 * 1024, // 50MB
}
```

### 3. 캐시 워밍

```tsx
// 빌드 타임에 중요한 데이터 프리렌더링
async function CriticalData() {
  'use cache'
  cacheLife('days')
  const data = await fetchCriticalData()
  return <div>{data}</div>
}
```

---

## 디버깅

### 캐시 히트/미스 확인

```bash
NEXT_PRIVATE_DEBUG_CACHE=1 npm run dev
```

콘솔 출력:
```
Cache [hit] /app/page.tsx
Cache [miss] /app/products/[id]/page.tsx
Cache [skip] /app/api/route.ts
```

### 캐시 무효화 추적

```tsx
import { updateTag } from 'next/cache'

export async function revalidateProducts() {
  console.log('무효화: products 태그')
  updateTag('products')
}
```

---

## 마이그레이션 가이드

### Pages Router에서 App Router로

```tsx
// Pages Router (getStaticProps)
export async function getStaticProps() {
  const data = await fetchData()
  return {
    props: { data },
    revalidate: 60,
  }
}

// App Router (use cache)
async function Page() {
  'use cache'
  cacheLife('minutes')
  const data = await fetchData()
  return <div>{data}</div>
}
```

### fetch 캐싱에서 use cache로

```tsx
// 기존 (fetch 캐싱)
const data = await fetch('/api/data', {
  next: { revalidate: 60 },
})

// 새로운 방식 (use cache)
async function getData() {
  'use cache'
  cacheLife('minutes')
  const data = await fetch('/api/data')
  return data
}
```

---

## 관련 API

- [`cacheLife()`](../functions/cacheLife.md) - 캐시 수명 설정
- [`cacheTag()`](../functions/cacheTag.md) - 캐시 태그 추가
- [`updateTag()`](../functions/updateTag.md) - 태그로 캐시 무효화
- [`unstable_cache()`](../functions/unstable_cache.md) - 프로그래밍 방식 캐싱

---

## 요약

- **용도:** 컴포넌트, 함수, 라우트 출력 캐싱
- **레벨:** 파일, 컴포넌트, 함수
- **직렬화:** 기본 타입, 객체, 배열, Date, Map, Set 지원
- **제약:** `cookies()`, `headers()`, `searchParams` 직접 사용 불가
- **수명:** `cacheLife()`로 제어, `cacheTag()`로 무효화
- **환경:** Node.js 서버, Docker 지원; Serverless는 제한적
- **베스트 프랙티스:** 동적 데이터는 인자로 전달, 적절한 캐시 세분도, 태그로 선택적 재검증
