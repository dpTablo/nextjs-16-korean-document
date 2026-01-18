# 접근성

Next.js 팀은 모든 개발자(및 그들의 최종 사용자)가 Next.js에 접근할 수 있도록 하기 위해 노력하고 있습니다. 기본적으로 Next.js에 접근성 기능을 추가함으로써 웹을 모든 사람에게 더 포용적으로 만들고자 합니다.

## 라우트 알림

서버에서 렌더링된 페이지 간 전환 시(`<a href>` 태그 사용), 스크린 리더와 기타 보조 기술은 페이지가 로드될 때 페이지 제목을 알려서 사용자에게 페이지가 변경되었음을 알립니다.

전통적인 페이지 네비게이션 외에도, Next.js는 향상된 성능을 위해 클라이언트 사이드 전환(`next/link` 사용)도 지원합니다. 클라이언트 사이드 전환도 보조 기술에 알려지도록 하기 위해 **Next.js는 기본적으로 라우트 알리미를 포함합니다**.

Next.js 라우트 알리미는 먼저 `document.title`을 검사하고, 그 다음 `<h1>` 요소, 마지막으로 URL 경로명을 확인하여 알릴 페이지 이름을 결정합니다. 가장 접근성 있는 사용자 경험을 위해 애플리케이션의 각 페이지에 고유하고 설명적인 제목이 있는지 확인하세요.

## 린팅

Next.js는 Next.js를 위한 커스텀 규칙을 포함한 [통합 ESLint 경험](/docs/app/building-your-application/configuring/eslint)을 기본으로 제공합니다. 기본적으로 Next.js는 접근성 문제를 조기에 감지하는 데 도움이 되는 `eslint-plugin-jsx-a11y`를 포함합니다. 예를 들어, 이 플러그인은 다음에 대해 경고합니다:

- `img` 태그에 alt 텍스트가 없는 경우
- 잘못된 `aria-*` 속성 사용
- 잘못된 `role` 속성 사용
- 그 외 다양한 접근성 문제

## 포함된 접근성 규칙

이 플러그인은 다음에 대해 경고합니다:

- **aria-props** - 잘못된 ARIA 속성
- **aria-proptypes** - 잘못된 ARIA 속성 타입
- **aria-unsupported-elements** - 지원되지 않는 요소에 ARIA 사용
- **role-has-required-aria-props** - 역할에 필요한 ARIA 속성 누락
- **role-supports-aria-props** - 특정 역할에 대한 잘못된 ARIA 속성

## 접근성 리소스

- [WebAIM WCAG 체크리스트](https://webaim.org/standards/wcag/checklist)
- [WCAG 2.2 가이드라인](https://www.w3.org/TR/WCAG22/)
- [The A11y Project](https://www.a11yproject.com/)
- [색상 대비 비율 가이드](https://developer.mozilla.org/docs/Web/Accessibility/Understanding_WCAG/Perceivable/Color_contrast)
- [애니메이션을 위한 `prefers-reduced-motion`](https://web.dev/prefers-reduced-motion/)
