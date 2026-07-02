# Testing — Interview Questions (Junior → Senior)

---

## 1. Testing Pyramid

**Level:** [Junior]

**EN:** Explain the testing pyramid. What are unit, integration, and end-to-end tests? When should you use each?

**VI:** Giải thích testing pyramid. Unit, integration, và e2e tests là gì? Khi nào dùng từng loại?

**Trả lời:**

```
         /\
        /e2e\        ← Ít nhất, chậm nhất, tốn kém nhất
       /------\
      /  integ  \    ← Trung bình
     /------------\
    /     unit      \ ← Nhiều nhất, nhanh nhất, rẻ nhất
   /------------------\
```

| Loại | Mục tiêu | Tốc độ | Ví dụ |
|------|----------|--------|-------|
| **Unit** | Một function/component | Milliseconds | Test `formatDate()`, test button render |
| **Integration** | Nhiều units phối hợp | Seconds | Test form submit gọi API, hook với state |
| **E2E** | Flow người dùng thực tế | Minutes | Login → thêm item → checkout |

```
Khi nào dùng:
- Unit: Logic thuần (utils, helpers, pure components), business logic phức tạp
- Integration: Tương tác giữa component và context/store/API, custom hooks
- E2E: Critical user journeys (checkout, login, signup), smoke tests sau deploy

Tỷ lệ hợp lý: 70% unit / 20% integration / 10% e2e
Đừng over-test trivial code (getters/setters, simple UI), focus vào business logic
```

---

## 2. Jest Basics

**Level:** [Junior]

**EN:** Explain Jest's structure: `describe`, `it/test`, `expect` matchers, and lifecycle hooks (`beforeEach`, `afterEach`, `beforeAll`, `afterAll`).

**VI:** Giải thích cấu trúc Jest: `describe`, `it/test`, `expect` matchers, và lifecycle hooks.

**Trả lời:**

```ts
// utils/math.ts
export const add = (a: number, b: number) => a + b;
export const divide = (a: number, b: number) => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};
export const formatCurrency = (amount: number, currency = 'USD') =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount);

// utils/math.test.ts
import { add, divide, formatCurrency } from './math';

// describe — nhóm các tests liên quan
describe('Math utilities', () => {

  // beforeAll — chạy 1 lần trước tất cả tests trong block này
  beforeAll(() => {
    console.log('Starting math tests');
  });

  // afterAll — chạy 1 lần sau tất cả tests
  afterAll(() => {
    console.log('Math tests complete');
  });

  // beforeEach — chạy trước MỖI test
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // afterEach — chạy sau mỗi test (cleanup)
  afterEach(() => {
    // cleanup side effects
  });

  // ==============================
  // MATCHERS
  // ==============================

  describe('add()', () => {
    it('adds two positive numbers', () => {
      expect(add(2, 3)).toBe(5);          // strict equality (===)
    });

    test('returns negative when result is negative', () => {
      // test và it là alias, dùng tùy preference
      expect(add(-1, -2)).toBeLessThan(0);
    });
  });

  describe('divide()', () => {
    it('divides correctly', () => {
      expect(divide(10, 2)).toBe(5);
      expect(divide(7, 2)).toBeCloseTo(3.5, 2); // floating point
    });

    it('throws on division by zero', () => {
      expect(() => divide(10, 0)).toThrow('Division by zero');
      expect(() => divide(10, 0)).toThrow(Error);
    });
  });

  describe('formatCurrency()', () => {
    it('formats USD by default', () => {
      expect(formatCurrency(1000)).toBe('$1,000.00');
    });

    it('formats other currencies', () => {
      expect(formatCurrency(50, 'EUR')).toContain('50');
    });
  });
});

// ==============================
// Common matchers reference
// ==============================
describe('Matcher examples', () => {
  it('equality', () => {
    expect(1 + 1).toBe(2);                      // ===
    expect({ a: 1 }).toEqual({ a: 1 });          // deep equal
    expect({ a: 1, b: 2 }).toMatchObject({ a: 1 }); // partial match
  });

  it('truthiness', () => {
    expect(true).toBeTruthy();
    expect(null).toBeFalsy();
    expect(null).toBeNull();
    expect(undefined).toBeUndefined();
    expect('hello').toBeDefined();
  });

  it('numbers', () => {
    expect(5).toBeGreaterThan(4);
    expect(5).toBeGreaterThanOrEqual(5);
    expect(3.14159).toBeCloseTo(3.14, 2);
  });

  it('strings', () => {
    expect('hello world').toContain('world');
    expect('hello').toMatch(/^hell/);
    expect('hello').toHaveLength(5);
  });

  it('arrays', () => {
    expect([1, 2, 3]).toContain(2);
    expect([1, 2, 3]).toHaveLength(3);
    expect([1, 2, 3]).toEqual(expect.arrayContaining([3, 1]));
  });

  it('negation', () => {
    expect(1 + 1).not.toBe(3);
    expect([]).not.toContain(99);
  });
});
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What's the difference between `toBe` and `toEqual` in Jest?"**

**Model Answer:**
"`toBe` uses strict equality — the same as `===`. It works for primitives: numbers, strings, booleans. `toEqual` does a deep recursive equality check, so it works for objects and arrays. If I write `expect({ a: 1 }).toBe({ a: 1 })` that fails, because those are two different object references even though the contents are the same. `toEqual` would pass. A related one is `toMatchObject`, which checks that the expected properties exist in the actual object, but the actual object can have extra properties. I use that when I only care about certain fields and don't want to assert the full shape."

**Trả lời (Tiếng Việt):**
"`toBe` dùng strict equality — giống hệt `===`. Nó hoạt động với primitive: số, string, boolean. `toEqual` thực hiện deep recursive equality check, nên dùng được cho object và array. Nếu mình viết `expect({ a: 1 }).toBe({ a: 1 })` thì sẽ fail, vì đó là hai object reference khác nhau dù nội dung giống nhau. `toEqual` thì sẽ pass. Có thêm một cái liên quan là `toMatchObject` — nó kiểm tra các property mong đợi có tồn tại trong actual object không, nhưng actual object có thể có thêm property khác. Mình dùng cái này khi chỉ quan tâm đến một số field nhất định và không muốn assert toàn bộ shape."

---

**Q2 (Junior/Mid): "What should you actually test in a frontend application? What shouldn't you bother testing?"**

**Model Answer:**
"I focus tests on behavior that can break and would be hard to catch manually: business logic functions like price calculations or input validation, component behavior like what renders when a prop changes or what happens when a button is clicked, error handling paths, and accessibility attributes on interactive elements. What I tend not to test: straightforward styling, which would make tests brittle to cosmetic changes; simple getters and setters with no logic; third-party library internals; or components where the test would be a line-for-line mirror of the implementation. The guiding question is: if this breaks, would a test catch it before it reaches users? If yes, write the test. If the test would mostly just confirm the code compiles, skip it."

**Trả lời (Tiếng Việt):**
"Mình tập trung test những behavior có thể bị lỗi và khó phát hiện thủ công: các hàm business logic như tính giá hay validate input, behavior của component như render gì khi prop thay đổi hoặc khi button được click, các đường xử lý lỗi, và accessibility attribute trên interactive element. Những thứ mình thường không test: styling thuần túy vì sẽ làm test dễ vỡ với các thay đổi cosmetic; getter/setter đơn giản không có logic; nội bộ thư viện third-party; hay component mà test chỉ là copy paste lại implementation. Câu hỏi dẫn đường là: nếu cái này bị lỗi, liệu test có bắt được trước khi đến tay user không? Nếu có, viết test. Nếu test chủ yếu chỉ xác nhận code compile được, bỏ qua."

---

**Q3 (Mid): "What are lifecycle hooks in Jest and when do you use each one?"**

**Model Answer:**
"There are four: `beforeAll` runs once before any test in the describe block — useful for expensive setup like a database connection or starting a mock server. `afterAll` runs once at the end — useful for cleanup like closing that connection. `beforeEach` runs before every individual test — the most common one, I use it to reset mocks with `jest.clearAllMocks()` and reset any state so tests are isolated from each other. `afterEach` runs after every test — useful for cleanup that's specific to a test, like clearing localStorage or removing DOM elements. The key principle is test isolation: each test should be able to run alone or in any order and produce the same result. `beforeEach` clearing shared state is usually what enforces that."

**Trả lời (Tiếng Việt):**
"Có bốn cái: `beforeAll` chạy một lần trước tất cả test trong describe block — dùng cho setup nặng như kết nối database hay khởi động mock server. `afterAll` chạy một lần ở cuối — dùng để cleanup như đóng kết nối đó. `beforeEach` chạy trước mỗi test — cái này mình dùng nhiều nhất, để reset mock bằng `jest.clearAllMocks()` và reset state để các test độc lập với nhau. `afterEach` chạy sau mỗi test — dùng cho cleanup cụ thể của từng test, như clear localStorage hay remove DOM element. Nguyên tắc then chốt là test isolation: mỗi test phải có thể chạy một mình hoặc theo bất kỳ thứ tự nào mà vẫn cho kết quả giống nhau. `beforeEach` clearing shared state thường là thứ đảm bảo điều đó."

---

**Q4 (Mid): "How do you test that a function throws an error?"**

**Model Answer:**
"You wrap the call in a function and pass that to `expect`, then chain `.toThrow`. So it's `expect(() => divide(10, 0)).toThrow('Division by zero')`. The reason you need the wrapper function is that if you just wrote `expect(divide(10, 0)).toThrow()`, the error would throw before Jest even evaluates the `expect`, so Jest would see an unhandled exception and fail the test for the wrong reason. You can also pass an Error class instead of a string — `.toThrow(Error)` — or a regex if you want to match part of the message. For async functions that reject, the pattern is `await expect(asyncFn()).rejects.toThrow('message')`."

**Trả lời (Tiếng Việt):**
"Mình bọc lời gọi trong một function rồi truyền vào `expect`, sau đó chain `.toThrow`. Tức là `expect(() => divide(10, 0)).toThrow('Division by zero')`. Lý do cần wrapper function là nếu viết thẳng `expect(divide(10, 0)).toThrow()`, error sẽ throw trước khi Jest kịp evaluate `expect`, nên Jest thấy unhandled exception và fail test vì lý do sai. Mình cũng có thể truyền Error class thay vì string — `.toThrow(Error)` — hoặc regex nếu muốn match một phần message. Với async function reject thì pattern là `await expect(asyncFn()).rejects.toThrow('message')`."

---

---

## 3. Mocking in Jest

**Level:** [Mid]

**EN:** Explain `jest.fn()`, `jest.mock()`, `jest.spyOn()`, module mocking, and fake timers. When do you use each?

**VI:** Giải thích `jest.fn()`, `jest.mock()`, `jest.spyOn()`, module mocking và fake timers. Khi nào dùng từng loại?

**Trả lời:**

```ts
// ==============================
// 1. jest.fn() — tạo mock function
// ==============================
describe('jest.fn()', () => {
  it('tracks calls and returns', () => {
    const mockFn = jest.fn();
    mockFn('hello');
    mockFn('world');

    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('hello');
    expect(mockFn).toHaveBeenLastCalledWith('world');
    expect(mockFn).toHaveBeenNthCalledWith(1, 'hello');
  });

  it('can return values', () => {
    const mockFetch = jest.fn()
      .mockReturnValueOnce({ data: 'first' })
      .mockReturnValueOnce({ data: 'second' })
      .mockReturnValue({ data: 'default' }); // fallback

    expect(mockFetch()).toEqual({ data: 'first' });
    expect(mockFetch()).toEqual({ data: 'second' });
    expect(mockFetch()).toEqual({ data: 'default' });
  });

  it('can return resolved promises', async () => {
    const mockApi = jest.fn().mockResolvedValue({ users: [] });
    const result = await mockApi();
    expect(result).toEqual({ users: [] });
  });

  it('can simulate rejections', async () => {
    const mockApi = jest.fn().mockRejectedValue(new Error('Network error'));
    await expect(mockApi()).rejects.toThrow('Network error');
  });
});

