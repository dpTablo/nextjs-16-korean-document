# 클라이언트 사이드 데이터 페칭

클라이언트 사이드 데이터 페칭은 페이지에 SEO 인덱싱이 필요 없거나, 데이터를 미리 렌더링할 필요가 없거나, 페이지 콘텐츠를 자주 업데이트해야 할 때 유용합니다.

서버 사이드 렌더링 API와 달리, 클라이언트 사이드 데이터 페칭은 컴포넌트 레벨에서 사용할 수 있습니다.

- 페이지 레벨에서 수행하면 런타임에 데이터를 페칭하고 데이터가 변경되면 페이지 콘텐츠가 업데이트됩니다.
- 컴포넌트 레벨에서 사용하면 컴포넌트가 마운트될 때 데이터를 페칭하고 데이터가 변경되면 컴포넌트 콘텐츠가 업데이트됩니다.

클라이언트 사이드 데이터 페칭은 애플리케이션 성능과 페이지 로드 속도에 영향을 줄 수 있다는 점에 유의하세요. 이는 컴포넌트나 페이지가 마운트될 때 데이터 페칭이 수행되고 데이터가 캐싱되지 않기 때문입니다.

## useEffect를 사용한 클라이언트 사이드 데이터 페칭

다음 예제는 `useEffect` 훅을 사용하여 클라이언트 측에서 데이터를 페칭하는 방법을 보여줍니다.

```jsx
import { useState, useEffect } from 'react'

function Profile() {
  const [data, setData] = useState(null)
  const [isLoading, setLoading] = useState(true)

  useEffect(() => {
    fetch('/api/profile-data')
      .then((res) => res.json())
      .then((data) => {
        setData(data)
        setLoading(false)
      })
  }, [])

  if (isLoading) return <p>로딩 중...</p>
  if (!data) return <p>프로필 데이터가 없습니다</p>

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.bio}</p>
    </div>
  )
}
```

## SWR을 사용한 클라이언트 사이드 데이터 페칭

Next.js 팀은 [SWR](https://swr.vercel.app/)이라는 데이터 페칭용 React 훅 라이브러리를 만들었습니다. 클라이언트 측에서 데이터를 페칭하는 경우 **강력히 권장**됩니다. 캐싱, 재검증, 포커스 추적, 주기적 재페칭 등을 처리합니다.

위의 예제와 동일하게 SWR을 사용하면 프로필 데이터를 페칭할 수 있습니다. SWR은 자동으로 데이터를 캐싱하고 데이터가 오래되면 재검증합니다.

```jsx
import useSWR from 'swr'

const fetcher = (...args) => fetch(...args).then((res) => res.json())

function Profile() {
  const { data, error } = useSWR('/api/profile-data', fetcher)

  if (error) return <div>불러오기 실패</div>
  if (!data) return <div>로딩 중...</div>

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.bio}</p>
    </div>
  )
}
```

SWR 사용에 대한 자세한 내용은 [SWR 문서](https://swr.vercel.app/)를 참조하세요.

## 예제

### 페이지네이션과 함께 사용

```jsx
import useSWR from 'swr'
import { useState } from 'react'

const fetcher = (url) => fetch(url).then((res) => res.json())

function PostList() {
  const [page, setPage] = useState(1)
  const { data, error, isLoading } = useSWR(
    `/api/posts?page=${page}`,
    fetcher
  )

  if (error) return <div>불러오기 실패</div>
  if (isLoading) return <div>로딩 중...</div>

  return (
    <div>
      <ul>
        {data.posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
      <button onClick={() => setPage(page - 1)} disabled={page === 1}>
        이전
      </button>
      <button onClick={() => setPage(page + 1)} disabled={!data.hasMore}>
        다음
      </button>
    </div>
  )
}
```

### 조건부 페칭

```jsx
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

function UserProfile({ userId }) {
  // userId가 없으면 요청하지 않음
  const { data, error } = useSWR(
    userId ? `/api/users/${userId}` : null,
    fetcher
  )

  if (!userId) return <div>사용자를 선택하세요</div>
  if (error) return <div>불러오기 실패</div>
  if (!data) return <div>로딩 중...</div>

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  )
}
```

### 뮤테이션과 재검증

```jsx
import useSWR, { mutate } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

function Profile() {
  const { data } = useSWR('/api/profile', fetcher)

  const updateProfile = async (newData) => {
    // API 호출
    await fetch('/api/profile', {
      method: 'PUT',
      body: JSON.stringify(newData),
    })

    // 로컬 데이터 재검증
    mutate('/api/profile')
  }

  return (
    <div>
      <h1>{data?.name}</h1>
      <button onClick={() => updateProfile({ name: 'New Name' })}>
        이름 변경
      </button>
    </div>
  )
}
```

### getStaticProps와 결합

초기 데이터는 서버에서 렌더링하고, 이후 클라이언트에서 업데이트할 수 있습니다:

```jsx
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

export async function getStaticProps() {
  const initialData = await fetcher('https://api.example.com/data')
  return {
    props: { initialData },
  }
}

function Page({ initialData }) {
  const { data } = useSWR('https://api.example.com/data', fetcher, {
    fallbackData: initialData,
  })

  return <div>{data.message}</div>
}

export default Page
```

## 관련 문서

- [getStaticProps](/docs/pages/api-reference/functions/getStaticProps) - 빌드 시 데이터 페칭
- [getServerSideProps](/docs/pages/api-reference/functions/getServerSideProps) - 요청 시 데이터 페칭
- [SWR 문서](https://swr.vercel.app/) - SWR 공식 문서
