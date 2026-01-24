# Instrumentation 설정하기

## 개요

Instrumentation은 코드를 사용하여 모니터링 및 로깅 도구를 애플리케이션에 통합하는 프로세스입니다. 이를 통해 애플리케이션의 성능과 동작을 추적하고 프로덕션에서 문제를 디버깅할 수 있습니다.

## 규칙

Instrumentation을 설정하려면 프로젝트의 **루트 디렉토리**(또는 [`src`](/app-router/api-reference/file-conventions/src-folder.md) 폴더를 사용하는 경우 그 안)에 `instrumentation.ts|js` 파일을 생성합니다.

그런 다음 파일에서 `register` 함수를 export합니다. 이 함수는 새로운 Next.js 서버 인스턴스가 시작될 때 **한 번** 호출됩니다.

예를 들어, Next.js를 [OpenTelemetry](https://opentelemetry.io/)와 [@vercel/otel](https://vercel.com/docs/observability/otel-overview)로 사용하려면:

```ts filename="instrumentation.ts"
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel('next-app')
}
```

```js filename="instrumentation.js"
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel('next-app')
}
```

완전한 구현 예제는 [Next.js with OpenTelemetry example](https://github.com/vercel/next.js/tree/canary/examples/with-opentelemetry)을 참조하세요.

> **알아두면 좋은 점:**
>
> - `instrumentation` 파일은 프로젝트 루트에 있어야 하며 `app` 또는 `pages` 디렉토리 내에 있으면 안 됩니다. `src` 폴더를 사용하는 경우, `pages`와 `app`과 함께 `src` 내에 파일을 배치합니다.
> - [`pageExtensions` config option](/app-router/api-reference/config/next-config-js/pageExtensions.md)을 사용하여 접미사를 추가하는 경우, `instrumentation` 파일명도 일치하도록 업데이트해야 합니다.

## 예제

### 부작용이 있는 파일 import하기

때때로 부작용 때문에 파일을 코드에 import하는 것이 유용할 수 있습니다. 예를 들어, 전역 변수 집합을 정의하는 파일을 import하지만 import된 파일을 명시적으로 사용하지 않을 수 있습니다. 그래도 패키지에서 선언한 전역 변수에 액세스할 수 있습니다.

`register` 함수 내에서 JavaScript `import` 구문을 사용하여 파일을 import하는 것을 권장합니다. 다음 예제는 `register` 함수에서 `import`의 기본 사용법을 보여줍니다:

```ts filename="instrumentation.ts"
export async function register() {
  await import('package-with-side-effect')
}
```

```js filename="instrumentation.js"
export async function register() {
  await import('package-with-side-effect')
}
```

> **알아두면 좋은 점:**
>
> 파일을 파일의 맨 위에서가 아니라 `register` 함수 내에서 import하는 것을 권장합니다. 이렇게 하면 모든 부작용을 코드의 한 곳에 함께 배치할 수 있으며, 파일 맨 위에서 전역적으로 import하는 것으로 인한 의도하지 않은 결과를 피할 수 있습니다.

### Runtime별 코드 import하기

Next.js는 모든 환경에서 `register`를 호출하므로, 특정 런타임을 지원하지 않는 코드(예: [Edge 또는 Node.js](/app-router/api-reference/edge.md))를 조건부로 import하는 것이 중요합니다. `NEXT_RUNTIME` 환경 변수를 사용하여 현재 환경을 확인할 수 있습니다:

```ts filename="instrumentation.ts"
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation-node')
  }

  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./instrumentation-edge')
  }
}
```

```js filename="instrumentation.js"
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation-node')
  }

  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./instrumentation-edge')
  }
}
```

## 더 알아보기

- [instrumentation.js](/app-router/api-reference/file-conventions/instrumentation.md) - instrumentation.js 파일의 API 참조
