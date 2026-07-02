# Phase 8 — Security

Security không phải là tính năng thêm vào sau — nó phải được design từ đầu. Node.js applications cần được protect chống lại OWASP Top 10 vulnerabilities. Phần này cover các security measures cụ thể với code examples.

---

## Helmet.js — Security Headers

Helmet set các HTTP security headers ngăn chặn nhiều loại attacks:

```js
const helmet = require('helmet')

// Default: CSP, HSTS, X-Frame-Options, X-Content-Type-Options, etc.
app.use(helmet())

// Custom configuration
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", 'trusted-cdn.com'],
      imgSrc: ["'self'", 'data:', 'cdn.example.com'],
      connectSrc: ["'self'", 'api.example.com'],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000,        // 1 year
    includeSubDomains: true,
    preload: true
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}))
```

**Headers được set bởi Helmet:**

| Header | Chống lại |
|--------|-----------|
| `Content-Security-Policy` | XSS, injection attacks |
| `Strict-Transport-Security` | Downgrade attacks, MITM |
| `X-Frame-Options: DENY` | Clickjacking |
| `X-Content-Type-Options: nosniff` | MIME sniffing |
| `Referrer-Policy` | Information leakage |
| `X-XSS-Protection` | Legacy XSS (modern browsers ignore) |

---

## CORS — Cross-Origin Resource Sharing

```js
const cors = require('cors')

// Development — cho phép tất cả
if (process.env.NODE_ENV === 'development') {
  app.use(cors())
}

// Production — explicit whitelist
const allowedOrigins = [
  'https://myapp.com',
  'https://www.myapp.com',
  'https://admin.myapp.com'
]

app.use(cors({
  origin: (origin, callback) => {
    // Cho phép requests không có origin (mobile apps, Postman)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error(`CORS: Origin ${origin} not allowed`))
    }
  },
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Authorization', 'Content-Type', 'X-Request-ID'],
  credentials: true,          // cho phép cookies
  maxAge: 86400               // preflight cache 24h
}))
```

---

## Rate Limiting

```js
const rateLimit = require('express-rate-limit')
const RedisStore = require('rate-limit-redis')
const { createClient } = require('redis')

const redisClient = createClient({ url: process.env.REDIS_URL })

// Global rate limit
const globalLimit = rateLimit({
  windowMs: 15 * 60 * 1000,   // 15 phút
  max: 100,                    // 100 requests per IP per window
  standardHeaders: 'draft-7',
  legacyHeaders: false,
  store: new RedisStore({       // dùng Redis để share giữa nhiều servers
    sendCommand: (...args) => redisClient.sendCommand(args)
  }),
  message: { error: 'Too many requests', retryAfter: '15 minutes' },
  keyGenerator: (req) => req.ip  // hoặc req.user?.id cho authenticated users
})

// Stricter cho auth endpoints
const authLimit = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // chỉ 10 login attempts per 15 min
  message: { error: 'Too many login attempts' },
  skipSuccessfulRequests: false  // count cả successful logins
})

app.use('/api/', globalLimit)
app.use('/auth/login', authLimit)
app.use('/auth/register', authLimit)
```

---

## Input Validation với Zod

Validate và sanitize mọi input từ user — **không bao giờ trust user input**:

```js
const { z } = require('zod')

// Strict schema
const createUserSchema = z.object({
  name: z.string()
    .min(1, 'Name required')
    .max(100, 'Name too long')
    .trim(),
  email: z.string()
    .email('Invalid email format')
    .toLowerCase(),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .max(100, 'Password too long')
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[0-9]/, 'Must contain number'),
  age: z.number().int().min(18).max(150).optional()
})

// Middleware factory
const validate = (schema, source = 'body') => (req, res, next) => {
  const result = schema.safeParse(req[source])
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation failed',
      issues: result.error.issues.map(({ path, message }) => ({
        field: path.join('.'),
        message
      }))
    })
  }
  req[source] = result.data  // use transformed/sanitized data
  next()
}

app.post('/users', validate(createUserSchema), createUserHandler)

// URL params validation
const idSchema = z.object({ id: z.coerce.number().int().positive() })
app.get('/users/:id', validate(idSchema, 'params'), getUserHandler)
```

---

## SQL Injection Prevention

