# unstable_cache

## 개요
`unstable_cache`는 Next.js에서 비용이 많이 드는 작업(예: 데이터베이스 쿼리)을 여러 요청에 걸쳐 캐시할 수 있게 합니다. 내장된 Data Cache를 사용하여 요청 및 배포 전반에 걸쳐 결과를 유지합니다.

> **경고:** 이 API는 안정화되면 [`use cache`](/docs/app/api-reference/directives/use-cache.md)로 대체될 예정입니다.

---

## 기본 사용법
```jsx
import { unstable_cache } from 'next/cache';
import { getUser } from './data';

const getCachedUser = unstable_cache(
  async (id) => getUser(id),
  ['my-app-user']
);

export default async function Component({ userID }) {
  const user = await getCachedUser(userID);
}
```

---

## 함수 시그니처
```jsx
const data = unstable_cache(fetchData, keyParts, options)()
```

---

## 매개변수

### 1. **fetchData** (필수)
- 캐시할 데이터를 페칭하는 비동기 함수
- `Promise`를 반환해야 함

### 2. **keyParts** (선택사항)
- 캐시 식별을 위한 추가 키 배열
- 기본적으로 함수 인수와 문자열화된 함수를 캐시 키로 사용
- **중요:** 매개변수로 전달되지 않은 클로저/외부 변수를 함수에서 사용하는 경우 포함해야 함

### 3. **options** (선택사항)
캐시 동작을 제어하는 객체:

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `tags` | `string[]` | `revalidateTag()`를 통한 캐시 무효화를 위한 태그 |
| `revalidate` | `number \| false` | 캐시 재검증까지의 초. 무기한 캐싱은 생략하거나 `false` 사용 |

---

## 중요한 제약 사항

⚠️ **동적 데이터 제한:**
- 캐시된 함수 내부에서 동적 데이터 소스(`headers`, `cookies`)에 접근할 수 없음
- **해결 방법:** 캐시된 함수 외부에서 동적 데이터에 접근하고 인수로 전달

---

## 완전한 예제

```tsx
// app/page.tsx
import { unstable_cache } from 'next/cache'

export default async function Page({
  params,
}: {
  params: Promise<{ userId: string }>
}) {
  const { userId } = await params

  const getCachedUser = unstable_cache(
    async () => {
      return { id: userId }
    },
    [userId], // 캐시 키에 사용자 ID 추가
    {
      tags: ['users'],
      revalidate: 60, // 60초마다 재검증
    }
  )

  const user = await getCachedUser()
  return <div>{user.id}</div>
}
```

---

## 실전 예제

### 데이터베이스 쿼리 캐싱

```tsx
// lib/data.ts
import { unstable_cache } from 'next/cache'
import { db } from './db'

export const getUser = unstable_cache(
  async (userId: string) => {
    return await db.user.findUnique({
      where: { id: userId }
    })
  },
  ['user'], // 캐시 키
  {
    tags: ['users'], // 재검증 태그
    revalidate: 3600, // 1시간마다 재검증
  }
)
```

```tsx
// app/users/[id]/page.tsx
import { getUser } from '@/lib/data'

export default async function UserPage({ params }) {
  const { id } = await params
  const user = await getUser(id)

  return <div>{user.name}</div>
}
```

### 외부 API 캐싱

```ts
// lib/api.ts
import { unstable_cache } from 'next/cache'

export const getGitHubUser = unstable_cache(
  async (username: string) => {
    const response = await fetch(`https://api.github.com/users/${username}`)
    return response.json()
  },
  ['github-user'],
  {
    revalidate: 3600, // 1시간
    tags: ['github'],
  }
)
```

### 복잡한 계산 캐싱

```ts
// lib/analytics.ts
import { unstable_cache } from 'next/cache'

export const calculateStats = unstable_cache(
  async (startDate: string, endDate: string) => {
    // 복잡한 통계 계산
    const data = await db.event.findMany({
      where: {
        createdAt: {
          gte: new Date(startDate),
          lte: new Date(endDate),
        }
      }
    })

    return {
      total: data.length,
      average: data.reduce((sum, d) => sum + d.value, 0) / data.length,
      // ... 더 많은 계산
    }
  },
  ['stats'], // 기본 키
  {
    revalidate: 300, // 5분마다 재검증
    tags: ['analytics'],
  }
)
```

```tsx
// app/analytics/page.tsx
export default async function AnalyticsPage({ searchParams }) {
  const stats = await calculateStats(
    searchParams.start || '2024-01-01',
    searchParams.end || '2024-12-31'
  )

  return <StatsDisplay data={stats} />
}
```

### 동적 데이터와 함께 사용

```tsx
// app/dashboard/page.tsx
import { cookies } from 'next/headers'
import { unstable_cache } from 'next/cache'

