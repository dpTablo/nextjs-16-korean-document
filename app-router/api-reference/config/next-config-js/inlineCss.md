# inlineCss

> **경고**: 이 기능은 실험적이며 프로덕션 환경에서는 권장되지 않습니다.

## 개요

`<head>`에서 CSS를 인라인화하는 실험적 지원 기능입니다. 활성화 시 모든 `<link>` 태그가 `<style>` 태그로 변환됩니다.

## 설정

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    inlineCss: true,
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    inlineCss: true,
  },
}

module.exports = nextConfig
```

## CSS 인라인화를 사용해야 할 때

1. **첫 방문 사용자**: CSS 파일 다운로드 지연 제거로 페이지 로드 성능 개선
2. **성능 지표 개선**: 추가 네트워크 요청 제거로 FCP, LCP 개선
3. **느린 연결**: 네트워크 왕복 감소로 지연 시간 단축
4. **Atomic CSS (Tailwind)**: 페이지 크기에 관계없이 스타일 크기가 O(1)이므로 경량이고 추가 요청 불필요

## CSS 인라인화를 피해야 할 때

1. **큰 CSS 번들**: HTML 크기 증가로 TTFB 증가 및 느린 연결 사용자에게 불리
2. **동적/페이지별 CSS**: 스타일 중복 및 코드 비대화 가능
3. **브라우저 캐싱**: 재방문 시 외부 CSS 파일 캐싱 이점 상실

## 제한사항

- CSS 인라인화는 전역 적용만 가능 (페이지별 설정 불가)
- 초기 페이지 로드 시 스타일 중복 (`<style>` 태그 + RSC payload)
- 정적 렌더링 페이지 네비게이션 시 중복 방지를 위해 `<link>` 태그 사용
- 개발 모드에서 미지원, 프로덕션 빌드에서만 작동
