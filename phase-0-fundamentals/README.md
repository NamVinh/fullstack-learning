# Phase 0 — Củng cố nền tảng

**Thời gian**: 2–3 tuần | **Mục tiêu**: Hiểu nguyên lý, không chỉ dùng được

> Format mỗi topic:
> 📖 = bài đọc | 🎥 = video | 🛠️ = exercise tự làm | ✅ = tick khi xong

---

## JavaScript/TypeScript nâng cao

---

### Event Loop, Call Stack, Microtask vs Macrotask

- [ ] Hiểu thực sự JS chạy như thế nào — không phải "async/await là magic"

**Học theo thứ tự:**

1. 📖 [JavaScript Visualized: Event Loop — Lydia Hallie](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif) — minh họa gif trực quan, đọc trước
2. 📖 [Tasks, microtasks, queues and schedules — Jake Archibald](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) — deep dive thực sự, đọc kỹ
3. 🎥 Search YouTube: **"Jake Archibald In The Loop JSConf Asia"** — 35 phút, xem sau khi đọc 2 bài trên

**Exercises:**

🛠️ **Exercise 1 — Đoán output (không được chạy code trước):**
```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output là gì? Tại sao?
```

🛠️ **Exercise 2 — Phức tạp hơn:**
```js
async function foo() {
  console.log('A');
  await Promise.resolve();
  console.log('B');
}
console.log('C');
foo();
console.log('D');
// Output: C, A, D, B — giải thích được tại sao không?
```

🛠️ **Exercise 3 — Tự implement:**
Viết hàm `sleep(ms)` trả về Promise, sau đó viết `runSequentially(tasks)` chạy array of async functions tuần tự (không phải song song).

---

### Prototype chain, `this`, Closure nâng cao

- [ ] Hiểu tại sao `this` hay bị "mất" và cách fix đúng

**Học theo thứ tự:**

