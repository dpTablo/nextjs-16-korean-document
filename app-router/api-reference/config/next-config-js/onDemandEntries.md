# onDemandEntries

Next.js는 개발 중 서버가 빌드된 페이지를 메모리에 관리하거나 유지하는 방법을 제어할 수 있는 몇 가지 옵션을 제공합니다.

기본값을 변경하려면 `next.config.js`를 열고 `onDemandEntries` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  onDemandEntries: {
    // 서버가 페이지를 버퍼에 유지할 기간 (밀리초)
    maxInactiveAge: 25 * 1000,
    // 폐기되지 않고 동시에 유지되어야 하는 페이지 수
    pagesBufferLength: 2,
  },
}
```

## 매개변수

| 매개변수 | 타입 | 기본값 | 설명 |
|----------|------|--------|------|
| `maxInactiveAge` | number (ms) | 25000 | 서버가 페이지를 폐기하기 전에 버퍼에 유지할 기간 (밀리초) |
| `pagesBufferLength` | number | 2 | 폐기되지 않고 동시에 메모리에 유지되어야 하는 페이지 수 |

## 목적

이 구성을 통해 개발자는 다음을 제어하여 개발 중 메모리 사용량과 페이지 로딩 동작을 최적화할 수 있습니다:

- 비활성 페이지가 서버 메모리에 남아 있는 기간
- 주어진 시간에 버퍼에 유지되는 페이지 수
