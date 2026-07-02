# Phase 13 — Performance & Monitoring

Production apps cần observability — khả năng hiểu system đang làm gì từ bên ngoài. Ba trụ cột: **Logging** (ghi lại events), **Metrics** (đo lường), và **Tracing** (theo dõi request flow). Ngoài ra, cần biết profiling để tìm performance bottlenecks.

---

## Logging với Pino

Pino là fastest JSON logger cho Node.js — 10x nhanh hơn Winston, output JSON structured logs để dễ search/analyze.

```bash
npm install pino pino-pretty  # pino-pretty chỉ cho dev
```

```js
// src/logger.js
const pino = require('pino')

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  // Dev: human-readable output
  transport: process.env.NODE_ENV === 'development' ? {
    target: 'pino-pretty',
    options: { colorize: true, translateTime: 'SYS:standard' }
  } : undefined,
  // Base context cho tất cả logs
  base: {
    pid: process.pid,
    service: 'api',
    env: process.env.NODE_ENV
  },
  // Redact sensitive fields khỏi logs
  redact: {
    paths: ['req.headers.authorization', 'body.password', '*.token'],
    censor: '[REDACTED]'
  }
})

module.exports = logger
```

```js
// Usage
const logger = require('./logger')

// Structured logging — object trước, message sau
logger.info({ userId: 1, action: 'login' }, 'User logged in')
logger.warn({ requestId: 'abc', duration: 2500 }, 'Slow request detected')
logger.error({ err: new Error('DB failed'), requestId: 'xyz' }, 'Database error')

// Child logger — thêm context tự động
const childLogger = logger.child({ requestId: 'abc123', userId: 42 })
childLogger.info('Processing request')
childLogger.error({ err }, 'Request failed')
// Logs sẽ luôn có requestId và userId
```

### Express Request Logger Middleware

```js
const logger = require('./logger')

function requestLogger(req, res, next) {
  const requestId = req.headers['x-request-id'] || crypto.randomUUID()
  const startTime = Date.now()

  req.log = logger.child({ requestId, method: req.method, path: req.path })
  res.setHeader('X-Request-ID', requestId)

  req.log.info({ ip: req.ip }, 'Request started')

  res.on('finish', () => {
    const duration = Date.now() - startTime
    const logFn = res.statusCode >= 500 ? 'error'
                : res.statusCode >= 400 ? 'warn'
                : 'info'

    req.log[logFn]({
      statusCode: res.statusCode,
      duration,
      contentLength: res.get('Content-Length')
    }, 'Request completed')
  })

  next()
}

app.use(requestLogger)
```

---

## Profiling

### CPU Profiling

```bash
# 1. Tạo CPU profile
node --prof src/index.js
# Chạy app, gửi traffic, Ctrl+C

# 2. Process profile
node --prof-process isolate-*.log > profile.txt
cat profile.txt | head -100

# 3. Hoặc dùng Chrome DevTools (inspector)
node --inspect src/index.js
# Mở Chrome: chrome://inspect
# "Open dedicated DevTools for Node"
# Tab Profiler → Start profiling
```

### Memory Profiling

```bash
# Heap snapshot
node --inspect src/index.js
# Chrome DevTools → Memory → Take Heap Snapshot
# So sánh snapshots để tìm memory leaks
```

```js
// Detect memory leaks trong code
const used = process.memoryUsage()
console.log({
  rss: `${Math.round(used.rss / 1024 / 1024)} MB`,        // Resident Set Size
  heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
  heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
  external: `${Math.round(used.external / 1024 / 1024)} MB`
})

// Limit memory
// node --max-old-space-size=512 src/index.js  → crash nếu > 512MB
```

---

## OpenTelemetry — Distributed Tracing

OpenTelemetry là standard cho observability — instrument code một lần, export sang nhiều backends (Jaeger, Zipkin, Datadog, etc.):

```bash
npm install @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node \
            @opentelemetry/exporter-trace-otlp-http
```

```js
// src/tracing.js — PHẢI import trước mọi thứ khác
const { NodeSDK } = require('@opentelemetry/sdk-node')
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node')
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http')
const { resourceFromAttributes } = require('@opentelemetry/resources')
const { SEMRESATTRS_SERVICE_NAME } = require('@opentelemetry/semantic-conventions')

const sdk = new NodeSDK({
  resource: resourceFromAttributes({
    [SEMRESATTRS_SERVICE_NAME]: 'my-api',
  }),
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4318/v1/traces'
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      // Auto-instrument: HTTP, Express, pg, MongoDB, Redis, gRPC...
      '@opentelemetry/instrumentation-fs': { enabled: false },  // disable fs
    })
  ]
})

sdk.start()

// Graceful shutdown
process.on('SIGTERM', () => sdk.shutdown())
```

