# Frontend Interview Questions — Junior to Senior
> Bilingual: English / Tiếng Việt | Vulcan Labs prep

---

## Table of Contents
1. [JavaScript](#javascript)
2. [TypeScript](#typescript)
3. [React](#react)
4. [Next.js](#nextjs)
5. [State Management](#state-management)

---

# JAVASCRIPT

---

### 1. [Junior] var / let / const — Sự khác biệt là gì?

**EN:** What are the differences between `var`, `let`, and `const`?

**VI:** Sự khác biệt giữa `var`, `let`, và `const` là gì?

**Trả lời:**

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function scope | Block scope | Block scope |
| Hoisting | Có (undefined) | Có (TDZ) | Có (TDZ) |
| Re-declare | Được | Không | Không |
| Re-assign | Được | Được | Không |

```javascript
// var — function scoped, bị hoisting
function example() {
  console.log(x); // undefined (không lỗi vì hoisting)
  var x = 10;
  if (true) {
    var x = 20; // cùng variable! override
  }
  console.log(x); // 20
}

// let/const — block scoped
function example2() {
  // console.log(y); // ReferenceError: Temporal Dead Zone
  let y = 10;
  if (true) {
    let y = 20; // variable khác, chỉ sống trong block này
    console.log(y); // 20
  }
  console.log(y); // 10
}

// const — không re-assign, nhưng object/array vẫn mutate được
const user = { name: "Vinh" };
user.name = "Nam"; // OK — mutate property
user = {}; // TypeError — không re-assign reference
```

**Temporal Dead Zone (TDZ):** Khoảng thời gian từ khi vào scope đến khi `let`/`const` được khai báo — truy cập trong khoảng này sẽ throw `ReferenceError`.

---

### 2. [Junior] Hoisting là gì?

**EN:** What is hoisting in JavaScript?

**VI:** Hoisting trong JavaScript là gì?

**Trả lời:**

Hoisting là cơ chế JavaScript **đưa khai báo** (declarations) lên đầu scope trước khi thực thi. Chỉ khai báo được hoist, **không phải giá trị**.

```javascript
// Function declaration — hoist hoàn toàn (cả body)
sayHello(); // "Hello!" — hoạt động bình thường

function sayHello() {
  console.log("Hello!");
}

// Function expression — chỉ hoist khai báo biến
greet(); // TypeError: greet is not a function

var greet = function() {
  console.log("Hi!");
};

// var — hoist khai báo, giá trị là undefined
console.log(a); // undefined
var a = 5;
console.log(a); // 5

// let/const — hoist nhưng vào TDZ, không dùng được
console.log(b); // ReferenceError
let b = 5;

// Class — cũng có TDZ như let/const
const obj = new MyClass(); // ReferenceError
class MyClass {}
```

---

### 3. [Mid] Closure là gì?

**EN:** What is a closure? Give a real-world example.

**VI:** Closure là gì? Cho ví dụ thực tế.

**Trả lời:**

Closure là function có khả năng **nhớ và truy cập** lexical scope của nó ngay cả khi function đó được thực thi bên ngoài scope đó.

```javascript
// Ví dụ cơ bản:
function makeCounter(initialValue = 0) {
  let count = initialValue; // biến này được "đóng gói" vào closure

  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
    reset: () => { count = initialValue; }
  };
}

const counter = makeCounter(10);
counter.increment(); // 11
counter.increment(); // 12
counter.decrement(); // 11
console.log(counter.getCount()); // 11
// Không thể truy cập `count` trực tiếp từ ngoài — encapsulation!

// Ví dụ thực tế — debounce function:
function debounce(fn, delay) {
  let timeoutId; // closure giữ timeoutId giữa các lần gọi

  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

const searchHandler = debounce((query) => {
  console.log(`Search: ${query}`);
}, 300);

// Lỗi phổ biến với closure trong loop:
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // In ra 3, 3, 3 — không phải 0, 1, 2!
}
// Vì var — i là cùng 1 biến, khi setTimeout chạy i đã là 3

// Fix 1 — dùng let:
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2 ✅
}

// Fix 2 — IIFE tạo scope mới:
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i); // 0, 1, 2 ✅
}
```

---

### 4. [Senior] Event Loop — Call Stack, Microtask, Macrotask

**EN:** Explain the JavaScript Event Loop. What is the difference between microtasks and macrotasks?

**VI:** Giải thích Event Loop. Sự khác biệt giữa microtask và macrotask?

**Trả lời:**

```javascript
// Thứ tự thực thi:
// 1. Call Stack (synchronous)
// 2. Microtask Queue (Promise callbacks, queueMicrotask, MutationObserver)
// 3. Macrotask Queue (setTimeout, setInterval, setImmediate, I/O)

console.log("1 — sync");

setTimeout(() => console.log("2 — macrotask"), 0);

Promise.resolve()
  .then(() => console.log("3 — microtask 1"))
  .then(() => console.log("4 — microtask 2"));

queueMicrotask(() => console.log("5 — microtask 3"));

console.log("6 — sync");

// Output: 1, 6, 3, 5, 4, 2
// Giải thích:
// - "1" và "6": sync code chạy trước
// - "3", "5", "4": microtasks chạy hết trước khi sang macrotask
//   (Note: "4" sau "5" vì "4" là .then thứ 2, queue sau "5")
// - "2": macrotask cuối cùng

// Ví dụ phức tạp hơn:
console.log("start");

setTimeout(() => {
  console.log("setTimeout 1");
  Promise.resolve().then(() => console.log("promise inside setTimeout"));
}, 0);

Promise.resolve().then(() => {
  console.log("promise 1");
  setTimeout(() => console.log("setTimeout inside promise"), 0);
});

console.log("end");

// Output:
// start
// end
// promise 1         ← microtask
// setTimeout 1      ← macrotask 1
// promise inside setTimeout  ← microtask (drains sau mỗi macrotask)
// setTimeout inside promise  ← macrotask 2
```

**Key rule:** Sau mỗi macrotask, JS engine drain **toàn bộ** microtask queue trước khi lấy macrotask tiếp theo.

---

### 5. [Mid] Prototype Chain

**EN:** Explain prototype chain and prototypal inheritance.

**VI:** Giải thích prototype chain và prototypal inheritance.

**Trả lời:**

```javascript
// Mọi object trong JS đều có [[Prototype]] link đến object khác
// Khi access property, JS tìm theo chain: object → prototype → prototype → null

const animal = {
  breathe() { return "breathing"; }
};

const dog = Object.create(animal); // dog.__proto__ === animal
dog.bark = function() { return "woof"; };

const myDog = Object.create(dog);
myDog.name = "Rex";

console.log(myDog.name);     // "Rex" — own property
console.log(myDog.bark());   // "woof" — from dog prototype
console.log(myDog.breathe()); // "breathing" — from animal prototype
console.log(myDog.toString()); // từ Object.prototype

// Class syntax — syntactic sugar cho prototype
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() { return `${this.name} makes a sound`; }
}

class Dog extends Animal {
  bark() { return `${this.name} barks`; }
}

const rex = new Dog("Rex");
rex.bark();   // "Rex barks"
rex.speak();  // "Rex makes a sound" — inherited

// Kiểm tra:
console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
console.log(Object.getPrototypeOf(rex) === Dog.prototype); // true

// hasOwnProperty — chỉ kiểm tra own properties, không leo chain
console.log(rex.hasOwnProperty("name")); // true
console.log(rex.hasOwnProperty("bark")); // false — bark là trên prototype
```

---

### 6. [Mid] this — Binding Rules

**EN:** How does `this` work in JavaScript? What are the binding rules?

**VI:** `this` trong JavaScript hoạt động như thế nào? Các quy tắc binding?

**Trả lời:**

```javascript
// 4 quy tắc binding (theo thứ tự ưu tiên):

// 1. Explicit binding — call, apply, bind (ưu tiên cao nhất sau new)
function greet() { return `Hello, ${this.name}`; }
const user = { name: "Vinh" };

greet.call(user);            // "Hello, Vinh"
greet.apply(user);           // "Hello, Vinh"
const boundGreet = greet.bind(user);
boundGreet();                // "Hello, Vinh"

// 2. Implicit binding — object method call
const obj = {
  name: "Vinh",
  greet() { return `Hello, ${this.name}`; }
};
obj.greet(); // "Hello, Vinh" — this = obj

// Mất binding khi tách khỏi object:
const fn = obj.greet;
fn(); // "Hello, undefined" — this = global/undefined (strict mode)

// 3. Default binding — standalone call
function show() { console.log(this); }
show(); // global object (window) / undefined (strict mode)

// 4. new binding
function Person(name) {
  this.name = name; // this = object mới tạo
}
const p = new Person("Vinh"); // p.name === "Vinh"

// Arrow function — KHÔNG có this riêng, kế thừa từ lexical scope
const timer = {
  name: "timer",
  start() {
    // Regular function — this bị mất:
    setTimeout(function() {
      console.log(this.name); // undefined
    }, 100);

    // Arrow function — giữ this từ start():
    setTimeout(() => {
      console.log(this.name); // "timer" ✅
    }, 100);
  }
};

// React component — lý do bind trong constructor hoặc dùng arrow:
class Button extends React.Component {
  handleClick() {
    console.log(this); // undefined nếu không bind!
  }

  handleClickArrow = () => {
    console.log(this); // component instance ✅
  }
}
```

---

### 7. [Mid] Promise, async/await, Error Handling

**EN:** Explain Promise and async/await. How do you handle errors properly?

**VI:** Giải thích Promise và async/await. Cách xử lý lỗi đúng?

**Trả lời:**

```javascript
// Promise — đại diện cho giá trị sẽ có trong tương lai
// 3 states: pending → fulfilled | rejected

// Promise.all — chạy song song, fail nếu 1 cái fail
const [users, products, orders] = await Promise.all([
  fetchUsers(),
  fetchProducts(),
  fetchOrders()
]);

// Promise.allSettled — chờ tất cả, không fail khi có lỗi
const results = await Promise.allSettled([
  fetchUsers(),
  fetchProducts(),
  fetchOrders()
]);
results.forEach(result => {
  if (result.status === "fulfilled") console.log(result.value);
  else console.log("Error:", result.reason);
});

// Promise.race — lấy cái resolve/reject đầu tiên
const withTimeout = (promise, ms) =>
  Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error("Timeout")), ms)
    )
  ]);

// Promise.any — lấy cái fulfilled đầu tiên (ES2021)
const fastestServer = await Promise.any([
  fetch("https://server1.com/api"),
  fetch("https://server2.com/api"),
  fetch("https://server3.com/api")
]);

// Error handling patterns:

// 1. try/catch với async/await
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    if (error.name === "AbortError") throw error; // re-throw abort
    console.error("fetchUser failed:", error);
    return null; // fallback
  }
}

// 2. Result type pattern — không throw, return error
async function safeApiCall(fn) {
  try {
    const data = await fn();
    return { data, error: null };
  } catch (error) {
    return { data: null, error };
  }
}

const { data, error } = await safeApiCall(() => fetchUser(id));
if (error) { /* handle */ }
else { /* use data */ }

// 3. Parallel với error handling cá nhân
const [usersResult, productsResult] = await Promise.allSettled([
  fetchUsers(),
  fetchProducts()
]);
```

---

### 8. [Mid] Debounce vs Throttle

**EN:** What is the difference between debounce and throttle? When to use each?

**VI:** Sự khác biệt giữa debounce và throttle? Khi nào dùng cái nào?

**Trả lời:**

```javascript
// DEBOUNCE — delay thực thi đến khi không gọi nữa trong X ms
// Use case: search input, form validation, window resize

function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

// User gõ "react" từng ký tự → chỉ search sau khi dừng gõ 300ms
const handleSearch = debounce((query) => {
  console.log("Searching:", query);
}, 300);

input.addEventListener("input", (e) => handleSearch(e.target.value));

// THROTTLE — đảm bảo function chỉ chạy tối đa 1 lần trong X ms
// Use case: scroll handler, mousemove, API rate limiting

function throttle(fn, limit) {
  let inThrottle = false;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Scroll event — chỉ handle 1 lần mỗi 100ms dù scroll liên tục
const handleScroll = throttle(() => {
  console.log("Scroll position:", window.scrollY);
}, 100);

window.addEventListener("scroll", handleScroll);

// PHÂN BIỆT:
// Debounce: "Chờ tôi dừng rồi mới làm" — search, validation
// Throttle: "Làm đều đặn, không quá X lần/giây" — scroll, resize, game loop
```

---

### 9. [Senior] Memory Leaks trong Frontend

**EN:** What causes memory leaks in frontend JavaScript? How do you detect and fix them?

**VI:** Điều gì gây ra memory leak trong frontend? Cách phát hiện và fix?

**Trả lời:**

```javascript
// 1. Event listeners không được remove
class Component {
  constructor() {
    // LEAK: listener không bao giờ bị remove
    window.addEventListener("resize", this.handleResize);
  }
  // Fix:
  destroy() {
    window.removeEventListener("resize", this.handleResize);
  }
}

// React — cleanup trong useEffect:
useEffect(() => {
  const handler = () => setWidth(window.innerWidth);
  window.addEventListener("resize", handler);
  return () => window.removeEventListener("resize", handler); // cleanup!
}, []);

// 2. setInterval / setTimeout không clear
useEffect(() => {
  const id = setInterval(() => {
    fetchData(); // LEAK: chạy mãi dù component unmounted
  }, 5000);
  return () => clearInterval(id); // fix
}, []);

// 3. Async operations sau unmount
useEffect(() => {
  let cancelled = false;

  async function load() {
    const data = await fetchData();
    if (!cancelled) setState(data); // check trước khi set state
  }

  load();
  return () => { cancelled = true; };
}, []);

// Hoặc dùng AbortController:
useEffect(() => {
  const controller = new AbortController();

  fetch("/api/data", { signal: controller.signal })
    .then(res => res.json())
    .then(data => setState(data))
    .catch(err => {
      if (err.name !== "AbortError") console.error(err);
    });

  return () => controller.abort();
}, []);

// 4. Closure giữ reference lớn
function createLeak() {
  const hugeArray = new Array(1000000).fill("data");

  return function() {
    // Closure này giữ hugeArray sống mãi
    return hugeArray[0];
  };
}

// 5. WeakMap/WeakRef — cho phép GC thu dọn
const cache = new WeakMap(); // key là object, GC được khi object không có reference

const userRef = new WeakRef(expensiveUserObject);
// Sau này:
const user = userRef.deref(); // null nếu đã bị GC

// Phát hiện: Chrome DevTools > Memory > Heap Snapshot, hoặc Performance Monitor
```

---

### 10. [Senior] Generator và Iterator

**EN:** What are generators and iterators? When are they useful?

**VI:** Generator và Iterator là gì? Khi nào chúng hữu ích?

**Trả lời:**

```javascript
// Iterator protocol: object có next() method trả về { value, done }
// Iterable: object có [Symbol.iterator]() method trả về iterator

// Custom iterator:
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

// Generator — function* có thể pause/resume execution
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++; // pause và trả về giá trị
  }
}

const gen = idGenerator();
gen.next().value; // 1
gen.next().value; // 2
gen.next().value; // 3

// Async generator — stream data từng phần
async function* fetchPages(url) {
  let page = 1;
  while (true) {
    const res = await fetch(`${url}?page=${page}`);
    const data = await res.json();
    if (data.items.length === 0) return;
    yield data.items;
    page++;
  }
}

// Dùng:
for await (const items of fetchPages("/api/products")) {
  processItems(items); // xử lý từng page khi load xong
}

// Use case thực tế:
// - Infinite scroll data loading
// - Streaming API responses
// - Lazy evaluation của large datasets
// - State machines
```

---

# TYPESCRIPT

---

### 1. [Junior] interface vs type

**EN:** What is the difference between `interface` and `type` in TypeScript?

**VI:** Sự khác biệt giữa `interface` và `type` trong TypeScript?

**Trả lời:**

```typescript
// INTERFACE — dùng cho object shapes
interface User {
  id: string;
  name: string;
  email?: string; // optional
}

// Extend bằng extends:
interface AdminUser extends User {
  role: "admin";
  permissions: string[];
}

// Declaration merging — tự động merge nếu cùng tên
interface Window {
  myCustomProperty: string; // thêm vào Window global
}

// TYPE — đa năng hơn
type ID = string | number; // union — interface không làm được

type Status = "active" | "inactive" | "pending"; // literal union

type Nullable<T> = T | null; // generic utility

// Intersection (tương tự extends):
type AdminUser = User & {
  role: "admin";
  permissions: string[];
};

// Conditional type — interface không làm được
type NonNullable<T> = T extends null | undefined ? never : T;

// Tuple — interface không làm được
type Point = [number, number];
type RGB = [number, number, number];

// RULE OF THUMB:
// - interface: object shapes, class contracts, public API
// - type: unions, intersections, utility types, primitives, tuples
// - Quan trọng nhất: chọn 1 và dùng nhất quán trong team
```

---

### 2. [Mid] Generics

**EN:** What are generics? Show practical examples.

**VI:** Generics là gì? Cho ví dụ thực tế.

**Trả lời:**

```typescript
// Generic — type parameter, giống như function parameter nhưng cho types

// 1. Generic function:
function identity<T>(value: T): T {
  return value;
}
identity<string>("hello"); // TypeScript biết return type là string
identity(42);              // inference: T = number

// 2. Generic interface — API response wrapper:
interface ApiResponse<T> {
  data: T;
  error: string | null;
  status: number;
  timestamp: Date;
}

type UsersResponse = ApiResponse<User[]>;
type ProductResponse = ApiResponse<Product>;

// 3. Generic với constraints:
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// T phải có id property — type-safe!
findById(users, "123");    // returns User | undefined
findById(products, "456"); // returns Product | undefined

// 4. Generic custom hook:
function useLocalStorage<T>(key: string, defaultValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? (JSON.parse(stored) as T) : defaultValue;
  });

  const setStored = (newValue: T) => {
    setValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  };

  return [value, setStored] as const;
}

const [theme, setTheme] = useLocalStorage("theme", "light");
// TypeScript biết theme là string

const [user, setUser] = useLocalStorage<User | null>("user", null);
// TypeScript biết user là User | null

// 5. Multiple type parameters:
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 } as T & U;
}

const merged = merge({ name: "Vinh" }, { age: 25 });
// merged.name và merged.age đều type-safe
```

---

### 3. [Mid] Utility Types

**EN:** Explain common utility types in TypeScript.

**VI:** Giải thích các utility types phổ biến trong TypeScript.

**Trả lời:**

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  role: "admin" | "user";
  createdAt: Date;
}

// Partial<T> — tất cả properties optional
type UpdateUserDTO = Partial<User>;
// { id?: string; name?: string; email?: string; ... }

// Required<T> — tất cả properties required
type StrictUser = Required<Partial<User>>; // ngược lại Partial

// Pick<T, K> — chọn subset của properties
type UserProfile = Pick<User, "id" | "name" | "email">;
// { id: string; name: string; email: string }

// Omit<T, K> — bỏ một số properties
type PublicUser = Omit<User, "password" | "createdAt">;
// { id: string; name: string; email: string; role: "admin" | "user" }

type CreateUserDTO = Omit<User, "id" | "createdAt">;

// Record<K, V> — object với key type K và value type V
type UserMap = Record<string, User>;
type StatusConfig = Record<"active" | "inactive" | "pending", { color: string; label: string }>;

const statusConfig: StatusConfig = {
  active: { color: "green", label: "Hoạt động" },
  inactive: { color: "gray", label: "Không hoạt động" },
  pending: { color: "yellow", label: "Chờ duyệt" }
};

// ReturnType<T> — lấy return type của function
async function fetchUser(id: string): Promise<User> { /* ... */ return {} as User; }

type FetchUserReturn = ReturnType<typeof fetchUser>; // Promise<User>
type ResolvedUser = Awaited<ReturnType<typeof fetchUser>>; // User

// Parameters<T> — lấy parameter types của function
type FetchUserParams = Parameters<typeof fetchUser>; // [id: string]

// Exclude<T, U> / Extract<T, U>
type AdminOrUser = "admin" | "user" | "guest";
type RegularUser = Exclude<AdminOrUser, "admin">; // "user" | "guest"
type OnlyAdmin = Extract<AdminOrUser, "admin" | "superadmin">; // "admin"

// NonNullable<T>
type MaybeUser = User | null | undefined;
type DefiniteUser = NonNullable<MaybeUser>; // User

// Readonly<T>
type ImmutableUser = Readonly<User>;
// Tất cả properties đều readonly — không mutate được
```

---

### 4. [Senior] Conditional Types & Mapped Types

**EN:** Explain conditional types and mapped types.

**VI:** Giải thích conditional types và mapped types.

**Trả lời:**

```typescript
// CONDITIONAL TYPES — T extends U ? X : Y

// Built-in NonNullable tự viết:
type MyNonNullable<T> = T extends null | undefined ? never : T;

// Unwrap Promise:
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;
// infer — "lấy" type từ bên trong

// Flatten array:
type Flatten<T> = T extends Array<infer Item> ? Item : T;
type StrArr = Flatten<string[]>; // string
type Num = Flatten<number>;      // number (không phải array)

// Conditional type với union (distributive):
type ToArray<T> = T extends any ? T[] : never;
type StrOrNumArr = ToArray<string | number>; // string[] | number[]

// Thực tế — deep readonly:
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// MAPPED TYPES — transform existing types

// Tự viết Partial:
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

// Tự viết Readonly:
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Nullable version của type:
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

// Getter methods từ interface:
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface User { name: string; age: number; }
type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number; }

// Form state từ data model:
type FormState<T> = {
  [P in keyof T]: {
    value: T[P];
    error: string | null;
    touched: boolean;
  };
};

type UserFormState = FormState<Pick<User, "name" | "email">>;
// {
//   name: { value: string; error: string | null; touched: boolean }
//   email: { value: string; error: string | null; touched: boolean }
// }
```

---

### 5. [Mid] unknown vs any vs never

**EN:** What is the difference between `unknown`, `any`, and `never`?

**VI:** Sự khác biệt giữa `unknown`, `any`, và `never`?

**Trả lời:**

```typescript
// any — tắt type checking hoàn toàn. Tránh dùng.
let x: any = "hello";
x.toUpperCase(); // OK
x.nonExistentMethod(); // OK — không báo lỗi nhưng runtime crash!
x = 42; // OK
const y: string = x; // OK — any "lây" sang type khác

// unknown — type-safe alternative cho any
let value: unknown = "hello";
// value.toUpperCase(); // Error — phải narrow type trước

// Phải narrow trước khi dùng:
if (typeof value === "string") {
  value.toUpperCase(); // OK
}

if (value instanceof Date) {
  value.toISOString(); // OK
}

// Type guard:
function isUser(value: unknown): value is User {
  return typeof value === "object"
    && value !== null
    && "id" in value
    && "name" in value;
}

// unknown không "lây":
const safe: string = value as string; // cần explicit assertion

// USE CASE: error handling
try {
  await fetchData();
} catch (error: unknown) { // catch clause type là unknown trong TypeScript 4+
  if (error instanceof Error) {
    console.log(error.message);
  } else if (typeof error === "string") {
    console.log(error);
  }
}

// never — type không bao giờ xảy ra
// 1. Function không bao giờ return:
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}

// 2. Exhaustive check — đảm bảo handle tất cả cases
type Shape = "circle" | "square" | "triangle";

function getArea(shape: Shape): number {
  switch (shape) {
    case "circle": return Math.PI * 5 * 5;
    case "square": return 5 * 5;
    case "triangle": return 0.5 * 5 * 5;
    default:
      const _exhaustive: never = shape; // TypeScript báo lỗi nếu có case chưa handle
      throw new Error(`Unknown shape: ${_exhaustive}`);
  }
}

// 3. never trong intersection:
type ImpossibleType = string & number; // never
```

---

### 6. [Senior] Discriminated Unions

**EN:** What are discriminated unions? Why are they useful?

**VI:** Discriminated unions là gì? Tại sao chúng hữu ích?

**Trả lời:**

```typescript
// Discriminated union — union của types có chung "discriminant" property (literal type)

type ApiState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function ProductPage() {
  const [state, setState] = useState<ApiState<Product[]>>({ status: "idle" });

  // TypeScript tự narrow type dựa trên status:
  switch (state.status) {
    case "idle":
      return <button onClick={load}>Load</button>;
    case "loading":
      return <Spinner />;
    case "success":
      return <ProductList products={state.data} />; // state.data: Product[] ✅
    case "error":
      return <Error message={state.error.message} />; // state.error: Error ✅
  }
}

// Ví dụ thực tế — payment methods:
type PaymentMethod =
  | { type: "credit_card"; cardNumber: string; expiry: string; cvv: string }
  | { type: "bank_transfer"; bankCode: string; accountNumber: string }
  | { type: "wallet"; walletId: string; provider: "momo" | "zalopay" };

function processPayment(method: PaymentMethod) {
  switch (method.type) {
    case "credit_card":
      // TypeScript biết: method.cardNumber, method.expiry, method.cvv tồn tại
      return chargeCreditCard(method.cardNumber, method.expiry, method.cvv);
    case "bank_transfer":
      return initiateBankTransfer(method.bankCode, method.accountNumber);
    case "wallet":
      return chargeWallet(method.walletId, method.provider);
  }
}

// Kết hợp với type guards:
function isSuccessState<T>(state: ApiState<T>): state is { status: "success"; data: T } {
  return state.status === "success";
}

if (isSuccessState(state)) {
  console.log(state.data); // type-safe access
}
```

---

# REACT

---

### 1. [Junior] Virtual DOM và Reconciliation

**EN:** What is the Virtual DOM? How does React's reconciliation work?

**VI:** Virtual DOM là gì? Reconciliation trong React hoạt động ra sao?

**Trả lời:**

React giữ một **Virtual DOM** — bản copy dạng JavaScript object của DOM thật. Khi state/props thay đổi:

1. React tạo **Virtual DOM mới**
2. **Diff** với Virtual DOM cũ (reconciliation)
3. Chỉ apply những thay đổi thực sự lên **DOM thật** (commit phase)

```jsx
// Tại sao key quan trọng trong list:
// Không có key → React không biết element nào thay đổi → rebuild toàn bộ list

// BAD — dùng index làm key khi list có thể reorder/delete
{items.map((item, index) => <Item key={index} item={item} />)}

// GOOD — dùng stable unique ID
{items.map(item => <Item key={item.id} item={item} />)}

// Tại sao xấu khi dùng index:
// List: [A(0), B(1), C(2)]
// Sau khi xóa A: [B(0), C(1)]
// React thấy key 0 vẫn tồn tại → update B với data của B nhưng giữ state của A (!)

// Diffing algorithm — 2 assumptions:
// 1. Elements khác type → destroy + rebuild toàn bộ subtree
// 2. key giúp identify list items

// React 18 — Concurrent Mode
// Fiber architecture cho phép:
// - Chia nhỏ render work thành units
// - Pause, resume, hay abort work
// - Ưu tiên user interactions hơn background updates
```

---

### 2. [Mid] useState vs useReducer

**EN:** When should you use `useReducer` instead of `useState`?

**VI:** Khi nào nên dùng `useReducer` thay vì `useState`?

**Trả lời:**

```jsx
// useState — đơn giản, state độc lập
const [count, setCount] = useState(0);
const [name, setName] = useState("");

// useReducer — khi state phức tạp, nhiều sub-values, logic chuyển state
// Giống Redux ở quy mô nhỏ

// Ví dụ form phức tạp:
type FormAction =
  | { type: "SET_FIELD"; field: string; value: string }
  | { type: "SET_ERROR"; field: string; error: string }
  | { type: "SUBMIT_START" }
  | { type: "SUBMIT_SUCCESS" }
  | { type: "SUBMIT_ERROR"; error: string }
  | { type: "RESET" };

interface FormState {
  values: Record<string, string>;
  errors: Record<string, string>;
  isSubmitting: boolean;
  submitError: string | null;
}

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case "SET_FIELD":
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
        errors: { ...state.errors, [action.field]: "" } // clear error khi type
      };
    case "SET_ERROR":
      return { ...state, errors: { ...state.errors, [action.field]: action.error } };
    case "SUBMIT_START":
      return { ...state, isSubmitting: true, submitError: null };
    case "SUBMIT_SUCCESS":
      return { ...state, isSubmitting: false };
    case "SUBMIT_ERROR":
      return { ...state, isSubmitting: false, submitError: action.error };
    case "RESET":
      return initialFormState;
    default:
      return state;
  }
}

