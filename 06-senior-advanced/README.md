# 06 — Senior / Advanced

> Học song song với công việc. Đây là thứ phân biệt Mid với Senior — không phải biết thêm tool, mà là biết **tại sao** và **khi nào** dùng gì.

---

## System Design (Mid → Senior)

> Khả năng thiết kế hệ thống scale, trade-off aware, communicate rõ với team.

### Scalability

| Topic | Phải hiểu + áp dụng |
|-------|---------------------|
| Horizontal vs Vertical scaling | Scale out (thêm server) vs scale up (nâng cấu hình) — khi nào dùng cái nào |
| Stateless services | Tại sao stateless dễ scale, session storage bên ngoài |
| Load balancing | Round-robin, Least Connections, IP Hash, Consistent Hashing |
| CDN | Serve static assets gần user, edge caching, cache invalidation |
| Auto-scaling | Scale theo CPU/memory/custom metrics |

### Caching Strategy Tổng thể

| Layer | Tool | Khi nào đặt |
|-------|------|-------------|
| DNS | Cloudflare | Domain level |
| CDN | Cloudflare / CloudFront | Static assets, edge |
| Application | Redis | Database query results, session |
| Database | PostgreSQL buffer pool | Query plan cache |

| Topic | Phải hiểu |
|-------|-----------|
| Cache hierarchy | Biết đặt cache ở đúng layer |
| Cache invalidation | Event-based, TTL, write-through — khi nào dùng loại nào |
| Cache stampede | Dog-pile effect, mutex lock, probabilistic expiry |
| Thundering herd | Tại sao xảy ra khi cache miss đồng loạt |

### Message Queues & Async

| Topic | Phải hiểu |
|-------|-----------|
| Tại sao cần message queue | Decoupling, buffering, retry, fan-out |
| BullMQ | Job queues trong Node.js — delay, retry, priority |
| RabbitMQ | Exchange types (direct/topic/fanout), dead letter queue |
| Kafka | Event streaming, consumer groups, partitions, offset |
| Kafka vs RabbitMQ | Kafka = log, RabbitMQ = queue — khác nhau chỗ nào |

### API Gateway & Microservices

| Topic | Phải hiểu |
|-------|-----------|
| API Gateway | Rate limiting, auth, routing, transformation (Kong, AWS API GW) |
| Service Discovery | Consul, Kubernetes DNS — service tìm nhau như thế nào |
| Circuit Breaker | Fail fast, fallback, half-open state |
| Monolith vs Microservices | Trade-offs: complexity, deployment, team ownership |
| Strangler Fig Pattern | Migrate dần từ monolith sang microservices |

### CAP Theorem & Consistency

| Topic | Phải giải thích được |
|-------|---------------------|
| CAP Theorem | Chỉ có thể chọn 2 trong 3: Consistency, Availability, Partition tolerance |
| Eventual consistency | Tại sao distributed systems thường chọn AP |
| Strong consistency | Khi nào cần (payment, inventory) vs khi không cần (social feed) |
| Two-phase commit | Distributed transactions — tại sao khó |

---

## Architecture Patterns (Senior)

### Clean Architecture

| Concept | Phải áp dụng được |
|---------|-------------------|
| Dependency Rule | Dependency chỉ trỏ vào, không trỏ ra — Domain không depend vào Database |
| Layers | Domain → Application → Infrastructure → Presentation |
| Use Cases | Application layer orchestrate domain logic |
| Ports & Adapters | Interface defines contract, adapter implements |
| Testability | Business logic không depend vào Express/Prisma → dễ unit test |

### Domain-Driven Design (DDD)

| Concept | Phải hiểu |
|---------|-----------|
| Bounded Context | Mỗi domain có model riêng, không share freely |
| Entity | Có identity (id), mutable |
| Value Object | Không có identity, immutable (Money, Address) |
| Aggregate | Cluster of objects với 1 root, enforce invariants |
| Repository | Interface để access aggregate (không phải ORM) |
| Domain Events | Something happened in domain — publish/subscribe |
| Domain Service | Business logic không thuộc về Entity nào |

### Event-Driven Architecture

| Topic | Phải hiểu |
|-------|-----------|
| Event Sourcing | Store events, không phải state — reconstruct state từ events |
| CQRS | Tách read model (Query) khỏi write model (Command) |
| Saga Pattern | Distributed transactions qua events — choreography vs orchestration |
| Outbox Pattern | Đảm bảo event được publish khi transaction commit |
| Idempotency | Xử lý duplicate events — `event_id` deduplication |

### 12-Factor App

| Factor | Ý nghĩa |
|--------|---------|
| Codebase | 1 codebase, nhiều deploys |
| Dependencies | Khai báo explicit (package.json), không rely vào system |
| Config | Mọi config qua environment variables |
| Backing services | Database, Redis là "attached resources" |
| Build / Release / Run | 3 giai đoạn tách biệt |
| Processes | Stateless, share-nothing |
| Port binding | Export service qua port |
| Concurrency | Scale bằng processes |
| Disposability | Fast startup, graceful shutdown |
| Dev/Prod parity | Dev environment gần giống production |
| Logs | Treat logs as event streams (stdout) |
| Admin processes | Chạy admin tasks như one-off processes |

---

## Performance Deep Dive (Senior)

### Node.js Profiling

| Topic | Phải làm được |
|-------|---------------|
| clinic.js | `clinic doctor`, `clinic flame` — tìm bottleneck |
| Flame graphs | Đọc được flame graph, tìm hot paths |
| Memory leaks | `node --inspect`, Chrome DevTools heap snapshots |
| Worker threads | CPU-intensive tasks — không block Event Loop |
| `--max-old-space-size` | Điều chỉnh V8 heap size |

### Database Performance

| Topic | Phải biết |
|-------|-----------|
| Slow query log | Enable và analyze slow queries |
| Query optimization | Rewrite queries, use indexes, avoid full table scan |
| Connection pool sizing | Công thức tính pool size theo Amdahl's Law |
| VACUUM / ANALYZE | PostgreSQL maintenance — prevent table bloat |
| Partitioning | Range/list/hash partitioning cho bảng lớn |

### Network & Protocol

| Topic | Phải biết |
|-------|-----------|
| HTTP/2 | Multiplexing, server push, header compression |
| HTTP/3 + QUIC | UDP-based, 0-RTT, không bị head-of-line blocking |
| gRPC | Protocol Buffers, streaming, bi-directional |
| GraphQL N+1 | DataLoader — batch + cache queries |
| Compression | gzip vs brotli — tradeoff CPU vs ratio |

---

## Interview / System Design Topics

> Các câu hỏi system design thường gặp — practice trước khi phỏng vấn Senior

- [ ] Design URL shortener (TinyURL)
- [ ] Design Rate Limiter
- [ ] Design Notification System (push/email/SMS)
- [ ] Design Chat System (WhatsApp-like)
- [ ] Design News Feed (Twitter/Facebook)
- [ ] Design Video Streaming (YouTube)
- [ ] Design Search Autocomplete
- [ ] Design Web Crawler

---

## Resources

- **System Design Interview Vol 1 + 2** (Alex Xu) — đọc sau khi có nền tảng
- **Designing Data-Intensive Applications** (Martin Kleppmann) — kinh điển, bắt buộc
- **Clean Architecture** (Robert C. Martin) — Uncle Bob
- **Domain-Driven Design** (Eric Evans) — "The Blue Book"
- [bytebytego.com](https://bytebytego.com) — Alex Xu's system design newsletter
- Hussein Nasser YouTube — Backend, database, networking sâu
