# devIndicators

`devIndicators`를 사용하면 개발 중 현재 라우트에 대한 컨텍스트를 표시하는 화면 표시기를 구성할 수 있습니다.

```ts
devIndicators: false | {
  position?: 'bottom-right'
    | 'bottom-left'
    | 'top-right'
    | 'top-left', // 기본값은 'bottom-left'
}
```

## 구성 옵션

- `false`로 설정하면 표시기가 숨겨지지만, Next.js는 계속해서 발생하는 빌드 또는 런타임 오류를 표시합니다.
- `position`은 브라우저에서 표시기의 위치를 설정합니다(기본값: `bottom-left`).

## 표시기 기호

표시기를 클릭하면 현재 라우트에 대한 정보가 표시됩니다. 표시기는 다음 기호를 사용하여 라우트 렌더링 유형을 보여줍니다:

- **`○`** (정적): 라우트가 정적 콘텐츠로 사전 렌더링됩니다
- **`ƒ`** (동적): 라우트가 요청 시 서버에서 렌더링됩니다

## 라우트가 정적이 아닌지 확인하기

`next build --debug`를 실행하고 터미널 출력에서 라우트 기호를 확인할 수 있습니다.

**출력 예제:**
```bash
Route (app)
┌ ○ /_not-found
└ ƒ /products/[id]

○  (Static)   정적 콘텐츠로 사전 렌더링
ƒ  (Dynamic)  요청 시 서버 렌더링
```

## 동적 라우트의 일반적인 원인

### 동적 API

다음 함수는 런타임 정보에 의존하며 자동으로 라우트를 동적으로 만듭니다:
- `cookies()`
- `headers()`
- `connection()`
- `searchParams`
- `unstable_noStore()`

### 캐시되지 않은 데이터 요청

데이터 요청이 기본적으로 캐시되지 않는 경우, 라우트가 동적으로 될 수 있습니다. 이는 일반적으로 ORM이나 데이터베이스 클라이언트를 사용한 직접 호출에서 발생합니다.

## 해결 방법

- 로딩 상태를 위해 [`loading.js`](/docs/app/api-reference/file-conventions/loading)를 사용하세요
- 스트리밍을 위해 [`<Suspense />`](https://react.dev/reference/react/Suspense)를 구현하세요
- 데이터 페칭 패턴을 검토하고 최적화하세요

## 버전 기록

| 버전      | 변경 사항                                                         |
| --------- | ----------------------------------------------------------------- |
| `v16.0.0` | `appIsrStatus`, `buildActivity`, `buildActivityPosition` 제거됨   |
| `v15.2.0` | 새로운 `position` 옵션 추가됨; 이전 옵션들 더 이상 사용되지 않음   |
| `v15.0.0` | 초기 정적 표시기 추가됨                                           |