function LoginForm() {
  const [state, dispatch] = useReducer(formReducer, initialFormState);

  const handleSubmit = async () => {
    dispatch({ type: "SUBMIT_START" });
    try {
      await login(state.values);
      dispatch({ type: "SUBMIT_SUCCESS" });
    } catch (err) {
      dispatch({ type: "SUBMIT_ERROR", error: err.message });
    }
  };

  return (/* ... */);
}

// RULE: dùng useReducer khi:
// - State là object với nhiều fields
// - Next state phụ thuộc vào previous state theo nhiều cách
// - Logic transition phức tạp, muốn test riêng
// - Nhiều actions ảnh hưởng cùng state
```

---

### 3. [Mid] useEffect — Cleanup và Dependency Array

**EN:** Explain `useEffect` cleanup and dependency array. What are the common mistakes?

**VI:** Giải thích cleanup và dependency array của `useEffect`. Những lỗi hay gặp?

**Trả lời:**

```jsx
// Dependency array:
useEffect(() => { /* chạy sau mỗi render */ });
useEffect(() => { /* chạy 1 lần khi mount */ }, []);
useEffect(() => { /* chạy khi userId thay đổi */ }, [userId]);

// Cleanup — chạy trước khi effect chạy lại, và khi unmount
useEffect(() => {
  const subscription = subscribeToData(userId, setData);
  return () => subscription.unsubscribe(); // cleanup
}, [userId]);

