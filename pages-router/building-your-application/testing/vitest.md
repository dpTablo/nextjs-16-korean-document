# Vitest

Vitest와 React Testing Library는 **단위 테스트(Unit Testing)**에 자주 사용됩니다. Vite 기반의 빠른 테스트 러너로, Jest와 호환되는 API를 제공합니다.

> **주의**: `async` Server Components는 현재 Vitest에서 지원되지 않습니다. 동기 Server와 Client Components에 대해서는 단위 테스트가 가능하지만, `async` components는 **E2E 테스트**를 권장합니다.

## 빠른 시작

```bash
npx create-next-app@latest --example with-vitest with-vitest-app
```

## 수동 설정

### 패키지 설치

```bash
# TypeScript 사용 시
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/dom vite-tsconfig-paths

# JavaScript 사용 시
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/dom
```

### Vitest 설정 파일

```ts
// vitest.config.mts (TypeScript)
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    environment: 'jsdom',
  },
})
```

```js
// vitest.config.js (JavaScript)
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
  },
})
```

### package.json 스크립트 추가

```json
{
  "scripts": {
    "test": "vitest"
  }
}
```

> `npm run test` 실행 시 Vitest는 기본적으로 변경사항을 **감시(watch)** 모드로 실행합니다.

## 첫 번째 테스트 작성

### 테스트할 컴포넌트

```tsx
// pages/index.tsx
import Link from 'next/link'

export default function Page() {
  return (
    <div>
      <h1>Home</h1>
      <Link href="/about">About</Link>
    </div>
  )
}
```

### 테스트 파일

```tsx
// __tests__/index.test.tsx
import { expect, test } from 'vitest'
import { render, screen } from '@testing-library/react'
import Page from '../pages/index'

test('Page', () => {
  render(<Page />)
  expect(screen.getByRole('heading', { level: 1, name: 'Home' })).toBeDefined()
})
```

## 테스트 실행

```bash
npm run test
```

## 추가 설정

### Setup 파일

테스트 실행 전에 공통 설정을 적용하려면 setup 파일을 사용합니다:

```ts
// vitest.config.mts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
  },
})
```

```ts
// vitest.setup.ts
import '@testing-library/jest-dom'
```

### 커버리지 설정

```bash
npm install -D @vitest/coverage-v8
```

```ts
// vitest.config.mts
export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
})
```

```bash
npm run test -- --coverage
```

### 모듈 별칭

`tsconfig.json`의 경로 별칭을 사용하려면 `vite-tsconfig-paths` 플러그인이 필요합니다:

```ts
// vitest.config.mts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    environment: 'jsdom',
  },
})
```

## Jest와의 차이점

Vitest는 Jest와 호환되는 API를 제공하지만 몇 가지 차이점이 있습니다:

| 기능 | Jest | Vitest |
|------|------|--------|
| 설정 파일 | `jest.config.js` | `vitest.config.ts` |
| 실행 속도 | 보통 | 빠름 (ESBuild 기반) |
| Watch 모드 | `--watch` 옵션 필요 | 기본 활성화 |
| ESM 지원 | 설정 필요 | 기본 지원 |

## 일반적인 테스트 패턴

### 컴포넌트 렌더링 테스트

```tsx
import { expect, test } from 'vitest'
import { render, screen } from '@testing-library/react'
import Button from '../components/Button'

test('renders button with text', () => {
  render(<Button>Click me</Button>)
  expect(screen.getByRole('button', { name: 'Click me' })).toBeDefined()
})
```

### 이벤트 테스트

```tsx
import { expect, test, vi } from 'vitest'
import { render, screen, fireEvent } from '@testing-library/react'
import Button from '../components/Button'

test('calls onClick when clicked', () => {
  const handleClick = vi.fn()
  render(<Button onClick={handleClick}>Click me</Button>)

  fireEvent.click(screen.getByRole('button'))

  expect(handleClick).toHaveBeenCalledTimes(1)
})
```

### 비동기 테스트

```tsx
import { expect, test } from 'vitest'
import { render, screen, waitFor } from '@testing-library/react'
import AsyncComponent from '../components/AsyncComponent'

test('loads and displays data', async () => {
  render(<AsyncComponent />)

  await waitFor(() => {
    expect(screen.getByText('Data loaded')).toBeDefined()
  })
})
```

### 모킹

```tsx
import { expect, test, vi } from 'vitest'

// 모듈 모킹
vi.mock('../lib/api', () => ({
  fetchData: vi.fn(() => Promise.resolve({ data: 'mocked' })),
}))

test('uses mocked data', async () => {
  const { fetchData } = await import('../lib/api')
  const result = await fetchData()
  expect(result.data).toBe('mocked')
})
```

## CI/CD

### GitHub Actions 예시

```yaml
# .github/workflows/test.yml
name: Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run test -- --run
```

## 추가 리소스

- [Next.js with Vitest 예제](https://github.com/vercel/next.js/tree/canary/examples/with-vitest)
- [Vitest 공식 문서](https://vitest.dev/guide/)
- [React Testing Library 문서](https://testing-library.com/docs/react-testing-library/intro/)
