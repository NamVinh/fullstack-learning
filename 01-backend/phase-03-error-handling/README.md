# Phase 3 — Error Handling

Error handling tốt là dấu hiệu của code production-ready. Trong Node.js, errors có thể đến từ nhiều nguồn: synchronous throws, async callbacks, rejected Promises, và unhandled system errors. Không handle đúng cách sẽ dẫn đến server crash, data corruption, hoặc silent failures.

---

## Error-First Callback Pattern (Legacy)

Node.js convention truyền thống — callback luôn có `err` là tham số đầu tiên:

```js
const fs = require('fs')

fs.readFile('./config.json', 'utf8', (err, data) => {
  if (err) {
    // Xử lý lỗi — đừng để rỗng!
    if (err.code === 'ENOENT') {
      console.error('File not found:', err.path)
    } else {
      console.error('Read error:', err.message)
    }
    return  // QUAN TRỌNG: phải return, tránh code tiếp tục chạy
  }

  // Happy path
  const config = JSON.parse(data)
})
```

---

## Async/Await Error Handling

```js
// try/catch với async/await
async function getUser(id) {
  try {
    const user = await db.user.findById(id)
    if (!user) throw new NotFoundError(`User ${id} not found`)
    return user
  } catch (err) {
    if (err instanceof NotFoundError) throw err  // re-throw specific errors
    throw new DatabaseError('Failed to fetch user', { cause: err })
  }
}

// Avoid nesting — extract thành separate try/catch
async function processRequest(req) {
  let user
  try {
    user = await getUser(req.params.id)
  } catch (err) {
    if (err instanceof NotFoundError) {
      return res.status(404).json({ error: err.message })
    }
    throw err  // re-throw unexpected errors
  }

  // tiếp tục xử lý với user
}
```

---

## Custom Error Classes

Tạo error hierarchy rõ ràng giúp xử lý errors có type-specific behavior:

```js
// errors.js

// Base class cho tất cả app errors
class AppError extends Error {
  constructor(message, options = {}) {
    super(message)
    this.name = this.constructor.name
    this.statusCode = options.statusCode || 500
    this.isOperational = options.isOperational ?? true  // lỗi dự kiến vs unexpected
    if (options.cause) this.cause = options.cause       // wrap original error

    // Giữ stack trace đúng (V8 specific)
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor)
    }
  }
}

class ValidationError extends AppError {
  constructor(message, fields = {}) {
    super(message, { statusCode: 400 })
    this.fields = fields
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, { statusCode: 404 })
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Authentication required') {
    super(message, { statusCode: 401 })
  }
}

class ForbiddenError extends AppError {
  constructor(message = 'Access denied') {
    super(message, { statusCode: 403 })
  }
}

class ConflictError extends AppError {
  constructor(message) {
    super(message, { statusCode: 409 })
  }
}

class DatabaseError extends AppError {
  constructor(message, options = {}) {
    super(message, { statusCode: 500, isOperational: false, ...options })
  }
}

module.exports = {
  AppError,
  ValidationError,
  NotFoundError,
  UnauthorizedError,
  ForbiddenError,
  ConflictError,
  DatabaseError
}
```

---

## Express Error Handler Middleware

```js
const { AppError } = require('./errors')

// Global error handler — phải có 4 params (err, req, res, next)
// Đặt CUỐI CÙNG trong Express app, sau tất cả routes
function errorHandler(err, req, res, next) {
  // Log tất cả errors
  console.error({
    error: err.message,
    stack: err.stack,
    requestId: req.id,
    path: req.path,
    method: req.method
  })

  // Xử lý các error types cụ thể
  if (err.name === 'ValidationError') {
    // Mongoose validation error
    return res.status(400).json({
      error: 'Validation failed',
      details: Object.values(err.errors).map(e => e.message)
    })
  }

  if (err.name === 'JsonWebTokenError') {
    return res.status(401).json({ error: 'Invalid token' })
  }

  if (err.name === 'TokenExpiredError') {
    return res.status(401).json({ error: 'Token expired' })
  }

  // AppError — lỗi có thể dự đoán
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: err.message,
      ...(err.fields && { fields: err.fields })  // validation fields nếu có
    })
  }

  // Unexpected error — không leak details ở production
  const isProduction = process.env.NODE_ENV === 'production'
  res.status(500).json({
    error: isProduction ? 'Internal server error' : err.message,
    ...(isProduction ? {} : { stack: err.stack })
  })
}

// Route không tìm thấy
function notFoundHandler(req, res) {
  res.status(404).json({ error: `Route ${req.method} ${req.path} not found` })
}

module.exports = { errorHandler, notFoundHandler }
```

