# appDir

> **레거시 API**: 이 옵션은 더 이상 권장되지 않습니다. 하위 호환성을 위해 계속 지원됩니다.

## 개요

`appDir` 옵션은 Next.js 13.4 이전에 App Router를 활성화하는 데 사용되었던 실험적 옵션입니다.

**Next.js 13.4 이후로는 이 옵션이 더 이상 필요하지 않습니다.** App Router가 안정적으로 완성되었기 때문입니다.

## App Router 기능

`app` 디렉토리를 사용하면 다음 기능이 지원됩니다:

1. **레이아웃** (Layouts) - 페이지 간 UI 공유
2. **서버 컴포넌트** (Server Components) - 서버에서 렌더링되는 컴포넌트
3. **스트리밍** (Streaming) - 점진적 UI 렌더링
4. **함께 배치된 데이터 페칭** (Colocated data fetching) - 컴포넌트 수준 데이터 가져오기

## 자동 활성화 기능

`app` 디렉토리를 사용하면 **React Strict Mode**가 자동으로 활성화됩니다.

## 마이그레이션

Pages Router에서 App Router로 마이그레이션하려면 [마이그레이션 가이드](/app-router/guides/migrating/app-router-migration.md)를 참조하세요.

점진적으로 App Router를 도입할 수 있으며, 두 라우터를 동시에 사용할 수도 있습니다.
