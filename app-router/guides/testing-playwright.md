# Playwright로 Next.js 테스트하기

**버전:** 16.1.1

## 개요
Playwright는 단일 API로 Chromium, Firefox, WebKit 브라우저를 자동화하는 테스팅 프레임워크로 E2E(End-to-End) 테스트를 가능하게 합니다.

## 빠른 설정 옵션

### 옵션 1: create-next-app 사용 (가장 빠름)
```bash
npx create-next-app@latest --example with-playwright with-playwright-app
```

### 옵션 2: 수동 설정
```bash
npm init playwright
# 또는
yarn create playwright
# 또는
pnpm create playwright
```
이는 `playwright.config.ts` 설정 파일을 자동으로 생성합니다.

---

## 첫 번째 테스트 작성하기

### 1단계: Next.js 페이지 생성

**홈 페이지** (`app/page.tsx`):
```tsx
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

**About 페이지** (`app/about/page.tsx`):
```tsx
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

### 2단계: 테스트 파일 작성

**`tests/example.spec.ts`**:
```ts
import { test, expect } from '@playwright/test'

test('should navigate to the about page', async ({ page }) => {
  await page.goto('http://localhost:3000/')
  await page.click('text=About')
  await expect(page).toHaveURL('http://localhost:3000/about')
  await expect(page.locator('h1')).toContainText('About')
})
```

**팁**: `playwright.config.ts`에서 `baseURL`을 사용하여 URL을 단순화하세요:
```ts
"baseURL": "http://localhost:3000"
```
그런 다음: `page.goto("/")`

---

## 테스트 실행

### 로컬 개발
```bash
npm run build
npm run start
# 다른 터미널에서:
npx playwright test
```

### webServer 기능 사용
Playwright가 자동으로 개발 서버를 시작하도록 하세요:
```ts
// playwright.config.ts에서
webServer: { ... }
```

### CI/CD 환경
```bash
npx playwright install-deps  # 브라우저 의존성 설치
npx playwright test          # 기본적으로 헤드리스 모드로 실행
```

---

## 리소스
- [Next.js Playwright 예시](https://github.com/vercel/next.js/tree/canary/examples/with-playwright)
- [Playwright CI 문서](https://playwright.dev/docs/ci)
- [Playwright Discord 커뮤니티](https://discord.com/invite/playwright-807756831384403968)