```js
// app.js
const express = require('express')
const { errorHandler, notFoundHandler } = require('./middleware/error-handler')

const app = express()
app.use(express.json())

// Routes
app.use('/api/users', userRoutes)
app.use('/api/posts', postRoutes)

// 404 handler — sau tất cả routes
app.use(notFoundHandler)

// Error handler — phải là CUỐI CÙNG
app.use(errorHandler)
```

---

## Process-Level Error Handling

```js
// Phải có ở mọi Node.js app production

// Synchronous errors không được catch
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION — shutting down:', err)
  // Không thể tiếp tục sau uncaughtException một cách an toàn
  process.exit(1)  // PM2/Docker sẽ restart process
})

// Promise rejections không được handle
process.on('unhandledRejection', (reason, promise) => {
  console.error('UNHANDLED REJECTION:', reason)
  // Node.js 15+ sẽ crash, nhưng tốt nhất là tự xử lý
  process.exit(1)
})

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received — starting graceful shutdown')
  server.close(async () => {
    await db.$disconnect()
    process.exit(0)
  })
})
```

---

## Error Wrapping (Error Cause)

```js
// ES2022: Error cause — chain errors với context
try {
  await db.user.create(data)
} catch (err) {
  // Wrap với context, giữ original error
  throw new DatabaseError('Failed to create user', { cause: err })
}

// Khi log, có full chain:
// DatabaseError: Failed to create user
//   caused by: PrismaClientKnownRequestError: Unique constraint violated
```

---

## Result Pattern (Alternative to try/catch)

```js
// Tránh throw/catch bằng cách return Result object
class Result {
  static ok(value) { return { success: true, value } }
  static err(error) { return { success: false, error } }
}

async function findUser(id) {
  try {
    const user = await db.user.findById(id)
    if (!user) return Result.err(new NotFoundError('User'))
    return Result.ok(user)
  } catch (err) {
    return Result.err(new DatabaseError('findUser failed', { cause: err }))
  }
}

// Usage — không cần try/catch
const result = await findUser(id)
if (!result.success) {
  return res.status(result.error.statusCode || 500).json({ error: result.error.message })
}
const user = result.value
```

---

## Bài tập thực hành

1. **Error hierarchy**: Tạo complete error class hierarchy cho một blog API: `AppError`, `ValidationError`, `NotFoundError`, `AuthError`, `ForbiddenError`, `DatabaseError`. Mỗi class có `statusCode` và `toJSON()` method.

2. **Express error middleware**: Implement global error handler cho Express app — phân biệt các loại errors, không leak stack trace ở production, log đầy đủ với request context.

3. **Async wrapper**: Viết `asyncHandler(fn)` HOF (Higher-Order Function) để tránh phải viết try/catch trong mọi async route handler. So sánh code có và không có wrapper.

4. **Error monitoring integration**: Integrate một simple error logger ghi errors ra file JSON với: timestamp, error type, message, stack trace, request context (method, path, body).

---

## Resources

- [Node.js docs — Error Handling](https://nodejs.org/api/errors.html) — Official errors reference
- [Express — Error Handling](https://expressjs.com/en/guide/error-handling.html) — Express error handling guide
- [Node.js Best Practices — Error Handling](https://github.com/goldbergyoni/nodebestpractices#2-error-handling-practices) — goldbergyoni/nodebestpractices
