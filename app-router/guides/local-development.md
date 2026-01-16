# 로컬 개발 환경을 최적화하는 방법

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-10-16

---

## 개요

Next.js는 훌륭한 개발자 경험을 제공하도록 설계되었습니다. 애플리케이션이 성장함에 따라 로컬 개발 중 컴파일 시간이 느려지는 것을 느낄 수 있습니다. 이 가이드는 일반적인 컴파일 시간 성능 문제를 식별하고 해결하는 데 도움이 됩니다.

## 로컬 개발 vs. 프로덕션

`next dev`를 사용한 개발 프로세스는 `next build` 및 `next start`와 다릅니다.

- **`next dev`**는 페이지를 열거나 탐색할 때 애플리케이션의 라우트를 컴파일합니다. 이를 통해 애플리케이션의 모든 라우트가 컴파일될 때까지 기다리지 않고 개발 서버를 시작할 수 있으며, 이는 더 빠르고 메모리를 적게 사용합니다.
- **프로덕션 빌드**는 파일 압축 및 콘텐츠 해시 생성과 같은 다른 최적화를 적용하며, 이는 로컬 개발에는 필요하지 않습니다.

## 로컬 개발 성능 향상

### 1. 컴퓨터의 안티바이러스 확인

안티바이러스 소프트웨어는 파일 접근 속도를 늦출 수 있습니다. Windows 시스템에서 더 일반적이지만, 안티바이러스 도구가 설치된 모든 시스템에서 문제가 될 수 있습니다.

#### Windows

