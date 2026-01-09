# 디버깅

**버전:** 16.1.1

## 개요
이 문서는 VS Code, Chrome DevTools 또는 Firefox DevTools를 사용하여 완전한 소스 맵 지원으로 Next.js 프론트엔드 및 백엔드 코드를 디버깅하는 방법을 다룹니다. Node.js에 연결할 수 있는 모든 디버거는 Next.js 애플리케이션을 디버깅할 수 있습니다.

## VS Code 디버깅 설정

프로젝트 루트에 `.vscode/launch.json` 생성:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js: debug server-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev -- --inspect"
    },
    {
      "name": "Next.js: debug client-side",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000"
    },
    {
      "name": "Next.js: debug full stack",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/next/dist/bin/next",
      "runtimeArgs": ["--inspect"],
      "skipFiles": ["<node_internals>/**"],
      "serverReadyAction": {
        "action": "debugWithEdge",
        "killOnServerStop": true,
        "pattern": "- Local:.+(https?://.+)",
        "uriFormat": "%s",
        "webRoot": "${workspaceFolder}"
      }
    }
  ]
}
```

**디버깅 시작:** 디버그 패널로 이동 (`Ctrl+Shift+D` / `⇧+⌘+D`), 구성 선택, `F5` 누르기.

**참고:** Firefox 디버깅은 [Firefox Debugger 확장](https://marketplace.visualstudio.com/items?itemName=firefox-devtools.vscode-firefox-debug)이 필요합니다.

## JetBrains WebStorm 디버깅

1. 런타임 구성 드롭다운 클릭
2. `Edit Configurations...` 선택
3. `http://localhost:3000`을 URL로 하는 `JavaScript Debug` 구성 생성
4. 구성을 실행하여 자동으로 브라우저 열기

## 브라우저 DevTools 디버깅

### 클라이언트 사이드 코드

**Chrome:**
- DevTools 열기: `Ctrl+Shift+J` (Windows/Linux) 또는 `⌥+⌘+I` (macOS)
- **Sources** 탭으로 이동
- 파일 검색: `Ctrl+P` / `⌘+P`
- 소스 파일은 `webpack://_N_E/./`로 시작하는 경로로 나타남

**Firefox:**
- DevTools 열기: `Ctrl+Shift+I` (Windows/Linux) 또는 `⌥+⌘+I` (macOS)
- **Debugger** 탭으로 이동
- 왼쪽 패널의 파일 트리 사용

**디버깅 트리거:** 코드 실행은 `debugger` 문 또는 수동으로 설정한 중단점에서 일시 중지됩니다.

### React Developer Tools

[React Developer Tools](https://react.dev/learn/react-developer-tools) 브라우저 확장 프로그램을 설치하여:
- React 컴포넌트 검사
- Props 및 state 편집
- 성능 문제 식별

### 서버 사이드 코드

개발 서버에 `--inspect` 플래그 전달:

```bash
next dev --inspect
```

출력 예시:
```
Debugger listening on ws://127.0.0.1:9229/0cf90313-350d-4466-a748-cd60f4e47c95
```

**Chrome:**
1. `chrome://inspect` 방문
2. **Remote Target** 섹션에서 앱 찾기
3. **inspect** 클릭
4. **Sources** 탭으로 이동

**Firefox:**
1. `about:debugging` 방문
2. **This Firefox** 클릭
3. **Remote Targets** 아래에서 앱 찾기
4. **Inspect** 클릭
5. **Debugger** 탭으로 이동

**원격 디버깅:** localhost 외부 접근을 위해 `--inspect=0.0.0.0` 사용 (예: Docker 컨테이너).

**고급 옵션:** `--inspect-brk` 또는 `--inspect-wait`를 위해 `NODE_OPTIONS=--inspect-brk next dev` 사용.

## 에러 검사

Next.js는 에러 오버레이의 버전 표시기 아래에 Node.js 아이콘을 표시합니다. 클릭하면 서버 프로세스를 검사하기 위한 DevTools URL이 복사됩니다.

## 문제 해결

**Windows 성능 문제:** Windows Defender를 비활성화하세요. 모든 파일 읽기를 확인하여 `next dev`의 Fast Refresh 시간에 상당한 영향을 미칩니다.

## 주요 단축키

| 작업 | Windows/Linux | macOS |
|--------|---------------|-------|
| VS Code 디버그 열기 | Ctrl+Shift+D | ⇧+⌘+D |
| Chrome DevTools | Ctrl+Shift+J | ⌥+⌘+I |
| Firefox DevTools | Ctrl+Shift+I | ⌥+⌘+I |
| 파일 검색 | Ctrl+P | ⌘+P |

---

## 일반적인 디버깅 시나리오

### 1. 서버 컴포넌트 디버깅

```tsx
// app/page.tsx
export default async function Page() {
  debugger // VS Code에서 여기서 중단됨
  const data = await fetchData()
  return <div>{data}</div>
}
```

**VS Code 설정:** "Next.js: debug server-side" 사용

### 2. 클라이언트 컴포넌트 디버깅

```tsx
'use client'

export default function ClientComponent() {
  const handleClick = () => {
    debugger // 브라우저 DevTools에서 여기서 중단됨
    console.log('Clicked')
  }

  return <button onClick={handleClick}>클릭</button>
}
```

**브라우저 설정:** Chrome/Firefox DevTools 사용

### 3. API Route 디버깅

```ts
// app/api/route.ts
export async function GET() {
  debugger // 서버 사이드 디버거에서 중단됨
  const data = await fetchData()
  return Response.json(data)
}
```

**설정:** `next dev --inspect` 사용

### 4. Full Stack 디버깅

**VS Code 설정:** "Next.js: debug full stack" 구성 사용
- 서버와 클라이언트 모두 디버깅
- 중단점 설정 가능
- 변수 검사 가능

---

## 추가 리소스

- [Node.js 디버깅 가이드](https://nodejs.org/en/docs/guides/debugging-getting-started/)
- [VS Code Node.js 디버깅: 중단점](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_breakpoints)
- [Chrome DevTools: JavaScript 디버그](https://developers.google.com/web/tools/chrome-devtools/javascript)
- [Firefox DevTools: 디버거](https://firefox-source-docs.mozilla.org/devtools-user/debugger/)
