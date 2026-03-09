# Phase 1 — Backend với Node.js + Express

**Thời gian**: 6–8 tuần | **Mục tiêu**: Tư duy BE, không phải FE đóng giả BE

---

## Node.js Core

---

### Node.js Runtime — Khác gì Browser?

- [ ] Hiểu môi trường trước khi viết code

**Học:**
1. 📖 [Node.js — About](https://nodejs.org/en/about)
2. 📖 [Differences between Node.js and the Browser](https://nodejs.org/en/learn/getting-started/differences-between-nodejs-and-the-browser)
3. 📖 [Node.js — The process object](https://nodejs.org/api/process.html) — đọc qua các property quan trọng

**Exercises:**

🛠️ Khám phá Node.js runtime — chạy từng lệnh và đọc output:
```js
// Tạo file explore.js
console.log(process.version);      // Node version
console.log(process.platform);     // OS
console.log(process.env);          // Env variables
console.log(process.argv);         // Command line args
console.log(process.memoryUsage()); // Memory info

// Thứ này có trong browser không?
console.log(typeof window);        // undefined trong Node
console.log(typeof document);      // undefined trong Node
console.log(typeof global);        // đây là "window" của Node
```

🛠️ Viết script đọc file và in ra stdout theo từng dòng — không dùng readline package:
```js
// Dùng fs.readFileSync hoặc fs.createReadStream
// Chạy: node script.js myfile.txt
```

---

### Streams & Buffer

- [ ] Xử lý file lớn mà không load hết vào RAM

**Học:**
1. 📖 [Node.js Streams — Official Docs](https://nodejs.org/api/stream.html) — đọc phần Overview
2. 📖 [Node.js Streams: Everything you need to know](https://www.freecodecamp.org/news/node-js-streams-everything-you-need-to-know-c9141306be93/)
3. 📖 [Buffer — Node.js Docs](https://nodejs.org/api/buffer.html)

**Exercises:**

🛠️ So sánh 2 cách đọc file lớn (tạo file test 100MB):
```bash
# Tạo file test 100MB
dd if=/dev/zero of=bigfile.txt bs=1M count=100
```
```js
// Cách 1: readFileSync — đo memory usage
const start = process.memoryUsage().heapUsed;
const data = fs.readFileSync('bigfile.txt');
console.log('Memory used:', process.memoryUsage().heapUsed - start, 'bytes');

// Cách 2: Stream — đo memory usage
// Thấy sự khác biệt chưa?
```

🛠️ Viết file copy function dùng pipe:
```js
function copyFile(src, dest) {
  // Dùng fs.createReadStream, fs.createWriteStream và .pipe()
}
```

---

### EventEmitter Pattern

- [ ] Nền tảng của toàn bộ Node.js ecosystem

**Học:**
1. 📖 [Node.js Events — Official Docs](https://nodejs.org/api/events.html)
2. 📖 [EventEmitter Pattern — Node.js Design Patterns (summary)](https://www.nodejsdesignpatterns.com/blog/node-js-event-emitter/)

**Exercises:**

🛠️ Build một EventBus đơn giản từ đầu (không extend EventEmitter):
```ts
class EventBus {
  private listeners: Map<string, Function[]> = new Map();

  on(event: string, listener: Function): void { /* ... */ }
  off(event: string, listener: Function): void { /* ... */ }
  emit(event: string, ...args: any[]): void { /* ... */ }
  once(event: string, listener: Function): void { /* ... */ }
}
```

🛠️ Viết `FileWatcher` class dùng EventEmitter:
- Emit `'change'` khi file thay đổi (dùng `fs.watch`)
- Emit `'error'` khi có lỗi
- Usage: `watcher.on('change', (filename) => console.log(filename))`

---

## Express.js

---

### Middleware Pattern

- [ ] Đây là trái tim của Express — hiểu trước khi dùng

**Học:**
1. 📖 [Express — Writing middleware](https://expressjs.com/en/guide/writing-middleware.html)
2. 📖 [Express — Using middleware](https://expressjs.com/en/guide/using-middleware.html)

**Exercises:**

🛠️ Implement Express từ đầu (~50 dòng) để hiểu middleware chain:
```ts
class MiniExpress {
  private middlewares: Function[] = [];

  use(fn: Function) { this.middlewares.push(fn); }

  listen(port: number) {
    require('http').createServer((req, res) => {
      let i = 0;
      const next = () => {
        const mw = this.middlewares[i++];
        if (mw) mw(req, res, next);
      };
      next();
    }).listen(port);
  }
}
```
Chạy được `app.use(logger)`, `app.use(auth)`, `app.use(handler)` theo chain.

🛠️ Viết 3 middleware thực tế:
1. `requestLogger` — log `[METHOD] /path` với timestamp
2. `requestId` — gán `req.id = uuid()` để trace request
3. `responseTime` — gán header `X-Response-Time: 123ms`

---

### Routing & Error Handling

- [ ] Cấu trúc route module hóa + xử lý lỗi tập trung

**Học:**
1. 📖 [Express — Routing](https://expressjs.com/en/guide/routing.html)
2. 📖 [Express — Error handling](https://expressjs.com/en/guide/error-handling.html)

**Exercises:**

🛠️ Tạo cấu trúc route module hóa:
```
src/
  routes/
    index.ts        ← mount tất cả routes
    user.routes.ts  ← /users
    post.routes.ts  ← /posts
  app.ts
```

🛠️ Viết Global Error Handler bắt tất cả lỗi:
```ts
// Error handler phải có 4 params
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  // Log lỗi
  // Trả về JSON error response chuẩn
  // Không expose stack trace khi production
});

// Test với:
app.get('/crash', () => { throw new Error('Oops!'); });
```

---

### Request Validation với Zod

- [ ] Validate input từ user — không tin tưởng bất cứ thứ gì từ bên ngoài

**Học:**
1. 📖 [Zod Documentation — Basic Usage](https://zod.dev/?id=basic-usage)
2. 📖 [Zod — Parsing và Error handling](https://zod.dev/?id=parse)

**Exercises:**

🛠️ Viết validation middleware dùng Zod:
```ts
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().min(18).max(120),
  role: z.enum(['user', 'admin']).default('user'),
});

// Middleware factory:
function validate(schema: z.ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      // Trả về lỗi validation dạng { errors: [...] }
    }
    req.body = result.data; // Parsed và typed
    next();
  };
}
```

---

## Bảo mật BE

---

### Password Hashing: bcrypt / argon2

- [ ] **Không bao giờ** lưu password plain text hoặc dùng MD5/SHA-1

**Học:**
1. 📖 [Why you should use bcrypt to hash passwords](https://auth0.com/blog/hashing-in-action-understanding-bcrypt/)
2. 📖 [bcrypt npm docs](https://www.npmjs.com/package/bcrypt)

**Exercises:**

🛠️ So sánh tốc độ hash:
```js
const bcrypt = require('bcrypt');

// Test với cost factor khác nhau
for (const rounds of [8, 10, 12, 14]) {
  console.time(`bcrypt rounds=${rounds}`);
  await bcrypt.hash('password123', rounds);
  console.timeEnd(`bcrypt rounds=${rounds}`);
}
// Tại sao cost factor lại quan trọng?
```

🛠️ Implement `hashPassword(plain: string)` và `verifyPassword(plain, hash)` — không để lộ timing attack (dùng `bcrypt.compare`, không phải `===`).

---

### JWT từ đầu

- [ ] Implement để hiểu — sau này dùng library mới biết nó làm gì

**Học:**
1. 📖 [JWT.io — JWT Debugger](https://jwt.io/) — decode và hiểu cấu trúc
2. 📖 [jsonwebtoken npm package](https://www.npmjs.com/package/jsonwebtoken)
3. 📖 [Access Token vs Refresh Token — best practices](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)

**Exercises:**

🛠️ Implement Auth flow hoàn chỉnh — không dùng library ngoài `jsonwebtoken` và `bcrypt`:
```
POST /auth/register  → hash password → lưu DB → trả về user
POST /auth/login     → verify password → tạo accessToken (15m) + refreshToken (7d) → lưu refreshToken
POST /auth/refresh   → verify refreshToken → tạo accessToken mới
DELETE /auth/logout  → xóa refreshToken khỏi DB
```

🛠️ Viết `authenticate` middleware kiểm tra Bearer token:
```ts
async function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    // Phân biệt TokenExpiredError vs JsonWebTokenError
  }
}
```

---

### OAuth2 / OpenID Connect

- [ ] Hiểu flow để implement "Login với Google" đúng cách

**Học:**
1. 📖 [OAuth 2.0 explained — Auth0](https://auth0.com/intro-to-iam/what-is-oauth-2)
2. 📖 [OAuth 2.0 Authorization Code Flow with PKCE](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)
3. 📖 [Passport.js — Google OAuth strategy](https://www.passportjs.org/packages/passport-google-oauth20/)

**Exercises:**

🛠️ Vẽ sơ đồ Authorization Code Flow với PKCE:
```
User → App → Google (với code_challenge)
Google → App (với code)
App → Google (code + code_verifier → access_token)
App → Google API (access_token → user info)
```

🛠️ Implement Google OAuth với Passport.js — flow hoàn chỉnh đến khi lấy được email user.

---

### CORS, Helmet, Rate Limiting

- [ ] Security headers — bật ngay từ đầu, đừng thêm sau

**Học:**
1. 📖 [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
2. 📖 [Helmet.js — What each header does](https://helmetjs.github.io/)

**Exercises:**

🛠️ Setup security headers và test bằng securityheaders.com:
```ts
import helmet from 'helmet';
import cors from 'cors';
import rateLimit from 'express-rate-limit';

app.use(helmet());
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') ?? [],
  credentials: true,
}));
app.use('/api', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100,
  message: { error: 'Too many requests' },
}));
```

Deploy app lên môi trường có public URL → paste vào [securityheaders.com](https://securityheaders.com) → xem điểm.

---

### Environment & Secrets Management

- [ ] Commit secret vào git = incident security — không bao giờ

**Học:**
1. 📖 [12factor.net — Config](https://12factor.net/config) — đọc chapter này
2. 📖 [dotenv npm docs](https://www.npmjs.com/package/dotenv)

**Exercises:**

🛠️ Setup config module với validation:
```ts
import { z } from 'zod';
import 'dotenv/config';

const envSchema = z.object({
  PORT: z.string().transform(Number).default('3000'),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']),
});

export const env = envSchema.parse(process.env);
// App crash ngay lúc start nếu thiếu biến — không lỗi âm thầm
```

🛠️ Cài `git-secrets` hoặc dùng pre-commit hook để prevent commit `.env`:
```bash
# .gitignore
.env
.env.local
.env.production
```

---

### Practical: Email & Webhook

- [ ] 2 tính năng gần như mọi app thực tế đều cần

**Học:**
1. 📖 [Nodemailer — Getting Started](https://nodemailer.com/about/)
2. 📖 [Resend — Node.js Quickstart](https://resend.com/docs/send-with-nodejs) — dễ hơn Nodemailer
3. 📖 [Webhook best practices — Stripe Docs](https://docs.stripe.com/webhooks/best-practices)

**Exercises:**

🛠️ Gửi email thực tế với Resend (free tier 3000 emails/tháng):
```ts
import { Resend } from 'resend';
const resend = new Resend(process.env.RESEND_API_KEY);

await resend.emails.send({
  from: 'noreply@yourdomain.com',
  to: 'test@example.com',
  subject: 'Hello from Node.js!',
  html: '<p>This is a test email</p>',
});
```

🛠️ Viết webhook endpoint nhận GitHub webhook (push event):
```ts
// Verify signature — quan trọng để tránh fake webhook
function verifyGithubSignature(payload: string, signature: string): boolean {
  const hmac = crypto.createHmac('sha256', process.env.WEBHOOK_SECRET!);
  const digest = 'sha256=' + hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(Buffer.from(digest), Buffer.from(signature));
}
```

---

## Project Phase 1: Blog API

**Stack**: Node.js + Express + TypeScript + Zod + JWT

Endpoints hoàn chỉnh:

```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
DELETE /auth/logout

GET    /posts              (public — pagination + filter by tag)
GET    /posts/:slug        (public)
POST   /posts              (auth required)
PUT    /posts/:id          (auth + phải là owner)
DELETE /posts/:id          (auth + RBAC: owner hoặc admin)
POST   /posts/:id/publish  (admin only)

POST   /posts/:id/upload-cover  (multipart/form-data)
```

**Yêu cầu kỹ thuật:**
- TypeScript, không có `any`
- Zod validation cho mọi input
- JWT access token (15 phút) + refresh token (7 ngày) lưu trong DB
- Structured logging với Pino (JSON format)
- Global error handler trả về consistent error format
- Rate limiting trên auth endpoints
- CORS + Helmet configured
- `.env` validated khi start

**Test thủ công**: Dùng file `.http` với VS Code REST Client extension (không cần Postman):
```http
### Login
POST http://localhost:3000/auth/login
Content-Type: application/json

{ "email": "test@example.com", "password": "password123" }

### Get posts (auth)
GET http://localhost:3000/posts
Authorization: Bearer {{token}}
```
