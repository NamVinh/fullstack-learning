# TypeScript — Interview Questions (Junior → Senior)

> Tài liệu ôn tập phỏng vấn TypeScript từ cấp độ Junior đến Senior.
> Câu hỏi song ngữ (EN/VI), trả lời chi tiết bằng tiếng Việt, code ví dụ bằng tiếng Anh.

---

## Table of Contents

1. [interface vs type](#1-interface-vs-type)
2. [Generics](#2-generics)
3. [Utility Types](#3-utility-types)
4. [unknown vs any vs never](#4-unknown-vs-any-vs-never)
5. [Discriminated Unions](#5-discriminated-unions)
6. [Conditional Types](#6-conditional-types)
7. [Mapped Types](#7-mapped-types)
8. [Template Literal Types](#8-template-literal-types)
9. [Declaration Merging & Module Augmentation](#9-declaration-merging--module-augmentation)
10. [Strict Mode](#10-strict-mode)
11. [Type Narrowing & Type Guards](#11-type-narrowing--type-guards)
12. [Decorators](#12-decorators)

---

## 1. interface vs type

### Q1 — [Junior]

**EN:** What are the differences between `interface` and `type` in TypeScript? When should you use each?

**VI:** Sự khác biệt giữa `interface` và `type` trong TypeScript là gì? Khi nào dùng từng loại?

**Trả lời:**

Cả hai đều dùng để mô tả hình dạng (shape) của một object. Tuy nhiên có một số điểm khác biệt quan trọng:

---

**Điểm khác biệt chính:**

**1. Declaration Merging — chỉ `interface` hỗ trợ:**

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}

// TypeScript tự động merge thành:
// interface User { name: string; age: number; }

const user: User = { name: "Alice", age: 30 }; // OK

// `type` KHÔNG hỗ trợ declaration merging
type Config = { host: string };
type Config = { port: number }; // Error: Duplicate identifier 'Config'
```

**2. `type` linh hoạt hơn — hỗ trợ nhiều kiểu:**

```ts
// Union types — chỉ type mới làm được
type ID = string | number;
type Status = "pending" | "active" | "inactive";
type StringOrArray = string | string[];

// Intersection types
type AdminUser = User & { permissions: string[] };

// Primitive alias
type Seconds = number;
type Username = string;

// Tuple
type Pair = [string, number];
type RGB = [red: number, green: number, blue: number];

// Function type
type Callback = (error: Error | null, result?: string) => void;

// Mapped type alias
type ReadonlyUser = Readonly<User>;

// Conditional type — chỉ type mới làm được
type IsString<T> = T extends string ? true : false;
```

**3. `interface` extend — cú pháp `extends`:**

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// type dùng intersection (&)
type AnimalType = { name: string };
type DogType = AnimalType & { breed: string };

// interface có thể extend type, và ngược lại
interface HybridDog extends DogType { color: string }
type HybridAnimal = Animal & { weight: number };
```

**4. Implements — cả hai đều được:**

```ts
interface Printable {
  print(): void;
}

type Serializable = {
  serialize(): string;
};

class Document implements Printable, Serializable {
  print() { console.log("printing..."); }
  serialize() { return JSON.stringify(this); }
}
```

---

**Khi nào dùng gì?**

| Tình huống | Dùng |
|---|---|
| Định nghĩa shape của object / API response | `interface` |
| Public API của library (cho phép user extend) | `interface` |
| Union, intersection, tuple, conditional types | `type` |
| Function type standalone | `type` |
| Primitive alias | `type` |
| Kế thừa phức tạp với nhiều nguồn | `type` với `&` |

**Recommendation thực tế:** Dùng `interface` cho object shapes và class contracts. Dùng `type` cho mọi thứ còn lại. Trong một dự án, hãy chọn một convention và đi nhất quán.

---

## 2. Generics

### Q2 — [Junior]

**EN:** What are generics in TypeScript? Show examples with basic generics, constraints, multiple type params, and generic hooks.

**VI:** Generics trong TypeScript là gì? Cho ví dụ với generic cơ bản, constraints, nhiều type params, và generic hooks.

**Trả lời:**

**Generics** cho phép viết code có thể làm việc với nhiều kiểu dữ liệu khác nhau trong khi vẫn giữ được type safety — giống như "placeholder" cho kiểu dữ liệu, được xác định khi sử dụng.

**Generic cơ bản:**

```ts
// Không có generic — phải overload hoặc dùng any
function identity(value: any): any { return value; } // mất type info

// Với generic
function identity<T>(value: T): T {
  return value;
}

const str = identity("hello");   // T = string, return: string
const num = identity(42);        // T = number, return: number
const arr = identity([1, 2, 3]); // T = number[], return: number[]

// Type inference — TypeScript tự suy ra T
identity("hello");       // TypeScript biết T = string
identity<string>("hi");  // explicit khi cần
```

**Generic với Constraints (`extends`):**

```ts
// Chỉ chấp nhận type có property `length`
function logLength<T extends { length: number }>(value: T): T {
  console.log(`Length: ${value.length}`);
  return value;
}

logLength("hello");    // OK — string có .length
logLength([1, 2, 3]);  // OK — array có .length
logLength({ length: 5, data: "test" }); // OK
logLength(42);         // Error — number không có .length

// keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
getProperty(user, "name"); // OK, return type: string
getProperty(user, "age");  // OK, return type: number
getProperty(user, "xyz");  // Error — "xyz" không phải key của user

// Extending interface
interface Repository<T extends { id: number }> {
  findById(id: number): Promise<T>;
  findAll(): Promise<T[]>;
  save(item: T): Promise<T>;
  delete(id: number): Promise<void>;
}
```

**Nhiều type parameters:**

```ts
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

pair("hello", 42);    // [string, number]
pair(true, { x: 1 }); // [boolean, { x: number }]

// Map type-safe
function mapObject<T, U>(
  obj: Record<string, T>,
  transform: (value: T, key: string) => U
): Record<string, U> {
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [k, transform(v, k)])
  );
}

const doubled = mapObject({ a: 1, b: 2, c: 3 }, v => v * 2);
// { a: 2, b: 4, c: 6 } — type: Record<string, number>

// Generic với default type
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
  message: string;
}

const response: ApiResponse = { data: "anything", status: 200, message: "OK" };
const typed: ApiResponse<User[]> = { data: [], status: 200, message: "OK" };
```

**Generic Hooks (React):**

```ts
// Generic useState wrapper với validation
function useValidatedState<T>(
  initialValue: T,
  validate: (value: T) => string | null
): [T, (value: T) => boolean, string | null] {
  const [value, setValue] = useState<T>(initialValue);
  const [error, setError] = useState<string | null>(null);

  const setValidatedValue = (newValue: T): boolean => {
    const validationError = validate(newValue);
    setError(validationError);
    if (!validationError) setValue(newValue);
    return !validationError;
  };

  return [value, setValidatedValue, error];
}

// Usage
const [age, setAge, ageError] = useValidatedState<number>(
  0,
  v => (v < 0 || v > 150) ? "Age must be 0-150" : null
);

// Generic fetch hook
function useFetch<T>(url: string): {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
} {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    try {
      const res = await fetch(url);
      const json: T = await res.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err : new Error("Unknown error"));
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => { fetchData(); }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

// Usage — TypeScript biết data là User
const { data, loading, error } = useFetch<User>("/api/users/1");
data?.name; // TypeScript autocomplete hoạt động!
```

---

## 3. Utility Types

### Q3 — [Mid]

**EN:** Explain the built-in utility types in TypeScript and implement some of them from scratch.

**VI:** Giải thích các utility types có sẵn trong TypeScript và implement một số loại từ đầu.

**Trả lời:**

**Nhóm 1 — Object transformation:**

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Partial<T> — tất cả properties trở thành optional
type UserUpdate = Partial<User>;
// { id?: number; name?: string; email?: string; ... }
function updateUser(id: number, changes: Partial<User>) { /* ... */ }

// Required<T> — tất cả optional thành required
interface Config {
  host?: string;
  port?: number;
}
type StrictConfig = Required<Config>;
// { host: string; port: number; }

// Readonly<T> — tất cả properties thành readonly
const frozenUser: Readonly<User> = { id: 1, name: "Alice", /* ... */ };
frozenUser.name = "Bob"; // Error: Cannot assign to 'name' because it is a read-only property

// Pick<T, Keys> — chọn một số properties
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string; }
type PublicUser = Pick<User, "id" | "name" | "email">;

// Omit<T, Keys> — loại bỏ một số properties
type UserWithoutPassword = Omit<User, "password">;
// { id: number; name: string; email: string; createdAt: Date; }
type UserForm = Omit<User, "id" | "createdAt">; // form không cần id, createdAt

// Record<Keys, Type> — tạo object type với keys cụ thể
type UserRoles = Record<string, "admin" | "user" | "guest">;
type StatusMap = Record<"pending" | "active" | "inactive", { label: string; color: string }>;

const statusConfig: StatusMap = {
  pending:  { label: "Pending",  color: "yellow" },
  active:   { label: "Active",   color: "green"  },
  inactive: { label: "Inactive", color: "gray"   }
};
```

**Nhóm 2 — Function types:**

```ts
function fetchUsers(page: number, limit: number): Promise<User[]> {
  return fetch(`/api/users?page=${page}&limit=${limit}`).then(r => r.json());
}

// ReturnType<T> — lấy return type của function
type FetchResult = ReturnType<typeof fetchUsers>;
// Promise<User[]>

type SyncResult = ReturnType<() => { name: string; age: number }>;
// { name: string; age: number }

// Parameters<T> — lấy parameters làm tuple type
type FetchParams = Parameters<typeof fetchUsers>;
// [page: number, limit: number]

function wrapper(...args: Parameters<typeof fetchUsers>): ReturnType<typeof fetchUsers> {
  console.log("Calling fetchUsers with:", args);
  return fetchUsers(...args);
}

// Awaited<T> — unwrap Promise (TypeScript 4.5+)
type ResolvedUsers = Awaited<ReturnType<typeof fetchUsers>>;
// User[] (unwrap Promise<User[]>)

type DeepAwaited = Awaited<Promise<Promise<string>>>;
// string
```

**Nhóm 3 — Union manipulation:**

```ts
type Status = "pending" | "active" | "inactive" | "banned";
type Permission = "read" | "write" | "admin";

// Extract<T, U> — lấy types trong T có thể assign cho U
type ActiveStatus = Extract<Status, "pending" | "active">;
// "pending" | "active"

type StringOrNumber = Extract<string | number | boolean, string | number>;
// string | number

// Exclude<T, U> — loại bỏ types trong T có thể assign cho U
type PublicStatus = Exclude<Status, "banned">;
// "pending" | "active" | "inactive"

type NonBoolean = Exclude<string | number | boolean, boolean>;
// string | number

// NonNullable<T> — loại bỏ null và undefined
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// string
```

**Custom implementations — giải thích cơ chế bên trong:**

```ts
// MyPartial
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// MyRequired
type MyRequired<T> = {
  [K in keyof T]-?: T[K]; // -? xóa optional modifier
};

// MyReadonly
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

// MyPick
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// MyOmit
type MyOmit<T, K extends keyof T> = MyPick<T, Exclude<keyof T, K>>;

// MyRecord
type MyRecord<K extends keyof any, V> = {
  [P in K]: V;
};

// MyReturnType
type MyReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : never;

// MyParameters
type MyParameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any ? P : never;

// MyAwaited
type MyAwaited<T> =
  T extends Promise<infer Inner> ? MyAwaited<Inner> : T;

// MyNonNullable
type MyNonNullable<T> = T extends null | undefined ? never : T;

// MyExclude
type MyExclude<T, U> = T extends U ? never : T;

// MyExtract
type MyExtract<T, U> = T extends U ? T : never;
```

---

## 4. unknown vs any vs never

### Q4 — [Mid]

**EN:** Explain the difference between `unknown`, `any`, and `never`. When would you use each?

**VI:** Giải thích sự khác biệt giữa `unknown`, `any`, và `never`. Khi nào dùng từng loại?

**Trả lời:**

Ba loại đặc biệt này thường gây nhầm lẫn nhưng có semantic rất khác nhau.

**`any` — tắt type checking:**

```ts
let value: any = "hello";
value = 42;
value = { x: 1 };
value.someMethod();      // OK — no error (runtime crash nếu method không tồn tại)
value[0];                // OK
const x: string = value; // OK — không có type check

// Khi nào dùng any:
// - Migration từ JS sang TS (tạm thời)
// - Thư viện bên thứ ba chưa có type definitions
// - Prototype nhanh (nhưng hãy refactor sau)
// TRÁNH dùng any trong production code!
```

**`unknown` — type-safe top type:**

```ts
let value: unknown = "hello";
value = 42;       // OK — có thể assign bất kỳ type nào vào unknown
value = { x: 1 }; // OK

// KHÔNG thể dùng trực tiếp mà không narrow
value.toUpperCase(); // Error: Object is of type 'unknown'
value[0];            // Error
const x: string = value; // Error: Type 'unknown' is not assignable to type 'string'

// BẮT BUỘC narrow trước khi dùng
if (typeof value === "string") {
  value.toUpperCase(); // OK — TypeScript biết đây là string
}

if (value instanceof Date) {
  value.toISOString(); // OK
}

// Use case — error handling (error trong catch là unknown từ TS 4.0)
async function fetchData() {
  try {
    const data = await fetch("/api");
    return data.json();
  } catch (error: unknown) {
    // error là unknown — phải narrow
    if (error instanceof Error) {
      console.error(error.message); // OK
    } else if (typeof error === "string") {
      console.error(error); // OK
    } else {
      console.error("Unknown error occurred");
    }
  }
}

// Use case — API response với unknown data
function parseApiResponse(response: unknown): User {
  if (
    typeof response === "object" &&
    response !== null &&
    "name" in response &&
    "age" in response
  ) {
    return response as User; // đã validate, cast an toàn
  }
  throw new Error("Invalid response shape");
}
```

**`never` — bottom type, không thể có giá trị:**

```ts
// never = kiểu không có giá trị nào — function không bao giờ return
function throwError(message: string): never {
  throw new Error(message);
  // code sau đây unreachable — OK vì never
}

function infiniteLoop(): never {
  while (true) {}
}

// Exhaustive checking — đảm bảo xử lý TẤT CẢ cases
type Shape = "circle" | "square" | "triangle";

function getArea(shape: Shape): number {
  switch (shape) {
    case "circle":   return Math.PI * 5 ** 2;
    case "square":   return 25;
    case "triangle": return 12.5;
    default:
      const exhaustiveCheck: never = shape;
      // Nếu thêm "rectangle" vào Shape nhưng quên xử lý ở đây,
      // TypeScript sẽ báo lỗi: Type '"rectangle"' is not assignable to type 'never'
      throw new Error(`Unhandled shape: ${exhaustiveCheck}`);
  }
}

// never trong conditional types — loại bỏ type
type NonNullable<T> = T extends null | undefined ? never : T;
type RemoveString<T> = T extends string ? never : T;

type NumOrBool = RemoveString<string | number | boolean>;
// number | boolean (string bị loại)

// Intersection với never = never
type ImpossibleType = string & number; // never — không thể vừa string vừa number
```

**Bảng so sánh:**

| | `any` | `unknown` | `never` |
|---|---|---|---|
| Assign từ bất kỳ type | Có | Có | Không |
| Assign vào bất kỳ type | Có | Không (cần narrow) | Có (vì không có value) |
| Operations trực tiếp | Có | Không | N/A |
| Use case | Migration, tạm thời | API input, error handling | Exhaustive check, throw fn |

---

## 5. Discriminated Unions

### Q5 — [Mid]

**EN:** What are discriminated unions? Show real-world examples with payment/status patterns.

**VI:** Discriminated unions là gì? Cho ví dụ thực tế với payment và status patterns.

**Trả lời:**

**Discriminated Union** (còn gọi là tagged union / algebraic data type) là union của nhiều types, mỗi type có một **literal property chung** (discriminant) để TypeScript có thể phân biệt.

**Ví dụ 1 — Payment System:**

```ts
// Discriminant: `type` property
interface CreditCardPayment {
  type: "credit_card";
  cardNumber: string;
  expiryDate: string;
  cvv: string;
}

interface BankTransferPayment {
  type: "bank_transfer";
  accountNumber: string;
  routingNumber: string;
  bankName: string;
}

interface CryptoPayment {
  type: "crypto";
  walletAddress: string;
  currency: "BTC" | "ETH" | "USDT";
}

interface MoMoPayment {
  type: "momo";
  phoneNumber: string;
}

type PaymentMethod =
  | CreditCardPayment
  | BankTransferPayment
  | CryptoPayment
  | MoMoPayment;

// TypeScript tự động narrow dựa vào discriminant
function processPayment(payment: PaymentMethod): string {
  switch (payment.type) {
    case "credit_card":
      // payment: CreditCardPayment — TypeScript biết
      return `Charging card **** ${payment.cardNumber.slice(-4)}`;

    case "bank_transfer":
      // payment: BankTransferPayment
      return `Transferring to ${payment.bankName} ${payment.accountNumber}`;

    case "crypto":
      // payment: CryptoPayment
      return `Sending ${payment.currency} to ${payment.walletAddress}`;

    case "momo":
      // payment: MoMoPayment
      return `Sending via MoMo to ${payment.phoneNumber}`;

    default:
      const _exhaustive: never = payment; // exhaustive check
      throw new Error(`Unknown payment type`);
  }
}
```

**Ví dụ 2 — Async State (RemoteData pattern):**

```ts
type RemoteData<T, E = Error> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: E };

// React component với discriminated union
interface UserState {
  users: RemoteData<User[]>;
}

function UserList({ users }: { users: RemoteData<User[]> }) {
  switch (users.status) {
    case "idle":
      return <div>Click to load users</div>;

    case "loading":
      return <Spinner />;

    case "success":
      // TypeScript biết users.data: User[]
      return (
        <ul>
          {users.data.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      );

    case "error":
      // TypeScript biết users.error: Error
      return <ErrorMessage message={users.error.message} />;
  }
}
```

**Ví dụ 3 — API Response:**

```ts
type ApiResponse<T> =
  | { success: true; data: T; statusCode: 200 | 201 }
  | { success: false; error: string; statusCode: 400 | 401 | 403 | 404 | 500 };

async function apiCall<T>(url: string): Promise<ApiResponse<T>> {
  const res = await fetch(url);
  const body = await res.json();

  if (res.ok) {
    return { success: true, data: body, statusCode: res.status as 200 | 201 };
  }
  return {
    success: false,
    error: body.message ?? "Unknown error",
    statusCode: res.status as 400 | 401 | 403 | 404 | 500
  };
}

// Usage
const result = await apiCall<User[]>("/api/users");

if (result.success) {
  // result.data: User[] — TypeScript biết!
  result.data.forEach(u => console.log(u.name));
} else {
  // result.error: string
  console.error(result.error);
  if (result.statusCode === 401) redirectToLogin();
}
```

---

## 6. Conditional Types

### Q6 — [Senior]

**EN:** Explain conditional types in TypeScript — the `extends` keyword, `infer`, and distributive conditional types.

**VI:** Giải thích conditional types — từ khóa `extends`, `infer`, và distributive conditional types.

**Trả lời:**

**Cú pháp cơ bản:**

```ts
type TypeName<T> = T extends string  ? "string"  :
                   T extends number  ? "number"  :
                   T extends boolean ? "boolean" :
                   T extends null    ? "null"     :
                   T extends undefined ? "undefined" :
                   "object";

type A = TypeName<string>;  // "string"
type B = TypeName<number>;  // "number"
type C = TypeName<boolean>; // "boolean"
type D = TypeName<{}>;      // "object"
```

**`infer` keyword — suy ra type từ context:**

```ts
// infer R — TypeScript suy ra R từ cấu trúc của T
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type UnpackPromise<T> = T extends Promise<infer Value> ? Value : T;

type UnpackArray<T> = T extends Array<infer Item> ? Item : T;

// Test
type A = UnpackPromise<Promise<string>>;  // string
type B = UnpackPromise<Promise<User[]>>; // User[]
type C = UnpackPromise<number>;           // number (không phải Promise, trả về T)

type D = UnpackArray<string[]>;  // string
type E = UnpackArray<User[]>;    // User
type F = UnpackArray<number>;    // number (không phải array)

// infer với nested
type DeepUnpackPromise<T> =
  T extends Promise<infer Inner>
    ? DeepUnpackPromise<Inner>
    : T;

type G = DeepUnpackPromise<Promise<Promise<string>>>;  // string
type H = DeepUnpackPromise<Promise<string>>;            // string

// infer first/last element
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;
type Last<T extends any[]>  = T extends [...any[], infer L] ? L : never;
type Rest<T extends any[]>  = T extends [any, ...infer R]   ? R : never;

type I = First<[string, number, boolean]>; // string
type J = Last<[string, number, boolean]>;  // boolean
type K = Rest<[string, number, boolean]>;  // [number, boolean]

// Infer function params với names
type PromisifyFn<T extends (...args: any[]) => any> =
  T extends (...args: infer A) => infer R
    ? (...args: A) => Promise<R>
    : never;

type SyncFn = (x: number, y: string) => boolean;
type AsyncFn = PromisifyFn<SyncFn>; // (x: number, y: string) => Promise<boolean>
```

**Distributive Conditional Types:**

Khi T là naked type parameter và union type, conditional type tự động "distribute" qua từng member:

```ts
// Distributive behavior
type ToArray<T> = T extends any ? T[] : never;

type A = ToArray<string | number>;
// Phân phối: (string extends any ? string[] : never) | (number extends any ? number[] : never)
// = string[] | number[]
// (KHÔNG phải (string | number)[])

// Tắt distributive behavior — wrap T trong tuple
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;
type B = ToArrayNonDist<string | number>; // (string | number)[]

// Ứng dụng — filter union types
type FilterOut<T, U> = T extends U ? never : T;

type WithoutString = FilterOut<string | number | boolean, string>;
// number | boolean

// Ứng dụng — flatten union
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type C = Flatten<string[] | number[] | boolean>;
// string | number | boolean
```

---

## 7. Mapped Types

### Q7 — [Senior]

**EN:** Explain mapped types — `keyof`, `in`, key remapping with `as`, and template literals in mapped types.

**VI:** Giải thích mapped types — `keyof`, `in`, key remapping với `as`, và template literals trong mapped types.

**Trả lời:**

**Mapped Types cơ bản:**

```ts
// Cú pháp: { [K in Keys]: ValueType }
// Duyệt qua từng key và tạo ra type mới

type Stringify<T> = {
  [K in keyof T]: string; // chuyển tất cả value thành string
};

interface User {
  id: number;
  name: string;
  active: boolean;
}

type StringifiedUser = Stringify<User>;
// { id: string; name: string; active: string; }

// Giữ nguyên value type
type Clone<T> = {
  [K in keyof T]: T[K]; // tương đương với T nhưng re-maps mọi property
};
```

**Modifiers trong mapped types:**

```ts
// +readonly, -readonly, +?, -?
type Immutable<T> = {
  +readonly [K in keyof T]: T[K]; // thêm readonly (+ mặc định có thể bỏ qua)
};

type Mutable<T> = {
  -readonly [K in keyof T]: T[K]; // xóa readonly
};

type AllOptional<T> = {
  [K in keyof T]+?: T[K]; // thêm ? (optional)
};

type AllRequired<T> = {
  [K in keyof T]-?: T[K]; // xóa ? (required)
};
```

**Key Remapping với `as` (TypeScript 4.1+):**

```ts
// as clause cho phép rename keys trong mapped type
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User {
  name: string;
  age: number;
  email: string;
}

type UserGetters = Getters<User>;
// {
//   getName: () => string;
//   getAge: () => number;
//   getEmail: () => string;
// }

// Lọc keys với as + never
type FilterKeys<T, ValueType> = {
  [K in keyof T as T[K] extends ValueType ? K : never]: T[K];
};

type OnlyStrings = FilterKeys<User, string>;
// { name: string; email: string; } — loại bỏ age (number)

type OnlyFunctions<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K];
};

// Tạo event handler type
type EventMap = {
  click: MouseEvent;
  keydown: KeyboardEvent;
  resize: UIEvent;
};

type EventHandlers = {
  [K in keyof EventMap as `on${Capitalize<K>}`]: (event: EventMap[K]) => void;
};
// {
//   onClick: (event: MouseEvent) => void;
//   onKeydown: (event: KeyboardEvent) => void;
//   onResize: (event: UIEvent) => void;
// }
```

**Template Literals trong Mapped Types:**

```ts
// CRUD operations generator
type CrudActions<T extends string> = {
  [K in T as `create${Capitalize<K>}`]: () => void;
} & {
  [K in T as `read${Capitalize<K>}`]: (id: number) => void;
} & {
  [K in T as `update${Capitalize<K>}`]: (id: number) => void;
} & {
  [K in T as `delete${Capitalize<K>}`]: (id: number) => void;
};

type UserCrud = CrudActions<"user" | "post">;
// {
//   createUser, createPost
//   readUser, readPost
//   updateUser, updatePost
//   deleteUser, deletePost
// }

// Prefix all keys
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};

type PrefixedUser = Prefixed<User, "initial">;
// { initialName: string; initialAge: number; initialEmail: string; }
```

---

## 8. Template Literal Types

### Q8 — [Mid]

**EN:** Explain template literal types, intrinsic string manipulation types, and the getter pattern.

**VI:** Giải thích template literal types, intrinsic string manipulation types, và getter pattern.

**Trả lời:**

**Template Literal Types cơ bản:**

```ts
type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

type CSSUnit = "px" | "em" | "rem" | "vh" | "vw";
type CSSValue = `${number}${CSSUnit}`;
// `${number}px` | `${number}em` | etc.
// Chú ý: đây là infinite set, TypeScript represent là `${number}px` etc.

// Combine multiple unions — TypeScript tạo tích Cartesian
type Direction = "Top" | "Bottom" | "Left" | "Right";
type Margin = `margin${Direction}`;
// "marginTop" | "marginBottom" | "marginLeft" | "marginRight"

type Padding = `padding${Direction}`;
type BoxModel = `${"margin" | "padding"}${Direction}`;
// "marginTop" | "marginBottom" | ... | "paddingTop" | "paddingBottom" | ...
```

**Intrinsic String Manipulation Types:**

```ts
// Built-in string transformations
type A = Uppercase<"hello">;   // "HELLO"
type B = Lowercase<"HELLO">;   // "hello"
type C = Capitalize<"hello">;  // "Hello"
type D = Uncapitalize<"Hello">; // "hello"

// Ứng dụng thực tế
type HTTPMethod = "get" | "post" | "put" | "delete" | "patch";
type UpperMethod = Uppercase<HTTPMethod>;
// "GET" | "POST" | "PUT" | "DELETE" | "PATCH"

// Route params extraction
type ExtractRouteParams<T extends string> =
  string extends T ? Record<string, string> :
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractRouteParams<Rest>]: string }
  : T extends `${infer _Start}:${infer Param}`
    ? { [K in Param]: string }
  : {};

type Params = ExtractRouteParams<"/users/:userId/posts/:postId">;
// { userId: string; postId: string; }
```

**Getter Pattern — `get${Capitalize<string & P>}`:**

```ts
// Pattern phổ biến trong TypeScript advanced types
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

// string & K — tại sao cần string & K?
// K là keyof T — có thể là string | number | symbol
// Capitalize chỉ hoạt động với string
// string & K — intersect để đảm bảo K là string

interface FormState {
  username: string;
  email: string;
  age: number;
}

type FormGetters = Getters<FormState>;
// {
//   getUsername: () => string;
//   getEmail: () => string;
//   getAge: () => number;
// }

type FormSetters = Setters<FormState>;
// {
//   setUsername: (value: string) => void;
//   setEmail: (value: string) => void;
//   setAge: (value: number) => void;
// }

type FormAccessors = FormGetters & FormSetters;
// Có đầy đủ getters và setters

// Implement class từ type
class Form implements FormAccessors {
  private state: FormState = { username: "", email: "", age: 0 };

  getUsername = () => this.state.username;
  getEmail    = () => this.state.email;
  getAge      = () => this.state.age;

  setUsername = (v: string) => { this.state.username = v; };
  setEmail    = (v: string) => { this.state.email = v; };
  setAge      = (v: number) => { this.state.age = v; };
}
```

---

## 9. Declaration Merging & Module Augmentation

### Q9 — [Senior]

**EN:** What is declaration merging? How do you augment existing modules and the global scope?

**VI:** Declaration merging là gì? Làm thế nào để augment module và global scope?

**Trả lời:**

**Interface Merging:**

```ts
// Trong cùng một scope, TypeScript tự động merge các interface cùng tên
interface Window {
  analytics: AnalyticsService;
}

interface Window {
  featureFlags: Record<string, boolean>;
}

// Kết quả:
// interface Window {
//   analytics: AnalyticsService;
//   featureFlags: Record<string, boolean>;
//   // ...tất cả properties mặc định của Window
// }

window.analytics.track("page_view"); // OK
window.featureFlags["new_ui"];       // OK
```

**Module Augmentation — mở rộng third-party modules:**

```ts
// File: src/types/express.d.ts
import "express";

declare module "express" {
  interface Request {
    user?: {
      id: string;
      email: string;
      role: "admin" | "user";
    };
    requestId: string;
  }
}

// Bây giờ Express Request có thêm properties:
app.use((req, res, next) => {
  req.user?.id;      // OK — TypeScript biết type
  req.requestId;     // OK
  next();
});
```

**Mở rộng interface của thư viện:**

```ts
// Augment Vue 3 global properties
// File: src/types/vue.d.ts
import { ComponentCustomProperties } from "vue";

declare module "vue" {
  interface ComponentCustomProperties {
    $toast: (message: string) => void;
    $router: Router;
    $store: Store<RootState>;
  }
}

// Trong Vue component:
// this.$toast("Hello!"); // OK với TypeScript

// Augment Pinia store
declare module "pinia" {
  export interface PiniaCustomProperties {
    router: Router;
  }
}

// Augment axios
import "axios";
declare module "axios" {
  export interface AxiosRequestConfig {
    retry?: number;
    _retry?: boolean;
  }
}
```

**Global augmentation:**

```ts
// Thêm vào global scope
declare global {
  interface Array<T> {
    first(): T | undefined;
    last(): T | undefined;
    groupBy<K extends string>(
      keyFn: (item: T) => K
    ): Record<K, T[]>;
  }

  interface String {
    toTitleCase(): string;
    truncate(maxLength: number): string;
  }

  var ENV: "development" | "staging" | "production";
}

// Implementation
Array.prototype.first = function() { return this[0]; };
Array.prototype.last  = function() { return this[this.length - 1]; };

String.prototype.toTitleCase = function() {
  return this.replace(/\w\S*/g, txt =>
    txt.charAt(0).toUpperCase() + txt.slice(1).toLowerCase()
  );
};

// Usage
[1, 2, 3].first(); // 1 — TypeScript không báo lỗi
"hello world".toTitleCase(); // "Hello World"
```

**Namespace merging:**

```ts
// Merge namespace với function/class
function createLogger(name: string) {
  return {
    log: (msg: string) => console.log(`[${name}] ${msg}`),
    error: (msg: string) => console.error(`[${name}] ${msg}`)
  };
}

namespace createLogger {
  export type Logger = ReturnType<typeof createLogger>;
  export const defaultName = "App";
}

const logger = createLogger(createLogger.defaultName);
type L = createLogger.Logger; // { log: ...; error: ...; }
```

---

## 10. Strict Mode

### Q10 — [Mid]

**EN:** What are the implications of TypeScript's strict mode options: `strictNullChecks`, `noImplicitAny`, and `strictFunctionTypes`?

**VI:** Các option strict mode của TypeScript ảnh hưởng như thế nào: `strictNullChecks`, `noImplicitAny`, và `strictFunctionTypes`?

**Trả lời:**

Bật `"strict": true` trong `tsconfig.json` bật nhiều checks cùng lúc. Đây là các option quan trọng nhất:

**`strictNullChecks` — bảo vệ khỏi null/undefined:**

```ts
// Khi TẮT strictNullChecks (default off)
let name: string = null;    // OK — nguy hiểm!
let age: number = undefined; // OK — nguy hiểm!

// Khi BẬT strictNullChecks
let name: string = null;    // Error: null không phải string
let name: string | null = null;    // OK — explicit
let name: string | undefined = undefined; // OK — explicit

// Phải handle null/undefined trước khi dùng
function greet(name: string | null) {
  console.log(name.toUpperCase()); // Error: name có thể null

  // Cần narrow:
  if (name !== null) {
    console.log(name.toUpperCase()); // OK
  }

  // Hoặc non-null assertion (!)  — dùng khi chắc chắn không null
  console.log(name!.toUpperCase()); // OK nhưng nguy hiểm nếu null ở runtime

  // Hoặc optional chaining + nullish coalescing
  console.log((name ?? "stranger").toUpperCase()); // Safe
}

// strictNullChecks bắt được nhiều bug phổ biến
const el = document.getElementById("app");
el.innerHTML = "Hello"; // Error: el có thể null

// Fix:
el?.innerHTML = "Hello";
// hoặc
if (el) el.innerHTML = "Hello";
```

**`noImplicitAny` — không cho phép any ngầm định:**

```ts
// Khi TẮT noImplicitAny
function process(data) { // data ngầm định là any
  return data.someMethod(); // không có lỗi TypeScript
}

// Khi BẬT noImplicitAny
function process(data) { // Error: Parameter 'data' implicitly has an 'any' type
  return data.someMethod();
}

// Phải khai báo type rõ ràng
function process(data: ProcessableData) { /* OK */ }
function process(data: unknown) { /* OK — nhưng phải narrow trước khi dùng */ }
function process(data: any) { /* OK — explicit any, bạn tự chịu trách nhiệm */ }

// Với array và object
const items = []; // Error: Inferred type 'any[]'
const items: string[] = []; // OK

function getKey(obj, key) { // Error: 2 implicit any parameters
  return obj[key];
}

function getKey<T, K extends keyof T>(obj: T, key: K): T[K] { // OK
  return obj[key];
}
```

**`strictFunctionTypes` — contravariance trong function params:**

```ts
// strictFunctionTypes kiểm tra variance của function types

type Handler = (event: MouseEvent) => void;

// MouseEvent extends UIEvent extends Event
const mouseHandler: Handler = (e: MouseEvent) => console.log(e.button); // OK
const uiHandler: Handler = (e: UIEvent) => console.log(e.detail);       // Error!
// Error: UIEvent không đủ specific — nếu gọi với MouseEvent, e.button sẽ không tồn tại

// Contravariance: function param types được check ngược
// Handler expects MouseEvent — nên chỉ chấp nhận function nhận type >= MouseEvent
const eventHandler: Handler = (e: Event) => console.log(e.type); // OK!
// Event là supertype của MouseEvent — safe vì MouseEvent có tất cả properties của Event

// Ví dụ thực tế với callbacks
type Callback<T> = (value: T) => void;

function runWith<T>(value: T, callback: Callback<T>): void {
  callback(value);
}

const numberCallback: Callback<number> = (n) => console.log(n * 2);
runWith(42, numberCallback); // OK

// Method shorthand (trong interface/class) — BIVARIANT kể cả với strictFunctionTypes
// Đây là trade-off do backward compatibility
interface Collection<T> {
  forEach(callback: (item: T) => void): void;  // bivariant
  map: (callback: (item: T) => any) => any[];  // contravariant (strict)
}
```

**tsconfig.json khuyến nghị:**

```json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true
  }
}
```

---

## 11. Type Narrowing & Type Guards

### Q11 — [Mid]

**EN:** Explain type narrowing techniques — `typeof`, `instanceof`, `in`, and custom type guards with the `is` keyword.

**VI:** Giải thích các kỹ thuật type narrowing — `typeof`, `instanceof`, `in`, và custom type guards với từ khóa `is`.

**Trả lời:**

**Type Narrowing** là quá trình TypeScript thu hẹp type từ broad (ví dụ `string | number`) xuống specific (`string`) dựa trên code flow.

**`typeof` narrowing:**

```ts
function processInput(input: string | number | boolean) {
  if (typeof input === "string") {
    // input: string
    return input.toUpperCase();
  }

  if (typeof input === "number") {
    // input: number
    return input.toFixed(2);
  }

  // input: boolean (TypeScript loại string và number)
  return input ? "yes" : "no";
}

// typeof checks: "string" | "number" | "bigint" | "boolean" | "symbol" | "undefined" | "function" | "object"
// Lưu ý: typeof null === "object" — bug JavaScript lịch sử
```

**`instanceof` narrowing:**

```ts
class ApiError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
    this.name = "ApiError";
  }
}

class NetworkError extends Error {
  constructor(public retryAfter?: number) {
    super("Network unavailable");
    this.name = "NetworkError";
  }
}

function handleError(error: Error | ApiError | NetworkError) {
  if (error instanceof ApiError) {
    // error: ApiError — có statusCode
    if (error.statusCode === 401) redirectToLogin();
    else showErrorMessage(error.message);
    return;
  }

  if (error instanceof NetworkError) {
    // error: NetworkError — có retryAfter
    if (error.retryAfter) retryLater(error.retryAfter);
    else showOfflineMessage();
    return;
  }

  // error: Error (base)
  console.error("Unexpected error:", error.message);
}
```

**`in` narrowing:**

```ts
interface Cat  { type: "cat";  meow(): void; lives: number }
interface Dog  { type: "dog";  bark(): void; breed: string }
interface Fish { type: "fish"; swim(): void; }

type Animal = Cat | Dog | Fish;

function makeSound(animal: Animal) {
  if ("meow" in animal) {
    // animal: Cat
    animal.meow();
  } else if ("bark" in animal) {
    // animal: Dog
    animal.bark();
  } else {
    // animal: Fish
    animal.swim();
  }
}

// `in` operator checks property existence — useful for duck typing
interface Square { kind: "square"; size: number }
interface Circle { kind: "circle"; radius: number }
type Shape = Square | Circle;

function area(shape: Shape): number {
  if ("size" in shape) {
    return shape.size ** 2; // TypeScript: shape is Square
  }
  return Math.PI * shape.radius ** 2; // TypeScript: shape is Circle
}
```

**Custom Type Guards — `is` keyword:**

```ts
// Type predicate: (param: Type) => param is NarrowedType
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    "email" in value
  );
}

