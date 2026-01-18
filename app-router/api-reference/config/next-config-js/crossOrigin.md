# crossOrigin

`crossOrigin` 옵션을 사용하면 `next/script` 컴포넌트에서 생성된 모든 `<script>` 태그에 `crossOrigin` 속성을 추가하여 교차 출처 요청을 처리하는 방법을 정의합니다.

`next.config.js`에 `crossOrigin` 옵션을 추가하세요:

```js filename="next.config.js"
module.exports = {
  crossOrigin: 'anonymous',
}
```

## 사용 가능한 옵션

| 옵션 | 동작 |
|------|------|
| `'anonymous'` | `crossOrigin="anonymous"` 속성을 추가합니다. 자격 증명 없이 요청이 전송됩니다. |
| `'use-credentials'` | `crossOrigin="use-credentials"` 속성을 추가합니다. 요청에 자격 증명(쿠키, 인증 헤더 등)이 포함됩니다. |

## 관련 리소스

- [`next/script` 컴포넌트 문서](/docs/app/building-your-application/optimizing/scripts)
- [MDN: crossorigin 속성](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)
