# UrBox Interview Prep — Fullstack Engineer
> Tổng hợp từ repo + bổ sung NestJS/Database cho JD cụ thể

---

## THỨ TỰ ÔN TẬP (theo độ quan trọng với JD)

1. NestJS ← không có trong repo, JD essential
2. Database (PostgreSQL / MongoDB / Redis) ← không có trong repo
3. JavaScript
4. TypeScript
5. React
6. Next.js
7. State Management
8. Web Security & API Design

---

# 1. NESTJS

---

**Q: Request đi qua những bước nào trong NestJS?**

```
Request
  → Middleware        (không biết handler nào, không có ExecutionContext)
  → Guard             (có ExecutionContext — allow/deny)
  → Interceptor PRE   (trước handler)
  → Pipe              (validate & transform input)
  → Handler           (controller method)
  → Interceptor POST  (sau handler — transform response)
  → Exception Filter  (nếu throw)
```

---

**Q: Guard vs Middleware khác nhau thế nào?**

> **Middleware**: không có ExecutionContext — không biết handler nào đang được gọi. Dùng cho logging, parse cookie, attach request ID.
>
> **Guard**: có ExecutionContext — biết rõ controller, method. Return `true/false`. Dùng cho **authentication, authorization, role check**.

---

**Q: Pipe dùng để làm gì?**

> Validate và transform input **trước khi vào handler**.

```ts
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  // id đã convert từ string "1" → number 1
}

// ValidationPipe + class-validator
class CreateUserDto {
  @IsEmail()    email: string;
  @MinLength(6) password: string;
}
```

---

**Q: Interceptor dùng để làm gì? Cho ví dụ.**

> Wrap response, logging, caching — chạy **cả trước lẫn sau** handler.

```ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    return next.handle().pipe(
      map(data => ({ success: true, data }))
    );
  }
}
// Mọi response đều có format: { success: true, data: {...} }
```

---

**Q: Dependency Injection trong NestJS hoạt động thế nào?**

> NestJS có IoC container quản lý instances. Khai báo `@Injectable()` + register trong module → container tự tạo và inject:

```ts
@Injectable()
export class UserService { ... }

@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}
  // NestJS inject — không cần new UserService()
}
```

> **Benefit**: dễ test (mock), share singleton, loose coupling.

---

**Q: Handle lỗi trong NestJS như thế nào?**

```ts
// Throw built-in exceptions
throw new NotFoundException('User not found');
throw new BadRequestException('Invalid input');

// Custom ExceptionFilter để format tất cả lỗi
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse();
    res.status(exception.getStatus()).json({
      success: false,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

---

**Q: FeatherJS bạn có biết không?**

> *(Gap trong CV — trả lời honest)*
> "Tôi chưa làm việc trực tiếp với FeatherJS, nhưng tôi biết đây là Node.js framework nhẹ hơn NestJS, built-in real-time qua Socket.io, dùng hooks pattern thay vì middleware. Với nền NestJS và Express tôi có, tôi tự tin pick up nhanh vì concepts service layer và middleware rất tương đồng."

---

**Q: Rate limiting implement thế nào trong NestJS?**

```ts
// Dùng @nestjs/throttler
ThrottlerModule.forRoot({ ttl: 60, limit: 10 })
// Mỗi IP: tối đa 10 requests / 60 giây

