---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/cacheComponents
버전: 16.1.6
---

# cacheComponents

## 개요

`cacheComponents`는 App Router에서 데이터 가져오기 작업을 명시적으로 캐시하지 않는 한 사전 렌더링에서 제외하는 Next.js 기능입니다. 런타임에 새로운 데이터 가져오기를 가능하게 하여 Server Components의 동적 데이터 가져오기 성능을 최적화합니다.

## 설정

`next.config.ts`에서 `cacheComponents` 활성화:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  cacheComponents: true,
}

module.exports = nextConfig
```

## 사용 패턴

활성화되면 `cacheComponents`는 `use cache`와 함께 작동합니다:
- 데이터 가져오기가 **기본적으로 런타임에** 발생
- 페이지, 함수 또는 컴포넌트 레벨에서 `use cache`를 사용하여 애플리케이션의 특정 부분을 캐시할 수 있음

## 사용 가능한 캐시 기능

`cacheComponents`가 활성화되면 다음을 사용할 수 있습니다:

| 기능 | 설명 |
|------|------|
| `use cache` 지시어 | 컴포넌트/함수를 캐싱 대상으로 표시 |
| `cacheLife` 함수 | `use cache`와 함께 캐시 기간 구성 |
| `cacheTag` 함수 | 캐시 항목에 태그 지정 |

## 예제

```tsx filename="app/page.tsx"
import { cacheLife, cacheTag } from 'next/cache'

async function getData() {
  'use cache'
  cacheLife('blog')
  cacheTag('blog-data')

  const res = await fetch('https://api.example.com/data')
  return res.json()
}

export default async function Page() {
  const data = await getData()
  return <div>{data.title}</div>
}
```

## 성능 고려사항

| 장점 | 단점 |
|------|------|
| 런타임 중 새로운 데이터 가져오기 보장 | 사전 렌더링된 콘텐츠 제공에 비해 추가 지연 시간 발생 가능 |
| 동적 데이터 시나리오 최적화 | |

## 통합 플래그

`cacheComponents`는 다음 플래그를 통합 구성으로 제어합니다:
- `ppr` (Partial Prerendering)
- `useCache`
- `dynamicIO`

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v16.0.0 | `cacheComponents` 도입; `ppr`, `useCache`, `dynamicIO` 플래그를 통합 구성으로 제어 |
