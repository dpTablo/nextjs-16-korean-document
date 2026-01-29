---
원문: https://nextjs.org/docs/app/api-reference/functions/permanentRedirect
버전: 16.1.6
---

# permanentRedirect

`permanentRedirect` 함수는 사용자를 다른 URL로 영구적으로 리디렉션합니다. Server Components, Client Components, Route Handlers, Server Actions에서 사용할 수 있습니다.

---

## 함수 시그니처

```typescript
permanentRedirect(path: string, type?: 'replace' | 'push'): never
```

### 매개변수

| 매개변수 | 타입 | 설명 |
|---------|------|------|
| `path` | `string` | 리디렉션할 URL (상대/절대 경로) |
| `type` | `'replace'` \| `'push'` | 리디렉션 방식 (기본값: Server Actions은 'push', 나머지는 'replace') |

---

## 동작 방식

`permanentRedirect` 함수는 컨텍스트에 따라 다르게 동작합니다:

- **스트리밍 컨텍스트**: 클라이언트 측에서 리디렉션을 수행하는 meta 태그 삽입
- **Server Actions**: 303 HTTP 리디렉션 응답 반환
- **기타 (Server Components, Route Handlers)**: **308 (Permanent) HTTP 리디렉션** 응답 반환

---

## 주요 특징

- ✅ **영구 리디렉션**: HTTP 308 상태 코드 사용
- ✅ **반환값 없음**: `never` 타입 사용
- ✅ **절대 URL 지원**: 외부 링크로 리디렉션 가능
- ✅ **다양한 컨텍스트 지원**: Server Components, Client Components (Server Action을 통해), Route Handlers, Server Actions

---

## redirect와의 차이점

| 함수 | HTTP 상태 코드 | 용도 |
|------|---------------|------|
| `redirect` | 307 (Temporary) | 임시 리디렉션 |
| `permanentRedirect` | 308 (Permanent) | 영구 리디렉션, SEO에 유리 |

---

## 사용 예제

### Server Component에서 사용

```tsx
// app/team/[id]/page.tsx
import { permanentRedirect } from 'next/navigation'

async function fetchTeam(id: string) {
  const res = await fetch('https://...')
  if (!res.ok) return undefined
  return res.json()
}

export default async function TeamProfile({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const team = await fetchTeam(id)

  if (!team) {
    permanentRedirect('/teams')
  }

  return <div>Team: {team.name}</div>
}
```

### 리소스 이동 처리

```tsx
// app/blog/old-post-url/page.tsx
import { permanentRedirect } from 'next/navigation'

export default function OldPostPage() {
  // 영구적으로 새 URL로 리디렉션
  permanentRedirect('/blog/new-post-url')
}
```

### Server Action에서 사용

```tsx
// app/actions.ts
'use server'

import { permanentRedirect } from 'next/navigation'

export async function updateUserProfile(formData: FormData) {
  const userId = formData.get('userId')

  // 프로필 업데이트
  await updateProfile(userId)

  // 새 프로필 URL로 영구 리디렉션
  permanentRedirect(`/users/${userId}`)
}
```

### Route Handler에서 사용

```ts
// app/api/old-endpoint/route.ts
import { permanentRedirect } from 'next/navigation'

export async function GET() {
  // API 엔드포인트가 영구적으로 이동됨
  permanentRedirect('/api/v2/new-endpoint')
}
```

### 외부 URL로 리디렉션

```tsx
import { permanentRedirect } from 'next/navigation'

export default function Page() {
  // 외부 URL로 영구 리디렉션
  permanentRedirect('https://new-domain.com/page')
}
```

---

## SEO 고려사항

`permanentRedirect`는 검색 엔진에 페이지가 영구적으로 이동했음을 알립니다.

### 언제 사용해야 하나요?

✅ **permanentRedirect 사용:**
- 페이지가 영구적으로 새 URL로 이동한 경우
- 이전 URL을 제거하고 새 URL로 SEO 값을 전달하려는 경우
- URL 구조를 변경한 경우

❌ **redirect 사용:**
- 임시 리디렉션 (유지보수, A/B 테스팅 등)
- 조건부 리디렉션
- 사용자 행동에 따른 리디렉션

```tsx
// ✅ 좋은 예: 블로그 포스트 URL 구조 변경
// /blog/2024/01/post -> /blog/post
permanentRedirect('/blog/post')

// ❌ 나쁜 예: 임시 유지보수
// redirect('/maintenance')를 사용하세요
```

---

## try/catch 블록과 함께 사용

`permanentRedirect`는 내부적으로 에러를 throw하므로 `try/catch` 블록 **밖에서** 호출해야 합니다.

```tsx
// ❌ 잘못된 사용
export async function ServerAction() {
  try {
    const user = await getUser()
    if (!user) {
      permanentRedirect('/login') // 작동하지 않음!
    }
  } catch (error) {
    console.error(error)
  }
}

// ✅ 올바른 사용
export async function ServerAction() {
  const user = await getUser()

  if (!user) {
    permanentRedirect('/login')
  }

  // 나머지 로직
}
```

---

## type 매개변수

`type` 매개변수로 리디렉션 방식을 제어할 수 있습니다.

```tsx
import { permanentRedirect } from 'next/navigation'

// 기본값: Server Actions에서는 'push', 나머지는 'replace'
permanentRedirect('/new-page')

// 명시적으로 push 사용 (히스토리에 추가)
permanentRedirect('/new-page', 'push')

// 명시적으로 replace 사용 (히스토리 대체)
permanentRedirect('/new-page', 'replace')
```

---

## 중요한 주의사항

> **Good to know**:
> * Client Components에서는 렌더링 프로세스 중에만 사용 가능 (이벤트 핸들러에서는 불가)
> * 이벤트 핸들러에서 리디렉션이 필요한 경우 `useRouter` hook 사용
> * `permanentRedirect`는 에러를 throw하므로 `try/catch` 블록 밖에서 호출
> * TypeScript에서 `never` 타입을 반환하므로 `return` 문 불필요
> * 리소스가 없는 경우 `permanentRedirect` 대신 [`notFound`](./notFound.md) 함수 사용 권장

> **SEO 팁**:
> * 308 상태 코드는 검색 엔진에 페이지 이동이 영구적임을 알립니다
> * 이전 URL의 SEO 가치가 새 URL로 전달됩니다
> * 영구 리디렉션은 검색 엔진 인덱스를 업데이트합니다

---

## HTTP 상태 코드

- **308**: 영구 리디렉션 (기본값, POST 메서드 보존)
- **303**: Server Actions에서 사용 (POST → GET 변환)

---

## 관련 함수

- **[`redirect`](./redirect.md)**: 307 임시 리디렉션
- **[`notFound`](./notFound.md)**: 404 페이지로 리디렉션
- **`useRouter`**: 클라이언트 측 이벤트 핸들러에서의 네비게이션

---

## 버전 히스토리

| 버전 | 변경 사항 |
|------|----------|
| v13.0.0 | `permanentRedirect` 도입 |

---

## 관련 문서

- [redirect](./redirect.md)
- [notFound](./notFound.md)
- [Linking and Navigating](../../getting-started/04-linking-and-navigating.md)