// Hoặc Redis-based cho multi-instance
```

---

# 2. DATABASE

---

**Q: PostgreSQL vs MongoDB — khi nào dùng cái nào?**

> **PostgreSQL**: structured data, relations phức tạp, cần ACID transactions. Ví dụ: **point balance, transactions** — không thể sai 1 đồng.
>
> **MongoDB**: flexible schema, nested documents, scale horizontal dễ. Ví dụ: **activity logs, audit trails, notifications**.
>
> *Với loyalty system như UrBox: core data → PostgreSQL, event logs → MongoDB.*

---

**Q: MySQL bạn chưa dùng — nếu được hỏi thì sao?**

> "Tôi có solid experience với PostgreSQL trong production. MySQL và PostgreSQL đều là relational database, SQL syntax rất tương tự — khác biệt chính ở storage engine, JSON support và một số features nâng cao. Tôi tự tin adapt trong 1-2 ngày."

---

**Q: Index là gì? Khi nào nên và không nên dùng?**

> Index tạo B-tree structure riêng — query O(log n) thay vì full scan O(n).
>
> **Nên dùng**: columns thường xuất hiện trong `WHERE`, `JOIN`, `ORDER BY`.
>
> **Không nên**: over-index vì mỗi `INSERT/UPDATE/DELETE` phải update tất cả indexes. Table nhỏ thì full scan còn nhanh hơn.

```sql
-- Composite index cho query thường gặp
CREATE INDEX idx_transactions_user_status
ON point_transactions(user_id, status);

-- Partial index — chỉ index active records
CREATE INDEX idx_active_users
ON users(email) WHERE is_active = true;
```

---

**Q: Transaction là gì? ACID?**

> **A**tomicity: tất cả hoặc rollback hết
> **C**onsistency: data luôn valid
> **I**solation: transactions không ảnh hưởng nhau
> **D**urability: sau commit, data tồn tại dù crash

---

**Q: Bạn xử lý concurrent transactions thế nào? (câu hỏi quan trọng nhất với loyalty system)**

> *"Tôi đã gặp bài toán này thực tế tại Herond Points: 2 requests cùng lúc cộng điểm cho cùng 1 user → cả 2 đọc balance cũ → cả 2 write → mất điểm."*

**Cách 1 — Redis distributed locking:**
```
SET user:{id}:lock 1 NX PX 5000
→ NX: chỉ set nếu chưa tồn tại (atomic)
→ PX 5000: tự expire sau 5s phòng server crash
→ Chỉ 1 request acquire lock tại 1 thời điểm
→ Xử lý xong → DEL key
```

**Cách 2 — PostgreSQL SELECT FOR UPDATE:**
```sql
BEGIN;
SELECT balance FROM point_accounts
WHERE user_id = $1 FOR UPDATE; -- lock row
-- validate + update
UPDATE point_accounts
SET balance = balance - $2 WHERE user_id = $1;
COMMIT;
```

---

**Q: N+1 query problem là gì? Cách fix?**

```ts
// BAD — N+1
const users = await prisma.user.findMany();
for (const user of users) {
  user.orders = await prisma.order.findMany({ where: { userId: user.id } });
}
// 1 query lấy users + N queries lấy orders

// GOOD — eager loading
const users = await prisma.user.findMany({
  include: { orders: true } // 1 JOIN query
});
```

---

**Q: Redis dùng cho những use case nào?**

> 1. **Distributed locking** — tránh race condition (đã làm tại Herond)
> 2. **Caching** — cache query results, giảm DB load
> 3. **Session storage** — fast read/write
> 4. **Rate limiting** — counter với TTL
> 5. **Pub/Sub** — real-time notifications

---

# 3. JAVASCRIPT

*(Từ interview/javascript.md)*

---

**Q: Event Loop hoạt động thế nào?**

```
Call Stack (sync code)
  → khi rỗng: drain Microtask Queue (Promise.then, async/await)
  → khi rỗng: lấy 1 Macrotask (setTimeout, I/O)
  → lặp lại
```

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1 → 4 → 3 → 2
// Promise (microtask) chạy trước setTimeout (macrotask) dù cùng 0ms
```

---

**Q: Promise.all / allSettled / race / any?**

| Method | Behavior | Dùng khi |
|--------|----------|----------|
| `Promise.all` | Reject ngay nếu 1 fail | Cần tất cả thành công |
| `Promise.allSettled` | Chờ hết, trả kết quả mọi promise | Muốn biết cái nào fail |
| `Promise.race` | Resolve/reject theo cái xong trước | Timeout pattern |
| `Promise.any` | Resolve ngay khi 1 thành công | Thử nhiều nguồn, lấy cái nhanh nhất |