// LỖI PHỔ BIẾN 1 — thiếu dependency
const [count, setCount] = useState(0);

useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // BUG: count luôn là giá trị ban đầu (stale closure)
  }, 1000);
  return () => clearInterval(id);
}, []); // thiếu count trong deps

// Fix 1 — thêm dependency:
useEffect(() => {
  const id = setInterval(() => setCount(count + 1), 1000);
  return () => clearInterval(id);
}, [count]); // reset interval mỗi khi count thay đổi — không tốt

// Fix 2 — dùng functional update (tốt hơn):
useEffect(() => {
  const id = setInterval(() => {
    setCount(prev => prev + 1); // không cần count trong deps
  }, 1000);
  return () => clearInterval(id);
}, []); // deps rỗng là đúng

// LỖI PHỔ BIẾN 2 — object/function trong deps tạo vòng lặp vô tận
function Component({ config }) { // config object mới mỗi render
  useEffect(() => {
    fetchData(config);
  }, [config]); // re-run mỗi render vì {} !== {}
}

// Fix — useMemo/useCallback để stable reference
const stableConfig = useMemo(() => config, [config.id, config.type]);

// LỖI PHỔ BIẾN 3 — async trong useEffect
useEffect(async () => { // SAUF — useEffect không nên là async
  const data = await fetchData();
  setState(data);
}, []);

