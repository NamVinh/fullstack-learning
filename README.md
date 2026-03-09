# Lộ trình Fullstack Developer

> Dành cho Frontend Developer 5 năm kinh nghiệm (React + Vue + TypeScript)
> Mục tiêu: Đi làm tại công ty với vai trò Fullstack thực sự
> Thời gian học: > 3 tiếng/ngày

---

## Theo dõi tiến độ tổng quan

| Giai đoạn | Chủ đề | Tiến độ |
|-----------|--------|---------|
| Phase 0 | Nền tảng | 0 / 16 |
| Phase 1 | Node.js + Express | 0 / 22 |
| Phase 2 | Database | 0 / 20 |
| Phase 3 | Fullstack Integration | 0 / 26 |
| Phase 4 | DevOps cơ bản | 0 / 22 |
| Phase 5 | Nâng cao (ongoing) | 0 / 24 |

> Cập nhật cột "Tiến độ" thủ công sau mỗi lần hoàn thành topic. Hoặc dùng **GitHub Projects** (hướng dẫn ở cuối file).

---

## Triết lý học
- **Hiểu nguyên lý** trước khi học framework — framework đến rồi đi, nguyên lý ở lại mãi
- **Build project thực tế** sau mỗi giai đoạn, không học chay
- **Không FOMO**: Học xong 1 thứ thật chắc rồi mới sang thứ khác
- **TypeScript xuyên suốt**: Tận dụng tối đa ở cả FE lẫn BE
- **Fullstack = hiểu cả hai phía**: Không chỉ biết code FE và code BE riêng rẽ — phải hiểu chúng nói chuyện với nhau như thế nào

---

## Phase 0 — Củng cố nền tảng (2–3 tuần)

> Trước khi học BE, cần chắc những thứ này. Nhiều dev 5 năm vẫn có lỗ hổng ở đây.

### JavaScript/TypeScript nâng cao
- [ ] Event Loop, Call Stack, Microtask vs Macrotask — hiểu thực sự cách JS chạy
- [ ] Prototype chain, `this`, Closure nâng cao
- [ ] TypeScript: Generic nâng cao, Mapped Types, Conditional Types, Utility Types
- [ ] TypeScript: Decorators — cần cho NestJS sau này
- [ ] Module system: CommonJS vs ESM — biết sự khác nhau và tại sao quan trọng

### Networking & Web cơ bản (cực kỳ quan trọng khi học BE)
- [ ] HTTP/HTTPS: Request/Response cycle, Headers, Status codes
- [ ] REST nguyên lý (không chỉ dùng, phải hiểu constraints của REST)
- [ ] Cookie, Session, JWT — khác nhau như thế nào và dùng khi nào
- [ ] DNS, TCP/IP cơ bản — biết một request đi từ browser đến server như thế nào
- [ ] HTTPS/TLS: handshake hoạt động như thế nào, certificate là gì

### Git workflow nâng cao *(thường bị bỏ qua)*
- [ ] Branching strategy: Git Flow vs Trunk-based development
- [ ] Rebase vs Merge — khi nào dùng cái nào
- [ ] Conventional Commits — chuẩn commit message (quan trọng khi làm team)
- [ ] Squash, cherry-pick, bisect
- [ ] Pull Request best practices — code review culture

**Project**: Viết lại 1 utility function phức tạp bằng TypeScript với đầy đủ type, không dùng `any`. Commit với conventional commits.

---

## Phase 1 — Backend với Node.js + Express (6–8 tuần)

> Chọn Node.js vì bạn đã biết JS/TS — không cần học ngôn ngữ mới, tập trung học tư duy BE

### Node.js Core
- [ ] Node.js runtime: Khác gì browser JS? (no DOM, no window, có `process`, `fs`, `path`...)
- [ ] Streams & Buffer — xử lý dữ liệu lớn
- [ ] EventEmitter pattern
- [ ] `fs`, `path`, `http` module — built-in, không cần thư viện
- [ ] `child_process` cơ bản

