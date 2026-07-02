# Phase 1.6 — Events & EventEmitter

EventEmitter là nền tảng của mô hình lập trình event-driven trong Node.js. Hầu hết các built-in modules (http, fs, stream, net) đều extend từ EventEmitter. Hiểu EventEmitter giúp bạn hiểu cách Node.js hoạt động ở tầng sâu hơn và cho phép bạn xây dựng các loosely coupled systems.

---

## Cơ bản về EventEmitter

```js
const EventEmitter = require('events')

// Tạo emitter
const emitter = new EventEmitter()

// Đăng ký listener
emitter.on('data', (payload) => {
  console.log('Received:', payload)
})

// Listener chỉ chạy một lần
emitter.once('connect', () => {
  console.log('Connected! (sẽ không in lại)')
})

// Emit event
emitter.emit('data', { value: 42, timestamp: Date.now() })
emitter.emit('connect')
emitter.emit('connect')  // listener không chạy lần 2

// Remove listener
const handler = (data) => console.log(data)
emitter.on('message', handler)
emitter.off('message', handler)       // remove specific listener
emitter.removeAllListeners('message') // remove tất cả listeners của event này
emitter.removeAllListeners()          // remove tất cả listeners
```

---

## Extend EventEmitter (Pattern phổ biến nhất)

```js
const EventEmitter = require('events')

class DataProcessor extends EventEmitter {
  constructor() {
    super()
    this.processedCount = 0
  }

  async process(data) {
    this.emit('start', { data })

    try {
      // Giả sử xử lý phức tạp
      const result = await this._doWork(data)
      this.processedCount++
      this.emit('done', { result, count: this.processedCount })
    } catch (err) {
      this.emit('error', err)  // LUÔN emit 'error' event khi có lỗi
    }
  }

  async _doWork(data) {
    return { processed: data, at: new Date() }
  }
}

// Sử dụng
const processor = new DataProcessor()

processor.on('start', ({ data }) => console.log('Processing:', data))
processor.on('done', ({ result, count }) => console.log(`Done #${count}:`, result))
processor.on('error', (err) => console.error('Failed:', err))

await processor.process({ id: 1, value: 'hello' })
```

---

## Error Events — QUAN TRỌNG

Nếu bạn emit `'error'` event mà không có listener, Node.js sẽ **throw error và crash process**:

```js
const emitter = new EventEmitter()

// NGUY HIỂM: không có error listener
emitter.emit('error', new Error('Something wrong'))
// → Uncaught Error: Something wrong — PROCESS CRASHES

// ĐÚNG: luôn có error listener
emitter.on('error', (err) => {
  console.error('Handled error:', err.message)
  // xử lý gracefully, không crash
})

emitter.emit('error', new Error('Something wrong'))
// → Handled error: Something wrong
```

---

## Listener Management

```js
const emitter = new EventEmitter()

// Default max listeners = 10 (Node cảnh báo nếu > 10)
emitter.setMaxListeners(20)         // tăng giới hạn
emitter.setMaxListeners(Infinity)   // không giới hạn

// Kiểm tra listeners
emitter.listenerCount('data')        // số listeners cho event 'data'
emitter.listeners('data')            // mảng listener functions
emitter.eventNames()                 // ['data', 'error', 'connect']

// prepend — listener chạy TRƯỚC các listeners khác
emitter.prependListener('data', (chunk) => console.log('First!', chunk))
emitter.prependOnceListener('data', (chunk) => console.log('First once!', chunk))
```

---

## Async Listeners

EventEmitter không natively support async listeners — errors trong async listener có thể bị nuốt:

```js
const emitter = new EventEmitter()

// NGUY HIỂM: error trong async listener không propagate đúng
emitter.on('data', async (data) => {
  await someAsyncWork(data)  // nếu throw, không ai bắt được
})

// ĐÚNG: tự wrap try/catch
emitter.on('data', async (data) => {
  try {
    await someAsyncWork(data)
  } catch (err) {
    emitter.emit('error', err)  // forward error đúng cách
  }
})

// Hoặc dùng EventEmitter.captureRejections = true (Node 12+)
const emitter = new EventEmitter({ captureRejections: true })
emitter.on('error', (err) => console.error(err))  // sẽ catch async errors
emitter.on('data', async (data) => {
  await someAsyncWork(data)  // errors sẽ được forward đến 'error' event
})
```

---

## Ví dụ thực tế: Custom Logger với Events

```js
const EventEmitter = require('events')
const fs = require('fs')

class Logger extends EventEmitter {
  constructor(options = {}) {
    super()
    this.logFile = options.logFile
    this.level = options.level || 'info'

    if (this.logFile) {
      this.fileStream = fs.createWriteStream(this.logFile, { flags: 'a' })
    }
  }

  log(level, message, meta = {}) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...meta
    }

    this.emit('log', entry)

    if (this.fileStream) {
      this.fileStream.write(JSON.stringify(entry) + '\n')
    }
  }

  info(message, meta) { this.log('info', message, meta) }
  error(message, meta) { this.log('error', message, meta) }
  warn(message, meta) { this.log('warn', message, meta) }
}

const logger = new Logger({ logFile: './app.log' })
logger.on('log', (entry) => {
  const color = entry.level === 'error' ? '\x1b[31m' : '\x1b[32m'
  console.log(`${color}[${entry.level.toUpperCase()}]\x1b[0m ${entry.message}`)
})

logger.info('Server started', { port: 3000 })
logger.error('Database connection failed', { host: 'localhost' })
```

---

## Bài tập thực hành

1. **Custom EventEmitter**: Implement một `TaskQueue` class extend EventEmitter với methods: `add(task)`, `run()`, `pause()`, `resume()`. Emit events: `'task:start'`, `'task:done'`, `'task:error'`, `'queue:empty'`.

2. **Event bus**: Tạo singleton `EventBus` (module-level instance) cho global communication giữa các modules. Implement `subscribe(event, handler)`, `publish(event, data)`, `unsubscribe(event, handler)`.

3. **Typed events với TypeScript**: (nếu biết TS) Tạo `TypedEventEmitter<T>` generic class cho type-safe events.

4. **Observer pattern**: Implement `Store` class (như Redux store đơn giản) dùng EventEmitter — `setState(updates)` emit `'change'` event, `subscribe(listener)` return unsubscribe function.

---

## Resources

- [Node.js Events docs](https://nodejs.org/api/events.html) — Official API reference
- [Node.js Design Patterns — Observer Pattern](https://www.nodejsdesignpatterns.com) — Sách Mario Casciaro
- [EventEmitter vs Pub/Sub vs Observable](https://medium.com/@szaranger/nodejs-eventemitter-pub-sub-and-observable-pattern-comparison) — So sánh các patterns
