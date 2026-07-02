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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What's the difference between `interface` and `type` in TypeScript? Which one do you prefer?"**

**Model Answer:**
"Both let you describe the shape of an object, but they have a few key differences. The biggest one is that interfaces support declaration merging — if you declare the same interface name twice, TypeScript merges them automatically. That's how you augment third-party types like Express's `Request`. Types can't do that; you'll get a duplicate identifier error.

The other difference is flexibility. `type` can represent almost anything — unions, intersections, tuples, primitive aliases, conditional types — while `interface` is really scoped to object shapes and class contracts.

In practice, I use `interface` for API response shapes, component prop types, and anything I'd want to `implement` in a class. I use `type` for unions like `type Status = 'pending' | 'active'`, function signatures, and anything computed with utility types. The important thing is picking a convention and being consistent within a project."

**Trả lời (Tiếng Việt):**
"Cả hai đều cho phép bạn mô tả hình dạng của một object, nhưng có vài điểm khác biệt chính. Lớn nhất là interface hỗ trợ declaration merging — nếu bạn khai báo cùng tên interface hai lần, TypeScript tự động merge chúng. Đó là cách bạn augment type của bên thứ ba như `Request` của Express. Type không làm được điều đó; bạn sẽ gặp lỗi duplicate identifier.

Điểm khác biệt còn lại là tính linh hoạt. `type` có thể đại diện cho gần như mọi thứ — union, intersection, tuple, primitive alias, conditional type — trong khi `interface` thực sự chỉ phù hợp với object shape và class contract.

Trong thực tế, tôi dùng `interface` cho API response shape, component prop type, và bất cứ thứ gì tôi muốn `implement` trong class. Tôi dùng `type` cho union như `type Status = 'pending' | 'active'`, function signature, và bất cứ thứ gì được tính toán với utility type. Quan trọng là chọn một convention và nhất quán trong dự án."

---

**Q2 (Junior/Mid): "Can an interface extend a type, and vice versa?"**

**Model Answer:**
"Yes, both directions work. An interface can extend a type alias using `extends`, and a type can intersect with an interface using `&`. For example:

```ts
type Animal = { name: string };
interface Dog extends Animal { breed: string }       // interface extends type alias

interface Serializable { serialize(): string }
type Document = Serializable & { title: string }     // type intersects interface
```

This is useful when you're working with third-party types that use one style and you want to build on top of them in your own style."

**Trả lời (Tiếng Việt):**
"Có, cả hai chiều đều hoạt động. Interface có thể extend type alias bằng `extends`, và type có thể intersect với interface bằng `&`. Ví dụ:

```ts
type Animal = { name: string };
interface Dog extends Animal { breed: string }       // interface extends type alias

interface Serializable { serialize(): string }
type Document = Serializable & { title: string }     // type intersects interface
```

Điều này hữu ích khi bạn làm việc với type của bên thứ ba dùng một style và bạn muốn xây dựng thêm theo style của mình."

---

**Q3 (Mid): "When would declaration merging with `interface` actually be useful in a real project?"**

**Model Answer:**
"The classic use case is module augmentation — extending types from libraries you don't own. For instance, when you add a custom `user` property to an Express `Request` object in middleware, you declare the same `Request` interface in a `.d.ts` file and TypeScript merges it. Without that, every route handler would need a type cast.

Another common one is adding custom properties to `Window` — say you set `window.analytics` on startup and want TypeScript to know about it. You just re-declare the `Window` interface with that property, and everything downstream gets the correct type for free.

It's also used for polyfills — if you add a method to `Array.prototype`, you declare it in the `Array<T>` interface so TypeScript won't complain when you call it."

**Trả lời (Tiếng Việt):**
"Use case kinh điển là module augmentation — mở rộng type từ thư viện bạn không sở hữu. Ví dụ, khi bạn thêm property `user` tùy chỉnh vào object `Request` của Express trong middleware, bạn khai báo lại interface `Request` trong file `.d.ts` và TypeScript sẽ merge nó. Không có điều đó, mỗi route handler cần type cast.

Một use case phổ biến khác là thêm custom property vào `Window` — ví dụ bạn set `window.analytics` khi khởi động và muốn TypeScript biết về nó. Bạn chỉ cần khai báo lại interface `Window` với property đó, và mọi thứ phía sau tự có type đúng.

Nó cũng dùng cho polyfill — nếu bạn thêm method vào `Array.prototype`, bạn khai báo nó trong interface `Array<T>` để TypeScript không phàn nàn khi bạn gọi nó."

---

