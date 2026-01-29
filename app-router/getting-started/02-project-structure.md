---
원문: https://nextjs.org/docs/app/getting-started/project-structure
버전: 16.1.6
---

# Next.js 프로젝트 구조 및 구성

## 폴더 및 파일 규칙

### 최상위 폴더

| 폴더 | 용도 |
|--------|---------|
| `app` | App Router |
| `pages` | Pages Router |
| `public` | 제공될 정적 에셋 |
| `src` | 선택적 애플리케이션 소스 폴더 |

### 최상위 파일

**Next.js 설정:**
- `next.config.js` - Next.js 설정 파일
- `package.json` - 프로젝트 의존성 및 스크립트
- `instrumentation.ts` - OpenTelemetry 및 계측 파일
- `proxy.ts` - Next.js 요청 프록시
- `eslint.config.mjs` - ESLint 설정
- `.gitignore` - Git에서 무시할 파일 및 폴더
- `tsconfig.json` - TypeScript 설정
- `jsconfig.json` - JavaScript 설정

**환경 변수:**
- `.env` - 환경 변수 (버전 관리에서 제외)
- `.env.local` - 로컬 환경 변수
- `.env.production` - 프로덕션 환경 변수
- `.env.development` - 개발 환경 변수

**TypeScript:**
- `next-env.d.ts` - TypeScript 선언 파일 (버전 관리에서 제외)

### 라우팅 파일

| 파일 | 확장자 | 용도 |
|------|-----------|---------|
| `layout` | `.js` `.jsx` `.tsx` | 레이아웃 |
| `page` | `.js` `.jsx` `.tsx` | 페이지 |
| `loading` | `.js` `.jsx` `.tsx` | 로딩 UI |
| `not-found` | `.js` `.jsx` `.tsx` | 404 UI |
| `error` | `.js` `.jsx` `.tsx` | 에러 UI |
| `global-error` | `.js` `.jsx` `.tsx` | 전역 에러 UI |
| `route` | `.js` `.ts` | API 엔드포인트 |
| `template` | `.js` `.jsx` `.tsx` | 재렌더링되는 레이아웃 |
| `default` | `.js` `.jsx` `.tsx` | 병렬 라우트 폴백 페이지 |

### 중첩 라우트 예시

| 경로 | URL 패턴 | 비고 |
|------|-----------|-------|
| `app/layout.tsx` | — | 루트 레이아웃이 모든 라우트를 감쌈 |
| `app/blog/layout.tsx` | — | `/blog` 및 하위 라우트를 감쌈 |
| `app/page.tsx` | `/` | 공개 라우트 |
| `app/blog/page.tsx` | `/blog` | 공개 라우트 |
| `app/blog/authors/page.tsx` | `/blog/authors` | 공개 라우트 |

### 동적 라우트

대괄호를 사용하여 세그먼트를 매개변수화:
- `[segment]` - 단일 매개변수
- `[...segment]` - 캐치올 라우트
- `[[...segment]]` - 선택적 캐치올 라우트

| 경로 | URL 패턴 |
|------|------------|
| `app/blog/[slug]/page.tsx` | `/blog/my-first-post` |
| `app/shop/[...slug]/page.tsx` | `/shop/clothing`, `/shop/clothing/shirts` |
| `app/docs/[[...slug]]/page.tsx` | `/docs`, `/docs/layouts-and-pages`, `/docs/api-reference/use-router` |

### 라우트 그룹 및 프라이빗 폴더

| 경로 | URL 패턴 | 비고 |
|------|-----------|-------|
| `app/(marketing)/page.tsx` | `/` | 그룹이 URL에서 제외됨 |
| `app/(shop)/cart/page.tsx` | `/cart` | `(shop)` 내에서 레이아웃 공유 |
| `app/blog/_components/Post.tsx` | — | 라우팅 불가; UI 유틸리티를 위한 안전한 공간 |
| `app/blog/_lib/data.ts` | — | 라우팅 불가; 유틸리티를 위한 안전한 공간 |

### 병렬 및 인터셉팅 라우트

| 패턴 | 의미 | 일반적인 사용 사례 |
|---------|---------|-----------------|
| `@slot` | 명명된 슬롯 | 사이드바 + 메인 콘텐츠 |
| `(.)folder` | 같은 레벨 인터셉트 | 형제 라우트를 모달로 미리보기 |
| `(..)folder` | 부모 인터셉트 | 부모의 자식을 오버레이로 열기 |
| `(..)(..)folder` | 두 레벨 인터셉트 | 깊이 중첩된 오버레이 |
| `(...)folder` | 루트에서 인터셉트 | 현재 뷰에서 임의 라우트 표시 |

### 메타데이터 파일 규칙

**앱 아이콘:**
- `favicon.ico` - 파비콘 파일
- `icon` - `.ico` `.jpg` `.jpeg` `.png` `.svg`
- `icon` - `.js` `.ts` `.tsx` (생성된 앱 아이콘)
- `apple-icon` - `.jpg` `.jpeg` `.png`
- `apple-icon` - `.js` `.ts` `.tsx` (생성된 Apple 앱 아이콘)

**Open Graph 및 Twitter 이미지:**
- `opengraph-image` - `.jpg` `.jpeg` `.png` `.gif`
- `opengraph-image` - `.js` `.ts` `.tsx` (생성됨)
- `twitter-image` - `.jpg` `.jpeg` `.png` `.gif`
- `twitter-image` - `.js` `.ts` `.tsx` (생성됨)

**SEO:**
- `sitemap.xml` - 사이트맵 파일
- `sitemap` - `.js` `.ts` (생성된 사이트맵)
- `robots.txt` - Robots 파일
- `robots` - `.js` `.ts` (생성된 Robots 파일)

