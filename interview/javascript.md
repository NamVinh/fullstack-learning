# JavaScript — Interview Questions (Junior → Senior)

> Tài liệu ôn tập phỏng vấn JavaScript từ cấp độ Junior đến Senior.
> Câu hỏi song ngữ (EN/VI), trả lời chi tiết bằng tiếng Việt, code ví dụ bằng tiếng Anh.

---

## Table of Contents

1. [var / let / const, Hoisting, TDZ](#1-var--let--const-hoisting-tdz)
2. [Closures](#2-closures)
3. [Event Loop](#3-event-loop)
4. [Prototype Chain & Prototypal Inheritance](#4-prototype-chain--prototypal-inheritance)
5. [this Binding](#5-this-binding)
6. [Promise API](#6-promise-api)
7. [async / await & Error Handling](#7-async--await--error-handling)
8. [Debounce vs Throttle](#8-debounce-vs-throttle)
9. [ES6+ Features](#9-es6-features)
10. [Scope Chain & Lexical Scope](#10-scope-chain--lexical-scope)
11. [Memory Leaks](#11-memory-leaks)
12. [Generators & Iterators](#12-generators--iterators)
13. [Proxy & Reflect](#13-proxy--reflect)
14. [Map vs WeakMap, Set vs WeakSet](#14-map-vs-weakmap-set-vs-weakset)
15. [Immutability Patterns](#15-immutability-patterns)

---

## 1. var / let / const, Hoisting, TDZ

### Q1 — [Junior]

**EN:** What is the difference between `var`, `let`, and `const`? Explain hoisting behavior for each.

**VI:** Sự khác biệt giữa `var`, `let`, và `const` là gì? Giải thích hành vi hoisting của từng loại.

**Trả lời:**

`var`, `let`, `const` đều được **hoist** (kéo lên đầu scope), nhưng hành vi khác nhau:

| Đặc điểm | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function scope | Block scope | Block scope |
| Hoisting | Có (khởi tạo = `undefined`) | Có (nhưng **không** khởi tạo — TDZ) | Có (nhưng **không** khởi tạo — TDZ) |
| Reassign | Được | Được | Không được |
| Redeclare | Được | Không được | Không được |
| Global object property | Có (`window.x`) | Không | Không |

**`var` hoisting:**

```js
console.log(name); // undefined (không throw error)
var name = "Alice";
// Trình dịch hiểu là:
// var name; (hoisted, init = undefined)
// console.log(name); => undefined
// name = "Alice";
```

**`let` / `const` — Temporal Dead Zone (TDZ):**

```js
console.log(age); // ReferenceError: Cannot access 'age' before initialization
let age = 25;
```

`let` và `const` được hoist nhưng **không được khởi tạo**. Khoảng thời gian từ đầu block đến khai báo gọi là **Temporal Dead Zone (TDZ)**. Truy cập trong TDZ ném `ReferenceError`.

```js
{
  // -- TDZ bắt đầu cho `x` --
  console.log(x); // ReferenceError
  let x = 10;    // TDZ kết thúc
  console.log(x); // 10
}
```

**`const` — bắt buộc gán ngay:**

```js
const PI = 3.14;
PI = 3.15; // TypeError: Assignment to constant variable

const user = { name: "Bob" };
user.name = "Charlie"; // OK — thay đổi property của object được phép
user = {};             // TypeError — không thể gán lại reference
```

**Lưu ý thực tế:** Ưu tiên `const` > `let` > `var`. Tránh dùng `var` trong code hiện đại do function scope gây ra nhiều bug khó debug.

---

### Q2 — [Mid]

**EN:** What is the Temporal Dead Zone (TDZ)? Why does it exist?

**VI:** Temporal Dead Zone (TDZ) là gì? Tại sao nó tồn tại?

**Trả lời:**

**TDZ (Temporal Dead Zone)** là khoảng thời gian trong block scope từ khi block bắt đầu thực thi đến khi khai báo `let`/`const` được xử lý. Trong khoảng này, biến **tồn tại** trong scope nhưng **chưa được khởi tạo** — truy cập sẽ ném `ReferenceError`.

```js
function example() {
  // TDZ for `value` starts here
  typeof value; // ReferenceError (khác với var — typeof var === "undefined")

  let value = 42; // TDZ ends here
  console.log(value); // 42
}
```

**Tại sao TDZ tồn tại?**

1. **Phát hiện lỗi sớm hơn:** Buộc lập trình viên khai báo biến trước khi dùng — tránh bug do hoisting của `var`.
2. **Hành vi nhất quán:** `const` không thể có giá trị `undefined` tạm thời (vì `const` không thể reassign).
3. **Thiết kế ngôn ngữ tốt hơn:** Khuyến khích viết code có thứ tự rõ ràng.

```js
// Bug điển hình với var (không có TDZ)
function bugWithVar() {
  console.log(result); // undefined — không báo lỗi, khó debug
  var result = calculate();
  return result;
}

// let buộc bạn nhận ra lỗi ngay
function safeWithLet() {
  console.log(result); // ReferenceError — lỗi rõ ràng
  let result = calculate();
  return result;
}
```

---

## 2. Closures

### Q3 — [Junior]

**EN:** What is a closure? Give a practical example.

**VI:** Closure là gì? Cho ví dụ thực tế.

**Trả lời:**

**Closure** là khả năng của một function **ghi nhớ và truy cập** lexical scope (scope nơi nó được định nghĩa) ngay cả khi function đó thực thi **bên ngoài** scope ban đầu.

Nói đơn giản: function "đóng gói" các biến xung quanh nó, giữ chúng sống ngay cả khi outer function đã chạy xong.

```js
function makeCounter() {
  let count = 0; // biến này bị "đóng gói" bởi closure

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount() { return count; }
  };
}

const counter = makeCounter();
counter.increment();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.getCount()); // 2

// `count` không thể truy cập trực tiếp từ bên ngoài
console.log(counter.count); // undefined
```

**Ứng dụng thực tế:**

```js
// 1. Module pattern — private state
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private

  return {
    deposit(amount) {
      if (amount > 0) balance += amount;
    },
    withdraw(amount) {
      if (amount <= balance) balance -= amount;
      else console.log("Insufficient funds");
    },
    getBalance() { return balance; }
  };
}

// 2. Function factories
function multiply(factor) {
  return (number) => number * factor; // closure over `factor`
}

const double = multiply(2);
const triple = multiply(3);
console.log(double(5));  // 10
console.log(triple(5));  // 15

// 3. Memoization
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

---

### Q4 — [Mid]

**EN:** What is the classic closure-in-loop pitfall with `var`? How do you fix it with `let`, IIFE, and `bind`?

**VI:** Bẫy closure trong vòng lặp với `var` là gì? Cách khắc phục bằng `let`, IIFE, và `bind`?

**Trả lời:**

Đây là một trong những câu hỏi phỏng vấn JavaScript kinh điển nhất.

**Vấn đề:**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i); // Kỳ vọng: 0, 1, 2
  }, 1000);
}
// Thực tế in ra: 3, 3, 3
```

**Tại sao?** `var` có function scope, không phải block scope. Tất cả 3 callback `setTimeout` **chia sẻ cùng một biến `i`**. Khi callback chạy (sau 1 giây), vòng lặp đã kết thúc, `i` đã bằng `3`.

**Cách khắc phục:**

**Cách 1 — Dùng `let` (đơn giản nhất):**

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i); // 0, 1, 2
  }, 1000);
}
// `let` tạo một binding mới cho mỗi iteration
```

**Cách 2 — IIFE (Immediately Invoked Function Expression):**

```js
for (var i = 0; i < 3; i++) {
  (function(capturedI) {
    setTimeout(() => {
      console.log(capturedI); // 0, 1, 2
    }, 1000);
  })(i); // truyền i vào ngay lập tức, tạo closure mới với giá trị riêng
}
```

**Cách 3 — Dùng `.bind()`:**

```js
function logNumber(n) {
  console.log(n);
}

for (var i = 0; i < 3; i++) {
  setTimeout(logNumber.bind(null, i), 1000); // bind tạo copy của i
  // 0, 1, 2
}
```

**Cách 4 — Dùng mảng + forEach:**

```js
[0, 1, 2].forEach(i => {
  setTimeout(() => console.log(i), 1000); // 0, 1, 2
  // forEach callback tạo scope mới cho mỗi lần gọi
});
```

---

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Can you explain what a closure is in JavaScript?"**

**Model Answer:**
"Sure! A closure is when a function 'remembers' the variables from the scope where it was defined, even after that outer scope has finished executing. The classic example is a counter factory — you call `makeCounter()`, the outer function returns, but the returned inner functions still have access to the `count` variable. That variable is kept alive in memory because the inner functions close over it. In practice, closures are used everywhere: for data privacy, for function factories like `multiply(2)` returning a `double` function, and for memoization."

**Trả lời (Tiếng Việt):**
"Closure là khi một function 'nhớ' các biến từ scope nơi nó được định nghĩa, ngay cả sau khi outer scope đó đã chạy xong. Ví dụ kinh điển là counter factory — bạn gọi `makeCounter()`, outer function trả về, nhưng các inner function được trả về vẫn có thể truy cập biến `count`. Biến đó được giữ sống trong bộ nhớ vì các inner function 'đóng gói' nó. Trong thực tế, closure được dùng ở khắp nơi: để tạo data privacy, tạo function factory như `multiply(2)` trả về hàm `double`, và để memoization."

---

**Q2 (Junior/Mid): "What is the closure-in-loop bug with `var`, and how do you fix it?"**

**Model Answer:**
"This is a classic gotcha. If you write a `for` loop with `var` and put a `setTimeout` inside, all the callbacks share the same `i` variable because `var` is function-scoped — not block-scoped. By the time the callbacks run, the loop has finished and `i` is already at its final value, so you get that printed every time instead of `0, 1, 2`.

The easiest fix is to just use `let` instead of `var`, because `let` creates a new binding for each loop iteration. Before `let` existed, the classic fix was an IIFE — you immediately invoke a function with `i` as an argument, creating a new scope that captures the current value:

```js
for (var i = 0; i < 3; i++) {
  (function(capturedI) {
    setTimeout(() => console.log(capturedI), 1000);
  })(i);
}
```

Today, `let` is the idiomatic solution."

**Trả lời (Tiếng Việt):**
"Đây là lỗi cổ điển. Khi bạn viết vòng lặp `for` với `var` và đặt `setTimeout` bên trong, tất cả callback chia sẻ cùng một biến `i` vì `var` có function scope — không phải block scope. Đến lúc callback chạy, vòng lặp đã kết thúc và `i` đã bằng giá trị cuối cùng, nên bạn thấy giá trị đó được in ra mọi lúc thay vì `0, 1, 2`.

Cách sửa đơn giản nhất là dùng `let` thay vì `var`, vì `let` tạo một binding mới cho mỗi lần lặp. Trước khi có `let`, cách cổ điển là dùng IIFE — bạn gọi ngay một function với `i` làm argument, tạo scope mới capture giá trị hiện tại:

```js
for (var i = 0; i < 3; i++) {
  (function(capturedI) {
    setTimeout(() => console.log(capturedI), 1000);
  })(i);
}
```

Ngày nay, `let` là giải pháp chuẩn."

---

**Q3 (Mid): "What are some practical use cases for closures beyond just counters?"**

**Model Answer:**
"Beyond counters, closures power several real patterns. First, the module pattern — before ES modules existed, developers used closures to create private state. You expose only what you want public and keep the rest hidden inside the closure.

Second, function factories — you create a higher-order function that captures configuration and returns a specialized function:

```js
function multiply(factor) {
  return (n) => n * factor; // closes over `factor`
}
const double = multiply(2); // `factor` is 2, forever
```

Third, memoization — caching expensive computation results in a Map that the returned function closes over. Fourth, event handlers that need to remember context — like a button that closes over the `user` object that was in scope when the handler was created. The key insight is: closures let you associate persistent state with a function without using global variables or classes."

**Trả lời (Tiếng Việt):**
"Ngoài counter, closure còn là nền tảng của nhiều pattern thực tế. Đầu tiên là module pattern — trước khi có ES modules, developer dùng closure để tạo private state. Bạn chỉ expose những gì muốn public và giữ phần còn lại ẩn trong closure.

Thứ hai là function factory — bạn tạo một higher-order function capture cấu hình và trả về một function chuyên dụng:

```js
function multiply(factor) {
  return (n) => n * factor; // closes over `factor`
}
const double = multiply(2); // `factor` là 2, mãi mãi
```

Thứ ba là memoization — cache kết quả tính toán tốn kém trong một Map mà function trả về đóng gói. Thứ tư là event handler cần nhớ context — ví dụ một button đóng gói object `user` đang trong scope khi handler được tạo. Điểm mấu chốt là: closure cho phép bạn kết hợp state bền vững với function mà không cần dùng biến global hay class."

---

**Q4 (Senior): "What are the memory implications of closures? When can they cause memory leaks?"**

**Model Answer:**
"Closures keep their entire lexical environment alive as long as the closure itself is reachable. That's normally fine, but it becomes a problem in a few scenarios.

The most common one is event listeners that close over large objects. If you attach a listener that captures a big data structure and never remove that listener, the data structure can never be garbage collected — even if nothing else references it.

Another pattern is accidentally closing over more than you intend. The entire scope chain is captured, not just the specific variable you use. So if you have a closure deep in a module that only uses one small variable, it still holds a reference to everything in its outer scopes.

The fix is to be explicit: remove event listeners when components unmount, use `WeakMap` or `WeakRef` when you want a reference that doesn't prevent GC, and avoid capturing large objects in long-lived closures. In React, this shows up as stale closure bugs — a `useEffect` captures an old value of a prop or state because the closure was created when that value was set."

**Trả lời (Tiếng Việt):**
"Closure giữ toàn bộ lexical environment sống chừng nào closure đó còn reachable. Thường thì không sao, nhưng có vài tình huống thành vấn đề.

Phổ biến nhất là event listener đóng gói object lớn. Nếu bạn attach một listener capture một data structure lớn và không bao giờ remove listener đó, data structure đó không bao giờ được garbage collect — dù không có gì khác reference nó.

Một pattern khác là vô tình đóng gói nhiều hơn mức cần. Toàn bộ scope chain đều bị capture, không chỉ biến bạn đang dùng. Nên nếu bạn có closure sâu trong một module chỉ dùng một biến nhỏ, nó vẫn giữ reference đến tất cả mọi thứ trong outer scope.

Cách sửa là phải rõ ràng: remove event listener khi component unmount, dùng `WeakMap` hoặc `WeakRef` khi bạn muốn reference không ngăn GC, và tránh capture object lớn trong long-lived closure. Trong React, vấn đề này xuất hiện dưới dạng stale closure bug — một `useEffect` capture giá trị cũ của prop hoặc state vì closure được tạo khi giá trị đó còn đang được set."

---

## 3. Event Loop

### Q5 — [Mid]

**EN:** Explain the JavaScript Event Loop. What is the difference between the microtask queue and the macrotask queue? Predict the output of the following code.

**VI:** Giải thích Event Loop trong JavaScript. Microtask queue và Macrotask queue khác nhau như thế nào? Dự đoán output của đoạn code sau.

**Trả lời:**

**Kiến trúc Event Loop:**

JavaScript là single-threaded — chỉ một việc chạy tại một thời điểm. Event Loop là cơ chế cho phép JS xử lý bất đồng bộ mà không block thread.

```
   ┌─────────────────────┐
   │     Call Stack      │  ← JS thực thi code đồng bộ ở đây
   └────────┬────────────┘
            │ (khi stack rỗng)
            ▼
   ┌─────────────────────┐
   │   Microtask Queue   │  ← Promise.then, queueMicrotask, MutationObserver
   └────────┬────────────┘
            │ (khi microtask queue rỗng)
            ▼
   ┌─────────────────────┐
   │   Macrotask Queue   │  ← setTimeout, setInterval, I/O, setImmediate
   └─────────────────────┘
```

**Quy tắc thực thi:**

1. Chạy hết code đồng bộ (Call Stack)
2. **Drain hết Microtask Queue** (bao gồm microtask mới được thêm vào trong quá trình drain)
3. Lấy **một** macrotask từ Macrotask Queue, thực thi
4. Quay lại bước 2

**Microtask Queue:** `Promise.then/catch/finally`, `queueMicrotask()`, `MutationObserver`, `async/await`

**Macrotask Queue:** `setTimeout`, `setInterval`, `setImmediate` (Node.js), I/O callbacks, UI rendering

---

**Quiz — Dự đoán output:**

```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve()
  .then(() => console.log("3"))
  .then(() => console.log("4"));

queueMicrotask(() => console.log("5"));

console.log("6");
```

**Output:** `1`, `6`, `3`, `5`, `4`, `2`

**Giải thích từng bước:**

```
Call Stack (đồng bộ):
  - console.log("1")          → in "1"
  - setTimeout(cb, 0)         → cb vào Macrotask Queue
  - Promise.resolve().then()  → cb "3" vào Microtask Queue
  - queueMicrotask()          → cb "5" vào Microtask Queue
  - console.log("6")          → in "6"

Stack rỗng → drain Microtask Queue:
  - cb "3" chạy → in "3", đẩy cb "4" vào Microtask Queue
  - cb "5" chạy → in "5"
  - cb "4" chạy → in "4" (được thêm vào khi drain, vẫn được xử lý)

Microtask Queue rỗng → lấy 1 Macrotask:
  - cb "2" chạy → in "2"

Final: 1, 6, 3, 5, 4, 2
```

---

**Quiz nâng cao:**

```js
async function main() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
  await new Promise(resolve => setTimeout(resolve, 0));
  console.log("C");
}

console.log("start");
main();
console.log("end");
```

**Output:** `start`, `A`, `end`, `B`, `C`

**Giải thích:**

- `"start"` — đồng bộ
- `main()` được gọi: in `"A"`, gặp `await Promise.resolve()` → suspend, đẩy phần sau vào microtask
- `"end"` — đồng bộ, stack chạy tiếp
- Microtask: in `"B"`, gặp `await new Promise(resolve => setTimeout(...))` → setTimeout vào macrotask
- Macrotask: setTimeout resolve promise → `"C"` vào microtask, chạy → in `"C"`

---

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Can you explain what the JavaScript Event Loop is?"**

**Model Answer:**
"Sure! JavaScript is single-threaded, meaning it can only do one thing at a time. The Event Loop is the mechanism that lets it handle asynchronous operations without blocking.

Think of it like a restaurant: the chef (Call Stack) can only cook one dish at a time. When a customer orders something that takes a while — say, a web request — the kitchen hands it off to external staff (Web APIs) and keeps taking other orders. When that long-running task finishes, it puts its callback into a queue. The Event Loop is the manager who constantly checks: 'Is the chef free? Is there anything in the queue?' — and if so, hands the next callback to the chef.

The key detail is that there are two queues: the microtask queue (Promises, `queueMicrotask`) and the macrotask queue (`setTimeout`, `setInterval`, I/O). After each task, the Event Loop drains the entire microtask queue before picking up the next macrotask."

**Trả lời (Tiếng Việt):**
"JavaScript là single-threaded, có nghĩa là nó chỉ làm một việc tại một thời điểm. Event Loop là cơ chế cho phép nó xử lý bất đồng bộ mà không bị block.

Hãy hình dung như một nhà hàng: đầu bếp (Call Stack) chỉ nấu một món một lúc. Khi khách gọi món mất nhiều thời gian — ví dụ web request — bếp giao cho nhân viên bên ngoài (Web APIs) và tiếp tục nhận order. Khi task lâu đó xong, nó đẩy callback vào queue. Event Loop là người quản lý liên tục kiểm tra: 'Đầu bếp rảnh chưa? Queue có gì không?' — và nếu có, đưa callback tiếp theo cho đầu bếp.

Chi tiết quan trọng là có hai queue: microtask queue (Promises, `queueMicrotask`) và macrotask queue (`setTimeout`, `setInterval`, I/O). Sau mỗi task, Event Loop drain toàn bộ microtask queue trước khi chuyển sang macrotask tiếp theo."

---

**Q2 (Mid): "What is the difference between the microtask queue and the macrotask queue? What outputs would you expect from this code?"**

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
```

**Model Answer:**
"The output is `1`, `4`, `3`, `2`. Here's the reasoning:

`1` and `4` are synchronous — they run immediately on the Call Stack. The `setTimeout` callback goes into the macrotask queue, and the Promise `.then` callback goes into the microtask queue.

Once the synchronous code finishes, the Event Loop checks the microtask queue first — it's always drained completely before the next macrotask is picked up. So `3` (the Promise callback) runs before `2` (the setTimeout callback), even though both had a delay of 0ms.

The rule is: **microtasks have priority over macrotasks**. After every single task completes (including synchronous code finishing), the engine drains every pending microtask — including any new ones added during that drain — before moving to the next macrotask."

**Trả lời (Tiếng Việt):**
"Output là `1`, `4`, `3`, `2`. Giải thích như sau:

`1` và `4` là đồng bộ — chạy ngay trên Call Stack. Callback `setTimeout` vào macrotask queue, callback `.then` của Promise vào microtask queue.

Khi code đồng bộ chạy xong, Event Loop kiểm tra microtask queue trước — nó luôn được drain hoàn toàn trước khi macrotask tiếp theo được chọn. Nên `3` (callback Promise) chạy trước `2` (callback setTimeout), dù cả hai đều có delay 0ms.

Quy tắc là: **microtask có ưu tiên cao hơn macrotask**. Sau khi mỗi task hoàn thành (bao gồm cả code đồng bộ), engine drain tất cả microtask đang chờ — kể cả những cái mới được thêm trong quá trình drain — trước khi chuyển sang macrotask tiếp theo."

---

**Q3 (Mid): "Why does `async/await` affect event loop ordering?"**

**Model Answer:**
"Because `await` is syntactic sugar over Promises, and Promises use microtasks. When you `await` something, execution of that `async` function is suspended and the rest of it is scheduled as a microtask. Control returns to the caller immediately.

So in this code:
```js
async function main() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}
console.log("start");
main();
console.log("end");
```
You get: `start`, `A`, `end`, `B`.

`A` prints synchronously when `main()` is called. Then `await` suspends `main` — the `console.log("end")` in the outer scope runs next. Only after the current synchronous code finishes does the microtask for `B` run."

**Trả lời (Tiếng Việt):**
"Vì `await` là syntactic sugar trên Promise, mà Promise dùng microtask. Khi bạn `await` một thứ gì đó, việc thực thi của `async` function đó bị suspend và phần còn lại của nó được lên lịch như một microtask. Control trả về caller ngay lập tức.

Nên với code này:
```js
async function main() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}
console.log("start");
main();
console.log("end");
```
Bạn sẽ thấy: `start`, `A`, `end`, `B`.

`A` in ra đồng bộ khi `main()` được gọi. Sau đó `await` suspend `main` — `console.log("end")` trong outer scope chạy tiếp. Chỉ sau khi code đồng bộ kết thúc, microtask của `B` mới chạy."

---

**Q4 (Senior): "How would you avoid starving the macrotask queue with long microtask chains?"**

**Model Answer:**
"This is a real production concern. Since the engine drains the entire microtask queue before touching the next macrotask, if your microtask callbacks keep adding more microtasks — say, a recursive Promise chain — you can effectively starve UI rendering or `setTimeout` callbacks indefinitely.

The fix is to deliberately yield to the macrotask queue. You can do this by wrapping a chunk of work in a `setTimeout(..., 0)` or using `scheduler.yield()` in browsers that support it. In Node.js, `setImmediate` gives other I/O a chance to run.

A practical example is processing a large array in chunks:
```js
async function processInChunks(items, chunkSize = 100) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    processChunk(chunk);
    await new Promise(resolve => setTimeout(resolve, 0)); // yield to event loop
  }
}
```
This lets the browser render between chunks, keeping the UI responsive."

**Trả lời (Tiếng Việt):**
"Đây là vấn đề thực tế trong production. Vì engine drain toàn bộ microtask queue trước khi xử lý macrotask tiếp theo, nếu microtask callback của bạn cứ tiếp tục thêm microtask mới — ví dụ một Promise chain đệ quy — bạn có thể làm đói UI rendering hoặc callback `setTimeout` mãi mãi.

Cách sửa là cố tình nhường cho macrotask queue. Bạn có thể làm điều này bằng cách wrap một đoạn work trong `setTimeout(..., 0)` hoặc dùng `scheduler.yield()` trên các browser hỗ trợ. Trong Node.js, `setImmediate` cho phép các I/O khác có cơ hội chạy.

Ví dụ thực tế là xử lý một mảng lớn theo từng chunk:
```js
async function processInChunks(items, chunkSize = 100) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    processChunk(chunk);
    await new Promise(resolve => setTimeout(resolve, 0)); // yield to event loop
  }
}
```
Điều này cho phép browser render giữa các chunk, giữ UI responsive."

---

## 4. Prototype Chain & Prototypal Inheritance

### Q6 — [Junior]

**EN:** What is the prototype chain in JavaScript? How does property lookup work?

**VI:** Prototype chain trong JavaScript là gì? Cơ chế tìm kiếm property hoạt động như thế nào?

**Trả lời:**

Mỗi object trong JavaScript có một thuộc tính ẩn `[[Prototype]]` (truy cập qua `__proto__` hoặc `Object.getPrototypeOf()`), trỏ đến object cha. Khi truy cập một property, JS sẽ tìm theo **chuỗi prototype** cho đến khi gặp `null`.

```
myObj → MyClass.prototype → Object.prototype → null
```

**Property Lookup:**

```js
const animal = {
  type: "Animal",
  describe() {
    return `I am a ${this.type}`;
  }
};

const dog = Object.create(animal); // dog.__proto__ === animal
dog.type = "Dog";
dog.bark = function() { return "Woof!"; };

console.log(dog.bark());      // "Woof!" — own property
console.log(dog.describe());  // "I am a Dog" — found on prototype
console.log(dog.toString());  // "[object Object]" — found on Object.prototype
console.log(dog.xyz);         // undefined — not found in entire chain
```

**Kiểm tra prototype:**

```js
console.log(dog.hasOwnProperty("type"));     // true
console.log(dog.hasOwnProperty("describe")); // false (trên prototype)
console.log("describe" in dog);              // true (tìm cả chain)

console.log(Object.getPrototypeOf(dog) === animal); // true
```

---

### Q7 — [Mid]

**EN:** How does prototypal inheritance work with constructor functions and ES6 classes? Are classes just syntactic sugar?

**VI:** Kế thừa prototypal hoạt động như thế nào với constructor function và ES6 class? Class chỉ là syntactic sugar?

**Trả lời:**

**Constructor Function (cách cũ):**

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} makes a sound`;
};

function Dog(name, breed) {
  Animal.call(this, name); // kế thừa properties
  this.breed = breed;
}
// Thiết lập prototype chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // fix lại constructor

Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

const rex = new Dog("Rex", "German Shepherd");
console.log(rex.speak()); // "Rex makes a sound" — từ Animal.prototype
console.log(rex.bark());  // "Rex barks!" — từ Dog.prototype
console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
```

**ES6 Class (cách mới):**

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }

  static create(name) { // static method — không qua prototype
    return new Animal(name);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // gọi Animal constructor — BẮT BUỘC trước khi dùng this
    this.breed = breed;
  }

  bark() {
    return `${this.name} barks!`;
  }

  speak() { // override
    return `${super.speak()} (woof!)`;
  }
}

const rex = new Dog("Rex", "German Shepherd");
console.log(rex.speak()); // "Rex makes a sound (woof!)"
```

**Class có phải chỉ là syntactic sugar không?**

Gần như vậy, nhưng có một số điểm khác biệt quan trọng:

```js
// 1. Class KHÔNG được hoisted (TDZ, giống let/const)
const d = new Dog("Rex", "Lab"); // ReferenceError nếu class chưa khai báo

// 2. Class body luôn chạy ở strict mode
// 3. class method không enumerable (for...in sẽ không duyệt)
// 4. constructor function có thể gọi không cần `new` (trả về undefined hoặc this)
//    class BẮT BUỘC phải dùng new — TypeError nếu không có new

// Về prototype chain, kết quả hoàn toàn giống nhau:
console.log(typeof Dog); // "function" — class vẫn là function bên dưới
console.log(Object.getPrototypeOf(Dog) === Animal); // true (static kế thừa)
console.log(Object.getPrototypeOf(Dog.prototype) === Animal.prototype); // true
```

---

## 5. this Binding

### Q8 — [Mid]

**EN:** Explain the 4 rules of `this` binding in JavaScript. How does arrow function differ?

**VI:** Giải thích 4 quy tắc binding `this` trong JavaScript. Arrow function khác như thế nào?

**Trả lời:**

`this` trong JavaScript được xác định tại **thời điểm gọi hàm** (call-time), không phải khi khai báo (trừ arrow function).

**Quy tắc 1 — Default Binding (gọi trực tiếp):**

```js
function greet() {
  console.log(this); // window (browser) hoặc global (Node) — strict mode: undefined
}
greet();

"use strict";
function greetStrict() {
  console.log(this); // undefined
}
greetStrict();
```

**Quy tắc 2 — Implicit Binding (gọi qua object):**

```js
const user = {
  name: "Alice",
  greet() {
    console.log(this.name); // "Alice" — this = user
  }
};
user.greet(); // implicit binding

// Mất implicit binding:
const fn = user.greet;
fn(); // undefined — this = global/undefined (lost context!)

// Fix:
const boundFn = user.greet.bind(user);
boundFn(); // "Alice"
```

**Quy tắc 3 — Explicit Binding (call, apply, bind):**

```js
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I am ${this.name}${punctuation}`);
}

const alice = { name: "Alice" };
const bob   = { name: "Bob" };

introduce.call(alice, "Hello", "!");   // "Hello, I am Alice!"
introduce.apply(bob, ["Hi", "."]);     // "Hi, I am Bob."
const aliceIntro = introduce.bind(alice);
aliceIntro("Hey", "~");               // "Hey, I am Alice~"
```

**Quy tắc 4 — new Binding:**

```js
function Person(name) {
  // new tạo object mới, this trỏ đến object đó
  this.name = name;
  this.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
  };
}

const alice = new Person("Alice");
alice.greet(); // "Hi, I'm Alice"
```

**Độ ưu tiên:** `new` > `explicit` > `implicit` > `default`

---

**Arrow Function — Lexical `this`:**

Arrow function **không có `this` riêng**. Nó kế thừa `this` từ **lexical scope** (scope nơi nó được định nghĩa).

```js
const timer = {
  seconds: 0,

  // WRONG — regular function, this = global trong callback
  startWrong() {
    setInterval(function() {
      this.seconds++; // this = window — bug!
      console.log(this.seconds);
    }, 1000);
  },

  // CORRECT — arrow function, this = timer object
  startCorrect() {
    setInterval(() => {
      this.seconds++; // this = timer — correct!
      console.log(this.seconds);
    }, 1000);
  }
};

// Arrow function KHÔNG bị ảnh hưởng bởi call/apply/bind
const obj = { name: "test" };
const arrowFn = () => console.log(this); // this = outer scope
arrowFn.call(obj); // vẫn là outer `this`, không phải obj

// KHÔNG dùng arrow function cho object methods
const person = {
  name: "Alice",
  greet: () => console.log(this.name) // WRONG — this không phải person
};
```

---

## 6. Promise API

### Q9 — [Mid]

**EN:** Explain `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`. When would you use each?

**VI:** Giải thích `Promise.all`, `Promise.allSettled`, `Promise.race`, `Promise.any`. Khi nào dùng từng loại?

**Trả lời:**

```js
const p1 = Promise.resolve(1);
const p2 = new Promise(resolve => setTimeout(() => resolve(2), 100));
const p3 = Promise.reject(new Error("failed"));
const p4 = new Promise(resolve => setTimeout(() => resolve(4), 200));
```

---

**`Promise.all(iterable)`:**

- Chờ **tất cả** promise resolve
- Nếu **một** promise reject → toàn bộ reject ngay lập tức (fail-fast)
- Kết quả: array theo thứ tự input (không phải thứ tự hoàn thành)

```js
// Dùng khi: tất cả đều phải thành công, các task độc lập, muốn chạy song song
Promise.all([p1, p2, p4])
  .then(([r1, r2, r4]) => console.log(r1, r2, r4)) // 1, 2, 4
  .catch(err => console.error(err));

Promise.all([p1, p3, p4]) // p3 reject
  .catch(err => console.error(err.message)); // "failed" — p4 bị bỏ qua

// Ví dụ thực tế: tải user + posts + comments song song
const [user, posts, comments] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id)
]);
```

---

**`Promise.allSettled(iterable)`:**

- Chờ **tất cả** promise kết thúc (dù resolve hay reject)
- **Không bao giờ** reject
- Kết quả: array `{ status: "fulfilled" | "rejected", value | reason }`

```js
// Dùng khi: muốn biết kết quả của TẤT CẢ, kể cả promise thất bại
Promise.allSettled([p1, p3, p4]).then(results => {
  results.forEach(result => {
    if (result.status === "fulfilled") {
      console.log("Success:", result.value);
    } else {
      console.log("Failed:", result.reason.message);
    }
  });
});
// Success: 1
// Failed: failed
// Success: 4

// Ví dụ thực tế: gửi notification đến nhiều user, cần biết cái nào fail
const results = await Promise.allSettled(userIds.map(id => sendEmail(id)));
const failed = results.filter(r => r.status === "rejected");
```

---

**`Promise.race(iterable)`:**

- Trả về kết quả của promise **nhanh nhất** (dù resolve hay reject)
- Dùng cho timeout pattern

```js
// Dùng khi: muốn lấy kết quả đầu tiên, timeout API call
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Timeout!")), ms)
  );
  return Promise.race([promise, timeout]);
}

try {
  const data = await withTimeout(fetchData(), 5000); // fail nếu > 5s
} catch (err) {
  console.error(err.message); // "Timeout!" hoặc lỗi thật
}
```

---

**`Promise.any(iterable)`:**

- Trả về promise **đầu tiên resolve thành công**
- Nếu **tất cả** reject → throw `AggregateError`
- Ngược với `Promise.all` (all resolve) — chỉ cần một thành công

```js
// Dùng khi: thử nhiều nguồn, lấy cái nào trả về trước (fallback servers)
const fastestMirror = await Promise.any([
  fetch("https://cdn1.example.com/data"),
  fetch("https://cdn2.example.com/data"),
  fetch("https://cdn3.example.com/data")
]);

Promise.any([
  Promise.reject(new Error("err1")),
  Promise.reject(new Error("err2"))
]).catch(err => {
  console.log(err instanceof AggregateError); // true
  console.log(err.errors); // [Error: err1, Error: err2]
});
```

**Bảng tổng hợp:**

| Method | Resolve khi | Reject khi | Use case |
|---|---|---|---|
| `Promise.all` | Tất cả resolve | Một cái reject | Parallel tasks, tất cả bắt buộc thành công |
| `Promise.allSettled` | Tất cả kết thúc | Không bao giờ | Cần biết mọi kết quả |
| `Promise.race` | Một cái xong đầu tiên | Một cái reject đầu tiên | Timeout, fastest response |
| `Promise.any` | Một cái resolve đầu tiên | Tất cả reject | Fallback, first success |

---

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is a Promise in JavaScript, and why do we need it?"**

**Model Answer:**
"A Promise is an object that represents a value that isn't available yet — it will either resolve with a value or reject with an error at some point in the future. We needed Promises because the older callback approach led to deeply nested, hard-to-read code known as 'callback hell.'

A Promise has three states: `pending`, `fulfilled`, and `rejected`. Once it settles, it stays that way — you can attach multiple `.then` handlers and they all receive the same value. This makes async code much easier to chain and reason about:
```js
fetchUser(id)
  .then(user => fetchPosts(user.id))
  .then(posts => renderPosts(posts))
  .catch(err => showError(err));
```
Each `.then` returns a new Promise, so you can chain operations sequentially without nesting."

**Trả lời (Tiếng Việt):**
"Promise là một object đại diện cho giá trị chưa có sẵn — nó sẽ resolve với một giá trị hoặc reject với lỗi tại một thời điểm nào đó trong tương lai. Chúng ta cần Promise vì cách dùng callback truyền thống dẫn đến code lồng nhau sâu khó đọc, gọi là 'callback hell.'

Promise có ba trạng thái: `pending`, `fulfilled`, và `rejected`. Khi đã settle, nó không đổi nữa — bạn có thể attach nhiều handler `.then` và chúng đều nhận cùng một giá trị. Điều này làm code async dễ chain và dễ hiểu hơn nhiều:
```js
fetchUser(id)
  .then(user => fetchPosts(user.id))
  .then(posts => renderPosts(posts))
  .catch(err => showError(err));
```
Mỗi `.then` trả về một Promise mới, nên bạn có thể chain các operation theo thứ tự mà không cần lồng nhau."

---

**Q2 (Mid): "What is the difference between `Promise.all` and `Promise.allSettled`? When would you pick one over the other?"**

**Model Answer:**
"The key difference is how they handle failures. `Promise.all` is fail-fast — if any single promise rejects, the whole thing rejects immediately and you lose the results of everything else. `Promise.allSettled` always waits for all promises to complete and gives you an array with the status of every single one.

I'd use `Promise.all` when all tasks are required for the operation to succeed — like loading a user's profile, their settings, and their permissions in parallel. If any one fails, there's no point continuing.

I'd use `Promise.allSettled` when some tasks can fail without breaking the whole flow — like sending notifications to a batch of users. I want to know which ones failed so I can retry or log them, but I don't want one failed notification to abort the whole batch:
```js
const results = await Promise.allSettled(users.map(u => sendEmail(u)));
const failed = results.filter(r => r.status === 'rejected');
console.log(`${failed.length} emails failed`);
```"

**Trả lời (Tiếng Việt):**
"Điểm khác biệt chính là cách chúng xử lý lỗi. `Promise.all` là fail-fast — nếu bất kỳ promise nào reject, toàn bộ bị reject ngay và bạn mất kết quả của mọi thứ còn lại. `Promise.allSettled` luôn chờ tất cả promise hoàn thành và trả về mảng với status của từng cái một.

Tôi dùng `Promise.all` khi tất cả task đều cần thiết để operation thành công — như load profile, settings, và permissions của user song song. Nếu một cái fail, không có lý do gì tiếp tục.

Tôi dùng `Promise.allSettled` khi một số task có thể fail mà không phá vỡ toàn bộ flow — như gửi notification cho một batch user. Tôi muốn biết cái nào fail để retry hoặc log, nhưng không muốn một notification thất bại làm hỏng cả batch:
```js
const results = await Promise.allSettled(users.map(u => sendEmail(u)));
const failed = results.filter(r => r.status === 'rejected');
console.log(`${failed.length} emails failed`);
```"

---

**Q3 (Mid): "How would you implement a timeout for a Promise?"**

**Model Answer:**
"You use `Promise.race`. The idea is to create two competing promises: your actual async operation and a 'timeout bomb' that rejects after a specified duration. Whichever settles first wins:

```js
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

try {
  const data = await withTimeout(fetchData(), 5000);
} catch (err) {
  if (err.message.includes('Timed out')) {
    // handle timeout specifically
  }
}
```

One thing to be aware of: canceling the underlying fetch if the timeout fires requires an `AbortController`. `Promise.race` doesn't cancel the losing promise — it just stops you from waiting for it. So the fetch might still be running in the background consuming resources."

**Trả lời (Tiếng Việt):**
"Bạn dùng `Promise.race`. Ý tưởng là tạo hai promise cạnh tranh: async operation thật sự và một 'quả bom timeout' reject sau khoảng thời gian nhất định. Cái nào settle trước thì thắng:

```js
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

try {
  const data = await withTimeout(fetchData(), 5000);
} catch (err) {
  if (err.message.includes('Timed out')) {
    // xử lý timeout cụ thể
  }
}
```

Một điều cần lưu ý: để hủy fetch bên dưới khi timeout xảy ra cần dùng `AbortController`. `Promise.race` không cancel promise thua — nó chỉ không chờ kết quả nữa. Nên fetch có thể vẫn đang chạy ngầm và tiêu tốn tài nguyên."

---

**Q4 (Senior): "What is Promise chaining vs. `async/await` — when would you prefer one over the other?"**

**Model Answer:**
"They're equivalent under the hood, but they read differently. Promise chaining is elegant for linear pipelines where each step transforms the data:
```js
fetchUser(id)
  .then(normalizeUser)
  .then(enrichWithPermissions)
  .then(renderProfile);
```
`async/await` shines when you have conditional logic, loops, or need to use multiple awaited values together, because it reads like synchronous code:
```js
async function loadDashboard(userId) {
  const user = await fetchUser(userId);
  if (user.role === 'admin') {
    const [stats, logs] = await Promise.all([fetchStats(), fetchLogs()]);
    return buildAdminView(user, stats, logs);
  }
  return buildUserView(user);
}
```
Mixing the two is also fine. I often use `Promise.all` inside an `async/await` function because it's the clearest way to express 'run these in parallel, wait for all of them.' The important thing is that forgetting `await` silently gives you a Promise object instead of its resolved value — that's a common bug worth watching for."

**Trả lời (Tiếng Việt):**
"Về bản chất chúng tương đương, nhưng đọc khác nhau. Promise chaining thanh lịch cho pipeline tuyến tính nơi mỗi bước transform data:
```js
fetchUser(id)
  .then(normalizeUser)
  .then(enrichWithPermissions)
  .then(renderProfile);
```
`async/await` tỏa sáng khi bạn có logic điều kiện, vòng lặp, hoặc cần dùng nhiều giá trị đã await cùng nhau, vì nó đọc như code đồng bộ:
```js
async function loadDashboard(userId) {
  const user = await fetchUser(userId);
  if (user.role === 'admin') {
    const [stats, logs] = await Promise.all([fetchStats(), fetchLogs()]);
    return buildAdminView(user, stats, logs);
  }
  return buildUserView(user);
}
```
Kết hợp cả hai cũng hoàn toàn ổn. Tôi thường dùng `Promise.all` bên trong `async/await` function vì đó là cách rõ ràng nhất để diễn đạt 'chạy song song, chờ tất cả.' Điều quan trọng cần chú ý là quên `await` sẽ im lặng trả về một Promise object thay vì giá trị đã resolve — đó là bug phổ biến cần để ý."

---

## 7. async / await & Error Handling

### Q10 — [Mid]

**EN:** How do you handle errors properly with async/await? What are the common patterns?

**VI:** Làm thế nào để xử lý lỗi đúng cách với async/await? Các pattern phổ biến là gì?

**Trả lời:**

**Pattern 1 — try/catch/finally (cơ bản):**

```js
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Failed to fetch user:", error.message);
    throw error; // re-throw để caller xử lý
  } finally {
    console.log("Request completed"); // luôn chạy
  }
}
```

**Pattern 2 — Go-style error handling (tránh nested try/catch):**

```js
// Helper function
async function to(promise) {
  try {
    const data = await promise;
    return [null, data];
  } catch (err) {
    return [err, null];
  }
}

async function processOrder(orderId) {
  const [userErr, user] = await to(fetchUser(orderId));
  if (userErr) return { error: "Failed to fetch user" };

  const [orderErr, order] = await to(fetchOrder(orderId));
  if (orderErr) return { error: "Failed to fetch order" };

  const [payErr, payment] = await to(processPayment(order));
  if (payErr) return { error: "Payment failed" };

  return { success: true, order, payment };
}
```

**Pattern 3 — Xử lý lỗi theo loại:**

```js
class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = "NetworkError";
    this.statusCode = statusCode;
  }
}

class ValidationError extends Error {
  constructor(message, fields) {
    super(message);
    this.name = "ValidationError";
    this.fields = fields;
  }
}

async function submitForm(data) {
  try {
    const result = await api.submit(data);
    return result;
  } catch (error) {
    if (error instanceof ValidationError) {
      showFieldErrors(error.fields);
    } else if (error instanceof NetworkError) {
      if (error.statusCode === 401) redirectToLogin();
      else showToast("Network error, please retry");
    } else {
      // Unknown error — log to monitoring (Sentry, etc.)
      reportError(error);
      throw error;
    }
  }
}
```

**Pattern 4 — Parallel với error handling:**

```js
// SAI — sequential, chậm
async function slowApproach(ids) {
  const results = [];
  for (const id of ids) {
    const data = await fetchItem(id); // mỗi cái chờ nhau
    results.push(data);
  }
  return results;
}

// ĐÚNG — parallel
async function fastApproach(ids) {
  try {
    return await Promise.all(ids.map(id => fetchItem(id)));
  } catch (err) {
    console.error("One of the requests failed:", err);
    throw err;
  }
}

// ĐÚNG — parallel, không fail-fast
async function resilientApproach(ids) {
  const results = await Promise.allSettled(ids.map(id => fetchItem(id)));
  return {
    succeeded: results.filter(r => r.status === "fulfilled").map(r => r.value),
    failed: results.filter(r => r.status === "rejected").map(r => r.reason)
  };
}
```

---

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What does `async/await` do, and how does it relate to Promises?"**

**Model Answer:**
"`async/await` is syntactic sugar built on top of Promises — it doesn't introduce a new async mechanism, it just makes Promise-based code look synchronous. When you mark a function with `async`, it automatically returns a Promise. When you use `await` inside it, you pause execution of that function until the awaited Promise settles, then resume with the resolved value.

So instead of:
```js
fetchUser(id).then(user => console.log(user.name));
```
You write:
```js
const user = await fetchUser(id);
console.log(user.name);
```
Same behavior, much easier to read — especially when you have multiple sequential async steps."

**Trả lời (Tiếng Việt):**
"`async/await` là syntactic sugar xây trên Promise — nó không giới thiệu cơ chế async mới, chỉ làm code dựa trên Promise trông như đồng bộ. Khi bạn đánh dấu function với `async`, nó tự động trả về Promise. Khi bạn dùng `await` bên trong, bạn pause việc thực thi function đó cho đến khi Promise được settle, rồi resume với giá trị đã resolve.

Thay vì:
```js
fetchUser(id).then(user => console.log(user.name));
```
Bạn viết:
```js
const user = await fetchUser(id);
console.log(user.name);
```
Hành vi giống nhau, dễ đọc hơn nhiều — đặc biệt khi bạn có nhiều bước async tuần tự."

---

**Q2 (Junior/Mid): "How do you handle errors with async/await?"**

**Model Answer:**
"The primary way is `try/catch`. Since `await` unwraps the Promise, any rejection throws an exception that the `catch` block catches — just like synchronous errors:

```js
async function loadUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (err) {
    console.error('Failed to load user:', err.message);
    throw err; // re-throw so the caller knows it failed
  } finally {
    hideLoadingSpinner(); // always runs, success or failure
  }
}
```

One thing I always do is re-throw the error after logging it, unless I'm truly handling it and want to return a fallback. If you swallow the error silently, the caller gets `undefined` and has no idea something went wrong — which leads to very confusing bugs downstream."

**Trả lời (Tiếng Việt):**
"Cách chính là `try/catch`. Vì `await` unwrap Promise, bất kỳ rejection nào đều throw exception mà `catch` block bắt — giống như lỗi đồng bộ:

```js
async function loadUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (err) {
    console.error('Failed to load user:', err.message);
    throw err; // re-throw để caller biết nó đã fail
  } finally {
    hideLoadingSpinner(); // luôn chạy, dù thành công hay thất bại
  }
}
```

Một điều tôi luôn làm là re-throw lỗi sau khi log, trừ khi tôi thực sự xử lý nó và muốn trả về fallback. Nếu bạn im lặng nuốt lỗi, caller nhận được `undefined` và không biết có chuyện gì xảy ra — dẫn đến bug rất khó debug ở phía sau."

---

**Q3 (Mid): "What is the common `async/await` performance mistake, and how do you fix it?"**

**Model Answer:**
"The classic mistake is using `await` inside a `for` loop when the requests are actually independent:

```js
// SLOW — sequential, each waits for the previous
for (const id of userIds) {
  const user = await fetchUser(id); // total time = sum of all requests
}
```

Each `await` pauses the loop, so requests run one after another. If you have 10 requests at 100ms each, that's 1 second total. The fix is to kick off all requests at once with `Promise.all`:

```js
// FAST — parallel
const users = await Promise.all(userIds.map(id => fetchUser(id)));
// total time ≈ slowest single request
```

The only time you want sequential awaits in a loop is when each request depends on the result of the previous one — like paginated fetching where you need the cursor from response N to make request N+1."

**Trả lời (Tiếng Việt):**
"Lỗi kinh điển là dùng `await` trong vòng lặp `for` khi các request thực ra độc lập với nhau:

```js
// CHẬM — tuần tự, mỗi cái chờ cái trước
for (const id of userIds) {
  const user = await fetchUser(id); // tổng thời gian = tổng tất cả request
}
```

Mỗi `await` pause vòng lặp, nên request chạy lần lượt. Nếu bạn có 10 request mỗi cái 100ms, đó là 1 giây tổng cộng. Cách sửa là kick off tất cả request cùng lúc với `Promise.all`:

```js
// NHANH — song song
const users = await Promise.all(userIds.map(id => fetchUser(id)));
// tổng thời gian ≈ request đơn chậm nhất
```

Trường hợp duy nhất bạn muốn await tuần tự trong vòng lặp là khi mỗi request phụ thuộc vào kết quả của request trước — như paginated fetching cần cursor từ response N để tạo request N+1."

---

**Q4 (Senior): "How do you handle errors when using `Promise.all` with async/await? What's the difference between letting it throw vs. using `allSettled`?"**

**Model Answer:**
"With `Promise.all`, if any promise rejects, the whole `await` throws. You handle it with a `try/catch` around the `await Promise.all(...)`. That's appropriate when you need all results — if one fails, the whole operation is invalid.

But if you want granular error handling per-item, `Promise.allSettled` is better:

```js
const results = await Promise.allSettled(ids.map(id => fetchItem(id)));
const succeeded = results.filter(r => r.status === 'fulfilled').map(r => r.value);
const failed = results.filter(r => r.status === 'rejected');

if (failed.length > 0) {
  logger.warn(`${failed.length} items failed`, failed.map(r => r.reason));
}
return succeeded;
```

I also use a Go-style helper in complex flows to avoid deeply nested `try/catch`:
```js
const [err, user] = await to(fetchUser(id));
if (err) return handleError(err);
```
This pattern keeps error handling inline and explicit without the visual noise of multiple try/catch blocks."

**Trả lời (Tiếng Việt):**
"Với `Promise.all`, nếu bất kỳ promise nào reject, toàn bộ `await` throw. Bạn xử lý bằng `try/catch` bọc quanh `await Promise.all(...)`. Điều đó phù hợp khi bạn cần tất cả kết quả — nếu một cái fail, toàn bộ operation không còn hợp lệ.

Nhưng nếu bạn muốn xử lý lỗi riêng lẻ cho từng item, `Promise.allSettled` tốt hơn:

```js
const results = await Promise.allSettled(ids.map(id => fetchItem(id)));
const succeeded = results.filter(r => r.status === 'fulfilled').map(r => r.value);
const failed = results.filter(r => r.status === 'rejected');

if (failed.length > 0) {
  logger.warn(`${failed.length} items failed`, failed.map(r => r.reason));
}
return succeeded;
```

Tôi cũng dùng Go-style helper trong các flow phức tạp để tránh nested `try/catch` sâu:
```js
const [err, user] = await to(fetchUser(id));
if (err) return handleError(err);
```
Pattern này giữ error handling inline và rõ ràng mà không bị rối bởi nhiều khối try/catch."

---

**Q5 (Senior): "What is an unhandled Promise rejection, and how do you prevent it in production?"**

**Model Answer:**
"An unhandled rejection is when a Promise rejects and nothing catches it. In Node.js, this used to just print a warning; in modern Node.js versions, it crashes the process. In the browser, it fires an `unhandledrejection` event.

The common causes: calling an `async` function without `await` and without a `.catch()`, or throwing inside a `.then()` callback without a downstream `.catch()`.

Prevention strategies: always `await` or `.catch()` async calls, use a global unhandled rejection handler as a safety net:
```js
process.on('unhandledRejection', (reason, promise) => {
  logger.error('Unhandled rejection:', reason);
  // Don't swallow it — alert your monitoring system
});
```
In React, the Error Boundary doesn't catch async errors, so async event handlers need their own error handling. I treat unhandled rejection warnings in development the same as TypeScript errors — fix them before merging."

**Trả lời (Tiếng Việt):**
"Unhandled rejection là khi Promise reject mà không có gì bắt nó. Trong Node.js, điều này trước đây chỉ in warning; trong các phiên bản Node.js hiện đại, nó crash process. Trong browser, nó fire event `unhandledrejection`.

Nguyên nhân phổ biến: gọi `async` function mà không có `await` và không có `.catch()`, hoặc throw trong `.then()` callback mà không có `.catch()` ở phía sau.

Chiến lược phòng ngừa: luôn `await` hoặc `.catch()` các async call, dùng global unhandled rejection handler như lưới an toàn:
```js
process.on('unhandledRejection', (reason, promise) => {
  logger.error('Unhandled rejection:', reason);
  // Đừng nuốt nó — alert hệ thống monitoring của bạn
});
```
Trong React, Error Boundary không bắt được async error, nên async event handler cần tự xử lý lỗi của chúng. Tôi coi unhandled rejection warning trong development giống như TypeScript error — fix trước khi merge."

---

## 8. Debounce vs Throttle

### Q11 — [Mid]

**EN:** What is the difference between debounce and throttle? Implement both from scratch.

**VI:** Sự khác biệt giữa debounce và throttle là gì? Implement cả hai từ đầu.

**Trả lời:**

**Debounce** — trì hoãn thực thi đến khi người dùng **dừng** gọi trong khoảng thời gian `delay`.
Ví dụ: search box — chỉ gọi API sau khi người dùng ngừng gõ 300ms.

**Throttle** — đảm bảo hàm chỉ được gọi **tối đa một lần** trong mỗi `interval`.
Ví dụ: scroll handler — gọi tối đa 1 lần mỗi 200ms dù scroll liên tục.

```
User events: --A--B--C---------D--E--
Debounce:    ------------------C-----E  (chỉ gọi khi dừng)
Throttle:    --A-----------D----------  (gọi đầu mỗi interval)
```

---

**Debounce Implementation:**

```js
function debounce(fn, delay) {
  let timeoutId = null;

  return function(...args) {
    // Hủy timeout cũ nếu hàm được gọi lại trong delay
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      fn.apply(this, args);
      timeoutId = null;
    }, delay);
  };
}

// Usage
const searchInput = document.getElementById("search");
const handleSearch = debounce((event) => {
  console.log("API call with:", event.target.value);
}, 300);

searchInput.addEventListener("input", handleSearch);

// Debounce với immediate option (gọi ngay lần đầu, debounce sau)
function debounceAdvanced(fn, delay, immediate = false) {
  let timeoutId = null;

  return function(...args) {
    const shouldCallNow = immediate && !timeoutId;

    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!immediate) fn.apply(this, args);
    }, delay);

    if (shouldCallNow) fn.apply(this, args);
  };
}
```

**Throttle Implementation:**

```js
// Throttle dùng timestamp (leading edge)
function throttle(fn, interval) {
  let lastCallTime = 0;

  return function(...args) {
    const now = Date.now();
    const timeSinceLast = now - lastCallTime;

    if (timeSinceLast >= interval) {
      lastCallTime = now;
      fn.apply(this, args);
    }
  };
}

// Throttle dùng flag (trailing edge option)
function throttleWithTrailing(fn, interval) {
  let timeoutId = null;
  let lastArgs = null;

  const later = function() {
    timeoutId = null;
    if (lastArgs) {
      fn.apply(this, lastArgs);
      lastArgs = null;
      timeoutId = setTimeout(later, interval);
    }
  };

  return function(...args) {
    if (!timeoutId) {
      fn.apply(this, args);
      timeoutId = setTimeout(later, interval);
    } else {
      lastArgs = args; // lưu args để gọi trailing
    }
  };
}

// Usage
window.addEventListener(
  "scroll",
  throttle(() => {
    console.log("Scroll position:", window.scrollY);
  }, 200)
);
```

---

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is debouncing, and when would you use it?"**

**Model Answer:**
"Debouncing means delaying a function call until after a specified quiet period has passed. Every time the function is triggered, you reset the timer. The function only actually runs when the user stops triggering it.

The textbook example is a search input. You don't want to fire an API call on every single keystroke — that would spam the server. With a 300ms debounce, you wait until the user pauses typing before sending the request:

```js
const handleSearch = debounce((event) => {
  fetchSearchResults(event.target.value);
}, 300);

searchInput.addEventListener('input', handleSearch);
```

Other good use cases: form validation (validate after the user finishes typing in a field), window resize handlers where you recalculate layout, and auto-save — save a draft only after the user pauses editing."

**Trả lời (Tiếng Việt):**
"Debounce có nghĩa là trì hoãn việc gọi function cho đến khi hết một khoảng thời gian yên lặng nhất định. Mỗi khi function được trigger, bạn reset timer. Function chỉ thực sự chạy khi người dùng ngừng trigger nó.

Ví dụ điển hình là ô search. Bạn không muốn gọi API mỗi keystroke — điều đó sẽ spam server. Với debounce 300ms, bạn đợi cho đến khi người dùng dừng gõ mới gửi request:

```js
const handleSearch = debounce((event) => {
  fetchSearchResults(event.target.value);
}, 300);

searchInput.addEventListener('input', handleSearch);
```

Các use case tốt khác: form validation (validate sau khi người dùng gõ xong trong một field), window resize handler nơi bạn tính toán lại layout, và auto-save — chỉ lưu draft sau khi người dùng dừng chỉnh sửa."

---

**Q2 (Junior/Mid): "What is throttling, and how does it differ from debouncing?"**

**Model Answer:**
"Throttling ensures a function runs at most once per time interval, no matter how many times it's triggered. Unlike debouncing — which postpones until activity stops — throttling guarantees regular execution while activity is ongoing.

A scroll event handler is the classic use case. If a user scrolls for 3 seconds straight, you don't want your handler to fire hundreds of times per second. With a 200ms throttle, it fires at most 5 times per second — much more manageable:

```js
window.addEventListener('scroll', throttle(() => {
  updateScrollProgressBar();
}, 200));
```

The mental model: debounce = 'wait until they stop', throttle = 'run at most once every N milliseconds'. Use debounce when you only care about the final state. Use throttle when you want regular sampling during continuous activity."

**Trả lời (Tiếng Việt):**
"Throttle đảm bảo function chạy nhiều nhất một lần mỗi khoảng thời gian, dù nó được trigger bao nhiêu lần. Khác với debounce — cái trì hoãn cho đến khi hoạt động dừng lại — throttle đảm bảo thực thi đều đặn trong khi hoạt động đang diễn ra.

Scroll event handler là use case kinh điển. Nếu người dùng scroll liên tục trong 3 giây, bạn không muốn handler chạy hàng trăm lần mỗi giây. Với throttle 200ms, nó chạy nhiều nhất 5 lần mỗi giây — dễ quản lý hơn nhiều:

```js
window.addEventListener('scroll', throttle(() => {
  updateScrollProgressBar();
}, 200));
```

Cách nhớ: debounce = 'đợi đến khi họ dừng', throttle = 'chạy nhiều nhất một lần mỗi N milliseconds'. Dùng debounce khi bạn chỉ quan tâm đến state cuối cùng. Dùng throttle khi bạn muốn sampling đều đặn trong suốt quá trình hoạt động liên tục."

---

**Q3 (Mid): "Can you implement a basic debounce function from scratch?"**

**Model Answer:**
"Sure. The core idea is: return a wrapper function. That wrapper clears any existing timer and sets a new one. If the wrapper gets called again before the timer fires, the old timer is canceled and a new one starts:

```js
function debounce(fn, delay) {
  let timerId = null;

  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args);
      timerId = null;
    }, delay);
  };
}
```

The `apply(this, args)` is important — it preserves the calling context and passes along any arguments. A common enhancement is the `immediate` option, which calls the function right away on the leading edge and then blocks subsequent calls until the delay expires. Lodash's `debounce` supports both leading and trailing edge options."

**Trả lời (Tiếng Việt):**
"Chắc chắn rồi. Ý tưởng cốt lõi là: trả về một wrapper function. Wrapper đó clear timer hiện có và set timer mới. Nếu wrapper được gọi lại trước khi timer fire, timer cũ bị cancel và timer mới bắt đầu:

```js
function debounce(fn, delay) {
  let timerId = null;

  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args);
      timerId = null;
    }, delay);
  };
}
```

`apply(this, args)` quan trọng — nó giữ nguyên calling context và truyền các argument. Một cải tiến phổ biến là option `immediate`, gọi function ngay lập tức ở leading edge rồi block các lần gọi tiếp theo cho đến khi delay hết. Lodash `debounce` hỗ trợ cả leading và trailing edge option."

---

**Q4 (Senior): "In a React component, what goes wrong if you create a debounced function inline in the render body, and how do you fix it?"**

**Model Answer:**
"Great question — this is a real trap. If you write `const handleInput = debounce(search, 300)` directly inside your component function body, a new debounced function is created on every render. Each new instance has its own internal timer state, so the debouncing never actually works — every render resets it.

The fix is to memoize the debounced function with `useMemo` or `useCallback` so it's only created once:

```js
const handleInput = useMemo(
  () => debounce((value) => fetchResults(value), 300),
  [] // empty deps — create once
);
```

But there's a follow-on issue: if that debounced function closes over stale props or state (because it was created on mount), you'll get stale closure bugs. One clean solution is to use a `useRef` to hold the latest handler, and have the stable debounced function always call through the ref:

```js
const latestSearch = useRef(search);
useEffect(() => { latestSearch.current = search; });

const debouncedSearch = useMemo(
  () => debounce((...args) => latestSearch.current(...args), 300),
  []
);
```

Also, don't forget to cancel the debounce timer on unmount — call `debouncedSearch.cancel()` in a `useEffect` cleanup to avoid state updates on unmounted components."

**Trả lời (Tiếng Việt):**
"Câu hỏi hay — đây là bẫy thực tế. Nếu bạn viết `const handleInput = debounce(search, 300)` trực tiếp trong body của component function, một hàm debounce mới được tạo ra mỗi lần render. Mỗi instance mới có timer state riêng, nên debouncing thực ra không bao giờ hoạt động — mỗi lần render reset nó.

Cách sửa là memoize hàm debounce với `useMemo` hoặc `useCallback` để nó chỉ được tạo một lần:

```js
const handleInput = useMemo(
  () => debounce((value) => fetchResults(value), 300),
  [] // empty deps — tạo một lần
);
```

Nhưng có vấn đề tiếp theo: nếu hàm debounce đó đóng gói props hoặc state cũ (vì nó được tạo khi mount), bạn sẽ gặp stale closure bug. Một giải pháp sạch là dùng `useRef` để giữ handler mới nhất, và hàm debounce ổn định luôn gọi qua ref:

```js
const latestSearch = useRef(search);
useEffect(() => { latestSearch.current = search; });

const debouncedSearch = useMemo(
  () => debounce((...args) => latestSearch.current(...args), 300),
  []
);
```

Cũng đừng quên cancel debounce timer khi unmount — gọi `debouncedSearch.cancel()` trong `useEffect` cleanup để tránh state update trên unmounted component."

---

**Q5 (Senior): "When would you use throttle instead of debounce for a real-time feature like a collaborative editor?"**

**Model Answer:**
"In a collaborative editor — like Google Docs — you need to send keystrokes to other users in near-real-time, so debouncing is wrong. Debounce would only broadcast changes after the user pauses, which means collaborators would see nothing until you stop typing. That's a terrible experience.

Throttle is the right tool here: send an update at most every 100ms, so collaborators see changes flowing in regularly while you type, without overwhelming the WebSocket connection.

You'd typically combine it with an operational transform or CRDT algorithm to merge concurrent edits. The throttle controls the send frequency; the conflict resolution algorithm handles what happens when two people edit the same spot simultaneously.

For the cursor position specifically — showing where other users' cursors are — you'd also throttle the `mousemove` or selection change events. Maybe every 50-100ms. Sending cursor positions on every single mouse pixel would be prohibitively expensive at scale."

**Trả lời (Tiếng Việt):**
"Trong collaborative editor — như Google Docs — bạn cần gửi keystroke đến người dùng khác gần real-time, nên debounce là sai. Debounce sẽ chỉ broadcast thay đổi sau khi người dùng dừng gõ, có nghĩa là người cộng tác không thấy gì cho đến khi bạn ngừng. Đó là trải nghiệm tệ.

Throttle mới là công cụ đúng: gửi update nhiều nhất mỗi 100ms, để người cộng tác thấy thay đổi chảy đến đều đặn trong khi bạn gõ, mà không làm quá tải kết nối WebSocket.

Thường bạn kết hợp với thuật toán operational transform hoặc CRDT để merge các chỉnh sửa đồng thời. Throttle kiểm soát tần suất gửi; thuật toán conflict resolution xử lý khi hai người chỉnh sửa cùng chỗ cùng lúc.

Riêng với vị trí con trỏ — hiển thị con trỏ của người dùng khác đang ở đâu — bạn cũng throttle sự kiện `mousemove` hoặc selection change. Khoảng 50-100ms. Gửi vị trí con trỏ mỗi pixel chuột di chuyển sẽ rất tốn kém ở quy mô lớn."

---

## 9. ES6+ Features

### Q12 — [Junior]

**EN:** Explain destructuring, spread/rest, optional chaining, nullish coalescing, and template literals.

**VI:** Giải thích destructuring, spread/rest, optional chaining, nullish coalescing, và template literals.

**Trả lời:**

**Destructuring:**

```js
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest);  // [3, 4, 5]

// Swap variables
let a = 1, b = 2;
[a, b] = [b, a]; // a=2, b=1

// Object destructuring
const { name, age, address: { city } = {} } = user;

// Rename + default value
const { name: userName = "Anonymous", role = "user" } = user;

// Function parameter destructuring
function displayUser({ name, age, email = "N/A" }) {
  console.log(`${name} (${age}) — ${email}`);
}

// Nested destructuring
const { data: { users: [firstUser] } } = apiResponse;
```

**Spread / Rest:**

```js
// Spread — mở rộng iterable
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];        // [1, 2, 3, 4, 5]
const merged = [...arr1, ...arr2];    // merge arrays

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, b: 3, c: 4 }; // { a: 1, b: 3, c: 4 } — override b

// Clone (shallow)
const copy = [...arr1];
const objCopy = { ...obj1 };

// Rest — thu gom tham số còn lại
function sum(first, second, ...others) {
  return first + second + others.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4, 5); // 15

// Spread với function call
const numbers = [5, 3, 8, 1];
Math.max(...numbers); // 8
```

**Optional Chaining (`?.`):**

```js
// Không cần kiểm tra null/undefined trung gian
const city = user?.address?.city;           // undefined nếu bất kỳ bước nào null
const firstTag = post?.tags?.[0];           // truy cập array
const value = obj?.method?.();              // gọi method

// Trước ES2020 (verbose):
const city = user && user.address && user.address.city;

// Kết hợp với nullish coalescing
const displayName = user?.profile?.displayName ?? "Anonymous";
```

**Nullish Coalescing (`??`):**

```js
// ?? — chỉ fallback khi giá trị là null hoặc undefined
// || — fallback khi giá trị là falsy (0, "", false, null, undefined)

const count = 0;
console.log(count || 10);  // 10 — BUG! 0 là falsy
console.log(count ?? 10);  // 0  — CORRECT! 0 không phải null/undefined

const name = "";
console.log(name || "default");  // "default" — có thể không mong muốn
console.log(name ?? "default");  // "" — giữ string rỗng

// Nullish assignment
let config = null;
config ??= { theme: "dark" }; // chỉ gán nếu null/undefined
console.log(config); // { theme: "dark" }
```

**Template Literals:**

```js
const name = "Alice";
const age = 30;

// Multiline + expression
const message = `
  Hello, ${name}!
  You are ${age > 18 ? "an adult" : "a minor"}.
  Next year you'll be ${age + 1}.
`;

// Tagged templates
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i] !== undefined
      ? `<strong>${values[i]}</strong>`
      : "";
    return result + str + value;
  }, "");
}

const output = highlight`Hello ${name}, you are ${age} years old.`;
// "Hello <strong>Alice</strong>, you are <strong>30</strong> years old."
```

---

## 10. Scope Chain & Lexical Scope

### Q13 — [Junior]

**EN:** What is lexical scope and scope chain? How does JavaScript resolve variable names?

**VI:** Lexical scope và scope chain là gì? JavaScript resolve tên biến như thế nào?

**Trả lời:**

**Lexical Scope** (còn gọi là static scope): scope của một biến được xác định bởi **vị trí khai báo** trong source code, không phải bởi luồng thực thi runtime.

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";

    // inner có thể thấy: innerVar, outerVar, globalVar
    console.log(innerVar);  // "inner" — own scope
    console.log(outerVar);  // "outer" — parent scope (closure)
    console.log(globalVar); // "global" — global scope
  }

  // outer KHÔNG thể thấy innerVar
  console.log(innerVar); // ReferenceError

  inner();
}
```

**Scope Chain — cơ chế lookup:**

Khi JS tìm kiếm một biến, nó đi theo chuỗi: `current scope → parent scope → ... → global scope → ReferenceError`

```js
let x = 1; // global

function a() {
  let x = 2; // scope của a

  function b() {
    let x = 3; // scope của b
    console.log(x); // 3 — tìm thấy ngay trong scope hiện tại, dừng lại
  }

  function c() {
    // không có x riêng
    console.log(x); // 2 — leo lên scope của a, tìm thấy
  }

  b(); // 3
  c(); // 2
}

a();
console.log(x); // 1
```

**Lexical scope xác định lúc định nghĩa, không phải lúc gọi:**

```js
const value = "global";

function createFn() {
  const value = "local";
  return function() {
    console.log(value); // "local" — scope của hàm bên trong được xác định
                        // tại nơi nó được ĐỊNH NGHĨA (trong createFn)
  };
}

function execute(fn) {
  const value = "execute"; // scope này không ảnh hưởng
  fn(); // gọi ở đây nhưng scope đã được lock khi định nghĩa
}

const fn = createFn();
execute(fn); // "local" — không phải "execute" hay "global"
```

---

## 11. Memory Leaks

### Q14 — [Senior]

**EN:** What causes memory leaks in JavaScript? How do you detect and fix them?

**VI:** Nguyên nhân gây memory leak trong JavaScript là gì? Cách phát hiện và khắc phục?

**Trả lời:**

Memory leak xảy ra khi bộ nhớ được cấp phát nhưng không bao giờ được giải phóng, dù không còn cần thiết nữa.

**Nguyên nhân 1 — Event listeners không được remove:**

```js
// LEAK
class Component {
  constructor() {
    // Mỗi lần tạo component, thêm listener — không bao giờ xóa
    window.addEventListener("resize", this.handleResize);
    document.addEventListener("keydown", this.handleKeyDown);
  }

  handleResize = () => { /* ... */ }
  handleKeyDown = () => { /* ... */ }
}

// FIX
class Component {
  constructor() {
    this.handleResize = this.handleResize.bind(this); // giữ reference để remove
    window.addEventListener("resize", this.handleResize);
  }

  destroy() {
    window.removeEventListener("resize", this.handleResize); // cleanup!
  }
}

// React: dùng useEffect cleanup
useEffect(() => {
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize); // cleanup
}, []);
```

**Nguyên nhân 2 — setInterval không clear:**

```js
// LEAK — interval tiếp tục chạy dù component đã unmount
function startPolling() {
  setInterval(() => {
    fetchLatestData().then(update);
  }, 5000);
}

// FIX
function startPolling() {
  const intervalId = setInterval(() => {
    fetchLatestData().then(update);
  }, 5000);

  return () => clearInterval(intervalId); // trả về cleanup function
}

// React
useEffect(() => {
  const id = setInterval(fetchData, 5000);
  return () => clearInterval(id);
}, []);
```

**Nguyên nhân 3 — Async operations sau khi component unmount:**

```js
// LEAK — setState sau unmount (React warning)
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(data => {
      setUser(data); // có thể chạy sau khi component đã unmount
    });
  }, [userId]);
}

// FIX — dùng AbortController
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    fetchUser(userId, { signal: controller.signal })
      .then(data => setUser(data))
      .catch(err => {
        if (err.name !== "AbortError") throw err;
      });

    return () => controller.abort(); // cleanup — hủy request
  }, [userId]);
}
```

**Nguyên nhân 4 — Closures giữ reference không cần thiết:**

```js
// LEAK — closure giữ toàn bộ largeData trong bộ nhớ
function processData(largeData) {
  const processed = largeData.map(item => item.value); // chỉ cần values

  return function() {
    // closure này giữ reference đến cả largeData lẫn processed
    // dù chỉ dùng processed
    return processed.reduce((a, b) => a + b, 0);
    // largeData vẫn trong bộ nhớ vì closure scope
  };
}

