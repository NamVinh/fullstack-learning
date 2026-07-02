# Node.js Developer Roadmap

> Dựa trên roadmap.sh/nodejs + nodejs.org + research thực tế (17 claims verified 3-0).
> Học theo thứ tự từ trên xuống. Mỗi phase có bài tập thực hành.

---

## Phase 0 — Nền tảng bắt buộc trước khi học Node

| Kiến thức | Tại sao cần |
|-----------|------------|
| JavaScript (ES6+) nhuần nhuyễn | async/await, destructuring, closures, prototypes |
| Git cơ bản | commit, branch, merge, pull request |
| Terminal / CLI | cd, ls, mkdir, chmod, pipes |
| HTTP protocol | request/response, methods, status codes, headers |

---

## Phase 1 — Node.js Runtime (Nền tảng cốt lõi)

> **Đây là phần quan trọng nhất.** Nếu bỏ qua, bạn sẽ dùng Node như một black box.

### 1.1 What is Node.js

- Runtime chạy JS ngoài browser, dùng **V8 engine** (của Google Chrome)
- Non-blocking, event-driven — thiết kế gốc cho real-time push-based architectures
- Single thread nhưng xử lý concurrency qua **Event Loop + libuv**
- Khác browser: không có DOM/`window`, có `process`, `fs`, `path`, `http`

```
┌─────────────────────────────────────────────┐
│              Node.js Process                │
│  ┌─────────────┐     ┌────────────────────┐ │
│  │  V8 Engine  │     │  libuv (C library) │ │
│  │  (JS code)  │     │  Thread pool (×4)  │ │
│  │             │     │  epoll/kqueue/IOCP │ │
│  └─────────────┘     └────────────────────┘ │
└─────────────────────────────────────────────┘
```

**Bài tập**: Cài Node.js, chạy `node --version`, thử REPL (`node` không có argument).

---

### 1.2 Event Loop ⭐ (quan trọng nhất)

> "The Event Loop is one of the most critical aspects of Node.js — it explains how Node.js can be asynchronous and have non-blocking I/O." — roadmap.sh

**6 phases theo thứ tự (verified 3-0 từ nodejs.org):**

```
   ┌──────────────────────────┐
┌─>│   1. timers              │  setTimeout(), setInterval()
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   2. pending callbacks   │  I/O errors từ vòng trước
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   3. idle, prepare       │  internal only
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   4. poll                │  nhận I/O events mới, chạy I/O callbacks
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   5. check               │  setImmediate()
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
└──┤   6. close callbacks     │  socket.destroy(), etc.
   └──────────────────────────┘
```

**Quy tắc quan trọng:**
- `process.nextTick()` chạy **ngay sau operation hiện tại**, trước khi event loop chuyển phase — KHÔNG phải là một phase
- Gọi `process.nextTick()` đệ quy → **starve I/O** (event loop không bao giờ đến poll phase)
- Trong I/O callback: `setImmediate()` luôn chạy trước `setTimeout(0)`
- Ngoài I/O callback (main module): thứ tự `setImmediate` vs `setTimeout` là **non-deterministic**

**libuv (verified 3-0):**
- Cross-platform C library cung cấp event loop
- OS backends: **epoll** (Linux), **kqueue** (macOS/BSD), **IOCP** (Windows), event ports (Solaris)
- Thread pool mặc định **4 threads** dùng cho file I/O và DNS lookup
- Thay đổi: `UV_THREADPOOL_SIZE=8 node app.js`
- Tất cả callbacks chạy trên main thread

**Node.js exit:** process chỉ thoát khi có **zero referenced handles** và zero active requests (verified 3-0). Dùng `.unref()` trên timer để không block exit.

**Bài tập Event Loop:**
```js
// Đoán output rồi chạy để verify
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));
console.log('sync');
// Output: sync → nextTick → Promise → setTimeout/setImmediate (non-deterministic)
```

---

### 1.3 Modules System

**CommonJS (CJS) — default:**
```js
// export
module.exports = { foo, bar }
exports.foo = foo  // shorthand, nhưng KHÔNG dùng exports = {} (thay module.exports)

// import
const { foo } = require('./foo')
```

**ES Modules (ESM) — modern:**
```js
// package.json: "type": "module"  hoặc file .mjs
export const foo = () => {}
export default class Foo {}
import { foo } from './foo.js'  // phải có .js extension
```

**Module resolution order (require):**
1. Core modules (`fs`, `path`, etc.)
2. `node_modules/` folder tìm lên từ file hiện tại
3. File paths (`./`, `../`, `/`)

