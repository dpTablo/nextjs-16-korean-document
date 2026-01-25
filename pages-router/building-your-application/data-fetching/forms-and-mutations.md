# Forms and Mutations (폼과 데이터 변경)

Pages Router에서 폼 제출과 데이터 변경은 주로 **API Routes**를 통해 처리됩니다. 이 가이드에서는 Next.js Pages Router에서 폼을 처리하는 다양한 패턴을 설명합니다.

## 기본 폼 처리

### API Route 생성

먼저 폼 데이터를 처리할 API Route를 생성합니다:

```js
// pages/api/submit.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' })
  }

  try {
    const { name, email, message } = req.body

    // 데이터 유효성 검사
    if (!name || !email) {
      return res.status(400).json({ message: '필수 필드가 누락되었습니다' })
    }

    // 데이터베이스에 저장하거나 다른 작업 수행
    // await db.insert({ name, email, message })

    return res.status(200).json({ message: '성공적으로 제출되었습니다' })
  } catch (error) {
    return res.status(500).json({ message: '서버 오류가 발생했습니다' })
  }
}
```

### 폼 컴포넌트

```jsx
// pages/contact.js
import { useState } from 'react'

export default function ContactPage() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
  })
  const [status, setStatus] = useState({ loading: false, error: null, success: false })

  const handleChange = (e) => {
    const { name, value } = e.target
    setFormData((prev) => ({ ...prev, [name]: value }))
  }

  const handleSubmit = async (e) => {
    e.preventDefault()
    setStatus({ loading: true, error: null, success: false })

    try {
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      })

      if (!response.ok) {
        throw new Error('제출에 실패했습니다')
      }

      setStatus({ loading: false, error: null, success: true })
      setFormData({ name: '', email: '', message: '' })
    } catch (error) {
      setStatus({ loading: false, error: error.message, success: false })
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">이름</label>
        <input
          type="text"
          id="name"
          name="name"
          value={formData.name}
          onChange={handleChange}
          required
        />
      </div>
      <div>
        <label htmlFor="email">이메일</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          required
        />
      </div>
      <div>
        <label htmlFor="message">메시지</label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
        />
      </div>
      <button type="submit" disabled={status.loading}>
        {status.loading ? '제출 중...' : '제출'}
      </button>
      {status.error && <p style={{ color: 'red' }}>{status.error}</p>}
      {status.success && <p style={{ color: 'green' }}>성공적으로 제출되었습니다!</p>}
    </form>
  )
}
```

## 서버 사이드 유효성 검사

### Zod를 사용한 유효성 검사

```js
// pages/api/register.js
import { z } from 'zod'

const schema = z.object({
  email: z.string().email('유효한 이메일을 입력하세요'),
  password: z.string().min(8, '비밀번호는 최소 8자 이상이어야 합니다'),
  name: z.string().min(2, '이름은 최소 2자 이상이어야 합니다'),
})

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' })
  }

  try {
    const validatedData = schema.parse(req.body)

    // 유효성 검사 통과 - 데이터 처리
    // await createUser(validatedData)

    return res.status(201).json({ message: '회원가입이 완료되었습니다' })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        message: '유효성 검사 실패',
        errors: error.errors,
      })
    }
    return res.status(500).json({ message: '서버 오류' })
  }
}
```

## 파일 업로드

### 파일 업로드 API Route

```js
// pages/api/upload.js
import formidable from 'formidable'
import fs from 'fs'
import path from 'path'

export const config = {
  api: {
    bodyParser: false, // formidable을 사용하기 위해 비활성화
  },
}

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' })
  }

  const uploadDir = path.join(process.cwd(), 'public/uploads')

  // 업로드 디렉토리가 없으면 생성
  if (!fs.existsSync(uploadDir)) {
    fs.mkdirSync(uploadDir, { recursive: true })
  }

  const form = formidable({
    uploadDir,
    keepExtensions: true,
    maxFileSize: 5 * 1024 * 1024, // 5MB
  })

  form.parse(req, (err, fields, files) => {
    if (err) {
      return res.status(500).json({ message: '파일 업로드 실패' })
    }

    const file = files.file[0]
    return res.status(200).json({
      message: '파일이 업로드되었습니다',
      filename: file.newFilename,
    })
  })
}
```

### 파일 업로드 폼