// Fix — tạo async function bên trong:
useEffect(() => {
  let cancelled = false;

  async function load() {
    const data = await fetchData();
    if (!cancelled) setState(data);
  }

  load();
  return () => { cancelled = true; };
}, []);
```

---

### 4. [Mid] useMemo vs useCallback — Khi nào nên dùng?

**EN:** When should you use `useMemo` vs `useCallback`? What are common pitfalls?

**VI:** Khi nào dùng `useMemo` vs `useCallback`? Những lỗi hay gặp?

**Trả lời:**

```jsx
// useMemo — cache GIẤY TRỊ tính toán nặng
// useCallback — cache HÀM (để pass xuống child không bị re-render)

// useMemo — chỉ dùng khi computation thực sự expensive:
const sortedProducts = useMemo(() => {
  return [...products].sort((a, b) => a.price - b.price); // O(n log n)
}, [products]);

// useCallback — chỉ hữu ích khi kết hợp với React.memo:
const handleDelete = useCallback((id: string) => {
  dispatch(deleteItem(id));
}, [dispatch]); // dispatch stable từ useReducer/Redux

// Nếu không có React.memo → useCallback vô nghĩa
// vì child vẫn re-render khi parent re-render

// React.memo — skip re-render nếu props không thay đổi
const ExpensiveChild = React.memo(({ data, onDelete }) => {
  console.log("render");
  return <div>{/* ... */}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const data = useMemo(() => processData(rawData), [rawData]); // stable
  const handleDelete = useCallback((id) => deleteItem(id), []); // stable

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {/* ExpensiveChild không re-render khi count thay đổi */}
      <ExpensiveChild data={data} onDelete={handleDelete} />
    </>
  );
}