---

**Q: Closure là gì?**

> Function "nhớ" biến từ outer scope dù outer function đã return.

```js
function makeCounter() {
  let count = 0;
  return () => ++count; // closure giữ reference đến count
}
const counter = makeCounter();
counter(); // 1
counter(); // 2
```

---

**Q: var / let / const?**

| | var | let | const |
|---|---|---|---|
| Scope | Function | Block | Block |
| Hoisting | Có (undefined) | Có (TDZ) | Có (TDZ) |
| Reassign | Được | Được | Không |

**TDZ (Temporal Dead Zone)**: khoảng từ đầu block đến khi `let/const` được khai báo — truy cập trong khoảng này → ReferenceError.

---

# 4. TYPESCRIPT

*(Từ interview/typescript.md)*

---

**Q: interface vs type?**

```ts
// interface — có thể extend và declaration merge
interface User { name: string }
interface User { age: number } // merge thành { name, age }

// type — linh hoạt hơn, union/intersection
type Status = 'active' | 'inactive'
type ID = string | number
```

> **Rule of thumb**: object shape → `interface`, complex types → `type`.

---

**Q: Generic là gì?**

```ts
function getFirst<T>(arr: T[]): T {
  return arr[0];
}
getFirst([1, 2, 3])   // T inferred = number
getFirst(['a', 'b'])  // T inferred = string
```

---

**Q: Utility types hay dùng?**

```ts
Partial<User>    // tất cả fields optional
Required<User>   // tất cả fields required
Pick<User, 'id' | 'name'>   // chỉ lấy 1 số fields
Omit<User, 'password'>      // bỏ fields không muốn
Readonly<User>   // không mutate được
Record<string, number>       // { [key: string]: number }
```

---

# 5. REACT

*(Từ interview/react.md)*

---

**Q: React re-render khi nào? Tránh re-render thế nào?**

> Re-render khi: state thay đổi, props thay đổi, parent re-render, context thay đổi.

```ts
// React.memo — skip re-render nếu props không đổi
const Child = React.memo(({ name }) => <div>{name}</div>);

// useMemo — cache kết quả tính toán nặng
const sorted = useMemo(() => items.sort(fn), [items]);

// useCallback — cache function reference
const handleClick = useCallback(() => doSomething(id), [id]);
```

---

**Q: useEffect cleanup dùng khi nào?**

```ts
useEffect(() => {
  const sub = subscribe(userId);
  return () => sub.unsubscribe(); // cleanup khi unmount
}, [userId]);
```

> Bắt buộc cleanup: WebSocket, subscriptions, event listeners, timers.

---

**Q: key prop trong list tại sao quan trọng?**

> React dùng `key` để xác định element nào thay đổi/thêm/xóa trong reconciliation. Dùng stable unique ID, không dùng index khi list có thể reorder.

---

# 6. NEXT.JS

*(Từ interview/nextjs.md)*

---

**Q: SSR / SSG / ISR / CSR khác nhau thế nào?**

| | Render khi nào | Dùng cho |
|---|---|---|
| **CSR** | Client, sau JS load | Dashboard cần auth |
| **SSG** | Build time | Blog, landing page |
| **SSR** | Mỗi request | Data thay đổi + cần SEO |
| **ISR** | Build time + revalidate sau N giây | E-commerce |

---

**Q: Pages Router vs App Router?**

> **Pages Router** (cũ): `getServerSideProps`, `getStaticProps`, `pages/` directory.
> **App Router** (mới): Server Components mặc định, `app/` directory, `fetch` với cache options built-in.

---

**Q: next/image tối ưu gì?**

> Tự động resize, lazy load, WebP conversion, prevent CLS với width/height. Dùng `priority` cho LCP image.

---

