# Phase 1.8 — Worker Threads

Worker Threads cho phép chạy JavaScript code song song trong các threads riêng biệt — mỗi thread có V8 engine instance và event loop riêng. Đây là giải pháp cho bài toán **CPU-bound tasks** mà không block main thread. Khác với `child_process`, worker threads share memory thông qua `SharedArrayBuffer` và truyền dữ liệu hiệu quả hơn qua `transferList`.

---

## Tại sao cần Worker Threads?

Node.js là single-threaded — nếu bạn chạy CPU-intensive code (image processing, compression, cryptography, machine learning inference), nó sẽ block event loop và làm chậm toàn bộ server:

```js
// BAD: block event loop ~200ms
app.get('/compute', (req, res) => {
  const result = heavyComputation(req.body.data)  // 200ms CPU work
  res.json(result)  // requests khác bị queue trong 200ms này
})

// GOOD: offload sang worker thread
app.get('/compute', (req, res) => {
  runInWorker(req.body.data).then(result => res.json(result))
  // event loop tiếp tục xử lý requests khác
})
```

---

## Cú pháp cơ bản

```js
// main.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads')

if (isMainThread) {
  // === Chạy ở main thread ===

  const worker = new Worker('./worker.js', {
    workerData: { numbers: [1, 2, 3, 4, 5] }  // data truyền vào worker
  })

  // Nhận kết quả
  worker.on('message', (result) => {
    console.log('Sum:', result.sum)
  })

  // Handle errors
  worker.on('error', (err) => {
    console.error('Worker error:', err)
  })

  // Handle exit
  worker.on('exit', (code) => {
    if (code !== 0) console.error(`Worker exited with code ${code}`)
  })

  // Gửi thêm data cho worker
  worker.postMessage({ type: 'compute', data: [6, 7, 8] })

} else {
  // === Chạy ở worker thread ===

  // workerData từ constructor
  const { numbers } = workerData
  const initialSum = numbers.reduce((a, b) => a + b, 0)
  parentPort.postMessage({ sum: initialSum })

  // Nhận messages từ main
  parentPort.on('message', ({ type, data }) => {
    if (type === 'compute') {
      const sum = data.reduce((a, b) => a + b, 0)
      parentPort.postMessage({ sum })
    }
  })
}
```

---

## Một file duy nhất cho cả main và worker

```js
// Tách code main và worker trong cùng một file
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads')

function heavyTask(data) {
  // CPU-intensive work
  let result = 0
  for (let i = 0; i < data.n; i++) {
    result += Math.sqrt(i) * Math.sin(i)
  }
  return result
}

if (isMainThread) {
  // Main thread: tạo worker
  function runWorker(data) {
    return new Promise((resolve, reject) => {
      const worker = new Worker(__filename, { workerData: data })
      worker.on('message', resolve)
      worker.on('error', reject)
      worker.on('exit', (code) => {
        if (code !== 0) reject(new Error(`Worker exited: ${code}`))
      })
    })
  }

  // Chạy 4 tasks song song
  const results = await Promise.all([
    runWorker({ n: 1000000 }),
    runWorker({ n: 1000000 }),
    runWorker({ n: 1000000 }),
    runWorker({ n: 1000000 }),
  ])
  console.log(results)

} else {
  // Worker thread: thực hiện tính toán
  const result = heavyTask(workerData)
  parentPort.postMessage(result)
}
```

---

## SharedArrayBuffer — Chia sẻ Memory

```js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads')

if (isMainThread) {
  // Tạo shared memory
  const sharedBuffer = new SharedArrayBuffer(4)  // 4 bytes = 1 Int32
  const sharedArray = new Int32Array(sharedBuffer)
  sharedArray[0] = 0  // initial value

  const worker = new Worker(__filename, {
    workerData: { sharedBuffer }
  })

  worker.on('exit', () => {
    console.log('Final value:', sharedArray[0])  // sẽ thấy value worker đã tăng
  })

} else {
  const sharedArray = new Int32Array(workerData.sharedBuffer)

  // Tăng atomically (thread-safe)
  Atomics.add(sharedArray, 0, 1)  // sharedArray[0]++

  // Atomic wait (dùng để synchronize)
  // Atomics.wait(sharedArray, 0, 0)  // chờ khi value là 0
  // Atomics.notify(sharedArray, 0, 1)  // wake 1 thread đang wait
}
```

