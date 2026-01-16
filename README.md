# Next.js 16.1.1 한국어 문서

Next.js 공식 문서의 한국어 번역입니다.

## 📚 문서 구조

### ✅ 완료된 섹션

#### App Router - Getting Started (12개 문서)
1. [설치 (Installation)](./app-router/getting-started/01-installation.md)
2. [프로젝트 구조 (Project Structure)](./app-router/getting-started/02-project-structure.md)
3. [레이아웃과 페이지 (Layouts and Pages)](./app-router/getting-started/03-layouts-and-pages.md)
4. [링크 및 네비게이션 (Linking and Navigating)](./app-router/getting-started/04-linking-and-navigating.md)
5. [서버 및 클라이언트 컴포넌트 (Server and Client Components)](./app-router/getting-started/05-server-and-client-components.md)
6. [캐싱 및 재검증 (Caching and Revalidating)](./app-router/getting-started/06-caching-and-revalidating.md)
7. [에러 처리 (Error Handling)](./app-router/getting-started/07-error-handling.md)
8. [CSS 스타일링 (CSS)](./app-router/getting-started/08-css.md)
9. [이미지 최적화 (Image Optimization)](./app-router/getting-started/09-image-optimization.md)
10. [폰트 최적화 (Font Optimization)](./app-router/getting-started/10-font-optimization.md)
11. [Route Handlers](./app-router/getting-started/11-route-handlers.md)
12. [배포 (Deploying)](./app-router/getting-started/12-deploying.md)

#### App Router - Guides (25개 핵심 문서)
1. [Forms (폼 처리)](./app-router/guides/forms.md)
2. [Authentication (인증)](./app-router/guides/authentication.md)
3. [Environment Variables (환경 변수)](./app-router/guides/environment-variables.md)
4. [Testing - Jest](./app-router/guides/testing-jest.md)
5. [Testing - Playwright](./app-router/guides/testing-playwright.md)
6. [Internationalization (국제화)](./app-router/guides/internationalization.md)
7. [Caching (캐싱)](./app-router/guides/caching.md)
8. [Self-Hosting (셀프 호스팅)](./app-router/guides/self-hosting.md)
9. [Static Exports (정적 내보내기)](./app-router/guides/static-exports.md)
10. [Debugging (디버깅)](./app-router/guides/debugging.md)
11. [Data Fetching Patterns (데이터 페칭 패턴)](./app-router/guides/data-fetching-patterns.md)
12. [Server Actions Patterns (Server Actions 패턴)](./app-router/guides/server-actions-patterns.md)
13. [Rendering (렌더링 전략)](./app-router/guides/rendering.md)
14. [Tailwind CSS v3 (Tailwind 스타일링)](./app-router/guides/tailwind-css.md)
15. [Analytics (분석)](./app-router/guides/analytics.md)
16. [Lazy Loading (지연 로딩)](./app-router/guides/lazy-loading.md)
17. [MDX (마크다운 + JSX)](./app-router/guides/mdx.md)
18. [Third Party Libraries (서드파티 라이브러리)](./app-router/guides/third-party-libraries.md)
19. [Prefetching (프리페칭)](./app-router/guides/prefetching.md)
20. [TypeScript (타입스크립트)](./app-router/guides/typescript.md)
21. [Content Security Policy (보안 정책)](./app-router/guides/content-security-policy.md)
22. [Draft Mode (초안 모드)](./app-router/guides/draft-mode.md) 🆕
23. [OpenTelemetry (관찰성)](./app-router/guides/opentelemetry.md) 🆕
24. [CSS-in-JS (CSS-in-JS 라이브러리)](./app-router/guides/css-in-js.md) 🆕
25. [Sass (Sass 스타일링)](./app-router/guides/sass.md) 🆕

#### App Router - API Reference (54개 핵심 문서)

**지시어 (2개):**
1. [use client 지시어](./app-router/api-reference/use-client.md)
2. [use server 지시어](./app-router/api-reference/use-server.md)

**컴포넌트 (4개):**
3. [Link 컴포넌트](./app-router/api-reference/link.md)
4. [Image 컴포넌트](./app-router/api-reference/image.md)
5. [Script 컴포넌트](./app-router/api-reference/components/script.md) - 스크립트 최적화
6. [Font 최적화](./app-router/api-reference/components/font.md) - 폰트 최적화