// FIX — dùng WeakRef hoặc nullify reference
function processDataFixed(largeData) {
  const processed = largeData.map(item => item.value);
  largeData = null; // giải phóng reference — GC có thể thu hồi

  return function() {
    return processed.reduce((a, b) => a + b, 0);
  };
}
```

**WeakMap / WeakRef cho cache không gây leak:**

```js
// Map thông thường — giữ strong reference, object không bị GC
const cache = new Map();
cache.set(domNode, computedData);
// Dù domNode bị xóa khỏi DOM, nó vẫn còn trong bộ nhớ vì Map giữ reference

// WeakMap — weak reference, không ngăn GC
const weakCache = new WeakMap();
weakCache.set(domNode, computedData);
// Khi domNode bị xóa khỏi DOM và không còn reference nào khác,
// GC tự động thu hồi cả key lẫn value

// WeakRef — tham chiếu yếu, cho phép GC
let heavyObject = new WeakRef(createHeavyObject());

function getHeavyObject() {
  const obj = heavyObject.deref(); // lấy object nếu còn tồn tại
  if (obj) return obj;

  // Object đã bị GC, tạo lại
  heavyObject = new WeakRef(createHeavyObject());
  return heavyObject.deref();
}
```

**Phát hiện memory leak:**

```
1. Chrome DevTools → Memory tab → Take heap snapshots
2. So sánh snapshot trước/sau action → xem retained size tăng bất thường
3. Performance tab → record → kiểm tra memory graph có tăng liên tục không
4. Node.js: --expose-gc flag + global.gc() để force GC trước khi measure
```

---

## 12. Generators & Iterators

### Q15 — [Senior]

**EN:** What are generators and iterators? What are their practical use cases?

**VI:** Generator và Iterator là gì? Các use case thực tế là gì?

**Trả lời:**

**Iterator Protocol:**

Một object là *iterable* nếu implement `Symbol.iterator` trả về iterator. Một object là *iterator* nếu có method `next()` trả về `{ value, done }`.

```js
// Custom iterator
function range(start, end, step = 1) {
  return {
    [Symbol.iterator]() {
      let current = start;
      return {
        next() {
          if (current <= end) {
            const value = current;
            current += step;
            return { value, done: false };
          }
          return { value: undefined, done: true };
        }
      };
    }
  };
}

