# Phase 1.3 — Built-in Modules (fs, path, http, crypto, os, process)

Node.js đi kèm với một thư viện chuẩn phong phú không cần cài thêm package. Đây là những module bạn sẽ dùng hàng ngày — nắm vững chúng giúp bạn không phụ thuộc quá nhiều vào third-party libraries cho các tác vụ cơ bản.

---

## `fs` — File System

Module `fs` cung cấp API để đọc/ghi files và thao tác với filesystem. Có 3 dạng API:

```js
const fs = require('fs')
const { readFile, writeFile, mkdir, readdir } = require('fs/promises')

// === Sync — BLOCK event loop, chỉ dùng khi startup ===
const data = fs.readFileSync('./config.json', 'utf8')
const config = JSON.parse(data)

// === Callback — cũ, tránh dùng nếu có thể ===
fs.readFile('./file.txt', 'utf8', (err, data) => {
  if (err) throw err
  console.log(data)
})

// === Promises (fs/promises) — KHUYÊN DÙNG ===
const data = await readFile('./file.txt', 'utf8')
await writeFile('./output.txt', 'Hello World', 'utf8')

// Tạo thư mục (recursive = không lỗi nếu đã tồn tại)
await mkdir('./logs/2024', { recursive: true })

// Đọc danh sách files
const files = await readdir('./src')

// Kiểm tra file tồn tại
import { access, constants } from 'fs/promises'
try {
  await access('./file.txt', constants.F_OK)
  console.log('File exists')
} catch {
  console.log('File not found')
}
```

### File streaming (cho file lớn)

```js
// Đọc file lớn không load hết vào memory
const stream = fs.createReadStream('./bigfile.csv', { encoding: 'utf8' })
stream.on('data', (chunk) => process.stdout.write(chunk))
stream.on('end', () => console.log('Done'))

// Ghi file lớn
const writeStream = fs.createWriteStream('./output.csv')
writeStream.write('header,row\n')
writeStream.end('last,line\n')
```

**Bài tập fs**: Viết script đọc file CSV, parse từng dòng, ghi ra file JSON.

---

## `path` — File Paths

Module `path` giúp thao tác với đường dẫn file cross-platform (Windows dùng `\`, Unix dùng `/`).

```js
const path = require('path')

// join — nối paths, tự handle separators
path.join('/foo', 'bar', 'baz.txt')      // /foo/bar/baz.txt
path.join(__dirname, 'src', 'index.js')  // absolute path đến src/index.js

// resolve — tạo absolute path từ relative
path.resolve('src', 'index.js')      // /current/working/dir/src/index.js
path.resolve('/foo', '../bar')        // /bar

// Parse path thành các components
path.dirname('/foo/bar/baz.txt')      // /foo/bar
path.basename('/foo/bar/baz.txt')     // baz.txt
path.basename('/foo/bar/baz.txt', '.txt')  // baz
path.extname('index.html')            // .html

// parse() và format()
const parsed = path.parse('/home/user/file.txt')
// { root: '/', dir: '/home/user', base: 'file.txt', ext: '.txt', name: 'file' }

// CJS globals (không có trong ESM)
console.log(__dirname)   // directory của file hiện tại
console.log(__filename)  // full path của file hiện tại
```

---

## `http` — HTTP Server

Module `http` cho phép tạo HTTP servers và clients không cần thư viện bên ngoài.

```js
const http = require('http')

const server = http.createServer((req, res) => {
  const { method, url, headers } = req

  // Parse URL
  const parsedUrl = new URL(url, `http://${headers.host}`)
  const pathname = parsedUrl.pathname
  const query = Object.fromEntries(parsedUrl.searchParams)

  // Route handling
  if (method === 'GET' && pathname === '/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' })
    res.end(JSON.stringify([{ id: 1, name: 'Alice' }]))
    return
  }

  if (method === 'POST' && pathname === '/users') {
    let body = ''
    req.on('data', (chunk) => body += chunk)
    req.on('end', () => {
      const data = JSON.parse(body)
      res.writeHead(201, { 'Content-Type': 'application/json' })
      res.end(JSON.stringify({ id: 2, ...data }))
    })
    return
  }

  // 404
  res.writeHead(404, { 'Content-Type': 'application/json' })
  res.end(JSON.stringify({ error: 'Not found' }))
})

