# urlImports

URL 가져오기는 로컬 디스크 대신 외부 서버에서 직접 모듈을 가져올 수 있게 해주는 실험적 기능입니다.

> **경고**: 이 기능은 실험적입니다. 신뢰할 수 있는 도메인만 사용하여 다운로드하고 컴퓨터에서 코드를 실행하세요. 기능이 안정화로 표시될 때까지 주의하여 사용하세요.

기능을 활성화하려면 `next.config.js` 내에 허용된 URL 접두사를 추가하세요:

```js filename="next.config.js"
module.exports = {
  experimental: {
    urlImports: ['https://example.com/assets/', 'https://cdn.skypack.dev'],
  },
}
```

그런 다음 URL에서 직접 모듈을 가져올 수 있습니다:

```js
import { a, b, c } from 'https://example.com/assets/some/module.js'
```

URL 가져오기는 일반 패키지 가져오기가 사용될 수 있는 모든 곳에서 사용할 수 있습니다.

## 보안 모델

이 기능은 **보안을 최우선**으로 설계되고 있습니다. 먼저 URL 가져오기를 허용할 도메인을 명시적으로 허용해야 하는 실험적 플래그를 추가했습니다. 이를 더 발전시켜 Edge Runtime을 사용하여 브라우저 샌드박스에서 실행되는 URL 가져오기로 제한할 예정입니다.

## 잠금 파일

URL 가져오기를 사용할 때 Next.js는 잠금 파일과 가져온 자산을 포함하는 `next.lock` 디렉토리를 생성합니다. 이 디렉토리는 **Git에 커밋되어야 하며**, `.gitignore`에서 무시되어서는 안 됩니다.

- `next dev`를 실행할 때 Next.js는 새로 발견된 모든 URL 가져오기를 다운로드하고 잠금 파일에 추가합니다
- `next build`를 실행할 때 Next.js는 프로덕션용 애플리케이션을 빌드하기 위해 잠금 파일만 사용합니다

일반적으로 네트워크 요청이 필요하지 않으며 오래된 잠금 파일은 빌드 실패를 유발합니다. 한 가지 예외는 `Cache-Control: no-cache`로 응답하는 리소스입니다. 이러한 리소스는 잠금 파일에 `no-cache` 항목을 가지며 각 빌드에서 항상 네트워크에서 가져옵니다.

## 예제

### Skypack

```js
import confetti from 'https://cdn.skypack.dev/canvas-confetti'
import { useEffect } from 'react'

export default () => {
  useEffect(() => {
    confetti()
  })
  return <p>Hello</p>
}
```

### 정적 이미지 가져오기

```js
import Image from 'next/image'
import logo from 'https://example.com/assets/logo.png'

export default () => (
  <div>
    <Image src={logo} placeholder="blur" />
  </div>
)
```

### CSS의 URL

```css
.className {
  background: url('https://example.com/assets/hero.jpg');
}
```

### 자산 가져오기

```js
const logo = new URL('https://example.com/assets/file.txt', import.meta.url)
console.log(logo.pathname)
// 출력: "/_next/static/media/file.a9727b5d.txt"
```
