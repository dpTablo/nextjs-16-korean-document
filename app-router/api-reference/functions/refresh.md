# refresh

`refresh` 함수는 Server Action 내에서 클라이언트 라우터를 새로고침할 수 있게 해줍니다. 데이터 변경 후 페이지를 새로고침하여 최신 데이터를 표시하는 데 사용됩니다.

---

## 사용 제약사항

| 컨텍스트 | 사용 가능 |
|----------|----------|
| Server Actions | ✅ 가능 |
| Route Handlers | ❌ 불가 |
| Client Components | ❌ 불가 |
| 기타 컨텍스트 | ❌ 불가 |

---

## 시그니처

```tsx
refresh(): void
```

### 매개변수

없음

### 반환값

값을 반환하지 않습니다 (`void`).

---

## 기본 사용법

```ts
// app/actions.ts
'use server'

import { refresh } from 'next/cache'

export async function updateData() {
  // 데이터 업데이트 로직
  await db.data.update({ ... })

  // 클라이언트 라우터 새로고침
  refresh()
}
```

---

## 실제 예제

### 게시글 생성 후 새로고침

```ts
// app/actions.ts
'use server'

import { refresh } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  // 데이터베이스에 게시글 생성
  const post = await db.post.create({
    data: { title, content },
  })

  // 클라이언트 라우터 새로고침
  refresh()
}
```

```tsx
// app/posts/page.tsx
import { createPost } from '../actions'

export default function PostsPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="제목" required />
      <textarea name="content" placeholder="내용" required />
      <button type="submit">게시글 작성</button>
    </form>
  )
}
```

### 사용자 프로필 업데이트

```ts
// app/actions.ts
'use server'

import { refresh } from 'next/cache'

export async function updateProfile(formData: FormData) {
  const name = formData.get('name') as string
  const bio = formData.get('bio') as string

  await db.user.update({
    where: { id: getCurrentUserId() },
    data: { name, bio },
  })

  refresh()
}
```

### 댓글 삭제 후 새로고침

```ts
// app/actions.ts
'use server'

import { refresh } from 'next/cache'

export async function deleteComment(commentId: string) {
  await db.comment.delete({
    where: { id: commentId },
  })

  refresh()
}
```

### 좋아요 토글

```ts
// app/actions.ts
'use server'

import { refresh } from 'next/cache'

export async function toggleLike(postId: string, userId: string) {
  const existingLike = await db.like.findFirst({
    where: { postId, userId },
  })

  if (existingLike) {
    await db.like.delete({
      where: { id: existingLike.id },
    })
  } else {
    await db.like.create({
      data: { postId, userId },
    })
  }

  refresh()
}
```

---

## 잘못된 사용 예제

### Route Handler에서 사용 (오류 발생)

```ts
// app/api/posts/route.ts
import { refresh } from 'next/cache'

export async function POST() {
  // ❌ 이 코드는 에러를 발생시킵니다
  refresh()

  return Response.json({ success: true })
}
```

### Client Component에서 사용 (오류 발생)

```tsx
// app/components/RefreshButton.tsx
'use client'

import { refresh } from 'next/cache'

export default function RefreshButton() {
  return (
    // ❌ 이 코드는 에러를 발생시킵니다
    <button onClick={() => refresh()}>
      새로고침
    </button>
  )
}
```

---

## refresh vs revalidatePath vs revalidateTag

| 함수 | 용도 | 범위 |
|------|------|------|
| `refresh` | 클라이언트 라우터 새로고침 | 현재 페이지 |
| `revalidatePath` | 특정 경로의 캐시 재검증 | 지정된 경로 |
| `revalidateTag` | 특정 태그의 캐시 재검증 | 지정된 태그 |

### 사용 시나리오

```ts
// app/actions.ts
'use server'

import { refresh } from 'next/cache'
import { revalidatePath } from 'next/cache'
import { revalidateTag } from 'next/cache'

export async function updatePost(postId: string, formData: FormData) {
  await db.post.update({
    where: { id: postId },
    data: { ... },
  })

  // 현재 페이지만 새로고침
  refresh()

  // 또는 특정 경로의 캐시 재검증
  // revalidatePath(`/posts/${postId}`)

  // 또는 특정 태그의 캐시 재검증
  // revalidateTag(`post-${postId}`)
}
```

---

## 중요한 주의사항

> **Good to know**:
> - `refresh`는 Server Action 내에서만 호출할 수 있습니다
> - Route Handler나 Client Component에서 호출하면 에러가 발생합니다
> - `refresh`는 현재 페이지의 클라이언트 라우터만 새로고침합니다

> **언제 사용해야 하나요?**:
> - 데이터 변경 후 즉시 UI를 업데이트해야 할 때
> - 특정 캐시 태그나 경로가 아닌 현재 페이지 전체를 새로고침해야 할 때
> - `revalidatePath`나 `revalidateTag`로는 충분하지 않을 때

---

## 관련 문서

- [revalidatePath](./revalidatePath.md)
- [revalidateTag](./revalidateTag.md)
- [Server Actions](../../guides/server-actions-patterns.md)
- [Forms and Mutations](../../guides/forms.md)