1. 📖 [YDKJS — this & Object Prototypes (free)](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/README.md) — đọc Ch.1–3
2. 📖 [MDN — Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
3. 📖 [JavaScript Closures Explained — MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

**Exercises:**

🛠️ **Exercise 1 — Implement `new` từ đầu:**
```js
function myNew(Constructor, ...args) {
  // Implement: tạo object, gán prototype, gọi constructor, return
}
// Test: myNew(Person, 'Nam') phải hoạt động như new Person('Nam')
```

🛠️ **Exercise 2 — Implement `bind` từ đầu:**
```js
Function.prototype.myBind = function(context, ...args) {
  // Implement
}
```

🛠️ **Exercise 3 — Closure thực tế:**
Viết hàm `memoize(fn)` nhận vào 1 function và trả về phiên bản cache kết quả. Dùng closure, không dùng thư viện ngoài.

---

### TypeScript: Generic, Mapped Types, Conditional Types

- [ ] Không chỉ dùng được `<T>` cơ bản — phải viết được utility types phức tạp

**Học theo thứ tự:**

1. 📖 [TypeScript Handbook — Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
2. 📖 [TypeScript Handbook — Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
3. 📖 [TypeScript Handbook — Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
4. 🛠️ Practice: [type-challenges trên GitHub](https://github.com/type-challenges/type-challenges) — làm bài Easy trước (Pick, Readonly, TupleToObject...)

**Exercises (từ type-challenges):**

🛠️ Implement lại các built-in Utility Types từ đầu, không nhìn source:
```ts
// Tự implement:
type MyPick<T, K extends keyof T> = ???
type MyReadonly<T> = ???
type MyPartial<T> = ???
type MyRequired<T> = ???
type MyReturnType<T> = ???
type MyParameters<T> = ???
```

🛠️ Conditional type thực tế:
```ts
// Viết type IsArray<T> = true nếu T là array, false nếu không
// Viết type UnwrapPromise<T> — nếu T là Promise<X> thì trả về X, không thì T
type UnwrapPromise<T> = ???
```

---

### TypeScript: Decorators

- [ ] Cần cho NestJS ở Phase 5 — hiểu nguyên lý trước khi dùng framework

**Học theo thứ tự:**

1. 📖 [TypeScript Decorators — Official Handbook](https://www.typescriptlang.org/docs/handbook/decorators.html)
2. 📖 [A practical guide to TypeScript decorators](https://blog.logrocket.com/practical-guide-typescript-decorators/)

**Exercises:**

🛠️ Viết `@Log` class method decorator in chắc ra console mỗi lần method được gọi (tên method + arguments):
```ts
class Calculator {
  @Log
  add(a: number, b: number) { return a + b; }
}
new Calculator().add(1, 2); // log: "add called with [1, 2]"
```

🛠️ Viết `@Validate` decorator kiểm tra arguments không phải `null/undefined` trước khi chạy method.

---

### Module system: CommonJS vs ESM

- [ ] Quan trọng khi setup Node.js project — sai module system gây lỗi khó debug

**Học theo thứ tự:**

1. 📖 [Node.js Docs — Modules: CommonJS](https://nodejs.org/api/modules.html)
2. 📖 [Node.js Docs — Modules: ECMAScript](https://nodejs.org/api/esm.html)
3. 📖 [CommonJS vs ESM — tại sao nó quan trọng](https://redfin.engineering/node-modules-at-war-why-commonjs-and-es-modules-cant-get-along-9617135eeca1)

**Exercises:**

🛠️ Tạo 2 file:
- `math.cjs` — export theo CommonJS: `module.exports = { add, multiply }`
- `math.mjs` — export theo ESM: `export const add = ...`

Thử import lẫn nhau và quan sát lỗi. Giải thích tại sao xảy ra.

🛠️ Trong `package.json`, thêm `"type": "module"` và quan sát code CJS cũ bị lỗi như thế nào. Fix lại.

---

## Networking & Web cơ bản

---

### HTTP/HTTPS: Request/Response cycle

- [ ] Fullstack dev phải hiểu HTTP như hiểu tiếng mẹ đẻ

**Học theo thứ tự:**

1. 📖 [MDN — An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
2. 📖 [MDN — HTTP Messages](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages)
3. 📖 [HTTP Status Codes — tất cả và khi nào dùng](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

**Exercises:**

🛠️ Dùng `curl` (không phải Postman) để:
```bash
# 1. GET request và xem headers
curl -I https://jsonplaceholder.typicode.com/posts/1

# 2. POST với JSON body
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"test","body":"hello","userId":1}'

# 3. Xem cả request headers
curl -v https://example.com 2>&1 | head -50
```

🛠️ Giải thích sự khác nhau giữa 200, 201, 204, 301, 302, 400, 401, 403, 404, 409, 422, 429, 500, 503. Viết ra use case của từng cái.

---

### REST nguyên lý

- [ ] Hiểu constraints của REST — không phải "có dùng JSON là REST"

**Học theo thứ tự:**

1. 📖 [RESTful Web Services Cookbook — Chapter 1 (tóm tắt)](https://www.oreilly.com/library/view/restful-web-services/9780596809140/) hoặc tìm summary online
2. 📖 [REST API Design Best Practices](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
3. 📖 [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html) — Level 0 đến Level 3

**Exercises:**

🛠️ Thiết kế REST API cho một ứng dụng Twitter đơn giản (tweet, follow, like). Viết ra:
- Endpoints (URL + method)
- Tại sao chọn route name đó
- Status code nào trả về khi nào

🛠️ Critique API design này — cái gì sai và tại sao:
```
GET  /getUsers
POST /createUser
GET  /getUserById?id=123
POST /deleteUser/123
GET  /user/123/getUserPosts
```

---

### Cookie, Session, JWT

- [ ] Hiểu để implement auth đúng — không copy-paste stack overflow

**Học theo thứ tự:**

1. 📖 [MDN — HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
2. 📖 [JWT.io Introduction](https://jwt.io/introduction)
3. 📖 [Session vs Token Authentication in 100 seconds](https://dev.to/thecodearcher/what-really-is-the-difference-between-session-and-token-based-authentication-2o39)

**Exercises:**

🛠️ Decode JWT thủ công (không dùng library):
```js
// JWT có 3 phần, mỗi phần base64url encoded
const token = "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0In0.xxx";
const [header, payload, sig] = token.split('.');
console.log(JSON.parse(atob(header)));   // Decode header
console.log(JSON.parse(atob(payload)));  // Decode payload
// Signature thì không decode được — tại sao?
```

🛠️ So sánh: Vẽ sơ đồ flow cho session-based auth và JWT-based auth. Khi nào dùng cái nào?

---

### DNS, TCP/IP, HTTPS/TLS cơ bản

- [ ] Hiểu request đi từ browser đến server như thế nào

**Học theo thứ tự:**

1. 📖 [How DNS Works — Comic (howdns.works)](https://howdns.works/) — đọc hết 6 phần
2. 📖 [What happens when you type google.com](https://github.com/alex/what-happens-when) — đọc phần DNS + TCP + TLS
3. 📖 [TLS/SSL explained simply — Cloudflare](https://www.cloudflare.com/learning/ssl/what-is-ssl/)

**Exercises:**

🛠️ Chạy và đọc output:
```bash
# Trace DNS resolution
dig google.com +trace

# Xem TLS certificate
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -text | head -40

# traceroute — xem packet đi qua những hop nào
traceroute google.com
```

🛠️ Giải thích bằng lời: Khi bạn gõ `https://github.com` và nhấn Enter, điều gì xảy ra từng bước (DNS → TCP → TLS → HTTP)?

---

## Git workflow nâng cao

---

### Branching Strategy & Conventional Commits

- [ ] Làm việc team mà không biết cái này rất dễ gây chaos

**Học theo thứ tự:**

1. 📖 [Atlassian — Git Flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
2. 📖 [Trunk Based Development](https://trunkbaseddevelopment.com/) — đọc phần "What is it?"
3. 📖 [Conventional Commits spec](https://www.conventionalcommits.org/en/v1.0.0/)

**Exercises:**

🛠️ Setup commit-msg hook để enforce conventional commits:
```bash
# Tạo file .git/hooks/commit-msg
cat > .git/hooks/commit-msg << 'EOF'
#!/bin/sh
msg=$(cat "$1")
pattern='^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+'
if ! echo "$msg" | grep -qE "$pattern"; then
  echo "❌ Commit message không đúng format Conventional Commits"
  echo "   Ví dụ: feat(auth): add JWT login"
  exit 1
fi
EOF
chmod +x .git/hooks/commit-msg
```

🛠️ Thực hành rebase:
```bash
git checkout -b feature/test
# Tạo 3 commits rác
git commit --allow-empty -m "wip"
git commit --allow-empty -m "fix"
git commit --allow-empty -m "done"
# Squash 3 commits thành 1 với rebase interactive
git rebase -i HEAD~3
```

---

## Project Phase 0

**Mục tiêu**: Áp dụng tất cả TypeScript advanced vào 1 bài thực tế.

Viết một module `http-client.ts` — một HTTP client nhỏ với TypeScript thuần, không dùng axios/fetch wrapper:

```ts
// Yêu cầu:
// 1. Generic type cho response: get<User>('/api/user/1')
// 2. Interceptor pattern (như axios.interceptors)
// 3. Retry logic với exponential backoff
// 4. Type-safe error handling
// 5. Không có bất kỳ 'any' nào
```

Commit theo Conventional Commits. Push lên GitHub với PR description đàng hoàng.
