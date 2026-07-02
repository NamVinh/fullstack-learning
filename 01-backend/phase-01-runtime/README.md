# Phase 1 — Node.js Runtime (Nền tảng cốt lõi)

> **Đây là phần quan trọng nhất.** Nếu bỏ qua, bạn sẽ dùng Node như một black box — build được app nhưng không hiểu tại sao nó hoạt động hay lý do tại sao nó chậm/crash.

Node.js là một JavaScript runtime chạy ngoài browser, sử dụng **V8 engine** của Google Chrome. Điểm đặc biệt là Node.js là single-threaded nhưng xử lý concurrency cực kỳ hiệu quả thông qua **Event Loop + libuv**, cho phép xử lý hàng nghìn kết nối đồng thời mà không tạo thread mới cho mỗi request.

```
┌─────────────────────────────────────────────┐
│              Node.js Process                │
│  ┌─────────────┐     ┌────────────────────┐ │
│  │  V8 Engine  │     │  libuv (C library) │ │
│  │  (JS code)  │     │  Thread pool (×4)  │ │
│  │             │     │  epoll/kqueue/IOCP │ │
│  └─────────────┘     └────────────────────┘ │
└─────────────────────────────────────────────┘
```

---

## Các chủ đề trong Phase 1

| # | Chủ đề | Mô tả |
|---|--------|-------|
| 1.1 | [Event Loop](./01-event-loop/README.md) | Cơ chế xử lý bất đồng bộ cốt lõi của Node.js — 6 phases, nextTick, libuv |
| 1.2 | [Modules System](./02-modules/README.md) | CommonJS vs ES Modules, module resolution, caching |
| 1.3 | [Built-in Modules](./03-built-in-modules/README.md) | fs, path, http, crypto, os, process — các module có sẵn quan trọng nhất |
| 1.4 | [Streams](./04-streams/README.md) | Xử lý dữ liệu lớn không load vào memory, backpressure, pipe |
| 1.5 | [Buffers](./05-buffers/README.md) | Biểu diễn binary data, chuyển đổi encoding |
| 1.6 | [Events & EventEmitter](./06-events-eventemitter/README.md) | Observer pattern trong Node.js, custom events |
| 1.7 | [Child Process](./07-child-process/README.md) | exec, spawn, fork — chạy processes con |
| 1.8 | [Worker Threads](./08-worker-threads/README.md) | CPU-intensive tasks không block event loop |
| 1.9 | [Cluster](./09-cluster/README.md) | Scale Node.js server lên nhiều CPU cores |

---

## Thứ tự học khuyến nghị

Học theo thứ tự 1.1 → 1.2 → 1.3 → 1.4 → 1.5 → 1.6 → 1.7 → 1.8 → 1.9. Event Loop là nền tảng cho mọi thứ còn lại — hiểu rõ nó trước khi tiếp tục.

## Điểm kiểm tra

Sau Phase 1, bạn phải trả lời được:
- Event Loop có bao nhiêu phases? `process.nextTick()` chạy ở phase nào?
- Khác nhau giữa `require()` (CJS) và `import` (ESM)?
- Tại sao phải dùng Streams thay vì `readFileSync` cho file lớn?
- Khi nào dùng `child_process.fork()` vs `worker_threads`?

## Resources

- [nodejs.org/docs](https://nodejs.org/docs/latest/api/) — Official API reference
- [Node.js Design Patterns — Mario Casciaro](https://www.nodejsdesignpatterns.com) — Sách hay nhất về Node.js internals
- [learnyounode](https://github.com/workshopper/learnyounode) — Interactive CLI workshop (13 exercises)
