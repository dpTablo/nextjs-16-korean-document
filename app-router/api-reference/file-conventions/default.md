# default.js

## 개요
`default.js` 파일은 **Parallel Routes**에서 Next.js가 전체 페이지 로드 후 슬롯의 활성 상태를 복구할 수 없을 때 사용되는 폴백 컴포넌트입니다.

## 목적 및 동작

### 사용 시기
- **소프트 네비게이션**: Next.js는 각 슬롯의 활성 상태를 추적합니다
- **하드 네비게이션 (전체 페이지 로드)**: 활성 상태를 복구할 수 없으므로 `default.js`가 일치하지 않는 하위 페이지에 대한 폴백으로 렌더링됩니다

### 예제 시나리오
`@team`에는 `settings` 페이지가 있지만 `@analytics`에는 없는 구조를 가정합니다:
- `/settings`로 네비게이션: `@team`은 settings 페이지를 렌더링하고 `@analytics`는 현재 페이지를 유지합니다
- 페이지 새로고침: `@analytics`에 일치하는 라우트가 없으므로 `default.js`가 렌더링됩니다

## 필수 구현

### Named Slots의 경우
Named 슬롯(`@team`, `@analytics` 등)에 대해 `default.js`가 존재하지 않으면 오류가 반환됩니다. 다음 중 하나를 수행해야 합니다:

1. **폴백 컴포넌트 생성** - 기본 콘텐츠를 렌더링
2. **404 동작 유지** - `notFound()` 사용:

```tsx
// app/@team/default.js
import { notFound } from 'next/navigation'

export default function Default() {
  notFound()
}
```

### children 슬롯의 경우
`children` 슬롯은 암시적이며 마찬가지로 `default.js` 파일이 필요합니다. 없으면 라우트에 대해 404 페이지가 반환됩니다.

## Props 참조

### `params` (선택사항)
루트 세그먼트에서 슬롯의 하위 페이지까지의 동적 라우트 매개변수를 포함하는 객체로 해석되는 프로미스입니다.

**TypeScript 예제:**
```tsx
// app/[artist]/@sidebar/default.js
export default async function Default({
  params,
}: {
  params: Promise<{ artist: string }>
}) {
  const { artist } = await params
}
```

**JavaScript 예제:**
```jsx
// app/[artist]/@sidebar/default.js
export default async function Default({ params }) {
  const { artist } = await params
}
```

### 매개변수 예제

| 파일 경로 | URL | params |
|-----------|-----|--------|
| `app/[artist]/@sidebar/default.js` | `/zack` | `Promise<{ artist: 'zack' }>` |
| `app/[artist]/[album]/@sidebar/default.js` | `/zack/next` | `Promise<{ artist: 'zack', album: 'next' }>` |

## 중요 참고사항

- **`params`는 프로미스입니다** - 값에 접근하려면 `async/await` 또는 React의 [`use`](https://react.dev/reference/react/use) 함수를 사용하세요
- **하위 호환성** - Next.js 14 이하에서는 `params`가 동기식이었습니다. Next.js 15는 여전히 동기식 접근을 지원하지만 이 동작은 향후 deprecated될 예정입니다

## Parallel Routes 예제

```
app
├── @team
│   ├── settings
│   │   └── page.js
│   └── default.js
├── @analytics
│   └── default.js  // settings가 없으므로 폴백 필요
├── layout.js
└── page.js
```

**layout.js에서 Parallel Routes 사용:**
```tsx
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode
  team: React.ReactNode
  analytics: React.ReactNode
}) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v15.0.0 | `params`가 프로미스로 변경 |
| v13.0.0 | Parallel Routes 및 `default.js` 도입 |

## 관련 문서

- [Parallel Routes](https://nextjs.org/docs/app/building-your-application/routing/parallel-routes)
- [notFound](../functions/notFound.md)
- [layout.js](./layout.md)
