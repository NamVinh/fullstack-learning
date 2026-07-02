# Phase 4 — Async Programming

Async programming là trái tim của Node.js. Hiểu cách JavaScript xử lý bất đồng bộ — từ callbacks cũ đến Promises hiện đại đến async/await — giúp bạn viết code hiệu quả, tránh callback hell, và tận dụng được non-blocking I/O của Node.js.

---

## Evolution: Callbacks → Promises → Async/Await

### Callbacks (Legacy — callback hell)

```js
const fs = require('fs')

// Callback hell — hard to read, hard to maintain
fs.readFile('./users.json', 'utf8', (err, usersData) => {
  if (err) return handleError(err)

  const users = JSON.parse(usersData)
  db.query('SELECT * FROM roles', (err, roles) => {
    if (err) return handleError(err)

    users.forEach(user => {
      db.query(`UPDATE users SET role = ? WHERE id = ?`, [roles[0].id, user.id], (err) => {
        if (err) return handleError(err)
        // deeply nested, error handling repeated everywhere
      })
    })
  })
})
```

### Promises

```js
// Promises — flat chaining
readFile('./users.json', 'utf8')
  .then(data => JSON.parse(data))
  .then(users => db.query('SELECT * FROM roles').then(roles => ({ users, roles })))
  .then(({ users, roles }) => {
    return Promise.all(
      users.map(user => db.query('UPDATE...', [roles[0].id, user.id]))
    )
  })
  .then(() => console.log('Done'))
  .catch(err => handleError(err))
  .finally(() => db.close())
```

### Async/Await (Prefer)

```js
// Async/await — đọc như synchronous code
async function updateUserRoles() {
  try {
    const data = await readFile('./users.json', 'utf8')
    const users = JSON.parse(data)
    const roles = await db.query('SELECT * FROM roles')

    await Promise.all(
      users.map(user => db.query('UPDATE...', [roles[0].id, user.id]))
    )

    console.log('Done')
  } catch (err) {
    handleError(err)
  } finally {
    await db.close()
  }
}
```

---

## Promisify — Convert Callbacks sang Promises

```js
const { promisify } = require('util')
const fs = require('fs')

// Promisify callback-based functions
const readFile = promisify(fs.readFile)
const writeFile = promisify(fs.writeFile)

// Hoặc dùng trực tiếp fs/promises (Node.js built-in)
const { readFile, writeFile, mkdir } = require('fs/promises')

// Custom promisify cho non-standard callbacks
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timeout after ${ms}ms`)), ms)
  )
  return Promise.race([promise, timeout])
}
```

---

## Promise Combinators

### Promise.all — Parallel, fail fast

```js
// Chạy tất cả song song — throw nếu BẤT KỲ promise nào reject
const [users, posts, comments] = await Promise.all([
  db.user.findMany(),
  db.post.findMany(),
  db.comment.findMany()
])
// Nếu posts reject → toàn bộ Promise.all reject

// Use case: fetch data độc lập song song
const [profile, settings, notifications] = await Promise.all([
  fetchProfile(userId),
  fetchSettings(userId),
  fetchNotifications(userId)
])
```

### Promise.allSettled — Parallel, không fail

```js
// Chờ TẤT CẢ hoàn thành, kể cả reject
const results = await Promise.allSettled([
  sendEmail(user1),
  sendEmail(user2),
  sendEmail(user3)  // có thể fail
])

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`Email ${index + 1} sent`)
  } else {
    console.error(`Email ${index + 1} failed:`, result.reason)
  }
})

// Use case: batch operations, send emails, fire-and-forget với error tracking
```

### Promise.race — Lấy cái nào xong trước

```js
// Resolve/reject theo promise đầu tiên hoàn thành
const result = await Promise.race([
  fetchFromPrimaryDB(id),
  fetchFromReplicaDB(id)   // lấy cái nào trả về trước
])

