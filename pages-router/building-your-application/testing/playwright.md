# Playwright

Playwright는 Chromium, Firefox, WebKit을 단일 API로 자동화하는 테스트 프레임워크로, **End-to-End (E2E) 테스트**를 작성하는 데 사용됩니다.

## 빠른 시작

```bash
npx create-next-app@latest --example with-playwright with-playwright-app
```

## 수동 설정

### Playwright 설치

```bash
npm init playwright
# 또는
yarn create playwright
# 또는
pnpm create playwright
```

이 명령어는 설정 프롬프트를 진행하며 `playwright.config.ts` 파일을 자동 생성합니다.

## 첫 번째 E2E 테스트 작성

### Next.js 페이지 생성

```tsx
// pages/index.tsx
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

```tsx
// pages/about.tsx
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

### 테스트 작성

```ts
// tests/example.spec.ts
import { test, expect } from '@playwright/test'

test('should navigate to the about page', async ({ page }) => {
  // 인덱스 페이지에서 시작
  await page.goto('http://localhost:3000/')

  // 'About' 텍스트가 있는 요소를 찾아 클릭
  await page.click('text=About')

  // 새 URL이 "/about"이어야 함
  await expect(page).toHaveURL('http://localhost:3000/about')

  // 새 페이지에 "About"이 포함된 h1이 있어야 함
  await expect(page.locator('h1')).toContainText('About')
})
```

### baseURL 설정

`playwright.config.ts`에 `baseURL`을 설정하면 상대 경로를 사용할 수 있습니다:

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test'

export default defineConfig({
  use: {
    baseURL: 'http://localhost:3000',
  },
})
```

```ts
// 테스트에서 상대 경로 사용
await page.goto('/')  // http://localhost:3000/ 으로 이동
```

## 테스트 실행

### 로컬 실행

```bash
# 프로덕션 빌드 및 서버 시작
npm run build
npm run start

# 다른 터미널에서 테스트 실행
npx playwright test
```

Playwright는 **Chromium, Firefox, WebKit** 세 가지 브라우저로 테스트를 수행합니다.

> **권장사항**: 프로덕션 코드에 대해 테스트하여 실제 동작에 더 가깝게 검증합니다.

### webServer 설정

Playwright가 개발 서버를 자동으로 시작하도록 설정할 수 있습니다:

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test'

export default defineConfig({
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
  use: {
    baseURL: 'http://localhost:3000',
  },
})
```

## 일반적인 테스트 패턴

### 요소 찾기

```ts
// 텍스트로 찾기
await page.click('text=Submit')

// CSS 선택자로 찾기
await page.click('button.primary')

// data-testid로 찾기
await page.click('[data-testid="submit-button"]')

// 역할로 찾기
await page.getByRole('button', { name: 'Submit' }).click()
```

### 폼 작성

```ts
await page.fill('input[name="email"]', 'test@example.com')
await page.fill('input[name="password"]', 'password123')
await page.click('button[type="submit"]')
```

### 응답 대기

```ts
// 네비게이션 대기
await page.waitForNavigation()

// 특정 요소 대기
await page.waitForSelector('.success-message')

// 네트워크 요청 대기
await page.waitForResponse('**/api/users')
```

### 스크린샷 캡처

```ts
await page.screenshot({ path: 'screenshot.png' })
await page.screenshot({ path: 'fullpage.png', fullPage: true })
```

## CI/CD 환경에서 실행

### 의존성 설치

```bash
npx playwright install-deps
```

### Headless 모드

Playwright는 CI 환경에서 기본적으로 headless 모드로 테스트를 실행합니다.

### GitHub Actions 예시

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests
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
      - name: Install dependencies
        run: npm ci
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      - name: Build
        run: npm run build
      - name: Run Playwright tests
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## 디버깅

### UI 모드

```bash
npx playwright test --ui
```

### Debug 모드

```bash
npx playwright test --debug
```

### 브라우저 표시

```bash
npx playwright test --headed
```

## 추가 리소스

- [Next.js with Playwright 예제](https://github.com/vercel/next.js/tree/canary/examples/with-playwright)
- [Playwright 공식 문서](https://playwright.dev/docs/intro)
- [Playwright CI 설정](https://playwright.dev/docs/ci)