```js
// SAI — string concatenation → SQL injection
const userId = req.params.id
const query = `SELECT * FROM users WHERE id = ${userId}`
// nếu userId = "1 OR 1=1", query trả về tất cả users!

// ĐÚNG — parameterized queries (luôn dùng)
const { rows } = await pool.query('SELECT * FROM users WHERE id = $1', [userId])

// ĐÚNG — ORM (Prisma, etc.) tự handle
const user = await prisma.user.findUnique({ where: { id: Number(userId) } })

// Nếu cần dynamic query (ví dụ: filter fields)
// KHÔNG concatenate field names từ user input
const allowedFields = ['name', 'email', 'created_at']
const sortField = allowedFields.includes(req.query.sort) ? req.query.sort : 'created_at'
const query = `SELECT * FROM users ORDER BY ${sortField} DESC LIMIT $1`
// An toàn vì sortField được validate từ whitelist
```

---

## XSS Prevention

```js
// Server-side: escape HTML output
const escapeHtml = require('escape-html')

// Khi render HTML (nếu không dùng template engine)
const safeTitle = escapeHtml(post.title)
const html = `<h1>${safeTitle}</h1>`

// Khi trả về JSON API — browsers không execute JSON như HTML
// → Nhưng nếu client render JSON vào DOM mà không escape, vẫn có XSS

// CSP header ngăn execution của injected scripts
app.use(helmet.contentSecurityPolicy({
  directives: {
    scriptSrc: ["'self'"]  // chỉ scripts từ same origin
  }
}))

// HttpOnly cookies — JS không thể access
res.cookie('session', token, {
  httpOnly: true,   // không thể đọc bằng JS
  secure: true,     // chỉ HTTPS
  sameSite: 'strict'  // CSRF protection
})
```

---

## OWASP Top 10 — Checklist

| # | Vulnerability | Node.js Protection |
|---|---------------|-------------------|
| A01 | Broken Access Control | RBAC middleware, check ownership |
| A02 | Cryptographic Failures | HTTPS, bcrypt passwords, AES encryption |
| A03 | Injection | Parameterized queries, Zod validation |
| A04 | Insecure Design | Threat modeling, defense in depth |
| A05 | Security Misconfiguration | Helmet, disable verbose errors in prod |
| A06 | Vulnerable Components | `npm audit`, Snyk, dependabot |
| A07 | Auth Failures | bcrypt, JWT best practices, rate limiting |
| A08 | Data Integrity | Input validation, signed JWTs |
| A09 | Logging Failures | Structured logging, no sensitive data in logs |
| A10 | SSRF | Validate URLs, allowlist external services |

---

## Environment Variables Security

```js
// KHÔNG commit secrets vào git
// .gitignore
.env
.env.local
.env.production

// .env.example (commit cái này)
PORT=3000
DATABASE_URL=
JWT_SECRET=
REDIS_URL=

// Validate env vars khi startup
const requiredEnv = ['DATABASE_URL', 'JWT_SECRET', 'JWT_REFRESH_SECRET']
const missing = requiredEnv.filter(key => !process.env[key])
if (missing.length > 0) {
  console.error('Missing required env vars:', missing.join(', '))
  process.exit(1)
}

// Dùng Zod để validate env
import { z } from 'zod'
const envSchema = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development')
})

const env = envSchema.parse(process.env)
```

---

## Security Checklist cho Production

```
✓ Tất cả traffic qua HTTPS
✓ Helmet.js với CSP configured
✓ CORS whitelist (không dùng *)
✓ Rate limiting trên tất cả endpoints
✓ Input validation với Zod/Joi
✓ Parameterized SQL queries
✓ Passwords hashed với bcrypt (rounds >= 12)
✓ JWT secrets dài (>= 256-bit)
✓ Không log sensitive data (passwords, tokens, PII)
✓ npm audit — zero high/critical vulnerabilities
✓ Error messages không leak internal info ở production
✓ Dependencies updated regularly
✓ Secrets trong environment variables, không trong code
```

---

## Bài tập thực hành

1. **Security audit**: Lấy một Express app, tìm và fix: SQL injection point, missing Helmet, CORS misconfiguration, missing rate limiting.

2. **Penetration test**: Dùng OWASP ZAP hoặc Burp Suite để scan local app. Fix các vulnerabilities được báo cáo.

3. **Secrets management**: Implement env validation với Zod — app không start nếu thiếu required vars. Test với missing/invalid vars.

4. **Security headers check**: Deploy app lên local HTTPS (dùng `mkcert`), check headers với `securityheaders.com` hoặc `Mozilla Observatory`.

---

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/) — Web application security risks
- [Node.js Security Checklist](https://github.com/goldbergyoni/nodebestpractices#6-security-best-practices) — goldbergyoni/nodebestpractices
- [Snyk](https://snyk.io) — Dependency vulnerability scanner
