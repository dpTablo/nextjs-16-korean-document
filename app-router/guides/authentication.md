---
원문: https://nextjs.org/docs/app/guides/authentication
버전: 16.1.6
---

# 인증

## 개요

인증은 사용자가 본인이 주장하는 사람이 맞는지 확인합니다. 사용자 이름과 비밀번호 또는 Google과 같은 서비스를 통해 신원을 증명해야 합니다. 이는 사용자의 데이터와 애플리케이션을 무단 액세스 또는 사기 활동으로부터 보호합니다.

## 인증 전략

현대 웹 애플리케이션은 일반적으로 여러 인증 전략을 사용합니다:

1. **OAuth/OpenID Connect (OIDC)**: 사용자 자격 증명을 공유하지 않고 타사 액세스를 활성화합니다. 소셜 미디어 로그인 및 Single Sign-On (SSO) 솔루션에 이상적입니다. OpenID Connect로 ID 레이어를 추가합니다.
2. **자격 증명 기반 로그인 (이메일 + 비밀번호)**: 사용자가 이메일과 비밀번호로 로그인하는 웹 애플리케이션의 표준 선택입니다. 익숙하고 구현하기 쉽지만 피싱과 같은 위협에 대한 강력한 보안 조치가 필요합니다.
3. **비밀번호 없는/토큰 기반 인증**: 이메일 매직 링크 또는 SMS 일회용 코드를 사용하여 비밀번호 없이 안전하게 액세스합니다. 편리하고 향상된 보안으로 인기가 높으며 비밀번호 피로를 줄이는 데 도움이 됩니다. 제한 사항은 사용자의 이메일 또는 전화 가용성에 대한 의존성입니다.
4. **Passkeys/WebAuthn**: 각 사이트에 고유한 암호화 자격 증명을 사용하여 피싱에 대한 높은 보안을 제공합니다. 안전하지만 이 전략은 새롭고 구현하기 어려울 수 있습니다.

인증 전략을 선택하는 것은 애플리케이션의 특정 요구 사항, 사용자 인터페이스 고려 사항 및 보안 목표와 일치해야 합니다.

## 인증 구현

이 섹션에서는 웹 애플리케이션에 기본 이메일-비밀번호 인증을 추가하는 프로세스를 살펴봅니다. 이 방법은 기본 수준의 보안을 제공하지만, 일반적인 보안 위협에 대한 더 나은 보호를 위해 OAuth 또는 비밀번호 없는 로그인과 같은 더 고급 옵션을 고려하는 것이 좋습니다. 우리가 논의할 인증 흐름은 다음과 같습니다:

1. 사용자가 로그인 폼을 통해 자격 증명을 제출합니다.
2. 폼은 Server Action에 의해 처리되는 요청을 보냅니다.
3. 성공적으로 검증되면 프로세스가 완료되어 사용자의 성공적인 인증을 나타냅니다.
4. 검증이 실패하면 오류 메시지가 표시됩니다.

사용자가 자격 증명을 입력할 수 있는 로그인 폼을 고려해 보세요:

```tsx
import { authenticate } from '@/app/lib/actions'

export default async function Page() {
  return (
    <form action={authenticate}>
      <input type="email" name="email" placeholder="이메일" required />
      <input type="password" name="password" placeholder="비밀번호" required />
      <button type="submit">로그인</button>
    </form>
  )
}
```

위의 폼에는 사용자의 이메일과 비밀번호를 캡처하는 두 개의 입력 필드가 있습니다. 제출 시 `authenticate` Server Action을 호출합니다.

그런 다음 Server Action에서 인증 제공자의 API를 호출하여 인증을 처리할 수 있습니다:

```tsx
'use server'

import { signIn } from '@/auth'

export async function authenticate(_currentState: unknown, formData: FormData) {
  try {
    await signIn('credentials', formData)
  } catch (error) {
    if (error) {
      switch (error.type) {
        case 'CredentialsSignin':
          return '잘못된 자격 증명입니다.'
        default:
          return '문제가 발생했습니다.'
      }
    }
    throw error
  }
}
```