// ==============================
// 2. jest.mock() — mock toàn bộ module
// ==============================

// services/api.ts
export const fetchUser = async (id: string) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
};

// components/UserProfile.test.tsx
jest.mock('../services/api'); // mock toàn bộ module — phải ở top level

import { fetchUser } from '../services/api';

const mockFetchUser = fetchUser as jest.MockedFunction<typeof fetchUser>;

describe('UserProfile', () => {
  beforeEach(() => {
    mockFetchUser.mockResolvedValue({
      id: '1',
      name: 'Alice',
      email: 'alice@example.com',
    });
  });

  it('displays user data', async () => {
    // ...test logic
  });
});

// Mock với factory function (để control implementation)
jest.mock('react-router-dom', () => ({
  ...jest.requireActual('react-router-dom'), // giữ implementation thực
  useNavigate: () => jest.fn(),
  useParams: () => ({ id: '123' }),
}));

// ==============================
// 3. jest.spyOn() — spy on existing function
// ==============================
describe('jest.spyOn()', () => {
  it('spies on method without changing it', () => {
    const spy = jest.spyOn(console, 'log');

    console.log('test');

    expect(spy).toHaveBeenCalledWith('test');
    spy.mockRestore(); // khôi phục implementation gốc
  });

  it('spies and replaces implementation', () => {
    const spy = jest.spyOn(Math, 'random').mockReturnValue(0.5);

    expect(Math.random()).toBe(0.5);

    spy.mockRestore();
  });

  it('spies on object method', () => {
    const userService = {
      getUser: async (id: string) => ({ id, name: 'Real User' }),
    };

    const spy = jest.spyOn(userService, 'getUser').mockResolvedValue({
      id: '1',
      name: 'Mock User',
    });

    // test...
    expect(spy).toHaveBeenCalled();
  });
});

// ==============================
// 4. Fake Timers
// ==============================
describe('Fake timers', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('tests setTimeout', () => {
    const callback = jest.fn();
    setTimeout(callback, 1000);

    expect(callback).not.toHaveBeenCalled();

    jest.advanceTimersByTime(1000);

    expect(callback).toHaveBeenCalledTimes(1);
  });

  it('tests debounced function', () => {
    const handler = jest.fn();
    const debouncedHandler = debounce(handler, 300);

    debouncedHandler();
    debouncedHandler();
    debouncedHandler();

    jest.advanceTimersByTime(300);

    expect(handler).toHaveBeenCalledTimes(1); // debounce hoạt động
  });

  it('runs all timers immediately', () => {
    const callback = jest.fn();
    setTimeout(callback, 99999);

    jest.runAllTimers();

    expect(callback).toHaveBeenCalled();
  });
});
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "What's the difference between `jest.fn()`, `jest.mock()`, and `jest.spyOn()`?"**

**Model Answer:**
"`jest.fn()` creates a standalone mock function from scratch — it's just a callable function that tracks calls. You'd use it when you need to pass a callback as a prop, like `render(<Button onClick={jest.fn()} />)`. `jest.mock('moduleName')` replaces an entire module — all its exports become mock functions. You use it when you want to control what an imported module returns, like mocking your API service so it doesn't make real HTTP requests. `jest.spyOn(object, 'methodName')` is different — it wraps a real method on an existing object, so you can observe calls without necessarily changing the behavior. It's useful when you want to assert that a method was called, but still let the original implementation run. And unlike `jest.mock`, spy mocks can be restored with `.mockRestore()` after the test."

**Trả lời (Tiếng Việt):**
"`jest.fn()` tạo một mock function độc lập từ đầu — chỉ là một callable function theo dõi các lần được gọi. Dùng khi cần truyền callback làm prop, ví dụ `render(<Button onClick={jest.fn()} />)`. `jest.mock('moduleName')` thay thế toàn bộ một module — tất cả export của nó trở thành mock function. Dùng khi muốn kiểm soát những gì module import trả về, ví dụ mock API service để nó không gửi HTTP request thật. `jest.spyOn(object, 'methodName')` thì khác — nó bọc lấy một method thực trên object có sẵn, nên mình có thể quan sát lời gọi mà không nhất thiết phải thay đổi behavior. Dùng khi muốn assert rằng một method đã được gọi nhưng vẫn để implementation thật chạy. Và khác với `jest.mock`, spy mock có thể restore về ban đầu bằng `.mockRestore()` sau khi test."

---

**Q2 (Mid): "You have a component that calls `Date.now()` or `Math.random()` — how do you test deterministic output?"**

**Model Answer:**
"For `Math.random` I'd use `jest.spyOn(Math, 'random').mockReturnValue(0.5)` to control what it returns. For `Date.now` I'd do `jest.spyOn(Date, 'now').mockReturnValue(1700000000000)` or use `jest.useFakeTimers()` which gives you control over the entire time system. Both approaches should call `.mockRestore()` in `afterEach` so they don't bleed into other tests. The principle is that any side input to a function — current time, random numbers, environment variables — needs to be injectable or mockable, otherwise your tests are non-deterministic and will flake."

