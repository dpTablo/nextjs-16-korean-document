# webVitalsAttribution

Web Vitals와 관련된 문제를 디버깅할 때 문제의 원인을 찾아내는 것이 도움이 되는 경우가 많습니다. 예를 들어, Cumulative Layout Shift(CLS)의 경우 가장 큰 레이아웃 시프트 중에 이동한 첫 번째 요소가 무엇인지 알고 싶을 수 있습니다. 또는 Largest Contentful Paint(LCP)의 경우 페이지의 LCP에 해당하는 요소를 식별하고 싶을 수 있습니다. LCP 요소가 이미지인 경우 리소스의 URL을 아는 것이 최적화해야 할 자산을 찾는 데 도움이 될 수 있습니다.

Web Vitals 점수에서 가장 큰 기여자를 찾아내는 것([attribution](https://github.com/GoogleChrome/web-vitals/blob/4ca38ae64b8d1e899028c692f94d4c56acfc996c/README.md#extractattributionfromevententryl)이라고 함)을 통해 [PerformanceEventTiming](https://developer.mozilla.org/docs/Web/API/PerformanceEventTiming), [PerformanceNavigationTiming](https://developer.mozilla.org/docs/Web/API/PerformanceNavigationTiming), [PerformanceResourceTiming](https://developer.mozilla.org/docs/Web/API/PerformanceResourceTiming)에서 더 심층적인 정보를 얻을 수 있습니다.

Attribution은 Next.js에서 기본적으로 비활성화되어 있지만 `next.config.js`에서 다음을 지정하여 **메트릭별로** 활성화할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  experimental: {
    webVitalsAttribution: ['CLS', 'LCP'],
  },
}
```

## 유효한 값

[`NextWebVitalsMetric`](https://github.com/vercel/next.js/blob/442378d21dd56d6e769863eb8c2cb521a463a2e0/packages/next/shared/lib/utils.ts#L43) 타입에 지정된 모든 `web-vitals` 메트릭을 사용할 수 있습니다.

## 목적

Attribution을 통해 개발자는 Web Vitals 점수에 가장 큰 영향을 미치는 특정 요소나 리소스를 식별할 수 있습니다:

- **CLS (Cumulative Layout Shift)**: 가장 큰 레이아웃 시프트 중에 이동한 첫 번째 요소 식별
- **LCP (Largest Contentful Paint)**: LCP에 해당하는 요소와 리소스 URL 식별 (이미지 최적화에 유용)