// Usage
function processValue(value: unknown) {
  if (isString(value)) {
    // value: string — TypeScript biết nhờ type predicate
    console.log(value.toUpperCase());
  }

  if (isUser(value)) {
    // value: User
    console.log(value.name);
  }
}

// Generic type guard
function isNotNull<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

const maybeUsers: (User | null)[] = [user1, null, user2, null, user3];
const validUsers: User[] = maybeUsers.filter(isNotNull); // User[] — không có null

// Assertion function (TypeScript 3.7+)
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new TypeError(`Expected string, got ${typeof value}`);
  }
}

function assertDefined<T>(value: T | null | undefined): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error("Value is null or undefined");
  }
}

// Usage
function processConfig(config: unknown) {
  assertIsString(config); // throw nếu không phải string
  // config: string sau dòng này
  console.log(config.toUpperCase()); // OK
}

// Discriminated union narrowing (kết hợp các kỹ thuật)
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: Error };

function isOk<T>(result: Result<T>): result is { ok: true; value: T } {
  return result.ok === true;
}

const result: Result<User> = await fetchUser(1);

if (isOk(result)) {
  // result: { ok: true; value: User }
  console.log(result.value.name);
}

// Hoặc
if (result.ok) {
  result.value; // TypeScript narrow vì discriminant `ok`
}
```

---

## 12. Decorators

### Q12 — [Senior]

**EN:** What are decorators in TypeScript? Explain class, method, and property decorators as used in frameworks like NestJS.

**VI:** Decorator trong TypeScript là gì? Giải thích class, method, và property decorators như trong NestJS.

**Trả lời:**

**Decorator** là một pattern cho phép thêm metadata hoặc thay đổi hành vi của class, method, property, hay parameter mà không sửa code gốc. Cần bật `"experimentalDecorators": true` trong tsconfig.

**Class Decorator:**

```ts
// Class decorator nhận constructor của class
function Controller(path: string) {
  return function(target: Function) {
    // Gắn metadata vào class
    Reflect.defineMetadata("path", path, target);
  };
}