**Bài tập**: Tạo 3 files — `math.js`, `logger.js`, `index.js`. Dùng cả CJS và ESM.

---

### 1.4 Built-in Modules

#### `fs` module
```js
const fs = require('fs')

// Sync (block event loop — chỉ dùng lúc startup)
const data = fs.readFileSync('./file.txt', 'utf8')

// Async callback
fs.readFile('./file.txt', 'utf8', (err, data) => {})

// Promises (khuyên dùng)
const { readFile, writeFile } = require('fs/promises')
const data = await readFile('./file.txt', 'utf8')

// Streaming (dùng cho file lớn)
const stream = fs.createReadStream('./bigfile.csv')
```

**Bài tập**: Viết script đọc file CSV, parse từng dòng, ghi ra file JSON.

#### `path` module
```js
const path = require('path')
path.join('/foo', 'bar', 'baz.txt')  // /foo/bar/baz.txt
path.resolve('src', 'index.js')      // absolute path
path.dirname('/foo/bar/baz.txt')     // /foo/bar
path.basename('/foo/bar/baz.txt')    // baz.txt
path.extname('index.html')           // .html
__dirname  // directory của file hiện tại (CJS only)
__filename // path của file hiện tại (CJS only)
```

#### `http` module
```js
const http = require('http')
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' })
  res.end(JSON.stringify({ hello: 'world' }))
})
server.listen(3000)
```

**Bài tập**: Tạo HTTP server thuần (không dùng Express) trả về JSON, handle `/users` và `/posts`.

#### `process` object
```js
process.env.NODE_ENV    // 'development', 'production'
process.argv            // ['node', 'script.js', 'arg1']
process.cwd()           // current working directory
process.exit(0)         // thoát với code 0 (success)
process.on('uncaughtException', handler)
process.on('unhandledRejection', handler)
process.on('SIGTERM', handler)  // graceful shutdown
```

#### `crypto` module
```js
const crypto = require('crypto')
crypto.randomUUID()                    // UUID v4
crypto.randomBytes(16).toString('hex') // random token
crypto.createHash('sha256').update('data').digest('hex')
```

#### `os` module
```js
const os = require('os')
os.cpus()     // CPU cores info
os.totalmem() // total RAM
os.tmpdir()   // /tmp
```

---

### 1.5 Streams ⭐

> Dùng để xử lý dữ liệu lớn mà không load hết vào memory.

**4 loại stream (verified 3-0 từ nodejs.org):**

| Type | Ví dụ |
|------|-------|
| **Readable** | `fs.createReadStream()`, HTTP request |
| **Writable** | `fs.createWriteStream()`, HTTP response |
| **Duplex** | `net.Socket` (vừa read vừa write) |
| **Transform** | `zlib.createGzip()` (Transform là subclass của Duplex) |

**Backpressure (verified 3-0):**
- `writable.write()` trả về `false` → dừng write, đợi `'drain'` event
- `pipe()` tự động handle backpressure

```js
// Pipe chain — đọc file → gzip → ghi
const { createReadStream, createWriteStream } = require('fs')
const { createGzip } = require('zlib')

createReadStream('./input.txt')
  .pipe(createGzip())
  .pipe(createWriteStream('./input.txt.gz'))

// Async iteration (modern approach)
for await (const chunk of readStream) {
  process.stdout.write(chunk)
}
```

**`highWaterMark` (verified 3-0):** threshold, KHÔNG phải hard limit. Khi buffer đạt mức này, stream dừng request thêm data từ nguồn.

**Bài tập Streams:**
1. Đọc file 1GB bằng `readStream`, đếm số dòng — đo memory usage vs `readFileSync`
2. Implement custom Transform stream: uppercase mọi text

---

### 1.6 Buffers

- Binary data representation trong Node.js
- `Buffer.alloc(10)` — tạo buffer 10 bytes, zero-filled
- `Buffer.from('hello', 'utf8')` — từ string
- `Buffer.from([0x48, 0x65])` — từ array of bytes
- `buf.toString('base64')` — convert sang base64
- `Buffer.concat([buf1, buf2])` — merge buffers

**Bài tập**: Implement base64 encoding/decoding thủ công dùng Buffer.

---

### 1.7 Events & EventEmitter

```js
const EventEmitter = require('events')

class MyEmitter extends EventEmitter {}
const emitter = new MyEmitter()

emitter.on('data', (payload) => console.log(payload))
emitter.once('connect', () => console.log('connected once'))
emitter.emit('data', { value: 42 })

// Error handling — luôn listen 'error' event
emitter.on('error', (err) => console.error(err))
```