**Q4 (Senior): "Why does TypeScript sometimes display a type alias name in errors, and sometimes expand it to its full structure? Does it matter?"**

**Model Answer:**
"TypeScript tries to be helpful in error messages by showing the name of a type alias when it's readable and distinct. But if the alias is just a simple intersection or wraps a built-in, it may expand it instead. This is mostly a display heuristic, not a semantic difference.

It matters in practice when you're debugging — named aliases like `UserId` or `ApiResponse<User>` make errors much easier to scan than a fully expanded object literal with 15 properties. One trick to force TypeScript to preserve a name is to use a conditional type trick: `type Prettify<T> = { [K in keyof T]: T[K] } & {}` — this expands the structure for readability at the call site while still being derived from named types.

In general, preferring named types over inline ones makes compiler messages much friendlier."

**Trả lời (Tiếng Việt):**
"TypeScript cố gắng hữu ích trong thông báo lỗi bằng cách hiển thị tên của type alias khi nó dễ đọc và rõ ràng. Nhưng nếu alias chỉ là một intersection đơn giản hay bao bọc một built-in, nó có thể expand ra thay. Đây chủ yếu là heuristic hiển thị, không phải sự khác biệt về ngữ nghĩa.

Điều này quan trọng trong thực tế khi bạn debug — tên alias có nghĩa như `UserId` hay `ApiResponse<User>` làm error dễ scan hơn nhiều so với object literal được expand đầy đủ với 15 property. Một trick để ép TypeScript giữ tên là dùng conditional type trick: `type Prettify<T> = { [K in keyof T]: T[K] } & {}` — cái này expand structure để dễ đọc ở call site trong khi vẫn được derive từ named type.

Nói chung, ưu tiên named type hơn inline type làm compiler message thân thiện hơn nhiều."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What are generics in TypeScript and why do we need them?"**

**Model Answer:**
"Generics are like type placeholders — you write a function or class once and let the caller decide what type it works with, while still keeping full type safety. Without generics, you'd have to either duplicate code for every type, or use `any` and lose all the type information.

The simplest example is an identity function. If you write `function identity(value: any): any`, TypeScript loses track of what type comes back. But with `function identity<T>(value: T): T`, if you pass in a string, TypeScript knows the return is also a string. The `T` gets inferred automatically — you usually don't have to write it explicitly."

**Trả lời (Tiếng Việt):**
"Generic giống như placeholder cho type — bạn viết function hay class một lần và để caller quyết định nó làm việc với type gì, trong khi vẫn giữ được đầy đủ type safety. Không có generic, bạn phải hoặc duplicate code cho mọi type, hoặc dùng `any` và mất tất cả thông tin type.

Ví dụ đơn giản nhất là identity function. Nếu bạn viết `function identity(value: any): any`, TypeScript mất dấu type nào trả về. Nhưng với `function identity<T>(value: T): T`, nếu bạn truyền vào string, TypeScript biết kết quả trả về cũng là string. `T` được suy ra tự động — thường bạn không cần viết nó tường minh."

---

**Q2 (Mid): "What are generic constraints and when would you use them?"**

**Model Answer:**
"Constraints let you say 'this generic T must have at least these properties.' You use `extends` for that. A common one is `<T extends { id: number }>` in a repository pattern — you're saying the generic type must have an `id` field, so the repository can use it safely.

Another pattern I use a lot is `<T, K extends keyof T>` — that lets you write a type-safe property getter where TypeScript knows the return type is `T[K]`. If you try to pass a key that doesn't exist on the object, you get a compile error rather than a runtime undefined.

The rule of thumb: when your generic function needs to call a method or access a property on T, use a constraint to tell TypeScript it's safe."

**Trả lời (Tiếng Việt):**
"Constraint cho phép bạn nói 'generic T này phải có ít nhất những property này.' Bạn dùng `extends` cho điều đó. Một cái phổ biến là `<T extends { id: number }>` trong repository pattern — bạn đang nói generic type phải có field `id`, nên repository có thể dùng nó an toàn.

Một pattern tôi hay dùng là `<T, K extends keyof T>` — cho phép bạn viết type-safe property getter nơi TypeScript biết kiểu trả về là `T[K]`. Nếu bạn cố truyền key không tồn tại trên object, bạn nhận compile error thay vì runtime undefined.

Nguyên tắc đơn giản: khi generic function của bạn cần gọi method hay truy cập property trên T, dùng constraint để nói với TypeScript rằng điều đó an toàn."

---

**Q3 (Mid): "How would you write a generic React hook for data fetching?"**

**Model Answer:**
"The pattern is to make the hook generic over the response data type, then use that type in the state declarations. Something like:

