# Fast Refresh

Fast Refresh는 Next.js에 통합된 React 기능으로, 파일을 저장할 때 임시 클라이언트 사이드 상태를 유지하면서 브라우저를 실시간으로 다시 로드할 수 있게 해줍니다. **Next.js 9.4 이상에서 기본적으로 활성화**되어 있으며, 편집 내용은 일반적으로 1초 이내에 표시됩니다.

## 작동 방식

### React 컴포넌트만 있는 파일

**React 컴포넌트만 내보내는** 파일을 편집하면, Fast Refresh는 해당 파일만 업데이트하고 컴포넌트를 다시 렌더링합니다. 스타일, 렌더링 로직, 이벤트 핸들러, 이펙트 등 무엇이든 편집할 수 있습니다.

### 컴포넌트가 아닌 내보내기가 있는 파일

파일이 React가 아닌 값을 내보내면, Fast Refresh는 해당 파일과 이를 가져오는 모든 파일을 다시 실행합니다. 예를 들어, `Button.js`와 `Modal.js` 모두 `theme.js`를 가져오는 경우, `theme.js`를 편집하면 두 컴포넌트 모두 업데이트됩니다.

### React 트리 외부에서 가져온 파일

편집된 파일이 React가 아닌 컴포넌트에서 가져온 경우, Fast Refresh는 **전체 새로고침으로 대체**됩니다. **해결책**: 공유 상수를 별도의 파일로 마이그레이션하고 두 파일 모두에서 가져옵니다.

## 오류 복원력

### 구문 오류

구문 오류는 수정하고 저장하면 자동으로 해제됩니다. **컴포넌트 상태가 유지됩니다** - 수동 새로고침이 필요하지 않습니다.

### 런타임 오류

런타임 오류는 컨텍스트 오버레이를 표시합니다. 오류를 수정하면 새로고침 없이 자동으로 오버레이가 해제됩니다.

**상태 유지**: 렌더링 중에 오류가 발생하지 않으면 유지됩니다. 오류가 발생하면 다시 마운트됩니다.

**Error Boundaries**: 구성된 경우 다음 편집 시 렌더링을 재시도하여 전체 상태 재설정을 방지합니다.

## 제한 사항

다음 경우에는 Fast Refresh 상태 유지가 **보장되지 않습니다**:

- **클래스 컴포넌트**: 함수 컴포넌트와 Hooks만 상태를 유지합니다
- **다중 내보내기**: 파일이 React 컴포넌트 외에 다른 내보내기가 있는 경우
- **고차 컴포넌트**: HOC가 클래스 컴포넌트를 반환하면 상태가 재설정됩니다
- **익명 화살표 함수**: `export default () => <div />;`는 상태 유지를 방지합니다
  - *해결책*: [`name-default-component` codemod](/docs/app/building-your-application/upgrading/codemods#name-default-component) 사용

## 팁 및 모범 사례

### 강제 상태 재설정

파일 어디에나 이 지시문을 추가하여 모든 편집 시 컴포넌트 다시 마운트를 강제할 수 있습니다:

```javascript
// @refresh reset
```

(마운트 전용 애니메이션을 조정하는 데 유용)

### 개발 도구

- 개발 중 컴포넌트에 `console.log` 또는 `debugger;` 문을 추가합니다
- **가져오기 대소문자 구분**: 가져오기가 파일 이름과 정확히 일치하는지 확인합니다 (예: `'./Header'` vs `'./header'`)

## Fast Refresh와 Hooks

### 상태 유지 Hooks

`useState`와 `useRef`는 다음 조건에서 값을 유지합니다:
- 인수가 변경되지 않음
- Hook 호출 순서가 동일하게 유지됨

### 종속성 기반 Hooks (항상 업데이트)

- `useEffect`, `useMemo`, `useCallback`은 **Fast Refresh 중에 항상 다시 실행**됩니다
- Fast Refresh 중에는 종속성이 **무시**됩니다
- 예: `useMemo(() => x * 2, [x])`를 `useMemo(() => x * 10, [x])`로 편집하면 `x`가 변경되지 않아도 다시 실행됩니다

### 모범 사례

가끔 `useEffect`가 다시 실행되는 것에 탄력적인 코드를 작성하세요. 이것은 Fast Refresh와 관계없이 좋은 관행이며 [React Strict Mode](/docs/app/api-reference/config/next-config-js/reactStrictMode)에 의해 강제됩니다 (강력히 권장).
