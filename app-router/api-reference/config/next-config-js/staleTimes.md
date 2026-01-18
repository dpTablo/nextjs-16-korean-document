# staleTimes

`staleTimes`는 클라이언트 사이드 라우터 캐시에서 페이지 세그먼트의 캐싱을 활성화하는 실험적 기능입니다.

`next.config.js` 파일에서 실험적 `staleTimes` 플래그를 설정하여 이 실험적 기능을 활성화하고 커스텀 재검증 시간을 제공할 수 있습니다:

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
      static: 180,
    },
  },
}

module.exports = nextConfig
```

## 매개변수

### `static` (초)

- **사용 대상:** 정적으로 생성된 페이지 또는 Link의 `prefetch={true}`나 `router.prefetch()`가 호출된 경우
- **기본값:** 5분 (300초)

### `dynamic` (초)

- **사용 대상:** 정적으로 생성되지 않았거나 완전히 프리페치되지 않은 페이지 (예: `prefetch={true}` 없이)
- **기본값:** 0초 (캐시되지 않음)

## 주요 동작

- **Loading 경계**는 정의된 `static` 기간 동안 재사용 가능한 것으로 간주됩니다
- 부분 렌더링에는 **영향을 미치지 않습니다** — 공유 레이아웃은 모든 네비게이션에서 자동으로 다시 가져오지 않습니다. 변경되는 페이지 세그먼트만 업데이트됩니다
- 레이아웃 이동을 방지하고 스크롤 위치를 유지하기 위해 **뒤로/앞으로 캐싱 동작을 변경하지 않습니다**

## 버전 기록

| 버전     | 변경 사항                              |
| -------- | -------------------------------------- |
| `v15.0.0` | `dynamic` 기본값이 30초에서 0초로 변경됨 |
| `v14.2.0` | 실험적 `staleTimes` 도입됨              |

## 관련 문서

- [클라이언트 사이드 라우터 캐시](/docs/app/building-your-application/caching#client-side-router-cache)
- [Link 컴포넌트 프리페칭](/docs/app/api-reference/components/link#prefetch)
