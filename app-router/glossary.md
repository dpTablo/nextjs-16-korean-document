# Next.js 용어집 (Glossary)

## A

### App Router
Next.js 13에서 도입된 라우터로, React Server Components 위에 구축되었습니다. 파일 시스템 기반 라우팅을 사용하며 레이아웃, 중첩 라우팅, 로딩 상태, 에러 처리 등을 지원합니다.

## B

### Build time
애플리케이션이 컴파일되는 단계입니다. Next.js가 코드를 프로덕션용 최적화된 파일로 변환하고, 정적 페이지를 생성하며, 배포용 자산을 준비합니다.

## C

### Cache Components
`"use cache"` 지시문을 사용한 컴포넌트 및 함수 레벨 캐싱 기능입니다. 정적 HTML 셸을 즉시 제공하면서 동적 콘텐츠는 준비되면 스트리밍합니다.

### Catch-all Segments
`[...folder]/page.js` 구문을 사용한 동적 라우트 세그먼트로 여러 URL 부분과 매칭될 수 있습니다.

### Client Bundles
브라우저로 전송되는 JavaScript 번들입니다. Next.js가 모듈 그래프 기반으로 자동 분할합니다.

### Client Component
브라우저에서 실행되는 React 컴포넌트입니다. `"use client"` 지시문으로 표시되며, 상태, 이펙트, 이벤트 핸들러, 브라우저 API를 사용할 수 있습니다.

### Client-side navigation
전체 페이지 리로드 없이 페이지 콘텐츠가 동적으로 업데이트되는 네비게이션 방식입니다. `<Link>` 컴포넌트를 사용합니다.

### Code Splitting
애플리케이션을 더 작은 JavaScript 청크로 분할하는 프로세스입니다. Next.js가 라우트 기반으로 자동 수행합니다.

## D