---

### 1.8 Child Processes

```js
const { exec, spawn, fork } = require('child_process')

// exec — shell command, buffer output
exec('ls -la', (err, stdout, stderr) => {})

// spawn — streaming output, no shell
const ls = spawn('ls', ['-la'])
ls.stdout.pipe(process.stdout)

// fork — chạy Node.js script riêng, có IPC channel
const child = fork('./worker.js')
child.send({ task: 'compute' })
child.on('message', (result) => {})
```

---

### 1.9 Worker Threads (CPU-bound tasks)

```js
const { Worker, isMainThread, parentPort } = require('worker_threads')

if (isMainThread) {
  const worker = new Worker('./worker.js')
  worker.on('message', (result) => console.log(result))
  worker.postMessage({ data: [1, 2, 3] })
} else {
  parentPort.on('message', ({ data }) => {
    const result = data.reduce((a, b) => a + b, 0)
    parentPort.postMessage(result)
  })
}
```

**Khi nào dùng gì:**
- `child_process.fork()` → chạy Node script riêng biệt, IPC
- `worker_threads` → CPU-intensive JS code (parsing, crypto, compression)
- `cluster` → fork nhiều process để dùng hết CPU cores cho HTTP server

---

### 1.10 Cluster Module

```js
const cluster = require('cluster')
const { cpus } = require('os')

if (cluster.isPrimary) {
  for (let i = 0; i < cpus().length; i++) {
    cluster.fork()
  }
  cluster.on('exit', (worker) => cluster.fork()) // restart crashed workers
} else {
  // mỗi worker chạy HTTP server riêng
  require('./server')
}
```

---

## Phase 2 — npm & Package Management

### 2.1 npm Basics

```bash
npm init -y                    # tạo package.json
npm install express            # install + thêm vào dependencies
npm install -D vitest          # devDependencies
npm install -g nodemon         # global
npm uninstall express
npm update
npm list --depth=0
```

### 2.2 package.json quan trọng

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "vitest"
  },
  "dependencies": {},
  "devDependencies": {},
  "engines": { "node": ">=20.0.0" }
}
```

### 2.3 Semantic Versioning (semver)

```
MAJOR.MINOR.PATCH
^1.2.3  → >=1.2.3 <2.0.0  (minor + patch updates OK)
~1.2.3  → >=1.2.3 <1.3.0  (patch updates only)
1.2.3   → exact version
```

### 2.4 npx

```bash
npx create-react-app my-app  # chạy package mà không cần cài global
npx prisma migrate dev        # run CLI của local package
```

### 2.5 Alternatives

| Tool | Điểm khác biệt |
|------|----------------|
| **yarn** | Parallel installs, workspaces |
| **pnpm** | Symlinks, tiết kiệm disk space |
| **bun** | Siêu nhanh, all-in-one runtime+bundler |

**Bài tập**: Tạo project, cấu hình scripts, thêm `engines` field, publish lên npm (optional).

---

## Phase 3 — Error Handling

```js
// Error-first callback pattern (Node.js convention cũ)
fs.readFile('./file', (err, data) => {
  if (err) throw err
  // ...
})

// Custom Error classes
class AppError extends Error {
  constructor(message, statusCode) {
    super(message)
    this.statusCode = statusCode
    this.name = this.constructor.name
  }
}

// Unhandled errors — PHẢI handle ở process level
process.on('uncaughtException', (err) => {
  console.error('Uncaught:', err)
  process.exit(1)  // restart via PM2/Docker
})
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason)
  process.exit(1)
})
```

---

## Phase 4 — Async Programming

### 4.1 Callbacks → Promises → async/await

```js
// Callbacks (callback hell)
fs.readFile('./a', (err, a) => {
  fs.readFile('./b', (err, b) => {
    // ...
  })
})

// Promises
readFile('./a').then(a => readFile('./b')).then(b => {})

// async/await (prefer)
const [a, b] = await Promise.all([readFile('./a'), readFile('./b')])
```

### 4.2 Promisify

```js
const { promisify } = require('util')
const readFile = promisify(fs.readFile)

// Hoặc dùng fs/promises trực tiếp
const { readFile } = require('fs/promises')
```

### 4.3 Common Patterns

```js
// Parallel (nhanh nhất)
const results = await Promise.all([fetchA(), fetchB(), fetchC()])

// Sequential
for (const item of items) {
  await processItem(item)
}