## 프로젝트 구성하기

### 컴포넌트 계층

컴포넌트는 다음 순서로 렌더링됩니다:
1. `layout.js`
2. `template.js`
3. `error.js` (React 에러 경계)
4. `loading.js` (React Suspense 경계)
5. `not-found.js` ("not found" UI를 위한 React 에러 경계)
6. `page.js` 또는 중첩된 `layout.js`

컴포넌트는 중첩 라우트에서 재귀적으로 렌더링되며, 라우트 세그먼트 컴포넌트가 부모 세그먼트의 컴포넌트 **내부에** 중첩됩니다.

### 코로케이션 (Colocation)

**핵심 사항:**
- `app` 디렉토리의 중첩 폴더는 라우트 구조를 정의합니다
- `page.js` 또는 `route.js` 파일이 추가되기 전까지 라우트는 **공개적으로 접근 불가**합니다
- `page.js` 또는 `route.js`에서 반환된 콘텐츠만 클라이언트로 전송됩니다
- 프로젝트 파일은 실수로 라우팅되지 않고 라우트 세그먼트 내에 **안전하게 배치**될 수 있습니다

### 프라이빗 폴더

언더스코어로 접두사를 붙여 프라이빗 폴더를 생성: `_folderName`

**이점:**
- UI 로직을 라우팅 로직에서 분리
- 프로젝트 전반에 걸쳐 일관되게 내부 파일 구성
- 코드 에디터에서 파일 정렬 및 그룹화
- 미래 Next.js 규칙과의 잠재적 이름 충돌 방지

**참고:** `%5F` (URL 인코딩된 언더스코어)를 사용하여 언더스코어로 시작하는 URL 세그먼트를 생성할 수 있습니다: `%5FfolderName`

### 라우트 그룹

폴더 이름을 괄호로 감싸서 라우트 그룹을 생성: `(folderName)`

**이점:**
- 사이트 섹션, 의도 또는 팀별로 라우트 구성 (예: 마케팅 페이지, 관리자 페이지)
- 동일한 라우트 세그먼트 레벨에서 중첩 레이아웃 활성화
- 여러 루트 레이아웃 생성
- 특정 세그먼트를 레이아웃에 선택적으로 포함

### `src` 폴더

Next.js는 애플리케이션 코드(`app` 포함)를 선택적 `src` 폴더에 저장하여 애플리케이션 코드를 프로젝트 설정 파일과 분리할 수 있도록 지원합니다.

## 구성 전략

### 1. `app` 외부에 프로젝트 파일 저장

모든 애플리케이션 코드를 프로젝트 **루트**의 공유 폴더에 저장하고 `app` 디렉토리를 순수하게 라우팅 용도로만 유지합니다.

**폴더 구조:**
```
project-root/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── [routes]
├── components/
├── lib/
└── [설정 파일]
```

### 2. `app` 내부의 최상위 폴더에 프로젝트 파일 저장

모든 애플리케이션 코드를 `app` 디렉토리 **루트**의 공유 폴더에 저장합니다.

**폴더 구조:**
```
project-root/
├── app/
│   ├── components/
│   ├── lib/
│   ├── layout.tsx
│   ├── page.tsx
│   └── [routes]
```

### 3. 기능 또는 라우트별로 프로젝트 파일 분할

전역적으로 공유되는 애플리케이션 코드는 루트 `app` 디렉토리에 저장하고, 더 구체적인 애플리케이션 코드는 이를 사용하는 라우트 세그먼트로 **분할**합니다.

**폴더 구조:**
```
project-root/
├── app/
│   ├── components/
│   ├── lib/
│   ├── (marketing)/
│   │   ├── components/
│   │   └── [routes]
│   └── (shop)/
│       ├── components/
│       └── [routes]
```

### 4. URL 경로에 영향을 주지 않고 라우트 구성

라우트 그룹을 사용하여 URL에 영향을 주지 않고 관련 라우트를 함께 유지합니다.

**예시:**
```
app/
├── (marketing)/
│   ├── layout.tsx
│   ├── page.tsx
│   └── about/page.tsx
├── (shop)/
│   ├── layout.tsx
│   ├── cart/page.tsx
│   └── account/page.tsx
```

URL은 `/`, `/about`, `/cart`, `/account`가 됩니다 (라우트 그룹은 제외됨).

### 5. 특정 세그먼트를 레이아웃에 포함

라우트 그룹을 생성하고 동일한 레이아웃을 공유하는 라우트를 그 안으로 이동합니다.

**예시:**
```
app/
├── (shop)/
│   ├── layout.tsx
│   ├── account/page.tsx
│   └── cart/page.tsx
└── checkout/page.tsx
```

`(shop)` 내부의 라우트는 레이아웃을 공유하지만 `checkout`은 공유하지 않습니다.

### 6. 여러 루트 레이아웃 생성

최상위 `layout.js`를 제거하고 각 라우트 그룹 내부에 `layout.js`를 추가합니다. 각 루트 레이아웃에는 `<html>` 및 `<body>` 태그가 필요합니다.

**예시:**
```
app/
├── (marketing)/
│   ├── layout.tsx
│   ├── page.tsx
│   └── about/page.tsx
├── (shop)/
│   ├── layout.tsx
│   ├── cart/page.tsx
│   └── account/page.tsx
```

`(marketing)`과 `(shop)` 모두 별도의 UI/UX를 가진 자체 루트 레이아웃을 가집니다.

---

**핵심 요점:** 팀에 적합한 전략을 선택하고 프로젝트 전반에 걸쳐 일관성을 유지하세요. Next.js는 구성에 대해 의견을 강요하지 않지만 효과적인 구성을 돕는 기능을 제공합니다.
