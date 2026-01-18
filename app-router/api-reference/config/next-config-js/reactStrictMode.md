# reactStrictMode

> **알아두면 좋은 점**: Next.js 13.5.1부터 `app` 라우터를 사용할 때 Strict Mode는 기본적으로 `true`이므로, 위 설정은 `pages`에만 필요합니다. `reactStrictMode: false`를 설정하여 Strict Mode를 비활성화할 수 있습니다.

> **권장 사항**: Next.js 애플리케이션에서 Strict Mode를 활성화하여 향후 React 개발에 더 잘 대비할 수 있도록 하는 것을 강력히 권장합니다.

React의 [Strict Mode](https://react.dev/reference/react/StrictMode)는 애플리케이션의 잠재적인 문제를 강조하는 개발 모드 전용 기능입니다. 안전하지 않은 생명주기, 레거시 API 사용 및 기타 여러 기능을 식별하는 데 도움이 됩니다.

Next.js 런타임은 Strict Mode를 준수합니다. Strict Mode를 선택하려면 `next.config.js`에서 다음 옵션을 구성하세요:

```js filename="next.config.js"
module.exports = {
  reactStrictMode: true,
}
```

팀이 전체 애플리케이션에서 Strict Mode를 사용할 준비가 되지 않은 경우, `<React.StrictMode>`를 사용하여 페이지별로 점진적으로 마이그레이션할 수 있습니다.

## 주요 포인트

- 개발 모드에서만 작동합니다 (프로덕션에는 영향 없음)
- 안전하지 않은 패턴과 더 이상 사용되지 않는 API를 식별하는 데 도움이 됩니다
- Next.js 런타임은 완전히 Strict Mode를 준수합니다
- 필요한 경우 비활성화 가능: `reactStrictMode: false`