```ts
function useFetch<T>(url: string): { data: T | null; loading: boolean; error: Error | null } {
  const [data, setData] = useState<T | null>(null);
  // ...fetch logic...
  return { data, loading, error };
}

const { data } = useFetch<User[]>('/api/users');
// data is User[] | null — TypeScript knows the shape
```

The benefit is that every consumer gets full autocomplete on `data` without any type casting. If the API changes and `User` no longer has a `name` property, TypeScript catches all the breakages at once."

**Trả lời (Tiếng Việt):**
"Pattern là làm hook generic trên kiểu dữ liệu response, rồi dùng kiểu đó trong khai báo state. Kiểu như:

```ts
function useFetch<T>(url: string): { data: T | null; loading: boolean; error: Error | null } {
  const [data, setData] = useState<T | null>(null);
  // ...fetch logic...
  return { data, loading, error };
}

const { data } = useFetch<User[]>('/api/users');
// data là User[] | null — TypeScript biết hình dạng
```

Lợi ích là mỗi consumer nhận được đầy đủ autocomplete trên `data` mà không cần type cast. Nếu API thay đổi và `User` không còn property `name`, TypeScript bắt tất cả các chỗ bị ảnh hưởng ngay lập tức."

---

**Q4 (Senior): "What's the difference between a generic default and a generic constraint? Can you use both on the same parameter?"**

**Model Answer:**
"A constraint (`extends`) sets the minimum shape a type must satisfy — TypeScript will error if you pass something that doesn't match. A default sets what TypeScript assumes if the caller doesn't provide a type argument.

Yes, you can use both together: `<T extends object = Record<string, unknown>>`. Here, `T` must be an object type, and if the caller doesn't specify, it defaults to `Record<string, unknown>`. This is useful for things like an API response wrapper where you want it typed generically but still usable without explicit annotation.

The key distinction is: the constraint is enforced at the call site, while the default is only used when the type is not inferred or provided."

**Trả lời (Tiếng Việt):**
"Constraint (`extends`) đặt ra hình dạng tối thiểu mà type phải thỏa mãn — TypeScript sẽ báo lỗi nếu bạn truyền thứ gì không match. Default đặt ra điều TypeScript giả định nếu caller không cung cấp type argument.

Có, bạn có thể dùng cả hai cùng nhau: `<T extends object = Record<string, unknown>>`. Ở đây, `T` phải là object type, và nếu caller không chỉ định, nó mặc định là `Record<string, unknown>`. Điều này hữu ích cho các thứ như wrapper API response nơi bạn muốn nó được type generic nhưng vẫn dùng được mà không cần annotation tường minh.

Điểm khác biệt chính: constraint được enforce tại call site, trong khi default chỉ được dùng khi type không được suy ra hay cung cấp."

---

**Q5 (Senior): "What happens with variance when generics are used in function parameters vs return types?"**

**Model Answer:**
"This is about covariance and contravariance. Return types are covariant — it's safe for a function to return a subtype of what's declared. Parameter types are contravariant — it's safe for a function to accept a supertype of what's declared.

In practice with generics: a `Producer<T>` that returns `T` is covariant in T, so you can use a `Producer<Dog>` where a `Producer<Animal>` is expected. But a `Consumer<T>` that accepts `T` as a parameter is contravariant, so you need `Consumer<Animal>` where `Consumer<Dog>` is expected — because the consumer must be able to handle any `Dog`, and an `Animal` consumer can.

TypeScript with `strictFunctionTypes` enforces contravariance on function types but not on method shorthand syntax, which is a known trade-off for backward compatibility. This matters when you're building utility types or higher-order functions."

**Trả lời (Tiếng Việt):**
"Đây là về covariance và contravariance. Return type là covariant — an toàn khi function trả về subtype của những gì được khai báo. Parameter type là contravariant — an toàn khi function chấp nhận supertype của những gì được khai báo.

Trong thực tế với generic: một `Producer<T>` trả về `T` là covariant trong T, nên bạn có thể dùng `Producer<Dog>` ở chỗ cần `Producer<Animal>`. Nhưng `Consumer<T>` nhận `T` làm parameter là contravariant, nên bạn cần `Consumer<Animal>` ở chỗ cần `Consumer<Dog>` — vì consumer phải xử lý được bất kỳ `Dog` nào, và `Animal` consumer thì có thể.

TypeScript với `strictFunctionTypes` enforce contravariance trên function type nhưng không phải trên method shorthand syntax, đây là trade-off đã biết để backward compatibility. Điều này quan trọng khi bạn xây dựng utility type hay higher-order function."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What's the difference between `Partial` and `Required` in TypeScript?"**

