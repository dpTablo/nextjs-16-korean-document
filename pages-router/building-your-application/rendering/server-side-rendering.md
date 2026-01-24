# 서버 사이드 렌더링 (SSR)

> "SSR" 또는 "동적 렌더링"이라고도 합니다.

서버 사이드 렌더링을 사용하는 페이지는 **매 요청마다** 페이지 HTML이 생성됩니다.

## 사용 방법

페이지에서 서버 사이드 렌더링을 사용하려면 `getServerSideProps`라는 `async` 함수를 `export`해야 합니다. 이 함수는 **모든 요청마다** 서버에서 호출됩니다.

예를 들어, 외부 API에서 자주 업데이트되는 데이터를 미리 렌더링해야 하는 페이지가 있다고 가정해 봅시다. `getServerSideProps`에서 이 데이터를 가져와 아래와 같이 `Page` 컴포넌트에 props로 전달할 수 있습니다:

```jsx
export default function Page({ data }) {
  // 데이터 렌더링...
}

// 매 요청마다 호출됩니다
export async function getServerSideProps() {
  // 외부 API에서 데이터 가져오기
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  // props를 통해 페이지에 데이터 전달
  return { props: { data } }
}
```

## getStaticProps와의 차이점

`getServerSideProps`는 `getStaticProps`와 유사하지만, 차이점은 `getServerSideProps`가 빌드 시간이 아닌 **모든 요청**에서 실행된다는 것입니다.

## 사용 사례

서버 사이드 렌더링은 다음과 같은 경우에 적합합니다:

- 요청마다 데이터가 변경되는 페이지
- 사용자 인증 상태에 따라 다른 콘텐츠를 보여주는 페이지
- 실시간 데이터가 필요한 페이지

## 주의사항

- 서버 사이드 렌더링은 매 요청마다 서버에서 실행되므로 정적 생성보다 느립니다
- CDN에서 캐싱할 수 없어 응답 시간이 늘어날 수 있습니다
- TTFB(Time to First Byte)가 증가할 수 있습니다

## 더 알아보기

`getServerSideProps`의 작동 방식에 대한 자세한 내용은 [getServerSideProps API 참조](/pages-router/api-reference/getServerSideProps.md)를 확인하세요.
