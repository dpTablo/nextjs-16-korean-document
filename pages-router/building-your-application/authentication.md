# Authentication (인증)

인증은 애플리케이션의 데이터를 보호하는 데 매우 중요합니다. 세 가지 핵심 개념이 있습니다:

1. **인증(Authentication)**: 자격 증명(사용자명/비밀번호)을 통해 사용자 신원을 확인합니다
2. **세션 관리(Session Management)**: 요청 간에 사용자 인증 상태를 추적합니다
3. **권한 부여(Authorization)**: 사용자가 접근할 수 있는 라우트와 데이터를 제어합니다

## 인증 구현

### 로그인 폼 컴포넌트

```tsx
// pages/login.tsx
import { FormEvent } from 'react'
import { useRouter } from 'next/router'

export default function LoginPage() {
  const router = useRouter()

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault()

    const formData = new FormData(event.currentTarget)
    const email = formData.get('email')
    const password = formData.get('password')

    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    })

    if (response.ok) {
      router.push('/profile')
    } else {
      // 에러 처리
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" name="email" placeholder="이메일" required />
      <input type="password" name="password" placeholder="비밀번호" required />
      <button type="submit">로그인</button>
    </form>
  )
}
```

### API Route 핸들러

```ts
// pages/api/auth/login.ts
import type { NextApiRequest, NextApiResponse } from 'next'
import { signIn } from '@/auth'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const { email, password } = req.body
    await signIn('credentials', { email, password })

    res.status(200).json({ success: true })
  } catch (error) {
    if (error.type === 'CredentialsSignin') {
      res.status(401).json({ error: '유효하지 않은 자격 증명입니다.' })
    } else {
      res.status(500).json({ error: '문제가 발생했습니다.' })
    }
  }
}
```

## 세션 관리

### 무상태 세션 (쿠키 기반)

```ts
// pages/api/login.ts
import { serialize } from 'cookie'
import type { NextApiRequest, NextApiResponse } from 'next'
import { encrypt } from '@/app/lib/session'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const sessionData = req.body
  const encryptedSessionData = encrypt(sessionData)

  const cookie = serialize('session', encryptedSessionData, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24 * 7, // 1주일
    path: '/',
  })
  res.setHeader('Set-Cookie', cookie)
  res.status(200).json({ message: '쿠키가 성공적으로 설정되었습니다!' })
}
```

**주요 특징:**
- 세션 데이터가 브라우저 쿠키에 저장됨
- 구현이 간단함
- 제대로 구현하지 않으면 보안이 취약할 수 있음
- 암호화된 세션 데이터 필요

### 데이터베이스 세션

```ts
// pages/api/create-session.ts
import db from '../../lib/db'
import type { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const user = req.body
    const sessionId = generateSessionId()
    await db.insertSession({
      sessionId,
      userId: user.id,
      createdAt: new Date(),
    })

    res.status(200).json({ sessionId })
  } catch (error) {
    res.status(500).json({ error: '내부 서버 오류' })
  }
}
```

**주요 특징:**
- 세션 데이터가 데이터베이스에 저장됨
- 더 안전한 접근 방식
- 브라우저는 암호화된 세션 ID만 받음
- 구현이 더 복잡함
- 서버 리소스 사용량이 더 높음

**권장 라이브러리:**
- [iron-session](https://github.com/vvo/iron-session)
- [Jose](https://github.com/panva/jose)

## 권한 부여

### API Routes 보호

```ts
// pages/api/route.ts
import { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const session = await getSession(req)

  // 사용자가 인증되었는지 확인
  if (!session) {
    res.status(401).json({
      error: '사용자가 인증되지 않았습니다',
    })
    return
  }

  // 사용자가 'admin' 역할을 가지고 있는지 확인
  if (session.user.role !== 'admin') {
    res.status(401).json({
      error: '권한 없음: 사용자에게 관리자 권한이 없습니다.',
    })
    return
  }

  // 권한이 있는 사용자를 위한 라우트 진행
  // ... API Route 구현
}
```

### getServerSideProps에서 인증

```tsx
// pages/profile.tsx
import { GetServerSideProps } from 'next'
import { getSession } from '@/lib/auth'

export const getServerSideProps: GetServerSideProps = async (context) => {
  const session = await getSession(context.req)

  if (!session) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    }
  }

  return {
    props: {
      user: session.user,
    },
  }
}

export default function ProfilePage({ user }) {
  return <div>환영합니다, {user.name}님!</div>
}
```

### 권한 확인 유형

1. **낙관적 확인**: 빠른 UI 결정을 위해 쿠키의 세션 데이터 사용
2. **보안 확인**: 민감한 작업에 대해 데이터베이스 검증

### 모범 사례

- 권한 부여 로직을 중앙화하기 위해 데이터 접근 계층(DAL) 생성
- 필요한 데이터만 반환하기 위해 데이터 전송 객체(DTO) 사용
- 선택적으로 낙관적 확인을 위해 미들웨어/프록시 사용
- 항상 데이터 소스 가까이에서 보안 확인 수행

## 권장 인증 라이브러리

- Auth0
- Better Auth
- Clerk
- Descope
- Kinde
- Logto
- NextAuth.js
- Ory
- Stack Auth
- Supabase
- Stytch
- WorkOS

## 핵심 권장사항

> **참고:** 교육 목적으로 커스텀 인증 구현이 가능하지만, 프로덕션 애플리케이션에는 내장 보안 기능, 소셜 로그인, 다단계 인증, 역할 기반 접근 제어를 제공하는 확립된 인증 라이브러리 사용을 권장합니다.

---

## 참고

- [API Routes](/pages-router/building-your-application/routing/api-routes.md)
- [getServerSideProps](/pages-router/api-reference/getServerSideProps.md)
- [환경 변수](/pages-router/building-your-application/configuring/environment-variables.md)
