# Phase 6.3 — Redis

Redis là in-memory data structure store — cực kỳ nhanh (microsecond latency) vì data nằm trong RAM. Redis không phải replacement cho PostgreSQL hay MongoDB — nó là **layer thêm vào** để giải quyết các vấn đề cụ thể: caching, sessions, rate limiting, pub/sub, và job queues.

---

## Redis trong Docker (dev)

```bash
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7-alpine

# Test connection
redis-cli ping  # → PONG
```

---

## Connect với Node.js

```js
const { createClient } = require('redis')

const client = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
  socket: {
    reconnectStrategy: (retries) => Math.min(retries * 50, 500)  // retry delay
  }
})

client.on('error', (err) => console.error('Redis error:', err))
client.on('connect', () => console.log('Redis connected'))

await client.connect()

// Graceful disconnect
process.on('SIGTERM', async () => {
  await client.quit()
})
```

---

## Data Structures và Commands

### Strings — Key-Value

```js
// Set và Get
await client.set('user:1:name', 'Alice')
const name = await client.get('user:1:name')  // 'Alice'

// Set với TTL (Time-To-Live)
await client.set('session:abc123', JSON.stringify(sessionData), {
  EX: 3600  // expire sau 3600 giây = 1 giờ
})
// Hoặc
await client.setEx('cache:users', 300, JSON.stringify(users))  // 5 minutes

// Increment/Decrement (atomic)
await client.incr('counter')          // 1
await client.incrBy('counter', 5)     // 6
await client.decr('counter')          // 5

// TTL operations
await client.ttl('session:abc123')    // còn bao nhiêu giây
await client.persist('key')           // xóa TTL
await client.expire('key', 600)       // set TTL mới

// Check và Delete
await client.exists('key')            // 1 (exists) hoặc 0
await client.del('key')
await client.del(['key1', 'key2'])    // delete nhiều keys
```

### Hashes — Object Fields

```js
// Lưu object dưới dạng hash (efficient hơn JSON string cho partial updates)
await client.hSet('user:42', {
  name: 'Alice',
  email: 'alice@example.com',
  role: 'admin'
})

// Get một field
const name = await client.hGet('user:42', 'name')

// Get tất cả fields
const user = await client.hGetAll('user:42')  // { name, email, role }

// Update một field (không overwrite cả object)
await client.hSet('user:42', 'role', 'user')

// Delete field
await client.hDel('user:42', 'avatar')
```

### Lists — Queue / Stack

```js
// Dùng làm queue (FIFO)
await client.rPush('queue:emails', JSON.stringify(emailJob))  // enqueue
const job = await client.lPop('queue:emails')                 // dequeue

// Blocking pop — đợi khi queue rỗng (consumer pattern)
const [queueName, item] = await client.blPop('queue:emails', 0)  // 0 = wait forever

// Stack (LIFO)
await client.lPush('stack', 'item1')
const top = await client.lPop('stack')
```

### Sets — Unique Collections

```js
// Thêm và kiểm tra members
await client.sAdd('tags:post:1', 'nodejs', 'backend', 'javascript')
await client.sIsMember('tags:post:1', 'nodejs')  // true/false
const tags = await client.sMembers('tags:post:1')  // Set(['nodejs', 'backend', 'javascript'])

// Set operations
await client.sInterStore('common', 'set:1', 'set:2')  // intersection
await client.sUnionStore('all', 'set:1', 'set:2')     // union
```

### Sorted Sets — Leaderboards, Rate Limiting

```js
// Score-based sorted set
await client.zAdd('leaderboard', [
  { score: 1500, value: 'alice' },
  { score: 2300, value: 'bob' },
  { score: 900, value: 'charlie' }
])

// Get top 10
const top10 = await client.zRangeWithScores('leaderboard', 0, 9, { REV: true })

// Get rank của user
const rank = await client.zRank('leaderboard', 'alice')  // 0-indexed

// Increment score
await client.zIncrBy('leaderboard', 100, 'alice')  // alice += 100
```

---

## Caching Patterns

### Cache-Aside (Lazy Loading)

```js
// Pattern phổ biến nhất
async function getUserById(id) {
  const cacheKey = `user:${id}`

  // 1. Check cache
  const cached = await redis.get(cacheKey)
  if (cached) {
    return JSON.parse(cached)  // Cache hit
  }

  // 2. Cache miss — query DB
  const user = await prisma.user.findUnique({ where: { id } })
  if (!user) return null

  // 3. Store trong cache với TTL
  await redis.setEx(cacheKey, 3600, JSON.stringify(user))

  return user
}

// Cache invalidation khi data thay đổi
async function updateUser(id, data) {
  const user = await prisma.user.update({ where: { id }, data })
  await redis.del(`user:${id}`)  // invalidate cache
  return user
}
```

