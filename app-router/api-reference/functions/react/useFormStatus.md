---
원문: https://nextjs.org/docs/app/api-reference/functions/use-form-status
버전: 16.1.6
---

# useFormStatus

## 개요

`useFormStatus`는 마지막 폼 제출에 대한 상태 정보를 제공하는 **React DOM Hook**입니다. React 19.2의 향상된 폼 처리 기능의 일부로, Next.js의 Server Actions와 함께 사용하면 강력한 폼 UI를 구현할 수 있습니다.

> **참고:** 이것은 React Hook이지만 Next.js의 Server Actions와 함께 자주 사용되므로 여기에 문서화되어 있습니다.

---

## 기본 사용법

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function Submit() {
  const { pending, data, method, action } = useFormStatus()

  return (
    <button type="submit" disabled={pending}>
      {pending ? '제출 중...' : '제출'}
    </button>
  )
}
```

---

## API 레퍼런스

### 함수 시그니처

```tsx
const status = useFormStatus()
```

### 반환 값

`status` 객체는 4가지 속성을 포함합니다:

| 속성 | 타입 | 설명 |
|------|------|------|
| `pending` | `boolean` | 부모 폼이 제출 중이면 `true`, 아니면 `false` |
| `data` | `FormData \| null` | 제출 중인 폼 데이터, 제출 중이 아니면 `null` |
| `method` | `'get' \| 'post'` | 부모 폼의 HTTP 메서드 (기본값: 'GET') |
| `action` | `function \| string \| null` | action 함수 참조 또는 URI, 없으면 `null` |

---

## 중요한 요구사항 및 주의사항

### ⚠️ 폼의 자식 컴포넌트에서 호출해야 함

`useFormStatus`는 **반드시 `<form>`의 자식 컴포넌트**에서 호출해야 합니다. 폼을 렌더링하는 같은 컴포넌트에서는 작동하지 않습니다.

✅ **올바른 패턴:**
```tsx
'use client'

import { useFormStatus } from 'react-dom'

function Submit() {
  const { pending } = useFormStatus() // ✅ 작동함
  return <button disabled={pending}>제출</button>
}

function Form() {
  return (
    <form action={submitAction}>
      <Submit /> {/* useFormStatus가 여기서 호출됨 */}
    </form>
  )
}
```

❌ **잘못된 패턴:**
```tsx
'use client'

import { useFormStatus } from 'react-dom'

function Form() {
  const { pending } = useFormStatus() // ❌ 이 폼을 추적하지 못함

  return (
    <form action={submitAction}>
      <button disabled={pending}>제출</button>
    </form>
  )
}
```

---

## Next.js에서의 실용적인 예제

### 1. Server Action과 함께 사용

```tsx
// app/actions.ts
'use server'

export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')

  // 데이터베이스에 저장
  await db.posts.create({
    data: { title, content }
  })

  // 캐시 재검증
  revalidatePath('/posts')
}
```

```tsx
// app/components/PostForm.tsx
'use client'

import { useFormStatus } from 'react-dom'
import { createPost } from '@/app/actions'

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button type="submit" disabled={pending}>
      {pending ? '작성 중...' : '포스트 작성'}
    </button>
  )
}

export default function PostForm() {
  return (
    <form action={createPost}>
      <input type="text" name="title" placeholder="제목" />
      <textarea name="content" placeholder="내용" />
      <SubmitButton />
    </form>
  )
}
```

### 2. 로딩 스피너 표시

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button
      type="submit"
      disabled={pending}
      className={pending ? 'opacity-50 cursor-not-allowed' : ''}
    >
      {pending && (
        <svg className="animate-spin h-5 w-5 mr-3" viewBox="0 0 24 24">
          {/* 스피너 아이콘 */}
        </svg>
      )}
      {pending ? '로딩 중...' : '제출'}
    </button>
  )
}
```

### 3. 제출 중인 데이터 표시

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function FormFields() {
  const { pending, data } = useFormStatus()

  return (
    <div>
      <input
        type="text"
        name="username"
        disabled={pending}
        placeholder="사용자명"
      />
      <button type="submit" disabled={pending}>
        제출
      </button>
      {data && (
        <p className="text-gray-600">
          {data.get('username')} 처리 중...
        </p>
      )}
    </div>
  )
}

