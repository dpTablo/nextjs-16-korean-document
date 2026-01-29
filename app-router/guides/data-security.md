---
원문: https://nextjs.org/docs/app/guides/data-security
버전: 16.1.6
---

# Next.js에서 데이터 보안에 대해 생각하는 방법

## 개요

React Server Components는 성능을 향상시키고 데이터 페칭을 단순화하지만, 데이터에 접근하는 위치와 방법을 변경하여 프론트엔드 앱에서 데이터를 처리하는 기존 보안 가정을 바꿉니다.

---

## 데이터 페칭 접근 방식

Next.js에서 데이터를 페칭하는 세 가지 주요 권장 접근 방식이 있습니다:

### 1. 외부 HTTP API

**Zero Trust** 모델을 따르는 기존 대규모 애플리케이션과 조직에 적합합니다.

```tsx
// app/page.tsx
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = cookies()
  const token = cookieStore.get('AUTH_TOKEN')?.value

  const res = await fetch('https://api.example.com/profile', {
    headers: {
      Cookie: `AUTH_TOKEN=${token}`,
      // 기타 헤더
    },
  })

  // ....
}
```

**적합한 경우:**
- 이미 보안 관행이 갖춰져 있는 경우
- 별도의 백엔드 팀이 다른 언어를 사용하거나 API를 독립적으로 관리하는 경우

### 2. 데이터 접근 계층 (DAL)

새 프로젝트를 위한 전용 내부 라이브러리로, 데이터 페칭과 렌더링 컨텍스트에 전달되는 내용을 제어합니다.

**데이터 접근 계층은 다음을 충족해야 합니다:**
- 서버에서만 실행
- 권한 확인 수행
- 안전하고 최소한의 데이터 전송 객체(DTO) 반환

```ts
// data/auth.ts
import { cache } from 'react'
import { cookies } from 'next/headers'

export const getCurrentUser = cache(async () => {
  const token = cookies().get('AUTH_TOKEN')
  const decodedToken = await decryptAndValidate(token)
  return new User(decodedToken.id)
})
```

```tsx
// data/user-dto.tsx
import 'server-only'
import { getCurrentUser } from './auth'

function canSeeUsername(viewer: User) {
  return true
}

function canSeePhoneNumber(viewer: User, team: string) {
  return viewer.isAdmin || team === viewer.team
}

export async function getProfileDTO(slug: string) {
  const [rows] = await sql`SELECT * FROM user WHERE slug = ${slug}`
  const userData = rows[0]

  const currentUser = await getCurrentUser()

  return {
    username: canSeeUsername(currentUser) ? userData.username : null,
    phonenumber: canSeePhoneNumber(currentUser, userData.team)
      ? userData.phonenumber
      : null,
  }
}
```

```tsx
// app/page.tsx
import { getProfile } from '../../data/user'

export async function Page({ params: { slug } }) {
  const profile = await getProfile(slug);
  // ...
}
```

### 3. 컴포넌트 수준 데이터 접근

빠른 프로토타입과 반복 개발을 위해 - Server Components에 직접 데이터베이스 쿼리를 배치합니다.

**⚠️ 주의:** 실수로 개인 데이터를 클라이언트에 노출시키기 쉽습니다.

```tsx
// 잘못된 예제 - app/page.tsx
import Profile from './components/profile.tsx'

export async function Page({ params: { slug } }) {
  const [rows] = await sql`SELECT * FROM user WHERE slug = ${slug}`
  const userData = rows[0]
  // 노출됨: 모든 필드가 클라이언트에 노출됩니다
  return <Profile user={userData} />
}
```

**✅ 더 나은 해결책:**

```ts
// data/user.ts
import { sql } from './db'

export async function getUser(slug: string) {
  const [rows] = await sql`SELECT * FROM user WHERE slug = ${slug}`
  const user = rows[0]

  return {
    name: user.name,
  }
}
```

```tsx
// app/page.tsx
import { getUser } from '../data/user'
import Profile from './ui/profile'

export default async function Page({
  params: { slug },
}: {
  params: { slug: string }
}) {
  const publicProfile = await getUser(slug)
  return <Profile user={publicProfile} />
}
```

---

## 데이터 읽기

### 서버에서 클라이언트로 데이터 전달

**Server Components:**
- 서버에서만 실행됩니다
- 환경 변수, 시크릿, 데이터베이스, 내부 API에 안전하게 접근할 수 있습니다

**Client Components:**
- 사전 렌더링 중에 서버에서 실행되지만 브라우저 보안 가정을 따릅니다
- 권한이 필요한 데이터나 서버 전용 모듈에 접근하면 안 됩니다

### 테인팅(Tainting)

React Taint API를 사용하여 개인 데이터의 우발적 노출을 방지합니다:

- `experimental_taintObjectReference` - 데이터 객체용
- `experimental_taintUniqueValue` - 특정 값용

`next.config.js`에서 활성화합니다:

```js
module.exports = {
  experimental: {
    taint: true,
  },
}
```

> **참고:** 테인팅은 추가적인 보호 계층입니다. React의 렌더링 컨텍스트에 전달하기 전에 DAL에서 데이터를 필터링하고 정제해야 합니다.

### 서버 전용 코드의 클라이언트 측 실행 방지

`server-only` 패키지를 사용합니다:

```bash
npm install server-only
```

```ts
// lib/data.ts
import 'server-only'

// ...
```

이렇게 하면 클라이언트 환경에서 임포트할 경우 빌드 오류가 발생하여 독점 코드가 서버에 유지됩니다.

---

## 데이터 변경