for (const num of range(1, 10, 2)) {
  console.log(num); // 1, 3, 5, 7, 9
}

console.log([...range(1, 5)]); // [1, 2, 3, 4, 5]
```

**Generator Function:**

Generator là cú pháp ngắn gọn hơn để tạo iterator, sử dụng `function*` và `yield`.

```js
function* rangeGenerator(start, end, step = 1) {
  for (let i = start; i <= end; i += step) {
    yield i; // tạm dừng và trả về giá trị
  }
}

const gen = rangeGenerator(1, 5);
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log([...gen]);   // [3, 4, 5] — tiếp tục từ chỗ dừng

// Infinite generator — không tốn memory
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function take(n, iterable) {
  const result = [];
  for (const value of iterable) {
    result.push(value);
    if (result.length >= n) break;
  }
  return result;
}

console.log(take(10, fibonacci())); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

**Thực tế — Lazy data processing (tiết kiệm memory):**

```js
// Xử lý 1 triệu records mà không load tất cả vào memory
function* processCSV(csvText) {
  const lines = csvText.split("\n");
  for (const line of lines) {
    if (!line.trim()) continue;
    const [id, name, score] = line.split(",");
    yield { id: Number(id), name, score: Number(score) };
  }
}

function* filter(predicate, iterable) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

function* map(transform, iterable) {
  for (const item of iterable) {
    yield transform(item);
  }
}

// Lazy pipeline — chỉ xử lý từng item khi cần, không load tất cả
const pipeline = map(
  user => user.name,
  filter(
    user => user.score > 80,
    processCSV(largeCSVData)
  )
);

for (const name of pipeline) {
  console.log(name); // xử lý từng item
}
```

