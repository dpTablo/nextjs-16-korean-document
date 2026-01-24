# taint

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

`taint` 옵션은 React의 실험적 API를 지원하여 민감한 데이터가 클라이언트로 실수로 전달되는 것을 방지합니다.

### 사용 가능한 API

- `experimental_taintObjectReference`: 객체 참조를 taint 처리
- `experimental_taintUniqueValue`: 고유값을 taint 처리

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    taint: true,
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    taint: true,
  },
}

module.exports = nextConfig
```

> **참고**: `taint`를 활성화하면 React `experimental` 채널이 `app` 디렉토리에서 활성화됩니다.

## 주요 특징

- **방어적 접근**: 서버-클라이언트 경계를 넘지 못하도록 데이터를 명시적으로 표시
- **자동 오류**: taint된 객체/값을 서버-클라이언트 경계로 전달하면 React가 오류 발생
- **사용 사례**:
  - 데이터 읽기 메서드가 통제 불가능할 때
  - 정의되지 않은 민감한 데이터 형식 처리
  - 서버 컴포넌트 렌더링 중 민감한 데이터 접근

## 사용 예제

### 객체 참조 Tainting

```ts filename="app/lib/data.ts"
import { experimental_taintObjectReference } from 'react'

export async function getUserDetails(id: string) {
  const user = await db.queryUserById(id)

  experimental_taintObjectReference(
    '전체 사용자 정보 객체를 사용하지 마세요. 필요한 필드만 선택하세요.',
    user
  )

  return user
}
```

```tsx filename="app/contact/[id]/page.tsx"
// 올바른 사용 - 필요한 필드만 선택
export default async function ContactPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const userDetails = await getUserDetails(id)

  return (
    <UserCard
      firstName={userDetails.firstName}
      lastName={userDetails.lastName}
    />
  )
}
```

```tsx
// 오류 발생 - 전체 객체 전달
export default async function ContactPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const userDetails = await getUserDetails(id)

  // ❌ 오류 발생: taint된 객체 전달
  return <UserCard user={userDetails} />
}
```

### 고유값 Tainting

```ts filename="app/lib/config.ts"
import { experimental_taintUniqueValue } from 'react'

export async function getSystemConfig() {
  const config = await config.getConfigDetails()

  experimental_taintUniqueValue(
    '구성 토큰을 클라이언트에 전달하지 마세요',
    config,
    config.SERVICE_API_KEY
  )

  return config
}
```

```tsx filename="app/dashboard/page.tsx"
// 올바른 사용 - 다른 필드 접근
export default async function Dashboard() {
  const systemConfig = await getSystemConfig()

  return <ClientDashboard version={systemConfig.SERVICE_API_VERSION} />
}
```

```tsx
// 오류 발생 - taint된 값 직접 전달
export default async function Dashboard() {
  const systemConfig = await getSystemConfig()
  const version = systemConfig.SERVICE_API_KEY

  // ❌ 오류 발생: taint된 값 재할당해도 추적됨
  return <ClientDashboard version={version} />
}
```

## 주의사항

1. **참조 기반**: 객체의 참조로만 추적 가능. 객체 복사 시 taint 해제됨
2. **파생 데이터**: taint된 값에서 파생된 데이터도 별도로 taint 필요
3. **참조 범위**: 참조가 스코프 내에 있을 때만 taint 유지

```tsx
// ⚠️ 주의: 파생 데이터는 taint 해제됨
const version = `version::${systemConfig.SERVICE_API_KEY}`
// 이 파생 문자열은 더 이상 taint되지 않음
```

## 보안 권장사항

- Taint API를 유일한 보안 메커니즘으로 의존하지 마세요
- 민감한 데이터가 필요 없는 컨텍스트로 반환되지 않도록 데이터 모델링을 검토하세요
