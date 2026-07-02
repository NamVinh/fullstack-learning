# Phase 5 — Web Frameworks

Frameworks là layer trên `http` module cung cấp routing, middleware, validation, và nhiều tiện ích khác. Chọn đúng framework phụ thuộc vào yêu cầu dự án, team size, và performance requirements.

---

## Framework Overview

| Framework | Điểm mạnh | Dùng khi |
|-----------|-----------|---------|
| **[Express](./01-express/README.md)** | Minimal, flexible, ecosystem khổng lồ | Junior — mọi dự án, phải biết |
| **[Fastify](./02-fastify/README.md)** | Nhanh nhất, schema-first, type-safe | Mid — khi cần performance |
| **[NestJS](./03-nestjs/README.md)** | Enterprise, opinionated, TypeScript-first | Senior — large team, complex apps |
| **[Hono](./04-hono/README.md)** | Ultra-lightweight, multi-runtime, edge | Modern — Cloudflare Workers, edge computing |

---

## So sánh Performance (requests/second, approx)

```
Raw http module:    ~70,000 req/s
Fastify:            ~60,000 req/s   (overhead ~14%)
Express:            ~30,000 req/s   (overhead ~57%)
NestJS (Fastify):   ~55,000 req/s
NestJS (Express):   ~28,000 req/s
Hono (Bun):         ~120,000 req/s
```

*Lưu ý: Trong real-world apps, bottleneck thường là database/I/O, không phải framework overhead.*

---

## Khi nào chọn gì

### Express.js
- Học Node.js lần đầu
- Prototype nhanh
- Tìm ecosystem packages phong phú nhất
- Team không quen TypeScript

### Fastify
- Cần throughput cao (nhiều requests/second)
- Muốn built-in validation qua JSON Schema
- Thích type-safe nhưng không muốn NestJS overhead

### NestJS
- Large team cần conventions rõ ràng
- Dự án phức tạp, nhiều modules
- Muốn Angular-like structure
- TypeScript mandatory
- Cần built-in: DI, testing utilities, microservices, GraphQL, WebSockets

### Hono
- Chạy trên Cloudflare Workers / edge
- Cần run trên Bun, Deno, Vercel Edge
- API surface nhỏ, không muốn overhead
- Serverless functions

---

## Điểm chung của mọi framework

```
Request → Middleware chain → Route handler → Response

Middleware:
- Parse body (JSON, multipart)
- Authentication
- Authorization
- Rate limiting
- Logging
- CORS
- Compression
- Error handling
```

---

## Resources

- [expressjs.com](https://expressjs.com) — Express documentation
- [fastify.dev](https://fastify.dev) — Fastify documentation
- [docs.nestjs.com](https://docs.nestjs.com) — NestJS documentation
- [hono.dev](https://hono.dev) — Hono documentation
