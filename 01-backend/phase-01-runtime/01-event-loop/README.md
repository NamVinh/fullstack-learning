# Phase 1.1 — Event Loop

Event Loop là cơ chế cốt lõi giải thích tại sao Node.js có thể xử lý hàng nghìn kết nối đồng thời dù chỉ có một thread. Đây là kiến thức **quan trọng nhất** trong toàn bộ roadmap Node.js — nếu hiểu Event Loop, bạn sẽ hiểu tại sao code async hoạt động theo cách nó hoạt động.

> "The Event Loop is one of the most critical aspects of Node.js — it explains how Node.js can be asynchronous and have non-blocking I/O." — roadmap.sh

---

## 6 Phases của Event Loop

Event Loop chạy liên tục qua 6 phases theo thứ tự:

```
   ┌──────────────────────────┐
┌─>│   1. timers              │  setTimeout(), setInterval()
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   2. pending callbacks   │  I/O errors từ vòng trước
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   3. idle, prepare       │  internal only
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   4. poll                │  nhận I/O events mới, chạy I/O callbacks
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │   5. check               │  setImmediate()
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
└──┤   6. close callbacks     │  socket.destroy(), etc.
   └──────────────────────────┘
```

### Mô tả từng phase

| Phase | Tên | Chứa gì |
|-------|-----|---------|
| 1 | **timers** | Chạy callbacks của `setTimeout()` và `setInterval()` khi timer đã hết hạn |
| 2 | **pending callbacks** | I/O callbacks bị defer từ vòng loop trước (thường là lỗi TCP) |
| 3 | **idle, prepare** | Chỉ dùng nội bộ bởi Node.js, bạn không thể hook vào |
| 4 | **poll** | Lấy I/O events mới từ OS; nếu queue rỗng sẽ đợi ở đây |
| 5 | **check** | Chạy callbacks của `setImmediate()` |
| 6 | **close callbacks** | Cleanup callbacks như `socket.on('close', ...)` |

---

## process.nextTick() và Microtask Queue

`process.nextTick()` **không phải là một phase** của Event Loop. Nó chạy ngay sau khi operation hiện tại hoàn thành, **trước khi** Event Loop chuyển sang phase tiếp theo.

Thứ tự ưu tiên (từ cao đến thấp):
1. Synchronous code (hiện tại đang chạy)
2. `process.nextTick()` callbacks
3. Promise microtasks (`.then()`, `async/await`)
4. Event Loop phases (timers → poll → check → ...)

### Cảnh báo: nextTick đệ quy gây starvation

```js
// NGUY HIỂM: event loop không bao giờ đến poll phase
function recursiveNextTick() {
  process.nextTick(recursiveNextTick)  // I/O sẽ bị blocked mãi mãi
}
```

---

## Quy tắc setImmediate vs setTimeout

```js
// Trong I/O callback: setImmediate() LUÔN chạy trước setTimeout(0)
fs.readFile('./file', () => {
  setTimeout(() => console.log('timeout'), 0)
  setImmediate(() => console.log('immediate'))
  // Output: immediate → timeout (đảm bảo)
})

// Ngoài I/O callback (main module): thứ tự là NON-DETERMINISTIC
setTimeout(() => console.log('timeout'), 0)
setImmediate(() => console.log('immediate'))
// Output: có thể là timeout → immediate HOẶC immediate → timeout
```

---

## libuv — Engine phía sau Event Loop

libuv là C library cung cấp Event Loop thực sự cho Node.js:

- **OS backends**: epoll (Linux), kqueue (macOS/BSD), IOCP (Windows), event ports (Solaris)
- **Thread pool**: mặc định **4 threads** dùng cho file I/O, DNS lookup, và một số crypto ops
- Tăng kích thước thread pool: `UV_THREADPOOL_SIZE=8 node app.js`
- **Tất cả JavaScript callbacks** chạy trên main thread — thread pool chỉ xử lý work ở C level

---

## Node.js Process Exit

Process chỉ thoát khi có **zero referenced handles** và zero active requests. Timer/server đang chạy sẽ giữ process sống:

```js
const timer = setTimeout(() => {}, 1000000)

// Nếu muốn process thoát dù timer chưa xong:
timer.unref()  // timer không còn giữ process sống
```

---

## Bài tập thực hành

```js
// Bài 1: Đoán output trước khi chạy, sau đó chạy để verify
setTimeout(() => console.log('setTimeout'), 0)
setImmediate(() => console.log('setImmediate'))
process.nextTick(() => console.log('nextTick'))
Promise.resolve().then(() => console.log('Promise'))
console.log('sync')

// Expected: sync → nextTick → Promise → setTimeout/setImmediate (non-deterministic)
```

```js
// Bài 2: nextTick vs setImmediate trong I/O callback
const fs = require('fs')
fs.readFile('./package.json', () => {
  setTimeout(() => console.log('timeout'), 0)
  setImmediate(() => console.log('immediate'))
  process.nextTick(() => console.log('nextTick'))
  Promise.resolve().then(() => console.log('promise'))
})
// Expected: nextTick → promise → immediate → timeout
```

### Bài tập nâng cao

1. **Đo thời gian thực**: Viết benchmark đo delay thực tế của `setTimeout(fn, 1)` — kết quả bao giờ cũng > 1ms, tại sao?
2. **Starvation demo**: Tạo một server đơn giản, thêm `process.nextTick` đệ quy, quan sát server không thể nhận request mới.
3. **UV_THREADPOOL_SIZE experiment**: Chạy 8 DNS lookups song song với thread pool size = 1 vs 8, đo thời gian.
4. **setImmediate use case**: Implement một function `setImmediateAsync(fn)` wrap setImmediate thành Promise.

---

## Resources

- [Node.js docs — Event Loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick) — Official explanation
- [libuv documentation](https://docs.libuv.org/en/v1.x/) — C library behind the loop
- [What the heck is the event loop anyway? (Philip Roberts)](https://www.youtube.com/watch?v=8aGhZQkoFbQ) — Talk nổi tiếng (dành cho browser JS nhưng concept tương tự)