// Race — lấy cái nào xong trước
const result = await Promise.race([fetch(url), timeout(5000)])

// AllSettled — không throw dù có reject
const results = await Promise.allSettled([p1, p2, p3])
results.forEach(r => {
  if (r.status === 'fulfilled') use(r.value)
})
```

---

## Phase 5 — Frameworks

### 5.1 Express.js (Junior — phải nắm vững)

```js
const express = require('express')
const app = express()

// Built-in middleware
app.use(express.json())
app.use(express.urlencoded({ extended: true }))

// Custom middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`)
  next()
})

// Routes
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.findAll()
    res.json(users)
  } catch (err) {
    next(err)  // forward to error handler
  }
})

// Router module hóa
const userRouter = require('./routes/users')
app.use('/api/users', userRouter)

// Global Error Handler (4 params, đặt CUỐI cùng)
app.use((err, req, res, next) => {
  const status = err.statusCode || 500
  res.status(status).json({ error: err.message })
})

app.listen(3000)
```

**Bài tập Express:**
1. CRUD API cho `todos` (không cần DB, dùng array in-memory)
2. Thêm input validation với **Zod**
3. Thêm auth middleware (fake JWT check)
4. Rate limiting với `express-rate-limit`

### 5.2 Fastify (Mid — biết sự khác biệt)

- Schema-first: JSON Schema cho request/response → validation + serialization tự động
- Plugin system với `fastify-plugin`
- Nhanh hơn Express ở raw throughput
- Built-in logging với Pino

```js
const fastify = require('fastify')({ logger: true })

fastify.get('/users', {
  schema: {
    response: {
      200: {
        type: 'array',
        items: { type: 'object', properties: { id: { type: 'number' } } }
      }
    }
  }
}, async (request, reply) => {
  return [{ id: 1 }]
})

await fastify.listen({ port: 3000 })
```

### 5.3 NestJS (Senior — enterprise)

- Opinionated framework: TypeScript-first, Angular-like architecture
- Dependency Injection, Modules, Controllers, Services, Guards, Interceptors, Pipes

```
src/
  app.module.ts       ← root module
  users/
    users.module.ts
    users.controller.ts
    users.service.ts
    users.entity.ts
```

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() { return this.usersService.findAll() }

  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() dto: CreateUserDto) { return this.usersService.create(dto) }
}
```

### 5.4 Hono (Modern — lightweight)

- Runs on Cloudflare Workers, Bun, Deno, Node.js
- Web Standards API (Request/Response)
- Excellent for edge computing

---

## Phase 6 — Databases

### 6.1 SQL — PostgreSQL

**Driver thuần (node-postgres):**
```js
const { Pool } = require('pg')
const pool = new Pool({ connectionString: process.env.DATABASE_URL })

// Parameterized query — tránh SQL injection
const { rows } = await pool.query('SELECT * FROM users WHERE id = $1', [id])
```

**Query Builder — Knex:**
```js
const users = await knex('users').where({ active: true }).orderBy('created_at', 'desc').limit(10)
```

**ORM — Prisma (khuyên dùng):**
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  posts     Post[]
  createdAt DateTime @default(now())
}
```
```js
const user = await prisma.user.create({ data: { email: 'a@b.com' } })
const users = await prisma.user.findMany({ where: { active: true }, include: { posts: true } })
```

**ORM — Drizzle (type-safe, modern alternative):**
```ts
const users = await db.select().from(usersTable).where(eq(usersTable.active, true))
```

**ORM — Sequelize (traditional):**
```js
const User = sequelize.define('User', { email: DataTypes.STRING })
await User.findAll({ where: { active: true } })
```

### 6.2 NoSQL — MongoDB + Mongoose

**Mongoose là ODM** (Object Document Mapper), KHÔNG phải ORM. (verified 3-0 từ MDN)

```js
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 0, max: 150 },
  role: { type: String, enum: ['user', 'admin'], default: 'user' }
})

const User = mongoose.model('User', userSchema)

// CRUD
const user = await User.create({ email: 'a@b.com', age: 25 })
const users = await User.find({ age: { $gte: 18 } }).sort('-createdAt').limit(10)
await User.findByIdAndUpdate(id, { $set: { age: 26 } }, { new: true })
```

**Mongoose Validators (verified 3-0):**
- Tất cả SchemaTypes: `required`
- Number: `min`, `max`
- String: `enum`, `match`, `maxLength`, `minLength`

**Populate — thay cho JOIN (verified 3-0):**
```js
const postSchema = new mongoose.Schema({
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
})