**Generator cho async flow control:**

```js
// Ý tưởng đằng sau async/await (dùng generator)
function* asyncFlow() {
  const user = yield fetchUser(1);    // pause, chờ promise
  const posts = yield fetchPosts(user.id);
  return posts;
}

// Redux-Saga dùng generator để manage side effects
function* fetchUserSaga(action) {
  try {
    yield put(setLoading(true));
    const user = yield call(fetchUser, action.userId);
    yield put(setUser(user));
  } catch (error) {
    yield put(setError(error.message));
  } finally {
    yield put(setLoading(false));
  }
}
```

---

## 13. Proxy & Reflect

### Q16 — [Senior]

**EN:** What are `Proxy` and `Reflect`? Give practical use cases.

**VI:** `Proxy` và `Reflect` là gì? Cho ví dụ thực tế.

**Trả lời:**

**`Proxy`** cho phép bạn đặt "bẫy" (trap) cho các thao tác cơ bản trên object như đọc, ghi, xóa property.

```js
const proxy = new Proxy(target, handler);
// target: object gốc
// handler: object chứa các trap (get, set, has, deleteProperty, apply, ...)
```

**Use case 1 — Validation:**

```js
function createValidatedUser(data) {
  return new Proxy(data, {
    set(target, prop, value) {
      if (prop === "age") {
        if (typeof value !== "number") throw new TypeError("age must be a number");
        if (value < 0 || value > 150) throw new RangeError("age out of range");
      }
      if (prop === "email" && !value.includes("@")) {
        throw new Error("Invalid email");
      }
      target[prop] = value;
      return true; // bắt buộc trả về true trong set trap
    }
  });
}

const user = createValidatedUser({ name: "Alice" });
user.age = 25;      // OK
user.age = -1;      // RangeError
user.email = "abc"; // Error: Invalid email
```

