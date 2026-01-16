# 코딩 에이전트를 위한 Next.js MCP 서버 활성화

**문서 버전:** 16.1.2
**최종 업데이트:** 2026-01-06

---

## 개요

[Model Context Protocol (MCP)](https://modelcontextprotocol.io)는 AI 에이전트와 코딩 어시스턴트가 표준화된 인터페이스를 통해 애플리케이션과 상호작용할 수 있도록 하는 개방형 표준입니다.

Next.js 16+는 코딩 에이전트가 애플리케이션의 내부에 실시간으로 접근할 수 있도록 하는 MCP 지원을 포함합니다. 이 기능을 사용하려면 [`next-devtools-mcp`](https://www.npmjs.com/package/next-devtools-mcp) 패키지를 설치하세요.

---

## 시작하기

**요구사항**: Next.js 16 이상

프로젝트 루트의 `.mcp.json` 파일에 `next-devtools-mcp`를 추가합니다:

```json filename=".mcp.json"
{
  "mcpServers": {
    "next-devtools": {
      "command": "npx",
      "args": ["-y", "next-devtools-mcp@latest"]
    }
  }
}
```

이제 완료되었습니다! 개발 서버를 시작하면 `next-devtools-mcp`가 자동으로 실행 중인 Next.js 인스턴스를 발견하고 연결합니다.

더 많은 구성 옵션은 [next-devtools-mcp 저장소](https://github.com/vercel/next-devtools-mcp)를 참조하세요.

---

## 기능

`next-devtools-mcp`는 코딩 에이전트에게 계속 증가하는 기능 세트를 제공합니다:

### 애플리케이션 런타임 액세스

* **오류 감지**: 개발 서버에서 현재 빌드 오류, 런타임 오류, 타입 오류 검색
* **라이브 상태 쿼리**: 실시간 애플리케이션 상태 및 런타임 정보 접근
* **페이지 메타데이터**: 페이지 라우트, 컴포넌트 및 렌더링 세부사항 쿼리
* **Server Actions**: Server Actions 및 컴포넌트 계층 구조 검사
* **개발 로그**: 개발 서버 로그 및 콘솔 출력 접근

### 개발 도구

* **Next.js 지식 기반**: 포괄적인 Next.js 문서 및 모범 사례 쿼리
* **마이그레이션 및 업그레이드 도구**: Next.js 16으로 업그레이드하기 위한 자동화된 코드모드 도구
* **캐시 컴포넌트 가이드**: Cache Components의 설정 및 구성 지원
* **브라우저 테스팅**: 브라우저에서 페이지를 검증하기 위한 [Playwright MCP](https://github.com/microsoft/playwright-mcp) 통합

> **참고**: Next.js 팀은 적극적으로 이러한 기능을 확장하고 있습니다. 에이전트 개발 경험을 개선하기 위해 새로운 도구와 기능이 정기적으로 추가됩니다.

---

## 개발 워크플로우

1. Next.js 개발 서버 시작:

```bash filename="Terminal"
npm run dev
```

2. 코딩 에이전트가 자동으로 `next-devtools-mcp`를 통해 실행 중인 Next.js 인스턴스에 연결됩니다.

3. 브라우저에서 애플리케이션을 열어 페이지를 확인합니다.

4. 에이전트에 인사이트 및 진단을 요청합니다 (아래 예제 참조).

### 사용 가능한 도구

`next-devtools-mcp`를 통해 에이전트는 다음 도구를 사용할 수 있습니다:

* **`get_errors`**: 개발 서버에서 현재 빌드 오류, 런타임 오류, 타입 오류 검색
* **`get_logs`**: 브라우저 콘솔 로그 및 서버 출력을 포함하는 개발 로그 파일의 경로 가져오기
* **`get_page_metadata`**: 라우트, 컴포넌트 및 렌더링 정보를 포함한 특정 페이지의 메타데이터 가져오기
* **`get_project_metadata`**: 프로젝트 구조, 구성 및 개발 서버 URL 검색
* **`get_routes`**: 파일 시스템을 스캔하여 진입점이 될 모든 라우트 가져오기. 라우트는 라우터 유형(appRouter, pagesRouter)별로 그룹화됩니다. 동적 세그먼트는 `[param]` 또는 `[...slug]` 패턴으로 표시됩니다
* **`get_server_action_by_id`**: ID로 Server Actions 조회하여 소스 파일 및 함수 이름 찾기

---

## 에이전트와 함께 사용

MCP 호환 코딩 에이전트는 자동으로 Next.js 개발 서버를 발견하고 연결하여 프로젝트에 대한 풍부한 컨텍스트를 제공합니다.

### 에이전트 지원 개발의 이점

에이전트는 다음을 수행할 수 있습니다:

* **컨텍스트 인식 제안 제공**: 기존 구조를 기반으로 새 기능을 추가할 올바른 위치 권장
* **라이브 애플리케이션 상태 쿼리**: 개발 중에 현재 구성, 라우트 및 미들웨어 확인
* **앱 라우터 페이지 레이아웃 이해**: 정확히 어떤 페이지와 레이아웃이 렌더링되는지 알기
* **정확한 구현 제공**: 프로젝트의 패턴 및 규칙을 따르는 코드 생성

---

## 예제

### 오류 감지 및 진단

실시간으로 에이전트에게 오류에 대해 물어봅시다:

```bash
User: "현재 애플리케이션에 어떤 오류가 있나요?"
```

에이전트는 다음을 수행합니다:

* `next-devtools-mcp`를 통해 실행 중인 Next.js 애플리케이션 쿼리
* 현재 빌드 오류, 런타임 오류 및 타입 오류 검색
* 오류를 분석하고 실행 가능한 수정 제공

<details>
<summary>에이전트 응답 (클릭하여 확장)</summary>

```bash
> 페이지의 오류를 수정해주세요

⏺ 페이지의 오류를 수정하는 것을 도와드리겠습니다. 먼저 프로젝트의 현재 상태를 확인하여
  어떤 오류가 있는지 확인하겠습니다.

⏺ next-devtools - nextjs_runtime (MCP)(action: "discover_servers")
⏺ next-devtools - nextjs_runtime (MCP)(action: "call_tool", toolName: "get_errors")

⏺ /about 페이지에서 하이드레이션 오류를 발견했습니다. 이 오류는 서버가 "server"를 렌더링하지만
  클라이언트가 "client"를 렌더링하여 불일치가 발생하는 것을 보여줍니다.
  할 일 목록을 만들고 이 문제를 수정하겠습니다.

⏺ 완벽합니다! 이제 페이지의 오류를 확인하겠습니다.

⏺ next-devtools - nextjs_runtime (MCP)(action: "call_tool", toolName: "get_errors")
  ⎿ {
      "success": true,
      "port": 3000,
      "toolName": "get_errors",
      "result": {
        "content": [
          {
            "type": "text",
            "text": "# 1개의 브라우저 세션에서 오류 발견\n\n## 세션: /about\n\n**1개의 오류 발견**\n\n### 런타임 오류\n\n#### 오류 1 (유형: 복구 가능)\n\n**오류**: 하이드레이션 실패"
```

</details>

### 업그레이드 및 모범 사례

Next.js 개념 및 마이그레이션에 대한 도움을 받으세요:

```bash
User: "Next.js 앱을 버전 16으로 업그레이드하는 것을 도와주세요"
```

에이전트는 현재 버전을 분석하고, 코드모드를 사용한 자동화된 마이그레이션을 안내하며, 주요 변경사항 처리에 대한 단계별 지침을 제공합니다.

개념적 질문을 하세요:

```bash
User: "App Router에서 언제 'use client'를 사용해야 하나요?"
```

에이전트는 Next.js 지식 기반을 쿼리하고 코드베이스의 예제와 함께 문서 지원 설명을 제공합니다.

---

## 작동 방식

Next.js 16+는 개발 서버 내에서 실행되는 `/_next/mcp`에 내장 MCP 엔드포인트를 포함합니다. `next-devtools-mcp` 패키지는 자동으로 이 엔드포인트를 발견하고 통신하여 다음을 수행할 수 있습니다:

* 다양한 포트에서 실행 중인 여러 Next.js 인스턴스에 연결
* 도구 호출을 적절한 Next.js 개발 서버로 전달
* 코딩 에이전트를 위한 통합 인터페이스 제공

이 아키텍처는 에이전트 인터페이스를 내부 구현에서 분리하여 `next-devtools-mcp`가 다양한 Next.js 프로젝트에서 원활하게 작동할 수 있습니다.

---

## 문제 해결

### MCP 서버가 연결되지 않음

* Next.js v16 이상을 사용 중인지 확인하세요
* `next-devtools-mcp`가 `.mcp.json`에 구성되어 있는지 확인하세요
* 개발 서버 시작: `npm run dev`
* 개발 서버가 이미 실행 중이면 다시 시작하세요
* 코딩 에이전트가 MCP 서버 구성을 로드했는지 확인하세요
