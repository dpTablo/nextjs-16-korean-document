# cacheLife

`cacheLife` 함수는 함수나 컴포넌트의 캐시 수명을 설정하는 데 사용됩니다. `use cache` 지시어와 함께 사용해야 합니다.

---

## 기본 설정

### 1단계: cacheComponents 플래그 활성화

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

### 주의사항

- `cacheLife`는 `use cache` 지시어가 필요합니다
- `cacheLife`는 캐시되는 함수 내부에 배치해야 합니다
- 함수 호출당 하나의 `cacheLife`만 실행되어야 합니다

---

## 캐시 프로필 프로퍼티

| 프로퍼티 | 설명 | 기본값 |
|----------|------|--------|
| `stale` | 클라이언트가 서버 확인 없이 캐시된 데이터를 사용할 수 있는 시간 (초) | 300 (5분) |
| `revalidate` | 서버가 백그라운드에서 캐시된 콘텐츠를 다시 생성하는 빈도 (초) | 900 (15분) |
| `expire` | 트래픽이 없을 때 캐시가 만료되는 최대 시간 (초) | 31536000 (1년) |

### stale

클라이언트 측 캐시 시간입니다. 클라이언트가 서버 확인 없이 캐시된 데이터를 사용할 수 있는 시간을 설정합니다.

```tsx
cacheLife({ stale: 300 }) // 5분
```

### revalidate

서버 재생성 빈도입니다. 서버가 백그라운드에서 캐시된 콘텐츠를 다시 생성하는 빈도를 설정합니다.

```tsx
cacheLife({ revalidate: 900 }) // 15분
```

### expire

최대 캐시 시간입니다. 트래픽이 없을 때 서버가 콘텐츠를 다시 생성해야 하는 최대 시간을 설정합니다.

```tsx
cacheLife({ expire: 3600 }) // 1시간
```

---

## 기본 제공 캐시 프로필

Next.js는 다양한 사용 사례에 맞는 사전 정의된 캐시 프로필을 제공합니다.

| 프로필 | 사용 사례 | `stale` | `revalidate` | `expire` |
|--------|----------|---------|--------------|----------|
| `default` | 표준 콘텐츠 | 5분 | 15분 | 1년 |
| `seconds` | 실시간 데이터 | 30초 | 1초 | 1분 |
| `minutes` | 자주 업데이트되는 콘텐츠 | 5분 | 1분 | 1시간 |
| `hours` | 하루 여러 번 업데이트 | 5분 | 1시간 | 1일 |
| `days` | 매일 업데이트 | 5분 | 1일 | 1주 |
| `weeks` | 주 1회 업데이트 | 5분 | 1주 | 30일 |
| `max` | 거의 변경되지 않음 | 5분 | 30일 | 1년 |

### 기본 제공 프로필 사용

```tsx
// app/blog/page.tsx
'use cache'
import { cacheLife } from 'next/cache'

export default async function BlogPage() {
  cacheLife('days') // 블로그 콘텐츠 매일 업데이트

  const posts = await getBlogPosts()
  return (
    <div>
      {posts.map((post) => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  )
}
```

---

## 사용자 정의 캐시 프로필

### next.config.ts에서 정의

프로젝트에 맞는 사용자 정의 캐시 프로필을 생성할 수 있습니다.

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
  cacheLife: {
    // 2주 단위 캐시 프로필
    biweekly: {
      stale: 60 * 60 * 24 * 14,   // 14일
      revalidate: 60 * 60 * 24,   // 1일
      expire: 60 * 60 * 24 * 14,  // 14일
    },
  },
}

export default nextConfig
```

### 사용자 정의 프로필 사용

```tsx
// app/page.tsx
'use cache'
import { cacheLife } from 'next/cache'

export default async function Page() {
  cacheLife('biweekly')
  return <div>2주 단위로 캐시되는 페이지</div>
}
```

### 기본 프로필 재정의

기본 제공 프로필을 재정의할 수도 있습니다.

```ts
// next.config.ts
const nextConfig = {
  cacheComponents: true,
  cacheLife: {
    days: {
      stale: 3600,      // 1시간
      revalidate: 900,  // 15분
      expire: 86400,    // 1일
    },
  },
}

export default nextConfig
```

---

## 인라인 캐시 프로필

일회성 사용의 경우 객체를 직접 전달할 수 있습니다.

```tsx
// app/page.tsx
'use cache'
import { cacheLife } from 'next/cache'

export default async function Page() {
  cacheLife({
    stale: 3600,      // 1시간
    revalidate: 900,  // 15분
    expire: 86400,    // 1일
  })

  return <div>인라인 캐시 설정이 적용된 페이지</div>
}
```

---

## 실제 예제

### 기본 제공 프로필 사용

**블로그 게시글 (매일 업데이트):**

```tsx
// app/blog/[slug]/page.tsx
import { cacheLife } from 'next/cache'

export default async function BlogPost({ params }: { params: Promise<{ slug: string }> }) {
  'use cache'
  cacheLife('days')

  const { slug } = await params
  const post = await fetchBlogPost(slug)

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  )
}
```

**제품 페이지 (매시간 업데이트):**

```tsx
// app/products/[id]/page.tsx
import { cacheLife } from 'next/cache'

