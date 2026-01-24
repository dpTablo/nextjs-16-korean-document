# cacheHandlers

## 개요

`cacheHandlers` 설정을 통해 `'use cache'` 및 `'use cache: remote'` 지시어에 대한 커스텀 캐시 스토리지 구현을 정의할 수 있습니다. 이를 통해 캐시된 컴포넌트와 함수를 외부 서비스에 저장하거나 캐싱 동작을 커스터마이즈할 수 있습니다.

> **참고**: `'use cache: private'`는 구성할 수 없으며 캐시 핸들러를 사용하지 않습니다.

## 커스텀 캐시 핸들러가 필요한 경우

**대부분의 애플리케이션에는 커스텀 캐시 핸들러가 필요하지 않습니다.** 기본 인메모리 캐시가 일반적인 사용 사례에서 잘 작동합니다.

커스텀 핸들러는 고급 시나리오를 위한 것입니다:

- **여러 인스턴스 간 캐시 공유**: 기본 인메모리 캐시는 각 Next.js 프로세스에 격리됩니다. 커스텀 핸들러를 사용하면 모든 인스턴스가 액세스할 수 있는 공유 스토리지 시스템(Redis, Memcached, DynamoDB)과 통합할 수 있습니다.
- **스토리지 유형 변경**: 재시작 후에도 지속성을 위해, 메모리 사용량 감소를 위해, 또는 기존 인프라 통합을 위해 디스크, 데이터베이스 또는 외부 서비스에 캐시를 저장합니다.

## 설정

별도의 파일에 캐시 핸들러를 정의하고 Next 설정에서 참조합니다:

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheHandlers: {
    default: require.resolve('./cache-handlers/default-handler.js'),
    remote: require.resolve('./cache-handlers/remote-handler.js'),
  },
}

export default nextConfig
```

```js filename="next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  cacheHandlers: {
    default: require.resolve('./cache-handlers/default-handler.js'),
    remote: require.resolve('./cache-handlers/remote-handler.js'),
  },
}

module.exports = nextConfig
```

### 핸들러 유형

| 핸들러 | 설명 |
|--------|------|
| `default` | `'use cache'` 지시어에서 사용 |
| `remote` | `'use cache: remote'` 지시어에서 사용 |

추가 명명된 핸들러(예: `sessions`, `analytics`)를 정의하고 `'use cache: <name>'`으로 참조할 수도 있습니다.

## API 레퍼런스

캐시 핸들러는 다음 메서드를 포함하는 `CacheHandler` 인터페이스를 구현해야 합니다:

### `get(cacheKey, softTags)`

주어진 키에 대한 캐시 항목을 검색합니다.

```ts
get(cacheKey: string, softTags: string[]): Promise<CacheEntry | undefined>
```

항목을 검색하고, `revalidate` 시간을 기준으로 만료 여부를 확인하며, 누락되거나 만료된 항목에 대해 `undefined`를 반환해야 합니다.

### `set(cacheKey, pendingEntry)`

캐시 항목을 저장합니다.

```ts
set(cacheKey: string, pendingEntry: Promise<CacheEntry>): Promise<void>
```

항목이 아직 생성 중일 수 있으므로 저장하기 전에 `pendingEntry` 프로미스를 await해야 합니다.

### `refreshTags()`

외부 태그 서비스와 동기화하기 위해 새 요청을 시작하기 전에 주기적으로 호출됩니다.

```ts
refreshTags(): Promise<void>
```

인메모리 캐시의 경우 no-op일 수 있습니다. 분산 캐시의 경우 외부 서비스에서 태그 상태를 동기화하는 데 사용합니다.

### `getExpiration(tags)`

태그 세트에 대한 최대 재검증 타임스탬프를 가져옵니다.

```ts
getExpiration(tags: string[]): Promise<number>
```

반환값:
- `0`: 태그가 한 번도 재검증되지 않은 경우
- 타임스탬프(밀리초): 가장 최근 재검증을 나타냄
- `Infinity`: `get` 메서드에서 소프트 태그 확인을 처리하는 경우

### `updateTags(tags, durations)`

태그가 재검증되거나 만료될 때 호출됩니다.

```ts
updateTags(tags: string[], durations?: { expire?: number }): Promise<void>
```

일치하는 태그가 있는 모든 캐시 항목을 무효화해야 합니다.

## CacheEntry 타입

```ts
interface CacheEntry {
  value: ReadableStream<Uint8Array>
  tags: string[]
  stale: number
  timestamp: number
  expire: number
  revalidate: number
}
```

| 속성 | 타입 | 설명 |
|------|------|------|
| `value` | `ReadableStream<Uint8Array>` | 스트림으로 된 캐시된 데이터 |
| `tags` | `string[]` | 캐시 태그 (소프트 태그 제외) |
| `stale` | `number` | 클라이언트 측 staleness 기간 (초) |
| `timestamp` | `number` | 항목이 생성된 시간 (밀리초) |
| `expire` | `number` | 항목이 사용될 수 있는 기간 (초) |
| `revalidate` | `number` | 재검증까지의 기간 (초) |

## 예제

### 기본 인메모리 캐시 핸들러

```js filename="cache-handlers/memory-handler.js"
const cache = new Map()
const pendingSets = new Map()

