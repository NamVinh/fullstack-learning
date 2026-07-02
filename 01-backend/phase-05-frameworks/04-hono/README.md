# Phase 5.4 — Hono

Hono là web framework siêu nhẹ (ultra-lightweight) xây dựng trên **Web Standards API** (Request/Response/Headers), cho phép chạy trên nhiều runtimes: **Cloudflare Workers, Bun, Deno, Node.js, AWS Lambda**. Nếu bạn cần deploy lên edge computing hoặc serverless, Hono là lựa chọn hàng đầu.

---

## Tại sao Hono?

- **Multi-runtime**: cùng một code chạy được trên Cloudflare, Bun, Node.js, Deno
- **Web Standards**: dùng native `Request`/`Response` — không phải `req`/`res` của Node.js
- **Tiny bundle**: ~14KB — phù hợp edge/serverless
- **Fast**: nhanh hơn Express, comparable với Fastify
- **TypeScript-first**: type-safe từ đầu

---

## Setup cho từng Runtime

```bash
# Cloudflare Workers
npm create cloudflare@latest -- --framework=hono

# Node.js
npm install hono @hono/node-server

# Bun
bun create hono my-app
```

---

## Cú pháp cơ bản

```typescript
import { Hono } from 'hono'

const app = new Hono()

// Routes
app.get('/', (c) => c.text('Hello Hono!'))
app.get('/json', (c) => c.json({ message: 'Hello', status: 'ok' }))

// Route params
app.get('/users/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ id })
})

// Query params
app.get('/search', (c) => {
  const q = c.req.query('q')
  const page = c.req.query('page') || '1'
  return c.json({ q, page: parseInt(page) })
})

// Request body
app.post('/users', async (c) => {
  const body = await c.req.json()
  return c.json({ created: body }, 201)
})

// HTTP methods
app.put('/users/:id', async (c) => {
  const id = c.req.param('id')
  const body = await c.req.json()
  return c.json({ id, ...body })
})

app.delete('/users/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ deleted: id })
})
```

---

## Run trên Node.js

```typescript
// src/index.ts
import { Hono } from 'hono'
import { serve } from '@hono/node-server'

const app = new Hono()

app.get('/', (c) => c.json({ message: 'Hello from Hono on Node.js' }))

serve({
  fetch: app.fetch,   // Hono dùng fetch handler, không phải http.createServer
  port: 3000
}, (info) => {
  console.log(`Server running on http://localhost:${info.port}`)
})
```

---

## Middleware

```typescript
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { logger } from 'hono/logger'
import { prettyJSON } from 'hono/pretty-json'
import { bearerAuth } from 'hono/bearer-auth'

const app = new Hono()

// Built-in middleware
app.use('*', logger())          // request logging
app.use('*', cors({
  origin: ['https://myapp.com'],
  allowMethods: ['GET', 'POST', 'PUT', 'DELETE']
}))
app.use('*', prettyJSON())      // format JSON responses

// Custom middleware
app.use('*', async (c, next) => {
  const startTime = Date.now()
  await next()
  const duration = Date.now() - startTime
  c.header('X-Response-Time', `${duration}ms`)
})

// Auth middleware cho specific routes
const authMiddleware = bearerAuth({ token: process.env.API_TOKEN! })
app.use('/api/*', authMiddleware)
```

---

## Validation với Zod Validator

```typescript
import { Hono } from 'hono'
import { zValidator } from '@hono/zod-validator'
import { z } from 'zod'

const app = new Hono()

const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(18).optional()
})

app.post(
  '/users',
  zValidator('json', createUserSchema),  // validate JSON body
  async (c) => {
    const { name, email, age } = c.req.valid('json')  // type-safe!
    // TypeScript biết types từ schema
    return c.json({ name, email, age }, 201)
  }
)

// Validate query params
const querySchema = z.object({
  page: z.coerce.number().default(1),
  limit: z.coerce.number().max(100).default(10)
})