// Populate replace ObjectId bằng document thật
const post = await Post.findById(id).populate('author', 'name email')
```

### 6.3 Redis

- In-memory key-value store
- Dùng cho: caching, session storage, rate limiting, pub/sub, job queues

```js
const { createClient } = require('redis')
const client = createClient({ url: process.env.REDIS_URL })
await client.connect()

// Cache-aside pattern
async function getUser(id) {
  const cached = await client.get(`user:${id}`)
  if (cached) return JSON.parse(cached)

  const user = await db.user.findById(id)
  await client.setEx(`user:${id}`, 3600, JSON.stringify(user))  // TTL 1 hour
  return user
}

// Pub/Sub
const subscriber = client.duplicate()
await subscriber.subscribe('notifications', (message) => console.log(message))
await client.publish('notifications', JSON.stringify({ type: 'alert' }))
```

**Bài tập Database:**
1. CRUD API với Prisma + PostgreSQL (trong Docker)
2. Same API với Mongoose + MongoDB
3. Thêm Redis cache layer cho GET endpoints

---

## Phase 7 — Authentication & Authorization

### 7.1 Password Hashing với bcrypt

```js
const bcrypt = require('bcrypt')
const SALT_ROUNDS = 12  // 10-12 là standard

const hash = await bcrypt.hash(password, SALT_ROUNDS)
const isValid = await bcrypt.compare(plaintext, hash)
```

### 7.2 JWT từ đầu

```js
const jwt = require('jsonwebtoken')

// Sign
const accessToken = jwt.sign(
  { userId: user.id, role: user.role },  // payload
  process.env.JWT_SECRET,                // secret
  { expiresIn: '15m' }                  // options
)

// Verify
try {
  const payload = jwt.verify(token, process.env.JWT_SECRET)
  // payload.userId, payload.role, payload.exp, payload.iat
} catch (err) {
  // TokenExpiredError, JsonWebTokenError
}
```

**Access Token + Refresh Token flow:**
```
Client          Server
  │── POST /login ───────────────────────────────►│
  │◄── { accessToken (15m), refreshToken (7d) } ──│  store refreshToken in DB
  │
  │── GET /api/data + Authorization: Bearer <access> ─►│
  │◄── data ──────────────────────────────────────────│
  │
  │ (access expired)
  │── POST /auth/refresh + { refreshToken } ──────────►│  verify in DB
  │◄── { new accessToken } ───────────────────────────│
  │
  │── POST /logout ────────────────────────────────────►│  delete refreshToken from DB
```

### 7.3 Auth Middleware

```js
const authenticate = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1]
  if (!token) return res.status(401).json({ error: 'No token' })

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET)
    next()
  } catch {
    res.status(401).json({ error: 'Invalid token' })
  }
}

// RBAC
const authorize = (...roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) return res.status(403).json({ error: 'Forbidden' })
  next()
}

app.get('/admin', authenticate, authorize('admin'), handler)
```

### 7.4 OAuth2 / OIDC

- **Authorization Code flow**: User → App → Auth Server (Google/GitHub) → App
- **PKCE**: cho public clients (SPA, mobile) — thêm `code_verifier`/`code_challenge`
- **OpenID Connect**: OAuth2 + Identity layer → `id_token` (JWT chứa user info)
- **Passport.js**: middleware library cho nhiều strategies (local, google, github, jwt)

### 7.5 Sessions vs JWT

| | Sessions | JWT |
|--|---------|-----|
| Storage | Server-side (Redis/DB) | Client-side |
| Revocation | Dễ (xóa từ store) | Khó (phải dùng blacklist) |
| Scale | Cần shared store | Stateless — dễ scale |
| Dùng khi | Truyền thống web app | SPA, mobile, microservices |

---

## Phase 8 — Security

### 8.1 Helmet.js (security headers)

```js
const helmet = require('helmet')
app.use(helmet())  // tự động set CSP, HSTS, X-Frame-Options, X-Content-Type, etc.
```

### 8.2 CORS

```js
const cors = require('cors')
app.use(cors({
  origin: ['https://myapp.com'],  // KHÔNG dùng '*' ở production
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Authorization', 'Content-Type'],
  credentials: true
}))
```

### 8.3 Rate Limiting

```js
const rateLimit = require('express-rate-limit')
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  message: { error: 'Too many requests' }
}))
```

### 8.4 Input Validation với Zod

```js
const { z } = require('zod')

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  age: z.number().int().min(18).max(150).optional()
})