**Trả lời (Tiếng Việt):**
"Với `Math.random` mình dùng `jest.spyOn(Math, 'random').mockReturnValue(0.5)` để kiểm soát giá trị nó trả về. Với `Date.now` thì `jest.spyOn(Date, 'now').mockReturnValue(1700000000000)` hoặc dùng `jest.useFakeTimers()` để có control toàn bộ hệ thống thời gian. Cả hai cách đều nên gọi `.mockRestore()` trong `afterEach` để không rò rỉ sang các test khác. Nguyên tắc là bất kỳ input phụ nào vào function — thời gian hiện tại, số ngẫu nhiên, environment variable — đều cần có thể inject hoặc mock được, nếu không test sẽ non-deterministic và flaky."

---

**Q3 (Mid): "How do fake timers work in Jest, and what's a common use case?"**

**Model Answer:**
"When you call `jest.useFakeTimers()`, Jest replaces the global timer functions — `setTimeout`, `setInterval`, `Date` — with mock versions it controls. Your code still calls `setTimeout(fn, 1000)` normally, but that timer doesn't actually tick on real time. You then call `jest.advanceTimersByTime(1000)` in your test to simulate time passing, which fires the callback synchronously. A classic use case is testing debounce or throttle logic — you don't want to actually wait 300 milliseconds in your test suite. Another use case is testing retry logic with exponential backoff. Always pair `useFakeTimers` in `beforeEach` with `useRealTimers` in `afterEach` to avoid leaking fake timers into other tests."

**Trả lời (Tiếng Việt):**
"Khi gọi `jest.useFakeTimers()`, Jest thay thế các global timer function — `setTimeout`, `setInterval`, `Date` — bằng các version mock mà nó kiểm soát. Code của mình vẫn gọi `setTimeout(fn, 1000)` bình thường, nhưng timer đó không thực sự chạy theo thời gian thật. Rồi mình gọi `jest.advanceTimersByTime(1000)` trong test để giả lập thời gian trôi qua, kích hoạt callback đồng bộ. Use case cổ điển là test debounce hoặc throttle logic — mình không muốn thực sự chờ 300 milliseconds trong test suite. Trường hợp khác là test retry logic với exponential backoff. Luôn ghép `useFakeTimers` trong `beforeEach` với `useRealTimers` trong `afterEach` để tránh fake timer rò rỉ sang test khác."

---

**Q4 (Senior): "Your test suite is slow because many tests hit real `localStorage`. How do you approach fixing that?"**

**Model Answer:**
"I'd mock `localStorage` at the module level using `Object.defineProperty` on `window.localStorage` with a hand-rolled mock object backed by a plain JavaScript object. The mock implements `getItem`, `setItem`, `removeItem`, and `clear` using `jest.fn()` so I can assert on what was called and control return values. In `beforeEach` I call `.clear()` and `jest.clearAllMocks()` so each test starts with a clean slate. This is faster than real `localStorage` and also avoids tests interfering with each other through shared storage. An alternative for newer setups is to use `jsdom`'s built-in localStorage which does work in tests, but it persists between tests in the same file unless you clear it manually — so explicit clearing in `beforeEach` is still needed."

**Trả lời (Tiếng Việt):**
"Mình sẽ mock `localStorage` ở module level bằng cách dùng `Object.defineProperty` trên `window.localStorage` với một mock object tự viết, backed bởi plain JavaScript object. Mock đó implement `getItem`, `setItem`, `removeItem`, và `clear` bằng `jest.fn()` để mình có thể assert xem cái gì đã được gọi và kiểm soát return value. Trong `beforeEach` gọi `.clear()` và `jest.clearAllMocks()` để mỗi test bắt đầu với trạng thái sạch. Cách này nhanh hơn `localStorage` thật và cũng tránh các test can thiệp lẫn nhau qua shared storage. Một alternative cho setup mới hơn là dùng built-in localStorage của `jsdom` — cái này cũng hoạt động trong test, nhưng nó persist giữa các test trong cùng file trừ khi mình clear thủ công — nên vẫn cần explicit clearing trong `beforeEach`."

---

---

## 4. React Testing Library Philosophy

**Level:** [Junior] / [Mid]

**EN:** What is the core philosophy of React Testing Library? Explain `render`, screen queries (`getBy`, `queryBy`, `findBy`), and the difference between `userEvent` and `fireEvent`.

**VI:** Triết lý cốt lõi của React Testing Library là gì? Giải thích các loại queries và sự khác biệt giữa `userEvent` và `fireEvent`.

**Trả lời:**

**Triết lý**: "Test như cách người dùng sử dụng, không test implementation details." Truy vấn DOM giống screen reader và người dùng thực — qua text, role, label — không qua class, id, hoặc component internals.

