# trailingSlash

기본적으로 Next.js는 후행 슬래시가 있는 URL을 후행 슬래시가 없는 대응 URL로 리디렉션합니다. 예를 들어 `/about/`은 `/about`으로 리디렉션됩니다. `trailingSlash` 구성을 사용하여 이 동작을 반대로 구성할 수 있습니다.

`next.config.js`를 열고 `trailingSlash` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  trailingSlash: true,
}
```

이 옵션을 설정하면 `/about`과 같은 URL은 `/about/`으로 리디렉션됩니다.

[`output: "export"`](/docs/app/building-your-application/deploying/static-exports) 구성과 함께 사용할 때, `/about` 페이지는 `/about.html` 대신 `/about/index.html`로 출력됩니다.

## 예외 사항

`trailingSlash: true`가 설정되어 있을 때, 다음 URL에는 후행 슬래시가 추가되지 **않습니다**:

1. **정적 파일 URL** - 확장자가 있는 파일
   - 예: `/file.txt`, `images/photos/picture.png`

2. **`.well-known/` 경로** - `.well-known/` 디렉토리 아래의 모든 경로
   - 예: `.well-known/subfolder/config.json`

## 버전 기록

| 버전     | 변경 사항             |
| -------- | --------------------- |
| `v9.5.0` | `trailingSlash` 추가됨 |
