---
원문: https://nextjs.org/docs/app/api-reference/cli/create-next-app
버전: 16.1.6
---

# create-next-app

`create-next-app`은 Next.js 애플리케이션을 기본 템플릿 또는 GitHub 저장소의 예제로 생성할 수 있는 CLI 도구입니다.

---

## 기본 사용법

```bash
npx create-next-app@latest [project-name] [options]
```

프로젝트 이름 없이 실행하면 대화형 프롬프트가 시작됩니다.

```bash
npx create-next-app@latest
```

---

## 옵션

### 일반 옵션

| 옵션 | 설명 |
|------|------|
| `-h`, `--help` | 사용 가능한 모든 옵션 표시 |
| `-v`, `--version` | 버전 번호 출력 |
| `--no-*` | 기본 옵션 부정 (예: `--no-ts`) |
| `--yes` | 이전 설정 또는 기본값 사용 |
| `--reset-preferences` | 저장된 설정 리셋 |

### 언어 옵션

| 옵션 | 설명 |
|------|------|
| `--ts`, `--typescript` | TypeScript 프로젝트 초기화 (기본값) |
| `--js`, `--javascript` | JavaScript 프로젝트 초기화 |

### 프로젝트 구조 옵션

| 옵션 | 설명 |
|------|------|
| `--app` | App Router 프로젝트 초기화 |
| `--api` | 라우트 핸들러만 포함하는 프로젝트 |
| `--src-dir` | `src/` 디렉토리 내부에 초기화 |
| `--empty` | 빈 프로젝트 초기화 |
| `--import-alias <alias>` | import alias 지정 (기본값: `@/*`) |

### 도구 옵션

| 옵션 | 설명 |
|------|------|
| `--tailwind` | Tailwind CSS 설정 초기화 (기본값) |
| `--eslint` | ESLint 설정 초기화 |
| `--biome` | Biome 설정 초기화 |
| `--no-linter` | 린터 설정 건너뛰기 |
| `--react-compiler` | React Compiler 활성화 |
| `--turbopack` | Turbopack 강제 활성화 (기본값) |
| `--webpack` | Webpack 강제 활성화 |

### 패키지 매니저 옵션

| 옵션 | 설명 |
|------|------|
| `--use-npm` | npm으로 부트스트랩 |
| `--use-pnpm` | pnpm으로 부트스트랩 |
| `--use-yarn` | Yarn으로 부트스트랩 |
| `--use-bun` | Bun으로 부트스트랩 |
| `--skip-install` | 패키지 설치 건너뛰기 |

### 예제 옵션

| 옵션 | 설명 |
|------|------|
| `-e`, `--example [name]` | 예제로 앱 부트스트랩 |
| `--example-path <path>` | 예제 경로 별도 지정 |

### Git 옵션

| 옵션 | 설명 |
|------|------|
| `--disable-git` | Git 초기화 비활성화 |

---

## 대화형 프롬프트

프로젝트 이름만 입력하고 옵션을 지정하지 않으면 대화형 프롬프트가 시작됩니다.

### 기본 프롬프트

```
What is your project named? my-app
Would you like to use the recommended Next.js defaults?
    - Yes, use recommended defaults (TypeScript, ESLint, Tailwind CSS, App Router, Turbopack)
    - No, reuse previous settings
    - No, customize settings
```

### 커스터마이즈 옵션 선택 시

```
Would you like to use TypeScript? No / Yes
Which linter would you like to use? ESLint / Biome / None
Would you like to use React Compiler? No / Yes
Would you like to use Tailwind CSS? No / Yes
Would you like your code inside a `src/` directory? No / Yes
Would you like to use App Router? (recommended) No / Yes
Would you like to customize the import alias (`@/*` by default)? No / Yes
What import alias would you like configured? @/*
```

---

## 린터 옵션

### ESLint

전통적이고 인기 있는 JavaScript 린터입니다. Next.js 전용 `@next/eslint-plugin-next` 플러그인이 포함됩니다.

```bash
npx create-next-app@latest --eslint
```

### Biome

ESLint와 Prettier 기능을 결합한 빠르고 현대적인 린터입니다.

```bash
npx create-next-app@latest --biome
```

### 린터 없음

린터 설정을 건너뜁니다.

```bash
npx create-next-app@latest --no-linter
```

---

## 실제 예제

### 기본 템플릿으로 생성

```bash
npx create-next-app@latest my-app
```

### TypeScript + Tailwind CSS + ESLint (권장 설정)

```bash
npx create-next-app@latest my-app --ts --tailwind --eslint
```

### JavaScript 프로젝트

```bash
npx create-next-app@latest my-app --js
```

### src 디렉토리 사용

```bash
npx create-next-app@latest my-app --src-dir
```

### 공식 Next.js 예제로 생성

[Next.js 예제 저장소](https://github.com/vercel/next.js/tree/canary/examples)에서 예제 이름을 확인할 수 있습니다.

```bash
# with-docker 예제
npx create-next-app@latest my-app --example with-docker

# blog-starter 예제
npx create-next-app@latest my-app --example blog-starter

# with-tailwindcss 예제
npx create-next-app@latest my-app --example with-tailwindcss
```

### GitHub 저장소로 생성

```bash
npx create-next-app@latest my-app --example "https://github.com/user/repo"
```

### 모노레포 내 특정 경로

```bash
npx create-next-app@latest my-app --example "https://github.com/user/repo" --example-path "packages/my-example"
```

### pnpm 사용

```bash
npx create-next-app@latest my-app --use-pnpm
```

### 빈 프로젝트

```bash
npx create-next-app@latest my-app --empty
```

### API 전용 프로젝트

```bash
npx create-next-app@latest my-api --api
```

---

## 기본값

`create-next-app`의 기본 설정은 다음과 같습니다:

| 설정 | 기본값 |
|------|--------|
| 언어 | TypeScript |
| 스타일링 | Tailwind CSS |
| 린터 | ESLint |
| 라우터 | App Router |
| 번들러 | Turbopack |
| import alias | `@/*` |

---

## 환경 변수

| 변수 | 설명 |
|------|------|
| `NEXT_PRIVATE_TEST_VERSION` | 테스트용 특정 버전 지정 |

---

## 중요한 주의사항

> **Good to know**:
> - `--yes` 플래그를 사용하면 이전에 저장된 설정 또는 기본값을 사용합니다
> - `--reset-preferences`로 저장된 설정을 초기화할 수 있습니다
> - Next.js 16부터 Turbopack이 기본 번들러입니다

> **팁**:
> - 프로덕션 프로젝트에는 TypeScript 사용을 권장합니다
> - 팀 프로젝트에서는 린터 설정을 통일하세요
> - 모노레포에서는 `--example-path` 옵션을 활용하세요

---

## 관련 문서

- [next CLI](./next.md)
- [Installation](../../getting-started/01-installation.md)
- [Project Structure](../../getting-started/02-project-structure.md)