// Middleware
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body)
  if (!result.success) return res.status(400).json({ errors: result.error.issues })
  req.body = result.data
  next()
}

app.post('/users', validate(createUserSchema), createUserHandler)
```

### 8.5 OWASP Top 10 cần biết

| Vulnerability | Phòng |
|---------------|-------|
| SQL Injection | Parameterized queries, ORM |
| XSS | Escape output, CSP header |
| Broken Auth | JWT best practices, bcrypt |
| IDOR | Kiểm tra ownership trước khi access |
| Security Misconfiguration | Helmet, disable verbose errors in prod |
| Sensitive Data Exposure | HTTPS, encrypt at rest |

---

## Phase 9 — Testing

### 9.1 Unit Testing với Jest / Vitest

```js
// vitest (recommended — faster, ESM native)
import { describe, it, expect, vi } from 'vitest'

describe('UserService', () => {
  it('should hash password before saving', async () => {
    const mockRepo = { create: vi.fn().mockResolvedValue({ id: 1 }) }
    const service = new UserService(mockRepo)

    await service.createUser({ email: 'a@b.com', password: '123456' })

    const call = mockRepo.create.mock.calls[0][0]
    expect(call.password).not.toBe('123456')
    expect(call.password).toMatch(/^\$2b\$/)  // bcrypt hash
  })
})
```

### 9.2 Integration Testing với Supertest

```js
import request from 'supertest'
import app from '../src/app'

describe('POST /api/users', () => {
  it('should return 201 with created user', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ email: 'test@test.com', password: 'password123' })
      .expect(201)

    expect(res.body).toHaveProperty('id')
    expect(res.body.email).toBe('test@test.com')
    expect(res.body).not.toHaveProperty('password')
  })
})
```

### 9.3 Test Database

```js
// Dùng SQLite in-memory cho test nhanh
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient({ datasources: { db: { url: 'file::memory:' } } })

// Hoặc testcontainers — spin up real PostgreSQL trong Docker
const { PostgreSqlContainer } = require('@testcontainers/postgresql')
const container = await new PostgreSqlContainer().start()
```

### 9.4 E2E Testing (Cypress / Playwright)

```js
// Playwright
import { test, expect } from '@playwright/test'

test('login flow', async ({ page }) => {
  await page.goto('/login')
  await page.fill('[name=email]', 'user@test.com')
  await page.fill('[name=password]', 'password')
  await page.click('button[type=submit]')
  await expect(page).toHaveURL('/dashboard')
})
```

**Bài tập Testing:**
1. Unit test cho `UserService` — mock database, test edge cases
2. Integration test cho tất cả API endpoints (GET, POST, PUT, DELETE)
3. Coverage report: đạt >70% cho business logic

---

## Phase 10 — Message Queues & Background Jobs

### 10.1 BullMQ (Redis-backed job queue)

```js
const { Queue, Worker } = require('bullmq')

// Producer
const emailQueue = new Queue('emails', { connection: { host: 'localhost' } })
await emailQueue.add('send-welcome', { userId: 1, email: 'a@b.com' }, {
  delay: 0,
  attempts: 3,
  backoff: { type: 'exponential', delay: 2000 }
})

// Consumer (Worker)
new Worker('emails', async (job) => {
  const { userId, email } = job.data
  await sendEmail(email, 'Welcome!')
}, { connection: { host: 'localhost' } })
```

### 10.2 RabbitMQ

```js
const amqp = require('amqplib')

// Publisher
const conn = await amqp.connect('amqp://localhost')
const channel = await conn.createChannel()
await channel.assertQueue('tasks', { durable: true })
channel.sendToQueue('tasks', Buffer.from(JSON.stringify({ type: 'process' })), { persistent: true })

// Consumer
channel.consume('tasks', (msg) => {
  const data = JSON.parse(msg.content.toString())
  // process...
  channel.ack(msg)
})
```

**Khi nào dùng gì:**
- **BullMQ**: Simple job queues, cron jobs, retry logic — dùng chung Redis sẵn có
- **RabbitMQ**: Complex routing (fanout, direct, topic), dead letter queues, nhiều consumers

---

## Phase 11 — Real-time Communication

### 11.1 WebSockets với Socket.io

```js
const { Server } = require('socket.io')
const io = new Server(httpServer, { cors: { origin: '*' } })

