# 코드모드

**문서 버전:** 16.1.2
**최종 업데이트:** 2025-12-10

---

## 개요

코드모드는 코드베이스에서 프로그래밍 방식으로 실행되는 변환입니다. 이를 통해 모든 파일을 수동으로 거치지 않고도 많은 수의 변경 사항을 프로그래밍 방식으로 적용할 수 있습니다.

Next.js는 API가 업데이트되거나 더 이상 사용되지 않을 때 Next.js 코드베이스를 업그레이드하는 데 도움이 되는 코드모드 변환을 제공합니다.

---

## 사용법

터미널에서 프로젝트 폴더로 이동(`cd`)한 후 다음을 실행합니다:

```bash
npx @next/codemod <transform> <path>
```

### 매개변수
- `transform` - 변환 이름
- `path` - 변환할 파일 또는 디렉토리
- `--dry` - 드라이런 수행, 코드가 편집되지 않음
- `--print` - 비교를 위해 변경된 출력 인쇄

---

## 업그레이드

Next.js 애플리케이션을 업그레이드하고, 코드모드를 자동으로 실행하며 Next.js, React 및 React DOM을 업데이트합니다.

```bash
npx @next/codemod upgrade [revision]
```

### 옵션

- `revision` (선택): 업그레이드 유형 지정 (`patch`, `minor`, `major`), NPM dist 태그 (예: `latest`, `canary`, `rc`), 또는 정확한 버전 (예: `15.0.0`). 안정 버전의 기본값은 `minor`.
- `--verbose`: 업그레이드 과정에서 더 자세한 출력 표시.

### 예제

```bash
# 최신 패치로 업그레이드 (예: 16.0.7 -> 16.0.8)
npx @next/codemod upgrade patch

# 최신 마이너로 업그레이드 (예: 15.3.7 -> 15.4.8). 기본값입니다.
npx @next/codemod upgrade minor

# 최신 메이저로 업그레이드 (예: 15.5.7 -> 16.0.7)
npx @next/codemod upgrade major

# 특정 버전으로 업그레이드
npx @next/codemod upgrade 16

# canary 릴리스로 업그레이드
npx @next/codemod upgrade canary
```

> **알아두면 좋은 점:**
> - 대상 버전이 현재 버전과 같거나 낮으면 변경 없이 명령이 종료됩니다.
> - 업그레이드 중에 어떤 Next.js 코드모드를 적용할지 선택하라는 메시지가 표시되고, React를 업그레이드하는 경우 React 19 코드모드를 실행할 수 있습니다.

---

## 코드모드

### 버전 16.0

#### App Router 페이지와 레이아웃에서 `experimental_ppr` Route Segment Config 제거

**코드모드 이름:** `remove-experimental-ppr`

```bash
npx @next/codemod@latest remove-experimental-ppr .
```

이 코드모드는 App Router 페이지와 레이아웃에서 `experimental_ppr` Route Segment Config를 제거합니다.

```diff
- export const experimental_ppr = true;
```

---

#### 안정화된 API에서 `unstable_` 접두사 제거

**코드모드 이름:** `remove-unstable-prefix`

```bash
npx @next/codemod@latest remove-unstable-prefix .
```

이 코드모드는 안정화된 API에서 `unstable_` 접두사를 제거합니다.

**예제:**

```ts
import { unstable_cacheTag as cacheTag } from 'next/cache'

cacheTag()
```

다음으로 변환됩니다:

```ts
import { cacheTag } from 'next/cache'

cacheTag()
```

---

#### 더 이상 사용되지 않는 `middleware` 규칙에서 `proxy`로 마이그레이션

**코드모드 이름:** `middleware-to-proxy`

```bash
npx @next/codemod@latest middleware-to-proxy .
```

이 코드모드는 더 이상 사용되지 않는 `middleware` 규칙에서 `proxy` 규칙으로 프로젝트를 마이그레이션합니다:

- `middleware.<extension>`을 `proxy.<extension>`으로 이름 변경 (예: `middleware.ts`를 `proxy.ts`로)
- 명명된 내보내기 `middleware`를 `proxy`로 이름 변경
- Next.js 구성 속성 `experimental.middlewarePrefetch`를 `experimental.proxyPrefetch`로 이름 변경
- Next.js 구성 속성 `skipMiddlewareUrlNormalize`를 `skipProxyUrlNormalize`로 이름 변경

---

#### `next lint`에서 ESLint CLI로 마이그레이션

**코드모드 이름:** `next-lint-to-eslint-cli`

```bash
npx @next/codemod@canary next-lint-to-eslint-cli .
```

이 코드모드는 `next lint`에서 로컬 ESLint 구성을 사용하는 ESLint CLI로 프로젝트를 마이그레이션합니다:

