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
