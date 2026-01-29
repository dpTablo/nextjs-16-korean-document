---
원문: https://nextjs.org/docs/app/api-reference/directives/use-server
버전: 16.1.6
---

# 'use server' 지시어

## 개요
`use server` 지시어는 함수나 파일이 **서버 사이드**에서 실행되도록 지정합니다. 다음과 같이 적용할 수 있습니다:
- **파일 레벨**: 파일의 모든 함수가 서버 사이드
- **인라인**: 개별 함수를 서버 함수로 표시

---

## 사용 패턴

### 1. 파일 레벨 선언
파일 최상단에 `'use server'`를 사용하여 모든 export된 함수를 서버 함수로 표시합니다.

**TypeScript 예시:**
```tsx
// app/actions.ts
'use server'
import { db } from '@/lib/db'

export async function createUser(data: { name: string; email: string }) {
  const user = await db.user.create({ data })
  return user
}
```

### 2. 인라인 선언
서버 컴포넌트 내에서 개별 비동기 함수를 서버 함수로 표시합니다.

**TypeScript 예시:**
```tsx
// app/posts/[id]/page.tsx
import { EditPost } from './edit-post'
import { revalidatePath } from 'next/cache'

export default async function PostPage({ params }: { params: { id: string } }) {
  const post = await getPost(params.id)

  async function updatePost(formData: FormData) {
    'use server'
    await savePost(params.id, formData)
    revalidatePath(`/posts/${params.id}`)
  }

  return <EditPost updatePostAction={updatePost} post={post} />
}
```

---

## 클라이언트 컴포넌트에서 서버 함수 사용

1. 최상단에 `'use server'`가 있는 전용 파일에서 **서버 함수 생성**
2. 클라이언트 컴포넌트로 **Import**
3. 클라이언트 사이드 이벤트에서 **실행**

**예시:**

**서버 파일 (app/actions.ts):**
```tsx
'use server'
import { db } from '@/lib/db'

export async function fetchUsers() {
  const users = await db.user.findMany()
  return users
}
```

**클라이언트 컴포넌트 (app/components/my-button.tsx):**
```tsx
'use client'
import { fetchUsers } from '../actions'

export default function MyButton() {
  return <button onClick={() => fetchUsers()}>사용자 가져오기</button>
}
```

---

## 보안 고려사항

### 인증 및 권한 부여
민감한 작업을 수행하기 전에 항상 사용자를 인증하고 권한을 부여하세요:

```tsx
'use server'
import { db } from '@/lib/db'
import { authenticate } from '@/lib/auth'

export async function createUser(
  data: { name: string; email: string },
  token: string
) {
  const user = authenticate(token)
  if (!user) {
    throw new Error('인증되지 않음')
  }
  const newUser = await db.user.create({ data })
  return newUser
}
```

**주요 보안 사항:**
- 모든 입력 검증
- 사용자 인증/권한 확인
- 민감한 데이터를 서버 사이드에만 유지
- 안전한 인증 메커니즘 사용

---

## 참조
추가 정보는 [React Server Functions 문서](https://react.dev/reference/rsc/use-server)를 참조하세요.