이 코드에서 `signIn` 메서드는 저장된 사용자 데이터와 비교하여 자격 증명을 확인합니다. 인증 제공자가 자격 증명을 처리한 후 두 가지 가능한 결과가 있습니다:

- **성공적인 인증**: 이 결과는 로그인이 성공했음을 나타냅니다. 그런 다음 보호된 경로 액세스 및 사용자 정보 가져오기와 같은 추가 작업을 시작할 수 있습니다.
- **인증 실패**: 자격 증명이 잘못되었거나 오류가 발생한 경우 함수는 실패를 나타내는 오류 메시지를 반환합니다.

마지막으로, 로그인 컴포넌트에서 React의 `useActionState`를 사용하여 Server Action을 호출하고 폼 오류를 처리하며, `useFormStatus`를 사용하여 폼의 대기 상태를 처리할 수 있습니다:

```tsx
'use client'

import { authenticate } from '@/app/lib/actions'
import { useActionState } from 'react'
import { useFormStatus } from 'react-dom'

export default function Page() {
  const [errorMessage, formAction, isPending] = useActionState(
    authenticate,
    undefined,
  )

  return (
    <form action={formAction}>
      <input type="email" name="email" placeholder="이메일" required />
      <input type="password" name="password" placeholder="비밀번호" required />
      <LoginButton />
      {errorMessage && <p>{errorMessage}</p>}
    </form>
  )
}

function LoginButton() {
  const { pending } = useFormStatus()

  return (
    <button aria-disabled={pending} type="submit">
      로그인
    </button>
  )
}
```