io.on('connection', (socket) => {
  console.log('connected:', socket.id)

  socket.on('join-room', (roomId) => socket.join(roomId))
  socket.on('send-message', ({ roomId, text }) => {
    io.to(roomId).emit('new-message', { text, from: socket.id })
  })

  socket.on('disconnect', () => console.log('disconnected:', socket.id))
})
```

### 11.2 Server-Sent Events (SSE)

```js
// Server
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream')
  res.setHeader('Cache-Control', 'no-cache')
  res.setHeader('Connection', 'keep-alive')

  const send = (data) => res.write(`data: ${JSON.stringify(data)}\n\n`)
  const interval = setInterval(() => send({ time: Date.now() }), 1000)

  req.on('close', () => clearInterval(interval))
})

// Client (browser)
const evtSource = new EventSource('/events')
evtSource.onmessage = (e) => console.log(JSON.parse(e.data))
```

**WebSocket vs SSE:**
| | WebSocket | SSE |
|--|-----------|-----|
| Direction | Bidirectional | Server → Client only |
| Protocol | WS/WSS | HTTP |
| Reconnect | Manual | Automatic |
| Dùng khi | Chat, games, live collab | Notifications, live feed, progress |

---

## Phase 12 — Deployment & Production

### 12.1 Environment Configuration

```js
// .env
PORT=3000
NODE_ENV=production
DATABASE_URL=postgresql://...
JWT_SECRET=your-secret

// .env.example (commit cái này, KHÔNG commit .env)
PORT=3000
NODE_ENV=development
DATABASE_URL=
JWT_SECRET=
```

### 12.2 PM2 (Process Manager)

```bash
pm2 start src/index.js --name api --instances max  # cluster mode
pm2 logs api
pm2 restart api
pm2 monit
pm2 save && pm2 startup  # auto-start on reboot

# ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api',
    script: 'src/index.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: { NODE_ENV: 'production' }
  }]
}
```

### 12.3 Docker

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json .
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "src/index.js"]
```

```yaml
# docker-compose.yml
services:
  api:
    build: .
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgresql://postgres:pass@db:5432/mydb
    depends_on: [db, redis]

  db:
    image: postgres:16-alpine
    environment: { POSTGRES_PASSWORD: pass }
    volumes: [postgres-data:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine

volumes:
  postgres-data:
```

### 12.4 Graceful Shutdown

```js
const server = app.listen(3000)

const shutdown = async (signal) => {
  console.log(`Received ${signal}`)
  server.close(async () => {  // stop accepting new connections
    await prisma.$disconnect()  // close DB connections
    await redisClient.quit()
    process.exit(0)
  })
}

process.on('SIGTERM', () => shutdown('SIGTERM'))  // Docker stop
process.on('SIGINT', () => shutdown('SIGINT'))    // Ctrl+C
```

### 12.5 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: test }
        options: --health-cmd pg_isready
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm test
      - run: npm run build
```

---

## Phase 13 — Performance & Monitoring

### 13.1 Logging

```js
// Pino — fastest JSON logger
const pino = require('pino')
const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: { target: 'pino-pretty' }  // dev only
})

logger.info({ userId: 1 }, 'User logged in')
logger.error({ err, requestId }, 'Request failed')
```

### 13.2 Profiling

```bash
# CPU profiling
node --prof src/index.js
node --prof-process isolate-*.log > profile.txt

# Memory profiling
node --inspect src/index.js  # Chrome DevTools
```

### 13.3 OpenTelemetry (tracing + metrics)

```js
const { NodeSDK } = require('@opentelemetry/sdk-node')
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node')

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({ url: 'http://jaeger:4317' }),
  instrumentations: [getNodeAutoInstrumentations()]
})
sdk.start()

// Tự động instrument: HTTP, Express, pg, MongoDB, Redis...
```

### 13.4 Performance Tips

- `--max-old-space-size=512` để limit memory
- Dùng `compression` middleware cho HTTP responses
- Connection pooling cho DB (PG pool size = CPU cores × 2)
- Cache static data trong Redis (TTL phù hợp)
- Avoid blocking event loop: đẩy CPU work sang Worker Threads

---

## Phase 14 — CLI Tools

### 14.1 Commander + Inquirer + Chalk

```js
const { Command } = require('commander')
const inquirer = require('inquirer')
const chalk = require('chalk')

const program = new Command()
program
  .name('mytool')
  .description('My CLI tool')
  .version('1.0.0')

program
  .command('init <name>')
  .option('-t, --template <type>', 'template type', 'default')
  .action(async (name, options) => {
    const { confirm } = await inquirer.prompt([{
      type: 'confirm', name: 'confirm', message: `Create project "${name}"?`
    }])
    if (confirm) {
      console.log(chalk.green('✓'), `Created ${name}`)
    }
  })

