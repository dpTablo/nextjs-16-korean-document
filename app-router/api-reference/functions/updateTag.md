# updateTag

`updateTag` 함수는 Server Action 내에서 특정 캐시 태그의 데이터를 즉시 업데이트합니다. 사용자가 변경 후 즉시 UI에 반영되는 "read-your-own-writes" 시나리오를 위해 설계되었습니다.

---

## 사용 가능 범위

| 컨텍스트 | 사용 가능 |
|----------|----------|
| Server Actions | ✅ 가능 |
| Route Handlers | ❌ 불가 (`revalidateTag` 사용) |
| Client Components | ❌ 불가 |
| 기타 컨텍스트 | ❌ 불가 |

---

## 시그니처

```tsx
updateTag(tag: string): void
```

### 매개변수

| 매개변수 | 타입 | 설명 |
|----------|------|------|
| `tag` | `string` | 무효화할 캐시 태그 (최대 256자, 대소문자 구분) |

### 반환값

값을 반환하지 않습니다 (`void`).

---

## 태그 설정 방법

캐시 태그는 두 가지 방법으로 설정할 수 있습니다.

### fetch의 next.tags 옵션

```tsx
fetch(url, { next: { tags: ['posts'] } })
```

### cacheTag 함수 (use cache 지시어 내)

```tsx
import { cacheTag } from 'next/cache'

async function getData() {
  'use cache'
  cacheTag('posts')
  // ...
}
```

---

## 기본 사용법

```ts
// app/actions.ts
'use server'

import { updateTag } from 'next/cache'

export async function updatePost(postId: string, formData: FormData) {
  const title = formData.get('title') as string

  await db.post.update({
    where: { id: postId },
    data: { title },
  })

  // 캐시 태그 즉시 무효화
  updateTag(`post-${postId}`)
}
```

---

## updateTag vs revalidateTag 비교

| 항목 | `updateTag` | `revalidateTag` |
|------|-------------|-----------------|
| **사용 가능 위치** | Server Actions만 | Server Actions, Route Handlers |
| **캐시 처리** | 다음 요청이 신규 데이터 대기 | stale-while-revalidate 가능 |
| **사용 사례** | read-your-own-writes | 일반 캐시 무효화 |
| **즉시성** | 즉시 무효화 | 백그라운드 재검증 가능 |

---

## 실제 예제

### 게시글 생성 후 캐시 업데이트

```ts
// app/actions.ts
'use server'

import { updateTag } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  // 데이터베이스에 게시글 생성
  const post = await db.post.create({
    data: { title, content },
  })

  // 캐시 태그 무효화
  updateTag('posts')           // 게시글 목록 페이지
  updateTag(`post-${post.id}`) // 개별 게시글 페이지

  // 새 게시글로 리다이렉트 (신규 데이터 표시)
  redirect(`/posts/${post.id}`)
}
```

### 댓글 추가 후 캐시 업데이트

```ts
// app/actions.ts
'use server'

import { updateTag } from 'next/cache'

export async function addComment(postId: string, formData: FormData) {
  const content = formData.get('content') as string

  await db.comment.create({
    data: {
      postId,
      content,
    },
  })

  // 해당 게시글의 댓글 캐시 무효화
  updateTag(`comments-${postId}`)
}
```

### 사용자 프로필 업데이트

```ts
// app/actions.ts
'use server'

import { updateTag } from 'next/cache'
import { redirect } from 'next/navigation'

export async function updateProfile(userId: string, formData: FormData) {
  const name = formData.get('name') as string
  const bio = formData.get('bio') as string

  await db.user.update({
    where: { id: userId },
    data: { name, bio },
  })

  // 사용자 관련 캐시 모두 무효화
  updateTag(`user-${userId}`)
  updateTag(`user-${userId}-profile`)
  updateTag(`user-${userId}-posts`)

  redirect(`/users/${userId}`)
}
```

### 여러 태그 동시 업데이트

```ts
// app/actions.ts
'use server'

import { updateTag } from 'next/cache'

export async function publishPost(postId: string) {
  await db.post.update({
    where: { id: postId },
    data: { published: true, publishedAt: new Date() },
  })

  // 관련된 모든 캐시 태그 무효화
  updateTag('posts')
  updateTag('published-posts')
  updateTag(`post-${postId}`)
  updateTag('homepage-featured')
}
```

---

## 잘못된 사용 예제

### Route Handler에서 사용 (오류 발생)

```ts
// app/api/posts/route.ts
import { updateTag } from 'next/cache'
import { revalidateTag } from 'next/cache'

export async function POST() {
  // ❌ 이 코드는 에러를 발생시킵니다
  // Error: updateTag can only be called from within a Server Action
  updateTag('posts')

  // ✅ 대신 revalidateTag 사용
  revalidateTag('posts')

  return Response.json({ success: true })
}
```

---

## 언제 사용해야 하나요?

### updateTag를 사용하는 경우

- Server Action 내에서 작업할 때
- 즉시 캐시 무효화가 필요할 때
- 사용자가 변경 후 바로 업데이트된 데이터를 확인해야 할 때 (read-your-own-writes)

### revalidateTag를 사용하는 경우

- Route Handler나 다른 컨텍스트에서 작업할 때
- stale-while-revalidate 의미론이 필요할 때
- 웹훅이나 API 엔드포인트에서 캐시를 무효화할 때

---

## 중요한 주의사항

> **Good to know**:
> - `updateTag`는 Server Action 내에서만 호출할 수 있습니다
> - Route Handler에서는 `revalidateTag`를 사용하세요
> - 태그 이름은 최대 256자까지 가능합니다
> - 태그 이름은 대소문자를 구분합니다

> **모범 사례**:
> - 데이터 변경 직후 관련된 모든 캐시 태그를 무효화하세요
> - 일관된 태그 명명 규칙을 사용하세요
> - read-your-own-writes가 필요한 경우 `updateTag`를 사용하세요

---

## 관련 문서

- [revalidateTag](./revalidateTag.md)
- [revalidatePath](./revalidatePath.md)
- [cacheTag](./cacheTag.md)
- [Server Actions](../../guides/server-actions-patterns.md)
