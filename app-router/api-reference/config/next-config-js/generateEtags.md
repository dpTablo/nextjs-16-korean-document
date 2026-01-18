# generateEtags

Next.js는 기본적으로 모든 페이지에 대해 [ETag](https://en.wikipedia.org/wiki/HTTP_ETag)를 생성합니다. 캐시 전략에 따라 HTML 페이지에 대한 ETag 생성을 비활성화하고 싶을 수 있습니다.

`next.config.js`를 열고 `generateEtags` 옵션을 `false`로 설정하세요:

```js filename="next.config.js"
module.exports = {
  generateEtags: false,
}
```

## 세부 사항

- **기본 동작**: 모든 페이지에 대해 ETag가 자동으로 생성됩니다
- **사용 사례**: ETag가 필요하지 않은 특정 캐시 전략이 있는 경우 이 옵션을 비활성화하세요
- **영향**: HTML 페이지 캐싱 동작과 HTTP 캐시 검증에 영향을 미칩니다