program.parse()
```

---

## Phase 15 — Advanced Architecture

### 15.1 Repository Pattern

```js
// Tách data access khỏi business logic
class UserRepository {
  async findById(id) { return prisma.user.findUnique({ where: { id } }) }
  async create(data) { return prisma.user.create({ data }) }
  async update(id, data) { return prisma.user.update({ where: { id }, data }) }
}

class UserService {
  constructor(private userRepo: UserRepository) {}  // inject dependency
  async createUser(dto) {
    const hash = await bcrypt.hash(dto.password, 12)
    return this.userRepo.create({ ...dto, password: hash })
  }
}
```

### 15.2 Clean Architecture (Dependency Rule)

```
domain/          ← pure business logic, zero dependencies
  entities/
  use-cases/
application/     ← orchestration, depends on domain
  services/
infrastructure/  ← DB, HTTP, Redis — depends on application
  repositories/
  controllers/
```

### 15.3 gRPC

```js
// Protocol Buffers — binary serialization (nhanh hơn JSON)
// .proto file
service UserService {
  rpc GetUser (GetUserRequest) returns (User);
}

// Node.js
const { loadPackageDefinition } = require('@grpc/grpc-js')
const packageDefinition = protoLoader.loadSync('./user.proto')

// Dùng khi: microservices internal communication
// Dùng REST khi: public API
```

---

## Roadmap Summary — Thứ tự học

```
Phase 0 → Prerequisite (JS, Git, Terminal, HTTP)
Phase 1 → Runtime core (Event Loop, Modules, Built-ins, Streams)
Phase 2 → npm ecosystem
Phase 3 → Error Handling
Phase 4 → Async patterns
Phase 5 → Express.js (Junior checkpoint ★)
Phase 6 → Databases (SQL + MongoDB + Redis)
Phase 7 → Auth (JWT, OAuth2, RBAC) (Mid checkpoint ★★)
Phase 8 → Security (OWASP, Helmet, Rate limiting)
Phase 9 → Testing (Unit, Integration, E2E)
Phase 10 → Message Queues (BullMQ, RabbitMQ)
Phase 11 → Real-time (WebSockets, SSE)
Phase 12 → Deployment (Docker, PM2, CI/CD)
Phase 13 → Performance & Monitoring (Senior checkpoint ★★★)
Phase 14 → CLI Tools
Phase 15 → Architecture (Clean Arch, Repository, gRPC)
```

---

## Projects theo thứ tự (từ roadmap.sh/nodejs/projects)

### Beginner Projects
1. **Task Tracker CLI** — `commander` + `fs` để lưu tasks vào JSON file
2. **GitHub User Activity CLI** — gọi GitHub API, hiển thị recent events
3. **Expense Tracker CLI** — CRUD, categories, monthly summary
4. **Blogging Platform API** — CRUD posts, Express + in-memory storage
5. **Todo List API** — REST API đầy đủ với validation

### Intermediate Projects
6. **Blog API với Auth** — JWT, bcrypt, PostgreSQL/Prisma
7. **E-commerce API** — Products, orders, payments (Stripe mock)
8. **Real-time Chat** — Socket.io, rooms, typing indicators
9. **Job Queue System** — BullMQ, email/notification workers
10. **URL Shortener** — Redis cache, analytics

### Advanced Projects
11. **API Gateway** — Rate limiting, auth, routing to microservices
12. **Microservices** — 3+ services, RabbitMQ, service discovery
13. **Full-stack App** — NestJS + React + PostgreSQL + Redis + Docker

---

## Resources

| Resource | Mục đích |
|---------|---------|
| [nodejs.org/docs](https://nodejs.org/docs/latest/api/) | Official API reference |
| [roadmap.sh/nodejs](https://roadmap.sh/nodejs) | Visual roadmap |
| [learnyounode](https://github.com/workshopper/learnyounode) | Interactive CLI workshop (13 exercises) |
| **Node.js Design Patterns** — Mario Casciaro | Sách hay nhất về Node.js internals |
| [expressjs.com/guide](https://expressjs.com/en/guide/routing.html) | Express guide |
| [docs.nestjs.com](https://docs.nestjs.com) | NestJS official |
| [prisma.io/docs](https://www.prisma.io/docs) | Prisma ORM |
| [bullmq.io](https://docs.bullmq.io) | BullMQ job queues |
| [socket.io/docs](https://socket.io/docs/v4/) | Socket.io |
