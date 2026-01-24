# 라우트 그룹 (Route Groups)

라우트 그룹은 카테고리나 팀별로 라우트를 구성할 수 있게 해주는 폴더 규칙입니다.

## 규칙

라우트 그룹은 폴더 이름을 괄호로 감싸서 생성할 수 있습니다: `(folderName)`.

이 규칙은 폴더가 구조적 목적으로만 사용되며 라우트의 URL 경로에 **포함되지 않아야** 함을 나타냅니다.

```
app/
├── (marketing)/
│   ├── about/
│   │   └── page.js      → /about
│   └── blog/
│       └── page.js      → /blog
├── (shop)/
│   ├── cart/
│   │   └── page.js      → /cart
│   └── products/
│       └── page.js      → /products
└── layout.js
```

## 사용 사례

- 팀, 관심사 또는 기능별로 라우트를 구성
- 여러 [루트 레이아웃](/app-router/api-reference/file-conventions/layout.md#root-layout) 정의
- 특정 라우트 세그먼트는 레이아웃을 공유하면서 다른 세그먼트는 제외

### 예제: 여러 루트 레이아웃

여러 루트 레이아웃을 만들려면 최상위 `layout.js` 파일을 제거하고 각 라우트 그룹 내에 `layout.js` 파일을 추가합니다. 이는 완전히 다른 UI나 경험을 가진 섹션으로 애플리케이션을 분할하는 데 유용합니다.

```
app/
├── (marketing)/
│   ├── layout.js        → 마케팅용 루트 레이아웃
│   └── page.js
├── (shop)/
│   ├── layout.js        → 쇼핑용 루트 레이아웃
│   └── page.js
```

### 예제: 특정 세그먼트를 레이아웃에 포함

특정 라우트를 레이아웃에 포함하려면 새 라우트 그룹(예: `(shop)`)을 만들고 동일한 레이아웃을 공유하는 라우트를 해당 그룹으로 이동합니다(예: `account` 및 `cart`). 그룹 외부의 라우트는 레이아웃을 공유하지 않습니다(예: `checkout`).

```
app/
├── (shop)/
│   ├── account/
│   │   └── page.js      → 레이아웃 공유
│   ├── cart/
│   │   └── page.js      → 레이아웃 공유
│   └── layout.js        → 공유 레이아웃
└── checkout/
    └── page.js          → 레이아웃 공유 안 함
```

## 주의사항

- **전체 페이지 로드**: 서로 다른 루트 레이아웃을 사용하는 라우트 간에 이동하면 전체 페이지 새로고침이 트리거됩니다. 예를 들어, `app/(shop)/layout.js`를 사용하는 `/cart`에서 `app/(marketing)/layout.js`를 사용하는 `/blog`로 이동하면 전체 페이지 새로고침이 발생합니다. 이는 **여러 루트 레이아웃에만** 적용됩니다.

- **충돌하는 경로**: 서로 다른 그룹의 라우트가 동일한 URL 경로로 해석되어서는 안 됩니다. 예를 들어, `(marketing)/about/page.js`와 `(shop)/about/page.js`는 모두 `/about`으로 해석되어 에러가 발생합니다.

- **최상위 루트 레이아웃**: 최상위 `layout.js` 파일 없이 여러 루트 레이아웃을 사용하는 경우, 홈 라우트(/)가 라우트 그룹 중 하나에 정의되어 있는지 확인하세요. 예: `app/(marketing)/page.js`
