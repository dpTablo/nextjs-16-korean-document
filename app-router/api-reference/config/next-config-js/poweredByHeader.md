# poweredByHeader

기본적으로 Next.js는 `x-powered-by` 헤더를 추가합니다. 이를 비활성화하려면 `next.config.js`를 열고 `poweredByHeader` 구성을 비활성화하세요:

```js filename="next.config.js"
module.exports = {
  poweredByHeader: false,
}
```

## 요약

- **기본 동작:** Next.js는 기본적으로 `x-powered-by` 헤더를 추가합니다
- **목적:** 이 헤더를 비활성화하면 애플리케이션이 Next.js로 구동된다는 것을 식별하는 헤더가 제거됩니다
- **구성 타입:** `next.config.js`의 Boolean 옵션
- **값:** 헤더를 비활성화하려면 `false`로 설정

이것은 HTTP 응답에서 `x-powered-by` 헤더가 전송되는 것을 방지하는 간단한 보안/개인정보 관련 구성입니다.
