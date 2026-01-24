# Jest

Jest와 React Testing Library는 **단위 테스트(Unit Testing)**와 **스냅샷 테스트(Snapshot Testing)**에 자주 사용됩니다.

> **주의**: `async` Server Components는 Jest가 아직 지원하지 않습니다. 동기 Server/Client Components의 단위 테스트는 가능하지만, `async` 컴포넌트는 **E2E 테스트** 사용을 권장합니다.

## 빠른 시작

```bash
npx create-next-app@latest --example with-jest with-jest-app
```

## 수동 설정

### 패키지 설치

```bash
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom ts-node @types/jest
```

### Jest 초기화

```bash
npm init jest@latest
```

### Jest 설정 파일

```ts
// jest.config.ts
import type { Config } from 'jest'
import nextJest from 'next/jest.js'

const createJestConfig = nextJest({
  // Next.js 앱의 경로 (next.config.js와 .env 파일 로드)
  dir: './',
})

const config: Config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
  // 각 테스트 전에 추가 설정 옵션 실행
  // setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
}

export default createJestConfig(config)
```

```js
// jest.config.js
const nextJest = require('next/jest')

/** @type {import('jest').Config} */
const createJestConfig = nextJest({
  dir: './',
})

const config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
}

module.exports = createJestConfig(config)
```

### next/jest 자동 설정 항목

`next/jest`는 다음 항목을 자동으로 설정합니다:

- `transform` 설정 (Next.js Compiler 사용)
- 스타일시트 자동 모킹 (`.css`, `.module.css`, scss 변형)
- 이미지 imports 및 `next/font` 모킹
- `.env` 파일 로드
- `node_modules` 테스트에서 제외
- `.next` 디렉토리 테스트에서 제외
- `next.config.js` 로드 (SWC 변환 플래그 활성화용)

## Jest 커스텀 Matchers 확장

`@testing-library/jest-dom`을 사용하면 `.toBeInTheDocument()` 같은 편리한 matchers를 사용할 수 있습니다.

```ts
// jest.setup.ts
import '@testing-library/jest-dom'
```

설정 파일에 추가:

```ts
// jest.config.ts
setupFilesAfterEnv: ['<rootDir>/jest.setup.ts']
```

## package.json 스크립트 추가

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

## 첫 번째 테스트 작성

### 컴포넌트

```jsx
// pages/index.js
export default function Home() {
  return <h1>Home</h1>
}
```

### 테스트 파일

```jsx
// __tests__/index.test.js
import '@testing-library/jest-dom'
import { render, screen } from '@testing-library/react'
import Home from '../pages/index'

describe('Home', () => {
  it('renders a heading', () => {
    render(<Home />)

    const heading = screen.getByRole('heading', { level: 1 })

    expect(heading).toBeInTheDocument()
  })
})
```

### 스냅샷 테스트

```jsx
// __tests__/snapshot.js
import { render } from '@testing-library/react'
import Home from '../pages/index'

it('renders homepage unchanged', () => {
  const { container } = render(<Home />)
  expect(container).toMatchSnapshot()
})
```

> **주의**: 테스트 파일은 Pages Router 내부에 포함되면 안 됩니다.

## 테스트 실행

```bash
npm run test

# Watch 모드
npm run test:watch
```

## Babel 설정 (선택사항)

Next.js Compiler 대신 Babel을 사용하는 경우:

```bash
npm install -D babel-jest identity-obj-proxy
```

```js
// jest.config.js
module.exports = {
  collectCoverage: true,
  coverageProvider: 'v8',
  collectCoverageFrom: [
    '**/*.{js,jsx,ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
    '!<rootDir>/out/**',
    '!<rootDir>/.next/**',
    '!<rootDir>/*.config.js',
    '!<rootDir>/coverage/**',
  ],
  moduleNameMapper: {
    // CSS 모듈 처리
    '^.+\\.module\\.(css|sass|scss)$': 'identity-obj-proxy',
    // CSS imports 처리
    '^.+\\.(css|sass|scss)$': '<rootDir>/__mocks__/styleMock.js',
    // 이미지 imports 처리
    '^.+\\.(png|jpg|jpeg|gif|webp|avif|ico|bmp|svg)$': '<rootDir>/__mocks__/fileMock.js',
    // 모듈 별칭 처리
    '^@/components/(.*)$': '<rootDir>/components/$1',
  },
  testPathIgnorePatterns: ['<rootDir>/node_modules/', '<rootDir>/.next/'],
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': ['babel-jest', { presets: ['next/babel'] }],
  },
  transformIgnorePatterns: [
    '/node_modules/',
    '^.+\\.module\\.(css|sass|scss)$',
  ],
}
```

## Mock 파일

스타일시트와 이미지를 처리하기 위한 mock 파일:

```js
// __mocks__/fileMock.js
module.exports = 'test-file-stub'
```

```js
// __mocks__/styleMock.js
module.exports = {}
```

## 폰트 Mock

```js
// __mocks__/nextFontMock.js
module.exports = new Proxy(
  {},
  {
    get: function getter() {
      return () => ({
        className: 'className',
        variable: 'variable',
        style: { fontFamily: 'fontFamily' },
      })
    },
  }
)
```

## 모듈 경로 별칭

`tsconfig.json`의 경로 별칭을 Jest에서 사용하려면:

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/components/*": ["components/*"]
    }
  }
}
```

```js
// jest.config.js
moduleNameMapper: {
  '^@/components/(.*)$': '<rootDir>/components/$1',
}
```

## 추가 리소스

- [Next.js with Jest 예제](https://github.com/vercel/next.js/tree/canary/examples/with-jest)
- [Jest 문서](https://jestjs.io/docs/getting-started)
- [React Testing Library 문서](https://testing-library.com/docs/react-testing-library/intro/)