---

## MessageChannel — Giao tiếp trực tiếp giữa Workers

```js
const { Worker, MessageChannel } = require('worker_threads')

const { port1, port2 } = new MessageChannel()

const worker1 = new Worker('./worker1.js', {
  workerData: { port: port1 },
  transferList: [port1]  // transfer ownership — quan trọng!
})

const worker2 = new Worker('./worker2.js', {
  workerData: { port: port2 },
  transferList: [port2]
})

// Bây giờ worker1 và worker2 có thể nói chuyện trực tiếp
// mà không qua main thread
```

---

## Worker Thread Pool (Pattern thực tế)

```js
const { Worker } = require('worker_threads')
const path = require('path')

class ThreadPool {
  constructor(workerFile, size = 4) {
    this.size = size
    this.queue = []
    this.workers = []
    this.freeWorkers = []

    for (let i = 0; i < size; i++) {
      this._addWorker(workerFile)
    }
  }

  _addWorker(workerFile) {
    const worker = new Worker(workerFile)
    worker.on('message', (result) => {
      if (worker._pendingResolve) {
        worker._pendingResolve(result)
        worker._pendingResolve = null
      }
      this.freeWorkers.push(worker)
      this._processQueue()
    })
    worker.on('error', (err) => {
      if (worker._pendingReject) {
        worker._pendingReject(err)
        worker._pendingReject = null
      }
    })
    this.freeWorkers.push(worker)
    this.workers.push(worker)
  }

  run(data) {
    return new Promise((resolve, reject) => {
      const worker = this.freeWorkers.pop()
      if (worker) {
        worker._pendingResolve = resolve
        worker._pendingReject = reject
        worker.postMessage(data)
      } else {
        this.queue.push({ data, resolve, reject })
      }
    })
  }

  _processQueue() {
    if (this.queue.length > 0 && this.freeWorkers.length > 0) {
      const { data, resolve, reject } = this.queue.shift()
      const worker = this.freeWorkers.pop()
      worker._pendingResolve = resolve
      worker._pendingReject = reject
      worker.postMessage(data)
    }
  }

  async destroy() {
    await Promise.all(this.workers.map(w => w.terminate()))
  }
}

// Usage
const pool = new ThreadPool('./compute-worker.js', 4)
const result = await pool.run({ operation: 'fib', n: 40 })
await pool.destroy()
```

---

## Khi nào dùng Worker Threads vs Child Process

| | Worker Threads | Child Process |
|--|----------------|---------------|
| Memory | Shared (`SharedArrayBuffer`) | Isolated |
| Communication | Fast (postMessage) | IPC (slower) |
| Crash isolation | Thấp (same process) | Cao (separate process) |
| Use case | CPU-bound JS code | External programs, isolation |
| Startup time | Nhanh | Chậm hơn |

---

## Bài tập thực hành

1. **Fibonacci benchmark**: So sánh tính `fib(45)` ở main thread vs worker thread — đo impact lên request throughput của HTTP server.

2. **Image processing**: Dùng worker threads để resize nhiều ảnh song song (dùng package `sharp`). Benchmark với 1, 2, 4 workers.

3. **Thread pool**: Implement `ThreadPool` class đầy đủ với: max concurrent workers, task queuing, timeout per task, graceful shutdown.

---

## Resources

- [Node.js Worker Threads docs](https://nodejs.org/api/worker_threads.html) — Official API reference
- [Worker Threads in Node.js](https://blog.insiderattack.net/deep-dive-into-worker-threads-in-node-js-e75e10546b11) — Deep dive
- [MDN — SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) — Shared memory reference