- Next.js 권장 구성이 포함된 `eslint.config.mjs` 파일 생성
- `package.json` 스크립트를 `next lint` 대신 `eslint .`를 사용하도록 업데이트
- `package.json`에 필요한 ESLint 의존성 추가
- 기존 ESLint 구성이 있는 경우 보존

---

### 버전 15.0

#### App Router Route Segment Config `runtime` 값을 `experimental-edge`에서 `edge`로 변환

**코드모드 이름:** `app-dir-runtime-config-experimental-edge`

```bash
npx @next/codemod@latest app-dir-runtime-config-experimental-edge .
```

이 코드모드는 Route Segment Config `runtime` 값 `experimental-edge`를 `edge`로 변환합니다.

---

#### 비동기 Dynamic API로 마이그레이션

**코드모드 이름:** `next-async-request-api`

```bash
npx @next/codemod@latest next-async-request-api .
```

이 코드모드는 이제 비동기인 동적 API(`cookies()`, `headers()` 및 `next/headers`의 `draftMode()`)를 적절히 await하거나 해당하는 경우 `React.use()`로 래핑하도록 변환합니다.

---

#### `NextRequest`의 `geo` 및 `ip` 속성을 `@vercel/functions`로 대체

**코드모드 이름:** `next-request-geo-ip`

```bash
npx @next/codemod@latest next-request-geo-ip .
```

이 코드모드는 `@vercel/functions`를 설치하고 `NextRequest`의 `geo` 및 `ip` 속성을 해당하는 `@vercel/functions` 기능으로 변환합니다.

---

### 버전 14.0

#### `ImageResponse` 임포트 마이그레이션

**코드모드 이름:** `next-og-import`

```bash
npx @next/codemod@latest next-og-import .
```

이 코드모드는 Dynamic OG Image Generation 사용을 위해 `next/server`에서 `next/og`로 임포트를 이동합니다.

---

#### `viewport` 내보내기 사용

**코드모드 이름:** `metadata-to-viewport-export`

```bash
npx @next/codemod@latest metadata-to-viewport-export .
```

이 코드모드는 특정 뷰포트 메타데이터를 `viewport` 내보내기로 마이그레이션합니다.

---

### 버전 13.2

#### 내장 폰트 사용

**코드모드 이름:** `built-in-next-font`

```bash
npx @next/codemod@latest built-in-next-font .
```

이 코드모드는 `@next/font` 패키지를 제거하고 `@next/font` 임포트를 내장 `next/font`로 변환합니다.

---

### 버전 13.0

#### Next Image 임포트 이름 변경

**코드모드 이름:** `next-image-to-legacy-image`

```bash
npx @next/codemod@latest next-image-to-legacy-image .
```

기존 Next.js 10, 11 또는 12 애플리케이션의 `next/image` 임포트를 Next.js 13의 `next/legacy/image`로 안전하게 이름을 변경합니다. 또한 `next/future/image`를 `next/image`로 이름을 변경합니다.

---

#### 새 Image 컴포넌트로 마이그레이션

**코드모드 이름:** `next-image-experimental`

```bash
npx @next/codemod@latest next-image-experimental .
```

인라인 스타일을 추가하고 사용되지 않는 props를 제거하여 `next/legacy/image`에서 새 `next/image`로 위험하게 마이그레이션합니다.

---

#### Link 컴포넌트에서 `<a>` 태그 제거

**코드모드 이름:** `new-link`

```bash
npx @next/codemod@latest new-link .
```

Link 컴포넌트 내부의 `<a>` 태그를 제거합니다.

---

### 버전 11

#### CRA에서 마이그레이션

**코드모드 이름:** `cra-to-next`

```bash
npx @next/codemod cra-to-next
```

Create React App 프로젝트를 Next.js로 마이그레이션합니다.

---

### 버전 10

#### React 임포트 추가

**코드모드 이름:** `add-missing-react-import`

```bash
npx @next/codemod add-missing-react-import
```

새로운 React JSX 변환이 작동하도록 `React`를 임포트하지 않는 파일을 변환하여 임포트를 포함시킵니다.

---

### 버전 9

#### 익명 컴포넌트를 명명된 컴포넌트로 변환

**코드모드 이름:** `name-default-component`

```bash
npx @next/codemod name-default-component
```

Fast Refresh와 함께 작동하도록 익명 컴포넌트를 명명된 컴포넌트로 변환합니다.

---

### 버전 6

#### `withRouter` 사용

**코드모드 이름:** `url-to-withrouter`

```bash
npx @next/codemod url-to-withrouter
```

더 이상 사용되지 않는 자동 주입 `url` 속성을 `withRouter`와 그것이 주입하는 `router` 속성을 사용하도록 변환합니다.