export default function UsernameForm() {
  return (
    <form action={updateUsername}>
      <FormFields />
    </form>
  )
}
```

### 4. 모든 입력 필드 비활성화

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function FormInputs() {
  const { pending } = useFormStatus()

  return (
    <>
      <input
        type="text"
        name="name"
        disabled={pending}
        placeholder="이름"
      />
      <input
        type="email"
        name="email"
        disabled={pending}
        placeholder="이메일"
      />
      <textarea
        name="message"
        disabled={pending}
        placeholder="메시지"
      />
      <button type="submit" disabled={pending}>
        {pending ? '전송 중...' : '전송'}
      </button>
    </>
  )
}
```

### 5. 진행률 표시

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function ProgressBar() {
  const { pending } = useFormStatus()

  if (!pending) return null

  return (
    <div className="w-full bg-gray-200 rounded-full h-2.5">
      <div
        className="bg-blue-600 h-2.5 rounded-full animate-pulse"
        style={{ width: '100%' }}
      />
    </div>
  )
}

export default function Form() {
  return (
    <form action={submitAction}>
      <ProgressBar />
      <input type="text" name="data" />
      <SubmitButton />
    </form>
  )
}
```

### 6. 여러 제출 버튼

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function SaveButton() {
  const { pending, data } = useFormStatus()
  const isSaving = pending && data?.get('action') === 'save'

  return (
    <button
      type="submit"
      name="action"
      value="save"
      disabled={pending}
    >
      {isSaving ? '저장 중...' : '저장'}
    </button>
  )
}

function PublishButton() {
  const { pending, data } = useFormStatus()
  const isPublishing = pending && data?.get('action') === 'publish'

  return (
    <button
      type="submit"
      name="action"
      value="publish"
      disabled={pending}
    >
      {isPublishing ? '발행 중...' : '발행'}
    </button>
  )
}

export default function PostForm() {
  return (
    <form action={handlePost}>
      <input type="text" name="title" />
      <textarea name="content" />
      <div className="flex gap-2">
        <SaveButton />
        <PublishButton />
      </div>
    </form>
  )
}
```

### 7. 에러 메시지와 함께 사용

```tsx
'use client'

import { useFormStatus } from 'react-dom'
import { useState } from 'react'

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button type="submit" disabled={pending}>
      {pending ? '확인 중...' : '로그인'}
    </button>
  )
}

export default function LoginForm() {
  const [error, setError] = useState('')

  async function handleLogin(formData: FormData) {
    try {
      await loginAction(formData)
    } catch (err) {
      setError('로그인에 실패했습니다.')
    }
  }

  return (
    <form action={handleLogin}>
      <input type="email" name="email" />
      <input type="password" name="password" />
      {error && <p className="text-red-500">{error}</p>}
      <SubmitButton />
    </form>
  )
}
```

---

## 고급 패턴

### 1. 중첩된 폼 컴포넌트

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function NestedInput({ name, label }: { name: string; label: string }) {
  const { pending } = useFormStatus()

  return (
    <div>
      <label>{label}</label>
      <input type="text" name={name} disabled={pending} />
    </div>
  )
}

function SubmitSection() {
  const { pending } = useFormStatus()

  return (
    <div>
      <SubmitButton />
      {pending && <p>제출 중입니다. 잠시만 기다려주세요...</p>}
    </div>
  )
}

export default function ComplexForm() {
  return (
    <form action={submitAction}>
      <NestedInput name="firstName" label="이름" />
      <NestedInput name="lastName" label="성" />
      <NestedInput name="email" label="이메일" />
      <SubmitSection />
    </form>
  )
}
```

### 2. 조건부 UI 렌더링

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function FormContent() {
  const { pending, data } = useFormStatus()

  if (pending) {
    return (
      <div className="text-center">
        <Spinner />
        <p>"{data?.get('name')}" 처리 중...</p>
      </div>
    )
  }

  return (
    <>
      <input type="text" name="name" placeholder="이름" />
      <input type="email" name="email" placeholder="이메일" />
      <button type="submit">제출</button>
    </>
  )
}
```

### 3. 메서드별 다른 UI

```tsx
'use client'

import { useFormStatus } from 'react-dom'

function FormStatus() {
  const { pending, method } = useFormStatus()

  if (!pending) return null

  return (
    <p>
      {method === 'post' ? '데이터 생성 중...' : '데이터 조회 중...'}
    </p>
  )
}
```