프로젝트를 [Microsoft Defender 안티바이러스 제외 목록](https://support.microsoft.com/en-us/windows/virus-and-threat-protection-in-the-windows-security-app-1362f4cd-d71a-b52a-0b66-c2820032b65e#bkmk_threat-protection-settings)에 추가할 수 있습니다.

1. **"Windows 보안"** 애플리케이션을 열고 **"바이러스 및 위협 방지"** → **"설정 관리"** → **"제외 추가 또는 제거"**를 선택합니다.
2. **"폴더"** 제외를 추가합니다. 프로젝트 폴더를 선택합니다.

#### macOS

터미널 내에서 [Gatekeeper](https://support.apple.com/guide/security/gatekeeper-and-runtime-protection-sec5599b66df/web)를 비활성화할 수 있습니다.

1. 터미널에서 `sudo spctl developer-mode enable-terminal`을 실행합니다.
2. **"시스템 설정"** 앱을 열고 **"개인정보 보호 및 보안"** → **"개발자 도구"**를 선택합니다.
3. 터미널이 목록에 있고 활성화되어 있는지 확인합니다. iTerm이나 Ghostty와 같은 서드파티 터미널을 사용하는 경우 목록에 추가합니다.
4. 터미널을 재시작합니다.

본인이나 회사에서 시스템에 다른 안티바이러스 솔루션을 구성한 경우, 해당 제품의 관련 설정을 확인해야 합니다.

### 2. Next.js 업데이트 및 Turbopack 사용

최신 버전의 Next.js를 사용하고 있는지 확인하세요. 각 새 버전에는 성능 향상이 포함되어 있습니다.

Turbopack은 이제 Next.js 개발의 기본 번들러이며 webpack보다 상당한 성능 향상을 제공합니다.

```bash
npm install next@latest
npm run dev  # Turbopack이 기본으로 사용됩니다
```

Turbopack 대신 Webpack을 사용해야 하는 경우 옵트인할 수 있습니다:

```bash
npm run dev --webpack
```

[Turbopack에 대해 자세히 알아보기](/blog/turbopack-for-development-stable). 자세한 내용은 [업그레이드 가이드](/app-router/guides/upgrading.md)와 codemods를 참조하세요.

### 3. 임포트 확인

코드를 임포트하는 방식은 컴파일 및 번들링 시간에 크게 영향을 줄 수 있습니다. [패키지 번들링 최적화](/app-router/guides/package-bundling.md)에 대해 자세히 알아보고 [Dependency Cruiser](https://github.com/sverweij/dependency-cruiser)나 [Madge](https://github.com/pahen/madge)와 같은 도구를 살펴보세요.

#### 아이콘 라이브러리

`@material-ui/icons`, `@phosphor-icons/react` 또는 `react-icons`와 같은 라이브러리는 몇 개만 사용하더라도 수천 개의 아이콘을 임포트할 수 있습니다. 필요한 아이콘만 임포트하세요:

```jsx
// 이것 대신:
import { TriangleIcon } from '@phosphor-icons/react'

// 이렇게 하세요:
import { TriangleIcon } from '@phosphor-icons/react/dist/csr/Triangle'
```

사용 중인 아이콘 라이브러리의 문서에서 어떤 임포트 패턴을 사용해야 하는지 찾을 수 있습니다. 이 예제는 [`@phosphor-icons/react`](https://www.npmjs.com/package/@phosphor-icons/react#import-performance-optimization) 권장 사항을 따릅니다.

`react-icons`와 같은 라이브러리는 여러 다른 아이콘 세트를 포함합니다. 하나의 세트를 선택하고 그 세트만 사용하세요.

예를 들어, 애플리케이션이 `react-icons`를 사용하고 다음 모두를 임포트하는 경우:

- `pi` (Phosphor Icons)
- `md` (Material Design Icons)
- `tb` (tabler-icons)
- `cg` (cssgg)

각각에서 단일 임포트만 사용하더라도 컴파일러가 처리해야 하는 수만 개의 모듈이 결합됩니다.

#### 배럴 파일

"배럴 파일"은 다른 파일에서 여러 항목을 내보내는 파일입니다. 컴파일러가 임포트를 사용하여 모듈 스코프에 부작용이 있는지 파악하기 위해 파싱해야 하므로 빌드 속도가 느려질 수 있습니다.

가능하면 특정 파일에서 직접 임포트하세요. [배럴 파일에 대해 자세히 알아보기](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js) 및 Next.js의 내장 최적화를 확인하세요.

#### 패키지 임포트 최적화

Next.js는 특정 패키지에 대해 임포트를 자동으로 최적화할 수 있습니다. 배럴 파일을 활용하는 패키지를 사용하는 경우 `next.config.js`에 추가하세요:

```jsx
module.exports = {
  experimental: {
    optimizePackageImports: ['package-name'],
  },
}
```

Turbopack은 자동으로 임포트를 분석하고 최적화합니다. 이 구성이 필요하지 않습니다.

### 4. Tailwind CSS 설정 확인

Tailwind CSS를 사용하는 경우 올바르게 설정되었는지 확인하세요.

일반적인 실수는 `node_modules`나 스캔하지 않아야 하는 다른 대규모 파일 디렉토리를 포함하는 방식으로 `content` 배열을 구성하는 것입니다.

Tailwind CSS 버전 3.4.8 이상에서는 빌드 속도를 늦출 수 있는 설정에 대해 경고합니다.

1. `tailwind.config.js`에서 스캔할 파일을 구체적으로 지정하세요:

   ```jsx
   module.exports = {
     content: [
       './src/**/*.{js,ts,jsx,tsx}', // 좋음
       // 이것은 너무 광범위할 수 있습니다
       // `packages/**/node_modules`도 매칭됩니다
       // '../../packages/**/*.{js,ts,jsx,tsx}',
     ],
   }
   ```

2. 불필요한 파일 스캔을 피하세요:

   ```jsx
   module.exports = {
     content: [
       // 더 좋음 - 'src' 폴더만 스캔
       '../../packages/ui/src/**/*.{js,ts,jsx,tsx}',
     ],
   }
   ```

### 5. 커스텀 webpack 설정 확인

커스텀 webpack 설정을 추가한 경우 컴파일 속도가 느려질 수 있습니다.

로컬 개발에 정말 필요한지 고려하세요. 선택적으로 프로덕션 빌드에만 특정 도구를 포함하거나, 기본 Turbopack 번들러를 사용하고 [로더](/app-router/api-reference/config/next-config-js/turbopack.md#configuring-webpack-loaders)를 구성하는 방법을 탐색할 수 있습니다.

### 6. 메모리 사용량 최적화

앱이 매우 큰 경우 더 많은 메모리가 필요할 수 있습니다.

[메모리 사용량 최적화에 대해 자세히 알아보기](/app-router/guides/memory-usage.md).

### 7. Server Components와 데이터 페칭

Server Components를 변경하면 새로운 변경 사항을 표시하기 위해 전체 페이지가 로컬에서 다시 렌더링되며, 여기에는 컴포넌트에 대한 새 데이터 페칭이 포함됩니다.

실험적 `serverComponentsHmrCache` 옵션을 사용하면 로컬 개발에서 Hot Module Replacement(HMR) 새로고침 시 Server Components의 `fetch` 응답을 캐시할 수 있습니다. 이를 통해 더 빠른 응답과 유료 API 호출 비용 절감이 가능합니다.

[실험적 옵션에 대해 자세히 알아보기](/app-router/api-reference/config/next-config-js/serverComponentsHmrCache.md).

### 8. Docker 대신 로컬 개발 고려

Mac이나 Windows에서 Docker를 개발에 사용하는 경우 Next.js를 로컬에서 실행하는 것보다 상당히 느린 성능을 경험할 수 있습니다.

Mac과 Windows에서 Docker의 파일시스템 접근으로 인해 Hot Module Replacement(HMR)가 몇 초 또는 심지어 몇 분이 걸릴 수 있지만, 동일한 애플리케이션은 로컬에서 개발할 때 빠른 HMR로 실행됩니다.

이 성능 차이는 Docker가 Linux 환경 외부에서 파일시스템 작업을 처리하는 방식 때문입니다. 최상의 개발 경험을 위해:

- 개발 중에는 Docker 대신 로컬 개발(`npm run dev` 또는 `pnpm dev`)을 사용하세요
- Docker는 프로덕션 배포와 프로덕션 빌드 테스트용으로 예약하세요
- 개발에 Docker를 반드시 사용해야 하는 경우 Linux 머신이나 VM에서 Docker를 사용하는 것을 고려하세요

프로덕션용 [Docker 배포에 대해 자세히 알아보기](/app-router/getting-started/deploying.md#docker).

## 문제를 찾는 도구

### 상세한 fetch 로깅

`next.config.js` 파일의 `logging.fetches` 옵션을 사용하여 개발 중 발생하는 상황에 대한 더 자세한 정보를 확인하세요:

```js
module.exports = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}
```

[fetch 로깅에 대해 자세히 알아보기](/app-router/api-reference/config/next-config-js/logging.md).

### Turbopack 트레이싱

Turbopack 트레이싱은 로컬 개발 중 애플리케이션의 성능을 이해하는 데 도움이 되는 도구입니다. 각 모듈이 컴파일되는 데 걸리는 시간과 모듈 간의 관계에 대한 자세한 정보를 제공합니다.

1. 최신 버전의 Next.js가 설치되어 있는지 확인합니다.

2. Turbopack 트레이스 파일을 생성합니다:

   ```bash
   NEXT_TURBOPACK_TRACING=1 npm run dev
   ```

3. 애플리케이션을 탐색하거나 파일을 편집하여 문제를 재현합니다.

4. Next.js 개발 서버를 중지합니다.

5. `.next/dev` 폴더에서 `trace-turbopack`이라는 파일을 사용할 수 있습니다.

6. `npx next internal trace [path-to-file]`을 사용하여 파일을 해석할 수 있습니다:

   ```bash
   npx next internal trace .next/dev/trace-turbopack
   ```

   `trace`를 사용할 수 없는 버전에서는 명령이 `turbo-trace-server`였습니다:

   ```bash
   npx next internal turbo-trace-server .next/dev/trace-turbopack
   ```

7. 트레이스 서버가 실행되면 https://trace.nextjs.org/ 에서 트레이스를 볼 수 있습니다.

8. 기본적으로 트레이스 뷰어는 타이밍을 집계합니다. 각 개별 시간을 보려면 뷰어 오른쪽 상단에서 "Aggregated in order"에서 "Spans in order"로 전환할 수 있습니다.

> **알아두면 좋은 점:** 트레이스 파일은 개발 서버 디렉토리 아래에 위치하며, 기본값은 `.next/dev`입니다. 이는 Next config 파일의 [`isolatedDevBuild`](/app-router/api-reference/config/next-config-js/isolatedDevBuild.md) 플래그를 사용하여 제어할 수 있습니다.

### 여전히 문제가 있나요?

[Turbopack 트레이싱](#turbopack-트레이싱) 섹션에서 생성된 트레이스 파일을 공유하고 [GitHub Discussions](https://github.com/vercel/next.js/discussions) 또는 [Discord](https://nextjs.org/discord)에서 공유하세요.
