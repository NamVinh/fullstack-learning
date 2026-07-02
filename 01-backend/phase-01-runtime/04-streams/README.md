# Phase 1.4 — Streams

Streams là một trong những tính năng mạnh mẽ nhất của Node.js. Thay vì load toàn bộ dữ liệu vào memory cùng một lúc, streams xử lý dữ liệu theo từng **chunk** — điều này cho phép xử lý files hàng GB, HTTP responses lớn, hoặc bất kỳ nguồn dữ liệu liên tục nào mà không làm crash server vì hết RAM.

---

## 4 Loại Stream

| Type | Ví dụ | Mô tả |
|------|-------|-------|
| **Readable** | `fs.createReadStream()`, HTTP request body | Chỉ đọc — nguồn dữ liệu |
| **Writable** | `fs.createWriteStream()`, HTTP response | Chỉ ghi — đích đến dữ liệu |
| **Duplex** | `net.Socket` | Vừa đọc vừa ghi, độc lập |
| **Transform** | `zlib.createGzip()` | Duplex + biến đổi dữ liệu khi đi qua |

---

## Readable Streams

```js
const fs = require('fs')

// Tạo readable stream từ file
const readable = fs.createReadStream('./bigfile.csv', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024  // chunk size 64KB (default 16KB)
})

// Cách 1: Event-based
readable.on('data', (chunk) => {
  console.log(`Received ${chunk.length} chars`)
})
readable.on('end', () => console.log('Done reading'))
readable.on('error', (err) => console.error(err))

// Cách 2: Async iteration (KHUYÊN DÙNG — cleaner)
async function processFile() {
  for await (const chunk of readable) {
    // process chunk
    process.stdout.write(chunk)
  }
}

// Modes: paused (default) vs flowing
// readable.resume() — chuyển sang flowing mode
// readable.pause()  — quay lại paused mode
```

---

## Writable Streams

```js
const writable = fs.createWriteStream('./output.txt')

// write() trả về boolean:
// true  = internal buffer chưa đầy, có thể tiếp tục write
// false = buffer đầy, phải đợi 'drain' event
const canContinue = writable.write('Hello World\n')

if (!canContinue) {
  // Đợi buffer drain trước khi write tiếp
  writable.once('drain', () => {
    writable.write('Next chunk\n')
  })
}

// Kết thúc stream
writable.end('Final line\n')
writable.on('finish', () => console.log('All written'))
```

---

## Pipe — Kết nối Streams

`pipe()` tự động xử lý **backpressure** — đây là cách clean nhất để connect streams:

```js
const { createReadStream, createWriteStream } = require('fs')
const { createGzip } = require('zlib')

// Đọc file → compress với gzip → ghi ra file mới
createReadStream('./input.txt')
  .pipe(createGzip())
  .pipe(createWriteStream('./input.txt.gz'))
  .on('finish', () => console.log('Compression complete'))

// Nếu muốn xử lý lỗi:
const pipeline = require('stream').pipeline
pipeline(
  createReadStream('./input.txt'),
  createGzip(),
  createWriteStream('./output.gz'),
  (err) => {
    if (err) console.error('Pipeline failed:', err)
    else console.log('Pipeline succeeded')
  }
)

// Hoặc dùng util.promisify(pipeline) với async/await
const { promisify } = require('util')
const pipelineAsync = promisify(require('stream').pipeline)

await pipelineAsync(
  createReadStream('./input.txt'),
  createGzip(),
  createWriteStream('./output.gz')
)
```

---

## Backpressure

Backpressure xảy ra khi writable không theo kịp tốc độ của readable. Nếu không handle, buffer sẽ tăng không giới hạn gây tràn memory.

```js
// pipe() tự handle backpressure — dừng readable khi writable buffer đầy
readable.pipe(writable)

// Manual backpressure handling
readable.on('data', (chunk) => {
  const canContinue = writable.write(chunk)
  if (!canContinue) {
    readable.pause()                      // dừng nhận data
    writable.once('drain', () => {
      readable.resume()                   // tiếp tục khi writable sẵn sàng
    })
  }
})
```

**`highWaterMark`**: là threshold, KHÔNG phải hard limit. Khi internal buffer đạt mức này, stream báo hiệu "dừng gửi thêm" — stream vẫn có thể nhận thêm data nhưng nên tránh.

---

## Transform Streams

Transform streams cho phép modify dữ liệu khi nó đi qua:

```js
const { Transform } = require('stream')

// Custom Transform: uppercase mọi text
class UpperCaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase())
    callback()  // signal đã xử lý xong chunk này
  }

  _flush(callback) {
    // Gọi trước khi stream kết thúc — cleanup, flush buffer còn lại
    callback()
  }
}

const upper = new UpperCaseTransform()

// Sử dụng
fs.createReadStream('./input.txt')
  .pipe(upper)
  .pipe(fs.createWriteStream('./output.txt'))

// Hoặc dùng stream.Transform function shorthand
const { Transform } = require('stream')
const csvToJson = new Transform({
  objectMode: true,  // cho phép emit objects thay vì Buffer/string
  transform(chunk, encoding, callback) {
    const row = chunk.toString().split(',')
    this.push({ col1: row[0], col2: row[1] })
    callback()
  }
})
```

---

## Stream trong HTTP

```js
const http = require('http')
const fs = require('fs')
const { createGzip } = require('zlib')

const server = http.createServer((req, res) => {
  // Pipe file trực tiếp vào HTTP response — không load vào memory!
  const acceptEncoding = req.headers['accept-encoding'] || ''

  if (acceptEncoding.includes('gzip')) {
    res.writeHead(200, {
      'Content-Encoding': 'gzip',
      'Content-Type': 'text/plain'
    })
    fs.createReadStream('./bigfile.txt')
      .pipe(createGzip())
      .pipe(res)
  } else {
    res.writeHead(200, { 'Content-Type': 'text/plain' })
    fs.createReadStream('./bigfile.txt').pipe(res)
  }
})
```

---

## Bài tập thực hành

1. **Memory comparison**: Viết hai script đọc file 1GB — một dùng `readFileSync`, một dùng `createReadStream` và đếm số dòng. Đo memory usage bằng `process.memoryUsage()`.

2. **Custom Transform**: Implement Transform stream chuyển CSV sang JSON objects, pipe vào `JSON.stringify` stream và ghi ra file.

3. **Compression tool**: CLI tool nhận input file, compress bằng `zlib.createGzip()`, ghi ra `<filename>.gz`. Thêm progress indicator hiển thị % hoàn thành.

4. **Stream pipeline error handling**: Viết `safePipeline(...streams)` function wrap `stream.pipeline` với async/await và proper cleanup khi có lỗi ở bất kỳ stream nào.

---

## Resources

- [Node.js Streams docs](https://nodejs.org/api/stream.html) — Official API reference
- [Node.js Streams — Everything you need to know](https://www.freecodecamp.org/news/node-js-streams-everything-you-need-to-know-c9141306be93/) — Bài viết toàn diện
- [Backpressuring in Streams](https://nodejs.org/en/docs/guides/backpressuring-in-streams) — Official guide về backpressure
