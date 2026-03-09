# Phase 2 — Database

**Thời gian**: 5–6 tuần | **Mục tiêu**: Thiết kế và tối ưu database như một pro

---

## SQL & PostgreSQL

---

### Relational Model & SQL cơ bản

- [ ] Đọc và viết SQL thuần trước khi dùng ORM

**Học:**
1. 📖 [PostgreSQL Tutorial — Getting Started](https://www.postgresqltutorial.com/postgresql-getting-started/)
2. 📖 [SQLZoo — Interactive SQL Tutorial (free)](https://sqlzoo.net/) — làm hết phần SELECT

**Exercises:**

🛠️ Cài PostgreSQL local (hoặc dùng [neon.tech](https://neon.tech) free tier). Tạo database `blog_db`:
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  published BOOLEAN DEFAULT FALSE,
  author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE post_tags (
  post_id INTEGER REFERENCES posts(id),
  tag_id INTEGER REFERENCES tags(id),
  PRIMARY KEY (post_id, tag_id)
);
```

🛠️ Viết các queries:
```sql
-- 1. Lấy tất cả posts của user có email 'test@example.com'
-- 2. Lấy 10 posts mới nhất kèm tên tác giả
-- 3. Lấy top 5 tags được dùng nhiều nhất
-- 4. Lấy posts chưa có tag nào
-- 5. Đếm số posts của từng user
```

---

### JOINs

- [ ] Đây là điểm yếu phổ biến nhất — không hiểu JOIN là không làm được

**Học:**
1. 📖 [Visual Explanation of SQL JOINs](https://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins) — xem hình trước
2. 📖 [PostgreSQL JOIN — postgresqltutorial.com](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-joins/)

**Exercises:**

🛠️ Viết và giải thích từng loại JOIN trên database `blog_db`:
```sql
-- INNER JOIN: chỉ posts CÓ author
SELECT posts.title, users.name
FROM posts INNER JOIN users ON posts.author_id = users.id;

-- LEFT JOIN: tất cả users, kể cả người chưa có post
SELECT users.name, COUNT(posts.id) as post_count
FROM users LEFT JOIN posts ON posts.author_id = users.id
GROUP BY users.name;

-- Câu hỏi: Nếu thêm WHERE posts.id IS NULL sau LEFT JOIN — lấy được gì?
```

---

### Index

- [ ] Index đúng → query nhanh 1000x. Index sai → lãng phí disk và làm chậm write

**Học:**
1. 📖 [Use The Index, Luke — Chapter 1: Anatomy of an Index](https://use-the-index-luke.com/sql/anatomy) — free online, đọc kỹ
2. 📖 [PostgreSQL Indexes — Official Docs](https://www.postgresql.org/docs/current/indexes.html)

**Exercises:**

🛠️ Test performance với và không có index:
```sql
-- Insert 100,000 rows
INSERT INTO posts (title, author_id, created_at)
SELECT 'Post ' || i, (i % 100) + 1, NOW() - (i || ' minutes')::interval
FROM generate_series(1, 100000) i;

-- Query không có index
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 42;

-- Tạo index
CREATE INDEX idx_posts_author_id ON posts(author_id);

-- Query với index — so sánh execution time
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 42;
```

🛠️ Trả lời: Tại sao index làm chậm INSERT/UPDATE/DELETE? Khi nào thì KHÔNG nên tạo index?

---

### Transactions & ACID

- [ ] Transaction đảm bảo data consistency — cực quan trọng cho business logic

**Học:**
1. 📖 [PostgreSQL Transactions](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-transaction/)
2. 📖 [ACID Properties — explained simply](https://fauna.com/blog/database-transaction)

**Exercises:**

🛠️ Simulate transfer tiền giữa 2 tài khoản — phải dùng transaction:
```sql
-- Tạo bảng
CREATE TABLE accounts (id SERIAL, balance DECIMAL(10,2));
INSERT INTO accounts (balance) VALUES (1000), (500);

-- Transfer 200 từ account 1 → account 2
BEGIN;
  UPDATE accounts SET balance = balance - 200 WHERE id = 1;
  -- Thêm CHECK: nếu balance < 0 thì ROLLBACK
  UPDATE accounts SET balance = balance + 200 WHERE id = 2;
COMMIT;

-- Test: thử làm balance âm → phải ROLLBACK
```

🛠️ Giải thích: ACID là gì? Cho ví dụ thực tế của từng property (Atomicity, Consistency, Isolation, Durability).

---

### N+1 Query Problem

- [ ] Lỗi phổ biến nhất khi dùng ORM — giết chết performance của app

**Học:**
1. 📖 [What is the N+1 Query Problem?](https://planetscale.com/blog/what-is-n-1-query-problem-and-how-to-solve-it)
2. 📖 [Prisma — How to avoid N+1](https://www.prisma.io/docs/orm/prisma-client/queries/query-optimization-performance)

**Exercises:**

🛠️ Reproduce và fix N+1:
```ts
// ❌ N+1 problem
const posts = await Post.findAll(); // 1 query
for (const post of posts) {
  const author = await User.findById(post.authorId); // N queries!
  console.log(post.title, author.name);
}

// ✅ Fix với JOIN/include
const posts = await Post.findAll({
  include: [{ model: User, as: 'author' }]
}); // 1 query hoặc 2 queries tối đa
```

Dùng `console.log` để đếm số queries thực tế trong từng trường hợp.

---

### EXPLAIN ANALYZE

- [ ] Đây là tool mạnh nhất để debug query chậm

**Học:**
1. 📖 [EXPLAIN ANALYZE — PostgreSQL Docs](https://www.postgresql.org/docs/current/sql-explain.html)
2. 📖 [Depesz EXPLAIN visualizer](https://explain.depesz.com/) — paste execution plan vào đây để xem đẹp hơn

**Exercises:**

🛠️ Chạy query chậm và đọc execution plan:
```sql
EXPLAIN ANALYZE
SELECT p.title, u.name, COUNT(pt.tag_id) as tag_count
FROM posts p
JOIN users u ON p.author_id = u.id
LEFT JOIN post_tags pt ON p.id = pt.post_id
GROUP BY p.id, p.title, u.name
ORDER BY tag_count DESC
LIMIT 10;
```

Xác định: "Seq Scan" hay "Index Scan"? Execution time bao nhiêu ms? Tìm chỗ tốn nhiều time nhất.

---

## ORM — Prisma

---

### Prisma Schema & Migrations

- [ ] Prisma = type-safe database access, không cần viết SQL thủ công

**Học:**
1. 📖 [Prisma — Getting Started with PostgreSQL](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)
2. 📖 [Prisma Schema Reference](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference)
3. 📖 [Prisma Migrate](https://www.prisma.io/docs/orm/prisma-migrate)

**Exercises:**

🛠️ Dịch schema SQL ở trên thành Prisma schema:
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  posts     Post[]
  createdAt DateTime @default(now())
}

model Post {
  id        Int       @id @default(autoincrement())
  title     String
  content   String?
  published Boolean   @default(false)
  author    User      @relation(fields: [authorId], references: [id])
  authorId  Int
  tags      PostTag[]
  createdAt DateTime  @default(now())
}
// Tự viết tiếp Tag và PostTag
```

🛠️ Thực hành migration workflow:
```bash
# Tạo migration mới
npx prisma migrate dev --name add_posts_table

# Xem SQL được generate
cat prisma/migrations/*/migration.sql

# Thêm field mới vào Post: viewCount Int @default(0)
# Tạo migration: npx prisma migrate dev --name add_view_count
```

---

### Prisma Queries & Relations

- [ ] Query tối ưu — chỉ lấy data cần thiết

**Học:**
1. 📖 [Prisma — CRUD](https://www.prisma.io/docs/orm/prisma-client/queries/crud)
2. 📖 [Prisma — Select fields](https://www.prisma.io/docs/orm/prisma-client/queries/select-fields)
3. 📖 [Prisma — Relation queries](https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries)

**Exercises:**

🛠️ Viết các queries — chỉ lấy fields cần thiết:
```ts
// 1. Lấy 10 posts mới nhất, kèm author name và số lượng tag (không lấy content)
const posts = await prisma.post.findMany({
  take: 10,
  orderBy: { createdAt: 'desc' },
  select: {
    id: true,
    title: true,
    author: { select: { name: true } },
    _count: { select: { tags: true } },
  },
});

// 2. Tạo post với nhiều tags cùng lúc (nested create)
// 3. Xóa user và cascade xóa tất cả posts của họ
// 4. Cursor-based pagination
```

---

## Redis — Caching

---

### Redis Basics & Caching Strategies

- [ ] Redis không phải chỉ "lưu session" — đây là tool quan trọng

**Học:**
1. 📖 [Redis — Getting Started](https://redis.io/docs/getting-started/)
2. 📖 [Redis University — RU101: Introduction to Redis Data Structures (free)](https://university.redis.com/courses/ru101/)
3. 📖 [Caching strategies — AWS blog](https://aws.amazon.com/caching/best-practices/)

**Exercises:**

🛠️ Cài Redis local (hoặc dùng [Upstash](https://upstash.com/) free tier). Thực hành CLI:
```bash
redis-cli

SET user:1 '{"name":"Nam","email":"nam@example.com"}'
GET user:1
SETEX session:abc123 3600 '{"userId":1}'  # TTL 1 giờ
TTL session:abc123     # Xem còn bao nhiêu giây
KEYS user:*            # Tìm tất cả keys theo pattern
DEL user:1
```

🛠️ Implement Cache-Aside pattern cho endpoint GET /posts:
```ts
async function getPosts() {
  const cacheKey = 'posts:all';

  // 1. Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // 2. Cache miss → query DB
  const posts = await prisma.post.findMany({ take: 20 });

  // 3. Lưu vào cache, TTL 5 phút
  await redis.setex(cacheKey, 300, JSON.stringify(posts));

  return posts;
}

// Câu hỏi: Khi tạo post mới, phải làm gì với cache? (Cache invalidation)
```

---

## Project Phase 2

Nâng cấp Blog API từ Phase 1:
1. Migrate từ in-memory storage sang PostgreSQL với Prisma
2. Viết đầy đủ migrations
3. Thêm Redis caching cho `GET /posts` và `GET /posts/:slug`
4. Implement cache invalidation khi post được tạo/cập nhật/xóa
5. Dùng `EXPLAIN ANALYZE` để xác nhận index được dùng đúng
6. Thêm `X-Cache: HIT/MISS` header để debug

**Bonus**: Implement rate limiting lưu trong Redis (không phải in-memory) → hoạt động đúng khi scale nhiều instance.
