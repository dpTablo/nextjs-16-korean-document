# public 폴더

Next.js는 루트 디렉토리의 `public` 폴더에서 이미지 등의 정적 파일을 제공합니다. 파일은 기본 URL(`/`)에서 참조됩니다.

## 사용 예제

예를 들어, `public/avatars/me.png` 파일은 `/avatars/me.png`로 접근할 수 있습니다:

```jsx
// avatar.js
import Image from 'next/image'

export function Avatar({ id, alt }) {
  return <Image src={`/avatars/${id}.png`} alt={alt} width="64" height="64" />
}

export function AvatarOfMe() {
  return <Avatar id="me" alt="내 초상화" />
}
```

## 폴더 구조

```
my-project/
├── app/
├── public/
│   ├── avatars/
│   │   └── me.png
│   ├── images/
│   │   └── logo.svg
│   └── favicon.ico
└── package.json
```

## 캐싱

Next.js는 `public` 폴더의 에셋을 안전하게 캐시할 수 없습니다. 파일이 변경될 수 있기 때문입니다. 기본 캐싱 헤더는 다음과 같습니다:

```
Cache-Control: public, max-age=0
```

## 메타데이터 파일

`robots.txt`나 `favicon.ico`와 같은 정적 메타데이터 파일의 경우, `public` 폴더 대신 `app` 폴더의 [특수 메타데이터 파일](/app-router/api-reference/file-conventions/metadata/)을 사용하는 것을 권장합니다.

### public 폴더 vs 메타데이터 파일

| 파일 | 권장 위치 | 설명 |
|------|----------|------|
| `favicon.ico` | `app/` | 메타데이터 파일 규칙 사용 |
| `robots.txt` | `app/` | 메타데이터 파일 규칙 사용 |
| `sitemap.xml` | `app/` | 메타데이터 파일 규칙 사용 |
| 이미지, 폰트 등 | `public/` | 정적 에셋으로 사용 |

## 주요 사항

- `public` 폴더의 파일은 빌드 시점에 복사되어 정적으로 제공됩니다
- 파일 이름이 변경되면 캐시 무효화가 자동으로 발생하지 않습니다
- 대용량 파일이나 자주 변경되는 파일은 CDN 사용을 고려하세요
- `public` 폴더 내의 파일은 서버 컴포넌트나 클라이언트 컴포넌트에서 직접 import할 수 없습니다

## 예제: 정적 이미지 사용

```tsx
// app/page.tsx
import Image from 'next/image'

export default function Page() {
  return (
    <div>
      <Image
        src="/images/hero.jpg"
        alt="히어로 이미지"
        width={1200}
        height={600}
      />
    </div>
  )
}
```

## 예제: 정적 파일 다운로드 링크

```tsx
// app/downloads/page.tsx
export default function DownloadsPage() {
  return (
    <div>
      <h1>다운로드</h1>
      <a href="/files/document.pdf" download>
        문서 다운로드
      </a>
    </div>
  )
}
```

---

## 참고

- [이미지 최적화](/app-router/getting-started/09-image-optimization.md)
- [정적 에셋](/app-router/guides/static-exports.md)
- [메타데이터 파일](/app-router/api-reference/file-conventions/metadata/)