**Use case 2 — Reactive data (Vue 3 Reactivity System):**

```js
function reactive(obj) {
  const subscribers = new Map();

  return new Proxy(obj, {
    get(target, prop) {
      // Track dependency
      track(target, prop);
      const value = target[prop];
      // Nếu value là object, wrap nó trong reactive cũng
      return typeof value === "object" ? reactive(value) : value;
    },

    set(target, prop, value) {
      target[prop] = value;
      // Trigger updates
      trigger(target, prop);
      return true;
    }
  });
}
```

**Use case 3 — Logging / Debugging:**

```js
function createLoggingProxy(obj, name) {
  return new Proxy(obj, {
    get(target, prop) {
      const value = target[prop];
      console.log(`[GET] ${name}.${String(prop)} = ${JSON.stringify(value)}`);
      return typeof value === "function" ? value.bind(target) : value;
    },

    set(target, prop, value) {
      console.log(`[SET] ${name}.${String(prop)} = ${JSON.stringify(value)}`);
      target[prop] = value;
      return true;
    }
  });
}

const state = createLoggingProxy({ count: 0 }, "state");
state.count;      // [GET] state.count = 0
state.count = 5;  // [SET] state.count = 5
```

**`Reflect`** — mirror của Proxy traps, cung cấp default behavior:

