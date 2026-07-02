# Phase 5.2 — Fastify

Fastify là web framework nhanh nhất trong Node.js ecosystem, được thiết kế với performance và developer experience là ưu tiên hàng đầu. Điểm khác biệt chính: **schema-first** (JSON Schema cho validation tự động), plugin system chặt chẽ, và built-in Pino logger.

---

## Tại sao Fastify?

- **Nhanh nhất**: ~2x so với Express ở raw throughput
- **Schema-first**: validate request/response tự động, generate docs
- **Type-safe**: TypeScript support tốt hơn Express
- **Built-in logging**: Pino logger, JSON logs mặc định
- **Plugin system**: dependency tracking, scope isolation

---

## Setup cơ bản

```js
const Fastify = require('fastify')

const fastify = Fastify({
  logger: {
    level: 'info',
    transport: {
      target: 'pino-pretty',  // chỉ dev — human-readable logs
      options: { colorize: true }
    }
  }
})

fastify.get('/health', async (request, reply) => {
  return { status: 'ok', uptime: process.uptime() }
})

const start = async () => {
  try {
    await fastify.listen({ port: 3000, host: '0.0.0.0' })
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}

start()
```

---

## Schema-First Validation

JSON Schema được định nghĩa trong route options — Fastify tự động validate và serialize:

```js
// Route với đầy đủ schema
fastify.post('/users', {
  schema: {
    // Validate request body
    body: {
      type: 'object',
      required: ['name', 'email', 'password'],
      properties: {
        name: { type: 'string', minLength: 1, maxLength: 100 },
        email: { type: 'string', format: 'email' },
        password: { type: 'string', minLength: 8 },
        age: { type: 'integer', minimum: 18, maximum: 150 }
      },
      additionalProperties: false  // reject unknown fields
    },
    // Validate query parameters
    querystring: {
      type: 'object',
      properties: {
        notify: { type: 'boolean', default: false }
      }
    },
    // Serialize (và filter) response
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          email: { type: 'string' }
          // password KHÔNG có → tự động stripped từ response!
        }
      },
      400: {
        type: 'object',
        properties: {
          error: { type: 'string' },
          message: { type: 'string' }
        }
      }
    }
  }
}, async (request, reply) => {
  const { name, email, password } = request.body
  const user = await createUser({ name, email, password })
  reply.status(201).send(user)
})
```

---

## Plugins — Encapsulation

Fastify plugin system dùng `fastify-plugin` để share context hoặc isolate scope:

```js
// plugins/database.js
const fp = require('fastify-plugin')
const { PrismaClient } = require('@prisma/client')

async function databasePlugin(fastify, options) {
  const prisma = new PrismaClient()

  await prisma.$connect()

  // Decorate fastify instance
  fastify.decorate('db', prisma)

  // Cleanup khi server đóng
  fastify.addHook('onClose', async () => {
    await prisma.$disconnect()
  })
}

// fp() cho phép plugin được share với parent scope
module.exports = fp(databasePlugin)
```

```js
// plugins/auth.js
const fp = require('fastify-plugin')
const jwt = require('@fastify/jwt')

async function authPlugin(fastify, options) {
  await fastify.register(jwt, {
    secret: process.env.JWT_SECRET
  })

  fastify.decorate('authenticate', async function(request, reply) {
    try {
      await request.jwtVerify()
    } catch (err) {
      reply.status(401).send({ error: 'Unauthorized' })
    }
  })
}

module.exports = fp(authPlugin)
```

```js
// app.js
const fastify = Fastify({ logger: true })

// Register plugins
await fastify.register(require('./plugins/database'))
await fastify.register(require('./plugins/auth'))

// Register routes với prefix và scope
await fastify.register(require('./routes/users'), { prefix: '/api/users' })
await fastify.register(require('./routes/posts'), { prefix: '/api/posts' })
```

---

## Routes Module

