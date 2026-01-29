---
원문: https://nextjs.org/docs/app/api-reference/functions/unstable_noStore
버전: 16.1.6
---

# unstable_noStore

## 개요

`unstable_noStore`는 정적 렌더링을 옵트아웃하고 컴포넌트 캐싱을 방지하는 데 사용되는 레거시 Next.js API입니다.

> **⚠️ Deprecated (사용 중단)**
> **v15.0.0**부터 deprecated되었으며, 대신 [`connection()`](./connection.md)을 사용하는 것이 권장됩니다.

---

## 목적

- 특정 컴포넌트에 대한 정적 렌더링을 선언적으로 비활성화
- 컴포넌트 단위로 캐싱 방지
- `export const dynamic = 'force-dynamic'`보다 더 세밀한 제어 제공

---

## 기본 사용법

```jsx
import { unstable_noStore as noStore } from 'next/cache'

export default async function ServerComponent() {
  noStore()
  const result = await db.query(...)
  // 이 컴포넌트는 캐싱되지 않습니다
}
```

---

## 주요 특징

| 항목 | 세부사항 |
|------|---------|
| **동등한 기능** | fetch 요청에서 `cache: 'no-store'` 사용 |
| **비교 우위** | `export const dynamic = 'force-dynamic'`보다 세밀한 컴포넌트 단위 제어 |
| **`unstable_cache` 내 동작** | 정적 생성을 옵트아웃하지 않으며, 캐시 구성을 따름 |

---

## 상세 설명

### 1. 정적 렌더링 옵트아웃

```tsx
import { unstable_noStore as noStore } from 'next/cache'

export default async function DynamicData() {
  // 이 컴포넌트를 동적으로 만듭니다
  noStore()

  const data = await fetch('https://api.example.com/data')
  return <div>{/* 항상 최신 데이터 표시 */}</div>
}
```

### 2. fetch와의 비교

```tsx
// unstable_noStore 사용
async function Component1() {
  noStore()
  const data = await fetch('https://api.example.com')
  // 컴포넌트 전체가 동적으로 처리됨
}

// fetch의 cache 옵션 사용
async function Component2() {
  const data = await fetch('https://api.example.com', {
    cache: 'no-store' // 이 fetch만 캐싱 안함
  })
}
```

### 3. Route Segment Config와 비교

```tsx
// Route Segment Config (페이지 전체에 적용)
export const dynamic = 'force-dynamic'

export default async function Page() {
  // 이 페이지의 모든 컴포넌트가 동적으로 처리됨
}
```

```tsx
// unstable_noStore (컴포넌트 단위)
export default async function Page() {
  return (
    <>
      <StaticComponent />  {/* 정적으로 렌더링 */}
      <DynamicComponent /> {/* noStore() 사용 - 동적으로 렌더링 */}
    </>
  )
}

async function DynamicComponent() {
  noStore() // 이 컴포넌트만 동적으로 처리
  const data = await db.query(...)
  return <div>{data}</div>
}
```

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|---------|
| **v15.0.0** | `connection()` API로 대체되어 deprecated |
| **v14.0.0** | 최초 도입 |

---

## 마이그레이션 가이드

### 기존 코드 (unstable_noStore)

```tsx
import { unstable_noStore as noStore } from 'next/cache'

export default async function Component() {
  noStore()
  const data = await getData()
  return <div>{data}</div>
}
```

### 권장 방법 (connection)

```tsx
import { connection } from 'next/server'

export default async function Component() {
  await connection()
  const data = await getData()
  return <div>{data}</div>
}
```

---

## unstable_cache와 함께 사용 시 주의사항

```tsx
import { unstable_noStore as noStore, unstable_cache } from 'next/cache'

const getCachedData = unstable_cache(
  async () => {
    noStore() // ⚠️ 여기서는 효과 없음
    return await db.query(...)
  },
  ['cache-key'],
  { revalidate: 60 }
)

export default async function Component() {
  // unstable_cache의 설정이 우선 적용됨
  const data = await getCachedData()
}
```

**주의:** `unstable_cache` 내부에서 `noStore()`를 호출해도 캐시 동작에는 영향을 주지 않습니다. 캐시 구성이 우선 적용됩니다.

---

## 사용 시나리오

### 적합한 경우 (레거시 코드)
- ✅ 특정 컴포넌트만 동적으로 만들어야 하는 경우
- ✅ 페이지의 일부는 정적, 일부는 동적으로 유지하려는 경우
- ✅ `dynamic = 'force-dynamic'`보다 세밀한 제어가 필요한 경우

### 권장하지 않는 경우
- ❌ **새로운 프로젝트**: `connection()` API 사용 권장
- ❌ **Next.js 15 이상**: deprecated API이므로 마이그레이션 필요
- ❌ 전체 페이지를 동적으로 만들 때: `dynamic = 'force-dynamic'` 사용

---

## 대안

| 상황 | 권장 방법 |
|------|----------|
| **새 프로젝트 (v15+)** | [`connection()`](./connection.md) |
| **전체 페이지 동적화** | `export const dynamic = 'force-dynamic'` |
| **특정 fetch만 동적화** | `fetch(url, { cache: 'no-store' })` |
| **데이터 캐싱 비활성화** | `fetch(url, { cache: 'no-cache' })` |

---

## 관련 API

- [`connection()`](./connection.md) - **권장 대안** (v15+)
- [`fetch()`](./fetch.md) - Next.js 확장 fetch API
- [`unstable_cache()`](./unstable_cache.md) - 데이터 캐싱
- [`revalidatePath()`](./revalidatePath.md) - 경로 재검증
- [`revalidateTag()`](./revalidateTag.md) - 태그 재검증

---

## 요약

- **현재 상태:** Deprecated (v15.0.0+)
- **대체 API:** `connection()`
- **주 용도:** 컴포넌트 단위 정적 렌더링 옵트아웃
- **권장사항:** 새 코드에서는 `connection()` 사용
- **레거시 코드:** 점진적으로 `connection()`으로 마이그레이션 권장
