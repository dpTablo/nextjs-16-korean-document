# browserDebugInfoInTerminal

> **참고**: 이 옵션은 v16.2.x부터 `logging.browserToTerminal`로 이동되었습니다. 새 프로젝트에서는 `logging.browserToTerminal`을 사용하세요.

`browserDebugInfoInTerminal`은 브라우저 콘솔 로그를 터미널로 전달하는 실험적 설정 옵션이었습니다.

## 마이그레이션 가이드

### 이전 (실험적)

```js
// next.config.js
module.exports = {
  experimental: {
    browserDebugInfoInTerminal: true,
  },
}
```

### 이후 (안정)

```js
// next.config.js
module.exports = {
  logging: {
    browserToTerminal: true,
  },
}
```

## 지원되는 값

새로운 `logging.browserToTerminal` 옵션은 다음 로그 레벨 값을 허용합니다:

| 값 | 설명 |
|----|------|
| `true` | 모든 로그 활성화 |
| `'warn'` | 경고만 표시 |
| `'error'` | 에러만 표시 |

### 예제

```js
// next.config.js
module.exports = {
  logging: {
    browserToTerminal: 'error', // 에러만 터미널에 표시
  },
}
```

## 주요 변경사항

실험적 버전의 객체 설정 옵션들은 **더 이상 사용할 수 없습니다**:
- `depthLimit`
- `edgeLimit`
- `showSourceLocation`

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| v15.4.0 | `experimental.browserDebugInfoInTerminal` 도입 |
| v16.2.x | `logging.browserToTerminal`로 안정화 |

---

## 참고

- [logging](/app-router/api-reference/config/next-config-js/logging.md)
- [디버깅 가이드](/app-router/guides/debugging.md)