**함수 (25개):**
7. [generateMetadata](./app-router/api-reference/functions/generateMetadata.md) - 동적 메타데이터 생성
8. [generateStaticParams](./app-router/api-reference/functions/generateStaticParams.md) - 정적 경로 생성
9. [redirect](./app-router/api-reference/functions/redirect.md) - 리디렉션
10. [notFound](./app-router/api-reference/functions/notFound.md) - 404 페이지
11. [cookies](./app-router/api-reference/functions/cookies.md) - 쿠키 처리
12. [headers](./app-router/api-reference/functions/headers.md) - 헤더 처리
13. [revalidatePath](./app-router/api-reference/functions/revalidatePath.md) - 경로 재검증
14. [revalidateTag](./app-router/api-reference/functions/revalidateTag.md) - 태그 재검증
15. [useRouter](./app-router/api-reference/functions/useRouter.md) - 클라이언트 라우팅
16. [usePathname](./app-router/api-reference/functions/usePathname.md) - 현재 경로
17. [useSearchParams](./app-router/api-reference/functions/useSearchParams.md) - 쿼리 파라미터
18. [useParams](./app-router/api-reference/functions/useParams.md) - 동적 파라미터
19. [useSelectedLayoutSegment](./app-router/api-reference/functions/useSelectedLayoutSegment.md) - 활성 세그먼트 🆕
20. [permanentRedirect](./app-router/api-reference/functions/permanentRedirect.md) - 영구 리디렉션
21. [draftMode](./app-router/api-reference/functions/draftMode.md) - 초안 모드
22. [ImageResponse](./app-router/api-reference/functions/ImageResponse.md) - OG 이미지 생성 🆕
23. [generateSitemaps](./app-router/api-reference/functions/generateSitemaps.md) - 사이트맵 생성 🆕

**파일 규칙 (17개):**
24. [page.js](./app-router/api-reference/file-conventions/page.md) - 페이지 파일
25. [layout.js](./app-router/api-reference/file-conventions/layout.md) - 레이아웃 파일
26. [template.js](./app-router/api-reference/file-conventions/template.md) - 템플릿 파일
27. [loading.js](./app-router/api-reference/file-conventions/loading.md) - 로딩 UI
28. [error.js](./app-router/api-reference/file-conventions/error.md) - 에러 처리
29. [not-found.js](./app-router/api-reference/file-conventions/not-found.md) - 404 페이지
30. [route.js](./app-router/api-reference/file-conventions/route.md) - API 라우트
31. [middleware.js](./app-router/api-reference/file-conventions/middleware.md) - 미들웨어
32. [default.js](./app-router/api-reference/file-conventions/default.md) - Parallel Routes 폴백
33. [global-error.js](./app-router/api-reference/file-conventions/global-error.md) - 전역 오류
34. [instrumentation.js](./app-router/api-reference/file-conventions/instrumentation.md) - 관찰성/모니터링
35. [sitemap.xml](./app-router/api-reference/file-conventions/metadata/sitemap.md) - 사이트맵
36. [robots.txt](./app-router/api-reference/file-conventions/metadata/robots.md) - robots.txt
37. [opengraph-image](./app-router/api-reference/file-conventions/metadata/opengraph-image.md) - OG/Twitter 이미지
38. [app-icons](./app-router/api-reference/file-conventions/metadata/app-icons.md) - favicon, icon, apple-icon
39. [manifest.json](./app-router/api-reference/file-conventions/metadata/manifest.md) - PWA Manifest
40. [route-segment-config](./app-router/api-reference/file-conventions/route-segment-config.md) - 라우트 세그먼트 설정 🆕

**추가 함수 (14개):**
41. [fetch](./app-router/api-reference/functions/fetch.md) - Next.js 확장 fetch API
42. [NextRequest/NextResponse](./app-router/api-reference/functions/next-request-response.md) - 요청/응답 처리
43. [unstable_cache](./app-router/api-reference/functions/unstable_cache.md) - 데이터 캐싱
44. [generateViewport](./app-router/api-reference/functions/generateViewport.md) - 뷰포트 설정
45. [useSelectedLayoutSegments](./app-router/api-reference/functions/useSelectedLayoutSegments.md) - 활성 세그먼트 배열
46. [unstable_noStore](./app-router/api-reference/functions/unstable_noStore.md) - 정적 렌더링 옵트아웃 (Deprecated)
47. [after](./app-router/api-reference/functions/after.md) - 응답 후 작업 실행
48. [connection](./app-router/api-reference/functions/connection.md) - 동적 렌더링 강제
49. [userAgent](./app-router/api-reference/functions/userAgent.md) - User Agent 파싱
50. [generateImageMetadata](./app-router/api-reference/functions/generateImageMetadata.md) - 이미지 메타데이터 생성
51. [useReportWebVitals](./app-router/api-reference/functions/useReportWebVitals.md) - Web Vitals 보고
52. [forbidden](./app-router/api-reference/functions/forbidden.md) - 403 에러 응답 🆕
53. [unauthorized](./app-router/api-reference/functions/unauthorized.md) - 401 에러 응답 🆕
54. [unstable_rethrow](./app-router/api-reference/functions/unstable_rethrow.md) - Next.js 에러 재발생 🆕

