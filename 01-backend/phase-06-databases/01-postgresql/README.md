# Phase 6.1 — PostgreSQL với Node.js

PostgreSQL là relational database mạnh mẽ nhất trong open-source ecosystem — ACID compliant, có JSON support, full-text search, và extensions phong phú. Node.js có nhiều lựa chọn để kết nối: từ raw driver đến query builder đến ORM.

---

## PostgreSQL trong Docker (dev)

```bash
# Chạy PostgreSQL local với Docker
docker run -d \
  --name postgres \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypass \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  postgres:16-alpine

# Kết nối
psql postgresql://myuser:mypass@localhost:5432/mydb
```

---

## node-postgres (pg) — Raw Driver

Dùng khi cần full control, performance tối đa, hoặc query phức tạp.

```js
const { Pool } = require('pg')

// Pool — tái sử dụng connections
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                // max pool size (default 10)
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
})

// Query với parameterized inputs — LUÔN dùng để tránh SQL injection
const { rows } = await pool.query(
  'SELECT * FROM users WHERE id = $1 AND active = $2',
  [id, true]  // $1, $2 — PostgreSQL placeholder syntax
)

// Insert và trả về row vừa tạo
const { rows: [user] } = await pool.query(
  `INSERT INTO users (name, email, password_hash)
   VALUES ($1, $2, $3)
   RETURNING id, name, email, created_at`,
  [name, email, passwordHash]
)

// Transaction
const client = await pool.connect()
try {
  await client.query('BEGIN')

  const { rows: [order] } = await client.query(
    'INSERT INTO orders (user_id, total) VALUES ($1, $2) RETURNING id',
    [userId, total]
  )

  for (const item of items) {
    await client.query(
      'INSERT INTO order_items (order_id, product_id, quantity) VALUES ($1, $2, $3)',
      [order.id, item.productId, item.quantity]
    )
  }

  await client.query(
    'UPDATE products SET stock = stock - $1 WHERE id = $2',
    [item.quantity, item.productId]
  )

  await client.query('COMMIT')
  return order
} catch (err) {
  await client.query('ROLLBACK')
  throw err
} finally {
  client.release()  // LUÔN release connection về pool
}
```

---

## Knex.js — Query Builder

Giữa raw SQL và full ORM — cho phép build queries bằng JS method chaining với SQL-like syntax.

```js
const knex = require('knex')({
  client: 'pg',
  connection: process.env.DATABASE_URL,
  pool: { min: 2, max: 10 }
})

// Select với điều kiện
const users = await knex('users')
  .where({ active: true })
  .where('age', '>=', 18)
  .whereNotNull('email')
  .orderBy('created_at', 'desc')
  .limit(10)
  .offset(0)
  .select('id', 'name', 'email')

// Join
const postsWithAuthors = await knex('posts')
  .join('users', 'posts.user_id', 'users.id')
  .where('posts.published', true)
  .select('posts.*', 'users.name as author_name')

// Insert
const [id] = await knex('users')
  .insert({ name, email, password_hash: hash })
  .returning('id')

// Update
await knex('users')
  .where({ id })
  .update({ name, updated_at: knex.fn.now() })

// Delete
await knex('users').where({ id }).delete()

// Transaction
await knex.transaction(async (trx) => {
  const [order] = await trx('orders').insert({ user_id, total }).returning('*')
  await trx('order_items').insert(items.map(i => ({ order_id: order.id, ...i })))
})

// Migrations với Knex
// knex migrate:make create_users_table
// knex migrate:latest
// knex migrate:rollback
```

---

## Prisma ORM — Khuyên dùng

Prisma là ORM type-safe thế hệ mới với Developer Experience tốt nhất. Schema-first: định nghĩa models trong `schema.prisma`, Prisma generate TypeScript types và migration SQL.

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  password  String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}

enum Role {
  USER
  ADMIN
}

model Post {
  id          Int      @id @default(autoincrement())
  title       String
  content     String?
  published   Boolean  @default(false)
  author      User     @relation(fields: [authorId], references: [id])
  authorId    Int
  tags        Tag[]
  createdAt   DateTime @default(now())

  @@index([authorId, published])
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}
```

```bash
npx prisma migrate dev --name init   # tạo migration + apply
npx prisma generate                  # regenerate client
npx prisma studio                    # GUI để xem data
```

```js
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient()