```js
// Reflect methods tương ứng với Proxy traps
Reflect.get(target, prop, receiver)     // target[prop]
Reflect.set(target, prop, value, receiver) // target[prop] = value
Reflect.has(target, prop)               // prop in target
Reflect.deleteProperty(target, prop)    // delete target[prop]
Reflect.ownKeys(target)                 // Object.getOwnPropertyNames + Symbols

// Dùng Reflect trong Proxy để giữ default behavior
const proxy = new Proxy(target, {
  get(target, prop, receiver) {
    console.log(`Accessing ${String(prop)}`);
    return Reflect.get(target, prop, receiver); // đúng với getter/inheritance
  },

  set(target, prop, value, receiver) {
    console.log(`Setting ${String(prop)}`);
    return Reflect.set(target, prop, value, receiver); // đúng với setter
  }
});
```

---

## 14. Map vs WeakMap, Set vs WeakSet

### Q17 — [Mid]

**EN:** Explain `Map`, `WeakMap`, `Set`, and `WeakSet`. When would you use WeakMap over Map?

**VI:** Giải thích `Map`, `WeakMap`, `Set`, `WeakSet`. Khi nào dùng WeakMap thay vì Map?

**Trả lời:**

**Map vs Object:**

```js
const map = new Map();

// Map chấp nhận BẤT KỲ kiểu dữ liệu làm key (kể cả object)
map.set("string", 1);
map.set(42, "number key");
map.set({ id: 1 }, "object key");
map.set(true, "boolean key");

// Object chỉ có string hoặc Symbol làm key
// Map giữ thứ tự insertion
// Map có .size property

map.get("string"); // 1
map.has(42);       // true
map.size;          // 4

// Iteration
for (const [key, value] of map) {
  console.log(key, "->", value);
}

// Convert
const obj = Object.fromEntries(map.entries()); // chỉ hoạt động với string keys
```

