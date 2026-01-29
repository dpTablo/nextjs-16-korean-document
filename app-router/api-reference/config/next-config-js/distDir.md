---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/distDir
버전: 16.1.6
---

# distDir

기본 `.next` 디렉토리 대신 사용할 커스텀 빌드 디렉토리 이름을 지정할 수 있습니다.

`next.config.js`를 열고 `distDir` 구성을 추가하세요:

```js filename="next.config.js"
module.exports = {
  distDir: 'build',
}
```

이제 `next build`를 실행하면 Next.js는 기본 `.next` 폴더 대신 `build`를 사용합니다.

## 제약 사항

> **중요**: `distDir`은 프로젝트 디렉토리를 벗어나면 **안 됩니다**. 예를 들어 `../build`는 **유효하지 않은** 대상입니다.

- ✅ 유효: `'build'`, `'dist'`, `'.next'`
- ❌ 유효하지 않음: `'../build'` (프로젝트 디렉토리 외부)