app.get('/users', zValidator('query', querySchema), (c) => {
  const { page, limit } = c.req.valid('query')
  return c.json({ page, limit })
})
```

---

## Routing Groups

```typescript
import { Hono } from 'hono'

const app = new Hono()

// Sub-app với base path
const api = new Hono().basePath('/api')

const users = new Hono()
users.get('/', (c) => c.json([]))
users.post('/', async (c) => {
  const body = await c.req.json()
  return c.json(body, 201)
})
users.get('/:id', (c) => c.json({ id: c.req.param('id') }))

const posts = new Hono()
posts.get('/', (c) => c.json([]))

api.route('/users', users)  // /api/users
api.route('/posts', posts)  // /api/posts
app.route('/', api)
```

---

## Context (c)

Context object `c` là central API trong Hono:

```typescript
app.get('/example', async (c) => {
  // Request
  c.req.method          // 'GET'
  c.req.url             // full URL string
  c.req.path            // '/example'
  c.req.param('id')     // route param
  c.req.query('page')   // query param
  await c.req.json()    // parse JSON body
  await c.req.text()    // body as text
  await c.req.formData() // form data
  c.req.header('Authorization')  // header

  // Response helpers
  return c.json({ data: 'value' })           // JSON response
  return c.json({ data: 'value' }, 201)      // with status
  return c.text('Hello')                     // text/plain
  return c.html('<h1>Hello</h1>')            // text/html
  return c.redirect('/new-url')              // 302 redirect
  return c.redirect('/new-url', 301)         // permanent redirect

  // Set headers/status
  c.status(202)
  c.header('X-Custom', 'value')
  return c.json({ ok: true })

  // Store per-request state (như locals trong Express)
  c.set('user', { id: 1, name: 'Alice' })
  const user = c.get('user')
})
```

---

## Deploy trên Cloudflare Workers

```typescript
// src/worker.ts
import { Hono } from 'hono'

// Cloudflare Workers environment interface
type Env = {
  DB: D1Database
  KV: KVNamespace
  JWT_SECRET: string
}

const app = new Hono<{ Bindings: Env }>()

app.get('/users', async (c) => {
  // c.env.DB → D1 database
  const users = await c.env.DB.prepare('SELECT * FROM users').all()
  return c.json(users.results)
})

app.get('/cache/:key', async (c) => {
  const key = c.req.param('key')
  const value = await c.env.KV.get(key)  // KV storage
  if (!value) return c.json({ error: 'Not found' }, 404)
  return c.json({ key, value })
})

export default app  // export default — đây là Cloudflare Workers interface
```

---

## Error Handling

```typescript
// Global error handler
app.onError((err, c) => {
  console.error(`Error: ${err.message}`, err)
  return c.json({
    error: err.message,
    status: 'error'
  }, 500)
})

// 404 handler
app.notFound((c) => {
  return c.json({
    error: `${c.req.method} ${c.req.path} not found`
  }, 404)
})

// HTTPException (Hono built-in)
import { HTTPException } from 'hono/http-exception'

app.get('/protected', (c) => {
  throw new HTTPException(401, { message: 'Unauthorized' })
})
```

---

## Bài tập thực hành

1. **Multi-runtime app**: Tạo Hono app chạy được trên cả Node.js và Bun. Viết script riêng cho từng runtime trong `package.json`.

2. **Cloudflare Workers deploy**: Deploy một simple REST API lên Cloudflare Workers free plan. Dùng KV storage để lưu data.

3. **Type-safe API**: Dùng `zValidator` cho tất cả endpoints, verify TypeScript infer đúng types từ Zod schemas trong route handlers.

---

## Resources

- [hono.dev](https://hono.dev) — Official Hono documentation
- [hono.dev — Getting Started](https://hono.dev/docs/getting-started/basic) — Quick start
- [Cloudflare Workers docs](https://developers.cloudflare.com/workers/) — Deploy target
