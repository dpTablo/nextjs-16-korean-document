# Cypress

Cypress는 **End-to-End (E2E)** 및 **Component Testing**을 위한 테스트 러너입니다.

> **주의**: Cypress 13.6.3 미만 버전은 `moduleResolution:"bundler"`와 TypeScript 5를 지원하지 않습니다.

## 수동 설정

### 설치

```bash
npm install -D cypress
```

### package.json 스크립트 추가

```json
{
  "scripts": {
    "cypress:open": "cypress open"
  }
}
```

### 초기 실행

```bash
npm run cypress:open
```

E2E Testing 또는 Component Testing을 선택하면 `cypress.config.js` 파일과 `cypress` 폴더가 자동 생성됩니다.

## E2E 테스트

### 설정

```ts
// cypress.config.ts
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    setupNodeEvents(on, config) {},
  },
})
```

### Next.js 페이지 생성

```jsx
// pages/index.js
import Link from 'next/link'

export default function Home() {
  return (
    <div>
      <h1>Home</h1>
      <Link href="/about">About</Link>
    </div>
  )
}
```

```jsx
// pages/about.js
import Link from 'next/link'

export default function About() {
  return (
    <div>
      <h1>About</h1>
      <Link href="/">Home</Link>
    </div>
  )
}
```

### E2E 테스트 작성

```js
// cypress/e2e/app.cy.js
describe('Navigation', () => {
  it('should navigate to the about page', () => {
    // 인덱스 페이지에서 시작
    cy.visit('http://localhost:3000/')

    // "about"을 포함하는 href 속성을 가진 링크 찾아서 클릭
    cy.get('a[href*="about"]').click()

    // 새 URL에 "/about"이 포함되어야 함
    cy.url().should('include', '/about')

    // 새 페이지에 "About"이라는 h1이 포함되어야 함
    cy.get('h1').contains('About')
  })
})
```

### E2E 테스트 실행

```bash
# 프로덕션 빌드 및 서버 시작
npm run build && npm run start

# 다른 터미널에서 Cypress 실행
npm run cypress:open
```

### 자동 서버 시작

`start-server-and-test` 패키지를 사용하면 테스트 실행 시 자동으로 서버를 시작할 수 있습니다:

```bash
npm install -D start-server-and-test
```

```json
{
  "scripts": {
    "test": "start-server-and-test start http://localhost:3000 cypress:open"
  }
}
```

## Component 테스트

Component 테스트는 전체 애플리케이션을 번들링하거나 서버를 시작할 필요 없이 특정 컴포넌트를 빌드하고 마운트합니다.

### 설정

```ts
// cypress.config.ts
import { defineConfig } from 'cypress'

export default defineConfig({
  component: {
    devServer: {
      framework: 'next',
      bundler: 'webpack',
    },
  },
})
```

### Component 테스트 작성

```js
// cypress/component/about.cy.js
import AboutPage from '../../pages/about'

describe('<AboutPage />', () => {
  it('should render and display expected content', () => {
    // About 페이지의 React 컴포넌트 마운트
    cy.mount(<AboutPage />)

    // h1에 "About"이 포함되어야 함
    cy.get('h1').contains('About')

    // 예상 URL을 가진 링크가 있는지 검증
    cy.get('a[href="/"]').should('be.visible')
  })
})
```

### Component 테스트 실행

```bash
npm run cypress:open
```

Component Testing을 선택하고 테스트를 실행합니다.

> **제한사항**:
> - `async` Server Components의 Component Testing은 현재 지원되지 않습니다 (E2E 테스트 권장)
> - `<Image />` 같은 서버 의존 기능은 작동하지 않을 수 있습니다

## 일반적인 테스트 패턴

### 요소 찾기

```js
// CSS 선택자
cy.get('button.primary')

// 텍스트 내용
cy.contains('Submit')

// data-testid
cy.get('[data-testid="submit-button"]')

// 조합
cy.get('form').find('button[type="submit"]')
```

### 폼 작성

```js
cy.get('input[name="email"]').type('test@example.com')
cy.get('input[name="password"]').type('password123')
cy.get('button[type="submit"]').click()
```

### Assertions

```js
// 존재 여부
cy.get('.success-message').should('exist')

// 텍스트 내용
cy.get('h1').should('have.text', 'Welcome')

// CSS 클래스
cy.get('button').should('have.class', 'active')

// 속성
cy.get('input').should('have.attr', 'placeholder', 'Enter email')
```

### 네트워크 요청 인터셉트

```js
cy.intercept('POST', '/api/login', {
  statusCode: 200,
  body: { success: true },
}).as('loginRequest')

cy.get('button[type="submit"]').click()
cy.wait('@loginRequest')
```

## CI/CD 설정

Headless 모드로 실행:

```json
{
  "scripts": {
    "e2e": "start-server-and-test dev http://localhost:3000 \"cypress open --e2e\"",
    "e2e:headless": "start-server-and-test dev http://localhost:3000 \"cypress run --e2e\"",
    "component": "cypress open --component",
    "component:headless": "cypress run --component"
  }
}
```

### GitHub Actions 예시

```yaml
# .github/workflows/cypress.yml
name: Cypress Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Cypress run
        uses: cypress-io/github-action@v6
        with:
          build: npm run build
          start: npm start
          wait-on: 'http://localhost:3000'
```

## 추가 리소스

- [Next.js with Cypress 예제](https://github.com/vercel/next.js/tree/canary/examples/with-cypress)
- [Cypress 공식 문서](https://docs.cypress.io/)
- [Cypress CI 설정](https://docs.cypress.io/guides/continuous-integration/introduction)
