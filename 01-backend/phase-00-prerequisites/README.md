# Phase 0 — Nền tảng bắt buộc trước khi học Node.js

Trước khi đụng vào Node.js, bạn cần nắm vững những kiến thức nền này. Không có chúng, bạn sẽ liên tục bị mắc kẹt ở những lỗi cơ bản thay vì tập trung học Node.js thực sự.

## Checklist kiến thức cần có

| Kiến thức | Tại sao cần | Mức độ tối thiểu |
|-----------|------------|------------------|
| **JavaScript (ES6+)** | Node.js là JS runtime — mọi code đều là JS | async/await, destructuring, closures, prototypes, arrow functions, spread/rest |
| **Git cơ bản** | Version control trong mọi dự án thực tế | commit, branch, merge, pull request, .gitignore |
| **Terminal / CLI** | Chạy Node, npm, scripts đều qua terminal | cd, ls, mkdir, chmod, pipes, `|`, `>`, `>>` |
| **HTTP protocol** | Node.js chủ yếu build web servers | request/response cycle, methods (GET/POST/PUT/DELETE), status codes (2xx/4xx/5xx), headers, body |

---

## JavaScript ES6+ — Những gì phải nắm chắc

### Async/Await & Promises

```js
// Promise chain
fetch('/api/users')
  .then(res => res.json())
  .then(users => console.log(users))
  .catch(err => console.error(err))

// Async/await (prefer này)
async function getUsers() {
  try {
    const res = await fetch('/api/users')
    const users = await res.json()
    return users
  } catch (err) {
    console.error(err)
  }
}

// Parallel execution
const [users, posts] = await Promise.all([getUsers(), getPosts()])
```

### Destructuring & Spread

```js
// Object destructuring
const { name, age, role = 'user' } = user

// Array destructuring
const [first, second, ...rest] = items

// Spread
const newUser = { ...user, updatedAt: new Date() }
const allItems = [...arr1, ...arr2]
```

### Closures

```js
function makeCounter() {
  let count = 0
  return {
    increment: () => ++count,
    get: () => count
  }
}

const counter = makeCounter()
counter.increment()  // 1
counter.get()        // 1
```

### Modules (biết cả hai dạng vì Node.js dùng cả hai)

```js
// CommonJS (Node.js default)
const path = require('path')
module.exports = { myFunction }

// ES Modules
import { readFile } from 'fs/promises'
export const myFunction = () => {}
```

---

## HTTP Protocol — Những gì cần hiểu

### Request structure

```
GET /api/users?page=1&limit=10 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGc...
Content-Type: application/json
```

### Response structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{"users": [...]}
```

### Status codes quan trọng

| Code | Ý nghĩa |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request (lỗi input) |
| 401 | Unauthorized (chưa đăng nhập) |
| 403 | Forbidden (không có quyền) |
| 404 | Not Found |
| 409 | Conflict (ví dụ: email đã tồn tại) |
| 422 | Unprocessable Entity (validation fail) |
| 500 | Internal Server Error |

---

## Terminal / CLI

```bash
# Navigation
cd /path/to/folder
ls -la             # list files với details
pwd                # print working directory

# File operations
mkdir my-project
touch index.js
cp src.js dest.js
mv old.js new.js
rm file.js

# Pipes & redirection
node script.js | grep "error"
node script.js > output.log 2>&1

# Permissions
chmod +x script.sh

# Find & search
find . -name "*.js"
grep -r "TODO" ./src
```

---

## Bài tập

1. **JS Review**: Viết một hàm `fetchWithRetry(url, retries = 3)` dùng async/await, tự động retry khi request thất bại, có delay giữa các lần retry.

2. **HTTP Client thủ công**: Dùng `curl` hoặc browser DevTools, gọi `https://jsonplaceholder.typicode.com/users` và phân tích: request headers, response headers, status code, body format.

3. **Git workflow**: Tạo một repo mới, tạo file `index.js` với nội dung bất kỳ, commit, tạo branch `feature/hello`, thêm code mới, merge về `main`.

4. **Terminal script**: Viết bash script tạo cấu trúc thư mục project Node.js chuẩn (`src/`, `tests/`, `package.json`, `.gitignore`).

---

## Resources

- [javascript.info](https://javascript.info) — JavaScript tutorial đầy đủ nhất, miễn phí
- [MDN Web Docs — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) — HTTP protocol reference
- [The Odin Project — Command Line](https://www.theodinproject.com/lessons/foundations-command-line-basics) — Terminal cơ bản