**React 19 Hooks (3개):**
55. [useFormStatus](./app-router/api-reference/functions/react/useFormStatus.md) - 폼 제출 상태 🆕
56. [useActionState](./app-router/api-reference/functions/react/useActionState.md) - 폼 action 상태 관리 🆕
57. [useOptimistic](./app-router/api-reference/functions/react/useOptimistic.md) - 낙관적 UI 업데이트 🆕

**next.config.js 옵션 (2개):**
58. [images](./app-router/api-reference/config/next-config-js/images.md) - 이미지 최적화 설정 🆕
59. [redirects](./app-router/api-reference/config/next-config-js/redirects.md) - URL 리디렉트 설정 🆕

---

## 📊 번역 현황

| 섹션 | 진행률 | 문서 수 |
|------|--------|---------|
| **App Router - Getting Started** | ✅ 100% | 12/12 |
| **App Router - Guides** | ✅ 100% | 44/44 |
| **App Router - API Reference** | 🟢 75% | 60/80+ |
| **Pages Router** | ⬜ 0% | 0/60+ |
| **Architecture** | ⬜ 0% | 0/4 |
| **Community** | ⬜ 0% | 0/2 |

**총 번역 완료:** 123개 문서 ⬆️ (약 61.5%)
**전체 문서:** 약 200개

📋 **상세 번역 상태:** [TRANSLATION_STATUS.md](./TRANSLATION_STATUS.md) 참조

---

## 🎯 번역 전략

토큰 제한을 고려하여 **가장 자주 사용되는 핵심 문서**를 우선적으로 번역했습니다:

### ✅ 완료된 핵심 주제
- 기본 설정 및 프로젝트 구조
- 라우팅 및 네비게이션
- 서버/클라이언트 컴포넌트
- 데이터 페칭 및 캐싱
- 폼 처리 및 Server Actions
- 인증 및 권한 부여
- 환경 변수
- 테스팅 (Jest, Playwright)
- 국제화
- 이미지/폰트 최적화
- 주요 API (Link, Image, Directives)
- 캐싱 전략 및 최적화
- 셀프 호스팅 및 배포
- 정적 사이트 생성
- 디버깅 및 문제 해결
- 핵심 API 함수들 (메타데이터, 라우팅, 캐시 재검증)
- Hook 함수들 (useRouter, usePathname, useSearchParams, useParams, useSelectedLayoutSegment)
- 컴포넌트 최적화 (Script, Font)
- **SEO 및 메타데이터 (사이트맵, robots.txt, OG 이미지)** 🆕

### 🚧 향후 번역 예정
- 추가 Guides (Analytics, Caching, Production 등)
- 나머지 API Reference
- Pages Router
- Architecture 문서

---

## 🌐 원본 문서

- **버전:** 16.1.1
- **원본 URL:** https://nextjs.org/docs
- **최종 업데이트:** 2025-12-19

---

## 📖 사용 방법

각 섹션의 마크다운 파일을 직접 열어서 읽거나, 선호하는 마크다운 뷰어를 사용하세요.

### 권장 뷰어
- **VS Code** (Markdown Preview)
- **Obsidian**
- **Notion**
- **GitHub** (직접 렌더링)

### 학습 경로 추천

#### 1️⃣ 초보자
1. [설치](./app-router/getting-started/01-installation.md)
2. [프로젝트 구조](./app-router/getting-started/02-project-structure.md)
3. [레이아웃과 페이지](./app-router/getting-started/03-layouts-and-pages.md)
4. [서버 및 클라이언트 컴포넌트](./app-router/getting-started/05-server-and-client-components.md)

#### 2️⃣ 중급자
1. [폼 처리](./app-router/guides/forms.md)
2. [환경 변수](./app-router/guides/environment-variables.md)
3. [에러 처리](./app-router/getting-started/07-error-handling.md)
4. [테스팅](./app-router/guides/testing-jest.md)

#### 3️⃣ 고급자
1. [인증](./app-router/guides/authentication.md)
2. [캐싱 전략](./app-router/guides/caching.md) 🆕
3. [국제화](./app-router/guides/internationalization.md)
4. [셀프 호스팅](./app-router/guides/self-hosting.md) 🆕
5. [디버깅](./app-router/guides/debugging.md) 🆕

