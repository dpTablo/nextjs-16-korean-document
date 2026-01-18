# logging

`next.config.js`의 `logging` 옵션을 사용하면 Next.js 개발 모드에서 로깅 동작을 구성할 수 있습니다. 현재 이 옵션은 주로 `fetch` API를 사용한 데이터 페칭과 들어오는 요청에 적용됩니다.

## Fetch 로깅

### 전체 URL 로깅

fetch 요청의 전체 URL을 로깅합니다:

```js filename="next.config.js"
module.exports = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}
```

### HMR 새로고침 로깅

기본적으로 서버 컴포넌트 HMR 캐시에서 복원된 fetch 요청은 로깅되지 않습니다. HMR 새로고침에 대한 로깅을 활성화하려면:

```js filename="next.config.js"
module.exports = {
  logging: {
    fetches: {
      hmrRefreshes: true,
    },
  },
}
```

## 들어오는 요청 로깅

개발 중에는 기본적으로 모든 들어오는 요청이 로깅됩니다. 정규식 패턴을 사용하여 무시할 요청을 필터링할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  logging: {
    incomingRequests: {
      ignore: [/\api\/v1\/health/],
    },
  },
}
```

들어오는 요청 로깅을 완전히 비활성화하려면:

```js filename="next.config.js"
module.exports = {
  logging: {
    incomingRequests: false,
  },
}
```

## 모든 로깅 비활성화

```js filename="next.config.js"
module.exports = {
  logging: false,
}
```

## 중요 참고 사항

- **개발 전용**: 들어오는 요청 로깅은 개발 중에만 적용되며 프로덕션 빌드에는 영향을 미치지 않습니다
- **제한된 범위**: 현재 `fetch` API 호출과 들어오는 요청에만 적용되며, 다른 Next.js 로그에는 적용되지 않습니다