export default async function ProductPage({ params }: { params: Promise<{ id: string }> }) {
  'use cache'
  cacheLife('hours')

  const { id } = await params
  const product = await fetchProduct(id)

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}원</p>
    </div>
  )
}
```

### 다양한 캐시 프로필 정의

```ts
// next.config.ts
const nextConfig = {
  cacheComponents: true,
  cacheLife: {
    // 편집 콘텐츠용
    editorial: {
      stale: 600,       // 10분
      revalidate: 3600, // 1시간
      expire: 86400,    // 1일
    },
    // 마케팅 콘텐츠용
    marketing: {
      stale: 300,       // 5분
      revalidate: 1800, // 30분
      expire: 43200,    // 12시간
    },
  },
}

export default nextConfig
```

### API 라우트에서 인라인 캐시 사용

```tsx
// app/api/limited-offer/route.ts
import { cacheLife } from 'next/cache'

async function getLimitedOffer() {
  'use cache'

  cacheLife({
    stale: 60,       // 1분
    revalidate: 300, // 5분
    expire: 3600,    // 1시간
  })

  const offer = await db.offer.findFirst({
    where: { type: 'limited' },
    orderBy: { created_at: 'desc' },
  })

  return offer
}

export async function GET() {
  const offer = await getLimitedOffer()
  return Response.json(offer)
}
```

### 유틸리티 함수 캐싱

**거의 변경되지 않는 설정 데이터:**

```tsx
// lib/api.ts
import { cacheLife } from 'next/cache'

export async function getSettings() {
  'use cache'
  cacheLife('max')
  return await fetchSettings()
}
```

**실시간 통계 데이터:**

```tsx
// lib/stats.ts
import { cacheLife } from 'next/cache'

export async function getRealtimeStats() {
  'use cache'
  cacheLife('seconds')
  return await fetchStats()
}
```

### 조건부 캐시 수명

데이터 상태에 따라 다른 캐시 수명을 적용할 수 있습니다.

```tsx
// lib/posts.ts
import { cacheLife, cacheTag } from 'next/cache'

async function getPostContent(slug: string) {
  'use cache'

  const post = await fetchPost(slug)
  cacheTag(`post-${slug}`)

  if (!post) {
    // 발행되지 않은 콘텐츠는 짧게 캐시
    cacheLife('minutes')
    return null
  }

  // 발행된 콘텐츠는 길게 캐시
  cacheLife('days')
  return post.data
}
```

### 동적 캐시 수명

데이터에서 직접 캐시 타이밍을 읽어올 수 있습니다.

```tsx
// lib/posts.ts
import { cacheLife, cacheTag } from 'next/cache'

async function getPostContent(slug: string) {
  'use cache'

  const post = await fetchPost(slug)
  cacheTag(`post-${slug}`)

  if (!post) {
    cacheLife('minutes')
    return null
  }

  // 데이터에서 직접 캐시 타이밍 읽기
  cacheLife({
    revalidate: post.revalidateSeconds ?? 3600,
  })

  return post.data
}
```

---

## 중첩 캐싱 동작

### 명시적 외부 cacheLife가 있는 경우

외부 캐시가 자체 수명을 사용합니다.

```tsx
// app/dashboard/page.tsx
import { cacheLife } from 'next/cache'
import { Widget } from './widget'

export default async function Dashboard() {
  'use cache'
  cacheLife('hours') // 외부 스코프가 자체 수명 설정

  return (
    <div>
      <h1>대시보드</h1>
      <Widget /> {/* 내부 스코프는 'minutes' 수명 */}
    </div>
  )
}
```

### 명시적 외부 cacheLife가 없는 경우

내부 캐시가 외부에 영향을 미칠 수 있습니다.

```tsx
// app/dashboard/page.tsx
import { Widget } from './widget'

export default async function Dashboard() {
  'use cache'
  // cacheLife 호출 없음 - 기본값(15분) 사용
  // Widget이 5분이면 → Dashboard는 5분
  // Widget이 1시간이면 → Dashboard는 15분 유지

  return (
    <div>
      <h1>대시보드</h1>
      <Widget />
    </div>
  )
}
```

---

## 클라이언트 라우터 캐시 동작

- `stale` 프로퍼티는 `Cache-Control` 헤더가 아닌 클라이언트 측 라우터 캐시를 제어합니다
- 서버는 `x-nextjs-stale-time` 응답 헤더를 통해 stale 시간을 전송합니다
- **최소 30초가 적용됩니다** (프리페칭된 링크의 사용 가능성 보장)
- Server Action에서 재검증 함수 호출 시 클라이언트 캐시가 즉시 지워집니다

---

## 중요한 주의사항

> **Good to know**:
> - `cacheLife`는 `use cache` 지시어 없이 사용할 수 없습니다
> - 각 캐시된 함수에서 `cacheLife`는 한 번만 호출해야 합니다
> - 조건에 따라 다른 캐시 수명을 적용할 수 있습니다

> **성능 팁**:
> - 실시간 데이터에는 `seconds` 프로필 사용
> - 정적 콘텐츠에는 `max` 프로필 사용
> - CMS 콘텐츠에는 `days` 또는 `weeks` 프로필 사용

---

## 관련 문서

- [use cache](../use-cache.md)
- [cacheTag](./cacheTag.md)
- [revalidateTag](./revalidateTag.md)
- [Caching Guide](../../guides/caching.md)