// Create
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
    password: hash
  }
})

// Read với relations
const users = await prisma.user.findMany({
  where: {
    role: 'USER',
    posts: { some: { published: true } }  // users có ít nhất 1 published post
  },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5
    }
  },
  orderBy: { createdAt: 'desc' },
  skip: 0,
  take: 10
})

// Update
const updatedUser = await prisma.user.update({
  where: { id: userId },
  data: { name: 'Bob', updatedAt: new Date() }
})

// Upsert
const user = await prisma.user.upsert({
  where: { email: 'alice@example.com' },
  update: { name: 'Alice Updated' },
  create: { email: 'alice@example.com', name: 'Alice', password: hash }
})

// Delete
await prisma.user.delete({ where: { id } })

// Transaction
await prisma.$transaction(async (tx) => {
  const order = await tx.order.create({ data: { userId, total } })
  await tx.orderItem.createMany({
    data: items.map(i => ({ orderId: order.id, ...i }))
  })
  await Promise.all(
    items.map(i => tx.product.update({
      where: { id: i.productId },
      data: { stock: { decrement: i.quantity } }
    }))
  )
  return order
})

// Raw SQL khi cần
const result = await prisma.$queryRaw`
  SELECT u.*, COUNT(p.id) as post_count
  FROM users u
  LEFT JOIN posts p ON p.author_id = u.id
  GROUP BY u.id
  HAVING COUNT(p.id) > ${minPosts}
`
```

---

## Drizzle ORM — Type-safe, SQL-like

Drizzle là ORM "type-safe SQL" — gần với SQL hơn Prisma, không có migration tool phức tạp.

```typescript
import { pgTable, serial, text, boolean, timestamp } from 'drizzle-orm/pg-core'
import { drizzle } from 'drizzle-orm/node-postgres'
import { eq, and, gt } from 'drizzle-orm'
import { Pool } from 'pg'

// Schema definition (TypeScript)
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  active: boolean('active').default(true),
  createdAt: timestamp('created_at').defaultNow()
})

const pool = new Pool({ connectionString: process.env.DATABASE_URL })
const db = drizzle(pool, { schema: { users } })

// Queries
const allUsers = await db.select().from(users)

const activeUsers = await db
  .select({ id: users.id, name: users.name })
  .from(users)
  .where(and(eq(users.active, true), gt(users.id, 5)))

await db.insert(users).values({ email, name })
await db.update(users).set({ name }).where(eq(users.id, id))
await db.delete(users).where(eq(users.id, id))
```

---

## Sequelize — Traditional ORM

```js
const { Sequelize, DataTypes } = require('sequelize')
const sequelize = new Sequelize(process.env.DATABASE_URL)

const User = sequelize.define('User', {
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: { isEmail: true }
  },
  name: DataTypes.STRING,
  password: DataTypes.STRING
}, {
  tableName: 'users',
  timestamps: true
})

// Associations
User.hasMany(Post, { foreignKey: 'authorId' })
Post.belongsTo(User, { foreignKey: 'authorId' })

// Queries
await User.findAll({ where: { active: true }, limit: 10 })
await User.findOne({ where: { email } })
await User.create({ email, name, password: hash })
await User.update({ name }, { where: { id } })
await User.destroy({ where: { id } })
```

---

## Bài tập thực hành

1. **Prisma CRUD API**: Blog API với Users, Posts, Comments, Tags — migrations, CRUD endpoints, pagination, filtering theo tags, sorting.

2. **Transaction demo**: Order system — khi đặt hàng: tạo Order, tạo OrderItems, giảm stock Products — tất cả trong một transaction, rollback nếu có lỗi.

3. **Raw SQL vs ORM**: Implement cùng một complex query (users với post count, recent activity) bằng cả raw `pg` và Prisma. So sánh code và performance.

4. **Migration strategy**: Tạo table `users` → add column `role` → rename column `name` → add index. Viết 3 Prisma migrations riêng biệt, test rollback.

---

## Resources

- [Prisma docs](https://www.prisma.io/docs) — ORM documentation
- [node-postgres (pg) docs](https://node-postgres.com) — Raw driver
- [Drizzle ORM docs](https://orm.drizzle.team) — Type-safe SQL ORM
