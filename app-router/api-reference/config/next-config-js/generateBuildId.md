---
원문: https://nextjs.org/docs/app/api-reference/config/next-config-js/generateBuildId
버전: 16.1.6
---

# generateBuildId

Next.js는 빌드 시 생성된 ID를 사용하여 애플리케이션의 어떤 버전이 제공되고 있는지 식별합니다. 이는 여러 컨테이너를 부팅할 때 동일한 빌드 ID를 사용해야 하는 다중 서버 배포에서 문제가 될 수 있습니다.

각 환경 스테이지마다 애플리케이션을 다시 빌드하는 경우, 컨테이너 간에 사용할 일관된 빌드 ID를 생성해야 합니다. `next.config.js`에서 `generateBuildId` 명령을 사용하세요:

```js filename="next.config.js"
module.exports = {
  generateBuildId: async () => {
    // 이것은 무엇이든 될 수 있습니다. 최신 git 해시를 사용합니다
    return process.env.GIT_HASH
  },
}
```

## 주요 세부 사항

- 함수는 **비동기**이며 문자열 값을 반환해야 합니다
- 빌드 ID에 모든 소스를 사용할 수 있습니다 (환경 변수, git 해시, 타임스탬프 등)
- 일반적인 패턴: `process.env.GIT_HASH`를 사용하여 빌드를 특정 커밋에 연결
- 동일한 빌드가 여러 컨테이너에 배포될 때 일관성을 보장합니다

## 사용 사례

각 환경 스테이지마다 애플리케이션을 다시 빌드할 때, `generateBuildId`를 구현하면 컨테이너 간에 공유할 수 있는 일관된 빌드 ID를 생성하여 배포 인프라 전반에서 버전 불일치를 방지할 수 있습니다.
