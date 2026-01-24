# 지연 로딩 (Lazy Loading)

Next.js에서 지연 로딩(Lazy Loading)은 라우트를 렌더링하는 데 필요한 JavaScript 양을 줄여 애플리케이션의 초기 로딩 성능을 개선하는 데 도움이 됩니다.

클라이언트 컴포넌트와 가져온 라이브러리의 로딩을 지연시켜 필요할 때만 클라이언트 번들에 포함되도록 할 수 있습니다.

## next/dynamic

`next/dynamic`은 `React.lazy()`와 `Suspense`를 결합한 도구로, `pages` 디렉토리에서 사용할 수 있습니다.

## 기본 사용법

```jsx
import dynamic from 'next/dynamic'

const DynamicHeader = dynamic(() => import('../components/header'), {
  loading: () => <p>로딩 중...</p>,
})

export default function Home() {
  return <DynamicHeader />
}
```

**동작 원리:**
1. `Header` 컴포넌트는 초기 JavaScript 번들에 포함되지 않습니다.
2. 먼저 `loading` 컴포넌트가 렌더링됩니다.
3. Suspense 경계가 해결되면 `Header` 컴포넌트가 렌더링됩니다.

## 주의사항

- `import('path/to/component')`의 경로는 명시적으로 작성되어야 합니다 (템플릿 문자열이나 변수 사용 불가).
- `import()`는 `dynamic()` 호출 내부에 위치해야 합니다.
- `dynamic()`은 React 렌더링 내부에서 사용할 수 없습니다 (모듈 최상위에 위치해야 함).

## Named Exports 사용

Named Export로 내보내진 컴포넌트를 동적으로 가져오려면 `then()` 메서드를 사용합니다:

```jsx
// components/hello.js
export function Hello() {
  return <p>안녕하세요!</p>
}

// pages/index.js
import dynamic from 'next/dynamic'

const DynamicComponent = dynamic(() =>
  import('../components/hello').then((mod) => mod.Hello)
)

export default function Page() {
  return <DynamicComponent />
}
```

## SSR 비활성화

클라이언트 사이드에서만 컴포넌트를 로드하려면 `ssr` 옵션을 사용합니다:

```jsx
import dynamic from 'next/dynamic'

const DynamicHeader = dynamic(() => import('../components/header'), {
  ssr: false,
})

export default function Page() {
  return <DynamicHeader />
}
```

**사용 시기:**
- 외부 라이브러리나 컴포넌트가 `window` 같은 브라우저 API에 의존할 때
- 서버 사이드 렌더링이 필요 없는 컴포넌트

## 외부 라이브러리 동적 로딩

라이브러리를 필요할 때만 가져올 수 있습니다:

```jsx
import { useState } from 'react'

const names = ['Tim', 'Joe', 'Bel', 'Lee']

export default function Page() {
  const [results, setResults] = useState()

  return (
    <div>
      <input
        type="text"
        placeholder="검색"
        onChange={async (e) => {
          const { value } = e.currentTarget
          // 사용자 입력 시에만 fuse.js 로드
          const Fuse = (await import('fuse.js')).default
          const fuse = new Fuse(names)
          setResults(fuse.search(value))
        }}
      />
      <pre>결과: {JSON.stringify(results, null, 2)}</pre>
    </div>
  )
}
```

**장점:** `fuse.js` 라이브러리는 사용자가 검색 입력 시에만 로드되어 초기 번들 크기가 감소합니다.

## 커스텀 로딩 컴포넌트

로딩 상태를 표시하는 커스텀 컴포넌트를 지정할 수 있습니다:

```jsx
import dynamic from 'next/dynamic'

const DynamicChart = dynamic(() => import('../components/chart'), {
  loading: () => (
    <div className="skeleton">
      <div className="skeleton-chart" />
    </div>
  ),
})

export default function Dashboard() {
  return (
    <div>
      <h1>대시보드</h1>
      <DynamicChart />
    </div>
  )
}
```

## 여러 컴포넌트 동적 로딩

조건부로 여러 컴포넌트를 로드할 수 있습니다:

```jsx
import dynamic from 'next/dynamic'

const DynamicLight = dynamic(() => import('../components/light-theme'))
const DynamicDark = dynamic(() => import('../components/dark-theme'))

export default function Page({ theme }) {
  const ThemeComponent = theme === 'dark' ? DynamicDark : DynamicLight
  return <ThemeComponent />
}
```

## Suspense와 함께 사용

`suspense` 옵션을 사용하면 React Suspense와 통합할 수 있습니다:

```jsx
import dynamic from 'next/dynamic'
import { Suspense } from 'react'

const DynamicComponent = dynamic(() => import('../components/heavy-component'), {
  suspense: true,
})

export default function Page() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <DynamicComponent />
    </Suspense>
  )
}
```

## 프리로딩

성능을 더 향상시키려면 컴포넌트를 미리 로드할 수 있습니다:

```jsx
import dynamic from 'next/dynamic'

const DynamicModal = dynamic(() => import('../components/modal'))

export default function Page() {
  const handleMouseEnter = () => {
    // 사용자가 버튼에 마우스를 올리면 모달 프리로드
    DynamicModal.preload()
  }

  return (
    <button onMouseEnter={handleMouseEnter} onClick={openModal}>
      모달 열기
    </button>
  )
}
```

## 일반적인 패턴

### 모달/다이얼로그

```jsx
import { useState } from 'react'
import dynamic from 'next/dynamic'

const DynamicModal = dynamic(() => import('../components/modal'), {
  ssr: false,
})

export default function Page() {
  const [showModal, setShowModal] = useState(false)

  return (
    <>
      <button onClick={() => setShowModal(true)}>모달 열기</button>
      {showModal && <DynamicModal onClose={() => setShowModal(false)} />}
    </>
  )
}
```

### 조건부 기능

```jsx
import { useState } from 'react'
import dynamic from 'next/dynamic'

const DynamicEditor = dynamic(() => import('../components/rich-editor'), {
  ssr: false,
  loading: () => <textarea placeholder="에디터 로딩 중..." />,
})

export default function Page() {
  const [useRichEditor, setUseRichEditor] = useState(false)

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={useRichEditor}
          onChange={(e) => setUseRichEditor(e.target.checked)}
        />
        리치 에디터 사용
      </label>
      {useRichEditor ? (
        <DynamicEditor />
      ) : (
        <textarea placeholder="기본 텍스트 입력" />
      )}
    </>
  )
}
```

## 성능 팁

1. **필요할 때만 로드**: 초기 렌더링에 필요하지 않은 컴포넌트는 지연 로드하세요.
2. **적절한 로딩 UI 제공**: 사용자 경험을 위해 로딩 상태를 표시하세요.
3. **프리로딩 활용**: 사용자 행동을 예측하여 미리 로드하세요.
4. **번들 분석**: `@next/bundle-analyzer`를 사용하여 번들 크기를 확인하세요.
