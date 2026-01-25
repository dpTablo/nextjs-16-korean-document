# 문서 기여 가이드

Next.js 문서에 기여하는 방법에 대한 공식 가이드입니다. 이 가이드는 커뮤니티 구성원이 오타 수정, 섹션 명확화, 누락된 주제 추가를 통해 문서를 개선할 수 있도록 합니다.

---

## 기여해야 하는 이유

- 문서는 커뮤니티 지원이 필요한 지속적인 프로세스입니다
- 오픈 소스 입문자에게 좋은 시작점입니다
- 경험 많은 개발자는 복잡한 주제를 명확히 할 수 있습니다
- 모든 기여(오타, 혼란스러운 섹션, 누락된 주제)를 환영합니다

---

## 기여 방법

**위치:** [Next.js 저장소](https://github.com/vercel/next.js/tree/canary/docs)

**방법:**
- GitHub에서 직접 파일 편집
- 저장소 복제 후 로컬에서 편집

### GitHub 워크플로우

- [GitHub 오픈 소스 가이드](https://opensource.guide/how-to-contribute/#opening-a-pull-request) 참조
- 저장소 포크, 브랜치 생성, 풀 리퀘스트 제출
- 참고: 문서 코드는 비공개 코드베이스에 있으며 공개 저장소와 동기화됩니다
- 변경 사항은 PR 병합 후 [nextjs.org](https://nextjs.org/docs)에 나타납니다

---

## MDX로 작성하기

문서는 JSX 구문을 지원하는 마크다운 형식인 [MDX](https://mdxjs.com/)를 사용합니다.

### VSCode 설정

```json
// settings.json
{
  "files.associations": {
    "*.mdx": "markdown"
  }
}
```

### 권장 확장 프로그램

- [MDX](https://marketplace.visualstudio.com/items?itemName=unifiedjs.vscode-mdx) - IntelliSense 및 구문 강조
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) - 저장 시 포맷

> **팁:** PR 제출 전에 `pnpm prettier-fix`를 실행하세요.

---

## 파일 구조

`/docs`의 폴더/파일이 라우트 세그먼트를 나타내는 **파일 시스템 라우팅**을 사용합니다.

### 정렬

- **기본값:** 알파벳순
- **커스텀:** 두 자리 접두사 추가 (예: `00-`, `01-`)

**예제 - 알파벳순 (API Reference):**

```
04-functions
├── after.mdx
├── cacheLife.mdx
├── cacheTag.mdx
└── ...
```

**예제 - 번호 매김 (학습 경로):**

```
01-getting-started
├── 01-installation.mdx
├── 02-project-structure.mdx
├── 03-layouts-and-pages.mdx
└── ...
```

---

## 메타데이터

각 페이지 상단에 세 개의 대시로 구분된 메타데이터 블록이 있습니다.

### 필수 필드

| 필드 | 설명 |
|------|------|
| `title` | SEO 및 OG 이미지용 페이지의 `<h1>` 제목 (2-3단어 권장) |
| `description` | SEO용 메타 설명 태그 (1-2문장 권장) |

```yaml
---
title: 페이지 제목
description: 페이지 설명
---
```

### 선택적 필드

| 필드 | 설명 |
|------|------|
| `nav_title` | 네비게이션에서 페이지 제목 재정의 (긴 제목용) |
| `source` | 공유 페이지로 콘텐츠 가져오기 |
| `related` | 카드로 표시되는 관련 페이지 목록 |
| `version` | 개발 단계 (`experimental`, `legacy`, `unstable`, `RC`) |

---

## App과 Pages 문서

다른 기능으로 인해 App Router와 Pages Router 문서가 분리되어 있습니다 (`02-app` 및 `03-pages`).

### 공유 페이지

콘텐츠 중복을 피하기 위해 `source` 필드 사용:

```mdx
// app/.../link.mdx
---
title: <Link>
description: Link 컴포넌트 API 레퍼런스.
---

이 API 레퍼런스는 Link 컴포넌트에 사용 가능한
props와 설정 옵션을 이해하는 데 도움이 됩니다.
```

```mdx
// pages/.../link.mdx
---
title: <Link>
description: Link 컴포넌트 API 레퍼런스.
source: app/api-reference/components/link
---

{/* 이 페이지를 편집하지 마세요. */}
{/* 이 페이지의 콘텐츠는 위 소스에서 가져옵니다. */}
```

### 공유 콘텐츠

라우터별 콘텐츠에 `<AppOnly>` 또는 `<PagesOnly>` 컴포넌트 사용:

```mdx
이 콘텐츠는 App과 Pages 간에 공유됩니다.

<PagesOnly>

이 콘텐츠는 Pages 문서에서만 표시됩니다.

</PagesOnly>

이 콘텐츠는 App과 Pages 간에 공유됩니다.
```

---

## 코드 블록

### 요구 사항

- 최소 작동 예제
- import 및 필요한 설정 포함
- 커밋 전에 항상 로컬에서 테스트

```tsx
// app/page.tsx
import Link from 'next/link'

export default function Page() {
  return <Link href="/about">About</Link>
}
```

### 언어 및 파일명 헤더

코드 블록에 언어와 `filename` prop 추가:

````mdx
```bash filename="Terminal"
npx create-next-app
```
````

### JavaScript/TypeScript 조합

| 타입 | 언어 | 확장자 |
|------|------|--------|
| JSX가 있는 JavaScript | `jsx` | `.js` |
| JSX가 없는 JavaScript | `js` | `.js` |
| JSX가 있는 TypeScript | `tsx` | `.tsx` |
| JSX가 없는 TypeScript | `ts` | `.ts` |

### TS와 JS 스위처

TypeScript와 JavaScript 간 토글을 위해 `switcher` prop 사용 (TypeScript 먼저):

````mdx
```tsx filename="app/page.tsx" switcher

```

```jsx filename="app/page.js" switcher

```
````

### 줄 하이라이팅

**단일 줄:**
```tsx highlight={1}
```

**여러 줄:**
```tsx highlight={1,3}
```

**범위:**
```tsx highlight={1-5}
```

---

## 노트

**단일 줄 노트:**

```mdx
> **알아두면 좋은 점**: 단일 줄 노트입니다.
```

**여러 줄 노트:**

```mdx
> **알아두면 좋은 점**:
>
> - 여러 줄 노트에도 이 형식을 사용합니다.
> - 알아두거나 기억해야 할 여러 항목이 있을 수 있습니다.
```

---

## 관련 링크

논리적인 다음 단계로 사용자의 학습 여정을 안내합니다.

```yaml
---
related:
  description: 첫 번째 애플리케이션을 빠르게 시작하는 방법을 알아보세요.
  links:
    - app/building-your-application/routing/defining-routes
    - app/building-your-application/data-fetching
    - app/api-reference/file-conventions/page
---
```

### 중첩 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `title` | 선택 | 카드 목록 제목 (기본값: "다음 단계") |
| `description` | 선택 | 카드 목록 설명 |
| `links` | 필수 | 문서 페이지에 대한 상대 URL 경로 목록 |

---

## 스타일 가이드

### 페이지 템플릿

- **개요:** 기능이 무엇이고 용도는 무엇인지 (최소 작동 예제 또는 API 레퍼런스 포함)
- **규칙:** 해당하는 경우 규칙 설명
- **예제:** 다양한 사용 사례 표시
- **API 테이블:** 섹션 점프 링크가 있는 개요 테이블
- **다음 단계 (관련 링크):** 학습 여정 안내

### 페이지 타입

| 타입 | 목적 | 위치 | 특성 |
|------|------|------|------|
| **개념적** | 개념/기능 설명 | Building Your Application | 길고 교육적, "you" 사용 |
| **레퍼런스** | 특정 API 설명 | API Reference | 짧고 집중적, 기술적, 명령형 |

### 작성 지침

- 명확하고 간결한 문장 작성; 주제 이탈 피하기
- 긴 문장을 여러 문장이나 목록으로 분리
- 간단한 단어 사용 (예: "use" not "utilize")
- "this" 사용 시 주의 - 불명확하면 주어 반복
- 능동태 사용 ("Next.js uses React" not "React is used by Next.js")
- 주관적인 단어 피하기: *easy*, *quick*, *simple*, *just*
- 부정적인 단어 피하기: *don't*, *can't*, *won't*
- 2인칭으로 작성 (you/your)
- 성 중립적 언어 사용 (*developers*, *users*, *readers*)
- 코드 예제가 올바르게 포맷되고 작동하는지 확인

---

## 검토 프로세스

1. 기여와 함께 풀 리퀘스트 제출
2. Next.js 또는 Developer Experience 팀이 변경 사항 검토
3. 피드백 제공
4. 준비되면 PR 병합

필요한 경우 PR 댓글에서 질문하세요.

---

**Next.js 문서에 기여하고 커뮤니티의 일원이 되어 주셔서 감사합니다!**