// Timeout pattern (phổ biến nhất)
async function fetchWithTimeout(url, timeoutMs = 5000) {
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Request timeout: ${url}`)), timeoutMs)
  )
  return Promise.race([fetch(url), timeoutPromise])
}
```

### Promise.any — Lấy cái nào resolve trước (ignore rejects)

```js
// Khác race: chỉ resolve khi có ít nhất 1 fulfilled
// Chỉ reject khi TẤT CẢ đều reject (AggregateError)
const html = await Promise.any([
  fetchFromCDN1(url),
  fetchFromCDN2(url),
  fetchFromOrigin(url)
])
// Nếu CDN1 fail nhưng CDN2 ok → trả về CDN2 result
```

---

## Sequential vs Parallel

```js
// ===== SEQUENTIAL — chậm hơn, dùng khi có dependencies =====
const user = await db.user.findById(userId)
const posts = await db.post.findByAuthor(user.id)  // phụ thuộc user
const enrichedPosts = await enrichWithMetadata(posts)  // phụ thuộc posts

// ===== PARALLEL — nhanh hơn, dùng khi độc lập =====
// Sai: sequential dù không cần
const profile = await fetchProfile(userId)
const settings = await fetchSettings(userId)  // không phụ thuộc profile

// Đúng: parallel
const [profile, settings] = await Promise.all([
  fetchProfile(userId),
  fetchSettings(userId)
])

// ===== SEQUENTIAL LOOP — đúng khi cần thứ tự =====
for (const item of items) {
  await processItem(item)  // xử lý tuần tự
}

// ===== PARALLEL LOOP — nhanh hơn khi độc lập =====
await Promise.all(items.map(item => processItem(item)))

// ===== CONTROLLED CONCURRENCY — không quá nhiều parallel =====
// Tránh gọi 10000 requests cùng lúc
const BATCH_SIZE = 10
for (let i = 0; i < items.length; i += BATCH_SIZE) {
  const batch = items.slice(i, i + BATCH_SIZE)
  await Promise.all(batch.map(processItem))
}
```

---

## Async Iteration

```js
// for await...of — dùng với async iterables
const { createReadStream } = require('fs')

async function countLines(filePath) {
  const stream = createReadStream(filePath, { encoding: 'utf8' })
  let lines = 0

  for await (const chunk of stream) {
    lines += chunk.split('\n').length - 1
  }

  return lines
}

// Async generator — tạo async iterable
async function* paginate(fetchPage, startPage = 1) {
  let page = startPage
  while (true) {
    const result = await fetchPage(page)
    if (!result.data.length) break
    yield result.data
    page++
  }
}

// Dùng async generator
for await (const pageData of paginate(fetchUsersPage)) {
  await processUsers(pageData)
}
```

---

## Common Async Patterns

```js
// Retry với exponential backoff
async function withRetry(fn, options = {}) {
  const { attempts = 3, delay = 1000, backoff = 2 } = options

  for (let i = 0; i < attempts; i++) {
    try {
      return await fn()
    } catch (err) {
      if (i === attempts - 1) throw err  // last attempt — re-throw
      const waitTime = delay * Math.pow(backoff, i)
      console.log(`Attempt ${i + 1} failed, retrying in ${waitTime}ms...`)
      await new Promise(resolve => setTimeout(resolve, waitTime))
    }
  }
}

// Usage
const user = await withRetry(
  () => fetchUser(id),
  { attempts: 3, delay: 500 }
)

// Debounce async function
function asyncDebounce(fn, delay) {
  let timer
  return (...args) => {
    clearTimeout(timer)
    return new Promise((resolve, reject) => {
      timer = setTimeout(() => fn(...args).then(resolve).catch(reject), delay)
    })
  }
}

// Memoize async function
function asyncMemoize(fn, ttl = 60000) {
  const cache = new Map()
  return async (...args) => {
    const key = JSON.stringify(args)
    const cached = cache.get(key)
    if (cached && Date.now() - cached.time < ttl) return cached.value

    const value = await fn(...args)
    cache.set(key, { value, time: Date.now() })
    return value
  }
}
```

---

## Bài tập thực hành

1. **Refactor callbacks**: Lấy một đoạn code có 3-level callback nesting, refactor sang Promises, sau đó sang async/await. Giải thích sự khác biệt về readability.

2. **Parallel fetch**: Viết function `fetchUserDashboard(userId)` fetch song song: profile, posts (10 mới nhất), friends (5), notifications (unread count). Return object gộp tất cả.

3. **Retry utility**: Implement `withRetry(fn, { attempts, initialDelay, backoffFactor, retryOn })` — `retryOn` là function nhận error và return boolean quyết định có retry không.

4. **Rate limiter**: Implement `rateLimitedMap(items, fn, { concurrency, ratePerSecond })` — giới hạn số concurrent operations và số operations per second.

---

## Resources

- [MDN — Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) — Promise API reference
- [javascript.info — Promises, async/await](https://javascript.info/async) — Tutorial đầy đủ
- [Node.js docs — util.promisify](https://nodejs.org/api/util.html#utilpromisifyoriginal) — Promisify API
