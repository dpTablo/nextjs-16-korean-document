# PostCSS

Next.js는 기본적으로 PostCSS를 사용하여 CSS를 컴파일합니다. 별도의 설정 없이도 다양한 CSS 변환이 자동으로 적용됩니다.

## 기본 동작

설정 없이도 다음 변환이 자동으로 적용됩니다:

- **Autoprefixer**: IE11까지 지원하는 벤더 프리픽스 자동 추가
- **Flexbox 버그 수정**: 크로스 브라우저 호환성 보장
- **IE11 호환 CSS 기능 컴파일**:
  - `all` Property
  - Break Properties
  - `font-variant` Property
  - Gap Properties
  - Media Query Ranges

**참고**: CSS Grid와 Custom Properties (CSS 변수)는 IE11용으로 컴파일되지 않습니다.

## CSS Grid IE11 지원

CSS Grid의 IE11 지원을 활성화하려면 파일 상단에 주석을 추가합니다:

```css
/* autoprefixer grid: autoplace */

.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}
```

또는 프로젝트 전체에 적용하려면 `postcss.config.json`을 설정합니다:

```json
{
  "plugins": [
    "postcss-flexbugs-fixes",
    [
      "postcss-preset-env",
      {
        "autoprefixer": {
          "flexbox": "no-2009",
          "grid": "autoplace"
        },
        "stage": 3,
        "features": {
          "custom-properties": false
        }
      }
    ]
  ]
}
```

## 타겟 브라우저 커스터마이징

`package.json`에 `browserslist` 키를 추가하여 타겟 브라우저를 지정할 수 있습니다:

```json
{
  "browserslist": [">0.3%", "not dead", "not op_mini all"]
}
```

[browsersl.ist](https://browsersl.ist/) 도구로 대상 브라우저를 확인할 수 있습니다.

## CSS Modules

CSS Modules는 별도 설정 없이 사용할 수 있습니다. 파일명을 `.module.css`로 변경하면 자동으로 적용됩니다.

## PostCSS 플러그인 커스터마이징

> **주의**: 커스텀 PostCSS 설정 파일을 생성하면 기본 동작이 완전히 비활성화됩니다. 필요한 모든 기능을 수동으로 설정해야 합니다.

### JSON 설정

```json
// postcss.config.json
{
  "plugins": [
    "postcss-flexbugs-fixes",
    [
      "postcss-preset-env",
      {
        "autoprefixer": {
          "flexbox": "no-2009"
        },
        "stage": 3,
        "features": {
          "custom-properties": false
        }
      }
    ]
  ]
}
```

### JavaScript 설정

환경별 조건부 설정이 가능합니다:

```js
// postcss.config.js
module.exports = {
  plugins:
    process.env.NODE_ENV === 'production'
      ? [
          'postcss-flexbugs-fixes',
          [
            'postcss-preset-env',
            {
              autoprefixer: {
                flexbox: 'no-2009',
              },
              stage: 3,
              features: {
                'custom-properties': false,
              },
            },
          ],
        ]
      : [],
}
```

### 객체 형식 설정

다른 도구와의 호환성을 위해 객체 형식을 사용할 수 있습니다:

```js
// postcss.config.js
module.exports = {
  plugins: {
    'postcss-flexbugs-fixes': {},
    'postcss-preset-env': {
      autoprefixer: {
        flexbox: 'no-2009',
      },
      stage: 3,
      features: {
        'custom-properties': false,
      },
    },
  },
}
```

## 추가 플러그인 사용

Tailwind CSS와 같은 추가 플러그인을 사용할 수 있습니다:

```js
// postcss.config.js
module.exports = {
  plugins: {
    'tailwindcss': {},
    'autoprefixer': {},
  },
}
```

### 인기 플러그인

| 플러그인 | 용도 |
|---------|------|
| `postcss-preset-env` | 최신 CSS 기능 사용 |
| `postcss-flexbugs-fixes` | Flexbox 버그 수정 |
| `autoprefixer` | 벤더 프리픽스 자동 추가 |
| `postcss-import` | `@import` 인라인화 |
| `postcss-nested` | 중첩 규칙 지원 (Sass 스타일) |
| `cssnano` | CSS 최소화 |

## 주의사항

1. **플러그인은 문자열로 제공**: `require()` 대신 문자열로 플러그인 이름을 지정하세요.

```js
// ✅ 올바른 방식
module.exports = {
  plugins: ['postcss-preset-env'],
}

// ❌ 잘못된 방식
module.exports = {
  plugins: [require('postcss-preset-env')],
}
```

2. **지원되는 파일명**:
   - `postcss.config.json`
   - `postcss.config.js`
   - `.postcssrc.json`
   - `.postcssrc.js`
   - `package.json`의 `postcss` 키

3. **CSS 변수 사용 주의**: CSS 변수는 안전하게 컴파일할 수 없으므로 필요한 경우 [Sass 변수](https://sass-lang.com/documentation/variables)를 사용하는 것을 권장합니다.