// ANTI-PATTERN — useMemo cho computation nhẹ:
// Overhead của useMemo > cost của computation
const doubled = useMemo(() => count * 2, [count]); // KHÔNG CẦN
const doubled = count * 2; // ĐỦ RỒI

// TÓMLẠI:
// - Profile trước, optimize sau
// - useMemo: computation tốn > 1ms, hoặc cần stable reference
// - useCallback: function pass vào React.memo component, hoặc là dep của useEffect
// - React.memo: component render nhiều lần với same props
```

---

### 5. [Mid] Custom Hooks

**EN:** What are custom hooks? Show common patterns.

**VI:** Custom hooks là gì? Các pattern thường gặp?

**Trả lời:**

```tsx
// Custom hook = hàm bắt đầu bằng "use", kết hợp hooks để reuse logic

// 1. Data fetching hook:
function useAsync<T>(asyncFn: () => Promise<T>, deps: any[] = []) {
  const [state, setState] = useState<{
    data: T | null;
    loading: boolean;
    error: Error | null;
  }>({ data: null, loading: true, error: null });

  useEffect(() => {
    let cancelled = false;
    setState(s => ({ ...s, loading: true }));

    asyncFn()
      .then(data => {
        if (!cancelled) setState({ data, loading: false, error: null });
      })
      .catch(error => {
        if (!cancelled) setState({ data: null, loading: false, error });
      });

    return () => { cancelled = true; };
  }, deps);

  return state;
}

// 2. Debounced value hook:
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

function SearchInput() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);

  // Chỉ fetch khi debouncedQuery thay đổi
  const { data } = useAsync(() => searchProducts(debouncedQuery), [debouncedQuery]);
}

// 3. Local storage hook:
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setStoredValue = useCallback((newValue: T | ((prev: T) => T)) => {
    setValue(prev => {
      const resolved = typeof newValue === "function"
        ? (newValue as (prev: T) => T)(prev)
        : newValue;
      localStorage.setItem(key, JSON.stringify(resolved));
      return resolved;
    });
  }, [key]);

  return [value, setStoredValue] as const;
}

// 4. Intersection Observer (infinite scroll / lazy load):
function useIntersectionObserver(
  ref: RefObject<Element>,
  options?: IntersectionObserverInit
) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    if (!ref.current) return;

    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);

    observer.observe(ref.current);
    return () => observer.disconnect();
  }, [ref, options]);

  return isIntersecting;
}
```

---

### 6. [Mid] Context API — Performance Pitfalls

**EN:** What are the performance issues with Context API? How do you fix them?

**VI:** Context API có những vấn đề performance gì? Cách fix?

**Trả lời:**

```tsx
// VẤN ĐỀ: Tất cả consumers re-render khi context value thay đổi
// Dù component chỉ dùng 1 phần nhỏ của context

// BAD — 1 context cho mọi thứ:
const AppContext = createContext({
  user: null,
  theme: "light",
  language: "vi",
  notifications: [],
  setUser: () => {},
  setTheme: () => {},
  // ...
});

// Khi setTheme gọi → TẤT CẢ consumers re-render, kể cả không dùng theme!

// FIX 1 — Tách context theo domain:
const UserContext = createContext<UserContextType | null>(null);
const ThemeContext = createContext<ThemeContextType | null>(null);
const NotificationContext = createContext<NotificationContextType | null>(null);

// FIX 2 — Tách value và setter:
const CountStateContext = createContext<number>(0);
const CountDispatchContext = createContext<Dispatch<Action>>(() => {});