async function getDashboardData(userId: string) {
  return unstable_cache(
    async () => {
      return await db.dashboard.findFirst({
        where: { userId }
      })
    },
    ['dashboard', userId], // userId를 키에 포함
    {
      revalidate: 60,
      tags: ['dashboard', `user-${userId}`],
    }
  )()
}

export default async function DashboardPage() {
  // 동적 데이터를 외부에서 가져옴
  const cookieStore = await cookies()
  const userId = cookieStore.get('userId')?.value

  // 캐시된 함수에 전달
  const data = await getDashboardData(userId)

  return <Dashboard data={data} />
}
```

### 태그로 재검증

```ts
// app/actions.ts
'use server'
import { revalidateTag } from 'next/cache'

export async function updateUser(userId: string) {
  // 사용자 업데이트
  await db.user.update({ where: { id: userId }, data: {...} })

  // 'users' 태그가 있는 모든 캐시 무효화
  revalidateTag('users')
}
```

---

## 모범 사례

### 1. 적절한 캐시 키 사용

```ts
// ❌ 나쁜 예 - 외부 변수를 키에 포함하지 않음
const orgId = 'org-123'
const getData = unstable_cache(
  async () => db.query({ orgId }), // orgId가 클로저
  ['data'] // 키에 orgId가 없음
)

// ✅ 좋은 예 - 외부 변수를 키에 포함
const orgId = 'org-123'
const getData = unstable_cache(
  async () => db.query({ orgId }),
  ['data', orgId] // 키에 orgId 포함
)

// ✅ 더 좋은 예 - 매개변수로 전달
const getData = unstable_cache(
  async (orgId) => db.query({ orgId }),
  ['data'] // orgId가 자동으로 키에 포함됨
)
```

### 2. 적절한 재검증 간격

```ts
// 자주 변경되는 데이터
const getLiveData = unstable_cache(
  async () => fetchLiveData(),
  ['live'],
  { revalidate: 10 } // 10초
)

// 가끔 변경되는 데이터
const getSettings = unstable_cache(
  async () => fetchSettings(),
  ['settings'],
  { revalidate: 3600 } // 1시간
)

// 거의 변경되지 않는 데이터
const getStaticContent = unstable_cache(
  async () => fetchStaticContent(),
  ['static'],
  { revalidate: false } // 무기한
)
```

### 3. 의미 있는 태그 사용

```ts
const getUserPosts = unstable_cache(
  async (userId: string) => {
    return await db.post.findMany({ where: { authorId: userId } })
  },
  ['user-posts'],
  {
    tags: [
      'posts',           // 모든 포스트
      `user-${userId}`,  // 특정 사용자의 데이터
    ],
  }
)

// 선택적 재검증
revalidateTag('posts')         // 모든 포스트 재검증
revalidateTag('user-123')      // 사용자 123의 데이터만 재검증
```

### 4. 동적 데이터 처리

```ts
// ❌ 나쁜 예 - 캐시 내부에서 동적 데이터 접근
const getData = unstable_cache(async () => {
  const cookieStore = await cookies() // ❌ 오류!
  return fetchData(cookieStore.get('id'))
})

// ✅ 좋은 예 - 외부에서 동적 데이터 가져오기
async function getData(userId: string) {
  return unstable_cache(
    async () => fetchData(userId),
    ['data', userId]
  )()
}

// 사용
const cookieStore = await cookies()
const userId = cookieStore.get('userId')?.value
const data = await getData(userId)
```

---

## 주의사항

### 불안정한 API
- 이 API는 `unstable_`로 시작하므로 변경될 수 있습니다
- 프로덕션에서 사용할 수 있지만 API가 변경될 수 있음을 인지하세요
- 향후 `use cache` 지시어로 대체될 예정

### 캐시 무효화
```ts
import { revalidateTag } from 'next/cache'

// 태그로 재검증
revalidateTag('users')

// 또는 경로로 재검증
revalidatePath('/users')
```

### 타입 안정성
```ts
import { unstable_cache } from 'next/cache'

type User = {
  id: string
  name: string
}

const getUser = unstable_cache(
  async (id: string): Promise<User> => {
    return await db.user.findUnique({ where: { id } })
  },
  ['user'],
  { revalidate: 3600 }
)
```

---

## 반환 값
- 호출 시 `Promise`를 반환하는 함수를 반환
- 캐시가 있으면 캐시된 데이터로 해석
- 그렇지 않으면 제공된 함수를 호출하고 결과를 캐시한 다음 반환

---

## 버전 히스토리
- **v14.0.0** - `unstable_cache` 도입

---

## 관련 문서

- [revalidateTag](./revalidateTag.md)
- [revalidatePath](./revalidatePath.md)
- [fetch](./fetch.md)
- [캐싱 가이드](../../guides/caching.md)
