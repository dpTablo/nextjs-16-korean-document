# Next.js에서 Cypress 설정하기

[Cypress](https://www.cypress.io/)는 **End-to-End (E2E)** 및 **Component Testing**을 위한 테스트 러너입니다. 이 페이지에서는 Cypress를 Next.js와 설정하고 첫 번째 테스트를 작성하는 방법을 보여줍니다.

> **경고:**
> - Cypress 13.6.3 미만 버전은 `moduleResolution:"bundler"`와 함께 [TypeScript 버전 5](https://github.com/cypress-io/cypress/issues/27731)를 지원하지 않습니다. 다만 Cypress 버전 13.6.3 이상에서는 이 문제가 해결되었습니다.

## 빠른 시작

`create-next-app`과 [with-cypress 예제](https://github.com/vercel/next.js/tree/canary/examples/with-cypress)를 사용하여 빠르게 시작할 수 있습니다.

```bash
npx create-next-app@latest --example with-cypress with-cypress-app
```

## 수동 설정

Cypress를 수동으로 설정하려면 `cypress`를 dev dependency로 설치하세요:

```bash
npm install -D cypress
# 또는
yarn add -D cypress
# 또는
pnpm install -D cypress
```

`package.json` scripts 필드에 Cypress `open` 명령을 추가하세요:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "cypress:open": "cypress open"
  }
}
```

Cypress를 처음 실행하여 Cypress 테스팅 스위트를 엽니다:

```bash
npm run cypress:open
```

**E2E Testing** 및/또는 **Component Testing**을 구성하도록 선택할 수 있습니다. 이 옵션 중 하나를 선택하면 자동으로 `cypress.config.js` 파일과 `cypress` 폴더가 프로젝트에 생성됩니다.

## 첫 번째 Cypress E2E 테스트 작성

`cypress.config` 파일이 다음 구성을 가지고 있는지 확인하세요:

```ts filename="cypress.config.ts"
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    setupNodeEvents(on, config) {},
  },
})
```

```js filename="cypress.config.js"
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  e2e: {
    setupNodeEvents(on, config) {},
  },
})
```

그런 다음 두 개의 새로운 Next.js 파일을 만듭니다:

```jsx filename="app/page.js"
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

```jsx filename="app/about/page.js"
import Link from 'next/link'

export default function Page() {
  return (
    <div>
      <h1>About</h1>
      <Link href="/">Home</Link>
    </div>
  )
}
```

네비게이션이 올바르게 작동하는지 확인하는 테스트를 추가하세요:

```js filename="cypress/e2e/app.cy.js"
describe('Navigation', () => {
  it('should navigate to the about page', () => {
    // 인덱스 페이지에서 시작
    cy.visit('http://localhost:3000/')

    // "about"을 포함하는 href 속성을 가진 링크를 찾아 클릭
    cy.get('a[href*="about"]').click()

    // 새 URL은 "/about"을 포함해야 함
    cy.url().should('include', '/about')

    // 새 페이지에는 "About" h1이 포함되어야 함
    cy.get('h1').contains('About')
  })
})
```

### E2E 테스트 실행

Cypress는 사용자가 애플리케이션을 탐색하는 것을 시뮬레이션합니다. 이를 위해 Next.js 서버가 실행 중이어야 합니다. 애플리케이션이 실제로 동작하는 방식을 더 정확히 파악하기 위해 프로덕션 코드에 대해 테스트를 실행하는 것을 권장합니다.

`npm run build && npm run start`를 실행하여 Next.js 애플리케이션을 빌드하고, 다른 터미널 창에서 `npm run cypress:open`을 실행하여 Cypress를 시작하고 E2E 테스팅 스위트를 실행합니다.

> **알아두면 좋은 점:**
> - `cypress.config.js` 구성 파일에 `baseUrl: 'http://localhost:3000'`을 추가하여 `cy.visit("/")`을 사용할 수 있습니다.
> - 또는 [`start-server-and-test`](https://www.npmjs.com/package/start-server-and-test) 패키지를 설치하여 Next.js 프로덕션 서버를 Cypress와 함께 실행할 수 있습니다. 설치 후 `package.json` scripts 필드에 `"test": "start-server-and-test start http://localhost:3000 cypress"`를 추가합니다. 새로운 변경 사항 후에는 애플리케이션을 다시 빌드하세요.

## 첫 번째 Cypress Component 테스트 작성

Component 테스트는 전체 애플리케이션을 번들하거나 서버를 시작할 필요 없이 특정 컴포넌트를 빌드하고 마운트합니다.

Cypress 앱에서 **Component Testing**을 선택한 다음 **Next.js**를 프론트엔드 프레임워크로 선택합니다. `cypress/component` 폴더가 프로젝트에 생성되고, `cypress.config.js` 파일이 업데이트되어 Component Testing이 활성화됩니다.

`cypress.config` 파일이 다음 구성을 가지고 있는지 확인하세요:

```ts filename="cypress.config.ts"
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

```js filename="cypress.config.js"
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  component: {
    devServer: {
      framework: 'next',
      bundler: 'webpack',
    },
  },
})
```

이전 섹션의 동일한 컴포넌트를 가정하여 컴포넌트가 예상된 출력을 렌더링하는지 검증하는 테스트를 추가합니다:

```tsx filename="cypress/component/about.cy.tsx"
import Page from '../../app/page'

describe('<Page />', () => {
  it('should render and display expected content', () => {
    // Home 페이지를 위한 React 컴포넌트 마운트
    cy.mount(<Page />)

    // 새 페이지에는 "Home" h1이 포함되어야 함
    cy.get('h1').contains('Home')

    // 예상된 URL을 가진 링크가 존재하는지 검증
    // 링크를 따라가는 것은 E2E 테스트에 더 적합
    cy.get('a[href="/about"]').should('be.visible')
  })
})
```

> **알아두면 좋은 점:**
> - Cypress는 현재 `async` Server Components에 대한 Component Testing을 지원하지 않습니다. E2E 테스팅 사용을 권장합니다.
> - Component 테스트는 Next.js 서버가 필요하지 않으므로 서버가 있어야 하는 `<Image />` 같은 기능이 즉시 작동하지 않을 수 있습니다.

### Component 테스트 실행

터미널에서 `npm run cypress:open`을 실행하여 Cypress를 시작하고 Component Testing 스위트를 실행합니다.

## CI (Continuous Integration)

대화형 테스팅 외에도 CI 환경에 더 적합한 `cypress run` 명령을 사용하여 Cypress를 헤드리스 모드로 실행할 수 있습니다:

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

다음 리소스에서 Cypress와 지속적 통합에 대해 자세히 알아볼 수 있습니다:

- [Next.js with Cypress 예제](https://github.com/vercel/next.js/tree/canary/examples/with-cypress)
- [Cypress 지속적 통합 문서](https://docs.cypress.io/guides/continuous-integration/introduction)
- [Cypress GitHub Actions 가이드](https://on.cypress.io/github-actions)
- [공식 Cypress GitHub Action](https://github.com/cypress-io/github-action)
- [Cypress Discord](https://discord.com/invite/cypress)
