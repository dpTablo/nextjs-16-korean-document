---
원문: https://nextjs.org/docs/app/api-reference/components/form
버전: 16.1.6
---

# Form

`<Form>` 컴포넌트는 HTML `<form>` 요소를 확장하여 **프리페칭**, **클라이언트 측 네비게이션**, **점진적 개선**을 제공합니다.

---

## 기본 사용법

```tsx
import Form from 'next/form'

export default function Page() {
  return (
    <Form action="/search">
      {/* 제출 시 입력값이 URL에 추가됨: /search?query=abc */}
      <input name="query" />
      <button type="submit">Submit</button>
    </Form>
  )
}
```

---

## 동작 원리

### String 액션 (문자열 경로)

`action`에 문자열을 전달하면 HTML GET 폼처럼 동작합니다.

**특징:**
- 폼 데이터가 URL 검색 매개변수로 인코딩됨
- **자동 프리페칭**: 폼이 뷰포트에 보이면 공유 UI(`layout.js`, `loading.js`)를 프리페칭
- **클라이언트 사이드 네비게이션**: 전체 페이지 새로고침 없이 이동
- 공유 레이아웃과 클라이언트 측 상태 유지

### Function 액션 (서버 액션)

`action`에 함수를 전달하면 React 폼처럼 동작합니다.

**특징:**
- 폼 제출 시 서버 액션 실행
- 데이터 뮤테이션에 적합

---

## Props

### String Action Props

| Prop | 예시 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| `action` | `action="/search"` | `string` | ✓ | 제출 시 이동할 URL |
| `replace` | `replace={false}` | `boolean` | - | 히스토리 스택 대체 여부 |
| `scroll` | `scroll={true}` | `boolean` | - | 스크롤 동작 제어 |
| `prefetch` | `prefetch={true}` | `boolean` | - | 프리페칭 활성화 여부 |

#### action

폼 제출 시 이동할 URL 경로입니다.

- 빈 문자열(`""`)을 사용하면 동일 경로에서 검색 매개변수만 업데이트합니다

```tsx
<Form action="/search">
  <input name="query" />
  <button type="submit">검색</button>
</Form>
```

#### replace

`true`로 설정하면 브라우저 히스토리 스택에 새 항목을 추가하는 대신 현재 상태를 대체합니다.

- 기본값: `false`

```tsx
<Form action="/search" replace={true}>
  <input name="query" />
  <button type="submit">검색</button>
</Form>
```

#### scroll

스크롤 동작을 제어합니다.

- 기본값: `true` (페이지 상단으로 스크롤)
- `false`로 설정하면 스크롤 위치 유지

```tsx
<Form action="/search" scroll={false}>
  <input name="query" />
  <button type="submit">검색</button>
</Form>
```

#### prefetch

폼이 뷰포트에 보일 때 대상 경로를 프리페칭할지 여부를 설정합니다.

- 기본값: `true`

```tsx
<Form action="/search" prefetch={false}>
  <input name="query" />
  <button type="submit">검색</button>
</Form>
```

### Function Action Props

| Prop | 예시 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| `action` | `action={myAction}` | `function` | ✓ | 실행할 서버 액션 |

```tsx
import Form from 'next/form'
import { createPost } from '@/app/actions'

export default function Page() {
  return (
    <Form action={createPost}>
      <input name="title" />
      <button type="submit">생성</button>
    </Form>
  )
}
```

---

## 제약사항

`<Form>` 컴포넌트 사용 시 다음 제약사항을 고려하세요:

| 항목 | 설명 |
|------|------|
| `formAction` | `<button>` 또는 `<input type="submit">`에서 `action`을 오버라이드할 수 있으나 프리페칭 미지원 |
| `key` | 문자열 `action`에서 `key` prop 미지원. 리렌더가 필요하면 함수 `action` 사용 |
| `onSubmit` | `event.preventDefault()` 호출 시 `<Form>` 동작이 무시됨 |
| `method`, `encType`, `target` | 미지원. 필요시 HTML `<form>` 사용 권장 |
| `<input type="file">` | 문자열 `action`에서는 파일명만 제출됨 (파일 객체가 아님) |

---

## 예제

### 검색 폼과 결과 페이지

**검색 폼:**

```tsx
// app/page.tsx
import Form from 'next/form'

export default function Page() {
  return (
    <Form action="/search">
      <input name="query" placeholder="검색어를 입력하세요" />
      <button type="submit">검색</button>
    </Form>
  )
}
```

