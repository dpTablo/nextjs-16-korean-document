# CSS Modules

Next.js는 `.module.css` 확장자를 사용하는 CSS Modules를 기본으로 지원합니다.

CSS Modules는 고유한 클래스명을 자동으로 생성하여 CSS를 로컬 범위로 제한합니다. 이를 통해 다른 파일에서 동일한 클래스명을 사용해도 충돌 걱정 없이 스타일을 적용할 수 있습니다.

## 기본 사용법

### CSS 파일 생성

```css
/* styles/blog.module.css */
.blog {
  padding: 24px;
}

.title {
  font-size: 2rem;
  font-weight: bold;
  color: #333;
}

.description {
  color: #666;
  line-height: 1.6;
}
```

### 컴포넌트에서 사용

```tsx
// pages/blog/index.tsx
import styles from '../../styles/blog.module.css'

export default function BlogPage() {
  return (
    <main className={styles.blog}>
      <h1 className={styles.title}>블로그</h1>
      <p className={styles.description}>블로그 글 목록입니다.</p>
    </main>
  )
}
```

## 여러 클래스 적용

여러 클래스를 적용할 때는 템플릿 리터럴이나 배열 조인을 사용할 수 있습니다:

```tsx
import styles from './button.module.css'

export function Button({ primary, children }) {
  return (
    <button
      className={`${styles.button} ${primary ? styles.primary : styles.secondary}`}
    >
      {children}
    </button>
  )
}
```

또는 `clsx`나 `classnames` 같은 라이브러리를 사용할 수 있습니다:

```tsx
import clsx from 'clsx'
import styles from './button.module.css'

export function Button({ primary, disabled, children }) {
  return (
    <button
      className={clsx(styles.button, {
        [styles.primary]: primary,
        [styles.disabled]: disabled,
      })}
    >
      {children}
    </button>
  )
}
```

## 컴포넌트와 함께 위치시키기

컴포넌트 파일과 같은 디렉토리에 CSS Module 파일을 위치시킬 수 있습니다:

```
components/
├── Button/
│   ├── Button.tsx
│   └── Button.module.css
├── Card/
│   ├── Card.tsx
│   └── Card.module.css
```

```tsx
// components/Button/Button.tsx
import styles from './Button.module.css'

export function Button({ children }) {
  return <button className={styles.button}>{children}</button>
}
```

## 전역 클래스와 함께 사용

CSS Modules 내에서 전역 클래스를 사용해야 할 때는 `:global()` 구문을 사용합니다:

```css
/* styles/component.module.css */
.container {
  padding: 20px;
}

/* 전역 클래스 선택 */
.container :global(.external-class) {
  color: blue;
}

/* 전역 클래스 정의 */
:global(.global-class) {
  font-size: 14px;
}
```

## Composition (합성)

CSS Modules의 `composes` 키워드를 사용하여 스타일을 합성할 수 있습니다:

```css
/* styles/base.module.css */
.baseButton {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
```

```css
/* styles/button.module.css */
.primary {
  composes: baseButton from './base.module.css';
  background-color: blue;
  color: white;
}

.secondary {
  composes: baseButton from './base.module.css';
  background-color: gray;
  color: black;
}
```

## TypeScript 지원

TypeScript 프로젝트에서 CSS Modules를 사용할 때 타입 정의를 추가할 수 있습니다.

### 전역 타입 선언

```typescript
// globals.d.ts 또는 next-env.d.ts
declare module '*.module.css' {
  const classes: { readonly [key: string]: string }
  export default classes
}

declare module '*.module.scss' {
  const classes: { readonly [key: string]: string }
  export default classes
}
```

### typed-css-modules 사용 (선택사항)

더 정확한 타입 체크를 원한다면 `typed-css-modules`를 사용할 수 있습니다:

```bash
npm install -D typed-css-modules
```

## 장점

1. **이름 충돌 방지**: 자동으로 고유한 클래스명이 생성됩니다.
2. **로컬 스코프**: 스타일이 해당 컴포넌트에만 적용됩니다.
3. **명시적 의존성**: import 구문으로 스타일 의존성이 명확합니다.
4. **Dead Code 제거**: 사용하지 않는 CSS가 빌드에서 제외됩니다.
5. **개발자 경험**: IDE에서 자동 완성을 지원합니다.

## 주의사항

- CSS Modules 파일은 반드시 `.module.css` 확장자를 사용해야 합니다.
- 클래스명에 하이픈(-)을 사용할 경우 대괄호 표기법을 사용해야 합니다:

```tsx
// button-primary 같은 클래스명
<button className={styles['button-primary']}>버튼</button>
```

- camelCase 클래스명을 사용하면 점 표기법으로 접근할 수 있습니다:

```css
.buttonPrimary {
  /* ... */
}
```

```tsx
<button className={styles.buttonPrimary}>버튼</button>
```
