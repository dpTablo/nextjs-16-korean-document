# 클라이언트 사이드 렌더링 (CSR)

클라이언트 사이드 렌더링(CSR)에서는 브라우저가 최소한의 HTML 페이지와 페이지에 필요한 JavaScript를 다운로드합니다. 그런 다음 JavaScript를 사용하여 DOM을 업데이트하고 페이지를 렌더링합니다.

애플리케이션이 처음 로드될 때 사용자는 전체 페이지를 보기 전에 약간의 지연을 느낄 수 있습니다. 이는 모든 JavaScript가 다운로드, 파싱, 실행될 때까지 페이지가 완전히 렌더링되지 않기 때문입니다.

페이지가 처음 로드된 후 동일한 웹사이트의 다른 페이지로 이동하는 것은 일반적으로 더 빠릅니다. 필요한 데이터만 가져오면 되고 JavaScript는 전체 페이지 새로고침 없이 페이지의 일부를 다시 렌더링할 수 있기 때문입니다.

## Next.js에서의 CSR 구현

Next.js에서 클라이언트 사이드 렌더링을 구현하는 두 가지 방법이 있습니다:

### 1. useEffect() 훅 사용

React의 `useEffect()` 훅을 사용하여 서버 사이드 렌더링 메서드(`getStaticProps`와 `getServerSideProps`) 대신 페이지 내에서 데이터를 가져올 수 있습니다.

```jsx
import React, { useState, useEffect } from 'react'

export function Page() {
  const [data, setData] = useState(null)

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch('https://api.example.com/data')
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      const result = await response.json()
      setData(result)
    }

    fetchData().catch((e) => {
      console.error('데이터를 가져오는 중 오류가 발생했습니다: ', e)
    })
  }, [])

  return <p>{data ? `데이터: ${data}` : '로딩 중...'}</p>
}
```

### 2. 데이터 페칭 라이브러리 사용 (권장)

클라이언트에서 데이터를 가져오는 데 [SWR](https://swr.vercel.app/) 또는 [TanStack Query](https://tanstack.com/query/latest/)와 같은 데이터 페칭 라이브러리를 사용하는 것을 권장합니다. 이러한 라이브러리는 메모이제이션, 캐싱, 재검증 등을 제공합니다.

다음은 SWR을 사용한 예시입니다:

```jsx
import useSWR from 'swr'

const fetcher = (...args) => fetch(...args).then((res) => res.json())

export function Page() {
  const { data, error, isLoading } = useSWR(
    'https://api.example.com/data',
    fetcher
  )

  if (error) return <p>로드 실패.</p>
  if (isLoading) return <p>로딩 중...</p>

  return <p>데이터: {data}</p>
}
```

## 알아두면 좋은 점

- **SEO 고려사항:** 검색 엔진 크롤러가 JavaScript를 실행하지 않을 수 있으므로 클라이언트 사이드에서만 렌더링되는 콘텐츠는 검색 결과에 나타나지 않을 수 있습니다.

- **성능:** 느린 인터넷 연결을 사용하는 사용자는 모든 JavaScript가 로드되고 실행될 때까지 오랜 시간을 기다려야 할 수 있습니다.

- **하이브리드 접근 권장:** 가능한 경우 서버 사이드 렌더링(SSR) 또는 정적 사이트 생성(SSG)과 클라이언트 사이드 렌더링을 조합하여 사용하는 것이 좋습니다.

## 다른 렌더링 방식과의 비교

| 방식 | 설명 | 장점 | 단점 |
|------|------|------|------|
| **CSR** | 브라우저에서 JavaScript로 렌더링 | 서버 부하 감소, 페이지 전환 빠름 | 초기 로딩 느림, SEO 불리 |
| **SSR** | 매 요청마다 서버에서 렌더링 | 항상 최신 데이터, SEO 유리 | 서버 부하 증가 |
| **SSG** | 빌드 시 사전 렌더링 | 가장 빠른 로딩, CDN 캐싱 가능 | 실시간 데이터에 부적합 |
| **ISR** | 런타임에 정적 페이지 재생성 | SSG + 실시간 데이터 업데이트 | 설정 복잡성 |

## 사용 사례

클라이언트 사이드 렌더링은 다음과 같은 경우에 적합합니다:

- 사용자 대시보드처럼 SEO가 중요하지 않은 비공개 페이지
- 실시간으로 업데이트되어야 하는 데이터 (채팅, 주식 시세 등)
- 서버에서 사전 렌더링할 필요가 없는 대화형 UI 컴포넌트
