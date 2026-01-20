# cacheTag

`cacheTag` 함수는 캐시된 데이터에 태그를 붙여 온디맨드 무효화를 가능하게 합니다. 캐시 항목과 연결된 태그를 통해 다른 캐시 데이터에 영향을 주지 않고 특정 캐시 항목만 선택적으로 제거하거나 재검증할 수 있습니다.

---

## 설정 요구사항

`next.config.ts`에서 `cacheComponents` 플래그를 활성화해야 합니다.

```ts
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

---

## 기본 사용법

```tsx
import { cacheTag } from 'next/cache'

export async function getData() {
  'use cache'
  cacheTag('my-data')

  const data = await fetch('/api/data')
  return data.json()
}
```

---

## 캐시 무효화

`revalidateTag` API를 사용하여 태그된 캐시를 무효화합니다.

```tsx
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function submit() {
  await addPost()
  revalidateTag('my-data')
}
```

---

## 주요 특징

| 특징 | 설명 |
|------|------|
| **멱등성** | 동일한 태그를 여러 번 적용해도 추가 효과 없음 |
| **다중 태그** | 한 항목에 여러 태그 가능 |
| **태그 길이 제한** | 최대 256자 |
| **최대 태그 개수** | 캐시 항목당 최대 128개 |

---

## 시그니처

```tsx
cacheTag(...tags: string[]): void
```

### 매개변수

| 매개변수 | 타입 | 설명 |
|----------|------|------|
| `tags` | `string[]` | 캐시 항목에 적용할 태그 문자열 (가변 인자) |

---

## 실제 예제

### 컴포넌트에 태그 붙이기

```tsx
// app/components/bookings.tsx
import { cacheTag } from 'next/cache'

interface BookingsProps {
  type?: string
}

export async function Bookings({ type = 'haircut' }: BookingsProps) {
  'use cache'
  cacheTag('bookings-data')

  const response = await fetch(`/api/bookings?type=${encodeURIComponent(type)}`)
  const data = await response.json()

  return (
    <ul>
      {data.bookings.map((booking: any) => (
        <li key={booking.id}>{booking.name}</li>
      ))}
    </ul>
  )
}
```

### 여러 태그 적용

하나의 캐시 항목에 여러 태그를 적용할 수 있습니다.

```tsx
import { cacheTag } from 'next/cache'

export async function getProducts(category: string) {
  'use cache'
  cacheTag('products', `category-${category}`, 'catalog')

  const response = await fetch(`/api/products?category=${category}`)
  return response.json()
}
```

### 외부 데이터로 동적 태그 생성

API 응답 데이터를 기반으로 동적으로 태그를 생성할 수 있습니다.

```tsx
// lib/bookings.ts
import { cacheTag } from 'next/cache'

export async function getBookingsData(type: string) {
  'use cache'

  const response = await fetch(`/api/bookings?type=${encodeURIComponent(type)}`)
  const data = await response.json()

  // 데이터 ID를 태그로 사용
  cacheTag('bookings-data', `booking-${data.id}`)

  return data
}
```

### 태그된 캐시 무효화

Server Action에서 특정 태그의 캐시를 무효화합니다.

```tsx
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updateBookings() {
  // 데이터베이스 업데이트
  await updateBookingData()

  // 관련 캐시 무효화
  revalidateTag('bookings-data')
}

export async function updateCategory(category: string) {
  await updateCategoryData(category)

  // 특정 카테고리 캐시만 무효화
  revalidateTag(`category-${category}`)
}
```

### 블로그 포스트 캐싱

```tsx
// lib/posts.ts
import { cacheTag, cacheLife } from 'next/cache'

export async function getPost(slug: string) {
  'use cache'
  cacheTag('posts', `post-${slug}`)
  cacheLife('days')

  const response = await fetch(`/api/posts/${slug}`)
  return response.json()
}

export async function getAllPosts() {
  'use cache'
  cacheTag('posts', 'posts-list')
  cacheLife('hours')

  const response = await fetch('/api/posts')
  return response.json()
}
```

```tsx
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function publishPost(slug: string) {
  await db.post.update({
    where: { slug },
    data: { published: true },
  })

  // 특정 포스트 캐시 무효화
  revalidateTag(`post-${slug}`)

  // 포스트 목록 캐시도 무효화
  revalidateTag('posts-list')
}

export async function deleteAllPosts() {
  await db.post.deleteMany()

  // 모든 포스트 관련 캐시 무효화
  revalidateTag('posts')
}
```

### 사용자 데이터 캐싱

```tsx
// lib/user.ts
import { cacheTag } from 'next/cache'

export async function getUserProfile(userId: string) {
  'use cache'
  cacheTag('users', `user-${userId}`, `user-${userId}-profile`)

  const response = await fetch(`/api/users/${userId}`)
  return response.json()
}

export async function getUserPosts(userId: string) {
  'use cache'
  cacheTag(`user-${userId}`, `user-${userId}-posts`)

  const response = await fetch(`/api/users/${userId}/posts`)
  return response.json()
}
```

```tsx
// app/actions.ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updateUserProfile(userId: string) {
  await updateUser(userId)

  // 사용자 프로필 캐시만 무효화
  revalidateTag(`user-${userId}-profile`)
}

export async function deleteUser(userId: string) {
  await db.user.delete({ where: { id: userId } })

  // 해당 사용자의 모든 캐시 무효화
  revalidateTag(`user-${userId}`)
}
```

---

## 태그 명명 규칙

효과적인 캐시 관리를 위한 태그 명명 규칙 예시:

```tsx
// 일반적인 패턴
cacheTag('entity-type')           // 'posts', 'users', 'products'
cacheTag('entity-type-id')        // 'post-123', 'user-456'
cacheTag('entity-type-relation')  // 'user-123-posts', 'category-abc-products'

// 계층적 태그 예시
cacheTag(
  'products',                     // 모든 제품
  `category-${categoryId}`,       // 특정 카테고리의 제품
  `product-${productId}`          // 특정 제품
)
```

---

## 중요한 주의사항

> **Good to know**:
> - `cacheTag`는 `use cache` 지시어 없이 사용할 수 없습니다
> - 태그 이름은 최대 256자까지 가능합니다
> - 캐시 항목당 최대 128개의 태그를 적용할 수 있습니다
> - 동일한 태그를 여러 번 적용해도 추가 효과가 없습니다 (멱등성)

> **모범 사례**:
> - 일관된 태그 명명 규칙을 사용하세요
> - 계층적 태그 구조를 활용하여 세밀한 캐시 제어가 가능합니다
> - 동적 태그와 정적 태그를 함께 사용하세요

---

## 관련 문서

- [use cache](../use-cache.md)
- [cacheLife](./cacheLife.md)
- [revalidateTag](./revalidateTag.md)
- [Caching Guide](../../guides/caching.md)
