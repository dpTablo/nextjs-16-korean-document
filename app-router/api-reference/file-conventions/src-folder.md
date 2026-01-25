# src 폴더

`src` 폴더는 `app` 또는 `pages` 디렉토리를 프로젝트 루트에 배치하는 것의 대안입니다. 이 패턴은 애플리케이션 코드를 프로젝트 설정 파일과 분리하며, 많은 개발자와 팀이 선호하는 방식입니다.

## 설정

`src` 폴더를 사용하려면 라우팅 디렉토리를 이동합니다:
- **App Router**: `app`을 `src/app`으로 이동
- **Pages Router**: `pages`를 `src/pages`로 이동

### 폴더 구조 예제

```
my-project/
├── src/
│   ├── app/           ← 라우팅 폴더
│   ├── components/    ← 컴포넌트
│   └── lib/           ← 유틸리티
├── public/            ← 정적 파일 (루트에 유지)
├── next.config.js     ← 설정 (루트에 유지)
├── package.json
└── tsconfig.json
```

## 중요 가이드라인

### 루트에 유지해야 하는 파일

다음 파일들은 프로젝트 루트에 남겨두어야 합니다:
- `/public` 디렉토리
- `package.json`
- `next.config.js`
- `tsconfig.json`
- `.env.*` 파일

### src로 이동하는 파일

- `app` 또는 `pages` (라우팅 폴더)
- `/components`나 `/lib` 같은 다른 애플리케이션 폴더
- 프록시 설정 (사용하는 경우)

## 주요 고려사항

### 우선순위 규칙

`app`/`pages`가 루트에 존재하고 **동시에** `src/app`/`src/pages`도 존재하는 경우, 루트 버전이 우선하며 `src` 버전은 무시됩니다.

```
my-project/
├── app/           ← 이것이 사용됨 (루트가 우선)
├── src/
│   └── app/       ← 이것은 무시됨
└── package.json
```

### Tailwind CSS 설정

Tailwind CSS를 사용하는 경우, `tailwind.config.js`의 `content` 섹션에 `/src` 접두사를 추가합니다:

```js
// tailwind.config.js
module.exports = {
  content: [
    './src/**/*.{js,ts,jsx,tsx}'
  ],
  // ...
}
```

### TypeScript 경로 별칭

TypeScript를 사용하는 경우, `tsconfig.json`의 경로에 `src/`를 포함하도록 업데이트합니다:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

이렇게 하면 import 시 `@/` 별칭을 사용할 수 있습니다:

```tsx
// src/app/page.tsx
import { Button } from '@/components/Button'
import { formatDate } from '@/lib/utils'
```

## 장점

1. **깔끔한 프로젝트 루트**: 설정 파일과 애플리케이션 코드가 분리됩니다
2. **표준화된 구조**: 많은 프레임워크와 라이브러리에서 일반적으로 사용되는 패턴입니다
3. **코드 탐색 용이**: 애플리케이션 로직이 한 곳에 집중됩니다

## 단점

1. **추가 폴더 깊이**: 파일 경로가 한 단계 더 깊어집니다
2. **설정 업데이트 필요**: Tailwind, TypeScript 등의 설정을 업데이트해야 합니다

---

## 참고

- [프로젝트 구조](/app-router/getting-started/02-project-structure.md)
- [TypeScript 가이드](/app-router/guides/typescript.md)
- [Tailwind CSS 가이드](/app-router/guides/tailwind-css.md)