### Dynamic rendering
[Request-time rendering](#request-time-rendering)을 참조하세요.

### Dynamic route segments
데이터로부터 요청 시간에 생성되는 라우트 세그먼트입니다. `[slug]` 형태로 생성됩니다.

## E

### Environment Variables
빌드 시간 또는 요청 시간에 접근 가능한 구성 값입니다. `NEXT_PUBLIC_` 접두사가 있는 변수는 브라우저에 노출됩니다.

### Error Boundary
자식 컴포넌트 트리에서 JavaScript 에러를 캐치하고 폴백 UI를 표시하는 React 컴포넌트입니다. `error.js` 파일로 구현됩니다.

## F

### Font Optimization
`next/font`를 사용한 자동 폰트 최적화입니다. Next.js가 폰트를 자체 호스팅하고 레이아웃 시프트를 제거합니다.

### File-system caching
컴파일러 아티팩트를 디스크에 저장하는 Turbopack 기능으로 컴파일 시간을 단축합니다.

## H

### Hydration
React가 DOM에 이벤트 핸들러를 부착하여 서버 렌더링 HTML을 인터랙티브하게 만드는 과정입니다.

## I

### Import Aliases
자주 사용되는 디렉토리에 대한 커스텀 경로 매핑으로 상대 경로를 간단하게 만듭니다.

### Incremental Static Regeneration (ISR)
전체 사이트를 재구축하지 않고 정적 콘텐츠를 업데이트하는 기술입니다. Next.js에서는 Revalidation이라고도 불립니다.

### Intercepting Routes
애플리케이션의 다른 부분에서 현재 레이아웃 내에 라우트를 로드하는 라우팅 패턴입니다. 모달 같은 콘텐츠 표시에 유용합니다.

### Image Optimization
`<Image>` 컴포넌트를 사용한 자동 이미지 최적화입니다. WebP 같은 현대 형식으로 제공됩니다.

## L

### Layout
여러 페이지 간에 공유되는 UI입니다. 상태를 유지하고 인터랙티브하며 네비게이션 시 다시 렌더링되지 않습니다. `layout.js` 파일에서 정의됩니다.

### Loading UI
라우트 세그먼트 로딩 중 표시되는 폴백 UI입니다. `loading.js` 파일로 생성하며 Suspense 경계로 자동 래핑됩니다.

## M

### Module Graph
앱의 파일 의존성 그래프입니다. 각 파일은 노드, import/export 관계는 엣지를 형성합니다.

### Metadata
페이지에 대한 정보(제목, 설명, Open Graph 이미지 등)입니다. `metadata` 내보내기 또는 `generateMetadata` 함수로 정의됩니다.

### Memoization
함수 반환 값을 캐싱하여 같은 함수를 여러 번 호출해도 한 번만 실행되게 합니다. Next.js가 같은 URL의 fetch 요청을 자동으로 메모이제이션합니다.

### Middleware
[Proxy](#proxy)를 참조하세요.

## N

### Not Found
라우트가 존재하지 않을 때 표시되는 특수 컴포넌트입니다. `not-found.js` 파일로 생성됩니다.

## P

### Private Folders
언더스코어로 시작하는 폴더(예: `_components`)입니다. 라우팅 시스템에서 제외됩니다.

### Page
라우트에 고유한 UI입니다. `page.js` 파일에서 React 컴포넌트로 정의됩니다.

### Parallel Routes
하나의 레이아웃 내에서 여러 페이지를 동시에 또는 조건부로 렌더링하는 패턴입니다. `@folder` 규칙을 사용합니다.

### Partial Prerendering (PPR)
단일 라우트에서 정적과 동적 렌더링을 결합한 최적화입니다. 정적 셸은 즉시 제공하고 동적 콘텐츠는 스트리밍합니다.

### Prefetching
사용자가 네비게이션하기 전에 백그라운드에서 라우트를 로드합니다. `<Link>` 컴포넌트가 자동으로 뷰포트에 들어온 링크를 프리페칭합니다.

### Prerendering
[Build time](#build-time) 또는 [revalidation](#revalidation) 중에 컴포넌트를 렌더링하는 것입니다. HTML과 RSC Payload가 결과물로 CDN에서 제공 가능합니다.

### Proxy
요청이 완료되기 전에 서버에서 코드를 실행하는 파일(`proxy.js`)입니다. 로깅, 리다이렉트, 재작성 같은 서버 측 로직을 구현합니다. 이전 이름은 Middleware입니다.

## R

### Redirect
사용자를 한 URL에서 다른 URL로 보내는 것입니다. `next.config.js`, Proxy, 또는 `redirect()` 함수로 구현됩니다.

### Request time
사용자가 애플리케이션에 요청을 할 때입니다. 동적 라우트가 렌더링되고 쿠키, 헤더에 접근 가능합니다.

### Request-time APIs
요청 특정 데이터에 접근하여 컴포넌트가 [request-time rendering](#request-time-rendering)으로 옵트인되게 하는 API입니다:
- `cookies()` - 요청 쿠키 접근
- `headers()` - 요청 헤더 접근
- `searchParams` - URL 쿼리 파라미터 접근
- `draftMode()` - 드래프트 모드 활성화/확인

### Request-time rendering
[Build time](#build-time) 대신 [request time](#request-time)에 컴포넌트를 렌더링하는 것입니다. [Request-time APIs](#request-time-apis)를 사용하면 동적이 됩니다.

### Revalidation
캐시된 데이터를 업데이트하는 프로세스입니다. 시간 기반(`cacheLife()` 사용) 또는 온디맨드(`cacheTag()` 태깅, `updateTag()`로 무효화)로 수행됩니다.

### Rewrite
URL을 브라우저에서 변경하지 않고 들어온 요청 경로를 다른 대상 경로에 매핑합니다. `next.config.js` 또는 Proxy에서 구성됩니다.

### Route Groups
URL 구조에 영향을 주지 않으면서 라우트를 정리하는 방법입니다. 괄호로 폴더명을 감싸서 생성합니다(예: `(marketing)`).

### Route Handler
특정 라우트의 HTTP 요청을 처리하는 함수입니다. `route.js` 파일에서 정의하며 Web Request/Response API를 사용합니다.

### Route Segment
URL 경로의 일부(두 슬래시 사이)입니다. `app` 디렉토리의 각 폴더가 세그먼트를 나타냅니다.

### RSC Payload
React Server Component Payload입니다. 렌더링된 서버 컴포넌트 트리의 컴팩트한 바이너리 표현입니다.

## S

### Server Component
App Router의 기본 컴포넌트 타입입니다. 서버에서 렌더링되고 직접 데이터 페칭이 가능하며 클라이언트 JavaScript 번들에 추가되지 않습니다.

### Server Action
클라이언트 컴포넌트에 props로 전달되거나 폼 액션에 바인딩되는 [Server Function](#server-function)입니다. 폼 제출과 데이터 뮤테이션에 자주 사용됩니다.

### Server Function
`"use server"` 지시문으로 표시된 서버에서 실행되는 비동기 함수입니다. 클라이언트 컴포넌트에서 호출 가능합니다.

### Static Export
`output: 'export'`를 `next.config.js`에 설정하여 생성되는 완전 정적 사이트입니다. Node.js 서버 없이 정적 파일 서버에서 호스팅 가능합니다.

### Static rendering
[Prerendering](#prerendering)을 참조하세요.

### Static Assets
이미지, 폰트, 비디오 등 처리 없이 직접 제공되는 파일입니다. 보통 `public` 디렉토리에 저장됩니다.

### Static Shell
페이지의 사전 렌더링된 HTML 구조로 즉시 브라우저에 제공됩니다. [Partial Prerendering](#partial-prerendering-ppr)에서 정적 셸은 렌더링 가능한 모든 콘텐츠와 나중에 스트리밍될 동적 콘텐츠의 Suspense 경계 폴백을 포함합니다.

### Streaming
전체 페이지 렌더링을 기다리지 않고 준비되는 대로 서버가 페이지 부분을 클라이언트로 전송하는 기술입니다. `loading.js` 또는 `<Suspense>` 경계로 자동 활성화됩니다.

### Suspense boundary
비동기 콘텐츠를 래핑하고 로드 중에 폴백 UI를 표시하는 React `<Suspense>` 컴포넌트입니다. Next.js에서 [Static Shell](#static-shell)이 끝나고 [Streaming](#streaming)이 시작되는 곳을 정의합니다.

## T

### Turbopack
Next.js용으로 구축된 빠른 Rust 기반 번들러입니다. `next dev`의 기본 번들러이며 `next build`에서도 사용 가능합니다. Webpack보다 훨씬 빠른 컴파일 시간을 제공합니다.

### Tree Shaking
빌드 과정에서 JavaScript 번들에서 사용되지 않은 코드를 제거하는 프로세스입니다. Next.js가 자동으로 번들 크기를 감소시킵니다.

## U

### `"use cache"` Directive
컴포넌트나 함수를 캐시 가능으로 표시하는 지시문입니다. 파일 최상단에 배치하여 모든 내보내기를 캐시 가능하게 하거나, 함수/컴포넌트 최상단에 인라인으로 배치하여 특정 범위만 캐시 가능하게 합니다.

### `"use client"` Directive
서버 코드와 클라이언트 코드 간의 경계를 표시하는 특수 React 지시문입니다. 파일 최상단, 모든 import 및 다른 코드보다 먼저 배치되어야 합니다.

### `"use server"` Directive
클라이언트 측 코드에서 호출 가능한 [Server Function](#server-function)을 표시하는 지시문입니다. 파일 최상단에 배치하거나 함수 최상단에 인라인으로 배치합니다.

---

**버전 정보**
- Next.js 버전: 16.1.6
- 문서 업데이트: 2026-01-29
