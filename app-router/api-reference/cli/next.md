# next CLI

Next.js CLI는 애플리케이션의 개발, 빌드, 실행 등을 관리합니다.

---

## 기본 사용법

```bash
npx next [command] [options]
```

> **참고**: `next` 명령어만 실행하면 `next dev`의 별칭으로 동작합니다.

---

## 전역 옵션

| 옵션 | 설명 |
|------|------|
| `-h`, `--help` | 사용 가능한 모든 옵션 표시 |
| `-v`, `--version` | Next.js 버전 번호 출력 |

---

## 명령어 목록

| 명령어 | 설명 |
|--------|------|
| `dev` | Hot Module Reloading, 에러 리포팅 등과 함께 개발 모드 시작 |
| `build` | 최적화된 프로덕션 빌드 생성 |
| `start` | 프로덕션 모드 시작 (`next build` 필요) |
| `info` | 시스템 정보 출력 (버그 리포트용) |
| `telemetry` | 익명 텔레메트리 수집 활성화/비활성화 |
| `typegen` | 전체 빌드 없이 TypeScript 정의 생성 |
| `upgrade` | Next.js를 최신 버전으로 업그레이드 |
| `experimental-analyze` | Turbopack을 사용한 번들 분석 |

---

## next dev

개발 모드로 애플리케이션을 시작합니다.

```bash
next dev [directory] [options]
```

### 옵션

| 옵션 | 설명 |
|------|------|
| `[directory]` | 빌드 디렉토리 (기본값: 현재 디렉토리) |
| `--turbopack` | Turbopack 강제 활성화 (기본값) |
| `--webpack` | Webpack 번들러 사용 |
| `-p`, `--port <port>` | 포트 번호 지정 (기본값: 3000) |
| `-H`, `--hostname <hostname>` | 호스트명 지정 (기본값: 0.0.0.0) |
| `--experimental-https` | HTTPS 서버 시작 (자체 서명 인증서) |
| `--experimental-https-key <path>` | HTTPS 키 파일 경로 |
| `--experimental-https-cert <path>` | HTTPS 인증서 파일 경로 |
| `--experimental-https-ca <path>` | HTTPS CA 파일 경로 |
| `--experimental-upload-trace <url>` | 디버깅 추적을 원격 URL로 리포트 |
| `--experimental-cpu-prof` | V8 인스펙터를 사용한 CPU 프로파일링 |

### 예제

```bash
# 기본 개발 서버 시작
next dev

# 포트 4000으로 시작
next dev -p 4000

# HTTPS로 시작
next dev --experimental-https

# Webpack 사용
next dev --webpack
```

---

## next build

최적화된 프로덕션 빌드를 생성합니다.

```bash
next build [directory] [options]
```

### 빌드 출력 예시

```bash
Route (app)
┌ ○ /_not-found
└ ƒ /products/[id]

○  (Static)   정적 콘텐츠로 프리렌더됨
ƒ  (Dynamic)  요청 시 서버에서 렌더링됨
```

### 옵션

| 옵션 | 설명 |
|------|------|
| `[directory]` | 빌드 디렉토리 (기본값: 현재 디렉토리) |
| `--turbopack` | Turbopack 강제 활성화 (기본값) |
| `--webpack` | Webpack 사용 |
| `-d`, `--debug` | 상세한 빌드 출력 |
| `--profile` | React 프로파일링 활성화 |
| `--no-lint` | 린팅 비활성화 |
| `--no-mangling` | Name mangling 비활성화 |
| `--experimental-app-only` | App Router 라우트만 빌드 |
| `--experimental-build-mode [mode]` | 빌드 모드 (default/compile/generate) |
| `--debug-prerender` | 프리렌더 에러 디버그 |
| `--debug-build-paths=<patterns>` | 특정 라우트만 빌드 |
| `--experimental-cpu-prof` | CPU 프로파일링 활성화 |

### 예제

```bash
# 기본 빌드
next build

# 디버그 모드로 빌드
next build --debug

# 특정 라우트만 빌드
next build --debug-build-paths="app/page.tsx"

# 여러 라우트 빌드
next build --debug-build-paths="app/page.tsx,pages/index.tsx"

# Glob 패턴으로 빌드
next build --debug-build-paths="app/**/page.tsx"
```

---

## next start

프로덕션 모드로 애플리케이션을 시작합니다. `next build`로 빌드한 후 실행해야 합니다.

```bash
next start [directory] [options]
```

### 옵션

| 옵션 | 설명 |
|------|------|
| `[directory]` | 시작 디렉토리 (기본값: 현재 디렉토리) |
| `-p`, `--port <port>` | 포트 번호 (기본값: 3000) |
| `-H`, `--hostname <hostname>` | 호스트명 (기본값: 0.0.0.0) |
| `--keepAliveTimeout <ms>` | 비활성 연결 종료 대기 시간 (밀리초) |
| `--experimental-cpu-prof` | CPU 프로파일링 활성화 |

### 예제

```bash
# 기본 프로덕션 서버 시작
next start

# 포트 8080으로 시작
next start -p 8080

# 다운스트림 프록시 타임아웃 설정
next start --keepAliveTimeout 70000
```