server.listen(3000, () => console.log('Server running on port 3000'))
```

**Bài tập http**: Tạo HTTP server thuần (không dùng Express) trả về JSON, handle `/users` (GET, POST) và `/posts` (GET).

---

## `process` — Process Control

`process` là global object — không cần `require()`.

```js
// Environment variables
process.env.NODE_ENV    // 'development', 'production', 'test'
process.env.PORT        // undefined nếu không set
const PORT = process.env.PORT || 3000

// Command line arguments
// node script.js --name Alice --verbose
process.argv  // ['node', '/path/to/script.js', '--name', 'Alice', '--verbose']
process.argv.slice(2)  // ['--name', 'Alice', '--verbose']

// Working directory
process.cwd()  // /current/working/directory

// Process info
process.pid       // process ID
process.platform  // 'linux', 'darwin', 'win32'
process.version   // 'v20.11.0'

// Exit
process.exit(0)   // thoát thành công
process.exit(1)   // thoát với lỗi

// Event handlers — PHẢI có cho production apps
process.on('uncaughtException', (err) => {
  console.error('Uncaught:', err)
  process.exit(1)  // không thể tiếp tục sau uncaughtException
})
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection at:', promise, 'reason:', reason)
  process.exit(1)
})
process.on('SIGTERM', () => {
  console.log('SIGTERM received — graceful shutdown')
  server.close(() => process.exit(0))
})
process.on('SIGINT', () => {  // Ctrl+C
  process.exit(0)
})

// Standard streams
process.stdout.write('Hello ')
process.stderr.write('Error message\n')
process.stdin.on('data', (data) => console.log(data.toString()))
```

---

## `crypto` — Cryptography

```js
const crypto = require('crypto')

// UUID v4
const id = crypto.randomUUID()  // 'a4e3b891-...'

// Random bytes (cho tokens, secrets)
const token = crypto.randomBytes(32).toString('hex')   // 64 char hex string
const b64token = crypto.randomBytes(32).toString('base64url')  // URL-safe base64

// Hashing
const hash = crypto.createHash('sha256')
  .update('some data')
  .update('more data')  // có thể chain
  .digest('hex')

// HMAC (hash with secret key)
const hmac = crypto.createHmac('sha256', 'secret-key')
  .update(JSON.stringify(payload))
  .digest('hex')

// Verify HMAC (timing-safe comparison)
const isValid = crypto.timingSafeEqual(
  Buffer.from(receivedHmac, 'hex'),
  Buffer.from(expectedHmac, 'hex')
)
```

---

## `os` — Operating System Info

```js
const os = require('os')

os.cpus()           // array of CPU core info [{model, speed, times}]
os.cpus().length    // số CPU cores
os.totalmem()       // total RAM in bytes
os.freemem()        // free RAM in bytes
os.tmpdir()         // '/tmp' on Unix, 'C:\Users\...\AppData\Local\Temp' on Windows
os.homedir()        // '/Users/username'
os.hostname()       // 'my-machine.local'
os.platform()       // 'linux', 'darwin', 'win32'
os.arch()           // 'x64', 'arm64'
os.networkInterfaces()  // network interface info

// Dùng thực tế
const numCores = os.cpus().length
const cluster = require('cluster')
for (let i = 0; i < numCores; i++) {
  cluster.fork()  // tạo 1 worker per CPU core
}
```

---

## Bài tập thực hành

1. **File manager**: Viết CLI tool có thể `list`, `read`, `write`, `delete` files trong một thư mục. Dùng `fs/promises` và `path`.

2. **HTTP server thuần**: Build REST API không dùng Express cho resource `todos` — GET all, GET by id, POST (tạo mới), DELETE. Lưu vào file JSON.

3. **System info dashboard**: Script in ra: CPU model, số cores, RAM total/free, uptime, platform, hostname. Format đẹp với đơn vị (GB, GHz).

4. **Crypto utilities**: Viết module `crypto-utils.js` export: `generateToken(length)`, `hashPassword(password)` dùng PBKDF2, `verifyPassword(password, hash)`, `signData(data, secret)`, `verifySignature(data, signature, secret)`.

---

## Resources

- [Node.js fs docs](https://nodejs.org/api/fs.html) — File System API
- [Node.js path docs](https://nodejs.org/api/path.html) — Path utilities
- [Node.js crypto docs](https://nodejs.org/api/crypto.html) — Cryptography