function CountProvider({ children }) {
  const [count, dispatch] = useReducer(reducer, 0);
  return (
    <CountStateContext.Provider value={count}>
      <CountDispatchContext.Provider value={dispatch}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}

// Component chỉ dispatch sẽ không re-render khi count thay đổi
// vì dispatch function là stable

// FIX 3 — useMemo cho context value:
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  const value = useMemo(() => ({
    theme,
    toggleTheme: () => setTheme(t => t === "light" ? "dark" : "light")
  }), [theme]); // chỉ tạo object mới khi theme thay đổi

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

// FIX 4 — Khi cần granular subscription → dùng Zustand thay Context
// Context không có selector mechanism
// Zustand: chỉ re-render khi đúng slice thay đổi
const count = useCountStore(state => state.count); // chỉ re-render khi count đổi
```

---

### 7. [Senior] React 18 — useTransition, useDeferredValue, useId

**EN:** What are the new hooks in React 18? When do you use them?

**VI:** Các hooks mới trong React 18 là gì? Khi nào dùng chúng?

**Trả lời:**

```tsx
// useTransition — đánh dấu state update là "non-urgent"
// UI vẫn responsive, update phức tạp chạy background

function SearchPage() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    setQuery(e.target.value); // urgent — update input ngay

    startTransition(() => {
      // non-urgent — có thể bị interrupt nếu user tiếp tục gõ
      setResults(filterProducts(e.target.value)); // expensive operation
    });
  };

  return (
    <>
      <input value={query} onChange={handleSearch} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  );
}

// useDeferredValue — defer việc re-render của expensive component
// Tương tự debounce nhưng ở React level

function ProductList({ query }) {
  // deferredQuery "lag" sau query thực tế
  const deferredQuery = useDeferredValue(query);

  // ExpensiveList chỉ re-render sau khi browser có thời gian rảnh
  const isStale = query !== deferredQuery;

  return (
    <div style={{ opacity: isStale ? 0.8 : 1 }}>
      <ExpensiveList query={deferredQuery} />
    </div>
  );
}

// useId — generate unique ID stable giữa server và client
// Dùng cho accessibility (htmlFor/id pairing)

function FormField({ label }) {
  const id = useId(); // "r0:", "r1:", etc. — unique và stable
  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} />
    </>
  );
}
// Không dùng Math.random() hay counter vì gây hydration mismatch với SSR!
```

---

### 8. [Senior] Error Boundaries và Suspense

**EN:** Explain Error Boundaries and Suspense in React.

**VI:** Giải thích Error Boundaries và Suspense trong React.

**Trả lời:**

```tsx
// Error Boundary — catch JavaScript errors trong component tree
// Phải là class component (chưa có hook equivalent)

class ErrorBoundary extends React.Component<
  { fallback: ReactNode; children: ReactNode },
  { hasError: boolean; error: Error | null }
> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log error to monitoring service:
    logErrorToService(error, errorInfo.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}

// Dùng react-error-boundary package — đơn giản hơn:
import { ErrorBoundary } from "react-error-boundary";

function ProductPage() {
  return (
    <ErrorBoundary
      fallback={<p>Có lỗi xảy ra. <button onClick={() => window.location.reload()}>Thử lại</button></p>}
      onError={(error, info) => logError(error, info)}
    >
      <ProductList />
    </ErrorBoundary>
  );
}

// Suspense — declarative loading states
// Kết hợp với lazy loading và data fetching (React 18+)

const ProductDetail = lazy(() => import("./ProductDetail"));

function App() {
  return (
    <Suspense fallback={<ProductDetailSkeleton />}>
      <ProductDetail />
    </Suspense>
  );
}

// Nested Suspense — granular loading:
function Dashboard() {
  return (
    <div>
      <Suspense fallback={<HeaderSkeleton />}>
        <DashboardHeader /> {/* stream riêng */}
      </Suspense>

      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />    {/* stream riêng */}
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <OrdersTable />     {/* stream riêng */}
      </Suspense>
    </div>
  );
}
```

---

# NEXT.JS

---

### 1. [Junior] Pages Router vs App Router

**EN:** What is the difference between Pages Router and App Router in Next.js?

**VI:** Sự khác biệt giữa Pages Router và App Router trong Next.js?

**Trả lời:**

| | Pages Router | App Router (Next.js 13+) |
|---|---|---|
| Thư mục | `/pages` | `/app` |
| Data fetching | `getServerSideProps`, `getStaticProps` | `async` component trực tiếp |
| Layouts | `_app.tsx` | `layout.tsx` (nested) |
| Loading UI | Custom | `loading.tsx` tự động |
| Error UI | Custom | `error.tsx` tự động |
| Server Components | Không | Mặc định |
| Streaming | Không | Có (Suspense) |

```tsx
// Pages Router:
// pages/products/[id].tsx
export async function getServerSideProps({ params }) {
  const product = await fetchProduct(params.id);
  return { props: { product } };
}
export default function ProductPage({ product }) { /* ... */ }

// App Router:
// app/products/[id]/page.tsx
async function ProductPage({ params }: { params: { id: string } }) {
  const product = await fetchProduct(params.id); // fetch trực tiếp trong component!
  return <div>{product.name}</div>;
}
```

---

### 2. [Mid] SSR vs SSG vs ISR vs CSR — Khi nào dùng?

**EN:** Explain each rendering strategy and when to use them.

**VI:** Giải thích từng rendering strategy và khi nào dùng.

**Trả lời:**

```tsx
// CSR — Client-Side Rendering
// HTML trống, data fetch trên browser
// Dùng cho: dashboard, SaaS app sau auth, không cần SEO
"use client";
function Dashboard() {
  const { data } = useSWR("/api/me", fetcher);
  return data ? <DashboardContent data={data} /> : <Spinner />;
}

// SSG — Static Site Generation (Pages Router)
// HTML render lúc build time → nhanh nhất, CDN cacheable
// Dùng cho: blog, docs, landing page
export async function getStaticProps() {
  const posts = await fetchAllBlogPosts();
  return { props: { posts } };
}

export async function getStaticPaths() {
  const posts = await fetchAllBlogPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: "blocking"
  };
}

// ISR — Incremental Static Regeneration
// SSG + tự động regenerate sau interval
// Dùng cho: e-commerce, news, content thay đổi không thường xuyên
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return {
    props: { product },
    revalidate: 60 // regenerate sau 60 giây
  };
}

// SSR — Server-Side Rendering (Pages Router)
// HTML render trên server mỗi request
// Dùng cho: personalized content, cần auth context, data luôn fresh
export async function getServerSideProps({ req, params }) {
  const token = req.cookies.token;
  const user = await verifyToken(token);
  if (!user) return { redirect: { destination: "/login" } };
  const data = await fetchPersonalizedData(user.id);
  return { props: { data } };
}

// App Router equivalents:
// SSG (default):
const data = await fetch(url, { cache: "force-cache" });
// ISR:
const data = await fetch(url, { next: { revalidate: 60 } });
// SSR:
const data = await fetch(url, { cache: "no-store" });
```

| | SSG | ISR | SSR | CSR |
|---|---|---|---|---|
| Khi nào build | Build time | Build + revalidate | Runtime | Runtime (browser) |
| TTFB | Nhanh nhất | Nhanh | Chậm hơn | Nhanh (HTML trống) |
| Data freshness | Cũ nhất | Configurable | Luôn fresh | Luôn fresh |
| SEO | Tốt | Tốt | Tốt | Kém |
| Dùng cho | Blog, Docs | E-commerce, News | Dashboard, Cart | SaaS app |

---

### 3. [Senior] Server Components vs Client Components

**EN:** What is the difference? What is the composition pattern?

**VI:** Sự khác biệt là gì? Composition pattern là gì?

**Trả lời:**

```tsx
// SERVER COMPONENT — mặc định trong app/
// - Render trên server, không gửi JS xuống client
// - Có thể access DB, filesystem trực tiếp
// - Không có state, hooks, browser APIs, event handlers

