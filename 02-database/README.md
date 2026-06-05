# 02 — Database

> Điểm yếu lớn nhất của FE dev lên Fullstack. Sai ở đây → cả hệ thống chậm.
> Học SQL trước, NoSQL sau — không được đảo ngược.

---

## Junior

> Có thể thiết kế schema, viết queries đúng, tích hợp được vào Node.js app.

### SQL & PostgreSQL

| Topic | Phải làm được |
|-------|---------------|
| Relational model | Table, row, column, primary key, foreign key — tại sao cần |
| CRUD | `SELECT`, `INSERT`, `UPDATE`, `DELETE` thuần thục |
| WHERE + Operators | `=`, `!=`, `>`, `<`, `LIKE`, `IN`, `BETWEEN`, `IS NULL` |
| JOINs | `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN` — biết dùng đúng loại |
| GROUP BY + Aggregation | `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`, `HAVING` |
| ORDER BY + LIMIT | Sorting, pagination cơ bản với offset |
| Subqueries | `WHERE id IN (SELECT ...)` |
| Constraints | `NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT`, `REFERENCES` |

### Prisma ORM

| Topic | Phải làm được |
|-------|---------------|
| Schema definition | `model`, field types, attributes (`@id`, `@unique`, `@default`) |
| Relations | one-to-one, one-to-many, many-to-many trong schema |
| Migrations | `prisma migrate dev`, `prisma migrate deploy` |
| Basic queries | `findMany`, `findUnique`, `create`, `update`, `delete` |
| Filtering | `where`, `orderBy`, `take`, `skip` |
| Nested reads | `include` — load related records |

### Connection

| Topic | Phải hiểu |
|-------|-----------|
| Connection pooling | Tại sao không tạo connection mới mỗi request, PgBouncer |
| Environment config | `DATABASE_URL` format, không hardcode credentials |

**Checkpoint Junior**: Thiết kế và implement database schema cho e-commerce (users, products, categories, orders, order_items, reviews).

---

## Mid

> Hiểu performance, biết khi nào query chậm và cách fix.

### Performance & Indexing

| Topic | Phải hiểu + áp dụng |
|-------|---------------------|
| Index cơ bản | B-tree index, khi nào có lợi, khi nào gây hại (write overhead) |
| Composite index | `(a, b)` index — column order quan trọng, leading column rule |
| EXPLAIN ANALYZE | Đọc execution plan: `Seq Scan` vs `Index Scan`, cost, rows |
| N+1 query problem | Phát hiện bằng query logs, fix bằng `include` / `select` trong Prisma |
| Partial index | Index với điều kiện `WHERE is_deleted = false` |

### Transactions & ACID

| Topic | Phải hiểu |
|-------|-----------|
| ACID | Atomicity, Consistency, Isolation, Durability — mỗi cái có nghĩa gì |
| Transaction syntax | `BEGIN`, `COMMIT`, `ROLLBACK` |
| Isolation levels | Read Uncommitted, Read Committed, Repeatable Read, Serializable |
| Deadlocks | Khi nào xảy ra, cách phòng tránh |
| Prisma transactions | `$transaction([...])`, interactive transactions |

### Database Design

| Topic | Phải hiểu |
|-------|-----------|
| Normalization | 1NF, 2NF, 3NF — khi nào normalize, khi nào denormalize |
| Foreign key constraints | `ON DELETE CASCADE` vs `RESTRICT` vs `SET NULL` |
| Enum vs Lookup table | Trade-offs, khi nào dùng cái nào |
| Soft delete pattern | `deleted_at` timestamp, partial indexes |

### MongoDB

| Topic | Phải hiểu |
|-------|-----------|
| Document model vs Relational | Embedding vs referencing — khi nào dùng loại nào |
| CRUD với MongoDB | `find`, `insertOne`, `updateOne`, `deleteOne` + operators |
| Schema design | Denormalization đúng cách trong NoSQL |
| Aggregation pipeline | `$match`, `$group`, `$project`, `$lookup`, `$unwind` |
| Mongoose | Schema definition, model, validation, middleware |
| Khi nào chọn MongoDB thay Postgres | Flexible schema, hierarchical data, document-centric |

### Redis

| Topic | Phải làm được |
|-------|---------------|
| Data types | String, Hash, List, Set, Sorted Set — dùng type đúng |
| Caching strategies | Cache-aside (lazy loading), Write-through, Write-behind |
| TTL | `EXPIRE`, `TTL` — quản lý cache expiry |
| Cache invalidation | Event-based, tag-based — bài toán khó nhất |
| Cache stampede | Dog-pile effect và cách phòng (mutex, probabilistic early expiry) |
| Session storage | Lưu session data, distributed sessions |

**Checkpoint Mid**: Thêm Redis caching cho e-commerce (cache product listings, user sessions). Fix tất cả N+1 queries. Thêm proper indexes, đọc được EXPLAIN ANALYZE.

---

## Senior

> Biết scale database, thiết kế cho hệ thống lớn.

### Scale & Replication

| Topic | Phải hiểu |
|-------|-----------|
| Read replicas | Master-slave replication, khi nào đọc từ replica |
| Connection pooling production | PgBouncer — transaction mode vs session mode |
| Horizontal partitioning (Sharding) | Range, hash, list sharding — trade-offs |
| Vertical partitioning | Tách bảng theo access pattern |
| CAP Theorem | Consistency vs Availability vs Partition tolerance |

### Advanced Redis

| Topic | Phải biết |
|-------|-----------|
| Pub/Sub | Publish/subscribe pattern trong Redis |
| Distributed locks | `SET key value NX EX` — implement mutex |
| Sorted Sets | Leaderboards, rate limiting với sliding window |
| Redis Cluster | Horizontal scaling, slot-based sharding |
| Redis Sentinel | High availability, automatic failover |

### Production Migrations

| Topic | Phải hiểu |
|-------|-----------|
| Zero-downtime migrations | Expand-contract pattern (add → migrate → remove) |
| Large table migrations | Không lock table, background migrations |
| Rollback strategy | Tại sao không thể rollback data migrations |
| Database versioning | Migration naming, ordering |

### Search

| Topic | Phải biết khi nào dùng |
|-------|------------------------|
| PostgreSQL Full-Text Search | `tsvector`, `tsquery`, `GIN` index — đủ dùng cho app nhỏ-vừa |
| Elasticsearch | Khi nào cần: complex search, large scale, analytics |

---

## Resources

- [Use The Index, Luke (free)](https://use-the-index-luke.com) — SQL + indexes, hay nhất
- [postgresqltutorial.com](https://www.postgresqltutorial.com)
- [prisma.io/docs](https://www.prisma.io/docs)
- [MongoDB University (free)](https://learn.mongodb.com) — official free courses
- [Redis documentation](https://redis.io/docs)
- **Database Internals** (Alex Petrov) — sâu về storage engines