### Express.js (framework, nhưng cần hiểu nó làm gì)
- [ ] Middleware pattern — đây là trái tim của Express, hiểu kỹ
- [ ] Routing, Router module hóa
- [ ] Error handling middleware (global error handler)
- [ ] Request validation (zod hoặc joi)
- [ ] File upload (multipart/form-data, multer)

### Bảo mật BE cơ bản *(đừng học sau, học ngay từ đầu)*
- [ ] CORS — tại sao có, cách cấu hình đúng
- [ ] Helmet.js — các HTTP security headers
- [ ] Rate limiting (express-rate-limit)
- [ ] Input sanitization / SQL Injection / XSS từ góc độ BE
- [ ] Password hashing: bcrypt / argon2 — **KHÔNG dùng MD5/SHA-1 cho password**
- [ ] Authentication: Implement JWT từ đầu (không dùng magic library)
- [ ] OAuth2 / OpenID Connect — hiểu flow (Authorization Code, PKCE), không chỉ "dùng được Google login"
- [ ] Authorization: Role-based access control (RBAC)

### Environment & Configuration
- [ ] `.env` và `process.env` — quản lý config theo môi trường
- [ ] Secrets management cơ bản — không commit secret vào git
- [ ] 12-Factor App methodology — đọc qua 12factor.net, hiểu nguyên lý

### Practical skills
- [ ] Gửi email (Nodemailer + SMTP hoặc Resend/SendGrid)
- [ ] Tích hợp webhook cơ bản (nhận và xử lý webhook từ bên thứ 3)

**Project**: Xây dựng REST API cho Blog có:
- CRUD hoàn chỉnh
- Auth (register/login/logout với JWT + refresh token)
- OAuth2 login với Google
- Role: admin vs user
- Input validation với Zod
- Proper error handling + structured logging

---

## Phase 2 — Database (5–6 tuần)

> Đây là điểm yếu lớn nhất của hầu hết FE dev lên Fullstack. Đừng bỏ qua.

