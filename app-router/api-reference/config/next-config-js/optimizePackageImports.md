# optimizePackageImports

일부 패키지는 수백 또는 수천 개의 모듈을 내보낼 수 있어 개발 및 프로덕션에서 성능 문제를 일으킬 수 있습니다.

`experimental.optimizePackageImports`에 패키지를 추가하면 실제로 사용하는 모듈만 로드하면서도 여러 명명된 내보내기가 있는 import 문을 작성하는 편의성을 유지할 수 있습니다.

```js filename="next.config.js"
module.exports = {
  experimental: {
    optimizePackageImports: ['package-name'],
  },
}
```

## 기본 최적화 라이브러리

다음 라이브러리는 기본적으로 최적화됩니다:

- `lucide-react`
- `date-fns`
- `lodash-es`
- `ramda`
- `antd`
- `react-bootstrap`
- `ahooks`
- `@ant-design/icons`
- `@headlessui/react`
- `@headlessui-float/react`
- `@heroicons/react/20/solid`
- `@heroicons/react/24/solid`
- `@heroicons/react/24/outline`
- `@visx/visx`
- `@tremor/react`
- `rxjs`
- `@mui/material`
- `@mui/icons-material`
- `recharts`
- `react-use`
- `@material-ui/core`
- `@material-ui/icons`
- `@tabler/icons-react`
- `mui-core`
- `react-icons/*`
- `effect`
- `@effect/*`

## 상태

이 기능은 실험적이며 변경될 수 있습니다.
