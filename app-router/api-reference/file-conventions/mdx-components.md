# mdx-components.js

`mdx-components.js|tsx` 파일은 Next.js App Router에서 `@next/mdx`를 사용하기 위해 **필수**입니다. MDX 통합을 활성화하고 스타일 및 컴포넌트를 사용자 정의할 수 있습니다.

## 파일 위치

`mdx-components.tsx` (또는 `.js`)를 프로젝트 루트에 배치합니다. 다음과 같은 수준에 위치해야 합니다:
- `pages` 또는 `app` 디렉토리와 같은 수준
- 또는 해당되는 경우 `src` 폴더 내에

```
my-project/
├── app/
├── mdx-components.tsx  ← 여기
└── package.json
```

또는:

```
my-project/
├── src/
│   ├── app/
│   └── mdx-components.tsx  ← 또는 여기
└── package.json
```

## 필수 내보내기

이 파일은 단일 함수 **`useMDXComponents`**를 내보내야 합니다.

이 함수는:
- **인수를 받지 않습니다**
- `MDXComponents` 객체를 반환합니다

### TypeScript 구현

```tsx
// mdx-components.tsx
import type { MDXComponents } from 'mdx/types'

const components: MDXComponents = {}

export function useMDXComponents(): MDXComponents {
  return components
}
```

### JavaScript 구현

```js
// mdx-components.js
const components = {}

export function useMDXComponents() {
  return components
}
```

## 컴포넌트 사용자 정의

MDX 콘텐츠에서 사용할 사용자 정의 컴포넌트를 정의할 수 있습니다:

```tsx
// mdx-components.tsx
import type { MDXComponents } from 'mdx/types'

export function useMDXComponents(): MDXComponents {
  return {
    // 기본 HTML 요소 재정의
    h1: ({ children }) => (
      <h1 className="text-3xl font-bold my-4">{children}</h1>
    ),
    h2: ({ children }) => (
      <h2 className="text-2xl font-semibold my-3">{children}</h2>
    ),
    p: ({ children }) => (
      <p className="my-2 leading-relaxed">{children}</p>
    ),
    a: ({ href, children }) => (
      <a href={href} className="text-blue-600 hover:underline">
        {children}
      </a>
    ),
    code: ({ children }) => (
      <code className="bg-gray-100 px-1 rounded">{children}</code>
    ),
  }
}
```

## 기존 컴포넌트와 결합

기존 컴포넌트와 사용자 정의 컴포넌트를 결합할 수 있습니다:

```tsx
// mdx-components.tsx
import type { MDXComponents } from 'mdx/types'
import { Button } from '@/components/Button'
import { Card } from '@/components/Card'

export function useMDXComponents(): MDXComponents {
  return {
    // 사용자 정의 컴포넌트
    Button,
    Card,
    // HTML 요소 재정의
    h1: ({ children }) => (
      <h1 className="text-3xl font-bold">{children}</h1>
    ),
  }
}
```

그런 다음 MDX 파일에서 사용합니다:

```mdx
# 안녕하세요

<Button>클릭하세요</Button>

<Card title="카드 제목">
  이것은 카드 내용입니다.
</Card>
```

## 주요 사항

- **App Router 필수**: `@next/mdx`를 App Router와 함께 사용할 때 이 파일은 필수입니다
- **파일 없이는 작동하지 않음**: MDX 기능은 이 파일의 존재에 의존합니다
- **사용자 정의**: MDX 콘텐츠의 사용자 정의 컴포넌트와 스타일을 정의하는 데 사용합니다

## 버전 기록

| 버전 | 변경 사항 |
|------|----------|
| v13.1.2 | MDX 컴포넌트 기능 추가 |

---

## 참고

- [MDX 가이드](/app-router/guides/mdx.md)
- [@next/mdx 패키지](https://www.npmjs.com/package/@next/mdx)