**Set:**

```js
const set = new Set([1, 2, 3, 2, 1]); // loại bỏ duplicates
console.log(set.size); // 3
console.log([...set]); // [1, 2, 3]

set.add(4);
set.has(3); // true
set.delete(2);

// Remove duplicates từ array
const unique = [...new Set(array)];

// Set operations
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

const union        = new Set([...a, ...b]);          // {1,2,3,4,5,6}
const intersection = new Set([...a].filter(x => b.has(x))); // {3,4}
const difference   = new Set([...a].filter(x => !b.has(x))); // {1,2}
```

**WeakMap — weak reference, key phải là object:**

```js
const weakMap = new WeakMap();

// Key BẮT BUỘC là object (hoặc non-registered Symbol)
let domNode = document.getElementById("app");
weakMap.set(domNode, { clickCount: 0 });

// Khi domNode bị remove khỏi DOM và không còn reference nào,
// GC tự động xóa entry trong WeakMap (không gây memory leak)
domNode = null; // entry trong weakMap cũng được GC

// WeakMap KHÔNG có: .size, .keys(), .values(), .entries(), forEach()
// Không thể enumerate — vì nội dung có thể bị GC bất cứ lúc nào

// Use cases của WeakMap:
// 1. Cache metadata cho DOM nodes
const nodeMetadata = new WeakMap();

// 2. Private data cho class instances
const _private = new WeakMap();
class Person {
  constructor(name, age) {
    _private.set(this, { name, age });
  }
  getName() { return _private.get(this).name; }
  getAge()  { return _private.get(this).age; }
}

// 3. Memoization không gây leak
const memo = new WeakMap();
function process(obj) {
  if (memo.has(obj)) return memo.get(obj);
  const result = expensiveOperation(obj);
  memo.set(obj, result);
  return result;
}
```

