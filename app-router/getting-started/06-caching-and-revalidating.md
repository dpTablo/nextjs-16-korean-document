# 캐싱 및 재검증

**버전:** 16.1.1
**최종 업데이트:** 2025-11-05

## 개요
캐싱은 데이터 페칭 및 계산 결과를 저장하여 향후 요청을 더 빠르게 처리합니다. 재검증을 통해 전체 애플리케이션을 재빌드하지 않고 캐시 항목을 업데이트할 수 있습니다.

**사용 가능한 API:**
- `fetch`
- `cacheTag`
- `revalidateTag`
- `updateTag`
- `revalidatePath`
- `unstable_cache` (레거시)

---

## `fetch`

기본적으로 fetch 요청은 **캐시되지 않습니다**. `cache` 옵션으로 캐싱을 활성화하세요:

```tsx
export default async function Page() {
  const data = await fetch('https://...', { cache: 'force-cache' })
}
```

**시간 기반 전략으로 재검증:**

```tsx
export default async function Page() {
  const data = await fetch('https://...', { next: { revalidate: 3600 } })
}
```

**온디맨드 무효화를 위한 태그 지정:**

```tsx
export async function getUserById(id: string) {
  const data = await fetch(`https://...`, {
    next: {
      tags: ['user'],
    },
  })
}
```

> **참고:** Next.js는 fetch 요청이 있는 라우트를 사전 렌더링하고 HTML을 캐시합니다. 동적 렌더링을 보장하려면 `connection` API를 사용하세요.

---

## `cacheTag`

온디맨드 재검증을 위해 캐시 컴포넌트의 캐시된 데이터에 태그를 지정합니다. `use cache` 지시어와 함께 작동하여 fetch 요청 이상의 모든 계산을 캐시합니다:

```tsx
import { cacheTag } from 'next/cache'

export async function getProducts() {
  'use cache'
  cacheTag('products')

  const products = await db.query('SELECT * FROM products')
  return products
}
```

캐시를 무효화하려면 `revalidateTag` 또는 `updateTag`와 함께 사용하세요.

---

## `revalidateTag`

태그를 기반으로 캐시 항목을 재검증합니다. 두 가지 동작을 지원합니다:

- **`profile="max"`와 함께**: Stale-while-revalidate 시맨틱 (백그라운드에서 새 데이터를 페칭하는 동안 오래된 콘텐츠 제공) - **권장**
- **두 번째 인수 없음**: 레거시 동작 (즉시 캐시 만료 - 더 이상 사용되지 않음)

```tsx
import { revalidateTag } from 'next/cache'

export async function updateUser(id: string) {
  // 데이터 변경
  revalidateTag('user', 'max')
}
```

**사용:** Route Handlers 또는 Server Actions

---

## `updateTag`

Server Actions에서 read-your-own-writes 시나리오를 위해 캐시된 데이터를 즉시 만료시키도록 설계됨:

```tsx
import { updateTag } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const post = await db.post.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
    },
  })

  updateTag('posts')
  updateTag(`post-${post.id}`)

  redirect(`/posts/${post.id}`)
}
```

**`revalidateTag`와의 주요 차이점:**
- Server Actions에서만 사용
- 즉시 캐시 만료
- read-your-own-writes 시나리오를 위함

---

## `revalidatePath`

이벤트 후 라우트를 재검증합니다:

```tsx
import { revalidatePath } from 'next/cache'

export async function updateUser(id: string) {
  // 데이터 변경
  revalidatePath('/profile')
}
```

**사용:** Route Handlers 또는 Server Actions

---

## `unstable_cache` (레거시)

> ⚠️ **더 이상 사용되지 않음:** 대신 `use cache` 지시어와 함께 캐시 컴포넌트를 사용하세요.

데이터베이스 쿼리 및 비동기 함수의 결과를 캐시합니다:

```tsx
import { unstable_cache } from 'next/cache'
import { getUserById } from '@/app/lib/data'

export default async function Page({
  params,
}: {
  params: Promise<{ userId: string }>
}) {
  const { userId } = await params

  const getCachedUser = unstable_cache(
    async () => {
      return getUserById(userId)
    },
    [userId]
  )
}
```

**재검증 옵션과 함께:**

```tsx
const getCachedUser = unstable_cache(
  async () => {
    return getUserById(userId)
  },
  [userId],
  {
    tags: ['user'],
    revalidate: 3600,
  }
)
```

**옵션:**
- `tags`: 캐시 재검증을 위한 태그 배열
- `revalidate`: 캐시 재검증 전 초 단위 시간