---

## next info

현재 시스템 정보를 출력합니다. 버그 리포트 시 유용합니다.

```bash
next info [options]
```

### 출력 정보

- 운영체제 (플랫폼, 아키텍처, 버전, 메모리, CPU 코어)
- 바이너리 (Node.js, npm, Yarn, pnpm)
- 패키지 버전 (next, react, react-dom, typescript 등)
- Next.js 설정

### 옵션

| 옵션 | 설명 |
|------|------|
| `--verbose` | 디버깅용 추가 정보 수집 |

---

## next telemetry

익명 텔레메트리 수집을 제어합니다.

```bash
next telemetry [options]
```

### 옵션

| 옵션 | 설명 |
|------|------|
| `--enable` | 텔레메트리 수집 활성화 |
| `--disable` | 텔레메트리 수집 비활성화 |

---

## next typegen

전체 빌드 없이 TypeScript 타입 정의를 생성합니다.

```bash
next typegen [directory] [options]
```

### 사용 사례

```bash
# 라우트 타입 생성 후 TypeScript 검증
next typegen && tsc --noEmit

# CI 워크플로우에서 빌드 없이 타입 검사
next typegen && npm run type-check
```

### 출력 위치

`<distDir>/types` (일반적으로 `.next/dev/types` 또는 `.next/types`)

생성되는 `next-env.d.ts` 파일은 `.gitignore`에 추가하는 것이 좋습니다.

---

## next upgrade

Next.js를 최신 버전으로 업그레이드합니다.

```bash
next upgrade [directory] [options]
```

### 옵션

| 옵션 | 설명 |
|------|------|
| `[directory]` | 업그레이드할 앱 디렉토리 (기본값: 현재 디렉토리) |
| `--revision <revision>` | 버전/태그 지정 (예: latest, canary, 15.0.0) |
| `--verbose` | 업그레이드 과정 상세 출력 |

### 예제

```bash
# 최신 버전으로 업그레이드
next upgrade

# 특정 버전으로 업그레이드
next upgrade --revision 16.0.0

# canary 버전으로 업그레이드
next upgrade --revision canary
```

---

## next experimental-analyze

Turbopack을 사용하여 번들을 분석합니다. 빌드 아티팩트를 생성하지 않습니다.

```bash
next experimental-analyze [directory] [options]
```

### 기능

- 라우트별 번들 필터링 및 클라이언트/서버 뷰 전환
- 모듈 포함 이유 표시 (전체 임포트 체인)
- 서버-클라이언트 컴포넌트 경계 및 동적 임포트 추적

### 옵션

| 옵션 | 설명 |
|------|------|
| `[directory]` | 분석 디렉토리 (기본값: 현재 디렉토리) |
| `--no-mangling` | Name mangling 비활성화 |
| `--profile` | React 프로파일링 활성화 |
| `-o`, `--output` | 분석 파일을 디스크에 저장 (`.next/diagnostics/analyze`) |
| `--port <port>` | 분석기 포트 (기본값: 4000) |

---

## 실용적인 팁

### 포트 변경

```bash
# -p 옵션 사용
next dev -p 4000

# PORT 환경변수 사용
PORT=4000 next dev
```

> **참고**: `.env` 파일에서 PORT를 설정할 수 없습니다. HTTP 서버 부팅이 다른 코드 초기화보다 먼저 진행되기 때문입니다.

### 개발 중 HTTPS 사용

```bash
# 자체 서명 인증서 자동 생성
next dev --experimental-https

# 커스텀 인증서 사용
next dev --experimental-https \
  --experimental-https-key ./certificates/localhost-key.pem \
  --experimental-https-cert ./certificates/localhost.pem
```

### 프리렌더 에러 디버깅

```bash
next build --debug-prerender
```

활성화되는 기능:
- 서버 코드 미니피케이션 비활성화
- 서버 번들 소스맵 생성
- 프리렌더 소스맵 소비 활성화
- 첫 번째 에러 후에도 빌드 계속 진행

> **경고**: 프로덕션 배포에 사용하지 마세요.

### Node.js 인자 전달

```bash
NODE_OPTIONS='--throw-deprecation' next
NODE_OPTIONS='-r esm' next
NODE_OPTIONS='--inspect' next
```

### CPU 프로파일링

```bash
# 빌드 프로파일
next build --experimental-cpu-prof

# 개발 서버 프로파일 (Ctrl+C로 저장)
next dev --experimental-cpu-prof

# 프로덕션 서버 프로파일
next start --experimental-cpu-prof
```

프로파일 출력 위치: `.next/cpu-profiles/`

Chrome DevTools의 Performance 탭에서 "Load profile"로 분석할 수 있습니다.

---

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v16.1.0 | `next experimental-analyze` 명령어 추가 |
| v16.0.0 | `next build`에서 JS 번들 크기 메트릭 제거 |
| v15.5.0 | `next typegen` 명령어 추가 |
| v15.4.0 | `next build`에 `--debug-prerender` 옵션 추가 |

---

## 관련 문서

- [create-next-app](./create-next-app.md)
- [Deploying](../../getting-started/12-deploying.md)
- [Environment Variables](../../guides/environment-variables.md)
