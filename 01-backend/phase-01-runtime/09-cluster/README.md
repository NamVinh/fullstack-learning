# Phase 1.9 — Cluster Module

Cluster module cho phép bạn tạo nhiều Node.js processes (workers) chia sẻ cùng một server port — mỗi worker chạy trên một CPU core riêng. Đây là cách scale Node.js application để tận dụng toàn bộ CPU của máy chủ production.

---

## Tại sao cần Cluster?

Node.js là single-threaded — mặc định chỉ chạy trên **1 CPU core**. Nếu server của bạn có 8 cores, 7 cores đang idle. Cluster cho phép chạy 8 instances của app, cùng lắng nghe port 3000, OS sẽ phân phối connections giữa các workers.

```
        ┌─────────────────────────────────┐
        │         Primary Process         │
        │  (cluster.isPrimary — manager)  │
        └──┬──────┬──────┬──────┬────────┘
           │      │      │      │
     ┌─────▼─┐ ┌──▼──┐ ┌─▼──┐ ┌▼────┐
     │Worker │ │Work │ │Work│ │Work │   ← mỗi worker = 1 CPU core
     │  :3000│ │ :3000│ │:3000│ │:3000│
     └───────┘ └─────┘ └────┘ └─────┘
           ↑         ↑
     OS load balancer phân phối connections
```

---

## Cú pháp cơ bản

```js
// server.js
const cluster = require('cluster')
const { cpus } = require('os')
const express = require('express')

const NUM_WORKERS = cpus().length  // match số CPU cores

if (cluster.isPrimary) {
  // === Primary process: tạo và quản lý workers ===
  console.log(`Primary ${process.pid} is running`)
  console.log(`Forking ${NUM_WORKERS} workers...`)

  // Fork workers
  for (let i = 0; i < NUM_WORKERS; i++) {
    cluster.fork()
  }

  // Xử lý worker crash — tự động restart
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (${signal || code})`)
    console.log('Forking a new worker...')
    cluster.fork()  // restart worker
  })

  cluster.on('online', (worker) => {
    console.log(`Worker ${worker.process.pid} is online`)
  })

} else {
  // === Worker process: chạy actual server ===
  const app = express()

  app.get('/', (req, res) => {
    res.json({
      message: 'Hello from worker',
      pid: process.pid,
      workerId: cluster.worker.id
    })
  })

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} listening on port 3000`)
  })
}
```

---

## PM2 — Cluster Mode (Production approach)

Trong thực tế, ít ai dùng cluster module trực tiếp — PM2 handle việc này với nhiều features hơn:

```bash
# Chạy với cluster mode, tạo 1 instance per CPU core
pm2 start src/index.js --name api --instances max

# Hoặc số cụ thể
pm2 start src/index.js --name api --instances 4

# Xem status
pm2 status
pm2 monit

# Restart không downtime (rolling restart)
pm2 reload api

# Logs
pm2 logs api
```

```js
// ecosystem.config.js — cấu hình PM2
module.exports = {
  apps: [{
    name: 'api',
    script: 'src/index.js',
    instances: 'max',          // tự detect CPU cores
    exec_mode: 'cluster',      // cluster mode
    watch: false,
    max_memory_restart: '1G',  // restart nếu memory > 1GB
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
}

// pm2 start ecosystem.config.js --env production
```

---

## IPC giữa Primary và Workers

```js
// Primary gửi message cho tất cả workers
Object.values(cluster.workers).forEach(worker => {
  worker.send({ type: 'config-update', config: newConfig })
})

// Worker nhận message từ primary
process.on('message', (msg) => {
  if (msg.type === 'config-update') {
    updateConfig(msg.config)
  }
})

// Worker gửi message cho primary
process.send({ type: 'stats', requestCount: localCounter })

// Primary nhận
cluster.on('message', (worker, msg) => {
  if (msg.type === 'stats') {
    totalRequests += msg.requestCount
  }
})
```

---

## Zero-downtime Restart

```js
// Graceful restart: restart từng worker một, không drop connections
if (cluster.isPrimary) {
  const workers = Object.values(cluster.workers)

  async function gracefulRestart() {
    for (const worker of workers) {
      // Gửi signal shutdown cho worker cũ
      worker.send({ type: 'shutdown' })

      // Đợi worker tự exit (timeout 30s)
      await new Promise(resolve => {
        const timer = setTimeout(() => {
          worker.kill('SIGKILL')
          resolve()
        }, 30000)

        worker.on('exit', () => {
          clearTimeout(timer)
          resolve()
        })
      })

      // Fork worker mới
      cluster.fork()

      // Đợi worker mới online
      await new Promise(resolve => {
        cluster.once('online', resolve)
      })
    }
    console.log('Graceful restart complete')
  }

  process.on('SIGUSR2', gracefulRestart)  // kill -USR2 <pid>
}

// Worker: handle shutdown signal
process.on('message', (msg) => {
  if (msg.type === 'shutdown') {
    server.close(() => process.exit(0))
  }
})
```

---

## Shared State Problem

Workers là separate processes — không share memory:

```js
// SẼ KHÔNG HOẠT ĐỘNG với cluster:
let counter = 0  // mỗi worker có counter riêng!

app.get('/increment', (req, res) => {
  counter++
  res.json({ counter })  // mỗi worker trả về giá trị khác nhau
})

// GIẢI PHÁP: dùng Redis để share state
const redis = require('redis')
const client = redis.createClient()

app.get('/increment', async (req, res) => {
  const counter = await client.incr('counter')  // atomic increment
  res.json({ counter })  // consistent across all workers
})
```

---

## Khi nào dùng Cluster vs Worker Threads vs Child Process

| Tình huống | Giải pháp |
|-----------|-----------|
| Scale HTTP server lên nhiều CPU cores | **Cluster** |
| CPU-intensive computation trong JS | **Worker Threads** |
| Chạy external program (Python, bash) | **Child Process (spawn/exec)** |
| Isolate crash-prone code | **Child Process (fork)** |
| Production HTTP server | **PM2 cluster mode** hoặc **Docker + multiple containers** |

---

## Bài tập thực hành

1. **Basic cluster**: Tạo Express server với cluster. Verify bằng cách log `process.pid` trong mỗi request — thấy nhiều PIDs khác nhau khi load test.

2. **Load testing**: Dùng `autocannon` hoặc `k6`, so sánh throughput (requests/second) giữa: no cluster vs cluster (max workers). Chạy trên machine có ít nhất 4 cores.

3. **Health monitoring**: Primary process collect metrics từ tất cả workers qua IPC mỗi 10s, tính tổng requests served, average response time, expose endpoint `/cluster/stats`.

4. **Zero-downtime restart**: Implement graceful restart khi nhận `SIGUSR2` signal — restart workers lần lượt, không có downtime.

---

## Resources

- [Node.js Cluster docs](https://nodejs.org/api/cluster.html) — Official API reference
- [PM2 docs](https://pm2.keymetrics.io/docs/usage/cluster-mode/) — PM2 cluster mode
- [Scaling Node.js Applications](https://www.freecodecamp.org/news/scaling-node-js-applications-8492bd8afadc/) — Comprehensive guide