**Model Answer:**
"`Partial<T>` makes every property in T optional — it adds a `?` to everything. `Required<T>` does the opposite — it removes the `?` from every optional property and makes them all required. 

The most common use case for `Partial` is update functions — when you want to allow patching an object with only the fields that changed, so `function updateUser(id: number, changes: Partial<User>)` lets the caller pass just `{ name: 'Alice' }` without supplying every field. `Required` is useful when you have a config type with all optional fields but at a certain point in the code you've set all the defaults and want TypeScript to treat them as definitely present."

**Trả lời (Tiếng Việt):**
"`Partial<T>` làm mọi property trong T thành optional — nó thêm `?` vào tất cả. `Required<T>` làm ngược lại — nó xóa `?` khỏi mọi optional property và làm tất cả required.

Use case phổ biến nhất của `Partial` là update function — khi bạn muốn cho phép patch object chỉ với các field đã thay đổi, nên `function updateUser(id: number, changes: Partial<User>)` cho phép caller truyền chỉ `{ name: 'Alice' }` mà không cần cung cấp mọi field. `Required` hữu ích khi bạn có config type với tất cả field optional nhưng tại một thời điểm nhất định trong code bạn đã set tất cả default và muốn TypeScript coi chúng là definitely present."

---

**Q2 (Junior/Mid): "When would you use `Pick` vs `Omit`? Are they interchangeable?"**

**Model Answer:**
"They're two sides of the same coin — `Pick` says 'give me only these fields,' `Omit` says 'give me everything except these fields.' Which one reads more clearly depends on the size of the type.

If I have a `User` type with 10 fields and I want a type with 2 fields for a card preview, `Pick<User, 'id' | 'name'>` is cleaner — I'm listing what I want. But if I want everything except the `password` and `hashedSalt` fields, `Omit<User, 'password' | 'hashedSalt'>` is cleaner — I'm listing the exceptions. The rule of thumb: use `Pick` when you want a small subset, use `Omit` when you want almost everything."

**Trả lời (Tiếng Việt):**
"Chúng là hai mặt của cùng một đồng xu — `Pick` nói 'cho tôi chỉ những field này,' `Omit` nói 'cho tôi mọi thứ trừ những field này.' Cái nào đọc rõ ràng hơn phụ thuộc vào kích thước của type.

Nếu tôi có type `User` với 10 field và muốn type với 2 field cho card preview, `Pick<User, 'id' | 'name'>` sạch hơn — tôi đang liệt kê những gì tôi muốn. Nhưng nếu tôi muốn mọi thứ trừ field `password` và `hashedSalt`, `Omit<User, 'password' | 'hashedSalt'>` sạch hơn — tôi đang liệt kê các ngoại lệ. Nguyên tắc đơn giản: dùng `Pick` khi bạn muốn subset nhỏ, dùng `Omit` khi bạn muốn gần như mọi thứ."

---

**Q3 (Mid): "How does `ReturnType` work, and when is it useful?"**

**Model Answer:**
"`ReturnType<T>` extracts the return type of a function type. Under the hood it uses a conditional type with `infer`: `T extends (...args: any) => infer R ? R : never`. 

It's really useful when you don't own or control the function's return type — for instance, if a library function returns a complex object and the author didn't export the type explicitly. Instead of copy-pasting or re-declaring the type, you write `type LibResult = ReturnType<typeof someLibraryFunction>` and you get the type automatically, staying in sync if the library updates.

I also use it with factory functions — `type Store = ReturnType<typeof createStore>` — because the store type is implicitly defined by what the factory returns."

**Trả lời (Tiếng Việt):**
"`ReturnType<T>` trích xuất return type của function type. Bên trong nó dùng conditional type với `infer`: `T extends (...args: any) => infer R ? R : never`.

Nó thực sự hữu ích khi bạn không sở hữu hay kiểm soát return type của function — ví dụ, nếu một library function trả về object phức tạp và tác giả không export type tường minh. Thay vì copy-paste hay re-khai báo type, bạn viết `type LibResult = ReturnType<typeof someLibraryFunction>` và tự động nhận type đó, giữ đồng bộ nếu library cập nhật.

Tôi cũng dùng nó với factory function — `type Store = ReturnType<typeof createStore>` — vì store type được định nghĩa ngầm bởi những gì factory trả về."

---

**Q4 (Mid/Senior): "Can you implement `Partial` from scratch? Walk me through it."**

**Model Answer:**
"Sure. `Partial<T>` maps over every key in T and marks it optional:

```ts
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};
```