### SQL & Relational Database (PostgreSQL)
- [ ] Relational model: table, row, column, primary key, foreign key
- [ ] CRUD với SQL thuần: `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- [ ] JOIN: INNER, LEFT, RIGHT, FULL — biết dùng đúng loại
- [ ] Index: tại sao cần, khi nào dùng, trade-off là gì
- [ ] Transaction: ACID là gì, `BEGIN`, `COMMIT`, `ROLLBACK`
- [ ] N+1 query problem — cực kỳ quan trọng
- [ ] Database normalization: 1NF, 2NF, 3NF
- [ ] Connection pooling — tại sao cần, pg-pool hoạt động như thế nào
- [ ] Explain / Explain Analyze — đọc query execution plan

### ORM — Prisma (khuyên dùng với TS)
- [ ] Schema definition với Prisma
- [ ] Migrations
- [ ] Relations (one-to-one, one-to-many, many-to-many)
- [ ] Query optimization với Prisma (include, select, avoid over-fetching)
- [ ] Raw query khi ORM không đủ mạnh

### NoSQL — MongoDB (học sau SQL, đừng học trước)
- [ ] Document model vs Relational model — khi nào dùng loại nào
- [ ] CRUD với MongoDB
- [ ] Schema design trong NoSQL (embedding vs referencing)
- [ ] Mongoose cơ bản
- [ ] Aggregation pipeline cơ bản

### Redis — Caching Layer
- [ ] Redis là gì — key-value store, in-memory
- [ ] Caching strategies: Cache-aside, Write-through, Write-behind
- [ ] Cache invalidation — bài toán khó nhất trong caching
- [ ] TTL (Time to Live) — quản lý cache expiry
- [ ] Redis pub/sub cơ bản
- [ ] Session storage với Redis

**Project**: Nâng cấp project Phase 1 — thêm PostgreSQL với Prisma, viết migration, tối ưu queries, thêm Redis caching cho các endpoint đọc nhiều

---

## Phase 3 — Fullstack Integration (4–5 tuần)

> Đây là phần quan trọng nhất — không chỉ biết code FE và BE riêng rẽ, mà phải hiểu chúng hoạt động cùng nhau

### API Design nâng cao
- [ ] RESTful API best practices: versioning, pagination (cursor vs offset), filtering, sorting
- [ ] GraphQL cơ bản — hiểu nguyên lý, so sánh với REST (không cần master)
- [ ] tRPC — type-safe API không cần code gen, rất hợp với TypeScript monorepo
- [ ] API documentation với Swagger/OpenAPI
- [ ] Error response format chuẩn (RFC 7807 Problem Details)

### Real-time & WebSockets
- [ ] WebSocket protocol — khác HTTP như thế nào
- [ ] Socket.io — implement chat / notification đơn giản
- [ ] Server-Sent Events (SSE) — khi nào dùng SSE thay WebSocket
- [ ] Long polling — biết tại sao người ta bỏ nó đi

### Next.js (Full-stack framework — rất phổ biến ở công ty)
- [ ] App Router vs Pages Router — hiểu cả hai
- [ ] React Server Components vs Client Components — tư duy mới, hiểu kỹ
- [ ] Server Actions — form và mutation không cần API route
- [ ] SSR, SSG, ISR, PPR — khi nào dùng gì
- [ ] API Routes / Route Handlers
- [ ] Middleware trong Next.js
- [ ] Streaming & Suspense với Next.js

### Fullstack Authentication (hiểu cả FE và BE)
- [ ] Token storage: localStorage vs httpOnly cookie — trade-offs bảo mật
- [ ] Refresh token rotation — implement đúng cách
- [ ] NextAuth / Auth.js — library phổ biến cho Next.js
- [ ] Session management phía server
- [ ] CSRF protection — khi dùng cookie-based auth

### File Storage & Media
- [ ] Upload file từ FE (form + presigned URL pattern)
- [ ] Lưu file lên S3 / Cloudflare R2 / Supabase Storage
- [ ] Image optimization (CDN, Next.js Image component hiểu sâu)

### Monorepo & Code Sharing *(quan trọng cho Fullstack thực sự)*
- [ ] Turborepo hoặc Nx — quản lý FE + BE trong cùng repo
- [ ] Share TypeScript types giữa FE và BE (không copy-paste)
- [ ] Share validation schemas (Zod) giữa FE và BE
- [ ] Shared ESLint/Prettier config

### State Management nhìn lại
- [ ] Server state vs Client state — phân biệt rõ
- [ ] TanStack Query — không chỉ fetch data mà hiểu caching, stale time, invalidation
- [ ] Optimistic updates — UX trick quan trọng
- [ ] Zustand cho client state

### Developer Experience
- [ ] API mocking với MSW (Mock Service Worker) — phát triển FE độc lập với BE
- [ ] Storybook — document và test UI components

**Project**: Xây dựng 1 ứng dụng Fullstack hoàn chỉnh với Next.js + tRPC hoặc REST:
- Ví dụ: Project management tool (Trello clone nhỏ)
- Có real-time updates (Socket.io hoặc SSE)
- Auth hoàn chỉnh (JWT + refresh token hoặc NextAuth)
- File upload (S3)
- Monorepo structure với shared types
- Deploy lên Vercel + Supabase/Neon

---

## Phase 4 — DevOps & Infrastructure cơ bản (3–4 tuần)

> Fullstack thực sự cần biết code của mình chạy ở đâu và như thế nào

### Linux & Command Line
- [ ] Navigation, file management, permissions (chmod, chown)
- [ ] `grep`, `awk`, `sed` cơ bản
- [ ] Process management: `ps`, `kill`, `top`, `htop`
- [ ] SSH, SCP, rsync
- [ ] Cron jobs — schedule tasks
- [ ] systemd cơ bản — quản lý service trên Linux

### Nginx (Web Server / Reverse Proxy)
- [ ] Nginx vs Apache — khi nào dùng gì
- [ ] Cấu hình Nginx làm reverse proxy cho Node.js app
- [ ] SSL termination với Nginx + Let's Encrypt (Certbot)
- [ ] Static file serving
- [ ] Load balancing với Nginx
- [ ] Nginx rate limiting

### Docker (cần thiết ở hầu hết công ty hiện nay)
- [ ] Container vs VM — hiểu nguyên lý
- [ ] Dockerfile: viết từ đầu cho Node.js app
- [ ] Docker Compose: chạy multi-service (app + PostgreSQL + Redis + Nginx)
- [ ] Image optimization (layer caching, multi-stage build)
- [ ] Docker networking — container giao tiếp với nhau như thế nào
- [ ] Docker volumes — persistent data

### Kubernetes (awareness level)
- [ ] Kubernetes là gì — tại sao Docker Compose không đủ ở production
- [ ] Pod, Deployment, Service, Ingress — hiểu concepts cơ bản
- [ ] Không cần master — biết đủ để làm việc với team DevOps

### CI/CD
- [ ] GitHub Actions: tự động test và deploy
- [ ] Viết pipeline: lint → test → build → push Docker image → deploy
- [ ] Environment secrets trong CI/CD
- [ ] Branch protection rules

### Monitoring & Logging *(bị bỏ qua nhiều nhất)*
- [ ] Structured logging — Winston hoặc Pino (JSON logs, không dùng `console.log`)
- [ ] Log levels: debug, info, warn, error — dùng đúng
- [ ] Sentry — error tracking (FE + BE)
- [ ] Metrics cơ bản (response time, error rate, throughput)
- [ ] Uptime monitoring (UptimeRobot hoặc Better Uptime — free tier)

### Cloud cơ bản (chọn 1)
- [ ] AWS: EC2, S3, RDS, ECR, ECS, IAM cơ bản
- [ ] **hoặc** GCP: Cloud Run, Cloud SQL, Cloud Storage
- [ ] **hoặc** Fly.io / Railway — dễ hơn để bắt đầu

**Project**: Containerize project Phase 3, viết GitHub Actions pipeline để auto-deploy lên cloud, cấu hình Nginx, setup Sentry

---

## Phase 5 — Nâng cao & Chuyên sâu (ongoing)

> Sau khi có việc làm Fullstack, học song song với công việc

### System Design
- [ ] Scalability: horizontal vs vertical scaling
- [ ] CAP theorem — Consistency, Availability, Partition tolerance
- [ ] Caching deep dive: Redis cluster, cache stampede, thundering herd
- [ ] Message Queue: RabbitMQ / BullMQ / Kafka — tại sao cần, use case
- [ ] Load balancing: round-robin, least connections, consistent hashing
- [ ] Monolith vs Microservices — trade-offs thực tế, đừng micro quá sớm
- [ ] API Gateway pattern

### Architecture Patterns
- [ ] Clean Architecture / Hexagonal Architecture — tách business logic khỏi framework
- [ ] Domain-Driven Design (DDD) cơ bản — Entity, Value Object, Repository, Service
- [ ] Event-driven architecture cơ bản
- [ ] CQRS + Event Sourcing (đọc hiểu, không cần implement ngay)

### NestJS (Enterprise Node.js)
- [ ] NestJS module system — Dependency Injection
- [ ] Guards, Interceptors, Pipes — tương đương Middleware nhưng có cấu trúc hơn
- [ ] NestJS + Prisma + PostgreSQL
- [ ] Microservices với NestJS (TCP, RabbitMQ transport)

### gRPC & Advanced APIs
- [ ] Protocol Buffers — binary serialization
- [ ] gRPC vs REST vs GraphQL — khi nào dùng gì
- [ ] gRPC trong Node.js (@grpc/grpc-js)

### Testing (đừng bỏ qua)
- [ ] Unit test: Vitest / Jest — test pure functions, services
- [ ] Integration test cho API: Supertest
- [ ] E2E test: Playwright
- [ ] Testing philosophy: test behavior, not implementation
- [ ] Test database: SQLite in-memory hoặc test container

### Performance
- [ ] FE: Core Web Vitals, bundle analysis, lazy loading nâng cao
- [ ] BE: Profiling Node.js (clinic.js), query optimization, connection pooling
- [ ] Caching strategy tổng thể (CDN → Application cache → Database cache)
- [ ] Compression (gzip/brotli) — cấu hình đúng

### Edge Computing
- [ ] Cloudflare Workers / Vercel Edge Functions — code chạy gần user nhất
- [ ] Edge middleware — authentication, A/B testing tại edge
- [ ] Durable Objects (Cloudflare) — stateful edge

---

## Tổng thời gian ước tính

| Giai đoạn | Thời gian |
|-----------|-----------|
| Phase 0 — Nền tảng | 2–3 tuần |
| Phase 1 — Node.js + Express | 6–8 tuần |
| Phase 2 — Database | 5–6 tuần |
| Phase 3 — Fullstack Integration | 4–5 tuần |
| Phase 4 — DevOps cơ bản | 3–4 tuần |
| **Tổng** | **~5–6 tháng** |

> Với > 3 tiếng/ngày và nền tảng JS/TS vững, bạn có thể rút ngắn còn **4–5 tháng**

---

## Tài nguyên học (chọn lọc)

### Node.js & Backend
- **Node.js**: Official docs + "Node.js Design Patterns" (Mario Casciaro — sách hay nhất)
- **Express**: expressjs.com/en/guide
- **NestJS**: docs.nestjs.com (rất chi tiết)

### Database
- **SQL**: "Use The Index, Luke" (usetheindexluke.com — free, giải thích SQL và index cực hay)
- **PostgreSQL**: postgresqltutorial.com
- **Prisma**: prisma.io/docs

### System Design
- "System Design Interview" (Alex Xu) — đọc sau khi có nền tảng
- "Designing Data-Intensive Applications" (Martin Kleppmann) — quyển kinh điển

### Fullstack
- **Next.js**: nextjs.org/docs (đủ tốt)
- **YouTube**: Josh tried coding, Theo (t3.gg), Fireship

### DevOps
- **Docker**: "Docker Deep Dive" (Nigel Poulton) — ngắn, đủ dùng
- **Nginx**: nginx.org/en/docs + digitalocean nginx tutorials

### Roadmap tham khảo
- roadmap.sh/full-stack
- roadmap.sh/backend
- roadmap.sh/devops

---

## Công cụ theo dõi tiến độ

### Option 1: GitHub Projects (khuyên dùng — miễn phí, tích hợp với code)
1. Push repo này lên GitHub
2. Vào tab **Projects** → New Project → chọn **Board** (Kanban)
3. Tạo columns: `Backlog | In Progress | Done`
4. Tạo Issues cho từng topic, gán vào Project
5. Khi học xong 1 topic → kéo card sang `Done`

### Option 2: Notion
- Tạo database với Property: Phase, Status (Not started / In progress / Done), Notes
- Filter theo Phase để focus từng giai đoạn

### Option 3: Checkbox trong file này (đơn giản nhất)
- Tick checkbox khi học xong
- GitHub render checkbox tự động nếu dùng `- [ ]`
- Cập nhật bảng tiến độ ở đầu file

---

## Nguyên tắc khi đi làm Fullstack

1. **Đừng làm FE giả vờ BE** — khi nhận task BE, nghĩ từ góc độ data, business logic, không phải UI
2. **Database là trái tim** — query chậm làm hỏng cả hệ thống, luôn chú ý
3. **Security không phải afterthought** — nghĩ đến bảo mật từ lúc thiết kế
4. **Đọc error log** — BE error khác FE error, học cách đọc stack trace phía server
5. **Hiểu cả hai phía của network call** — khi debug, trace từ browser request → DNS → server → database → response
6. **Shared types là superpower của TypeScript fullstack** — tận dụng tối đa, tránh type drift giữa FE và BE
