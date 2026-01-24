# Next.js에서 Vitest 설정하기

Vitest와 React Testing Library는 **단위 테스트(Unit Testing)**를 위해 자주 함께 사용됩니다. 이 가이드는 Next.js에서 Vitest를 설정하고 첫 번째 테스트를 작성하는 방법을 보여줍니다.

> **참고:** `async` Server Components는 React 생태계에서 새로운 기능이므로, Vitest는 현재 이를 지원하지 않습니다. 동기적 Server 및 Client Components에 대한 **단위 테스트**를 실행할 수 있지만, `async` 컴포넌트의 경우 **E2E 테스트** 사용을 권장합니다.

## 빠른 시작

`create-next-app`을 Next.js [with-vitest](https://github.com/vercel/next.js/tree/canary/examples/with-vitest) 예제와 함께 사용하여 빠르게 시작할 수 있습니다:

```bash
npx create-next-app@latest --example with-vitest with-vitest-app
```

## 수동 설정

Vitest를 수동으로 설정하려면 `vitest` 및 다음 패키지를 개발 종속성으로 설치하세요:

```bash
# TypeScript 사용
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/dom vite-tsconfig-paths

# JavaScript 사용
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/dom
```

프로젝트 루트에 `vitest.config.mts|js` 파일을 생성하고 다음 옵션을 추가하세요:

```typescript filename="vitest.config.mts"
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

```javascript filename="vitest.config.js"
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
  },
})
```

Vitest 설정에 대한 자세한 정보는 [Vitest Configuration](https://vitest.dev/config/#configuration) 문서를 참조하세요.

다음으로, `package.json`에 `test` 스크립트를 추가하세요:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test": "vitest"
  }
}
```

`npm run test`를 실행하면 Vitest는 기본적으로 프로젝트의 변경 사항을 **감시**합니다.

## 첫 번째 Vitest 단위 테스트 작성

`<Page />` 컴포넌트가 제목을 성공적으로 렌더링하는지 확인하는 테스트를 작성하여 모든 것이 작동하는지 확인하세요:

```tsx filename="app/page.tsx"
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

```jsx filename="app/page.jsx"
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

```tsx filename="__tests__/page.test.tsx"
import { expect, test } from 'vitest'
import { render, screen } from '@testing-library/react'
import Page from '../app/page'

test('Page', () => {
  render(<Page />)
  expect(screen.getByRole('heading', { level: 1, name: 'Home' })).toBeDefined()
})
```

```jsx filename="__tests__/page.test.jsx"
import { expect, test } from 'vitest'
import { render, screen } from '@testing-library/react'
import Page from '../app/page'

test('Page', () => {
  render(<Page />)
  expect(screen.getByRole('heading', { level: 1, name: 'Home' })).toBeDefined()
})
```

> **참고:** 위 예제는 일반적인 `__tests__` 규칙을 사용하지만, 테스트 파일은 `app` 라우터 내에 함께 배치할 수도 있습니다.

## 테스트 실행

다음 명령을 실행하여 테스트를 실행하세요:

```bash
npm run test
# 또는
yarn test
# 또는
pnpm test
# 또는
bun test
```

## 추가 리소스

다음 리소스가 도움이 될 수 있습니다:

- [Next.js with Vitest 예제](https://github.com/vercel/next.js/tree/canary/examples/with-vitest)
- [Vitest 문서](https://vitest.dev/guide/)
- [React Testing Library 문서](https://testing-library.com/docs/react-testing-library/intro/)