// app/users/page.tsx
import { db } from "@/lib/db";

async function UsersPage() {
  const users = await db.user.findMany(); // trực tiếp! không cần API route
  return (
    <div>
      {users.map(user => <UserCard key={user.id} user={user} />)}
    </div>
  );
}

// CLIENT COMPONENT — khai báo "use client"
// - Có thể dùng state, hooks, event handlers, browser APIs
// - JS được hydrate trên browser
"use client";
function SearchBar({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");
  return (
    <input
      value={query}
      onChange={e => { setQuery(e.target.value); onSearch(e.target.value); }}
    />
  );
}

// COMPOSITION — quan trọng nhất
// ✅ Server bọc Client là OK:
async function Page() {
  const data = await fetchData();
  return (
    <ClientShell> {/* Client Component */}
      <ServerContent data={data} /> {/* Server Component làm children */}
    </ClientShell>
  );
}

// ❌ Import Server Component trong Client là KHÔNG OK:
"use client";
import ServerComponent from "./ServerComponent"; // Error!

// RULE: Đẩy "use client" xuống sâu nhất có thể trong component tree
```

---

### 4. [Mid] Middleware trong Next.js

**EN:** What is Next.js Middleware? What can you do with it?

**VI:** Middleware trong Next.js là gì? Dùng để làm gì?

**Trả lời:**

```typescript
// middleware.ts — chạy TRƯỚC khi request được xử lý, chạy ở Edge Runtime
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // 1. Authentication
  const token = request.cookies.get("auth-token")?.value;
  if (pathname.startsWith("/dashboard") && !token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  // 2. Role-based access
  if (pathname.startsWith("/admin")) {
    const role = request.cookies.get("user-role")?.value;
    if (role !== "admin") {
      return NextResponse.redirect(new URL("/403", request.url));
    }
  }

  // 3. A/B Testing
  if (pathname === "/landing") {
    const bucket = Math.random() < 0.5 ? "a" : "b";
    const res = NextResponse.rewrite(new URL(`/landing-${bucket}`, request.url));
    res.cookies.set("ab-bucket", bucket, { maxAge: 86400 });
    return res;
  }

  // 4. Custom headers
  const response = NextResponse.next();
  response.headers.set("X-Custom-Header", "value");
  return response;
}

// Chỉ apply cho specific paths:
export const config = {
  matcher: ["/dashboard/:path*", "/admin/:path*"]
};
```

---

### 5. [Senior] Caching Strategies trong Next.js App Router

**EN:** Explain the 4 caching layers in Next.js App Router.

**VI:** Giải thích 4 layers caching trong Next.js App Router.

**Trả lời:**

```
1. Request Memoization  — trong 1 render pass
2. Data Cache           — persistent, cross-request
3. Full Route Cache     — static HTML tại build time
4. Router Cache         — client-side, in-memory
```

```typescript
// 1. Request Memoization — tự động deduplicate cùng request trong 1 render
// Gọi fetchUser(123) ở 10 components → chỉ network request 1 lần

// 2. Data Cache:
const data = await fetch(url, { cache: "force-cache" });       // SSG — vĩnh viễn
const data = await fetch(url, { cache: "no-store" });          // SSR — không cache
const data = await fetch(url, { next: { revalidate: 3600 } }); // ISR — 1 giờ
const data = await fetch(url, { next: { tags: ["products"] } }); // on-demand

// On-demand revalidation:
import { revalidateTag, revalidatePath } from "next/cache";

async function updateProduct(id: string) {
  await db.product.update({ where: { id }, data: {} });
  revalidateTag(`product-${id}`);
  revalidatePath("/products");
}

// 3. Route Segment Config:
export const dynamic = "force-dynamic"; // opt out mọi caching
export const revalidate = 60;           // ISR cho toàn route

// 4. Router Cache — invalidate:
import { useRouter } from "next/navigation";
const router = useRouter();
router.refresh(); // xóa router cache, re-fetch server data
```

---

# STATE MANAGEMENT

---

### 1. [Junior] Khi nào dùng Redux vs Context vs Zustand vs React Query?

**EN:** When do you choose each state management solution?

**VI:** Khi nào chọn từng solution?

**Trả lời:**

```
Câu hỏi đầu tiên: Data này từ API (server state) hay UI logic (client state)?

Server State → TanStack Query / RTK Query
  - Caching, background refetch, loading/error tự động
  - Optimistic updates, pagination

Client State → tiếp tục xét:
  ├─ Nhỏ, 1-2 components → useState / useReducer
  ├─ Share giữa vài components, ít thay đổi → Context API
  │    (theme, locale, auth user info)
  ├─ Phức tạp, thay đổi thường xuyên, nhiều components → Zustand
  └─ Enterprise scale, team lớn, cần DevTools mạnh → Redux/RTK
```

| | Context | Zustand | Redux/RTK | React Query |
|---|---|---|---|---|
| Bundle | 0 | ~1KB | ~20KB | ~13KB |
| Boilerplate | Thấp | Rất thấp | Cao | Thấp |
| Performance | Kém | Tốt | Tốt | Tốt |
| DevTools | Không | Có | Rất mạnh | Có |
| Server State | Không | Không | RTK Query | **Tốt nhất** |

---

### 2. [Mid] Redux Toolkit — Slice, Store, Async Thunk

**EN:** How do you set up Redux with RTK? Explain slices and async thunks.

**VI:** Cách setup Redux với RTK? Giải thích slices và async thunks.

**Trả lời:**

```typescript
// 1. SLICE — kết hợp actions + reducer:
import { createSlice, PayloadAction, createAsyncThunk } from "@reduxjs/toolkit";

// Async thunk:
export const fetchProducts = createAsyncThunk(
  "products/fetchAll",
  async (filters: ProductFilters, { rejectWithValue }) => {
    try {
      return await api.getProducts(filters);
    } catch (err: any) {
      return rejectWithValue(err.response.data);
    }
  }
);

const productsSlice = createSlice({
  name: "products",
  initialState: {
    items: [] as Product[],
    status: "idle" as "idle" | "loading" | "succeeded" | "failed",
    error: null as string | null
  },
  reducers: {
    addProduct(state, action: PayloadAction<Product>) {
      state.items.push(action.payload); // RTK dùng Immer — mutate OK!
    },
    removeProduct(state, action: PayloadAction<string>) {
      state.items = state.items.filter(p => p.id !== action.payload);
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.status = "loading";
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = "succeeded";
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.payload as string;
      });
  }
});

export const { addProduct, removeProduct } = productsSlice.actions;

