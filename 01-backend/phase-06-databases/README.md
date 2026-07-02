# Phase 6 — Databases

Database là nơi lưu trữ persistent data của application. Chọn đúng database và sử dụng đúng abstraction layer là kỹ năng quan trọng. Node.js ecosystem có excellent support cho cả SQL và NoSQL databases.

---

## Tổng quan các loại Database

| Loại | Ví dụ | Khi nào dùng |
|------|-------|------------|
| **Relational (SQL)** | PostgreSQL, MySQL, SQLite | Data có structure rõ ràng, cần ACID, joins phức tạp |
| **Document (NoSQL)** | MongoDB, CouchDB | Flexible schema, nested data, content management |
| **Key-Value** | Redis, DynamoDB | Caching, sessions, rate limiting, real-time |
| **Column-family** | Cassandra, ScyllaDB | Time-series, large-scale writes, IoT |
| **Graph** | Neo4j | Social networks, recommendations, relationships |
| **Search** | Elasticsearch | Full-text search, analytics |

---

## Database trong Phase này

| Database | File |
|---------|------|
| [PostgreSQL](./01-postgresql/README.md) | node-postgres, Knex, Prisma, Drizzle, Sequelize |
| [MongoDB + Mongoose](./02-mongodb-mongoose/README.md) | ODM, schemas, aggregation |
| [Redis](./03-redis/README.md) | Caching, Pub/Sub, job queues |

---

## SQL vs NoSQL — Khi nào dùng gì?

### Dùng PostgreSQL khi:
- Data có relationships (users → posts → comments)
- Cần ACID transactions (banking, orders, inventory)
- Schema stable và rõ ràng
- Cần complex queries (aggregations, joins)
- Compliance requirements (HIPAA, PCI-DSS)

### Dùng MongoDB khi:
- Schema thay đổi thường xuyên (MVP, prototyping)
- Data dạng documents/nested objects tự nhiên (CMS, catalogs)
- Không cần multi-collection joins
- Cần horizontal scaling đơn giản

### Dùng Redis khi:
- Caching (giảm DB load)
- Session storage (shared sessions trong cluster)
- Rate limiting (per-IP counters)
- Real-time features (pub/sub, leaderboards)
- Job queues (với BullMQ)

---

## Connection Best Practices

```js
// LUÔN dùng connection pool — đừng tạo connection mới mỗi request
// Pool size gợi ý: numCPUCores * 2 (cho PostgreSQL)

// LUÔN dùng parameterized queries — tránh SQL injection
// LUÔN close connections khi shutdown
// LUÔN handle connection errors và retry

// Environment config
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
REDIS_URL=redis://localhost:6379
MONGODB_URI=mongodb://localhost:27017/mydb
```

---

## Bài tập tổng hợp

1. **CRUD với Prisma + PostgreSQL**: Build blog API — Users, Posts, Comments, Tags. Implement: pagination, filtering, full CRUD, proper error handling.

2. **Same API với Mongoose + MongoDB**: Implement cùng blog API với Mongoose, so sánh code verbosity và DX.

3. **Redis cache layer**: Thêm Redis cache cho GET endpoints — cache hit/miss logging, TTL strategy, cache invalidation khi data thay đổi.

---

## Resources

- [PostgreSQL docs](https://www.postgresql.org/docs/) — Official PostgreSQL reference
- [MongoDB docs](https://www.mongodb.com/docs/) — Official MongoDB docs
- [Redis docs](https://redis.io/docs/) — Redis commands reference
