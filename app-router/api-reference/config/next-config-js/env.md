---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/env
버전: 16.1.6
---

# env

> Next.js 9.4 출시 이후, [환경 변수를 추가하는 더 직관적이고 인체공학적인 경험](/docs/app/building-your-application/configuring/environment-variables)이 제공됩니다. 한번 시도해 보세요!

> **알아두면 좋은 점**: 이 방식으로 지정된 환경 변수는 **항상** JavaScript 번들에 포함됩니다. 환경 변수 이름 앞에 `NEXT_PUBLIC_`을 붙이는 것은 [환경 또는 .env 파일을 통해 지정할 때](/docs/app/building-your-application/configuring/environment-variables)만 효과가 있습니다.

JavaScript 번들에 환경 변수를 추가하려면 `next.config.js`를 열고 `env` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  env: {
    customKey: 'my-value',
  },
}
```

이제 코드에서 `process.env.customKey`에 접근할 수 있습니다. 예를 들어:

```jsx
function Page() {
  return <h1>customKey의 값은: {process.env.customKey}</h1>
}

export default Page
```

Next.js는 빌드 시점에 `process.env.customKey`를 `'my-value'`로 대체합니다. webpack [DefinePlugin](https://webpack.js.org/plugins/define-plugin/)의 특성으로 인해 `process.env` 변수를 구조 분해하려고 하면 작동하지 않습니다.

예를 들어, 다음 줄:

```jsx
return <h1>customKey의 값은: {process.env.customKey}</h1>
```

다음과 같이 됩니다:

```jsx
return <h1>customKey의 값은: {'my-value'}</h1>
```

## 권장 사항

이 레거시 API 대신 최신 [환경 변수 접근 방식](/docs/app/building-your-application/configuring/environment-variables)을 사용하는 것이 좋습니다. 더 나은 개발자 경험과 보안을 제공합니다.