function Injectable() {
  return function(target: Function) {
    Reflect.defineMetadata("injectable", true, target);
  };
}

// Usage (NestJS-style)
@Controller("/users")
@Injectable()
class UserController {
  // ...
}

// Lấy metadata
const path = Reflect.getMetadata("path", UserController); // "/users"

// Decorator factory với logging
function Log(target: Function) {
  const originalMethods = Object.getOwnPropertyNames(target.prototype);
  originalMethods.forEach(method => {
    const original = target.prototype[method];
    if (typeof original === "function") {
      target.prototype[method] = function(...args: any[]) {
        console.log(`Calling ${method} with`, args);
        const result = original.apply(this, args);
        console.log(`${method} returned`, result);
        return result;
      };
    }
  });
}
```

**Method Decorator:**

```ts
// Method decorator: (target, propertyKey, descriptor) => descriptor | void
function HttpGet(path: string) {
  return function(
    target: object,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    Reflect.defineMetadata("method", "GET", target, propertyKey);
    Reflect.defineMetadata("path", path, target, propertyKey);
    return descriptor;
  };
}

function Roles(...roles: string[]) {
  return function(
    target: object,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    Reflect.defineMetadata("roles", roles, target, propertyKey);
    return descriptor;
  };
}

// Catch-and-rethrow decorator
function Catch(ErrorClass: new (...args: any[]) => Error) {
  return function(
    target: object,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value;
    descriptor.value = async function(...args: any[]) {
      try {
        return await original.apply(this, args);
      } catch (error) {
        if (error instanceof ErrorClass) {
          console.error(`Caught ${ErrorClass.name}:`, error.message);
          throw error; // re-throw
        }
        throw error;
      }
    };
    return descriptor;
  };
}