Next.js는 [Server Actions](https://react.dev/reference/rsc/server-functions)를 사용하여 변경을 처리합니다.

### 내장 Server Actions 보안 기능

**기본 동작:**
- Server Actions는 공개 HTTP 엔드포인트를 생성합니다
- 공개 API와 동일한 보안 가정으로 처리해야 합니다

**보안 기능:**
- **보안 액션 ID:** 컴파일 중에 생성된 암호화된 비결정적 ID (빌드 간에 재계산됨)
- **데드 코드 제거:** 사용되지 않는 Server Actions는 클라이언트 번들에서 제거됩니다

```jsx
// app/actions.js
'use server'

// 사용되는 경우: Next.js가 공개 엔드포인트용 보안 ID를 생성합니다
export async function updateUserAction(formData) {}

// 사용되지 않는 경우: Next.js가 빌드 중에 이를 제거합니다
export async function deleteUserAction(formData) {}
```

> **알아두면 좋은 점:** ID는 최대 14일 동안 캐시되며, 새 빌드나 캐시 무효화 시 재생성됩니다.

### 클라이언트 입력 유효성 검사

클라이언트의 입력은 항상 유효성을 검사합니다 - 폼 데이터, URL 매개변수, 헤더, searchParams는 쉽게 수정될 수 있습니다.

```tsx
// 잘못된 예: searchParams를 직접 신뢰
export default async function Page({ searchParams }) {
  const isAdmin = searchParams.get('isAdmin')
  if (isAdmin === 'true') {
    return <AdminPanel />
  }
}

// 올바른 예: 매번 재검증
import { cookies } from 'next/headers'
import { verifyAdmin } from './auth'

export default async function Page() {
  const token = cookies().get('AUTH_TOKEN')
  const isAdmin = await verifyAdmin(token)

  if (isAdmin) {
    return <AdminPanel />
  }
}
```

### 인증 및 권한 부여

사용자가 작업을 수행할 권한이 있는지 항상 확인합니다:

```tsx
// app/actions.ts
'use server'

import { auth } from './lib'

export function addItem() {
  const { user } = auth()
  if (!user) {
    throw new Error('이 작업을 수행하려면 로그인해야 합니다')
  }

  // ...
}
```

### 클로저와 암호화

컴포넌트 내부에서 정의된 Server Actions는 외부 함수 스코프에 접근할 수 있는 클로저를 생성합니다:

```tsx
export default async function Page() {
  const publishVersion = await getLatestVersion();

  async function publish() {
    "use server";
    if (publishVersion !== await getLatestVersion()) {
      throw new Error('발행 버튼을 누른 이후 버전이 변경되었습니다');
    }
    // ...
  }

  return (
    <form>
      <button formAction={publish}>발행</button>
    </form>
  );
}
```

캡처된 변수는 Next.js에 의해 자동으로 암호화됩니다. 모든 빌드에서 각 액션에 대해 새로운 비공개 키가 생성됩니다.

> **알아두면 좋은 점:** 클라이언트에서 민감한 정보 노출을 방지하기 위해 암호화에만 의존하지 마세요.

### 암호화 키 덮어쓰기 (고급)

셀프 호스팅 다중 서버 배포의 경우 환경 변수를 사용합니다:

```
NEXT_SERVER_ACTIONS_ENCRYPTION_KEY
```

이 변수는 **반드시** AES-GCM으로 암호화되어야 하며 빌드와 서버 간에 지속적인 암호화 키를 보장합니다.

### 허용된 출처 (고급)

Server Actions는 `POST` 메서드를 사용합니다 (SameSite 쿠키로 대부분의 CSRF 취약점을 방지합니다).

추가 보호: Server Actions는 `Origin` 헤더를 `Host` 헤더(또는 `X-Forwarded-Host`)와 비교합니다. 일치하지 않으면 요청이 중단됩니다.

리버스 프록시나 다계층 백엔드를 사용하는 애플리케이션의 경우 `next.config.js`에서 구성합니다:

```js
module.exports = {
  experimental: {
    serverActions: {
      allowedOrigins: ['my-proxy.com', '*.my-proxy.com'],
    },
  },
}
```

### 렌더링 중 부작용 방지

변경은 렌더링 메서드에서 부작용으로 발생해서는 안 됩니다.

```tsx
// 잘못된 예: 렌더링 중 변경 트리거
export default async function Page({ searchParams }) {
  if (searchParams.get('logout')) {
    cookies().delete('AUTH_TOKEN')
  }

  return <UserProfile />
}

// 올바른 예: Server Actions 사용
import { logout } from './actions'

export default function Page() {
  return (
    <>
      <UserProfile />
      <form action={logout}>
        <button type="submit">로그아웃</button>
      </form>
    </>
  )
}
```

---

## 감사 체크리스트

Next.js 프로젝트를 감사할 때:

- **데이터 접근 계층:** 분리된 DAL이 있나요? 데이터베이스 패키지와 환경 변수가 DAL 외부에서 임포트되지 않는지 확인하세요
- **`"use client"` 파일:** 컴포넌트 props가 개인 데이터를 기대하나요? 타입 시그니처가 너무 광범위한가요?
- **`"use server"` 파일:** 인수가 유효성 검사되나요? 사용자가 액션 내부에서 재인가되나요?
- **`/[param]/` 폴더:** 매개변수는 사용자 입력입니다 - 유효성이 검사되나요?
- **`proxy.ts`와 `route.ts`:** 높은 권한을 가진 파일입니다 - 감사에 추가 시간을 할애하세요. 침투 테스트와 취약점 스캐닝을 고려하세요

---

## 다음 단계

- [인증](/app-router/guides/authentication.md)
- [콘텐츠 보안 정책](/app-router/guides/content-security-policy.md)
- [폼](/app-router/guides/forms.md)