`keyof T` gives a union of all the property names. `in` iterates over them like a for-loop at the type level. `T[K]` gives the original value type for each key. And the `?` makes each property optional. That's all `Partial` does — it's a direct mapped type with the optional modifier added.

If you want `Required`, you use `-?` instead of `?` — the minus sign removes the optional modifier. And for `Readonly`, you add `readonly` before `[K in keyof T]`."

**Trả lời (Tiếng Việt):**
"Chắc chắn rồi. `Partial<T>` map qua mọi key trong T và đánh dấu nó optional:

```ts
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};
```

`keyof T` cho ra union của tất cả tên property. `in` lặp qua chúng như vòng lặp for ở type level. `T[K]` cho ra value type gốc của mỗi key. Và `?` làm mỗi property optional. Đó là tất cả những gì `Partial` làm — đây là mapped type trực tiếp với optional modifier được thêm vào.

Nếu bạn muốn `Required`, dùng `-?` thay vì `?` — dấu trừ xóa optional modifier. Và với `Readonly`, thêm `readonly` trước `[K in keyof T]`."

---

**Q5 (Senior): "What's the difference between `Exclude` and `Omit`? Developers sometimes mix them up."**

**Model Answer:**
"They operate on completely different things. `Exclude` works on union types — it removes members from a union. `Omit` works on object types — it removes keys from an object shape.

`Exclude<'a' | 'b' | 'c', 'a'>` gives you `'b' | 'c'`. But `Omit<User, 'password'>` gives you the `User` type without the `password` property.

Interestingly, `Omit` is actually implemented using `Exclude` internally: `Omit<T, K> = Pick<T, Exclude<keyof T, K>>`. It first uses `Exclude` to remove the unwanted keys from the `keyof T` union, then uses `Pick` to build the resulting object type from the remaining keys. So `Exclude` is the lower-level primitive and `Omit` is the higher-level utility built on top."

**Trả lời (Tiếng Việt):**
"Chúng hoạt động trên các thứ hoàn toàn khác nhau. `Exclude` hoạt động trên union type — nó loại bỏ member khỏi union. `Omit` hoạt động trên object type — nó loại bỏ key khỏi object shape.

`Exclude<'a' | 'b' | 'c', 'a'>` cho bạn `'b' | 'c'`. Nhưng `Omit<User, 'password'>` cho bạn type `User` không có property `password`.

Thú vị là `Omit` thực ra được implement bằng `Exclude` bên trong: `Omit<T, K> = Pick<T, Exclude<keyof T, K>>`. Nó đầu tiên dùng `Exclude` để loại bỏ các key không muốn khỏi union `keyof T`, rồi dùng `Pick` để xây object type kết quả từ các key còn lại. Nên `Exclude` là primitive cấp thấp hơn và `Omit` là utility cấp cao hơn xây trên nó."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What's wrong with using `any` everywhere? Why does it matter?"**

**Model Answer:**
"`any` essentially tells TypeScript to stop type-checking that value. So you lose autocomplete, you lose refactoring safety, and you can silently introduce runtime errors that TypeScript would have caught. If you write `const data: any = fetchSomething()` and then call `data.user.name`, TypeScript won't warn you even if `user` doesn't exist — you only find out when it crashes in production.

The other issue is that `any` is contagious. Once a value is typed as `any`, everything you derive from it also becomes `any`. It spreads through the codebase and undermines the whole benefit of using TypeScript. In a real codebase, I'd much prefer `unknown` for truly unknown values, because it forces you to narrow before using them."

**Trả lời (Tiếng Việt):**
"`any` về cơ bản nói với TypeScript ngừng type-check giá trị đó. Nên bạn mất autocomplete, mất refactoring safety, và có thể im lặng đưa vào runtime error mà TypeScript lẽ ra đã bắt được. Nếu bạn viết `const data: any = fetchSomething()` rồi gọi `data.user.name`, TypeScript sẽ không cảnh báo ngay cả khi `user` không tồn tại — bạn chỉ phát hiện ra khi nó crash trên production.

Vấn đề khác là `any` có tính lây lan. Khi một giá trị được type là `any`, mọi thứ bạn derive từ nó cũng thành `any`. Nó lan rộng qua codebase và phá hoại toàn bộ lợi ích của việc dùng TypeScript. Trong codebase thực tế, tôi sẽ ưu tiên `unknown` hơn nhiều cho các giá trị thực sự chưa biết, vì nó ép bạn phải narrow trước khi dùng."

---

**Q2 (Junior/Mid): "When should you actually use `unknown` instead of `any`?"**

