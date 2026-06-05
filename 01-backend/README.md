# 01 — Backend Developer

> Node.js vì bạn đã biết JS/TS. Mục tiêu không phải học ngôn ngữ mới mà học **tư duy Backend**.

---

## Junior

> Có thể xây dựng REST API hoàn chỉnh, tự handle auth, deploy được lên server.

### Node.js Runtime

| Topic | Phải hiểu được |
|-------|----------------|
| Node.js khác browser JS ở đâu | Không có DOM/window, có `process`, `fs`, `path`, `http` |
| Event Loop của Node | Single-thread nhưng non-blocking I/O — tại sao không bị block |
| `fs` module | Đọc/ghi file, `readFile` vs `createReadStream` |
| `path` module | `join`, `resolve`, `dirname`, `basename` |
| `http` module | Tạo server thuần, hiểu request/response object |
| `process` object | `env`, `argv`, `exit`, `cwd` |

### Express.js

| Topic | Phải làm được |
|-------|---------------|
| Middleware pattern | Hiểu `(req, res, next)` — đây là trái tim của Express |
| Routing | `app.get/post/put/delete`, Router module hóa |
| Global error handler | `(err, req, res, next)` — 4 params, đặt cuối cùng |
| Input validation | Zod: validate body, params, query trước khi xử lý |
| Response format | JSON responses nhất quán, HTTP status codes đúng |

### Authentication

| Topic | Phải hiểu + implement được |
|-------|---------------------------|
| Password hashing | bcrypt — `hash()`, `compare()`, salt rounds |
| JWT từ đầu | `sign()`, `verify()`, payload structure, expiry |
| Access token + Refresh token | Tại sao cần 2 token, flow hoạt động như thế nào |
| Auth middleware | Verify token trước khi vào route protected |

### Configuration & Secrets

| Topic | Phải làm được |
|-------|---------------|
| `.env` + dotenv | Quản lý config theo môi trường |
| Không commit secrets | `.gitignore`, `.env.example` |
| 12-Factor App | Đọc qua [12factor.net](https://12factor.net) — hiểu nguyên lý |

**Checkpoint Junior**: REST API Blog có CRUD, register/login/logout, JWT auth, Zod validation, global error handler.

---

## Mid

> Hiểu security đúng, biết design API tốt, có testing.

### Web Security

| Topic | Phải hiểu + áp dụng được |
|-------|--------------------------|
| OWASP Top 10 | Biết tên và cách phòng: Injection, XSS, Broken Auth, IDOR... |
| SQL Injection | Tại sao xảy ra, parameterized queries phòng như thế nào |
| CORS đúng | Cấu hình whitelist origins, không dùng `*` ở production |
| Security Headers | Helmet.js: CSP, HSTS, X-Frame-Options, X-Content-Type |
| Rate limiting | express-rate-limit, sliding window vs token bucket |
| Input sanitization | Không trust user input, escape output |

### OAuth2 & Advanced Auth

| Topic | Phải hiểu flow |
|-------|----------------|
| OAuth2 Authorization Code | User → App → Auth Server → App — từng bước |
| PKCE | Tại sao cần cho public clients (SPA, mobile) |
| OpenID Connect | OAuth2 + Identity layer, ID token vs Access token |
| RBAC | Role-based access control — thiết kế permission system |
| Session vs JWT | Trade-offs, khi nào dùng cái nào |

### API Design

| Topic | Phải biết |
|-------|-----------|
| RESTful best practices | Resource naming, HTTP methods đúng, idempotency |
| Versioning | `/v1/`, header versioning — trade-offs |
| Pagination | Offset vs cursor — khi nào dùng cursor |
| Filtering & Sorting | Query params pattern chuẩn |
| Error format | RFC 7807 Problem Details — `type`, `title`, `status`, `detail` |
| OpenAPI / Swagger | Document API, generate từ code |

### Testing

| Topic | Phải viết được |
|-------|----------------|
| Unit test | Vitest/Jest — test pure functions, mock dependencies |
| Integration test | Supertest — test API endpoints với real HTTP |
| Test database | SQLite in-memory hoặc test containers |
| Coverage | >70% cho business logic |

**Checkpoint Mid**: Nâng cấp Blog API — thêm OAuth2 Google login, RBAC, rate limiting, integration tests.

---

## Senior

> Thiết kế được hệ thống, không phụ thuộc vào framework cụ thể.

### Architecture

| Topic | Phải áp dụng được |
|-------|-------------------|
| Clean Architecture | Dependency rule: domain ← application ← infrastructure |
| Repository Pattern | Data access tách khỏi business logic |
| Dependency Injection | Không new dependencies trong class, inject từ ngoài |
| Service Layer | Business logic không ở trong route handler |

### NestJS (Enterprise Node.js)

| Topic | Phải hiểu |
|-------|-----------|
| Module system | Mỗi module tự quản lý dependencies |
| Dependency Injection | Constructor injection, providers |
| Guards | Authentication/Authorization tại decorator level |
| Interceptors | Transform response, logging, caching |
| Pipes | Validation + transformation |
| Microservices transport | TCP, RabbitMQ transport |

### Async & Real-time

| Topic | Phải hiểu khi nào dùng |
|-------|------------------------|
| Message Queues | BullMQ — background jobs, retry, scheduling |
| RabbitMQ | Publish/subscribe, routing, dead letter queue |
| WebSockets | Socket.io — real-time bidirectional |
| Server-Sent Events | One-way real-time, khi nào dùng thay WebSocket |
| Long polling | Tại sao người ta bỏ nó |

### gRPC

| Topic | Phải biết |
|-------|-----------|
| Protocol Buffers | Binary serialization vs JSON — tại sao nhanh hơn |
| gRPC vs REST | Khi nào dùng gRPC (microservices), khi nào REST (public API) |
| `@grpc/grpc-js` | Implement server + client trong Node.js |

---

## Resources

- **Node.js Design Patterns** (Mario Casciaro) — sách hay nhất về Node.js
- [nodejs.org/docs](https://nodejs.org/docs/latest/api/) — Official API reference
- [expressjs.com/en/guide](https://expressjs.com/en/guide/routing.html)
- [docs.nestjs.com](https://docs.nestjs.com) — NestJS official
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [12factor.net](https://12factor.net) — 12-Factor App methodology