**검색 결과 페이지:**

```tsx
// app/search/page.tsx
import { getSearchResults } from '@/lib/search'

export default async function SearchPage({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const { query } = await searchParams
  const results = await getSearchResults(query)

  return (
    <div>
      <h1>"{query}" 검색 결과</h1>
      {results.map((result) => (
        <div key={result.id}>{result.title}</div>
      ))}
    </div>
  )
}
```

**로딩 UI:**

```tsx
// app/search/loading.tsx
export default function Loading() {
  return <div>검색 중...</div>
}
```

### 제출 버튼 상태 표시

`useFormStatus` Hook을 사용하여 폼 제출 중 상태를 표시할 수 있습니다.

```tsx
// app/ui/search-button.tsx
'use client'

import { useFormStatus } from 'react-dom'

export default function SearchButton() {
  const { pending } = useFormStatus()

  return (
    <button type="submit" disabled={pending}>
      {pending ? '검색 중...' : '검색'}
    </button>
  )
}
```

```tsx
// app/page.tsx
import Form from 'next/form'
import SearchButton from './ui/search-button'

export default function Page() {
  return (
    <Form action="/search">
      <input name="query" />
      <SearchButton />
    </Form>
  )
}
```

### 서버 액션을 통한 데이터 생성

**폼 페이지:**

```tsx
// app/posts/create/page.tsx
import Form from 'next/form'
import { createPost } from '@/app/posts/actions'

export default function CreatePostPage() {
  return (
    <Form action={createPost}>
      <label>
        제목
        <input name="title" required />
      </label>
      <label>
        내용
        <textarea name="content" required />
      </label>
      <button type="submit">게시글 생성</button>
    </Form>
  )
}
```

**서버 액션:**

```tsx
// app/posts/actions.ts
'use server'

import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  // 데이터베이스에 저장
  const post = await db.post.create({
    data: { title, content },
  })

  // 생성된 게시글로 리디렉션
  redirect(`/posts/${post.id}`)
}
```

**동적 결과 페이지:**

```tsx
// app/posts/[id]/page.tsx
import { getPost } from '@/lib/posts'
import { notFound } from 'next/navigation'

export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post = await getPost(id)

  if (!post) {
    notFound()
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

### 필터링 폼

동일 페이지에서 검색 매개변수만 업데이트하는 필터링 폼입니다.

```tsx
// app/products/page.tsx
import Form from 'next/form'

export default function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ category?: string; sort?: string }>
}) {
  return (
    <div>
      <Form action="">
        <select name="category">
          <option value="">전체</option>
          <option value="electronics">전자제품</option>
          <option value="clothing">의류</option>
        </select>
        <select name="sort">
          <option value="newest">최신순</option>
          <option value="price-low">가격 낮은순</option>
          <option value="price-high">가격 높은순</option>
        </select>
        <button type="submit">필터 적용</button>
      </Form>

      {/* 제품 목록 */}
    </div>
  )
}
```

---

## HTML form과의 비교

| 기능 | `<Form>` | HTML `<form>` |
|------|----------|---------------|
| 프리페칭 | ✅ 자동 | ❌ 없음 |
| 클라이언트 네비게이션 | ✅ 지원 | ❌ 전체 새로고침 |
| 레이아웃 유지 | ✅ 지원 | ❌ 불가 |
| 점진적 개선 | ✅ 지원 | ✅ 기본 지원 |
| 서버 액션 | ✅ 지원 | ✅ 지원 |

---

## 중요한 주의사항

> **Good to know**:
> - 문자열 `action`은 GET 요청으로 처리됩니다
> - 함수 `action`은 POST 요청으로 처리됩니다
> - 프리페칭은 JavaScript가 활성화된 경우에만 동작합니다
> - JavaScript가 비활성화되어도 폼은 기본 HTML 동작으로 작동합니다 (점진적 개선)

> **성능 팁**:
> - 검색 폼에는 `loading.js`를 함께 구현하여 즉각적인 피드백 제공
> - 자주 방문하는 경로에는 `prefetch`를 활성화 상태로 유지
> - 히스토리가 불필요한 필터에는 `replace={true}` 사용

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.0.0 | `<Form>` 컴포넌트 도입 |

---

## 관련 문서

- [Server Actions](../functions/redirect.md)
- [useFormStatus](../functions/react/useFormStatus.md)
- [useActionState](../functions/react/useActionState.md)
- [Forms and Mutations](../../guides/forms.md)
