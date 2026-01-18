# typescript

프로젝트에 TypeScript 오류가 있으면 Next.js는 **프로덕션 빌드**(`next build`)를 실패시킵니다.

애플리케이션에 오류가 있더라도 Next.js가 위험하게 프로덕션 코드를 생성하도록 하려면, 내장된 타입 검사 단계를 비활성화할 수 있습니다.

비활성화하는 경우, 빌드 또는 배포 프로세스의 일부로 타입 검사를 실행하고 있는지 확인하세요. 그렇지 않으면 매우 위험할 수 있습니다.

`next.config.js`를 열고 `typescript` 구성에서 `ignoreBuildErrors` 옵션을 활성화하세요:

```js filename="next.config.js"
module.exports = {
  typescript: {
    // !! 경고 !!
    // 프로젝트에 타입 오류가 있어도 프로덕션 빌드가
    // 성공적으로 완료되도록 위험하게 허용합니다.
    // !! 경고 !!
    ignoreBuildErrors: true,
  },
}
```

> **알아두면 좋은 점**: `tsc --noEmit`을 실행하여 빌드 전에 직접 TypeScript 오류를 확인할 수 있습니다. 이는 배포 전에 TypeScript 오류를 확인하려는 CI/CD 파이프라인에 유용합니다.

## 구성 옵션

| 옵션               | 타입      | 기본값            | 설명                                              |
| ------------------ | --------- | ----------------- | ------------------------------------------------- |
| `ignoreBuildErrors`| `boolean` | `false`           | TypeScript 오류가 있어도 프로덕션 빌드 완료 허용   |
| `tsconfigPath`     | `string`  | `'tsconfig.json'` | 커스텀 `tsconfig.json` 파일의 경로                 |

## tsconfigPath

Next.js가 기본 `tsconfig.json` 대신 커스텀 TypeScript 구성 파일을 사용하도록 설정할 수 있습니다. 이는 빌드나 도구에 다른 TypeScript 설정이 필요한 경우 유용합니다.

```js filename="next.config.js"
module.exports = {
  typescript: {
    tsconfigPath: 'tsconfig.build.json',
  },
}
```