module.exports = {
  async get(cacheKey, softTags) {
    const pendingPromise = pendingSets.get(cacheKey)
    if (pendingPromise) {
      await pendingPromise
    }

    const entry = cache.get(cacheKey)
    if (!entry) {
      return undefined
    }

    // 항목이 만료되었는지 확인
    const now = Date.now()
    if (now > entry.timestamp + entry.revalidate * 1000) {
      return undefined
    }

    return entry
  },

  async set(cacheKey, pendingEntry) {
    let resolvePending
    const pendingPromise = new Promise((resolve) => {
      resolvePending = resolve
    })
    pendingSets.set(cacheKey, pendingPromise)

    try {
      const entry = await pendingEntry
      cache.set(cacheKey, entry)
    } finally {
      resolvePending()
      pendingSets.delete(cacheKey)
    }
  },

  async refreshTags() {
    // 인메모리 캐시의 경우 no-op
  },

  async getExpiration(tags) {
    return 0
  },

  async updateTags(tags, durations) {
    for (const [key, entry] of cache.entries()) {
      if (entry.tags.some((tag) => tags.includes(tag))) {
        cache.delete(key)
      }
    }
  },
}
```

### Redis 외부 스토리지 예제

```js filename="cache-handlers/redis-handler.js"
const { createClient } = require('redis')

const client = createClient({ url: process.env.REDIS_URL })
client.connect()

module.exports = {
  async get(cacheKey, softTags) {
    const stored = await client.get(cacheKey)
    if (!stored) return undefined

    const data = JSON.parse(stored)
    return {
      value: new ReadableStream({
        start(controller) {
          controller.enqueue(Buffer.from(data.value, 'base64'))
          controller.close()
        },
      }),
      tags: data.tags,
      stale: data.stale,
      timestamp: data.timestamp,
      expire: data.expire,
      revalidate: data.revalidate,
    }
  },

  async set(cacheKey, pendingEntry) {
    const entry = await pendingEntry
    const reader = entry.value.getReader()
    const chunks = []

    try {
      while (true) {
        const { done, value } = await reader.read()
        if (done) break
        chunks.push(value)
      }
    } finally {
      reader.releaseLock()
    }

    const data = Buffer.concat(chunks.map((chunk) => Buffer.from(chunk)))

    await client.set(
      cacheKey,
      JSON.stringify({
        value: data.toString('base64'),
        tags: entry.tags,
        stale: entry.stale,
        timestamp: entry.timestamp,
        expire: entry.expire,
        revalidate: entry.revalidate,
      }),
      { EX: entry.expire }
    )
  },

  async refreshTags() {},

  async getExpiration(tags) {
    return 0
  },

  async updateTags(tags, durations) {},
}
```

## 플랫폼 지원

| 배포 옵션 | 지원 여부 |
|-----------|-----------|
| Node.js 서버 | 예 |
| Docker 컨테이너 | 예 |
| 정적 내보내기 | 아니오 |
| 어댑터 | 플랫폼별 |

## 버전 히스토리

| 버전 | 변경사항 |
|------|----------|
| v16.0.0 | `cacheHandlers` 도입 |