```jsx
// pages/upload.js
import { useState } from 'react'

export default function UploadPage() {
  const [file, setFile] = useState(null)
  const [uploading, setUploading] = useState(false)

  const handleSubmit = async (e) => {
    e.preventDefault()
    if (!file) return

    setUploading(true)

    const formData = new FormData()
    formData.append('file', file)

    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
      })

      if (!response.ok) throw new Error('업로드 실패')

      const data = await response.json()
      alert(`업로드 완료: ${data.filename}`)
    } catch (error) {
      alert('업로드 중 오류가 발생했습니다')
    } finally {
      setUploading(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="file"
        onChange={(e) => setFile(e.target.files[0])}
        accept="image/*"
      />
      <button type="submit" disabled={!file || uploading}>
        {uploading ? '업로드 중...' : '업로드'}
      </button>
    </form>
  )
}
```

## SWR을 사용한 낙관적 업데이트

```jsx
// pages/todos.js
import useSWR, { mutate } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

export default function TodosPage() {
  const { data: todos, error } = useSWR('/api/todos', fetcher)

  const addTodo = async (text) => {
    const newTodo = { id: Date.now(), text, completed: false }

    // 낙관적 업데이트
    mutate('/api/todos', [...todos, newTodo], false)

    // 서버에 요청
    await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text }),
    })

    // 데이터 재검증
    mutate('/api/todos')
  }

  const toggleTodo = async (id) => {
    const updatedTodos = todos.map((todo) =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )

    // 낙관적 업데이트
    mutate('/api/todos', updatedTodos, false)

    // 서버에 요청
    await fetch(`/api/todos/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ completed: !todos.find((t) => t.id === id).completed }),
    })

    // 데이터 재검증
    mutate('/api/todos')
  }

  if (error) return <div>로딩 실패</div>
  if (!todos) return <div>로딩 중...</div>

  return (
    <div>
      <TodoForm onAdd={addTodo} />
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## CRUD 작업

### 완전한 CRUD API

```js
// pages/api/posts/index.js
import { posts } from '../../../lib/db'

export default async function handler(req, res) {
  switch (req.method) {
    case 'GET':
      return res.status(200).json(posts)

    case 'POST':
      const newPost = {
        id: Date.now(),
        ...req.body,
        createdAt: new Date().toISOString(),
      }
      posts.push(newPost)
      return res.status(201).json(newPost)

    default:
      return res.status(405).json({ message: 'Method not allowed' })
  }
}
```

```js
// pages/api/posts/[id].js
import { posts } from '../../../lib/db'

export default async function handler(req, res) {
  const { id } = req.query
  const postIndex = posts.findIndex((p) => p.id === parseInt(id))

  if (postIndex === -1) {
    return res.status(404).json({ message: '게시물을 찾을 수 없습니다' })
  }

  switch (req.method) {
    case 'GET':
      return res.status(200).json(posts[postIndex])

    case 'PUT':
      posts[postIndex] = { ...posts[postIndex], ...req.body }
      return res.status(200).json(posts[postIndex])

    case 'DELETE':
      const deleted = posts.splice(postIndex, 1)
      return res.status(200).json(deleted[0])

    default:
      return res.status(405).json({ message: 'Method not allowed' })
  }
}
```

## 에러 처리

### 일관된 에러 응답

```js
// lib/api-handler.js
export function apiHandler(handler) {
  return async (req, res) => {
    try {
      await handler(req, res)
    } catch (error) {
      console.error(error)

      if (error.name === 'ValidationError') {
        return res.status(400).json({ message: error.message })
      }

      if (error.name === 'UnauthorizedError') {
        return res.status(401).json({ message: '인증이 필요합니다' })
      }

      return res.status(500).json({ message: '서버 오류가 발생했습니다' })
    }
  }
}
```

```js
// pages/api/protected.js
import { apiHandler } from '../../lib/api-handler'

export default apiHandler(async (req, res) => {
  // 핸들러 로직
  res.status(200).json({ data: '보호된 데이터' })
})
```

---

## 참고

- [API Routes](/pages-router/building-your-application/routing/api-routes.md)
- [클라이언트 사이드 데이터 페칭](/pages-router/building-your-application/data-fetching/client-side.md)
- [getServerSideProps](/pages-router/api-reference/getServerSideProps.md)