**Model Answer:**
"`unknown` is the right choice whenever you're receiving data from the outside world and you genuinely don't know its shape yet — things like API responses, `JSON.parse()` results, data from third-party webhooks, or the error parameter in a `catch` block.

With `unknown`, TypeScript forces you to write the narrowing logic before touching the value. That check might be `typeof`, `instanceof`, or a custom type guard. It feels like more work upfront, but it makes your code provably correct. The checks you write for `unknown` are exactly the checks you should be writing anyway to handle unexpected input gracefully.

`any` is really only appropriate when migrating a JavaScript codebase to TypeScript incrementally, or when you're working with a library that has no type definitions and you're in a hurry."

**Trả lời (Tiếng Việt):**
"`unknown` là lựa chọn đúng khi bạn nhận data từ thế giới bên ngoài và bạn thực sự chưa biết hình dạng của nó — như API response, kết quả `JSON.parse()`, data từ third-party webhook, hay tham số error trong `catch` block.

Với `unknown`, TypeScript ép bạn viết logic narrowing trước khi chạm vào giá trị. Kiểm tra đó có thể là `typeof`, `instanceof`, hay custom type guard. Cảm giác như tốn công hơn ban đầu, nhưng làm cho code của bạn chắc chắn đúng. Các kiểm tra bạn viết cho `unknown` chính xác là những kiểm tra bạn nên viết để xử lý input bất ngờ một cách có duyên.

`any` thực sự chỉ phù hợp khi migrate codebase JavaScript sang TypeScript tăng dần, hoặc khi bạn làm việc với thư viện không có type definition và bạn đang vội."

---

**Q3 (Mid): "What is the `never` type and what's the exhaustive check pattern?"**

**Model Answer:**
"`never` represents a value that can never exist. A function that always throws returns `never`, because it never actually produces a value. A function with an infinite loop also returns `never`.

The most useful practical application is the exhaustive check pattern in switch statements. Say you have a `Shape` type that's a union of `'circle' | 'square' | 'triangle'`. In the default branch of your switch, you do:

```ts
default:
  const _check: never = shape;
  throw new Error(`Unhandled: ${_check}`);
```

If every case is covered, TypeScript is happy because `shape` has been narrowed to `never`. But if you later add `'rectangle'` to the union without adding a case, TypeScript immediately errors on that assignment — `'rectangle'` is not assignable to `never`. It's a compile-time guarantee that your switch stays complete as the type evolves."

**Trả lời (Tiếng Việt):**
"`never` đại diện cho một giá trị không bao giờ có thể tồn tại. Function luôn throw trả về `never`, vì nó không thực sự tạo ra giá trị. Function với vòng lặp vô hạn cũng trả về `never`.

Ứng dụng thực tế hữu ích nhất là exhaustive check pattern trong switch statement. Giả sử bạn có type `Shape` là union của `'circle' | 'square' | 'triangle'`. Trong default branch của switch, bạn làm:

```ts
default:
  const _check: never = shape;
  throw new Error(`Unhandled: ${_check}`);
```

Nếu mọi case đã được xử lý, TypeScript hài lòng vì `shape` đã được narrow xuống `never`. Nhưng nếu bạn sau này thêm `'rectangle'` vào union mà không thêm case tương ứng, TypeScript ngay lập tức báo lỗi trên assignment đó — `'rectangle'` không thể assign cho `never`. Đây là đảm bảo compile-time rằng switch của bạn luôn đầy đủ khi type phát triển."

---

**Q4 (Senior): "What's the difference between `never` and `void` in TypeScript?"**

**Model Answer:**
"`void` means the function doesn't return a meaningful value — it might return `undefined` implicitly or with an explicit `return` statement, but the return value isn't useful to the caller. Event handlers and `useEffect` callbacks typically return `void`.

`never` means the function literally never returns — it throws an exception, enters an infinite loop, or calls `process.exit`. The control flow never reaches the point after the call.

The practical difference: if a function returns `void`, TypeScript lets you call it and ignore the result. If it returns `never`, TypeScript knows that any code after that call is unreachable and will narrow types accordingly — which is why `throw` helper functions typed as `never` are so useful for narrowing.

One subtle thing: `void` is not `undefined`. A function returning `void` can actually return `undefined`, but a variable typed as `void` can't be used as `undefined` in strict mode."

**Trả lời (Tiếng Việt):**
"`void` có nghĩa là function không trả về giá trị có ý nghĩa — nó có thể trả về `undefined` ngầm hoặc với `return` statement tường minh, nhưng giá trị trả về không hữu ích cho caller. Event handler và callback `useEffect` thường trả về `void`.

