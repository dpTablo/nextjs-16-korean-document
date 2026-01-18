# productionBrowserSourceMaps

소스 맵은 개발 중에는 기본적으로 활성화되지만, 소스 코드가 클라이언트에 노출되는 것을 방지하기 위해 프로덕션 빌드에서는 기본적으로 비활성화됩니다.

Next.js는 `productionBrowserSourceMaps` 구성 플래그를 제공하여 프로덕션 빌드 중에 브라우저 소스 맵 생성을 활성화할 수 있습니다:

```js filename="next.config.js"
module.exports = {
  productionBrowserSourceMaps: true,
}
```

## 동작

`productionBrowserSourceMaps` 옵션이 활성화되면:

- 소스 맵이 JavaScript 파일과 동일한 디렉토리에 출력됩니다
- 요청 시 Next.js가 자동으로 이 파일을 제공합니다
- 프로덕션 빌드 중에 소스 맵이 생성됩니다

## 성능 영향

> **알아두면 좋은 점**: 이 옵션을 활성화하면 `next build` 시간이 증가하고 빌드 중 메모리 사용량이 증가합니다.

## 사용 사례

다음이 필요할 때 이 플래그를 활성화하세요:

- 프로덕션에서 향상된 디버깅 기능
- 문제 해결을 위한 원본 소스 코드 접근
- 프로덕션 환경에서 개발과 유사한 디버깅 경험
