---
원문: https://nextjs.org/docs/app/guides/forms
버전: 16.1.6
---

# Server Actions로 폼 만들기

## 개요

React Server Actions는 서버에서 실행되는 서버 함수로, 서버 및 클라이언트 컴포넌트에서 폼 제출을 처리하기 위해 호출될 수 있습니다.

## 작동 방식

React는 HTML `<form>` 요소를 확장하여 `action` 속성으로 Server Actions를 호출할 수 있게 합니다. 함수는 폼 데이터를 포함하는 `FormData` 객체를 자동으로 받습니다.

### 기본 예시

```tsx
export default function Page() {
  async function createInvoice(formData: FormData) {
    'use server'

    const rawFormData = {
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    }

    // 데이터 변경
    // 캐시 재검증
  }

  return <form action={createInvoice}>...</form>
}
```

**참고:** 여러 필드가 있는 폼의 경우 `Object.fromEntries(formData)`를 사용하세요. 이는 `$ACTION_`으로 접두사가 붙은 추가 속성을 포함합니다.

## 추가 인수 전달

JavaScript `bind` 메서드를 사용하여 서버 함수에 추가 인수를 전달하세요:

```tsx
'use client'

import { updateUser } from './actions'

export function UserProfile({ userId }: { userId: string }) {
  const updateUserWithId = updateUser.bind(null, userId)

  return (
    <form action={updateUserWithId}>
      <input type="text" name="name" />
      <button type="submit">사용자 이름 업데이트</button>
    </form>
  )
}
```

서버 함수 시그니처:
```ts
'use server'

export async function updateUser(userId: string, formData: FormData) {}
```

**대안:** 숨겨진 입력 필드 사용 (예: `<input type="hidden" name="userId" value={userId} />`), 다만 값이 렌더링된 HTML의 일부가 되며 인코딩되지 않습니다.

## 폼 검증

### 클라이언트 사이드 검증
기본 검증을 위해 `required` 및 `type="email"`과 같은 HTML 속성을 사용하세요.

### 서버 사이드 검증
[zod](https://zod.dev/)와 같은 라이브러리를 사용하세요:

```tsx
'use server'

import { z } from 'zod'

const schema = z.object({
  email: z.string({
    invalid_type_error: '잘못된 이메일',
  }),
})

export default async function createUser(formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
  })

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }

  // 데이터 변경
}
```

## 검증 에러

폼 컴포넌트를 클라이언트 컴포넌트로 전환하고 `useActionState` 훅을 사용하세요:

```tsx
'use server'

export async function createUser(initialState: any, formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
  })
  // ...
}
```

클라이언트 컴포넌트:
```tsx
'use client'

import { useActionState } from 'react'
import { createUser } from '@/app/actions'

const initialState = {
  message: '',
}

export function Signup() {
  const [state, formAction, pending] = useActionState(createUser, initialState)

  return (
    <form action={formAction}>
      <label htmlFor="email">이메일</label>
      <input type="text" id="email" name="email" required />
      <p aria-live="polite">{state?.message}</p>
      <button disabled={pending}>가입하기</button>
    </form>
  )
}
```

## 대기 상태

`useActionState` 훅은 로딩 인디케이터를 표시하기 위한 `pending` boolean을 노출합니다:

```tsx
const [state, formAction, pending] = useActionState(createUser, initialState)

return (
  <form action={formAction}>
    <button disabled={pending}>가입하기</button>
  </form>
)
```

### 대안: useFormStatus

별도 컴포넌트에서 `useFormStatus` 훅을 사용하세요:

```tsx
'use client'

import { useFormStatus } from 'react-dom'

export function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button disabled={pending} type="submit">
      가입하기
    </button>
  )
}
```

그런 다음 폼에 중첩하세요:

```tsx
export function Signup() {
  return (
    <form action={createUser}>
      <SubmitButton />
    </form>
  )
}
```

**참고:** React 19에서 `useFormStatus`는 추가 키를 포함합니다: `data`, `method`, `action`. 이전 버전은 `pending`만 있습니다.

## 낙관적 업데이트

서버 함수가 완료되기 전에 UI를 업데이트하려면 `useOptimistic` 훅을 사용하세요:

```tsx
'use client'

import { useOptimistic } from 'react'
import { send } from './actions'

export function Thread({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic<
    Message[],
    string
  >(messages, (state, newMessage) => [...state, { message: newMessage }])

  const formAction = async (formData: FormData) => {
    const message = formData.get('message') as string
    addOptimisticMessage(message)
    await send(message)
  }

  return (
    <div>
      {optimisticMessages.map((m, i) => (
        <div key={i}>{m.message}</div>
      ))}
      <form action={formAction}>
        <input type="text" name="message" />
        <button type="submit">전송</button>
      </form>
    </div>
  )
}
```

## 중첩 폼 요소

`<button>`, `<input type="submit">`, `<input type="image">`와 같은 중첩 요소에서 `formAction` prop이나 이벤트 핸들러를 사용하여 Server Actions를 호출하세요. 이를 통해 단일 폼 내에서 여러 Server Actions를 사용할 수 있습니다.

## 프로그래매틱 폼 제출

`requestSubmit()` 메서드를 사용하여 폼 제출을 트리거하세요:

```tsx
'use client'

export function Entry() {
  const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    if (
      (e.ctrlKey || e.metaKey) &&
      (e.key === 'Enter' || e.key === 'NumpadEnter')
    ) {
      e.preventDefault()
      e.currentTarget.form?.requestSubmit()
    }
  }

  return (
    <div>
      <textarea name="entry" rows={20} required onKeyDown={handleKeyDown} />
    </div>
  )
}
```

이는 가장 가까운 `<form>` 조상의 제출을 트리거하고 서버 함수를 호출합니다.