```js
// routes/users.js
async function userRoutes(fastify, options) {
  // fastify.db disponible vì databasePlugin dùng fastify-plugin

  fastify.get('/', {
    schema: {
      querystring: {
        type: 'object',
        properties: {
          page: { type: 'integer', default: 1 },
          limit: { type: 'integer', default: 10, maximum: 100 }
        }
      },
      response: {
        200: {
          type: 'array',
          items: {
            type: 'object',
            properties: {
              id: { type: 'integer' },
              name: { type: 'string' },
              email: { type: 'string' }
            }
          }
        }
      }
    }
  }, async (request, reply) => {
    const { page, limit } = request.query
    const users = await fastify.db.user.findMany({
      skip: (page - 1) * limit,
      take: limit
    })
    return users
  })

  // Protected route
  fastify.get('/me', {
    onRequest: [fastify.authenticate]
  }, async (request, reply) => {
    return request.user
  })
}

module.exports = userRoutes
```

---

## Hooks

```js
// Lifecycle hooks
fastify.addHook('onRequest', async (request, reply) => {
  request.startTime = Date.now()
})

fastify.addHook('onResponse', async (request, reply) => {
  const duration = Date.now() - request.startTime
  fastify.log.info({ duration, path: request.url }, 'Request completed')
})

fastify.addHook('preHandler', async (request, reply) => {
  // Chạy trước route handler — validate, auth, etc.
})

// Hook chỉ áp dụng cho scope hiện tại (không cần fp())
fastify.register(async function(scope) {
  scope.addHook('preHandler', fastify.authenticate)

  scope.get('/profile', async (req, reply) => {
    return { user: req.user }
  })
})
```

---

## Error Handling

```js
// Custom error handler
fastify.setErrorHandler((err, request, reply) => {
  fastify.log.error({ err, requestId: request.id }, 'Request failed')

  if (err.validation) {
    // Schema validation error
    return reply.status(400).send({
      error: 'Validation failed',
      issues: err.validation
    })
  }

  if (err.statusCode) {
    return reply.status(err.statusCode).send({ error: err.message })
  }

  reply.status(500).send({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : err.message
  })
})

// Not found handler
fastify.setNotFoundHandler((request, reply) => {
  reply.status(404).send({
    error: `Route ${request.method} ${request.url} not found`
  })
})
```

---

## TypeScript Support

```typescript
import Fastify, { FastifyInstance, FastifyRequest, FastifyReply } from 'fastify'

interface CreateUserBody {
  name: string
  email: string
  password: string
}

const server: FastifyInstance = Fastify({ logger: true })

server.post<{ Body: CreateUserBody }>(
  '/users',
  {
    schema: {
      body: {
        type: 'object',
        required: ['name', 'email', 'password'],
        properties: {
          name: { type: 'string' },
          email: { type: 'string', format: 'email' },
          password: { type: 'string', minLength: 8 }
        }
      }
    }
  },
  async (request: FastifyRequest<{ Body: CreateUserBody }>, reply: FastifyReply) => {
    const { name, email, password } = request.body
    // TypeScript biết types của body
    return reply.status(201).send({ name, email })
  }
)
```

---

## Bài tập thực hành

1. **Schema-first API**: Build REST API cho `products` với JSON Schema cho mọi endpoint. Verify rằng invalid data bị reject và password field bị strip khỏi responses.

2. **Plugin system**: Tách database connection, JWT auth, và rate limiting thành separate plugins. Register theo đúng thứ tự dependencies.

3. **Performance comparison**: Benchmark cùng một API endpoint trên Express vs Fastify dùng `autocannon`. Đo: requests/second, latency p50/p99.

4. **OpenAPI generation**: Dùng `@fastify/swagger` để auto-generate OpenAPI 3.0 spec từ JSON schemas. Serve Swagger UI tại `/docs`.

---

## Resources

- [fastify.dev — Getting Started](https://fastify.dev/docs/latest/Guides/Getting-Started/) — Official guide
- [fastify.dev — Plugins](https://fastify.dev/docs/latest/Reference/Plugins/) — Plugin system
- [@fastify ecosystem plugins](https://github.com/fastify/fastify/blob/main/docs/Guides/Ecosystem.md) — Official plugins list