---

## 베스트 프랙티스

### ✅ 권장사항

1. **별도의 버튼 컴포넌트 생성**
   ```tsx
   // ✅ 재사용 가능한 컴포넌트
   function SubmitButton() {
     const { pending } = useFormStatus()
     return <button disabled={pending}>제출</button>
   }
   ```

2. **적절한 로딩 상태 표시**
   ```tsx
   // ✅ 사용자에게 명확한 피드백
   {pending ? '저장 중...' : '저장'}
   ```

3. **pending 상태에서 입력 비활성화**
   ```tsx
   // ✅ 중복 제출 방지
   <input disabled={pending} />
   ```

### ❌ 피해야 할 사항

1. **폼과 같은 레벨에서 호출**
   ```tsx
   // ❌ 작동하지 않음
   function Form() {
     const { pending } = useFormStatus()
     return <form>...</form>
   }
   ```

2. **Server Component에서 사용**
   ```tsx
   // ❌ Client Component에서만 사용 가능
   export default function ServerForm() {
     const { pending } = useFormStatus() // 에러!
   }
   ```

3. **pending 없이 비활성화**
   ```tsx
   // ❌ 항상 비활성화됨
   <button disabled>제출</button>

   // ✅ pending 상태에 따라 비활성화
   <button disabled={pending}>제출</button>
   ```

---

## 문제 해결

### Q: `status.pending`이 항상 `false`입니다

**해결 방법:**
1. `useFormStatus()`를 호출하는 컴포넌트가 `<form>` 요소 **안에** 렌더링되는지 확인
2. 폼을 렌더링하는 컴포넌트와 다른 컴포넌트에서 호출하는지 확인
3. 폼에 유효한 `action` prop이 있는지 확인

```tsx
// ❌ 작동하지 않음
function Form() {
  const { pending } = useFormStatus()
  return <form action={submit}><button /></form>
}

// ✅ 작동함
function Submit() {
  const { pending } = useFormStatus()
  return <button />
}

function Form() {
  return <form action={submit}><Submit /></form>
}
```

### Q: TypeScript 타입 에러가 발생합니다

**해결 방법:**
React 19.2 이상과 `@types/react-dom` 타입 정의가 필요합니다:

```bash
npm install react@latest react-dom@latest
npm install -D @types/react@latest @types/react-dom@latest
```

### Q: 여러 폼이 있을 때 어떻게 하나요?

**해결 방법:**
`useFormStatus`는 부모 폼만 추적합니다. 각 폼마다 별도의 버튼 컴포넌트를 사용하세요:

```tsx
function Form1() {
  return (
    <form action={action1}>
      <Submit1Button />
    </form>
  )
}

function Form2() {
  return (
    <form action={action2}>
      <Submit2Button />
    </form>
  )
}
```

---

## 타입 정의

```typescript
interface FormStatus {
  pending: boolean
  data: FormData | null
  method: 'get' | 'post'
  action: ((formData: FormData) => void | Promise<void>) | string | null
}

function useFormStatus(): FormStatus
```

---

## 관련 API

- [Server Actions](../../../guides/forms.md) - 폼 처리 가이드
- [`useActionState`](./useActionState.md) - 폼 상태 관리 (React 19)
- [`useOptimistic`](./useOptimistic.md) - 낙관적 UI 업데이트
- [`revalidatePath`](../revalidatePath.md) - 경로 재검증

---

## 추가 리소스

- [React 공식 문서 - useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus)
- [Next.js Forms 가이드](../../../guides/forms.md)
- [Server Actions 패턴](../../../guides/server-actions-patterns.md)

---

## 버전 정보

- **도입 버전:** React 19.0.0
- **안정 버전:** React 19.2.0
- **Next.js 지원:** Next.js 14.0.0+
- **현재 상태:** Stable

---

## 요약

- **용도:** 폼 제출 상태 추적
- **주요 속성:** `pending`, `data`, `method`, `action`
- **제약:** 반드시 폼의 자식 컴포넌트에서 호출
- **주 사용처:** 제출 버튼 로딩 상태, 입력 비활성화
- **Next.js 통합:** Server Actions와 완벽 호환
- **권장사항:** 별도의 버튼 컴포넌트로 분리