```js
// src/index.js — import tracing FIRST
require('./tracing')

const express = require('express')
// ... rest of app

// Manual span creation
const { trace, context, SpanStatusCode } = require('@opentelemetry/api')
const tracer = trace.getTracer('my-api')

async function getUserWithPosts(userId) {
  const span = tracer.startSpan('getUserWithPosts')

  return context.with(trace.setSpan(context.active(), span), async () => {
    try {
      const user = await db.user.findById(userId)
      span.setAttribute('user.id', userId)

      const posts = await db.post.findByAuthor(userId)
      span.setAttribute('posts.count', posts.length)

      return { user, posts }
    } catch (err) {
      span.recordException(err)
      span.setStatus({ code: SpanStatusCode.ERROR, message: err.message })
      throw err
    } finally {
      span.end()
    }
  })
}
```

---

## Metrics với Prometheus

```bash
npm install prom-client
```

```js
const client = require('prom-client')

// Enable default metrics (CPU, memory, event loop lag, etc.)
client.collectDefaultMetrics({ prefix: 'myapp_' })

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5]
})

const activeConnections = new client.Gauge({
  name: 'http_active_connections',
  help: 'Number of active HTTP connections'
})

const dbQueryDuration = new client.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Database query duration',
  labelNames: ['operation', 'model']
})

// Middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer()
  activeConnections.inc()

  res.on('finish', () => {
    const route = req.route?.path || req.path
    end({ method: req.method, route, status_code: res.statusCode })
    activeConnections.dec()
  })

  next()
})

// Expose metrics endpoint cho Prometheus scraping
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType)
  res.end(await client.register.metrics())
})
```

---

## Performance Tips

```js
// 1. Giới hạn memory
// node --max-old-space-size=512 src/index.js

// 2. HTTP response compression
const compression = require('compression')
app.use(compression({
  threshold: 1024,  // chỉ compress response > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false
    return compression.filter(req, res)
  }
}))

// 3. Connection pooling cho PostgreSQL
const pool = new Pool({
  max: process.env.DB_POOL_SIZE || (os.cpus().length * 2),  // CPU cores × 2
  idleTimeoutMillis: 30000
})

// 4. Cache frequently accessed, rarely changed data
const NodeCache = require('node-cache')
const cache = new NodeCache({ stdTTL: 600 })  // 10 min default TTL

async function getCachedSettings() {
  const cached = cache.get('app-settings')
  if (cached) return cached

  const settings = await db.setting.findMany()
  cache.set('app-settings', settings)
  return settings
}

// 5. Avoid blocking event loop
// CPU-bound work → Worker Threads
// File I/O → fs/promises (non-blocking)
// Never: fs.readFileSync trong route handlers

// 6. Database query optimization
// - Use indexes trên fields thường dùng trong WHERE, ORDER BY
// - Select chỉ fields cần thiết (không SELECT *)
// - Use EXPLAIN ANALYZE để xem query plan
// - Implement pagination (không trả về unlimited rows)
```

---

## Event Loop Lag Monitoring

```js
// Detect event loop blocking
const eventLoopMonitor = () => {
  let lastCheck = Date.now()

  setInterval(() => {
    const now = Date.now()
    const lag = now - lastCheck - 1000  // expected 1000ms
    lastCheck = now

    if (lag > 100) {  // > 100ms lag = event loop bị blocked
      logger.warn({ lag }, 'Event loop lag detected')
    }
  }, 1000)
}

eventLoopMonitor()

// Với clinic.js — advanced profiling
// npm install -g clinic
// clinic doctor -- node src/index.js
// clinic flame -- node src/index.js  # flame graph
```

---

## Bài tập thực hành

1. **Structured logging**: Setup Pino với request ID tracking, child loggers, redacted sensitive fields. Verify logs có format đúng và requestId được trace qua toàn bộ request.

2. **Prometheus metrics**: Expose `/metrics` endpoint với: HTTP request count/duration per route/method, active connections, DB query duration. Setup Prometheus + Grafana với Docker Compose.

3. **CPU profiling**: Build một Express route có performance issue (nhân hai số lớn trong loop). Profile với `--prof`, identify hot spot, fix với Worker Thread.

4. **OpenTelemetry tracing**: Instrument app với auto-instrumentation, export sang Jaeger (Docker), trace một request từ HTTP endpoint → service → database → response.

---

## Resources

- [Pino docs](https://getpino.io) — Fast Node.js logger
- [OpenTelemetry Node.js](https://opentelemetry.io/docs/languages/js/) — Instrumentation guide
- [clinic.js](https://clinicjs.org) — Node.js performance profiling tool
