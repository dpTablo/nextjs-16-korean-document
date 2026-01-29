---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/compress
버전: 16.1.6
---

# compress

기본적으로 Next.js는 `next start`를 사용하거나 커스텀 서버를 사용할 때 렌더링된 콘텐츠와 정적 파일에 대해 **gzip** 압축을 사용합니다. 이는 압축이 아직 구성되지 않은 애플리케이션을 위한 최적화입니다. 애플리케이션에서 커스텀 서버를 통해 압축이 _이미_ 구성된 경우, Next.js는 압축을 추가하지 않습니다.

## 작동 방식

- Next.js는 대역폭과 성능을 최적화하기 위해 자동으로 gzip 압축을 적용합니다
- 압축이 애플리케이션에 이미 구성되어 있는 경우 (예: 커스텀 서버를 통해), Next.js는 추가 압축을 추가하지 않습니다
- HTTP 응답 헤더를 확인하여 압축 설정을 확인할 수 있습니다:
  - **`Accept-Encoding`** - 브라우저가 허용하는 압축 옵션
  - **`Content-Encoding`** - 현재 사용 중인 압축 알고리즘

## 압축 비활성화

압축을 비활성화하려면 `compress` 구성 옵션을 `false`로 설정하세요:

```js filename="next.config.js"
module.exports = {
  compress: false,
}
```

## 권장 사항

다음 경우가 아니라면 압축을 비활성화하지 **않는 것이** 좋습니다:

- 서버에 압축이 이미 구성되어 있는 경우
- 다른 압축 알고리즘을 사용하고 싶은 경우 (예: nginx를 통한 brotli)

### 사용 사례 예제

**nginx**를 사용하고 gzip 대신 **brotli** 압축으로 전환하려는 경우:

1. Next.js 설정에서 `compress: false`를 설정합니다
2. nginx가 brotli로 압축을 처리하도록 합니다

대체 방법 없이 압축을 비활성화하면 애플리케이션 성능에 부정적인 영향을 미치고 대역폭 사용량이 증가합니다.
