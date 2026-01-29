---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/httpAgentOptions
버전: 16.1.6
---

# httpAgentOptions

Next.js 18 이전 Node.js 버전에서는 `fetch()`를 [undici](/docs/architecture/supported-browsers)로 자동 폴리필하고 기본적으로 [HTTP Keep-Alive](https://developer.mozilla.org/docs/Web/HTTP/Headers/Keep-Alive)를 활성화합니다.

서버 사이드의 모든 `fetch()` 호출에 대해 HTTP Keep-Alive를 비활성화하려면 `next.config.js`를 열고 `httpAgentOptions` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  httpAgentOptions: {
    keepAlive: false,
  },
}
```

## 매개변수

- **`keepAlive`** (boolean): HTTP Keep-Alive 동작을 제어합니다
  - `true` (기본값): HTTP Keep-Alive를 활성화합니다
  - `false`: HTTP Keep-Alive를 비활성화합니다