```tsx
// components/LoginForm.tsx
export function LoginForm({ onSubmit }: { onSubmit: (data: { email: string; password: string }) => void }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  return (
    <form onSubmit={(e) => { e.preventDefault(); onSubmit({ email, password }); }}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />

      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />

      <button type="submit">Log In</button>
    </form>
  );
}

// components/LoginForm.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

// ==============================
// QUERY TYPES
// ==============================
/*
  getBy*   — tìm 1 element, THROW nếu không thấy hoặc thấy nhiều
  queryBy* — tìm 1 element, trả null nếu không thấy (không throw)
  findBy*  — async, trả Promise, chờ element xuất hiện

  getAllBy*, queryAllBy*, findAllBy* — tương tự nhưng trả mảng

  Khi nào dùng:
  - getBy  → element phải tồn tại ngay
  - queryBy → assert element KHÔNG tồn tại (expect(queryBy...).not.toBeInTheDocument())
  - findBy → element xuất hiện sau async operation
*/

/*
  Query methods (ưu tiên từ trên xuống theo RTL docs):
  ByRole          → getByRole('button', { name: /submit/i }) — PREFERRED
  ByLabelText     → getByLabelText('Email')
  ByPlaceholderText → getByPlaceholderText('Enter email')
  ByText          → getByText('Submit')
  ByDisplayValue  → getByDisplayValue('Alice')
  ByAltText       → getByAltText('Profile picture')
  ByTitle         → getByTitle('Close')
  ByTestId        → getByTestId('submit-btn') — LAST RESORT
*/

describe('LoginForm', () => {
  it('renders all fields and submit button', () => {
    render(<LoginForm onSubmit={jest.fn()} />);

    // ByLabelText — accessible label
    expect(screen.getByLabelText('Email')).toBeInTheDocument();
    expect(screen.getByLabelText('Password')).toBeInTheDocument();

    // ByRole — semantic role
    expect(screen.getByRole('button', { name: /log in/i })).toBeInTheDocument();
  });

  it('calls onSubmit with form values', async () => {
    // userEvent.setup() — cách mới (v14+), simulate events thực tế hơn
    const user = userEvent.setup();
    const mockSubmit = jest.fn();

    render(<LoginForm onSubmit={mockSubmit} />);

    // userEvent mô phỏng interactions thực — focus, keyboard events, click events
    await user.type(screen.getByLabelText('Email'), 'alice@example.com');
    await user.type(screen.getByLabelText('Password'), 'secret123');
    await user.click(screen.getByRole('button', { name: /log in/i }));

    expect(mockSubmit).toHaveBeenCalledWith({
      email: 'alice@example.com',
      password: 'secret123',
    });
  });

  it('does not show error initially', () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    // queryBy trả null thay vì throw — dùng để assert absence
    expect(screen.queryByText(/required/i)).not.toBeInTheDocument();
  });
});

/*
  userEvent vs fireEvent:
  - fireEvent: gửi DOM events đơn giản (fireEvent.click, fireEvent.change)
               Không mô phỏng đầy đủ user behavior
  - userEvent: mô phỏng đầy đủ — type gõ từng ký tự, focus/blur, hover
               Closer to real user interaction → preferred

  fireEvent.change(input, { target: { value: 'hello' } }); // chỉ trigger change event
  await userEvent.type(input, 'hello'); // trigger: focus, keydown, keypress, input, keyup cho mỗi ký tự
*/
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What's the difference between `getBy`, `queryBy`, and `findBy` queries in React Testing Library?"**

**Model Answer:**
"They differ in how they handle missing elements. `getBy` expects the element to be there right now — it throws immediately if it's not found, which gives you a clear error message in your test. Use it when the element should definitely exist at that moment. `queryBy` returns null instead of throwing when nothing is found — that makes it the right choice when you want to assert something is NOT in the document: `expect(screen.queryByText('Error')).not.toBeInTheDocument()`. If you used `getBy` for that, you'd get an error before your assertion even runs. `findBy` is the async version — it returns a Promise and keeps trying until the element appears or the timeout expires, default one second. Use it after things like API calls, animations, or any state update that happens asynchronously."

**Trả lời (Tiếng Việt):**
"Ba cái này khác nhau ở cách xử lý khi không tìm thấy element. `getBy` mong đợi element phải có ngay lúc đó — nó throw ngay nếu không tìm thấy, cho mình error message rõ ràng trong test. Dùng khi element chắc chắn phải tồn tại tại thời điểm đó. `queryBy` trả null thay vì throw khi không tìm thấy gì — đây là lựa chọn đúng khi mình muốn assert rằng thứ gì đó KHÔNG có trong document: `expect(screen.queryByText('Error')).not.toBeInTheDocument()`. Nếu dùng `getBy` cho trường hợp đó thì sẽ bị lỗi trước cả khi assertion chạy. `findBy` là version async — nó trả Promise và cứ thử tiếp cho đến khi element xuất hiện hoặc timeout, mặc định là một giây. Dùng sau các operation như API call, animation, hay bất kỳ state update nào xảy ra bất đồng bộ."

---

**Q2 (Junior/Mid): "Why does React Testing Library discourage `getByTestId`? What should you use instead?"**

**Model Answer:**
"The philosophy of RTL is 'test like a user' — users interact with text, labels, and semantic roles, not test IDs. `getByTestId` tests implementation details: it'll still pass even if the button is visually broken or has wrong text. `getByRole` is preferred because it finds elements by their semantic ARIA role — `getByRole('button', { name: /submit/i })` confirms the element exists, has the right role for accessibility, and has the right label. `getByLabelText` is great for form inputs because it confirms the label is correctly associated, which also validates accessibility. The hierarchy from RTL docs goes: ByRole, ByLabelText, ByPlaceholderText, ByText, ByDisplayValue, ByAltText, ByTitle — and `ByTestId` is explicitly the last resort for things with no other accessible identity."

**Trả lời (Tiếng Việt):**
"Triết lý của RTL là 'test như user' — user tương tác với text, label, và semantic role, không phải test ID. `getByTestId` test implementation detail: nó vẫn pass dù button bị hỏng về mặt visual hoặc có text sai. `getByRole` được ưu tiên hơn vì nó tìm element theo semantic ARIA role — `getByRole('button', { name: /submit/i })` xác nhận element tồn tại, có đúng role cho accessibility, và có đúng label. `getByLabelText` rất tốt cho form input vì nó xác nhận label được associate đúng cách, qua đó cũng validate accessibility. Thứ tự ưu tiên theo RTL docs: ByRole, ByLabelText, ByPlaceholderText, ByText, ByDisplayValue, ByAltText, ByTitle — và `ByTestId` rõ ràng là lựa chọn cuối cùng cho những thứ không có accessible identity nào khác."

---

**Q3 (Mid): "What's the difference between `userEvent` and `fireEvent`? Which one should you use?"**

**Model Answer:**
"`fireEvent` is a lower-level API that dispatches a single DOM event — `fireEvent.click(button)` fires one click event. `userEvent` simulates what a real user does, which involves many events. When a user types a character, there's a `focus`, `keydown`, `keypress`, `input`, and `keyup` event in sequence. When a user clicks, there's `mouseover`, `mousemove`, `mousedown`, `mouseup`, then `click`. `userEvent` simulates all of that, which means components that rely on the full event sequence behave correctly in tests. Since version 14 the setup is `const user = userEvent.setup()` at the top of your test and then `await user.type(...)` and `await user.click(...)`. Prefer `userEvent` for anything that simulates user interaction. `fireEvent` is acceptable for low-level edge cases where you specifically want to test a single event."

**Trả lời (Tiếng Việt):**
"`fireEvent` là API cấp thấp, chỉ dispatch một DOM event đơn lẻ — `fireEvent.click(button)` chỉ fire một click event. `userEvent` mô phỏng những gì user thực sự làm, bao gồm rất nhiều event. Khi user gõ một ký tự, có `focus`, `keydown`, `keypress`, `input`, và `keyup` nối tiếp nhau. Khi user click, có `mouseover`, `mousemove`, `mousedown`, `mouseup`, rồi mới `click`. `userEvent` mô phỏng tất cả cái đó, nên các component phụ thuộc vào chuỗi event đầy đủ sẽ hoạt động đúng trong test. Từ version 14, setup là `const user = userEvent.setup()` ở đầu test rồi dùng `await user.type(...)` và `await user.click(...)`. Ưu tiên `userEvent` cho bất kỳ thứ gì mô phỏng tương tác của user. `fireEvent` chấp nhận được cho edge case cấp thấp khi mình muốn test đúng một event cụ thể."

---

**Q4 (Mid/Senior): "You get a warning: 'An update to Component inside a test was not wrapped in act(...)'. What does that mean and how do you fix it?"**

**Model Answer:**
"It means a state update happened after your test finished — the component is still doing work asynchronously and updating, but React can't flush those updates cleanly. It usually means you have an async operation you didn't wait for. The fix depends on the cause. If you have a `useEffect` that fetches data and updates state, you need to `await` a `findBy` query or wrap assertions in `waitFor` so the test waits for that update to complete. If you're using `userEvent`, make sure you're awaiting it — `await user.click(...)`. If you're rendering a component that kicks off async work in `useEffect`, you need at least one `await` somewhere so React can process all pending state updates. The warning itself is React telling you your test assertions ran too early."

**Trả lời (Tiếng Việt):**
"Cảnh báo này có nghĩa là state update xảy ra sau khi test đã chạy xong — component vẫn đang làm việc bất đồng bộ và update, nhưng React không thể flush các update đó một cách sạch. Thường là do có async operation mình chưa await. Cách fix tùy vào nguyên nhân. Nếu có `useEffect` fetch data và update state, cần `await` một `findBy` query hoặc bọc assertion trong `waitFor` để test chờ update hoàn thành. Nếu đang dùng `userEvent`, đảm bảo có await — `await user.click(...)`. Nếu render một component khởi động async work trong `useEffect`, cần ít nhất một `await` ở đâu đó để React xử lý hết các state update đang pending. Bản thân cảnh báo là React nói với mình rằng assertions trong test chạy quá sớm."

---

---

## 5. Async Testing in RTL

**Level:** [Mid]

**EN:** How do you test async behavior in React Testing Library? Explain `waitFor`, `findBy` queries, and `act()`.

**VI:** Làm thế nào để test async behavior trong RTL? Giải thích `waitFor`, `findBy` queries, và `act()`.

**Trả lời:**

```tsx
// components/UserList.tsx
export function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(() => setError('Failed to load users'))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div role="status">Loading...</div>;
  if (error)   return <div role="alert">{error}</div>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// UserList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { server } from '../mocks/server'; // MSW server
import { http, HttpResponse } from 'msw';

describe('UserList', () => {
  it('shows loading state initially', () => {
    render(<UserList />);
    // Ngay lập tức — spinner/loading role tồn tại
    expect(screen.getByRole('status')).toHaveTextContent('Loading...');
  });

  it('displays users after loading — findBy', async () => {
    render(<UserList />);

    // findBy = queryBy + waitFor — tự động wait
    // Timeout mặc định: 1000ms, poll mỗi 50ms
    const userItem = await screen.findByText('Alice Johnson');
    expect(userItem).toBeInTheDocument();

    // findAllBy — chờ nhiều elements
    const items = await screen.findAllByRole('listitem');
    expect(items).toHaveLength(3);
  });

  it('displays users after loading — waitFor', async () => {
    render(<UserList />);

    // waitFor: retry assertion cho đến khi pass hoặc timeout
    await waitFor(() => {
      expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    });

    // waitFor với options
    await waitFor(
      () => expect(screen.getAllByRole('listitem')).toHaveLength(3),
      { timeout: 3000, interval: 100 }
    );
  });

  it('shows error when fetch fails', async () => {
    // Override MSW handler cho test này
    server.use(
      http.get('/api/users', () => HttpResponse.error())
    );

    render(<UserList />);

    const errorMessage = await screen.findByRole('alert');
    expect(errorMessage).toHaveTextContent('Failed to load users');
  });

  it('loading indicator disappears after fetch', async () => {
    render(<UserList />);

    // Trước: loading
    expect(screen.getByRole('status')).toBeInTheDocument();

    // Sau khi data load
    await waitFor(() => {
      expect(screen.queryByRole('status')).not.toBeInTheDocument();
    });
  });
});