---

## 🔍 빠른 참조

### 자주 찾는 문서

**컴포넌트:**
- [Link](./app-router/api-reference/link.md) - 클라이언트 사이드 네비게이션
- [Image](./app-router/api-reference/image.md) - 이미지 최적화

**지시어:**
- [use client](./app-router/api-reference/use-client.md) - 클라이언트 컴포넌트
- [use server](./app-router/api-reference/use-server.md) - 서버 함수

**주요 함수:**
- [generateMetadata](./app-router/api-reference/functions/generateMetadata.md) - 동적 메타데이터 생성
- [generateStaticParams](./app-router/api-reference/functions/generateStaticParams.md) - 정적 경로 생성
- [redirect](./app-router/api-reference/functions/redirect.md) - 리디렉션
- [notFound](./app-router/api-reference/functions/notFound.md) - 404 페이지
- [cookies](./app-router/api-reference/functions/cookies.md) - 쿠키 처리
- [headers](./app-router/api-reference/functions/headers.md) - 헤더 처리
- [revalidatePath](./app-router/api-reference/functions/revalidatePath.md) - 경로 재검증
- [revalidateTag](./app-router/api-reference/functions/revalidateTag.md) - 태그 재검증

**Hook 함수:** 🆕
- [useRouter](./app-router/api-reference/functions/useRouter.md) - 클라이언트 라우팅
- [usePathname](./app-router/api-reference/functions/usePathname.md) - 현재 경로
- [useSearchParams](./app-router/api-reference/functions/useSearchParams.md) - 쿼리 파라미터
- [useParams](./app-router/api-reference/functions/useParams.md) - 동적 파라미터
- [permanentRedirect](./app-router/api-reference/functions/permanentRedirect.md) - 영구 리디렉션
- [draftMode](./app-router/api-reference/functions/draftMode.md) - 초안 모드

**최적화 컴포넌트:** 🆕
- [Script](./app-router/api-reference/components/script.md) - 스크립트 최적화
- [Font](./app-router/api-reference/components/font.md) - 폰트 최적화

**주요 가이드:**
- [Forms](./app-router/guides/forms.md) - 폼 및 Server Actions
- [Authentication](./app-router/guides/authentication.md) - 인증 및 세션 관리
- [Environment Variables](./app-router/guides/environment-variables.md) - 환경 변수 설정
- [Caching](./app-router/guides/caching.md) - 캐싱 전략 및 성능 최적화
- [Self-Hosting](./app-router/guides/self-hosting.md) - 셀프 호스팅 가이드
- [Static Exports](./app-router/guides/static-exports.md) - 정적 사이트 생성
- [Debugging](./app-router/guides/debugging.md) - 디버깅 도구 및 방법

---

## 🤝 기여

오타나 번역 개선 사항이 있다면 자유롭게 Issue를 열거나 Pull Request를 제출해주세요.

### 기여 가이드라인
- 기술 용어는 일관성 있게 번역
- 코드 예제는 영어로 유지
- 주석은 한국어로 번역
- 공식 문서의 구조 유지

---

## 📝 번역 원칙

1. **기술 용어:** 일반적으로 사용되는 경우 영어 유지 (예: Server Actions, middleware)
2. **코드:** 모든 코드는 영어로 유지
3. **주석:** 코드 주석은 한국어로 번역
4. **일관성:** 용어 번역의 일관성 유지
5. **명확성:** 이해하기 쉬운 한국어 사용

---

## 📄 라이선스

이 번역 문서는 Next.js 공식 문서를 기반으로 하며, 원본 문서의 라이선스를 따릅니다.

---

## 🔗 관련 링크

- [Next.js 공식 사이트](https://nextjs.org)
- [Next.js GitHub](https://github.com/vercel/next.js)
- [Next.js 공식 문서](https://nextjs.org/docs)
- [Vercel](https://vercel.com)

---

## 💡 활용 팁

### VS Code에서 사이드바로 보기
1. VS Code 설치
2. 이 폴더 열기
3. 마크다운 파일 클릭
4. `Ctrl+K V` (또는 `Cmd+K V`) - 사이드바에 미리보기

### 검색하기
- VS Code: `Ctrl+Shift+F` (전체 검색)
- GitHub: 저장소에서 검색 기능 사용

---

**마지막 업데이트:** 2026-01-16
**번역 버전:** Next.js 16.1.2
**번역 완료:** Phase 19 완료 - 총 123개 핵심 문서 ✅
