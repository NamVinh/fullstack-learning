# Phase 5.1 — Express.js

Express.js là framework Node.js phổ biến nhất — minimal, unopinionated, và có ecosystem packages lớn nhất. Junior developer **phải nắm vững** Express trước khi học frameworks khác. Hầu hết các concepts (middleware, routing, error handling) của Express apply sang các frameworks khác.

---

## Setup cơ bản

```js
const express = require('express')
const app = express()

// Built-in middleware
app.use(express.json())                       // parse JSON body
app.use(express.urlencoded({ extended: true }))  // parse form data

// Third-party middleware
const helmet = require('helmet')
const cors = require('cors')
app.use(helmet())   // security headers
app.use(cors())     // CORS

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000')
})
```

---

## Routing

```js
// Basic routes
app.get('/users', getUsers)
app.post('/users', createUser)
app.put('/users/:id', updateUser)
app.patch('/users/:id', patchUser)
app.delete('/users/:id', deleteUser)

// Route params, query, body
app.get('/users/:id', (req, res) => {
  const { id } = req.params           // /users/42 → id = '42'
  const { page = 1, limit = 10 } = req.query  // ?page=2&limit=5
  res.json({ id, page, limit })
})

app.post('/users', (req, res) => {
  const { name, email, age } = req.body  // từ JSON body
  res.status(201).json({ name, email, age })
})

// Route chaining
app.route('/users/:id')
  .get(getUser)
  .put(updateUser)
  .delete(deleteUser)
```

---

## Router — Modularize Routes

```js
// routes/users.js
const express = require('express')
const router = express.Router()

router.get('/', async (req, res, next) => {
  try {
    const users = await User.findAll()
    res.json(users)
  } catch (err) {
    next(err)  // forward error sang error handler
  }
})

router.post('/', validate(createUserSchema), async (req, res, next) => {
  try {
    const user = await User.create(req.body)
    res.status(201).json(user)
  } catch (err) {
    next(err)
  }
})

router.get('/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id)
    if (!user) return res.status(404).json({ error: 'User not found' })
    res.json(user)
  } catch (err) {
    next(err)
  }
})

module.exports = router
```

```js
// app.js
const userRouter = require('./routes/users')
const postRouter = require('./routes/posts')

app.use('/api/users', userRouter)   // /api/users, /api/users/:id
app.use('/api/posts', postRouter)   // /api/posts, /api/posts/:id
```

---

## Middleware

Middleware là functions có signature `(req, res, next)` — xử lý request và hoặc chuyển sang middleware tiếp theo.

```js
// Application-level middleware
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} ${req.method} ${req.path}`)
  next()  // phải gọi next() để tiếp tục chain
})

// Route-level middleware
app.get('/admin', authenticate, authorize('admin'), adminHandler)

// Error-handling middleware — 4 params (err, req, res, next)
app.use((err, req, res, next) => {
  console.error(err)
  res.status(err.statusCode || 500).json({ error: err.message })
})
```

```js
// Useful middleware patterns

// Authentication middleware
const authenticate = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1]
  if (!token) return res.status(401).json({ error: 'No token provided' })

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET)
    next()
  } catch {
    res.status(401).json({ error: 'Invalid or expired token' })
  }
}

// Async wrapper — tránh try/catch lặp lại
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next)
}

app.get('/users', asyncHandler(async (req, res) => {
  const users = await db.user.findMany()
  res.json(users)
}))

// Request ID middleware
const { randomUUID } = require('crypto')
app.use((req, res, next) => {
  req.id = randomUUID()
  res.setHeader('X-Request-ID', req.id)
  next()
})
```

---

## Validation với Zod

```js
const { z } = require('zod')

const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string().min(8).max(100),
  age: z.number().int().min(18).max(150).optional()
})

// Validation middleware factory
const validate = (schema, source = 'body') => (req, res, next) => {
  const result = schema.safeParse(req[source])
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation failed',
      issues: result.error.issues.map(i => ({
        field: i.path.join('.'),
        message: i.message
      }))
    })
  }
  req[source] = result.data  // replace with parsed/transformed data
  next()
}

// Usage
app.post('/users', validate(createUserSchema), createUserHandler)
```

---

## Rate Limiting

```js
const rateLimit = require('express-rate-limit')

// Global rate limit
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 phút
  max: 100,                   // 100 requests per IP per window
  standardHeaders: true,      // Return rate limit info in headers
  legacyHeaders: false,
  message: { error: 'Too many requests, please try again later' }
}))

// Stricter limit cho auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,
  message: { error: 'Too many login attempts' }
})

app.post('/auth/login', authLimiter, loginHandler)
app.post('/auth/register', authLimiter, registerHandler)
```

---

## Global Error Handler — Pattern đầy đủ

```js
// Global error handler (đặt CUỐI app.js, sau tất cả routes)
app.use((err, req, res, next) => {
  // Log error với context
  console.error({
    message: err.message,
    stack: process.env.NODE_ENV !== 'production' ? err.stack : undefined,
    requestId: req.id,
    path: req.path,
    method: req.method
  })

  // Prisma errors
  if (err.code === 'P2002') {  // Unique constraint
    return res.status(409).json({ error: 'Resource already exists' })
  }
  if (err.code === 'P2025') {  // Record not found
    return res.status(404).json({ error: 'Resource not found' })
  }

  // Zod validation
  if (err.name === 'ZodError') {
    return res.status(400).json({ error: 'Validation failed', issues: err.issues })
  }

  // JWT
  if (err.name === 'JsonWebTokenError') return res.status(401).json({ error: 'Invalid token' })
  if (err.name === 'TokenExpiredError') return res.status(401).json({ error: 'Token expired' })

  // Custom AppError
  if (err.isOperational) {
    return res.status(err.statusCode || 400).json({ error: err.message })
  }

  // Unexpected error
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : err.message
  })
})
```

---

## Bài tập thực hành

1. **CRUD API**: Build REST API cho `todos` (in-memory array, không cần DB) — GET all, GET by id, POST, PUT, DELETE. Đúng HTTP methods, status codes, và JSON response format.

2. **Validation + Error handling**: Thêm Zod validation cho POST/PUT, custom error classes, global error handler. Test với invalid data.

3. **Auth middleware**: Implement fake JWT auth (sign/verify với `jsonwebtoken`) — protected routes, role-based access (`admin` vs `user`), refresh token endpoint.

4. **Rate limiting + Logging**: Thêm rate limiting (10 req/min cho auth, 100 req/15min globally), request ID header, structured logging với request/response details.

---

## Resources

- [Express.js guide](https://expressjs.com/en/guide/routing.html) — Official routing guide
- [Express.js API reference](https://expressjs.com/en/4x/api.html) — Full API docs
- [express-rate-limit](https://github.com/express-rate-limit/express-rate-limit) — Rate limiting
