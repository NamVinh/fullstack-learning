# Phase 1.5 — Buffers

Buffer là cách Node.js biểu diễn **binary data** — dữ liệu thô dưới dạng bytes. Trong browser, JavaScript chủ yếu làm việc với strings và numbers. Nhưng ở server-side, bạn thường xuyên phải xử lý: file contents dưới dạng bytes, network packets, image data, encrypted data, và protocol serialization. Buffer là công cụ cho tất cả những trường hợp này.

Buffer là subclass của `Uint8Array` — một TypedArray của JavaScript. Mỗi element là một byte (0–255).

---

## Tạo Buffer

```js
// Buffer.alloc — KHUYÊN DÙNG: zero-filled, safe
const buf1 = Buffer.alloc(10)         // 10 bytes, tất cả là 0x00
const buf2 = Buffer.alloc(10, 0xff)   // 10 bytes, tất cả là 0xFF

// Buffer.allocUnsafe — NHANH HƠN nhưng chứa garbage data cũ (memory reuse)
const buf3 = Buffer.allocUnsafe(10)   // 10 bytes, nội dung không xác định

// Buffer.from — từ dữ liệu có sẵn
const buf4 = Buffer.from('Hello, World!')           // từ string (default UTF-8)
const buf5 = Buffer.from('Hello', 'utf8')           // explicit encoding
const buf6 = Buffer.from([0x48, 0x65, 0x6c, 0x6c, 0x6f])  // từ byte array
const buf7 = Buffer.from('SGVsbG8=', 'base64')      // từ base64 string
const buf8 = Buffer.from(anotherBuffer)              // copy từ buffer khác
```

---

## Encodings

```js
const buf = Buffer.from('Hello')

// Chuyển buffer sang string
buf.toString('utf8')      // 'Hello'     — default
buf.toString('base64')    // 'SGVsbG8='
buf.toString('hex')       // '48656c6c6f'
buf.toString('ascii')     // 'Hello'
buf.toString('binary')    // Latin-1 (legacy)
buf.toString('base64url') // URL-safe base64 (không có +, /, =)

// Chuyển string sang buffer rồi sang encoding khác
const hexString = Buffer.from('Hello World').toString('hex')
// '48656c6c6f20576f726c64'

const b64 = Buffer.from('Hello World').toString('base64')
// 'SGVsbG8gV29ybGQ='
```

---

## Đọc và Ghi dữ liệu

```js
const buf = Buffer.alloc(16)

// Ghi số vào buffer (quan trọng cho binary protocols)
buf.writeUInt8(255, 0)           // 1 byte unsigned int tại offset 0
buf.writeUInt16BE(1000, 1)       // 2 bytes, big-endian tại offset 1
buf.writeUInt32LE(99999, 3)      // 4 bytes, little-endian tại offset 3
buf.writeDoubleBE(3.14, 7)       // 8 bytes, double precision tại offset 7

// Đọc số từ buffer
const val1 = buf.readUInt8(0)         // 255
const val2 = buf.readUInt16BE(1)      // 1000
const val3 = buf.readUInt32LE(3)      // 99999
const val4 = buf.readDoubleBE(7)      // 3.14

// Truy cập trực tiếp như array
console.log(buf[0])   // 255 (byte đầu tiên)
buf[0] = 128          // set byte value
```

---

## Thao tác với Buffer

```js
const buf1 = Buffer.from('Hello')
const buf2 = Buffer.from(' World')

// Nối buffers
const combined = Buffer.concat([buf1, buf2])
console.log(combined.toString())  // 'Hello World'

// Buffer.concat với length hint (optimize)
const result = Buffer.concat([buf1, buf2], buf1.length + buf2.length)

// Copy
const source = Buffer.from('Hello World')
const target = Buffer.alloc(5)
source.copy(target, 0, 0, 5)   // copy bytes 0-4 từ source vào target
console.log(target.toString())  // 'Hello'

// Slice (share memory với original!)
const slice = source.slice(0, 5)
console.log(slice.toString())   // 'Hello'
// CẢNH BÁO: thay đổi slice sẽ thay đổi source!

// subarray (safer, explicit)
const sub = source.subarray(6, 11)
console.log(sub.toString())  // 'World'

// So sánh buffers
const a = Buffer.from('abc')
const b = Buffer.from('abc')
console.log(a.equals(b))    // true (so sánh nội dung)
console.log(a === b)        // false (so sánh reference)

// Kiểm tra
buf.includes('Hello')           // true
buf.indexOf('World')            // 6
buf.length                      // số bytes
Buffer.isBuffer(buf)            // true
```

---

## Buffer trong thực tế

```js
// 1. Đọc file binary (image, PDF, etc.)
const { readFile } = require('fs/promises')
const imageBuffer = await readFile('./photo.jpg')  // trả về Buffer
const base64Image = imageBuffer.toString('base64')
// Dùng trong HTML: `<img src="data:image/jpeg;base64,${base64Image}">`

// 2. Tạo binary protocol message
function createMessage(type, payload) {
  const typeBuffer = Buffer.alloc(1)
  typeBuffer.writeUInt8(type, 0)

  const payloadBuffer = Buffer.from(JSON.stringify(payload))
  const lengthBuffer = Buffer.alloc(4)
  lengthBuffer.writeUInt32BE(payloadBuffer.length, 0)

  return Buffer.concat([typeBuffer, lengthBuffer, payloadBuffer])
}

// 3. Crypto với Buffer
const crypto = require('crypto')
const key = crypto.randomBytes(32)        // 256-bit key as Buffer
const iv = crypto.randomBytes(16)         // 128-bit IV

const cipher = crypto.createCipheriv('aes-256-cbc', key, iv)
let encrypted = cipher.update('secret data', 'utf8', 'hex')
encrypted += cipher.final('hex')

// 4. Stream chunk processing
stream.on('data', (chunk) => {
  // chunk là Buffer
  const lines = chunk.toString('utf8').split('\n')
  lines.forEach(line => processLine(line))
})
```

---

## Bài tập thực hành

1. **Base64 encode/decode**: Implement `encodeBase64(string)` và `decodeBase64(b64string)` thủ công dùng Buffer. Verify bằng cách compare với `btoa`/`atob` (nếu có).

2. **Binary protocol parser**: Tạo simple binary message format: [1 byte type][4 bytes length][payload bytes]. Viết `serialize(type, data)` và `deserialize(buffer)`.

3. **File checksum**: Đọc một file lớn bằng stream, tính SHA-256 checksum của từng chunk và tổng thể, so sánh với `sha256sum` command của terminal.

4. **Image to data URL**: Đọc một ảnh JPEG/PNG, convert sang base64, tạo HTML file với `<img>` tag embed ảnh đó.

---

## Resources

- [Node.js Buffer docs](https://nodejs.org/api/buffer.html) — Official API reference
- [Understanding Buffers in Node.js](https://nodejs.org/en/docs/guides/buffer-api-explanation) — Official guide
- [MDN — ArrayBuffer and TypedArrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays) — Nền tảng lý thuyết