더 간소화된 인증 설정을 위해 [인증 라이브러리](#인증-라이브러리) 사용을 고려하세요. 이는 세션 관리 및 권한 부여를 위한 내장 솔루션과 함께 소셜 로그인, 비밀번호 없는 인증, 다중 요소 인증(MFA)과 같은 추가 기능을 제공합니다.

## 권한 부여

사용자가 인증되면 사용자가 특정 경로를 방문하고, Server Actions로 데이터를 변경하고, Route Handlers를 호출할 수 있는지 확인해야 합니다.

### Middleware로 경로 보호

Next.js의 [Middleware](/docs/app/building-your-application/routing/middleware)는 웹사이트의 다른 부분에 누가 액세스할 수 있는지 제어하는 데 도움이 됩니다. 이는 대시보드와 같은 영역을 인증된 사용자만 사용할 수 있도록 유지하면서 마케팅 페이지와 같은 다른 페이지는 공개로 유지하는 데 중요합니다.

모든 경로에 Middleware를 적용하고 액세스를 위한 제외를 지정하는 것이 좋습니다.

다음은 Next.js에서 인증을 위한 Middleware를 구현하는 방법입니다:

#### Middleware 설정:

- 프로젝트의 루트 디렉토리에 `middleware.ts` 또는 `middleware.js` 파일을 생성합니다.
- 사용자 액세스를 권한 부여하는 로직(예: 인증 토큰 확인)을 포함합니다.

#### 보호된 경로 정의:

- 모든 경로가 권한 부여를 필요로 하는 것은 아닙니다. Middleware의 `matcher` 옵션을 사용하여 권한 부여 확인이 필요하지 않은 경로를 지정합니다.

#### Middleware 로직:

- 사용자가 인증되었는지 확인하는 로직을 작성합니다. 경로 권한 부여를 위해 사용자 역할 또는 권한을 확인합니다.

#### 무단 액세스 처리:

- 무단 사용자를 로그인 또는 오류 페이지로 적절히 리디렉션합니다.

Middleware 파일 예시:

```tsx
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const currentUser = request.cookies.get('currentUser')?.value

  if (currentUser && !request.nextUrl.pathname.startsWith('/dashboard')) {
    return Response.redirect(new URL('/dashboard', request.url))
  }

  if (!currentUser && !request.nextUrl.pathname.startsWith('/login')) {
    return Response.redirect(new URL('/login', request.url))
  }
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
}
```

이 예시는 간단한 리디렉션을 위해 `Response.redirect`를 사용하지만, 조건부 리디렉션에는 `NextResponse.redirect`를 사용할 수도 있습니다. [자세한 정보](/docs/app/building-your-application/routing/middleware#nextresponse)를 참조하세요.

#### Middleware의 한계:

Middleware는 초기 인증 확인에 유용할 수 있지만 보호된 리소스를 보호하기 위한 유일한 방어선이 되어서는 안 됩니다. 보안 확인의 대부분은 데이터 액세스 레이어(DAL)에서 데이터 소스에 가능한 한 가까이 발생해야 합니다. 이 접근 방식은 특정 데이터에 액세스하거나 변경하기 전에 권한 부여를 확인합니다. [Data Access Layer로 리소스 보호](#data-access-layer-dal로-리소스-보호)를 참조하세요.

**Middleware 팁:**

- Middleware는 쿠키가 필요한 경우 `Intercepting Routes` 및 `Parallel Routes`와 같은 복잡한 라우팅 시나리오에서 작동하지 않을 수 있습니다. 이러한 경우 레이아웃 또는 페이지에서 직접 권한 부여 로직을 사용하세요.

### Server Component에서 권한 부여 확인

[Server Components](/docs/app/building-your-application/rendering/server-components)는 서버 측 작업을 위해 설계되어 있어 서버 리소스에 직접 액세스가 필요한 로직을 통합하는 안전한 환경을 제공합니다.

Server Components에서 일반적인 패턴은 조건부 UI에 대한 권한 부여 로직을 사용하는 것입니다. 예를 들어, 사용자의 역할에 따라 조건부로 기능을 렌더링합니다:

```tsx
import { verifySession } from '@/app/lib/dal'

export default async function Dashboard() {
  const session = await verifySession()
  const userRole = session?.user?.role // 'role'이 세션 객체의 일부라고 가정

  if (userRole === 'admin') {
    return <AdminDashboard /> // 관리자 사용자를 위한 컴포넌트
  } else if (userRole === 'user') {
    return <UserDashboard /> // 일반 사용자를 위한 컴포넌트
  } else {
    return <AccessDenied /> // 역할이 인식되지 않을 때 표시되는 컴포넌트
  }
}
```

이 예시에서는 Data Access Layer의 `verifySession()` 함수를 사용하여 'admin', 'user' 및 권한 부여되지 않은 역할을 확인합니다. 이 패턴은 각 사용자가 자신의 역할에 적합한 컴포넌트와만 상호 작용하도록 보장합니다.

#### 레이아웃 및 인증 확인

[부분 렌더링](/docs/app/building-your-application/routing/linking-and-navigating#4-partial-rendering) 때문에 레이아웃에서 확인을 수행할 때 주의하세요. 레이아웃은 탐색 시 다시 렌더링되지 않으므로 사용자 세션이 모든 경로 변경 시 확인되지 않습니다.

대신 데이터 소스에 가까운 곳이나 조건부로 렌더링될 컴포넌트에서 확인을 수행해야 합니다.

예를 들어, 사용자 데이터를 가져오고 사용자의 이미지를 내비게이션에 표시하는 공유 레이아웃을 고려해 보세요. 레이아웃에서 권한 부여 확인을 수행하는 대신 레이아웃에서 사용자 데이터(`getUser()`)를 가져오고 DAL에서 권한 부여 로직을 수행해야 합니다.

이렇게 하면 애플리케이션 내에서 `getUser()`가 호출되는 모든 곳에서 권한 부여 확인이 수행되고 개발자가 사용자 데이터에 액세스할 수 있는 권한을 확인하는 것을 잊어버리는 것을 방지합니다.

```tsx
export default async function Layout({
  children,
}: {
  children: React.ReactNode
}) {
  const user = await getUser()

  return (
    // ...
  )
}
```

```tsx
export const getUser = cache(async () => {
  const session = await verifySession()
  if (!session) return null

  // 데이터베이스에서 사용자 가져오기
})
```

**알아두면 좋은 점:**

- 레이아웃에서 일반적인 패턴은 조건부 UI에 대해 `user?.id`를 확인하는 것이고, DAL은 데이터에 액세스하기 위해 확인하는 것입니다.
- 이 패턴은 잘 작동하지만 개발자가 레이아웃 확인을 우회하고 컴포넌트에서 직접 데이터를 가져올 수 있습니다. 따라서 DAL에서 권한 부여 확인을 수행하는 것이 중요합니다.

### Server Actions에서 권한 부여

[Server Actions](/docs/app/building-your-application/data-fetching/server-actions-and-mutations)를 공개 대면 엔드포인트와 동일한 보안 고려 사항으로 처리하고 사용자가 작업을 수행할 권한이 있는지 확인하세요.

아래 예시에서는 작업을 진행하기 전에 사용자의 역할을 확인합니다:

```tsx
'use server'
import { verifySession } from '@/app/lib/dal'

export async function serverAction(formData: FormData) {
  const session = await verifySession()
  const userRole = session?.user?.role

  // 사용자가 작업을 수행할 권한이 없는 경우 조기 반환
  if (userRole !== 'admin') {
    return null
  }

  // 권한이 부여된 사용자를 위해 작업 진행
}
```

### Route Handlers에서 권한 부여

Next.js에서 [Route Handlers](/docs/app/building-your-application/routing/route-handlers)는 들어오는 요청을 관리하는 데 중요한 역할을 합니다. Server Actions와 마찬가지로 사용자가 특정 기능에 액세스할 권한이 있는지 확인하기 위해 권한 부여되어야 합니다.

다음은 예시입니다:

```tsx
export async function GET() {
  // 사용자 인증 및 역할 확인
  const session = await verifySession()

  // 사용자가 인증되었는지 확인
  if (!session) {
    // 사용자가 인증되지 않음
    return new Response(null, { status: 401 })
  }

  // 사용자가 'admin' 역할을 가지고 있는지 확인
  if (session.user.role !== 'admin') {
    // 사용자가 인증되었지만 올바른 권한이 없음
    return new Response(null, { status: 403 })
  }

  // 권한이 부여된 사용자를 위한 데이터 가져오기
}
```

이 예시는 두 단계의 보안 확인이 있는 Route Handler를 보여줍니다. 먼저 활성 세션을 확인한 다음 로그인한 사용자가 'admin'인지 확인합니다.

### Context Providers를 사용한 권한 부여

[Context Providers](https://react.dev/reference/react/useContext)를 사용하여 권한 부여로 인해 [interleaving](/docs/app/building-your-application/rendering/composition-patterns#interleaving-server-and-client-components)이 발생하므로 Server Components에서 지원되지 않습니다.

그러나 클라이언트 컴포넌트에서는 작동합니다:

```tsx
'use client'

import { useUser } from '@/app/lib/hooks'

export default function Page() {
  const user = useUser()

  if (user.role !== 'admin') {
    return <p>권한이 없습니다</p>
  }

  return <></>
}
```

일부 개발자는 권한 부여를 위해 Server Components에서 Context Providers를 사용하지만 이는 작동하지 않습니다. 서버 측 렌더링 중에 Context를 사용하려고 하면 오류가 발생할 수 있습니다. Context는 주로 테마 또는 전역 상태와 같은 클라이언트 측 관심사를 위해 설계되었습니다.

Server Components에서 권한 부여를 처리하는 더 나은 방법:

- Data Access Layer.
- Data Transfer Object (DTO).

> **알아두면 좋은 점:** Next.js 문서에서는 프로젝트 아키텍처를 개선하기 위한 패턴으로 DTO를 사용합니다.

## Data Access Layer (DAL)로 리소스 보호

DAL은 데이터와 상호 작용하는 일관되고 중앙 집중화된 방법을 제공하여 애플리케이션의 보안을 강화할 수 있습니다. DAL은 특정 보안 규칙을 시행하고 권한 부여 로직을 단일 위치에 포함할 수 있습니다.

이는 Server Components 및 Server Actions에서 데이터에 액세스하는 모든 요청이 DAL을 거쳐야 하고 중앙 집중화된 보안 확인을 받도록 보장하는 데 도움이 됩니다. 이렇게 하면 데이터 액세스를 안전하고 일관되게 유지하여 보안 누수를 추적하고 모니터링하기가 더 쉬워집니다.

예를 들어, DAL은 다음과 같은 작업을 수행할 수 있습니다:

- 사용자가 특정 데이터에 액세스할 권한이 있는지 확인
- 데이터베이스 상호 작용을 위한 중앙 집중화된 위치 제공
- 보안 관련 로직 및 규칙을 단일 위치에 유지

다음은 DAL의 예시이며, `verifySession()` 함수를 사용하여 사용자를 확인하고 다른 경우에는 오류를 발생시킵니다:

```tsx
import 'server-only'

import { cookies } from 'next/headers'
import { decrypt } from '@/app/lib/session'

export const verifySession = cache(async () => {
  const cookie = (await cookies()).get('session')?.value
  const session = await decrypt(cookie)

  if (!session?.userId) {
    redirect('/login')
  }

  return { isAuth: true, userId: session.userId }
})

export const getUser = cache(async () => {
  const session = await verifySession()
  if (!session) return null

  try {
    const data = await db.query.users.findMany({
      where: eq(users.id, session.userId),
    })

    const user = data[0]

    return user
  } catch (error) {
    console.log('사용자를 가져오는 데 실패했습니다')
    return null
  }
})
```

**알아두면 좋은 점:**

- DAL의 일반적인 패턴은 사용자 객체와 함께 `verifySession()` 함수를 사용하는 것이므로 데이터베이스에 대한 추가 요청을 하지 않아도 됩니다. 그러나 세션 확인과 데이터 가져오기 간의 관심사를 명확하게 분리하기 위해 별도의 함수로 유지하는 것을 선호할 수 있습니다.
- `server-only` 패키지를 사용하여 서버 데이터 가져오기 함수가 클라이언트에서 사용되지 않도록 합니다.
- 권한 부여 로직이 여러 위치에 분산되어 있는 경우 DAL에서 중앙 집중화된 권한 부여 확인 또는 미들웨어를 사용하는 것을 고려하세요.

### DTO (Data Transfer Objects)를 사용하여 반환 값 제한

데이터를 가져올 때 애플리케이션에서 사용될 필요한 데이터만 반환하고 전체 객체를 반환하지 않는 것이 좋습니다. 예를 들어, 사용자 데이터를 가져오는 경우 비밀번호, 전화번호 등이 포함될 수 있는 전체 사용자 객체 대신 사용자의 ID와 이름만 반환할 수 있습니다.

그러나 반환된 데이터 구조를 제어할 수 없거나 전체 객체가 클라이언트로 전달되는 것을 방지하려는 팀에서 작업하는 경우 클라이언트에 노출하기에 안전한 필드를 지정하는 것과 같은 전략을 사용할 수 있습니다.

```tsx
import 'server-only'
import { getUser } from '@/app/lib/dal'

function canSeeUsername(viewer: User) {
  // ...
}

function canSeePhoneNumber(viewer: User, team: string) {
  // ...
}

export async function getProfileDTO(slug: string) {
  const data = await db.query.users.findMany({
    where: eq(users.slug, slug),
  })
  const user = data[0]

  const currentUser = await getUser(user.id)

  // 또는 특정 속성을 반환
  return {
    username: canSeeUsername(currentUser) ? user.username : null,
    phonenumber: canSeePhoneNumber(currentUser, user.team)
      ? user.phonenumber
      : null,
  }
}
```

DAL에서 데이터 가져오기 및 권한 부여 로직을 중앙 집중화하고 DTO를 사용함으로써 모든 데이터 요청이 안전하고 일관되게 보장되어 애플리케이션이 성장함에 따라 유지 관리, 감사 및 디버깅이 더 쉬워집니다.

**알아두면 좋은 점:**

- DTO를 정의하는 방법은 여러 가지가 있습니다. JavaScript의 `toJSON()`부터 위의 예시와 같은 개별 함수 또는 TypeScript의 유틸리티 타입인 `Pick` 및 `Omit`과 같은 다른 라이브러리까지 다양합니다. 이러한 것들은 패턴이므로 프로젝트에 가장 적합한 것을 찾기 위한 연구를 권장합니다.
- [React Essentials](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns#keeping-server-only-code-out-of-the-client-environment) 페이지에서 서버 전용 코드를 클라이언트 환경에서 제외하는 방법에 대해 자세히 알아보세요.

## 세션 관리

세션 관리는 사용자의 인증된 상태가 요청 간에 유지되도록 보장합니다. 이는 사용자의 로그인 정보를 저장하고 관리하여 각 요청마다 로그인할 필요가 없도록 합니다.

세션 관리에는 **쿠키 기반**과 **데이터베이스 세션**의 두 가지 주요 유형이 있습니다.

### 쿠키 기반 세션 (Stateless)

> **알아두면 좋은 점:** 쿠키 기반 세션을 사용하지만 더 많은 기능이 필요한 경우 [세션 관리 라이브러리](#세션-관리-라이브러리)를 살펴보세요.

쿠키 기반 세션은 암호화된 세션 정보를 브라우저 쿠키에 직접 저장하여 관리합니다. 사용자가 로그인하면 이 암호화된 데이터가 쿠키에 저장됩니다. 각 후속 서버 요청에는 이 쿠키가 포함되어 반복되는 서버 쿼리의 필요성을 최소화하고 클라이언트 측 효율성을 향상시킵니다.

그러나 이 방법에는 쿠키가 클라이언트 측 보안 위험에 취약할 수 있으므로 신중한 암호화가 필요합니다. 쿠키의 세션 데이터를 암호화하는 것은 사용자 정보를 무단 액세스로부터 보호하는 데 중요합니다. 쿠키가 도난당하더라도 그 안의 데이터는 읽을 수 없는 상태로 유지됩니다.

또한 개별 쿠키의 크기는 제한되어 있지만(일반적으로 약 4KB), 쿠키 청킹과 같은 기술을 사용하여 대용량 세션 데이터를 여러 쿠키로 분할하여 이 제한을 극복할 수 있습니다.

Next.js 프로젝트에서 쿠키를 설정하는 예시:

**1. 세션 생성:**

로그인 시 암호화된 세션을 생성하고 쿠키에 저장합니다.

```tsx
'use server'

import { cookies } from 'next/headers'
import { encrypt } from '@/app/lib/session'

export async function createSession(userId: string) {
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  const session = await encrypt({ userId, expiresAt })

  (await cookies()).set('session', session, {
    httpOnly: true,
    secure: true,
    expires: expiresAt,
    sameSite: 'lax',
    path: '/',
  })
}
```

**2. 세션 업데이트 (갱신):**

```tsx
'use server'

import { cookies } from 'next/headers'
import { decrypt, encrypt } from '@/app/lib/session'

export async function updateSession() {
  const session = (await cookies()).get('session')?.value
  const payload = await decrypt(session)

  if (!session || !payload) {
    return null
  }

  const expires = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)

  const cookieStore = await cookies()
  cookieStore.set('session', session, {
    httpOnly: true,
    secure: true,
    expires: expires,
    sameSite: 'lax',
    path: '/',
  })
}
```

**팁:**

- 인증 라이브러리가 세션 갱신을 지원하는지 확인하세요.
- 세션이 인증 확인 시 갱신되도록 Middleware에서 `updateSession()` 함수를 호출합니다.

**3. 세션 삭제:**

Middleware에서 세션을 삭제합니다.

```tsx
'use server'

import { cookies } from 'next/headers'

export async function deleteSession() {
  const cookieStore = await cookies()
  cookieStore.delete('session')
}
```

### 데이터베이스 세션

데이터베이스 세션 관리는 세션 데이터를 서버에 저장하고 사용자의 브라우저는 세션 ID만 받습니다. 이 ID는 민감한 세션 데이터 자체를 포함하지 않고 서버 측에 저장된 세션 데이터를 참조합니다. 이 방법은 보안을 강화하여 세션 데이터를 클라이언트 측 환경에 노출할 위험을 줄입니다. 데이터베이스 세션은 또한 더 확장 가능하여 더 많은 데이터를 저장할 수 있습니다.

그러나 이 접근 방식에는 절충안이 있습니다. 각 사용자 상호 작용에서 데이터베이스 조회가 필요하기 때문에 성능 오버헤드가 증가할 수 있습니다. 세션 데이터 캐싱과 같은 전략은 이를 완화하는 데 도움이 될 수 있습니다. 또한 데이터베이스에 대한 의존도는 세션 관리가 데이터베이스 성능과 가용성만큼만 안정적임을 의미합니다.

다음은 Next.js 애플리케이션에서 데이터베이스 세션을 구현하는 간단한 예시입니다:

**1. 세션 생성:**

```tsx
import db from '@/app/lib/db'
import { cookies } from 'next/headers'
import { encrypt } from '@/app/lib/session'

export async function createSession(id: number) {
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)

  // 1. 데이터베이스에 세션 생성
  const data = await db
    .insert(sessions)
    .values({
      userId: id,
      expiresAt,
    })
    .returning({ id: sessions.id })

  const sessionId = data[0].id

  // 2. 세션 ID 암호화
  const session = await encrypt({ sessionId, expiresAt })

  // 3. 쿠키에 세션 저장
  (await cookies()).set('session', session, {
    httpOnly: true,
    secure: true,
    expires: expiresAt,
    sameSite: 'lax',
    path: '/',
  })
}
```

**팁:**

- 더 빠른 데이터 검색을 위해 [Vercel Redis](https://vercel.com/docs/storage/vercel-kv)와 같은 데이터베이스를 사용하는 것을 고려하세요. 그러나 세션 데이터를 기본 데이터베이스에 보관하고 데이터 요청을 결합하여 쿼리 수를 줄일 수도 있습니다.

**2. 세션 업데이트 (갱신):**

```tsx
import { cookies } from 'next/headers'
import { decrypt, encrypt } from '@/app/lib/session'
import db from '@/app/lib/db'

export async function updateSession() {
  const session = (await cookies()).get('session')?.value
  const payload = await decrypt(session)

  if (!session || !payload) {
    return null
  }

  const expires = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)

  await db
    .update(sessions)
    .set({ expiresAt: expires })
    .where(eq(sessions.id, payload.sessionId))
}
```

**팁:**

- 인증 라이브러리가 세션 갱신을 지원하는지 확인하세요.

**3. 세션 삭제:**

```tsx
import { cookies } from 'next/headers'
import { decrypt } from '@/app/lib/session'
import db from '@/app/lib/db'

export async function deleteSession() {
  const cookieStore = await cookies()
  const session = cookieStore.get('session')?.value
  const payload = await decrypt(session)

  if (!payload?.sessionId) {
    return
  }

  await db.delete(sessions).where(eq(sessions.id, payload.sessionId))
  cookieStore.delete('session')
}
```

**팁:**

- Middleware 또는 클라이언트 측 함수에서 `deleteSession()`을 호출하여 사용자를 로그아웃시킵니다.

### `iron-session` 또는 `jose`로 세션 암호화

Next.js 프로젝트에서 세션을 암호화하기 위해 `iron-session` 또는 `jose`와 같은 라이브러리를 사용할 수 있습니다.

**`iron-session`의 경우:**

```tsx
import 'server-only'
import { cookies } from 'next/headers'
import { sealData, unsealData } from 'iron-session'

const key = process.env.SESSION_SECRET

export async function encrypt(data: any) {
  return sealData(data, { password: key })
}

export async function decrypt(session: string) {
  return unsealData(session, { password: key })
}
```

**`jose`의 경우:**

```tsx
import { SignJWT, jwtVerify } from 'jose'

const secretKey = process.env.SESSION_SECRET
const encodedKey = new TextEncoder().encode(secretKey)

export async function encrypt(payload: any) {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(encodedKey)
}

export async function decrypt(session: string | undefined = '') {
  try {
    const { payload } = await jwtVerify(session, encodedKey, {
      algorithms: ['HS256'],
    })
    return payload
  } catch (error) {
    console.log('세션 검증 실패')
    return null
  }
}
```

### 세션 관리 라이브러리

Next.js와 호환되는 세션 관리 라이브러리는 다음과 같습니다:

- [Iron Session](https://github.com/vvo/iron-session)
- [Lucia](https://lucia-auth.com/sessions/overview/)

## 예시

다음은 Next.js와 호환되는 인증 솔루션입니다. 아래 퀵스타트 가이드를 참조하여 Next.js 애플리케이션에서 이를 구성하는 방법을 알아보세요:

### NextAuth.js

[NextAuth.js](https://authjs.dev/getting-started/installation?framework=next.js)는 Next.js 애플리케이션을 위한 완전한 인증 솔루션입니다. 다양한 인증 패턴과 옵션을 지원하여 모든 사용 사례를 처리할 수 있습니다. Next.js 애플리케이션을 위한 자세한 설명서를 제공하며 시작하기 쉽습니다.

### Clerk

[Clerk](https://clerk.com/)는 임베드 가능한 UI, 유연한 API 및 관리 대시보드를 갖춘 Next.js를 위한 완전한 인증 및 사용자 관리 솔루션입니다.

### Auth0

[Auth0](https://auth0.com/docs/quickstart/webapp/nextjs/01-login)는 Next.js 애플리케이션에 인증 및 권한 부여를 빠르게 추가할 수 있는 유연한 플랫폼입니다.

### Stytch

[Stytch](https://stytch.com/)는 비밀번호 없는 인증, 세션 관리 및 사용자 관리를 위한 Next.js와 호환되는 플랫폼입니다.

### Kinde

[Kinde](https://kinde.com/)는 Next.js 애플리케이션을 위한 인증 및 권한 부여 플랫폼으로 사용자 관리, 다중 요소 인증 등을 지원합니다.

### WorkOS

[WorkOS](https://workos.com/)는 Next.js 애플리케이션에 SSO, SCIM 프로비저닝, RBAC 등의 엔터프라이즈 기능을 추가할 수 있는 플랫폼입니다.

### Stack Auth

[Stack Auth](https://stack-auth.com/)는 Next.js 애플리케이션을 위한 완전한 인증 시스템으로 다중 테넌시, M2M 인증 등을 지원합니다.

## 추가 자료

인증 및 보안에 대해 계속 학습하려면 다음 리소스를 확인하세요:

- [XSS 공격 이해](https://vercel.com/guides/understanding-xss-attacks)
- [CSRF 공격 이해](https://vercel.com/guides/understanding-csrf-attacks)