`never` có nghĩa là function theo nghĩa đen không bao giờ return — nó throw exception, vào vòng lặp vô hạn, hay gọi `process.exit`. Control flow không bao giờ đến điểm sau lần gọi đó.

Sự khác biệt thực tế: nếu function trả về `void`, TypeScript cho phép bạn gọi nó và bỏ qua kết quả. Nếu nó trả về `never`, TypeScript biết rằng bất kỳ code nào sau lần gọi đó là unreachable và sẽ narrow type tương ứng — đó là lý do tại sao helper function `throw` được type là `never` rất hữu ích để narrowing.

Một điều tinh tế: `void` không phải `undefined`. Function trả về `void` thực ra có thể trả về `undefined`, nhưng biến được type là `void` không thể dùng như `undefined` trong strict mode."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "What is a discriminated union? Why is it better than using a plain union of objects?"**

**Model Answer:**
"A discriminated union is when you have a union of types that all share one common literal property — the discriminant — which TypeScript uses to narrow which type you're dealing with. For example, `type PaymentMethod = CreditCard | BankTransfer | Crypto`, where each has a `type` property set to a unique string literal like `'credit_card'` or `'bank_transfer'`.

Without the discriminant, if you have two types with different optional properties, TypeScript can't tell them apart at runtime. With the discriminant, a single `switch (payment.type)` tells TypeScript exactly which variant you're in, and you get full autocomplete and type safety inside each branch with no casting needed.

It's essentially TypeScript's way of modeling algebraic data types — the same idea as tagged unions in functional languages."

**Trả lời (Tiếng Việt):**
"Discriminated union là khi bạn có union của các type đều chia sẻ một literal property chung — discriminant — mà TypeScript dùng để narrow type nào bạn đang xử lý. Ví dụ, `type PaymentMethod = CreditCard | BankTransfer | Crypto`, trong đó mỗi cái có property `type` được set thành string literal duy nhất như `'credit_card'` hay `'bank_transfer'`.

Không có discriminant, nếu bạn có hai type với các optional property khác nhau, TypeScript không thể phân biệt chúng ở runtime. Với discriminant, một `switch (payment.type)` đơn giản nói với TypeScript chính xác bạn đang ở variant nào, và bạn nhận được đầy đủ autocomplete và type safety trong mỗi branch mà không cần cast.

Đây về cơ bản là cách TypeScript mô hình algebraic data type — ý tưởng tương tự như tagged union trong các ngôn ngữ functional."

---

**Q2 (Mid): "Show me how you'd model loading states using a discriminated union."**

**Model Answer:**
"This is one of my favorite patterns. Instead of separate `isLoading`, `data`, and `error` booleans — which can get into impossible states like `isLoading: true` and `data: someValue` simultaneously — you use a single discriminated union:

```ts
type RemoteData<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

Then in your component, a switch on `status` gives TypeScript full knowledge of which properties exist in each branch. In the `'success'` branch, `data` is typed as `T` — no nullish checks needed. In the `'error'` branch, `error` is there. You literally can't access `data` in the error branch — TypeScript prevents it. It forces correct handling of every state."

**Trả lời (Tiếng Việt):**
"Đây là một trong những pattern tôi ưa thích nhất. Thay vì dùng các boolean riêng lẻ `isLoading`, `data`, và `error` — có thể rơi vào trạng thái không thể xảy ra như `isLoading: true` và `data: someValue` cùng lúc — bạn dùng một discriminated union duy nhất:

```ts
type RemoteData<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

Trong component của bạn, switch trên `status` cho TypeScript biết đầy đủ những property nào tồn tại trong mỗi branch. Trong branch `'success'`, `data` được type là `T` — không cần kiểm tra null. Trong branch `'error'`, `error` có mặt. Bạn hoàn toàn không thể truy cập `data` trong error branch — TypeScript ngăn điều đó. Nó bắt buộc xử lý đúng mọi state."

---

**Q3 (Mid): "What's the exhaustive check pattern with discriminated unions and why is it important?"**

**Model Answer:**
"When you switch over a discriminated union's discriminant, you should always have a `default` branch that assigns the remaining value to `never`:

```ts
default:
  const _exhaustive: never = payment;
  throw new Error(`Unhandled payment type`);
```

If you've handled every case in the union, TypeScript is satisfied because the value is genuinely `never` — it can't be anything. But the moment you add a new variant to the union and forget to add its case, TypeScript errors on that default assignment saying the new type isn't assignable to `never`.

This turns a potential runtime bug — silently falling through to a default case — into a compile-time error. For something like a payment system where missing a case would be a serious bug, this is invaluable."