# 7. STATE MANAGEMENT

*(Từ interview/state-management.md)*

---

**Q: Khi nào dùng Redux, khi nào Context?**

> **Context**: global state đơn giản, ít thay đổi (theme, auth user, language).
>
> **Redux/RTK Query**: server state cần caching/invalidation, nhiều component share và mutate, cần devtools.

---

**Q: RTK Query là gì?**

> Data fetching + caching layer tích hợp với Redux. Tự động handle loading/error states, cache invalidation, background refetch.

```ts
const { data, isLoading, error } = useGetUserQuery(userId);
// Tự cache, không fetch lại nếu data còn fresh
```

---

# 8. WEB SECURITY & API DESIGN

*(Từ interview/web-security.md)*

---

**Q: JWT hoạt động thế nào? Nhược điểm?**

> Token: `header.payload.signature` — server ký bằng secret, client gửi trong `Authorization: Bearer`.
>
> **Nhược điểm**: không thể invalidate trước expiry.
> **Fix**: access token ngắn (15 phút) + refresh token trong DB (có thể revoke).

---

**Q: SQL Injection — cách prevent?**

```ts
// NGUY HIỂM
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// AN TOÀN — parameterized query
prisma.user.findUnique({ where: { email } }); // Prisma tự escape
```

---

**Q: XSS là gì? React bảo vệ thế nào?**

> Attacker inject script vào page chạy trong browser nạn nhân. React auto-escape JSX expressions → an toàn mặc định. Chỉ nguy hiểm khi dùng `dangerouslySetInnerHTML` — phải sanitize bằng DOMPurify.

---

**Q: CORS là gì?**

> Browser policy block request từ origin khác nếu server không có `Access-Control-Allow-Origin` header.

```ts
app.enableCors({ origin: ['https://app.urbox.vn'], credentials: true });
```

---

**Q: RESTful API design principles?**

> - Resource-based URLs: `/users/:id` không phải `/getUser`
> - Đúng HTTP methods: GET (read), POST (create), PUT/PATCH (update), DELETE
> - Đúng status codes: 200/201/400/401/403/404/500
> - Stateless: mỗi request chứa đủ thông tin, không phụ thuộc session server

---

**Q: Pagination strategy nào tốt hơn?**

> **Offset**: `LIMIT 10 OFFSET 200` — simple nhưng slow khi offset lớn.
> **Cursor**: `WHERE id > last_id LIMIT 10` — nhanh hơn, consistent khi data thay đổi. Dùng cho large dataset.

---

# CÁC CÂU KHÓ ĐỞ — LIÊN QUAN CV CỤ THỂ

---

**"MongoDB bạn dùng ở dự án nào?"**
> Trả lời thật: nếu chỉ có trong skills mà không có project thực tế thì nói: *"Tôi có làm việc với MongoDB ở level cơ bản. Production experience chủ yếu là PostgreSQL — tôi comfortable với relational databases và đang expand thêm NoSQL."*

**"Trước Herond bạn toàn làm Frontend — backend có đủ strong không?"**
> *"4 năm đầu tôi focus frontend, nhưng tại Herond tôi đang thực sự làm cả hai. Tôi build NestJS modules từ schema design đến API đến deployment. Tôi không claim senior backend, nhưng tôi có solid foundation và đang grow nhanh."*

**"FeatherJS bạn biết không?"**
> *"Chưa dùng trong production, nhưng tôi biết concept — real-time built-in, hooks pattern tương tự NestJS interceptors. Với nền NestJS tôi có, pick up nhanh được."*

**"Payment integration bạn có chưa?"**
> *"Chưa làm payment gateway cụ thể. Nhưng tôi đã integrate nhiều third-party services — OAuth2, Azure services. Approach tương tự: đọc docs kỹ, handle errors và edge cases, retry mechanism."*

---

*Repo source: interview/javascript.md, nextjs.md, react.md, typescript.md, state-management.md, web-security.md*