// Usage
class UserController {
  @HttpGet("/")
  @Roles("admin", "user")
  async findAll() { /* ... */ }

  @HttpGet("/:id")
  @Catch(NotFoundException)
  async findOne(@Param("id") id: string) { /* ... */ }
}
```

**Property Decorator:**

```ts
// Property decorator: (target, propertyKey) => void
function MinLength(min: number) {
  return function(target: object, propertyKey: string) {
    let value: string;

    Object.defineProperty(target, propertyKey, {
      get() { return value; },
      set(newValue: string) {
        if (typeof newValue === "string" && newValue.length < min) {
          throw new Error(`${propertyKey} must be at least ${min} characters`);
        }
        value = newValue;
      },
      enumerable: true,
      configurable: true
    });
  };
}

function Column(options?: { nullable?: boolean; default?: any }) {
  return function(target: object, propertyKey: string) {
    const columns = Reflect.getMetadata("columns", target) ?? [];
    columns.push({ name: propertyKey, ...options });
    Reflect.defineMetadata("columns", columns, target);
  };
}

// Usage (TypeORM-style)
class UserEntity {
  @Column({ nullable: false })
  @MinLength(2)
  name: string = "";

  @Column({ nullable: true })
  bio?: string;
}
```

**Parameter Decorator:**

```ts
// Parameter decorator: (target, methodName, paramIndex) => void
function Body() {
  return function(target: object, methodName: string, parameterIndex: number) {
    const params = Reflect.getMetadata("body_params", target, methodName) ?? [];
    params.push(parameterIndex);
    Reflect.defineMetadata("body_params", params, target, methodName);
  };
}

function Param(name: string) {
  return function(target: object, methodName: string, parameterIndex: number) {
    const params = Reflect.getMetadata("route_params", target, methodName) ?? [];
    params.push({ index: parameterIndex, name });
    Reflect.defineMetadata("route_params", params, target, methodName);
  };
}

// Usage
class UserController {
  async create(@Body() createUserDto: CreateUserDto) { /* ... */ }
  async findOne(@Param("id") id: string) { /* ... */ }
}
```

**tsconfig.json cho Decorators:**

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "target": "ES2017"
  }
}
```

**Lưu ý:** TypeScript 5.0 giới thiệu **Stage 3 Decorators** (tiêu chuẩn ECMAScript) với API khác biệt. NestJS, TypeORM, và các framework cũ vẫn dùng decorator kiểu cũ (`experimentalDecorators`). Khi phỏng vấn, hỏi thêm về version TypeScript và framework đang dùng.

---

*Cập nhật lần cuối: 2026-03-25*