/*
  act() — đảm bảo tất cả state updates và effects được flush

  RTL tự động wrap trong act() cho hầu hết cases.
  Bạn cần wrap thủ công khi:
  1. Trigger state update bên ngoài render/userEvent (e.g., callback thủ công)
  2. Testing hooks với renderHook

  Khi thấy warning "not wrapped in act()" — thường có nghĩa là:
  - Async operation chưa được await
  - State update sau khi test kết thúc
  - Giải pháp: dùng findBy hoặc waitFor thay vì getBy
*/

// act() explicit — hiếm khi cần trong RTL
import { act } from '@testing-library/react';

it('updates on external event', async () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
    result.current.increment();
  });

  expect(result.current.count).toBe(2);
});
```

---

## 6. Testing Hooks

**Level:** [Mid]

**EN:** How do you test custom React hooks using `renderHook`? How do you handle state updates with `act()`?

**VI:** Làm thế nào để test custom hooks bằng `renderHook`? Cách handle state updates với `act()`?

**Trả lời:**

```ts
// hooks/useCounter.ts
export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount((c) => c + 1), []);
  const decrement = useCallback(() => setCount((c) => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  const incrementBy = useCallback((amount: number) => setCount((c) => c + amount), []);

  return { count, increment, decrement, reset, incrementBy };
}

// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.increment();
      result.current.increment();
    });

    expect(result.current.count).toBe(7);

    act(() => {
      result.current.reset();
    });

    expect(result.current.count).toBe(5); // về initial value, không phải 0
  });

  it('increments by amount', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.incrementBy(5);
    });

    expect(result.current.count).toBe(5);
  });
});
```

---

## 7. Testing Custom Hooks — useLocalStorage & useDebounce

**Level:** [Mid] / [Senior]

**EN:** Show how to test `useLocalStorage` and `useDebounce` hooks, including mocking `localStorage` and fake timers.

**VI:** Hướng dẫn test `useLocalStorage` và `useDebounce`, bao gồm mock `localStorage` và fake timers.

**Trả lời:**

```ts
// hooks/useLocalStorage.ts
export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue] as const;
}

// hooks/useLocalStorage.test.ts
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from './useLocalStorage';

// Mock localStorage
const localStorageMock = (() => {
  let store: Record<string, string> = {};
  return {
    getItem: jest.fn((key: string) => store[key] ?? null),
    setItem: jest.fn((key: string, value: string) => { store[key] = value; }),
    removeItem: jest.fn((key: string) => { delete store[key]; }),
    clear: jest.fn(() => { store = {}; }),
  };
})();

Object.defineProperty(window, 'localStorage', { value: localStorageMock });

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorageMock.clear();
    jest.clearAllMocks();
  });

  it('returns initial value when no stored value', () => {
    const { result } = renderHook(() => useLocalStorage('theme', 'light'));
    expect(result.current[0]).toBe('light');
  });

  it('returns stored value if exists', () => {
    localStorageMock.getItem.mockReturnValueOnce(JSON.stringify('dark'));

    const { result } = renderHook(() => useLocalStorage('theme', 'light'));
    expect(result.current[0]).toBe('dark');
  });

  it('stores value in localStorage on update', () => {
    const { result } = renderHook(() => useLocalStorage('theme', 'light'));

    act(() => {
      result.current[1]('dark');
    });

    expect(result.current[0]).toBe('dark');
    expect(localStorageMock.setItem).toHaveBeenCalledWith('theme', '"dark"');
  });

  it('supports functional updates', () => {
    const { result } = renderHook(() => useLocalStorage<number>('count', 0));

    act(() => {
      result.current[1]((prev) => prev + 1);
    });

    expect(result.current[0]).toBe(1);
  });
});

// ==============================
// hooks/useDebounce.ts
// ==============================
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// hooks/useDebounce.test.ts
import { renderHook, act } from '@testing-library/react';
import { useDebounce } from './useDebounce';

describe('useDebounce', () => {
  beforeEach(() => jest.useFakeTimers());
  afterEach(() => jest.useRealTimers());

  it('returns initial value immediately', () => {
    const { result } = renderHook(() => useDebounce('hello', 500));
    expect(result.current).toBe('hello');
  });

  it('returns stale value before delay', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'hello', delay: 500 } }
    );

    rerender({ value: 'world', delay: 500 });

    // Chưa đến 500ms — vẫn là giá trị cũ
    expect(result.current).toBe('hello');
  });

  it('updates value after delay', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'hello', delay: 500 } }
    );

    rerender({ value: 'world', delay: 500 });

    act(() => {
      jest.advanceTimersByTime(500);
    });

    expect(result.current).toBe('world');
  });

  it('only updates once for rapid changes', () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 300),
      { initialProps: { value: 'a' } }
    );

    rerender({ value: 'ab' });
    rerender({ value: 'abc' });
    rerender({ value: 'abcd' });

    act(() => {
      jest.advanceTimersByTime(300);
    });

    // Chỉ update với giá trị cuối cùng
    expect(result.current).toBe('abcd');
  });
});
```

---

## 8. Testing Forms

**Level:** [Mid]

**EN:** How do you test form interactions, including filling inputs, submitting, and validation errors?

**VI:** Làm thế nào để test form interactions: fill inputs, submit, validation errors?

**Trả lời:**

```tsx
// components/RegistrationForm.tsx
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type FormData = z.infer<typeof schema>;

export function RegistrationForm({ onSubmit }: { onSubmit: (data: FormData) => Promise<void> }) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="name">Name</label>
        <input id="name" {...register('name')} aria-describedby="name-error" />
        {errors.name && <span id="name-error" role="alert">{errors.name.message}</span>}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register('email')} aria-describedby="email-error" />
        {errors.email && <span id="email-error" role="alert">{errors.email.message}</span>}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input id="password" type="password" {...register('password')} />
        {errors.password && <span role="alert">{errors.password.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating account...' : 'Create Account'}
      </button>
    </form>
  );
}

// RegistrationForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { RegistrationForm } from './RegistrationForm';

const validData = {
  name: 'Alice Johnson',
  email: 'alice@example.com',
  password: 'securepassword123',
};

describe('RegistrationForm', () => {
  const user = userEvent.setup();
  const mockSubmit = jest.fn().mockResolvedValue(undefined);

  beforeEach(() => {
    mockSubmit.mockClear();
    render(<RegistrationForm onSubmit={mockSubmit} />);
  });

  async function fillForm(data: Partial<typeof validData>) {
    if (data.name)     await user.type(screen.getByLabelText('Name'), data.name);
    if (data.email)    await user.type(screen.getByLabelText('Email'), data.email);
    if (data.password) await user.type(screen.getByLabelText('Password'), data.password);
  }

  it('submits valid form data', async () => {
    await fillForm(validData);
    await user.click(screen.getByRole('button', { name: /create account/i }));

    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith(validData);
    });
  });

  it('shows validation errors for empty form', async () => {
    await user.click(screen.getByRole('button', { name: /create account/i }));

    // waitFor vì validation có thể là async
    await waitFor(() => {
      const alerts = screen.getAllByRole('alert');
      expect(alerts.length).toBeGreaterThan(0);
    });
  });

  it('shows name too short error', async () => {
    await user.type(screen.getByLabelText('Name'), 'A'); // chỉ 1 ký tự
    await user.click(screen.getByRole('button', { name: /create account/i }));

    await waitFor(() => {
      expect(screen.getByText(/at least 2 characters/i)).toBeInTheDocument();
    });
  });

  it('shows invalid email error', async () => {
    await user.type(screen.getByLabelText('Email'), 'not-an-email');
    await user.click(screen.getByRole('button', { name: /create account/i }));

    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
    });
  });

  it('disables button and shows loading during submission', async () => {
    // mock slow submission
    mockSubmit.mockImplementation(() => new Promise((res) => setTimeout(res, 500)));

    await fillForm(validData);
    await user.click(screen.getByRole('button', { name: /create account/i }));

    // Ngay sau click — button bị disable
    expect(screen.getByRole('button', { name: /creating account/i })).toBeDisabled();
  });

  it('allows form resubmission after success', async () => {
    await fillForm(validData);
    await user.click(screen.getByRole('button', { name: /create account/i }));

    await waitFor(() => expect(mockSubmit).toHaveBeenCalledTimes(1));

    // Clear fields và submit lại
    await user.clear(screen.getByLabelText('Email'));
    await user.type(screen.getByLabelText('Email'), 'bob@example.com');
  });
});
```

---

## 9. Testing with MSW (Mock Service Worker)

**Level:** [Mid] / [Senior]

**EN:** What is MSW and why is it better than mocking `fetch` directly? Show how to set up handlers and `setupServer`.

**VI:** MSW là gì và tại sao tốt hơn mock `fetch` trực tiếp? Cách setup handlers và `setupServer`?

**Trả lời:**

MSW intercept requests ở network level (Service Worker trong browser, Node http interceptor trong tests) thay vì mock function. Điều này giúp test gần với thực tế hơn.

**MSW vs mock fetch:**
- `jest.fn()` mock fetch → test implementation detail (bạn test rằng code gọi fetch, không test rằng nó handle response đúng)
- MSW → test thực sự HTTP layer, dễ share handlers giữa tests và Storybook, dễ test error cases

```ts
// mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export interface User {
  id: string;
  name: string;
  email: string;
}

