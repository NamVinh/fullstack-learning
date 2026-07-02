# Phase 1.2 — Modules System

Module system cho phép bạn tổ chức code thành các file riêng biệt, tái sử dụng code, và quản lý dependencies. Node.js hỗ trợ **hai** module systems: CommonJS (lịch sử) và ES Modules (hiện đại). Việc hiểu cả hai và biết khi nào dùng cái nào là kỹ năng thiết yếu.

---

## CommonJS (CJS) — Default của Node.js

CommonJS là module system gốc của Node.js, dùng `require()` và `module.exports`.

```js
// math.js — export
function add(a, b) { return a + b }
function multiply(a, b) { return a * b }

// Cách 1: export object
module.exports = { add, multiply }

// Cách 2: shorthand export từng property
exports.add = add
exports.multiply = multiply
// CẢNH BÁO: KHÔNG dùng exports = { add } — điều này thay thế reference, không hoạt động!
```

```js
// index.js — import
const { add, multiply } = require('./math')
const math = require('./math')  // hoặc import cả object

console.log(add(2, 3))        // 5
console.log(math.multiply(4, 5))  // 20
```

### Cách module.exports và exports hoạt động

```js
// Hiểu tại sao exports = {} không work:
// Ban đầu: exports === module.exports (cùng reference)
// Khi bạn làm: exports = { foo }
// exports trỏ đến object mới, module.exports vẫn là {}
// require() trả về module.exports, không phải exports

// ĐÚNG:
module.exports = { foo }      // replace module.exports
exports.foo = foo             // add property to module.exports

// SAI:
exports = { foo }             // tạo reference mới, module.exports không thay đổi
```

---

## ES Modules (ESM) — Modern Standard

ESM là module system chuẩn của JavaScript (browser + Node.js), dùng `import`/`export`.

**Kích hoạt ESM trong Node.js:**
- Thêm `"type": "module"` vào `package.json`, hoặc
- Đặt tên file với extension `.mjs`

```js
// math.mjs (hoặc math.js với "type": "module" trong package.json)

// Named exports
export function add(a, b) { return a + b }
export const PI = 3.14159

// Default export
export default class Calculator {
  add(a, b) { return a + b }
}
```

```js
// index.mjs

// Named imports — PHẢI có .js extension trong ESM
import { add, PI } from './math.js'

// Default import
import Calculator from './math.js'

// Import tất cả dưới namespace
import * as MathUtils from './math.js'

// Dynamic import (lazy loading)
const { add } = await import('./math.js')
```

---

## So sánh CJS vs ESM

| Tiêu chí | CommonJS | ES Modules |
|---------|----------|------------|
| Cú pháp | `require()` / `module.exports` | `import` / `export` |
| Thời điểm load | Runtime (dynamic) | Parse time (static) |
| Top-level await | Không | Có |
| `__dirname` / `__filename` | Có sẵn | Không có (dùng `import.meta.url`) |
| Tree shaking | Khó | Tốt (bundlers tận dụng) |
| Mặc định trong Node.js | Có | Cần config |

```js
// ESM equivalent của __dirname
import { fileURLToPath } from 'url'
import { dirname } from 'path'

const __filename = fileURLToPath(import.meta.url)
const __dirname = dirname(__filename)
```

---

## Module Resolution Order (require)

Khi bạn gọi `require('something')`, Node.js tìm theo thứ tự:

1. **Core modules** — `fs`, `path`, `http`, etc. (luôn ưu tiên trước)
2. **File paths** — nếu bắt đầu bằng `./`, `../`, hoặc `/`
   - Thử: `./something`, `./something.js`, `./something.json`, `./something.node`
   - Thử thư mục: `./something/index.js`
3. **node_modules** — tìm từ thư mục hiện tại lên root
   - `./node_modules/something`
   - `../node_modules/something`
   - ...

### Module caching

```js
// Module chỉ được execute MỘT LẦN dù require() nhiều lần
// Lần sau trả về cached exports
const a = require('./config')  // execute config.js
const b = require('./config')  // trả về cached result
console.log(a === b)  // true

// Clear cache (hiếm khi dùng, thường trong testing)
delete require.cache[require.resolve('./config')]
```

---

## Circular Dependencies

CJS xử lý circular deps bằng cách trả về **partial exports** tại thời điểm require:

```js
// a.js
const b = require('./b')
module.exports = { name: 'a', bName: b.name }

// b.js
const a = require('./a')
// a.name sẽ là undefined tại thời điểm này (circular!)
module.exports = { name: 'b', aName: a.name }
```

Cách tránh: restructure code để không có circular deps, hoặc dùng lazy require bên trong function.

---

## Bài tập thực hành

1. **CJS practice**: Tạo `math.js` với các hàm `add`, `subtract`, `multiply`, `divide`. Tạo `logger.js` với hàm `log(level, message)`. Tạo `index.js` import cả hai và sử dụng.

2. **ESM conversion**: Convert project trên sang ES Modules — thêm `"type": "module"` vào `package.json`, sửa tất cả `require()` thành `import`.

3. **Module caching experiment**: Tạo một module `counter.js` với state (biến đếm). Require nó nhiều lần từ các files khác nhau, verify rằng state được share.

4. **Dynamic import**: Implement một plugin system đơn giản — đọc thư mục `plugins/`, dynamic import từng file `.js`, gọi hàm `init()` của mỗi plugin.

---

## Resources

- [Node.js docs — Modules: CommonJS](https://nodejs.org/api/modules.html) — Official CJS documentation
- [Node.js docs — ECMAScript Modules](https://nodejs.org/api/esm.html) — Official ESM documentation
- [CommonJS vs ES Modules — Node.js blog](https://nodejs.org/en/blog/release/v12.0.0) — Context lịch sử