**Trả lời (Tiếng Việt):**
"Khi bạn switch trên discriminant của discriminated union, bạn nên luôn có branch `default` assign giá trị còn lại cho `never`:

```ts
default:
  const _exhaustive: never = payment;
  throw new Error(`Unhandled payment type`);
```

Nếu bạn đã xử lý mọi case trong union, TypeScript hài lòng vì giá trị thực sự là `never` — nó không thể là gì cả. Nhưng ngay khi bạn thêm variant mới vào union và quên thêm case của nó, TypeScript ngay lập tức báo lỗi trên default assignment đó nói rằng type mới không thể assign cho `never`.

Điều này biến potential runtime bug — im lặng fall through đến default case — thành compile-time error. Với hệ thống payment nơi thiếu case là bug nghiêm trọng, điều này vô giá."

---

**Q4 (Senior): "How are discriminated unions related to type guards? When would you create a custom type guard for a discriminated union?"**

**Model Answer:**
"TypeScript narrows discriminated unions automatically when you check the discriminant — a simple `if (result.success)` or a switch is all you need. So in most cases, you don't need an explicit type guard.

But custom type guards become useful when you want to abstract the narrowing logic and reuse it — for instance, if you're filtering an array:

```ts
function isSuccess<T>(result: ApiResponse<T>): result is { success: true; data: T } {
  return result.success === true;
}

const successes = results.filter(isSuccess);
// TypeScript knows each item has .data typed correctly
```

Without the type guard, `Array.filter` doesn't propagate the narrowing and you'd get `ApiResponse<T>[]` back instead of the narrowed type. Another case is when the discriminant isn't a simple equality check — maybe you're checking a combination of fields — and you want to centralize that logic in one place with a clear name."

**Trả lời (Tiếng Việt):**
"TypeScript tự động narrow discriminated union khi bạn kiểm tra discriminant — `if (result.success)` đơn giản hay switch là đủ. Nên trong hầu hết trường hợp, bạn không cần type guard tường minh.

Nhưng custom type guard hữu ích khi bạn muốn abstract logic narrowing và tái sử dụng — ví dụ khi filter một mảng:

```ts
function isSuccess<T>(result: ApiResponse<T>): result is { success: true; data: T } {
  return result.success === true;
}

const successes = results.filter(isSuccess);
// TypeScript biết mỗi item có .data được type đúng
```

Không có type guard, `Array.filter` không propagate narrowing và bạn nhận `ApiResponse<T>[]` thay vì type đã được narrow. Trường hợp khác là khi discriminant không phải là equality check đơn giản — có thể bạn đang kiểm tra kết hợp nhiều field — và bạn muốn tập trung logic đó vào một chỗ với tên rõ ràng."

---

**Q5 (Senior): "What happens when a discriminated union grows to many variants? How do you keep it maintainable?"**

**Model Answer:**
"The main risk is that switch statements get long and it becomes tempting to add a catch-all default that hides missed cases. The fix is always having the `never` exhaustiveness check — that way adding a new variant is a compiler error, not something you might forget.

For large unions, I tend to separate the type definitions into their own file and export a helper map or lookup object for the shared behaviors. For example, a config object keyed by the discriminant that contains labels, icons, and handler functions — then the switch statement becomes a single lookup: `const config = PAYMENT_CONFIG[payment.type]`. You still get exhaustiveness checking on the type definitions themselves, but the logic is centralized.

Another pattern I use is 'smart constructors' — functions like `makeSuccess(data)` and `makeError(error)` that return the correctly tagged variant. This prevents typos in the discriminant string and makes creating variants type-safe."

**Trả lời (Tiếng Việt):**
"Rủi ro chính là switch statement trở nên dài và có xu hướng thêm catch-all default ẩn đi các case bị bỏ qua. Cách sửa là luôn có `never` exhaustiveness check — cách đó thêm variant mới là compiler error, không phải thứ bạn có thể quên.

Với union lớn, tôi có xu hướng tách định nghĩa type vào file riêng và export helper map hay lookup object cho các behavior chia sẻ. Ví dụ, một config object được key bởi discriminant chứa label, icon, và handler function — thì switch statement trở thành một lookup duy nhất: `const config = PAYMENT_CONFIG[payment.type]`. Bạn vẫn nhận exhaustiveness checking trên định nghĩa type, nhưng logic được tập trung.

Một pattern khác tôi dùng là 'smart constructor' — function như `makeSuccess(data)` và `makeError(error)` trả về variant được tag đúng. Điều này ngăn typo trong discriminant string và làm việc tạo variant type-safe."

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