export const handlers = [
  // GET /api/users
  http.get('/api/users', () => {
    return HttpResponse.json<User[]>([
      { id: '1', name: 'Alice Johnson', email: 'alice@example.com' },
      { id: '2', name: 'Bob Smith', email: 'bob@example.com' },
    ]);
  }),

  // GET /api/users/:id
  http.get('/api/users/:id', ({ params }) => {
    const { id } = params;
    if (id === '404') {
      return HttpResponse.json({ error: 'User not found' }, { status: 404 });
    }
    return HttpResponse.json<User>({
      id: id as string,
      name: 'Alice Johnson',
      email: 'alice@example.com',
    });
  }),

  // POST /api/users
  http.post('/api/users', async ({ request }) => {
    const body = await request.json() as Omit<User, 'id'>;
    return HttpResponse.json<User>(
      { id: Date.now().toString(), ...body },
      { status: 201 }
    );
  }),
];

// mocks/server.ts — Node environment (Jest)
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// jest.setup.ts (setupFilesAfterFramework)
import { server } from './mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers()); // reset overrides sau mỗi test
afterAll(() => server.close());

// ==============================
// Tests sử dụng MSW
// ==============================
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { server } from '../mocks/server';
import { http, HttpResponse } from 'msw';
import { UserList } from './UserList';
import { CreateUserForm } from './CreateUserForm';

describe('UserList with MSW', () => {
  it('fetches and displays users', async () => {
    render(<UserList />);

    // MSW intercepts /api/users và trả data từ handler
    expect(await screen.findByText('Alice Johnson')).toBeInTheDocument();
    expect(await screen.findByText('Bob Smith')).toBeInTheDocument();
  });

  it('handles API error gracefully', async () => {
    // Override handler chỉ cho test này
    server.use(
      http.get('/api/users', () => {
        return HttpResponse.json({ error: 'Internal server error' }, { status: 500 });
      })
    );

    render(<UserList />);

    expect(await screen.findByRole('alert')).toHaveTextContent(/failed to load/i);
  });

  it('handles network error', async () => {
    server.use(
      http.get('/api/users', () => HttpResponse.error()) // network error
    );

    render(<UserList />);

    expect(await screen.findByRole('alert')).toBeInTheDocument();
  });
});

describe('CreateUserForm with MSW', () => {
  it('creates user and shows success', async () => {
    const user = userEvent.setup();
    render(<CreateUserForm />);

    await user.type(screen.getByLabelText('Name'), 'Charlie Brown');
    await user.type(screen.getByLabelText('Email'), 'charlie@example.com');
    await user.click(screen.getByRole('button', { name: /create/i }));

    expect(await screen.findByText(/user created/i)).toBeInTheDocument();
  });
});
```

---

## 10. Cypress Basics

**Level:** [Mid]

**EN:** Explain Cypress fundamentals: `cy.visit`, `cy.get`, `cy.intercept`, and custom commands.

**VI:** Giải thích các basics của Cypress: `cy.visit`, `cy.get`, `cy.intercept`, và custom commands.

**Trả lời:**

```ts
// cypress/e2e/auth.cy.ts
describe('Authentication Flow', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('logs in successfully', () => {
    // cy.get — tìm element (CSS selector hoặc data-cy attribute)
    cy.get('[data-cy="email-input"]').type('alice@example.com');
    cy.get('[data-cy="password-input"]').type('password123');
    cy.get('[data-cy="submit-button"]').click();

    // Assert redirect và welcome message
    cy.url().should('include', '/dashboard');
    cy.contains('Welcome, Alice').should('be.visible');
  });

  it('shows error for invalid credentials', () => {
    cy.intercept('POST', '/api/auth/login', {
      statusCode: 401,
      body: { error: 'Invalid credentials' },
    }).as('loginRequest'); // đặt alias

    cy.get('[data-cy="email-input"]').type('wrong@example.com');
    cy.get('[data-cy="password-input"]').type('wrongpassword');
    cy.get('[data-cy="submit-button"]').click();

    // Chờ request complete
    cy.wait('@loginRequest');

    cy.get('[data-cy="error-message"]')
      .should('be.visible')
      .and('contain', 'Invalid credentials');
  });
});

// cy.intercept — intercept network requests
describe('User List', () => {
  it('displays users from API', () => {
    cy.intercept('GET', '/api/users', {
      statusCode: 200,
      body: [
        { id: '1', name: 'Alice', email: 'alice@example.com' },
        { id: '2', name: 'Bob', email: 'bob@example.com' },
      ],
    }).as('getUsers');

    cy.visit('/users');
    cy.wait('@getUsers');

    cy.get('[data-cy="user-list"] li').should('have.length', 2);
    cy.contains('Alice').should('be.visible');
  });

  it('intercepts with dynamic handler', () => {
    cy.intercept('GET', '/api/users/*', (req) => {
      req.reply({
        statusCode: 200,
        body: { id: req.url.split('/').pop(), name: 'Mock User' },
      });
    });
  });
});

// ==============================
// Custom Commands
// ==============================
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      getByTestId(testId: string): Chainable<JQuery<HTMLElement>>;
    }
  }
}

// Reusable login command — tránh lặp code
Cypress.Commands.add('login', (email: string, password: string) => {
  // Option 1: qua UI (chậm nhưng test UI)
  cy.visit('/login');
  cy.get('[data-cy="email-input"]').type(email);
  cy.get('[data-cy="password-input"]').type(password);
  cy.get('[data-cy="submit-button"]').click();

  // Option 2: qua API (nhanh, dùng cho tests không liên quan đến auth)
  // cy.request('POST', '/api/auth/login', { email, password }).then(({ body }) => {
  //   window.localStorage.setItem('token', body.token);
  // });
});

Cypress.Commands.add('getByTestId', (testId: string) => {
  return cy.get(`[data-cy="${testId}"]`);
});

// Sử dụng custom commands
cy.login('alice@example.com', 'password123');
cy.getByTestId('dashboard-title').should('be.visible');
```

---

## 11. Cypress vs Playwright

**Level:** [Senior]

**EN:** Compare Cypress and Playwright. What are the tradeoffs and when would you choose one over the other?

**VI:** So sánh Cypress và Playwright. Tradeoffs và khi nào chọn từng tool?

**Trả lời:**

| | Cypress | Playwright |
|--|---------|-----------|
| Architecture | Runs IN browser | Controls browser from outside |
| Browser support | Chrome, Firefox, Edge | Chrome, Firefox, Safari, Edge |
| Cross-browser | Hạn chế (no Safari) | Tốt, bao gồm Safari/WebKit |
| Multi-tab/window | Không hỗ trợ tốt | Hỗ trợ tốt |
| iframes | Phức tạp | Dễ hơn |
| Parallel execution | Cần Cypress Cloud (trả phí) | Built-in |
| Mobile testing | Viewport emulation | Device emulation tốt hơn |
| Network intercept | cy.intercept() | page.route() |
| DX / Debug | Xuất sắc (Time Travel) | Tốt (Trace Viewer) |
| Speed | Trung bình | Nhanh hơn |
| Learning curve | Dễ | Trung bình |

```ts
// Playwright equivalent của Cypress examples trên
// playwright/tests/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('logs in successfully', async ({ page }) => {
    await page.goto('/login');

    await page.getByLabel('Email').fill('alice@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Log In' }).click();

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome, Alice')).toBeVisible();
  });

  test('intercepts network requests', async ({ page }) => {
    // Playwright route interception
    await page.route('/api/users', (route) => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([{ id: '1', name: 'Alice' }]),
      });
    });

    await page.goto('/users');
    await expect(page.getByText('Alice')).toBeVisible();
  });

  // Playwright-exclusive: multiple tabs
  test('opens link in new tab', async ({ page, context }) => {
    const [newPage] = await Promise.all([
      context.waitForEvent('page'),
      page.click('[data-testid="open-docs"]'),
    ]);

    await expect(newPage).toHaveURL(/docs/);
  });
});

