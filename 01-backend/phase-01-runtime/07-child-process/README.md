# Phase 1.7 — Child Process

Node.js là single-threaded, nhưng `child_process` module cho phép bạn spawn các process con — chạy shell commands, Python scripts, hoặc các Node.js scripts riêng biệt. Đây là cách để tận dụng multi-core CPUs, chạy external programs, hoặc isolate các tác vụ có thể crash.

---

## 4 Phương thức chính

| Method | Dùng khi | Có shell? | Streaming? | IPC? |
|--------|---------|-----------|------------|------|
| `exec()` | Shell command, output nhỏ | Có | Không | Không |
| `execFile()` | Chạy executable, an toàn hơn exec | Không | Không | Không |
| `spawn()` | Command với output lớn | Không | Có | Không |
| `fork()` | Chạy Node.js script với communication | Không | Có | Có |

---

## exec() — Shell Commands

```js
const { exec } = require('child_process')
const { promisify } = require('util')
const execAsync = promisify(exec)

// Callback style
exec('ls -la', (err, stdout, stderr) => {
  if (err) {
    console.error('Error:', err.message)
    return
  }
  console.log('stdout:', stdout)
  console.error('stderr:', stderr)
})

// Promise style (KHUYÊN DÙNG)
try {
  const { stdout, stderr } = await execAsync('git log --oneline -5')
  console.log(stdout)
} catch (err) {
  console.error('Exit code:', err.code)
  console.error('stderr:', err.stderr)
}

// Options
exec('npm install', {
  cwd: '/path/to/project',    // working directory
  env: { ...process.env, NODE_ENV: 'production' },
  timeout: 30000,             // kill after 30s
  maxBuffer: 1024 * 1024 * 10 // 10MB buffer
}, callback)

// CẢNH BÁO: exec dùng shell → có thể bị shell injection
const userInput = 'file.txt; rm -rf /'
exec(`cat ${userInput}`)  // NGUY HIỂM!
// Dùng execFile hoặc spawn thay thế khi có user input
```

---

## spawn() — Streaming Output

```js
const { spawn } = require('child_process')

// Chạy command với streaming output
const ls = spawn('ls', ['-la', '/tmp'])

ls.stdout.on('data', (data) => {
  process.stdout.write(data)  // stream output ra console
})

ls.stderr.on('data', (data) => {
  process.stderr.write(data)
})

ls.on('close', (code) => {
  console.log(`Process exited with code ${code}`)
})

ls.on('error', (err) => {
  console.error('Failed to start:', err)
})

// Pipe child output thẳng vào parent
const child = spawn('ping', ['localhost', '-c', '4'])
child.stdout.pipe(process.stdout)
child.stderr.pipe(process.stderr)

// stdin của child
const cat = spawn('cat')
cat.stdout.pipe(process.stdout)
cat.stdin.write('Hello from parent\n')
cat.stdin.end()
```

---

## fork() — Node.js Child với IPC

`fork()` là specialized version của `spawn()` dành riêng cho Node.js scripts. Nó tạo **IPC channel** (Inter-Process Communication) cho phép parent và child giao tiếp bằng messages.

```js
// parent.js
const { fork } = require('child_process')

const child = fork('./worker.js', [], {
  env: { ...process.env },
  silent: false  // false = child share stdout/stderr với parent
})

// Gửi message cho child
child.send({ task: 'compute', data: [1, 2, 3, 4, 5] })

// Nhận message từ child
child.on('message', (result) => {
  console.log('Result from child:', result)
})

child.on('exit', (code) => {
  console.log(`Worker exited: ${code}`)
})

child.on('error', (err) => {
  console.error('Worker error:', err)
})

// Timeout: kill worker nếu quá lâu
setTimeout(() => {
  child.kill('SIGTERM')
}, 5000)
```

```js
// worker.js
process.on('message', async (msg) => {
  const { task, data } = msg

  if (task === 'compute') {
    // CPU-intensive work ở đây không block parent's event loop
    const sum = data.reduce((a, b) => a + b, 0)
    const result = { sum, average: sum / data.length }
    process.send(result)
  }
})

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('Worker shutting down...')
  process.exit(0)
})
```

---

## Ví dụ thực tế: Worker Pool

```js
// worker-pool.js
const { fork } = require('child_process')

class WorkerPool {
  constructor(workerPath, poolSize = 4) {
    this.workers = []
    this.queue = []

    for (let i = 0; i < poolSize; i++) {
      this._createWorker(workerPath)
    }
  }

  _createWorker(workerPath) {
    const worker = fork(workerPath)
    worker.busy = false

    worker.on('message', (result) => {
      worker.busy = false
      if (worker._resolve) {
        worker._resolve(result)
        worker._resolve = null
      }
      // Process next queued task
      this._processQueue()
    })

    worker.on('error', (err) => {
      if (worker._reject) {
        worker._reject(err)
        worker._reject = null
      }
    })

    this.workers.push(worker)
  }

  _processQueue() {
    if (this.queue.length === 0) return

    const idleWorker = this.workers.find(w => !w.busy)
    if (!idleWorker) return

    const { task, resolve, reject } = this.queue.shift()
    idleWorker.busy = true
    idleWorker._resolve = resolve
    idleWorker._reject = reject
    idleWorker.send(task)
  }

  run(task) {
    return new Promise((resolve, reject) => {
      const idleWorker = this.workers.find(w => !w.busy)
      if (idleWorker) {
        idleWorker.busy = true
        idleWorker._resolve = resolve
        idleWorker._reject = reject
        idleWorker.send(task)
      } else {
        this.queue.push({ task, resolve, reject })
      }
    })
  }

  destroy() {
    this.workers.forEach(w => w.kill())
  }
}

// Usage
const pool = new WorkerPool('./compute-worker.js', 4)
const results = await Promise.all([
  pool.run({ data: [1, 2, 3] }),
  pool.run({ data: [4, 5, 6] }),
])
pool.destroy()
```

---

## Khi nào dùng gì

```
child_process.exec()     → chạy shell command, output nhỏ (< 200KB)
child_process.spawn()    → output lớn, stdin cần streaming
child_process.fork()     → Node.js script cần IPC, isolated process
worker_threads           → CPU-intensive JS code, share memory với SharedArrayBuffer
cluster                  → scale HTTP server lên nhiều cores
```

---

## Bài tập thực hành

1. **Shell wrapper**: Tạo module export async functions: `runCommand(cmd, args)`, `readDirectory(path)`, `getGitLog(repoPath)`. Dùng `spawn` với proper streaming.

2. **CPU worker**: Tạo `fibonacci-worker.js` tính số Fibonacci lớn. Dùng `fork()` để chạy từ parent, measure thời gian so với chạy trực tiếp trong main thread.

3. **Process monitor**: Spawn một long-running process, implement restart logic khi process crash, giới hạn số lần restart (max 3), emit events: `'start'`, `'exit'`, `'restart'`, `'failed'`.

---

## Resources

- [Node.js child_process docs](https://nodejs.org/api/child_process.html) — Official API reference
- [Node.js docs — Don't block the event loop](https://nodejs.org/en/docs/guides/dont-block-the-event-loop) — Khi nào cần offload work
- [Child Process vs Worker Threads](https://blog.insiderattack.net/deep-dive-into-worker-threads-in-node-js-e75e10546b11) — So sánh chi tiết