// 2. STORE:
export const store = configureStore({
  reducer: {
    products: productsSlice.reducer,
    [productApi.reducerPath]: productApi.reducer,
  },
  middleware: (getDefault) => getDefault().concat(productApi.middleware)
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// 3. TYPED HOOKS:
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector = <T>(selector: (s: RootState) => T) =>
  useSelector<RootState, T>(selector);

// 4. SỬ DỤNG:
function ProductList() {
  const dispatch = useAppDispatch();
  const { items, status } = useAppSelector(state => state.products);

  useEffect(() => {
    if (status === "idle") dispatch(fetchProducts({}));
  }, [status]);

  if (status === "loading") return <Skeleton />;
  return <ul>{items.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

---

### 3. [Senior] RTK Query — createApi, Tags, Cache Invalidation

**EN:** Explain RTK Query's `createApi`, providesTags, invalidatesTags.

**VI:** Giải thích createApi, providesTags, invalidatesTags trong RTK Query.

**Trả lời:**

```typescript
export const productApi = createApi({
  reducerPath: "productApi",
  baseQuery: fetchBaseQuery({
    baseUrl: "/api",
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) headers.set("Authorization", `Bearer ${token}`);
      return headers;
    }
  }),
  tagTypes: ["Product", "ProductList"],

  endpoints: (builder) => ({
    getProducts: builder.query<Product[], void>({
      query: () => "/products",
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: "Product" as const, id })),
              { type: "ProductList", id: "LIST" }
            ]
          : [{ type: "ProductList", id: "LIST" }]
    }),

    createProduct: builder.mutation<Product, Omit<Product, "id">>({
      query: (data) => ({ url: "/products", method: "POST", body: data }),
      invalidatesTags: [{ type: "ProductList", id: "LIST" }]
      // → tự động refetch getProducts sau khi tạo
    }),

    updateProduct: builder.mutation<Product, { id: string; data: Partial<Product> }>({
      query: ({ id, data }) => ({ url: `/products/${id}`, method: "PATCH", body: data }),
      invalidatesTags: (result, error, { id }) => [{ type: "Product", id }],
      // Optimistic update:
      async onQueryStarted({ id, data }, { dispatch, queryFulfilled }) {
        const patch = dispatch(
          productApi.util.updateQueryData("getProducts", undefined, (draft) => {
            const product = draft.find(p => p.id === id);
            if (product) Object.assign(product, data);
          })
        );
        try {
          await queryFulfilled;
        } catch {
          patch.undo(); // rollback nếu server fail
        }
      }
    }),

    deleteProduct: builder.mutation<void, string>({
      query: (id) => ({ url: `/products/${id}`, method: "DELETE" }),
      invalidatesTags: (result, error, id) => [
        { type: "Product", id },
        { type: "ProductList", id: "LIST" }
      ]
    })
  })
});

export const {
  useGetProductsQuery,
  useCreateProductMutation,
  useUpdateProductMutation,
  useDeleteProductMutation
} = productApi;

// Sử dụng:
function ProductList() {
  const { data, isLoading, isFetching } = useGetProductsQuery();
  const [deleteProduct] = useDeleteProductMutation();

  if (isLoading) return <Skeleton />;
  return (
    <div style={{ opacity: isFetching ? 0.7 : 1 }}>
      {data?.map(p => (
        <div key={p.id}>
          {p.name}
          <button onClick={() => deleteProduct(p.id)}>Xóa</button>
        </div>
      ))}
    </div>
  );
}
```

---

### 4. [Mid] Zustand

**EN:** How does Zustand work? Show a practical store setup.

**VI:** Zustand hoạt động như thế nào? Setup store thực tế?

**Trả lời:**

```typescript
import { create } from "zustand";
import { devtools, persist, immer } from "zustand/middleware";

interface UserStore {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

const useUserStore = create<UserStore>()(
  devtools(
    persist(
      immer((set, get) => ({
        user: null,
        token: null,
        isLoading: false,

        login: async (credentials) => {
          set({ isLoading: true });
          try {
            const { user, token } = await authApi.login(credentials);
            set({ user, token, isLoading: false });
          } catch (err) {
            set({ isLoading: false });
            throw err;
          }
        },

        logout: () => set({ user: null, token: null })
      })),
      {
        name: "user-storage",
        partialize: (state) => ({ token: state.token }) // chỉ persist token
      }
    )
  )
);

// SỬ DỤNG — chỉ re-render khi đúng slice thay đổi:
function UserName() {
  // Component này chỉ re-render khi user.name thay đổi
  const name = useUserStore(state => state.user?.name);
  return <span>{name}</span>;
}

function LoginButton() {
  // Component này không re-render khi user thay đổi
  const login = useUserStore(state => state.login);
  const isLoading = useUserStore(state => state.isLoading);
  return <button onClick={() => login(creds)} disabled={isLoading}>Login</button>;
}

// SO SÁNH VỚI CONTEXT:
// Context: tất cả consumers re-render khi bất kỳ value nào thay đổi
// Zustand: chỉ re-render component subscribe đúng slice → hiệu quả hơn nhiều
```

---

### 5. [Mid] TanStack Query / React Query

**EN:** Explain core concepts of TanStack Query.

**VI:** Giải thích các khái niệm cốt lõi của TanStack Query.

**Trả lời:**

```typescript
// Setup:
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,  // data "fresh" 5 phút — không refetch
      gcTime: 10 * 60 * 1000,    // xóa cache sau 10 phút không dùng
      retry: 3,
      refetchOnWindowFocus: true
    }
  }
});

// useQuery:
function ProductDetail({ id }: { id: string }) {
  const { data, isLoading, isFetching, isError, error } = useQuery({
    queryKey: ["product", id],    // cache key — unique, mô tả data
    queryFn: () => fetchProduct(id),
    enabled: !!id,               // chỉ fetch khi id có giá trị
    staleTime: 60_000,
    select: (data) => ({         // transform trước khi return
      ...data,
      formattedPrice: data.price.toLocaleString("vi-VN")
    })
  });

  if (isLoading) return <Skeleton />;
  if (isError) return <Error message={error.message} />;
  return <div>{data.name} — {data.formattedPrice}đ</div>;
}

// useMutation với optimistic update:
function DeleteButton({ productId }: { productId: string }) {
  const queryClient = useQueryClient();

  const deleteMutation = useMutation({
    mutationFn: (id: string) => api.delete(`/products/${id}`),

    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: ["products"] });
      const previous = queryClient.getQueryData(["products"]);

      // Update cache ngay lập tức:
      queryClient.setQueryData(["products"], (old: Product[]) =>
        old.filter(p => p.id !== id)
      );

      return { previous };
    },

    onError: (err, id, context) => {
      queryClient.setQueryData(["products"], context?.previous); // rollback
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ["products"] }); // sync với server
    }
  });

  return (
    <button
      onClick={() => deleteMutation.mutate(productId)}
      disabled={deleteMutation.isPending}
    >
      {deleteMutation.isPending ? "Đang xóa..." : "Xóa"}
    </button>
  );
}

// Stale-while-revalidate:
// 1. Có cache → hiển thị ngay (dù stale)
// 2. Đồng thời refetch background
// 3. Update UI khi có data mới
// → User không bao giờ thấy màn hình trắng
```

---

## Tips Phỏng Vấn Tại Vulcan Labs

### Câu hay bị hỏi nhất

1. **JS:** Event loop + Promise execution order, closure trong loop, `this` binding
2. **TS:** interface vs type, utility types thực tế, generics với constraints
3. **React:** Tại sao cần `key`, useMemo/useCallback khi nào cần, useEffect cleanup
4. **Next.js:** SSR vs SSG vs ISR theo scenario cụ thể, Server vs Client Components
5. **State:** Khi nào chọn Redux vs Zustand vs React Query, RTK Query cache invalidation

### Lời khuyên answer theo level Senior

- Luôn nói đến **trade-offs** — không chỉ "làm thế này" mà "tại sao chọn thế này thay vì thế kia"
- Kết nối câu trả lời với **dự án thực tế** — Design System tại Herond, Turborepo tại Biso24
- Show **system thinking** — nghĩ đến scalability, maintainability, team DX
- Với câu performance: **"profile trước, optimize sau"** — không premature optimization

---

*Cập nhật: 2026-03-25 | Chuẩn bị phỏng vấn Vulcan Labs*