// playwright.config.ts
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
  testDir: './playwright/tests',
  fullyParallel: true, // run all tests in parallel
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  reporter: [['html'], ['github']],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry', // Trace Viewer khi fail
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'firefox',  use: { browserName: 'firefox' } },
    { name: 'webkit',   use: { browserName: 'webkit' } }, // Safari — chỉ Playwright có
  ],
};

export default config;
```

**Khi nào chọn gì:**
- **Cypress**: Team mới với e2e, cần DX tốt, Time Travel debugging, đã dùng Cypress Cloud
- **Playwright**: Cần cross-browser (Safari), parallel miễn phí, multi-tab, CI/CD tốc độ cao, mobile emulation

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "What's the difference between unit tests, integration tests, and E2E tests? How do you decide what to write?"**

**Model Answer:**
"Unit tests cover a single function or component in isolation, with all dependencies mocked. They're fast — milliseconds each — and you can have thousands of them. Integration tests check that multiple pieces work together: a form component with its validation logic, a custom hook with the state it manages, or a component that reads from a context. E2E tests run against a real or staging environment and drive an actual browser through user flows. The classic advice is the testing pyramid: lots of unit tests, fewer integration tests, a small number of E2E tests. E2E tests are expensive to write, slow to run, and can be flaky on network or timing issues. I focus E2E on critical user journeys — login, checkout, core workflows — and rely on unit and integration tests for edge cases and error handling."

**Trả lời (Tiếng Việt):**
"Unit test bao phủ một function hoặc component đơn lẻ trong môi trường cô lập, tất cả dependency đều được mock. Chúng rất nhanh — vài millisecond mỗi test — và có thể có hàng nghìn cái. Integration test kiểm tra nhiều phần phối hợp với nhau: form component với validation logic của nó, custom hook với state nó quản lý, hoặc component đọc từ context. E2E test chạy trên môi trường thật hoặc staging và điều khiển browser thật đi qua các user flow. Lời khuyên cổ điển là testing pyramid: nhiều unit test, ít integration test hơn, một số ít E2E test. E2E test tốn công viết, chạy chậm, và có thể flaky vì network hay timing. Mình tập trung E2E vào các critical user journey — login, checkout, core workflow — và dựa vào unit và integration test cho edge case và error handling."

---

**Q2 (Mid): "What is Cypress's 'Time Travel' feature and why is it useful for debugging?"**

**Model Answer:**
"Cypress captures a DOM snapshot at every command step — every `cy.get`, every `cy.click`, every assertion. In the Cypress test runner, you can hover over any step in the command log and the UI rewinds to show you exactly what the DOM looked like at that point. This is extremely useful for debugging failing tests because you can see what was on screen when Cypress tried to click something, whether the element existed, what text it had, and whether it was visible. You can also pin a step and get browser devtools access to inspect the DOM at that exact moment. It's the biggest DX advantage Cypress has over alternatives, especially for developers who are new to writing E2E tests."

**Trả lời (Tiếng Việt):**
"Cypress chụp DOM snapshot ở mỗi bước lệnh — mỗi `cy.get`, mỗi `cy.click`, mỗi assertion. Trong Cypress test runner, mình có thể hover vào bất kỳ bước nào trong command log và UI sẽ tua lại để hiện chính xác DOM trông như thế nào tại thời điểm đó. Cực kỳ hữu ích để debug test fail vì mình có thể thấy trên màn hình có gì khi Cypress cố click, element có tồn tại không, text là gì, và có visible không. Mình cũng có thể pin một bước và mở browser devtools để inspect DOM tại đúng moment đó. Đây là lợi thế DX lớn nhất của Cypress so với các công cụ khác, đặc biệt với developer mới bắt đầu viết E2E test."

---

**Q3 (Mid/Senior): "Why would you choose Playwright over Cypress for a project?"**

**Model Answer:**
"There are a few clear reasons. First, Safari — Cypress doesn't support WebKit, so if your users include Safari on iPhone or Mac, Playwright is the only option for cross-browser E2E. Second, parallel execution: Playwright runs tests in parallel by default at no extra cost. Cypress requires the paid Cypress Cloud service for parallel CI runs, which adds budget friction. Third, multi-tab and multi-page scenarios — Playwright handles those natively, Cypress has significant limitations. Fourth, speed — Playwright is generally faster in CI because of the built-in parallel workers. I'd still choose Cypress if the team is new to E2E and values the interactive runner experience for writing and debugging tests, or if you're already invested in the Cypress ecosystem."

**Trả lời (Tiếng Việt):**
"Có một vài lý do rõ ràng. Đầu tiên là Safari — Cypress không hỗ trợ WebKit, nên nếu user của mình dùng Safari trên iPhone hay Mac, Playwright là lựa chọn duy nhất cho cross-browser E2E. Thứ hai là parallel execution: Playwright chạy test song song mặc định mà không tốn thêm chi phí. Cypress cần dịch vụ Cypress Cloud trả phí để chạy song song trên CI, điều đó gây friction về ngân sách. Thứ ba là multi-tab và multi-page — Playwright xử lý tốt, Cypress có nhiều hạn chế. Thứ tư là tốc độ — Playwright thường nhanh hơn trên CI nhờ built-in parallel workers. Mình vẫn chọn Cypress nếu team mới làm quen với E2E và cần interactive runner để viết và debug test, hoặc đã đầu tư vào Cypress ecosystem rồi."

---

**Q4 (Senior): "Your E2E tests are flaky — they pass most of the time but fail occasionally. What are the common causes and how do you fix them?"**

**Model Answer:**
"Flakiness in E2E tests almost always comes from timing. The most common cause is a test asserting something exists before the UI has finished updating — for example, clicking a button and immediately checking for a success message before the API call completes. The fix is to wait for something explicit rather than using fixed delays: in Cypress use `cy.wait('@aliasedRequest')` to wait for a specific network request, or chain `.should('be.visible')` which automatically retries. In Playwright use `await expect(page.getByText('Success')).toBeVisible()` which retries until the element appears. Other causes: network instability in CI — solved by stubbing requests with `cy.intercept` or `page.route`; shared state between tests — solved by resetting database and auth state in `beforeEach` hooks; and animation delays — solved by disabling CSS animations in test environments."

**Trả lời (Tiếng Việt):**
"Flakiness trong E2E test gần như luôn đến từ timing. Nguyên nhân phổ biến nhất là test assert thứ gì đó tồn tại trước khi UI cập nhật xong — ví dụ click button xong kiểm tra success message ngay trong khi API call chưa hoàn thành. Cách fix là chờ một thứ gì đó cụ thể thay vì dùng fixed delay: trong Cypress dùng `cy.wait('@aliasedRequest')` để chờ đúng network request, hoặc chain `.should('be.visible')` vốn tự động retry. Trong Playwright dùng `await expect(page.getByText('Success')).toBeVisible()` sẽ retry cho đến khi element xuất hiện. Các nguyên nhân khác: network không ổn định trong CI — giải quyết bằng cách stub request với `cy.intercept` hoặc `page.route`; shared state giữa các test — giải quyết bằng cách reset database và auth state trong `beforeEach` hook; và animation delay — giải quyết bằng cách disable CSS animation trong môi trường test."

---

---

## 12. Test Coverage

**Level:** [Mid]

**EN:** What does test coverage mean? What does 100% coverage NOT guarantee? What is the "80% rule"?

**VI:** Test coverage có nghĩa gì? 100% coverage KHÔNG đảm bảo điều gì? "80% rule" là gì?

**Trả lời:**

```ts
// Coverage types:
// - Line coverage: mỗi dòng đã được execute?
// - Branch coverage: mỗi nhánh if/else đã được test?
// - Function coverage: mỗi function đã được gọi?
// - Statement coverage: mỗi statement đã chạy?