**WeakSet:**

```js
const visitedNodes = new WeakSet();

function processTree(node) {
  if (visitedNodes.has(node)) return; // tránh vòng lặp circular reference
  visitedNodes.add(node);
  // process node...
  node.children?.forEach(processTree);
}

// Use case: đánh dấu object đã xử lý mà không gây leak
const processedRequests = new WeakSet();
function handleRequest(req) {
  if (processedRequests.has(req)) return;
  processedRequests.add(req);
  // process...
}
```

**Bảng so sánh:**

| | Map | WeakMap | Set | WeakSet |
|---|---|---|---|---|
| Key/Value type | Bất kỳ | Key phải là object | Bất kỳ | Chỉ object |
| Size property | Có | Không | Có | Không |
| Iterable | Có | Không | Có | Không |
| GC behavior | Strong ref | Weak ref | Strong ref | Weak ref |
| Use case | Dictionary, ordered data | Cache, private data | Unique values | Object marking |

---

## 15. Immutability Patterns

### Q18 — [Mid]

**EN:** What is immutability? Compare `Object.freeze`, spread operator, and `structuredClone` for achieving immutability.

**VI:** Immutability là gì? So sánh `Object.freeze`, spread operator, và `structuredClone` để đạt được immutability.

**Trả lời:**

**Immutability** là nguyên tắc không thay đổi data sau khi tạo ra — thay vào đó tạo ra bản copy mới khi cần thay đổi. Giúp code dễ predict, dễ debug, và quan trọng với React (re-render dựa trên reference equality).

**`Object.freeze` — freeze shallow:**

```js
const user = Object.freeze({
  name: "Alice",
  address: { city: "Hanoi" } // nested object KHÔNG bị freeze
});

user.name = "Bob";      // silent fail (non-strict) hoặc TypeError (strict)
console.log(user.name); // "Alice" — không thay đổi

// NHƯNG nested objects vẫn mutable!
user.address.city = "HCMC"; // thành công!
console.log(user.address.city); // "HCMC" — đã thay đổi

// Deep freeze
function deepFreeze(obj) {
  Object.getOwnPropertyNames(obj).forEach(name => {
    const value = obj[name];
    if (value && typeof value === "object") {
      deepFreeze(value);
    }
  });
  return Object.freeze(obj);
}

const frozenUser = deepFreeze({ name: "Alice", address: { city: "Hanoi" } });
frozenUser.address.city = "HCMC"; // fail silently
```

**Spread operator — Shallow clone:**

```js
const original = { name: "Alice", scores: [90, 85, 92] };

// Shallow copy — tạo object mới
const updated = { ...original, name: "Bob" };
console.log(original.name); // "Alice" — unchanged
console.log(updated.name);  // "Bob"

// NHƯNG nested là shared reference!
updated.scores.push(100);
console.log(original.scores); // [90, 85, 92, 100] — cũng bị thay đổi!

// Fix nested với spread
const safeUpdate = {
  ...original,
  scores: [...original.scores, 100] // clone nested array
};
console.log(original.scores); // [90, 85, 92] — unchanged

// Redux-style immutable update
const state = {
  users: [
    { id: 1, name: "Alice", active: true },
    { id: 2, name: "Bob",   active: false }
  ]
};

// Update user với id=2
const newState = {
  ...state,
  users: state.users.map(user =>
    user.id === 2 ? { ...user, active: true } : user
  )
};
```

**`structuredClone` — Deep clone (ES2022):**

```js
const original = {
  name: "Alice",
  scores: [90, 85, 92],
  address: { city: "Hanoi", zip: "10000" },
  date: new Date("2024-01-01"), // Date objects
  nested: { deep: { deeper: { value: 42 } } }
};

const deepCopy = structuredClone(original);

deepCopy.scores.push(100);
deepCopy.address.city = "HCMC";
deepCopy.nested.deep.deeper.value = 99;

console.log(original.scores);             // [90, 85, 92] — unchanged
console.log(original.address.city);       // "Hanoi" — unchanged
console.log(original.nested.deep.deeper.value); // 42 — unchanged

// structuredClone xử lý được:
// Date, RegExp, Map, Set, ArrayBuffer, TypedArray, Error, circular references

// structuredClone KHÔNG xử lý:
// Functions, DOM nodes, class instances (mất prototype)
const obj = { fn: () => {} };
structuredClone(obj); // TypeError — functions không thể clone

// Circular reference — OK với structuredClone
const circular = { name: "test" };
circular.self = circular;
const cloned = structuredClone(circular); // OK!
```

**So sánh các cách clone:**

```js
// JSON.parse(JSON.stringify()) — cách cũ, nhiều hạn chế
const jsonClone = JSON.parse(JSON.stringify(original));
// Mất: Date (convert to string), undefined, functions, Infinity, NaN
// Không xử lý: circular reference (throw error)

// Custom deep clone
function deepClone(obj, seen = new Map()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (seen.has(obj)) return seen.get(obj); // circular reference

  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Map) {
    const mapClone = new Map();
    seen.set(obj, mapClone);
    obj.forEach((v, k) => mapClone.set(deepClone(k, seen), deepClone(v, seen)));
    return mapClone;
  }

  const clone = Array.isArray(obj) ? [] : Object.create(Object.getPrototypeOf(obj));
  seen.set(obj, clone);
  Object.keys(obj).forEach(key => {
    clone[key] = deepClone(obj[key], seen);
  });
  return clone;
}
```

**Bảng so sánh:**

| Method | Deep? | Functions? | Class instances? | Circular ref? | Performance |
|---|---|---|---|---|---|
| `{ ...obj }` | Shallow | Giữ ref | Giữ ref | OK (giữ ref) | Nhanh |
| `Object.freeze` | Shallow | N/A | N/A | N/A | Nhanh |
| `JSON.parse(stringify)` | Deep | Mất | Mất prototype | Error | Trung bình |
| `structuredClone` | Deep | Error | Mất prototype | OK | Tốt |
| Custom `deepClone` | Deep | Tuỳ chỉnh | Giữ prototype | OK | Chậm hơn |

**Recommendation:**
- Shallow update → spread operator (`{ ...obj, key: value }`)
- Deep clone data thuần (không có function/class) → `structuredClone`
- Performance-critical + cần clone class → custom deep clone hoặc thư viện (Lodash `_.cloneDeep`)
- Prevent accidental mutation trong tests → `Object.freeze`

---

*Cập nhật lần cuối: 2026-03-25*