### Write-Through Cache

```js
// Update DB và cache đồng thời
async function updateUser(id, data) {
  const user = await prisma.user.update({ where: { id }, data })
  await redis.setEx(`user:${id}`, 3600, JSON.stringify(user))  // update cache
  return user
}
```

### Cache Middleware cho Express

```js
function cacheMiddleware(ttl = 300) {
  return async (req, res, next) => {
    const key = `cache:${req.method}:${req.originalUrl}`

    const cached = await redis.get(key)
    if (cached) {
      return res.json(JSON.parse(cached))
    }

    // Override res.json để cache response
    const originalJson = res.json.bind(res)
    res.json = async (data) => {
      await redis.setEx(key, ttl, JSON.stringify(data))
      return originalJson(data)
    }

    next()
  }
}

// Usage
app.get('/api/products', cacheMiddleware(600), getProducts)
```

---

## Rate Limiting với Redis

```js
// Sliding window rate limiter
async function rateLimiter(key, limit, windowSeconds) {
  const now = Date.now()
  const windowStart = now - windowSeconds * 1000

  // Multi-command pipeline (atomic)
  const pipeline = redis.multi()
  pipeline.zRemRangeByScore(key, 0, windowStart)     // remove old requests
  pipeline.zAdd(key, [{ score: now, value: `${now}` }])  // add current request
  pipeline.zCard(key)                                 // count requests in window
  pipeline.expire(key, windowSeconds)                 // set TTL

  const results = await pipeline.exec()
  const requestCount = results[2]  // zCard result

  return {
    allowed: requestCount <= limit,
    count: requestCount,
    remaining: Math.max(0, limit - requestCount),
    resetAt: new Date(now + windowSeconds * 1000)
  }
}

// Express middleware
const rateLimitMiddleware = (limit, windowSeconds) => async (req, res, next) => {
  const key = `rate:${req.ip}:${req.path}`
  const result = await rateLimiter(key, limit, windowSeconds)

  res.setHeader('X-RateLimit-Limit', limit)
  res.setHeader('X-RateLimit-Remaining', result.remaining)
  res.setHeader('X-RateLimit-Reset', result.resetAt.toISOString())

  if (!result.allowed) {
    return res.status(429).json({ error: 'Too many requests' })
  }
  next()
}
```

---

## Pub/Sub

```js
// Subscriber — cần separate connection
const subscriber = client.duplicate()
await subscriber.connect()

await subscriber.subscribe('notifications', (message, channel) => {
  const data = JSON.parse(message)
  console.log(`Received on ${channel}:`, data)
})

// Subscribe nhiều channels
await subscriber.subscribe(['user:events', 'order:events'], handler)

// Pattern subscribe
await subscriber.pSubscribe('user:*', (message, channel) => {
  console.log(`Message on ${channel}:`, message)
})

// Publisher (dùng main client)
await client.publish('notifications', JSON.stringify({
  type: 'new-order',
  userId: 42,
  orderId: 100
}))

// Unsubscribe
await subscriber.unsubscribe('notifications')
await subscriber.quit()
```

---

## Session Storage

```js
const session = require('express-session')
const { RedisStore } = require('connect-redis')

app.use(session({
  store: new RedisStore({ client }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000  // 24 hours
  }
}))
```

---

## Bài tập thực hành

1. **Cache layer**: Thêm Redis cache vào Express API — GET endpoints dùng cache-aside, automatic invalidation khi POST/PUT/DELETE.

2. **Rate limiter**: Implement sliding window rate limiter — different limits cho authenticated vs anonymous users, per-endpoint limits.

3. **Session store**: Implement session-based auth (thay JWT) dùng Redis store — login tạo session, logout delete session, session expiry.

4. **Pub/Sub notifications**: Implement real-time notification system — khi user A comment vào post của user B, publish event, subscriber gửi notification (log ra console).

---

## Resources

- [Redis docs](https://redis.io/docs/) — Official Redis documentation
- [node-redis docs](https://github.com/redis/node-redis) — Node.js Redis client
- [Redis Best Practices](https://redis.io/docs/manual/patterns/) — Common patterns