// Ví dụ: hàm này có 100% line coverage nhưng bug vẫn tồn tại
function calculateDiscount(price: number, userType: string): number {
  if (userType === 'premium') {
    return price * 0.8; // 20% off
  }
  return price; // no discount
}

// Test này đạt 100% line coverage
test('premium discount', () => {
  expect(calculateDiscount(100, 'premium')).toBe(80);  // ✓ dòng 1
  expect(calculateDiscount(100, 'regular')).toBe(100); // ✓ dòng 2
});

// Nhưng không test:
// - Giá trị âm: calculateDiscount(-100, 'premium') → -80 (có vấn đề không?)
// - userType = 'PREMIUM' (case sensitive?)
// - price = 0
// - price = NaN

// Coverage KHÔNG đảm bảo:
// 1. Logic correctness — test có thể pass sai assertion
// 2. Edge cases — boundary values
// 3. Integration correctness
// 4. User experience

// jest.config.ts — coverage configuration
export default {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{ts,tsx}',
    '!src/main.tsx',         // entry point
    '!src/vite-env.d.ts',
  ],
  // 80% rule — practical threshold
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Strict cho critical paths
    './src/utils/payment.ts': {
      branches: 100,
      functions: 100,
    },
  },
};

/*
  80% rule thực tế:
  - 80-90% là target hợp lý cho most codebases
  - 100% có chi phí maintenance cao, giảm tốc độ phát triển
  - Tập trung coverage vào: business logic, utils, custom hooks
  - Không cần 100% cho: UI styling, boilerplate, simple getters
  - Coverage tốt nhất: đo theo feature criticality, không chỉ số %
*/
```

---

## 13. Snapshot Testing

**Level:** [Mid]

**EN:** When is snapshot testing useful? When does it become a maintenance burden?

**VI:** Khi nào snapshot testing hữu ích? Khi nào nó trở thành gánh nặng maintenance?

**Trả lời:**

```tsx
// Snapshot testing — chụp ảnh output, so sánh lần sau
import { render } from '@testing-library/react';
import { Button } from './Button';

describe('Button snapshot', () => {
  it('matches snapshot', () => {
    const { container } = render(<Button variant="primary">Click me</Button>);
    expect(container.firstChild).toMatchSnapshot();
    // Lần đầu: tạo file __snapshots__/Button.test.tsx.snap
    // Lần sau: so sánh với snapshot cũ, fail nếu khác
  });
});

// __snapshots__/Button.test.tsx.snap (auto-generated)
// exports[`Button snapshot matches snapshot 1`] = `
// <button
//   class="button button--primary"
//   type="button"
// >
//   Click me
// </button>
// `;

// ==============================
// KHI NÀO SNAPSHOT HỮU ÍCH
// ==============================
// 1. Component output ổn định, ít thay đổi
// 2. Bắt unexpected UI regressions
// 3. Simple, pure presentational components

// Inline snapshots (tốt hơn, diff ngay trong test file)
it('renders correctly', () => {
  const { container } = render(<Tag>JavaScript</Tag>);
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span
      class="tag"
    >
      JavaScript
    </span>
  `);
});

// ==============================
// KHI NÀO SNAPSHOT LÀ GỌI NẶC
// ==============================

// Vấn đề: Bất kỳ thay đổi UI nào cũng fail snapshot
// Dev có xu hướng: jest --updateSnapshot (--u) mù quáng
// Snapshot mất ý nghĩa — chỉ là noise

// Ví dụ snapshot lớn, khó review:
it('matches full page snapshot', () => {
  const { container } = render(<DashboardPage data={mockData} />);
  expect(container).toMatchSnapshot(); // snapshot hàng trăm dòng HTML — VÔ DỤNG
});

// THAY VÀO ĐÓ: Test behavior cụ thể
it('dashboard shows user name', () => {
  render(<DashboardPage data={mockData} />);
  expect(screen.getByText('Welcome, Alice')).toBeInTheDocument();
});

it('dashboard shows correct stats', () => {
  render(<DashboardPage data={mockData} />);
  expect(screen.getByText('142 orders')).toBeInTheDocument();
});

/*
  Best practices:
  - Snapshot nhỏ, focused (chỉ phần quan trọng)
  - Dùng inline snapshots thay vì file snapshots
  - Tránh snapshot cho dynamic content (dates, IDs)
  - Prefer toMatchInlineSnapshot với toMatchSnapshot
  - Dùng visual regression (Chromatic) thay vì snapshot cho UI components
*/
```

---

## 14. Testing Strategy for Component Libraries

**Level:** [Senior]

**EN:** What is the testing strategy for a component library? How do you combine unit tests, visual regression, and Storybook/Chromatic?

**VI:** Chiến lược testing cho component library là gì? Kết hợp unit tests, visual regression, và Storybook/Chromatic như thế nào?

**Trả lời:**

```tsx
// ==============================
// 1. UNIT TESTS — behavior & accessibility
// ==============================

// components/Accordion/Accordion.test.tsx
describe('Accordion', () => {
  it('renders all items collapsed by default', () => {
    render(
      <Accordion>
        <Accordion.Item title="Item 1">Content 1</Accordion.Item>
        <Accordion.Item title="Item 2">Content 2</Accordion.Item>
      </Accordion>
    );

    expect(screen.getByText('Content 1')).not.toBeVisible();
  });

  it('expands item on click', async () => {
    const user = userEvent.setup();
    render(
      <Accordion>
        <Accordion.Item title="Item 1">Content 1</Accordion.Item>
      </Accordion>
    );

    await user.click(screen.getByRole('button', { name: 'Item 1' }));
    expect(screen.getByText('Content 1')).toBeVisible();
  });

  // ARIA attributes — accessibility
  it('has correct ARIA attributes', async () => {
    const user = userEvent.setup();
    render(
      <Accordion>
        <Accordion.Item title="Section 1">Content</Accordion.Item>
      </Accordion>
    );

    const trigger = screen.getByRole('button', { name: 'Section 1' });
    expect(trigger).toHaveAttribute('aria-expanded', 'false');

    await user.click(trigger);
    expect(trigger).toHaveAttribute('aria-expanded', 'true');
  });

  // Keyboard navigation
  it('supports keyboard navigation', async () => {
    const user = userEvent.setup();
    render(<Accordion>...</Accordion>);

    screen.getByRole('button', { name: 'Item 1' }).focus();
    await user.keyboard('{Enter}');
    // assert expanded
  });
});

// ==============================
// 2. STORYBOOK — Component Catalog + Manual Testing
// ==============================

// components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],  // auto-generate docs
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost', 'danger'],
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
    },
    disabled: { control: 'boolean' },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

// Mỗi story = 1 visual test case
export const Primary: Story = {
  args: { variant: 'primary', children: 'Primary Button' },
};

export const Secondary: Story = {
  args: { variant: 'secondary', children: 'Secondary Button' },
};

export const Disabled: Story = {
  args: { variant: 'primary', disabled: true, children: 'Disabled' },
};

export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', flexWrap: 'wrap' }}>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="danger">Danger</Button>
    </div>
  ),
};

// ==============================
// 3. CHROMATIC — Visual Regression Testing
// ==============================
// Chromatic chụp screenshot mỗi story, so sánh pixel-by-pixel khi PR

// .chromatic.config.ts
export default {
  projectToken: process.env.CHROMATIC_PROJECT_TOKEN,
  buildScriptName: 'build-storybook',
  exitOnceUploaded: true,
  onlyChanged: true, // chỉ test stories thay đổi
};

// GitHub Actions workflow
// chromatic:
//   runs-on: ubuntu-latest
//   steps:
//     - uses: chromaui/action@latest
//       with:
//         projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
//         onlyChanged: true

// ==============================
// 4. AXE — Accessibility Testing
// ==============================
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

it('has no accessibility violations', async () => {
  const { container } = render(
    <Button variant="primary" aria-label="Submit form">Submit</Button>
  );
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

/*
  Full testing strategy cho component library:
  1. Unit tests (Jest + RTL): behavior, interactions, keyboard nav, ARIA
  2. Accessibility tests (jest-axe): automated a11y violations
  3. Storybook stories: living documentation, manual QA
  4. Chromatic: visual regression CI (catch unintended visual changes)
  5. Integration tests: components trong context thực (forms, layouts)

  CI Pipeline:
  PR → lint → unit tests → build storybook → chromatic → deploy preview
*/
```
