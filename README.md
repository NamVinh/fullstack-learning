# Fullstack Learning Roadmap

> Dựa theo [roadmap.sh](https://roadmap.sh) — từ nền tảng đến senior, rõ từng cấp độ.
> Người học: Frontend Engineer 5 năm (React + TypeScript). Mục tiêu: Fullstack thực sự.

---

## Thứ tự học đúng

```
00-foundations  →  01-backend  →  02-database  →  03-frontend  →  04-fullstack  →  05-devops  →  06-senior-advanced
   (mọi path)      (yếu nhất       (song song        (lấp lỗ         (kết nối        (deploy &       (scale &
                   của FE dev)     với backend)       hổng FE)        tất cả)         infra)          design)
```

**Tại sao thứ tự này?**
- FE developer đã có Frontend base → học Backend trước (điểm yếu nhất)
- Database học song song/ngay sau Backend vì chúng gắn chặt nhau
- Frontend nâng cao điền vào gaps (React internals, Vue 3, Performance)
- Fullstack Integration = kết nối FE + BE + DB lại
- DevOps = biết code chạy ở đâu, như thế nào
- Senior = System Design, Architecture, Scale

---

## Cấp độ

| Level | Mô tả | Tương đương |
|-------|-------|------------|
| **Junior** | Biết dùng, làm theo hướng dẫn, cần giám sát | 0–2 năm |
| **Mid** | Hiểu tại sao, tự quyết định, giải quyết vấn đề | 2–5 năm |
| **Senior** | Thiết kế hệ thống, trade-offs, mentor người khác | 5+ năm |

---

## Tiến độ tổng quan

| Phase | Domain | Junior | Mid | Senior | Trạng thái |
|-------|--------|--------|-----|--------|------------|
| 00 | Foundations | 0/8 | 0/5 | — | ⬜ Chưa bắt đầu |
| 01 | Backend | 0/10 | 0/9 | 0/7 | ⬜ Chưa bắt đầu |
| 02 | Database | 0/8 | 0/7 | 0/5 | ⬜ Chưa bắt đầu |
| 03 | Frontend | 0/6 | 0/9 | 0/7 | ⬜ Chưa bắt đầu |
| 04 | Fullstack | 0/7 | 0/8 | — | ⬜ Chưa bắt đầu |
| 05 | DevOps | 0/8 | 0/8 | 0/6 | ⬜ Chưa bắt đầu |
| 06 | Senior/Advanced | — | 0/5 | 0/10 | ⬜ Chưa bắt đầu |

---

## Phase 00 — Foundations (Universal)

> Nền tảng chung cho **tất cả** các path: Backend, Frontend, DevOps. Học một lần, dùng mãi.
> Thời gian: 2–3 tuần

### Junior — Must Know

- [ ] **Internet & Web**: HTTP/HTTPS request-response cycle, DNS, domain, hosting
- [ ] **HTTP sâu hơn**: Methods (GET/POST/PUT/PATCH/DELETE), Status codes, Headers
- [ ] **Git cơ bản**: init, clone, add, commit, push, pull, branch, merge
- [ ] **Terminal/CLI**: Navigation, file management, pipes, redirects
- [ ] **SSH**: Kết nối server từ xa, key-based auth
- [ ] **Package managers**: npm/yarn/pnpm — cách hoạt động, lock file
- [ ] **Text editor**: Vim cơ bản (đủ để edit file trên server)
- [ ] **Encoding**: UTF-8, Base64 — tại sao tồn tại

### Mid — Should Know

- [ ] **HTTP nâng cao**: Caching (Cache-Control, ETag), Cookies, CORS, CSP
- [ ] **Git nâng cao**: Rebase, cherry-pick, squash, bisect, stash
- [ ] **Git workflow**: Git Flow vs Trunk-based, branch strategy, conventional commits
- [ ] **TCP/IP & OSI Model**: 7 layers, tại sao cần biết khi debug network
- [ ] **TLS/SSL**: Handshake hoạt động như thế nào, certificate, Let's Encrypt

**Project 00**: Viết document mô tả một HTTP request từ lúc gõ URL đến khi browser render xong. Commit với conventional commits.

---

## Phase 01 — Backend Developer

> Node.js vì bạn đã biết JS/TS. Không cần học ngôn ngữ mới — tập trung vào tư duy BE.
> Thời gian: 6–8 tuần | Folder: [01-backend/](01-backend/)

### Junior — REST APIs & Auth

- [ ] **Node.js runtime**: Khác browser JS ở đâu — `process`, `fs`, `path`, `http` module
- [ ] **Event Loop** của Node.js: single-thread nhưng non-blocking I/O hoạt động thế nào
- [ ] **Express.js**: Middleware pattern (trái tim của Express), routing, error handling
- [ ] **REST API**: Xây dựng CRUD hoàn chỉnh với proper HTTP methods + status codes
- [ ] **Input validation**: Zod hoặc Joi — validate trước khi xử lý
- [ ] **Authentication**: JWT từ đầu — sign, verify, refresh token (không dùng magic library)
- [ ] **Password security**: bcrypt/argon2 — **KHÔNG dùng MD5/SHA1 cho password**
- [ ] **Environment config**: `.env`, `process.env`, 12-Factor App — không commit secrets
- [ ] **Error handling**: Global error handler, structured error responses (RFC 7807)
- [ ] **Logging cơ bản**: Winston/Pino, log levels (debug/info/warn/error), JSON logs

**Project 01-Junior**: REST API Blog — CRUD posts, auth (register/login/logout/refresh), input validation, proper errors

### Mid — Security, Architecture, Testing

- [ ] **Web Security OWASP**: SQL Injection, XSS, CSRF — từ góc độ BE
- [ ] **CORS**: Tại sao có, cách cấu hình đúng, preflight request
- [ ] **Security headers**: Helmet.js — Content-Security-Policy, X-Frame-Options, HSTS
- [ ] **Rate limiting**: express-rate-limit, sliding window vs token bucket
- [ ] **OAuth2 / OpenID Connect**: Authorization Code + PKCE flow — hiểu thực sự, không chỉ dùng được
- [ ] **Role-based access control (RBAC)**: Thiết kế permission system
- [ ] **File upload**: Multipart/form-data, Multer, validate file type/size
- [ ] **Streams & Buffer**: Xử lý file lớn, pipe, không load toàn bộ vào memory
- [ ] **Unit Testing**: Vitest/Jest — test pure functions, mock dependencies

**Project 01-Mid**: Nâng cấp Blog API — thêm OAuth2 (Google login), RBAC (admin/user/moderator), rate limiting, unit tests coverage > 70%

### Senior — Performance, Patterns, Scalability

- [ ] **Clean Architecture**: Tách business logic khỏi framework, Dependency Injection
- [ ] **Repository Pattern**: Tách data access layer khỏi business logic
- [ ] **NestJS**: Module system, Guards, Interceptors, Pipes — Enterprise Node.js
- [ ] **gRPC**: Protocol Buffers, khi nào dùng thay REST
- [ ] **Message Queues**: BullMQ/RabbitMQ — tại sao cần, async job processing
- [ ] **WebSockets**: Socket.io — real-time, khi nào dùng SSE thay WebSocket
- [ ] **API Design nâng cao**: Versioning, cursor pagination, GraphQL trade-offs
- [ ] **Integration Testing**: Supertest, test với real database (không mock)

---

## Phase 02 — Database

> Điểm yếu lớn nhất của FE dev lên Fullstack. Học cẩn thận.
> Thời gian: 5–6 tuần | Folder: [02-database/](02-database/)

### Junior — SQL Fundamentals

- [ ] **Relational model**: Table, row, column, primary key, foreign key — tại sao cần
- [ ] **SQL CRUD**: `SELECT`, `INSERT`, `UPDATE`, `DELETE` thuần thục
- [ ] **JOINs**: INNER, LEFT, RIGHT, FULL OUTER — biết dùng đúng loại khi nào
- [ ] **Filtering & Sorting**: `WHERE`, `ORDER BY`, `GROUP BY`, `HAVING`
- [ ] **PostgreSQL setup**: Cài local, tạo user/database, connect từ Node.js
- [ ] **Prisma ORM**: Schema definition, migrations, basic queries
- [ ] **Prisma Relations**: one-to-one, one-to-many, many-to-many
- [ ] **Connection pooling**: pg-pool, tại sao không tạo connection mỗi request

**Project 02-Junior**: Thiết kế database schema cho e-commerce (users, products, orders, reviews) — viết đủ SQL queries, implement với Prisma

### Mid — Performance, NoSQL, Caching

- [ ] **Index**: B-tree index, khi nào có lợi, khi nào không, composite index
- [ ] **EXPLAIN ANALYZE**: Đọc query execution plan, tìm bottleneck
- [ ] **N+1 query problem**: Phát hiện và fix với `include`/`select` trong Prisma
- [ ] **Transactions (ACID)**: `BEGIN`, `COMMIT`, `ROLLBACK`, isolation levels
- [ ] **Database Normalization**: 1NF, 2NF, 3NF — khi nào denormalize
- [ ] **MongoDB**: Document model, khi nào dùng thay Postgres, Mongoose basics
- [ ] **Redis**: Key-value store, caching strategies (Cache-aside, Write-through)
- [ ] **Cache invalidation**: TTL, event-based invalidation, cache stampede

**Project 02-Mid**: Nâng cấp e-commerce — thêm Redis caching cho product listings, fix tất cả N+1 queries, thêm proper indexes

### Senior — Scale, Design

- [ ] **Database Sharding**: Horizontal partitioning, consistent hashing
- [ ] **Read Replicas**: Master-slave replication, khi nào đọc từ replica
- [ ] **CAP Theorem**: Consistency vs Availability vs Partition tolerance — trade-offs thực tế
- [ ] **Redis advanced**: Pub/Sub, sorted sets, distributed locks, Redis Cluster
- [ ] **Schema migrations production**: Zero-downtime migrations, expand-contract pattern
- [ ] **Full-text search**: PostgreSQL FTS vs Elasticsearch — khi nào cần Elasticsearch

---

## Phase 03 — Frontend Developer

> Bạn đã có 5 năm React. Phase này **lấp lỗ hổng** và nâng lên Senior FE level.
> Thời gian: 3–4 tuần | Folder: [03-frontend/](03-frontend/)

### Junior — Core (Ôn lại / Chắc chắn đã biết)

- [ ] **HTML semantic**: Accessibility (a11y), ARIA roles — nhiều công ty yêu cầu
- [ ] **CSS layout**: Flexbox, Grid — master edge cases
- [ ] **JavaScript**: Event Loop, Prototype chain, Closure, `this`
- [ ] **TypeScript**: Generics, Mapped Types, Conditional Types, Utility Types
- [ ] **Module system**: CommonJS vs ESM — biết sự khác nhau và tại sao quan trọng
- [ ] **Build tools**: Vite — tại sao nhanh hơn Webpack (native ESM, esbuild)

### Mid — Framework Depth + Vue 3

- [ ] **React internals**: Fiber architecture, reconciliation, diffing — tại sao React nhanh
- [ ] **React 18**: Concurrent rendering, `useTransition`, `useDeferredValue`, Suspense thực sự
- [ ] **React performance**: Khi nào **KHÔNG** nên dùng `memo`/`useMemo`/`useCallback`
- [ ] **Vue 3 Composition API**: `ref`, `reactive`, `computed`, `watch`, `watchEffect`
- [ ] **Vue 3**: Slots (default/named/scoped), provide/inject, Teleport
- [ ] **Pinia**: Vue's state management (tương đương Zustand)
- [ ] **Vue Router 4**: Dynamic routes, navigation guards, lazy loading
- [ ] **Testing**: Vitest + Testing Library, Playwright E2E

**Project 03-Mid**: Build cùng 1 app (Kanban board nhỏ) bằng cả React lẫn Vue — so sánh tư duy của 2 framework

### Senior — Performance, SSR, Architecture

- [ ] **React 19**: `use()` hook, Server Actions, React Compiler
- [ ] **Nuxt.js 3**: SSR, SSG, file-based routing, server routes, auto-imports
- [ ] **Core Web Vitals**: LCP, CLS, INP — đo và tối ưu thực tế với Lighthouse CI
- [ ] **Bundle optimization**: Tree shaking, code splitting, lazy loading, bundle analysis
- [ ] **CSS hiện đại**: Container Queries, `@layer`, CSS custom properties (design tokens)
- [ ] **PWA**: Service workers, offline support, Web App Manifest
- [ ] **Micro-frontends**: Module Federation — awareness level

---

## Phase 04 — Fullstack Integration

> Đây là phần quan trọng nhất. FE và BE không phải 2 thứ rời nhau — phải hiểu chúng nói chuyện với nhau.
> Thời gian: 4–5 tuần | Folder: [04-fullstack/](04-fullstack/)

### Junior — Connect FE + BE

- [ ] **Next.js App Router**: RSC vs Client Components — tư duy mới, hiểu kỹ
- [ ] **Next.js**: Server Actions, API Routes, SSR/SSG/ISR/PPR
- [ ] **TanStack Query**: Caching, stale time, invalidation, optimistic updates
- [ ] **Auth fullstack**: Token storage (localStorage vs httpOnly cookie), CSRF protection
- [ ] **tRPC**: Type-safe API không cần code gen — dùng với TypeScript monorepo
- [ ] **API documentation**: Swagger/OpenAPI spec
- [ ] **MSW (Mock Service Worker)**: Phát triển FE độc lập với BE

### Mid — Real-time, Files, Monorepo

- [ ] **WebSockets với Socket.io**: Real-time notifications/chat
- [ ] **Server-Sent Events**: Khi nào dùng SSE thay WebSocket
- [ ] **File upload pattern**: Presigned URL (FE → S3 trực tiếp, không qua server)
- [ ] **S3 / Cloudflare R2**: Lưu và serve file, CDN integration
- [ ] **Monorepo với Turborepo**: FE + BE trong cùng repo, shared types, shared Zod schemas
- [ ] **Refresh token rotation**: Implement đúng cách, handle race conditions
- [ ] **NextAuth / Auth.js**: Library phổ biến cho Next.js, multiple providers
- [ ] **Storybook**: Document và test UI components

**Project 04**: Fullstack app hoàn chỉnh — Project management tool (Trello-like)
- Next.js + tRPC hoặc REST
- Real-time (Socket.io)
- Auth đầy đủ (JWT + refresh rotation hoặc NextAuth)
- File upload (S3/R2)
- Monorepo với shared types
- Deploy: Vercel + Supabase/Neon

---

## Phase 05 — DevOps

> Fullstack thực sự cần biết code chạy ở đâu. Không cần thành DevOps engineer — cần biết đủ để tự deploy và không phá production.
> Thời gian: 4–5 tuần | Folder: [05-devops/](05-devops/)

### Junior — Linux, Docker, Basic CI/CD

- [ ] **Linux CLI**: Navigation, permissions (chmod/chown), process management (ps/kill/top)
- [ ] **Bash scripting**: Variables, conditionals, loops, functions — đủ để viết deploy script
- [ ] **systemd**: Quản lý service, start/stop/restart/enable, journalctl logs
- [ ] **Nginx**: Reverse proxy cho Node.js, SSL termination + Let's Encrypt (Certbot)
- [ ] **Docker**: Container vs VM, Dockerfile từ đầu cho Node.js app, image layers
- [ ] **Docker Compose**: Multi-service (app + PostgreSQL + Redis + Nginx)
- [ ] **GitHub Actions**: Pipeline đơn giản — lint → test → build → deploy
- [ ] **Secrets management**: GitHub Secrets, không commit `.env` vào git

**Project 05-Junior**: Containerize project Phase 04, viết GitHub Actions pipeline auto-deploy lên VPS

### Mid — Cloud, Monitoring, Security

- [ ] **Docker advanced**: Multi-stage build, image optimization, Docker networking, volumes
- [ ] **AWS fundamentals**: EC2, VPC, Route53, S3, RDS, ECR, IAM — hiểu từng service làm gì
- [ ] **AWS ECS hoặc Cloud Run**: Deploy container lên managed service
- [ ] **Infrastructure as Code**: Terraform cơ bản — không click-ops
- [ ] **Ansible**: Configuration management, không SSH thủ công vào từng server
- [ ] **Sentry**: Error tracking cho cả FE + BE
- [ ] **Structured logging**: Winston/Pino với correlation ID, không dùng `console.log` ở production
- [ ] **Monitoring**: Prometheus + Grafana cơ bản, hoặc Datadog free tier

**Project 05-Mid**: Infrastructure-as-code cho toàn bộ stack (Terraform + Ansible), monitoring với alerts

### Senior — Kubernetes, Scale, SRE

- [ ] **Kubernetes**: Pod, Deployment, Service, Ingress, ConfigMap, Secret
- [ ] **Kubernetes**: HPA (autoscaling), rolling updates, health checks, resource limits
- [ ] **Service Mesh**: Istio/Linkerd — awareness level
- [ ] **HashiCorp Vault**: Secret management đúng cách
- [ ] **Log aggregation**: ELK Stack hoặc Grafana Loki
- [ ] **SRE practices**: SLO, SLI, SLA — error budget, incident management

---

## Phase 06 — Senior / Advanced

> Học song song với công việc, không cần xong hết trước khi đi làm.
> Folder: [06-senior-advanced/](06-senior-advanced/)

### System Design (Mid → Senior)

- [ ] **Scalability**: Horizontal vs vertical scaling, stateless services
- [ ] **Load balancing**: Round-robin, least connections, consistent hashing
- [ ] **Caching tổng thể**: CDN → Application cache → Database cache — biết đặt cache ở đâu
- [ ] **Message queues nâng cao**: Kafka — event streaming, consumer groups
- [ ] **API Gateway**: Rate limiting, auth, routing, transformation
- [ ] **Monolith vs Microservices**: Trade-offs thực tế, khi nào tách, khi nào gộp

### Architecture Patterns (Senior)

- [ ] **Clean Architecture / Hexagonal**: Tách business logic khỏi framework — dependency rule
- [ ] **Domain-Driven Design (DDD)**: Entity, Value Object, Aggregate, Repository, Domain Service
- [ ] **Event-driven architecture**: Event sourcing, eventual consistency
- [ ] **CQRS**: Command Query Responsibility Segregation — khi nào cần
- [ ] **Twelve-Factor App**: Methodology cho modern cloud-native applications

### Performance Deep Dive (Senior)

- [ ] **Node.js profiling**: clinic.js, flame graphs, memory leaks detection
- [ ] **Database tuning**: Connection pool sizing, slow query log, vacuum/analyze
- [ ] **CDN & Edge**: Cloudflare Workers, Vercel Edge — computation gần user nhất
- [ ] **Compression**: gzip/brotli, khi nào compress, khi nào không

---

## Tài nguyên học (chọn lọc theo roadmap.sh)

### Foundations
- [roadmap.sh/full-stack](https://roadmap.sh/full-stack) — Roadmap chính
- [MDN Web Docs](https://developer.mozilla.org) — HTTP, HTML, CSS, JS reference
- [The Odin Project](https://www.theodinproject.com) — Free, hands-on

### Backend
- **Node.js Design Patterns** (Mario Casciaro) — sách hay nhất về Node.js
- [expressjs.com/en/guide](https://expressjs.com/en/guide/) — Official guide
- [docs.nestjs.com](https://docs.nestjs.com) — Enterprise Node.js

### Database
- [Use The Index, Luke](https://use-the-index-luke.com) — SQL + Index, free, rất hay
- [postgresqltutorial.com](https://www.postgresqltutorial.com) — PostgreSQL hands-on
- [prisma.io/docs](https://www.prisma.io/docs) — Prisma ORM

### Frontend
- [vuejs.org/guide](https://vuejs.org/guide) — Vue 3 official docs (rất tốt)
- [nuxt.com/docs](https://nuxt.com/docs) — Nuxt 3
- [react.dev](https://react.dev) — React docs mới với hooks-first approach

### DevOps
- **Docker Deep Dive** (Nigel Poulton) — ngắn, đủ dùng
- [roadmap.sh/devops](https://roadmap.sh/devops) — DevOps roadmap chi tiết
- [DigitalOcean Tutorials](https://www.digitalocean.com/community/tutorials) — Nginx, Linux, Docker

### System Design
- **System Design Interview Vol 1+2** (Alex Xu) — đọc sau khi có nền tảng
- **Designing Data-Intensive Applications** (Martin Kleppmann) — kinh điển

### YouTube
- Fireship — nhanh, rõ, overview tốt
- Theo (t3.gg) — TypeScript fullstack thực tế
- TechWorld with Nana — Docker, Kubernetes, DevOps
- Hussein Nasser — Backend, database, networking sâu

---

## Nguyên tắc học

1. **Hiểu tại sao** trước khi học cách dùng — framework đến rồi đi, nguyên lý ở lại
2. **Build project** sau mỗi phase — không học chay
3. **Không FOMO** — học 1 thứ thật chắc rồi mới sang thứ khác
4. **TypeScript xuyên suốt** — tận dụng ở cả FE lẫn BE
5. **Đọc error thực sự** — stack trace, logs, không chỉ google lỗi
6. **Bảo mật không phải afterthought** — nghĩ đến security từ lúc thiết kế

---

## Cấu trúc repo

```
fullstack-learning/
├── 00-foundations/      # Internet, HTTP, Git, Terminal, SSH
├── 01-backend/          # Node.js, Express, NestJS, APIs, Auth, Security
├── 02-database/         # PostgreSQL, Prisma, MongoDB, Redis
├── 03-frontend/         # React advanced, Vue 3, Performance, SSR
├── 04-fullstack/        # Next.js, tRPC, WebSockets, Monorepo, Auth fullstack
├── 05-devops/           # Linux, Docker, GitHub Actions, AWS, Kubernetes
├── 06-senior-advanced/  # System Design, Architecture, DDD, Performance
└── resources/           # Books, links, cheatsheets
```
