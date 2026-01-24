# 절대 경로 임포트 및 모듈 별칭

Next.js는 `tsconfig.json` 및 `jsconfig.json` 파일의 `baseUrl`과 `paths` 옵션을 내장 지원합니다.

이 옵션들을 사용하면 프로젝트 디렉토리를 절대 경로로 별칭화하여 모듈을 더 쉽게 임포트할 수 있습니다.

## 절대 경로 임포트

`baseUrl` 설정 옵션을 사용하면 프로젝트 루트에서 직접 임포트할 수 있습니다.

### 설정 예시

```json
// tsconfig.json 또는 jsconfig.json
{
  "compilerOptions": {
    "baseUrl": "."
  }
}
```

### 사용 예시

```tsx
// components/button.tsx
export default function Button() {
  return <button>버튼</button>
}
```

```tsx
// pages/index.tsx
// 상대 경로 대신 절대 경로 사용
import Button from 'components/button'

export default function Page() {
  return <Button />
}
```

## 모듈 별칭

`paths` 옵션을 사용하면 모듈 경로에 별칭을 설정할 수 있습니다.

### 설정 예시

```json
// tsconfig.json 또는 jsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["components/*"],
      "@/styles/*": ["styles/*"],
      "@/lib/*": ["lib/*"],
      "@/utils/*": ["utils/*"]
    }
  }
}
```

### 사용 예시

```tsx
// 변경 전
import Button from '../../../components/button'
import styles from '../../../styles/home.module.css'
import { formatDate } from '../../../lib/utils'

// 변경 후
import Button from '@/components/button'
import styles from '@/styles/home.module.css'
import { formatDate } from '@/lib/utils'
```

## src 디렉토리 사용 시

`src` 디렉토리를 사용하는 경우 `baseUrl`을 `src`로 설정합니다:

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/components/*": ["components/*"],
      "@/styles/*": ["styles/*"]
    }
  }
}
```

또는 `baseUrl`을 `.`로 유지하고 경로에 `src`를 포함합니다:

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["src/components/*"],
      "@/styles/*": ["src/styles/*"]
    }
  }
}
```

## 일반적인 패턴

### 단일 별칭

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

이 설정으로 `@/`를 사용하여 `src` 디렉토리의 모든 파일에 접근할 수 있습니다:

```tsx
import Button from '@/components/Button'
import { api } from '@/lib/api'
import styles from '@/styles/page.module.css'
```

### 다중 별칭

더 세분화된 별칭을 설정할 수 있습니다:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["src/components/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/lib/*": ["src/lib/*"],
      "@/styles/*": ["src/styles/*"],
      "@/types/*": ["src/types/*"],
      "@/utils/*": ["src/utils/*"],
      "@/constants/*": ["src/constants/*"]
    }
  }
}
```

## create-next-app 기본 설정

`create-next-app`으로 프로젝트를 생성할 때 import alias 커스터마이징 옵션이 제공됩니다.

기본값:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## 주의사항

1. **IDE 재시작**: `tsconfig.json` 또는 `jsconfig.json`을 수정한 후 IDE를 재시작해야 변경사항이 적용될 수 있습니다.

2. **JavaScript 프로젝트**: TypeScript를 사용하지 않는 경우 `jsconfig.json`을 사용합니다.

3. **경로 일관성**: 별칭 패턴에서 와일드카드(`*`)가 양쪽에 일관되게 사용되어야 합니다.

```json
// ✅ 올바른 설정
{
  "paths": {
    "@/components/*": ["src/components/*"]
  }
}

// ❌ 잘못된 설정
{
  "paths": {
    "@/components": ["src/components/*"]
  }
}
```

4. **Jest 설정**: Jest를 사용하는 경우 `moduleNameMapper`도 설정해야 합니다:

```js
// jest.config.js
module.exports = {
  moduleNameMapper: {
    '^@/components/(.*)$': '<rootDir>/src/components/$1',
    '^@/lib/(.*)$': '<rootDir>/src/lib/$1',
  },
}
```
