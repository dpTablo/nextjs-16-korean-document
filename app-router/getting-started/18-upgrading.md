# 업그레이드 (Upgrading)

Next.js를 최신 버전으로 업그레이드하는 방법을 알아봅니다.

## 최신 버전으로 업그레이드

### 업그레이드 명령어 사용

Next.js 16 이상에서는 내장된 업그레이드 명령어를 사용합니다:

```bash
next upgrade
```

### Next.js 15 이하 버전

codemod 패키지를 사용합니다:

```bash
npx @next/codemod@canary upgrade latest
```

### 수동 업그레이드

Next.js, React 및 관련 패키지의 최신 버전을 설치합니다:

**pnpm:**
```bash
pnpm i next@latest react@latest react-dom@latest eslint-config-next@latest
```

**npm:**
```bash
npm i next@latest react@latest react-dom@latest eslint-config-next@latest
```

**yarn:**
```bash
yarn add next@latest react@latest react-dom@latest eslint-config-next@latest
```

**bun:**
```bash
bun add next@latest react@latest react-dom@latest eslint-config-next@latest
```

## Canary 버전

최신 canary 릴리스로 업데이트하려면:

```bash
npm i next@canary
```

> **참고**: canary로 업데이트하기 전에 최신 안정 버전을 사용 중이고 모든 것이 정상 작동하는지 확인하세요.

### Canary에서 사용 가능한 기능

**인증 관련:**
- `forbidden` - 403 에러 함수
- `unauthorized` - 401 에러 함수
- `forbidden.js` - 403 에러 페이지
- `unauthorized.js` - 401 에러 페이지
- `authInterrupts` - 인증 중단 설정

## 버전별 가이드

각 버전별 상세 업그레이드 가이드는 다음을 참조하세요:

- [버전 16 업그레이드](/app-router/guides/upgrading/version-16.md) - 버전 15에서 16으로 업그레이드
- [버전 15 업그레이드](/app-router/guides/upgrading/version-15.md) - 버전 14에서 15로 업그레이드
- [버전 14 업그레이드](/app-router/guides/upgrading/version-14.md) - 버전 13에서 14로 업그레이드
- [Codemods](/app-router/guides/upgrading/codemods.md) - 자동 코드 변환 도구

---

## 참고

- [업그레이드 가이드 상세](/app-router/guides/upgrading/index.md)
- [릴리스 노트](https://github.com/vercel/next.js/releases)
