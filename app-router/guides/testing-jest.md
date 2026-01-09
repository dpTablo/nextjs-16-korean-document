# Jest로 Next.js 테스트하기

**버전:** 16.1.1

## 개요
Jest와 React Testing Library는 Next.js 프로젝트에서 **유닛 테스트** 및 **스냅샷 테스트**를 위해 함께 사용됩니다. Next.js 12부터 Jest 설정이 내장되어 있습니다.

> **참고:** Jest는 현재 `async` 서버 컴포넌트를 지원하지 않습니다. 비동기 컴포넌트에는 E2E 테스트를 사용하세요.

## 빠른 시작
```bash
npx create-next-app@latest --example with-jest with-jest-app
```

## 수동 설정

### 1. 의존성 설치
```bash
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom ts-node @types/jest
```

### 2. Jest 초기화
```bash
npm init jest@latest
```

### 3. Jest 설정 (`jest.config.ts`)
```typescript
import type { Config } from 'jest'
import nextJest from 'next/jest.js'

const createJestConfig = nextJest({
  dir: './',
})

const config: Config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
  // setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
}

export default createJestConfig(config)
```

### 4. 커스텀 Matcher 추가 (선택사항)
`jest.config.ts`에서 주석 해제:
```typescript
setupFilesAfterEnv: ['<rootDir>/jest.setup.ts']
```

`jest.setup.ts` 생성:
```typescript
import '@testing-library/jest-dom'
```

### 5. 모듈 별칭 설정 (선택사항)
경로 별칭을 사용하는 경우 `jest.config.js` 업데이트:
```javascript
moduleNameMapper: {
  '^@/components/(.*)$': '<rootDir>/components/$1',
}
```

### 6. `package.json`에 테스트 스크립트 추가
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

## 테스트 작성하기

### 예시 컴포넌트 (`app/page.js`)
```jsx
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

### 유닛 테스트 (`__tests__/page.test.jsx`)
```jsx
import '@testing-library/jest-dom'
import { render, screen } from '@testing-library/react'
import Page from '../app/page'

describe('Page', () => {
  it('renders a heading', () => {
    render(<Page />)
    const heading = screen.getByRole('heading', { level: 1 })
    expect(heading).toBeInTheDocument()
  })
})
```

### 스냅샷 테스트 (`__tests__/snapshot.js`)
```jsx
import { render } from '@testing-library/react'
import Page from '../app/page'

it('renders homepage unchanged', () => {
  const { container } = render(<Page />)
  expect(container).toMatchSnapshot()
})
```

## 테스트 실행
```bash
npm run test
npm run test:watch
```

## `next/jest`가 자동으로 설정하는 것

- Next.js Compiler를 사용한 변환
- 스타일시트 자동 모킹 (`.css`, `.module.css`, `.scss`)
- 이미지 import 및 `next/font` 자동 모킹
- `.env` 파일을 `process.env`로 로드
- `node_modules` 및 `.next` 폴더 무시
- `next.config.js` 로드

## 리소스
- [Next.js Jest 예시](https://github.com/vercel/next.js/tree/canary/examples/with-jest)
- [Jest 문서](https://jestjs.io/)
- [React Testing Library 문서](https://testing-library.com/)
- [Testing Playground](https://testing-playground.com/)
