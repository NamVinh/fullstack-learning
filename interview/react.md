# React — Interview Questions (Junior → Senior)

> Tổng hợp câu hỏi phỏng vấn React từ Junior đến Senior.
> Câu hỏi song ngữ (EN/VI), trả lời chi tiết bằng tiếng Việt, code ví dụ bằng tiếng Anh.

---

## Table of Contents

1. [JSX là gì? Nó compile thành gì?](#1-jsx)
2. [Props vs State — sự khác biệt, lifting state up](#2-props-vs-state)
3. [Component Lifecycle — class vs hooks](#3-lifecycle)
4. [Virtual DOM và Reconciliation](#4-virtual-dom)
5. [useState vs useReducer](#5-usestate-vs-usereducer)
6. [useEffect — dependency array, cleanup, common mistakes](#6-useeffect)
7. [useMemo vs useCallback](#7-usememo-vs-usecallback)
8. [React.memo — shallow comparison](#8-react-memo)
9. [Custom Hooks — patterns](#9-custom-hooks)
10. [Context API — provider pattern, performance](#10-context-api)
11. [useRef — DOM ref vs mutable value, forwardRef](#11-useref)
12. [Controlled vs Uncontrolled components](#12-controlled-vs-uncontrolled)
13. [React.lazy + Suspense — code splitting](#13-lazy-suspense)
14. [Error Boundaries](#14-error-boundaries)
15. [React 18 — new features](#15-react-18)
16. [Portals](#16-portals)
17. [HOC vs Render Props vs Hooks](#17-hoc-render-props-hooks)
18. [Performance Optimization](#18-performance)
19. [Strict Mode](#19-strict-mode)
20. [Concurrent Mode](#20-concurrent-mode)

---

## 1. JSX

**[Junior]**

**EN:** What is JSX? What does it compile to? Why do list items need a `key` prop?

**VI:** JSX là gì? Nó được biên dịch thành gì? Tại sao các phần tử trong danh sách cần prop `key`?

### Trả lời

**JSX (JavaScript XML)** là một cú pháp mở rộng của JavaScript cho phép viết code trông giống HTML bên trong file JS/TS. JSX không phải HTML thật — nó là syntactic sugar được Babel (hoặc SWC) biên dịch thành các lời gọi hàm JavaScript thuần.

**Trước React 17**, JSX biên dịch sang `React.createElement()`:

```jsx
// JSX
const element = <h1 className="title">Hello, World!</h1>;

// Compiled output (pre React 17)
const element = React.createElement(
  'h1',
  { className: 'title' },
  'Hello, World!'
);
```

**Từ React 17+**, Babel dùng transform mới — không cần import React nữa:

```jsx
// JSX
const element = <h1>Hello</h1>;

// Compiled output (React 17+ automatic transform)
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { children: 'Hello' });
```

**Tại sao `key` quan trọng trong danh sách?**

Khi render một danh sách, React cần xác định element nào thay đổi, được thêm, hoặc bị xóa giữa các lần render. `key` là định danh duy nhất giúp React làm điều này hiệu quả trong quá trình reconciliation.

```jsx
// BAD — dùng index làm key gây bug khi reorder
const list = items.map((item, index) => (
  <li key={index}>{item.name}</li>
));

// GOOD — dùng stable, unique ID
const list = items.map((item) => (
  <li key={item.id}>{item.name}</li>
));
```

Nếu không có `key` hoặc dùng `index` khi danh sách có thể bị sắp xếp lại/xóa, React có thể:
- Tái sử dụng sai DOM node → bug hiển thị
- Không unmount/remount component đúng cách → state bị giữ lại sai

---

## 2. Props vs State

**[Junior]**

**EN:** What is the difference between props and state? What does "lifting state up" mean?

**VI:** Props và State khác nhau như thế nào? "Lifting state up" nghĩa là gì?

### Trả lời

| Đặc điểm | Props | State |
|-----------|-------|-------|
| Nguồn gốc | Được truyền từ parent | Được quản lý nội bộ trong component |
| Ai thay đổi được | Chỉ parent mới thay đổi | Chính component đó thay đổi |
| Mutable từ component | Không (read-only) | Có (qua setter function) |
| Mục đích | Giao tiếp parent → child | Lưu trữ trạng thái nội bộ |

```jsx
// Props — read-only, từ parent xuống
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// State — local, component tự quản lý
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**Lifting State Up** là kỹ thuật di chuyển state lên component cha chung (closest common ancestor) khi nhiều component cần chia sẻ cùng một dữ liệu.

```jsx
// BEFORE: mỗi component giữ state riêng → không đồng bộ
function TemperatureInput() {
  const [temp, setTemp] = useState('');
  return <input value={temp} onChange={e => setTemp(e.target.value)} />;
}

// AFTER: lift state lên parent
function TemperatureCalculator() {
  const [celsius, setCelsius] = useState('');
  const fahrenheit = celsius ? (celsius * 9/5 + 32).toFixed(1) : '';

  return (
    <div>
      <TemperatureInput
        value={celsius}
        onChange={setCelsius}
        label="Celsius"
      />
      <TemperatureInput
        value={fahrenheit}
        onChange={(f) => setCelsius(((f - 32) * 5/9).toFixed(1))}
        label="Fahrenheit"
      />
    </div>
  );
}

function TemperatureInput({ value, onChange, label }) {
  return (
    <label>
      {label}: <input value={value} onChange={e => onChange(e.target.value)} />
    </label>
  );
}
```

**Nguyên tắc:** State nên ở component thấp nhất cần nó. Nếu nhiều siblings cần cùng state → lift lên parent chung của chúng.

---

## 3. Lifecycle

**[Junior → Mid]**

**EN:** Explain React component lifecycle. How do hooks replace lifecycle methods?

**VI:** Giải thích vòng đời (lifecycle) của React component. Hooks thay thế lifecycle methods như thế nào?

### Trả lời

**Class component lifecycle** có 3 giai đoạn chính:

1. **Mounting** — component được tạo và chèn vào DOM
2. **Updating** — component re-render khi props/state thay đổi
3. **Unmounting** — component bị xóa khỏi DOM

```jsx
// Class component lifecycle
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { data: null };
  }

  // Mounting
  componentDidMount() {
    // Chạy sau khi component mount — gọi API, setup subscription
    fetchData().then(data => this.setState({ data }));
  }

  // Updating
  componentDidUpdate(prevProps, prevState) {
    // Chạy sau mỗi render (trừ lần đầu)
    if (prevProps.id !== this.props.id) {
      fetchData(this.props.id).then(data => this.setState({ data }));
    }
  }

  // Unmounting
  componentWillUnmount() {
    // Cleanup — cancel subscription, clear timer
    this.subscription.unsubscribe();
  }

  render() {
    return <div>{this.state.data}</div>;
  }
}
```

**Hooks tương đương:**

```jsx
function MyComponent({ id }) {
  const [data, setData] = useState(null);

  // componentDidMount + componentDidUpdate + componentWillUnmount
  useEffect(() => {
    // Mounting & Updating — chạy khi id thay đổi
    fetchData(id).then(setData);

    // componentWillUnmount — cleanup function
    return () => {
      // cleanup
    };
  }, [id]); // dependency array = [] → chỉ chạy 1 lần (mount)
             //                = [id] → chạy khi id thay đổi
             // không có [] → chạy mỗi render

  return <div>{data}</div>;
}
```

**Bảng so sánh:**

| Lifecycle Method | Hook Equivalent |
|------------------|-----------------|
| `constructor` | `useState` initial value |
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [deps])` |
| `componentWillUnmount` | `useEffect(() => { return cleanup }, [])` |
| `shouldComponentUpdate` | `React.memo` + `useMemo` |
| `getDerivedStateFromProps` | Tính toán trong render hoặc `useMemo` |
| `getSnapshotBeforeUpdate` | `useLayoutEffect` với ref |

**Lưu ý quan trọng:** Không có hook tương đương hoàn toàn với `getSnapshotBeforeUpdate` — đây là trường hợp hiếm cần class component.

---

## 4. Virtual DOM

**[Mid]**

**EN:** How does the Virtual DOM work? Explain the diffing algorithm and React Fiber architecture.

**VI:** Virtual DOM hoạt động như thế nào? Giải thích thuật toán diffing và kiến trúc React Fiber.

### Trả lời

**Virtual DOM** là một biểu diễn nhẹ (lightweight representation) của DOM thật, được lưu trong bộ nhớ dưới dạng cây JavaScript object. Thay vì thao tác trực tiếp với DOM (chậm), React:

1. Render ra Virtual DOM mới khi state/props thay đổi
2. So sánh (diff) Virtual DOM mới với cái cũ
3. Chỉ update những phần DOM thật thực sự thay đổi (patch/reconcile)

**Thuật toán Diffing (Reconciliation):**

React sử dụng hai heuristic để giảm độ phức tạp từ O(n³) xuống O(n):

**Heuristic 1 — Elements of different types:** Nếu type thay đổi, React destroy cả subtree và build lại từ đầu.

```jsx
// Trước
<div>
  <Counter />
</div>

// Sau — type thay đổi từ div sang span → Counter bị unmount và remount
<span>
  <Counter />
</span>
```

**Heuristic 2 — Keys:** Dùng `key` để nhận diện elements trong danh sách.

```jsx
// React dùng key để match elements giữa renders
<ul>
  <li key="1">Alice</li>  // → vẫn là Alice
  <li key="2">Bob</li>    // → vẫn là Bob
  <li key="3">Charlie</li>// → mới thêm vào
</ul>
```

**React Fiber Architecture:**

Fiber (React 16+) là bộ reconciliation engine mới thay thế stack-based reconciler cũ.

**Vấn đề của reconciler cũ:** Là synchronous — một khi bắt đầu render, không thể dừng lại → gây jank trên UI với cây component lớn.

**Fiber giải quyết bằng cách:**
- Chia công việc render thành các đơn vị nhỏ gọi là **"fiber nodes"** (1 fiber = 1 component instance)
- Có thể **pause, resume, abort** công việc render
- Assign **priority** cho các update khác nhau
- Hỗ trợ **Concurrent Mode** — render nhiều versions của UI song song

```
Fiber Tree Structure:
App (fiber)
├── Header (fiber)
│   └── Nav (fiber)
└── Main (fiber)
    ├── Sidebar (fiber)
    └── Content (fiber)
        └── List (fiber)
            ├── Item key=1 (fiber)
            └── Item key=2 (fiber)
```

**Work Loop:** Fiber renderer có hai phase:
- **Render phase** (interruptible): Tính toán thay đổi, tạo work-in-progress tree. Có thể bị interrupt bởi higher-priority work.
- **Commit phase** (synchronous): Apply thay đổi vào DOM thật. Không thể bị interrupt — phải hoàn thành để UI consistent.

---

## 5. useState vs useReducer

**[Mid]**

**EN:** When should you use `useState` vs `useReducer`? Explain the reducer pattern.

**VI:** Khi nào dùng `useState` và khi nào dùng `useReducer`? Giải thích reducer pattern.

### Trả lời

**`useState`** phù hợp cho:
- State đơn giản, độc lập
- State là primitive (string, number, boolean)
- Logic update đơn giản

**`useReducer`** phù hợp cho:
- State phức tạp, có nhiều sub-values liên quan nhau
- Next state phụ thuộc vào previous state
- Logic update phức tạp cần tập trung ở một chỗ
- Dễ test hơn (reducer là pure function)

```jsx
// useState — phù hợp cho đơn giản
function SimpleCounter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// useReducer — phù hợp cho phức tạp
const initialState = {
  items: [],
  loading: false,
  error: null,
  total: 0,
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const newItems = [...state.items, action.payload];
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, item) => sum + item.price, 0),
      };
    case 'REMOVE_ITEM':
      const filtered = state.items.filter(item => item.id !== action.payload);
      return {
        ...state,
        items: filtered,
        total: filtered.reduce((sum, item) => sum + item.price, 0),
      };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload, loading: false };
    case 'CLEAR_CART':
      return { ...initialState };
    default:
      return state;
  }
}

function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  const addItem = (item) => dispatch({ type: 'ADD_ITEM', payload: item });
  const removeItem = (id) => dispatch({ type: 'REMOVE_ITEM', payload: id });

  return (
    <div>
      {state.loading && <Spinner />}
      {state.error && <ErrorMessage message={state.error} />}
      <p>Total: ${state.total}</p>
      {state.items.map(item => (
        <CartItem key={item.id} item={item} onRemove={removeItem} />
      ))}
    </div>
  );
}
```

**Kết hợp với Context để thay thế Redux đơn giản:**

```jsx
const CartContext = createContext(null);

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}

function useCart() {
  const context = useContext(CartContext);
  if (!context) throw new Error('useCart must be used within CartProvider');
  return context;
}
```

---

## 6. useEffect

**[Mid → Senior]**

**EN:** Explain `useEffect` in depth: dependency array behavior, cleanup function, and common mistakes (stale closure, async in useEffect, object deps causing infinite loop).

**VI:** Giải thích `useEffect` chi tiết: dependency array, cleanup function, và các lỗi thường gặp (stale closure, async trong useEffect, object deps gây infinite loop).

### Trả lời

**`useEffect`** là hook để thực hiện side effects sau khi render: fetch data, subscriptions, DOM manipulation, timers.

**Dependency Array behavior:**

```jsx
// Chạy sau MỖI render
useEffect(() => { console.log('every render'); });

// Chạy CHỈ sau lần render đầu (mount)
useEffect(() => { console.log('mount only'); }, []);

// Chạy khi userId thay đổi (và lần đầu)
useEffect(() => { console.log('userId changed'); }, [userId]);
```

**Cleanup function:**

```jsx
useEffect(() => {
  const subscription = eventEmitter.subscribe('event', handler);
  const timer = setInterval(tick, 1000);

  // Cleanup — chạy trước khi effect chạy lại, và khi unmount
  return () => {
    subscription.unsubscribe();
    clearInterval(timer);
  };
}, []);
```

---

**Lỗi 1: Stale Closure**

```jsx
// BUG — count luôn là 0 trong callback vì closure capture giá trị cũ
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count); // Luôn log 0!
      setCount(count + 1); // Luôn set 1!
    }, 1000);
    return () => clearInterval(interval);
  }, []); // [] → closure capture count = 0 mãi mãi
}

// FIX 1 — dùng functional update
useEffect(() => {
  const interval = setInterval(() => {
    setCount(c => c + 1); // Không phụ thuộc vào closure
  }, 1000);
  return () => clearInterval(interval);
}, []);

// FIX 2 — thêm count vào deps (effect re-run mỗi khi count thay đổi)
useEffect(() => {
  const interval = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(interval);
}, [count]);
```

---

**Lỗi 2: Async trong useEffect**

```jsx
// BUG — useEffect callback không được là async function
useEffect(async () => { // ❌ trả về Promise, không phải cleanup function
  const data = await fetchData();
  setData(data);
}, []);

// FIX 1 — define async function bên trong và gọi nó
useEffect(() => {
  async function loadData() {
    const data = await fetchData();
    setData(data);
  }
  loadData();
}, []);

// FIX 2 — handle race condition với cleanup flag
useEffect(() => {
  let cancelled = false;

  async function loadData() {
    const data = await fetchData(id);
    if (!cancelled) setData(data); // Không set state nếu đã cleanup
  }

  loadData();
  return () => { cancelled = true; };
}, [id]);

// FIX 3 — dùng AbortController (recommended)
useEffect(() => {
  const controller = new AbortController();

  async function loadData() {
    try {
      const res = await fetch(`/api/data/${id}`, { signal: controller.signal });
      const data = await res.json();
      setData(data);
    } catch (err) {
      if (err.name !== 'AbortError') setError(err);
    }
  }

  loadData();
  return () => controller.abort();
}, [id]);
```

---

**Lỗi 3: Object/Array deps gây Infinite Loop**

```jsx
// BUG — object mới tạo mỗi render → reference luôn khác → infinite loop
function Component({ userId }) {
  const options = { method: 'GET', headers: { Authorization: 'Bearer token' } };
  //              ↑ object literal mới mỗi render!

  useEffect(() => {
    fetch(`/api/users/${userId}`, options);
  }, [options]); // options luôn "thay đổi" → infinite loop
}

// FIX 1 — move object ra ngoài component (nếu không phụ thuộc props/state)
const OPTIONS = { method: 'GET', headers: { Authorization: 'Bearer token' } };

function Component({ userId }) {
  useEffect(() => {
    fetch(`/api/users/${userId}`, OPTIONS);
  }, [userId]); // Chỉ deps cần thiết
}

// FIX 2 — dùng useMemo để stable reference
function Component({ userId, token }) {
  const options = useMemo(
    () => ({ method: 'GET', headers: { Authorization: `Bearer ${token}` } }),
    [token]
  );

  useEffect(() => {
    fetch(`/api/users/${userId}`, options);
  }, [userId, options]);
}

// FIX 3 — inline object, chỉ deps relevant values
function Component({ userId, token }) {
  useEffect(() => {
    fetch(`/api/users/${userId}`, {
      method: 'GET',
      headers: { Authorization: `Bearer ${token}` },
    });
  }, [userId, token]); // Primitive values — safe
}
```

---

## 7. useMemo vs useCallback

**[Mid → Senior]**

**EN:** Explain `useMemo` and `useCallback`. When are they actually needed? What are the anti-patterns? How do they work with `React.memo`?

**VI:** Giải thích `useMemo` và `useCallback`. Khi nào thực sự cần dùng? Anti-patterns là gì? Kết hợp với `React.memo` như thế nào?

### Trả lời

**`useMemo`** — memoize **giá trị** (result của computation):

```jsx
const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);
```

**`useCallback`** — memoize **function** (stable reference):

```jsx
const memoizedFn = useCallback(() => doSomething(a, b), [a, b]);
// Tương đương: useMemo(() => () => doSomething(a, b), [a, b])
```

---

**Khi nào THỰC SỰ cần:**

```jsx
// 1. useMemo — computation thực sự tốn kém
function DataTable({ rows, filters }) {
  // Lọc/sort hàng nghìn records — nên memoize
  const filteredRows = useMemo(
    () => rows
      .filter(row => filters.status ? row.status === filters.status : true)
      .filter(row => filters.search
        ? row.name.toLowerCase().includes(filters.search.toLowerCase())
        : true
      )
      .sort((a, b) => a.name.localeCompare(b.name)),
    [rows, filters.status, filters.search]
  );

  return <Table rows={filteredRows} />;
}

// 2. useCallback — stabilize function reference cho child component dùng React.memo
const MemoizedChild = React.memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click me</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  // Nếu không có useCallback → MemoizedChild re-render mỗi lần Parent render
  // Vì onClick là function mới mỗi lần → props thay đổi → memo không có tác dụng
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // stable reference

  return <MemoizedChild onClick={handleClick} />;
}
```

---

**Anti-patterns — ĐỪNG làm:**

```jsx
// ❌ Anti-pattern 1: useMemo cho computation đơn giản
// Overhead của useMemo > lợi ích
const doubled = useMemo(() => count * 2, [count]); // Không cần!
const doubled = count * 2; // ✅ Đơn giản hơn

// ❌ Anti-pattern 2: useCallback không kèm React.memo
// useCallback không giúp ích gì nếu child không dùng React.memo
function Parent() {
  const handleClick = useCallback(() => console.log('clicked'), []);
  return <Child onClick={handleClick} />; // Child không phải React.memo → vẫn re-render
}

// ❌ Anti-pattern 3: memoize mọi thứ "phòng ngừa"
// Premature optimization — thêm complexity, không giúp ích
function Component({ name }) {
  const greeting = useMemo(() => `Hello, ${name}!`, [name]); // Không cần
  return <div>{greeting}</div>;
}

// ✅ Đúng approach: đo lường trước (React DevTools Profiler), tối ưu sau
```

---

**Pattern kết hợp React.memo + useCallback + useMemo:**

```jsx
// Scenario: Parent re-render thường xuyên, Child render tốn kém
const ExpensiveChild = React.memo(function ExpensiveChild({
  data,
  onSelect,
  formatter,
}) {
  const processed = useMemo(() => formatter(data), [formatter, data]);
  return (
    <div onClick={() => onSelect(data.id)}>
      {processed}
    </div>
  );
});

function Parent({ rawData, filter }) {
  const [selected, setSelected] = useState(null);

  // Stable references → ExpensiveChild không re-render khi Parent re-render
  // trừ khi rawData hoặc filter thay đổi
  const filteredData = useMemo(
    () => rawData.filter(d => d.category === filter),
    [rawData, filter]
  );

  const handleSelect = useCallback((id) => {
    setSelected(id);
  }, []);

  const formatter = useCallback((item) => {
    return `${item.name} — $${item.price.toFixed(2)}`;
  }, []);

  return (
    <div>
      {filteredData.map(item => (
        <ExpensiveChild
          key={item.id}
          data={item}
          onSelect={handleSelect}
          formatter={formatter}
        />
      ))}
    </div>
  );
}
```

---

## 8. React.memo

**[Mid]**

**EN:** How does `React.memo` work? When does it help vs hurt performance?

**VI:** `React.memo` hoạt động như thế nào? Khi nào nó giúp ích và khi nào gây hại?

### Trả lời

`React.memo` là Higher Order Component bọc một functional component và thực hiện **shallow comparison** của props. Nếu props không thay đổi → React skip re-render component đó.

```jsx
// Không có React.memo — re-render mỗi khi parent re-render
function ProductCard({ product }) {
  console.log('ProductCard render');
  return <div>{product.name} — ${product.price}</div>;
}

// Với React.memo — chỉ re-render khi product thay đổi (shallow comparison)
const ProductCard = React.memo(function ProductCard({ product }) {
  console.log('ProductCard render');
  return <div>{product.name} — ${product.price}</div>;
});

// Custom comparison function — kiểm soát khi nào re-render
const ProductCard = React.memo(
  function ProductCard({ product }) {
    return <div>{product.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true → SKIP re-render (props "equal")
    // Return false → DO re-render (props "not equal")
    return prevProps.product.id === nextProps.product.id &&
           prevProps.product.price === nextProps.product.price;
  }
);
```

**Shallow comparison nghĩa là gì?**

```jsx
// Primitives — so sánh giá trị → hoạt động đúng
'hello' === 'hello' // true → skip re-render ✅

// Objects/Arrays — so sánh reference, KHÔNG phải deep equality
{ name: 'Alice' } === { name: 'Alice' } // false → re-render dù giống nhau ❌

// Ví dụ thực tế gây bug
function Parent() {
  const [count, setCount] = useState(0);
  // Object literal mới mỗi render → React.memo không có tác dụng!
  const style = { color: 'red' };
  return <MemoizedChild style={style} />;
}
```

**Khi nào React.memo GIÚP ÍCH:**
- Component render tốn kém (danh sách lớn, chart, heavy computation)
- Props thực sự ít thay đổi
- Parent re-render thường xuyên vì lý do không liên quan đến child

**Khi nào React.memo KHÔNG GIÚP / GÂY HẠI:**
- Props luôn thay đổi (như object/array/function literals)
- Component nhẹ, render nhanh → overhead của comparison > lợi ích
- Component luôn cần re-render khi parent thay đổi

---

## 9. Custom Hooks

**[Mid → Senior]**

**EN:** What are custom hooks? Walk through patterns: `useAsync`, `useDebounce`, `useLocalStorage`, `useIntersectionObserver`, `useForm`.

**VI:** Custom hooks là gì? Trình bày các pattern: `useAsync`, `useDebounce`, `useLocalStorage`, `useIntersectionObserver`, `useForm`.

### Trả lời

**Custom hooks** là các function JavaScript bắt đầu bằng `use` và có thể gọi các hooks khác bên trong. Chúng cho phép tái sử dụng stateful logic giữa các components.

```jsx
// useAsync — quản lý async operations
function useAsync(asyncFn, deps = []) {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null,
  });

  useEffect(() => {
    let cancelled = false;
    setState(s => ({ ...s, loading: true, error: null }));

    asyncFn()
      .then(data => {
        if (!cancelled) setState({ data, loading: false, error: null });
      })
      .catch(error => {
        if (!cancelled) setState({ data: null, loading: false, error });
      });

    return () => { cancelled = true; };
  }, deps); // eslint-disable-line react-hooks/exhaustive-deps

  return state;
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useAsync(
    () => fetchUser(userId),
    [userId]
  );

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  return <div>{user.name}</div>;
}
```

```jsx
// useDebounce — debounce value để giảm API calls
function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage — search input
function SearchBar() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) searchAPI(debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

```jsx
// useLocalStorage — sync state với localStorage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// Usage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return (
    <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

```jsx
// useIntersectionObserver — lazy load, infinite scroll, animation on scroll
function useIntersectionObserver(ref, options = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    if (!ref.current) return;

    const observer = new IntersectionObserver(
      ([entry]) => setIsIntersecting(entry.isIntersecting),
      { threshold: 0, rootMargin: '0px', ...options }
    );

    observer.observe(ref.current);
    return () => observer.disconnect();
  }, [ref, options.threshold, options.rootMargin]);

  return isIntersecting;
}

// Usage — lazy load image
function LazyImage({ src, alt }) {
  const ref = useRef(null);
  const isVisible = useIntersectionObserver(ref, { threshold: 0.1 });

  return (
    <div ref={ref}>
      {isVisible ? <img src={src} alt={alt} /> : <Skeleton />}
    </div>
  );
}
```

```jsx
// useForm — form state management
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setValues(v => ({ ...v, [name]: value }));
  }, []);

  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(t => ({ ...t, [name]: true }));
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
    }
  }, [values, validate]);

  const handleSubmit = useCallback((onSubmit) => async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    const validationErrors = validate ? validate(values) : {};
    setErrors(validationErrors);
    setTouched(Object.keys(initialValues).reduce((acc, key) => ({ ...acc, [key]: true }), {}));

    if (Object.keys(validationErrors).length === 0) {
      await onSubmit(values);
    }
    setIsSubmitting(false);
  }, [values, validate, initialValues]);

  return { values, errors, touched, isSubmitting, handleChange, handleBlur, handleSubmit };
}

// Usage
function LoginForm() {
  const { values, errors, touched, handleChange, handleBlur, handleSubmit } = useForm(
    { email: '', password: '' },
    (vals) => {
      const errs = {};
      if (!vals.email) errs.email = 'Email required';
      if (vals.password.length < 8) errs.password = 'Min 8 characters';
      return errs;
    }
  );

  return (
    <form onSubmit={handleSubmit(async (data) => await login(data))}>
      <input name="email" value={values.email} onChange={handleChange} onBlur={handleBlur} />
      {touched.email && errors.email && <span>{errors.email}</span>}
      <input name="password" type="password" value={values.password} onChange={handleChange} onBlur={handleBlur} />
      {touched.password && errors.password && <span>{errors.password}</span>}
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## 10. Context API

**[Mid → Senior]**

**EN:** Explain the Context API provider pattern. What are the performance pitfalls and how do you fix re-renders (split context, useMemo, separate state/dispatch)?

**VI:** Giải thích Context API provider pattern. Các vấn đề về performance là gì và cách fix re-render?

### Trả lời

**Context API** cho phép truyền data qua component tree mà không cần prop drilling.

```jsx
// Basic pattern
const ThemeContext = createContext('light');

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  return useContext(ThemeContext);
}
```

---

**Performance Pitfall — Mọi consumer re-render khi value thay đổi:**

```jsx
// BUG — mỗi khi user hoặc setUser thay đổi, TẤT CẢ consumers re-render
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [theme, setTheme] = useState('light');

  // Object mới mỗi render → tất cả consumers re-render!
  return (
    <AppContext.Provider value={{ user, setUser, posts, setPosts, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}
```

---

**FIX 1 — Split Context theo domain:**

```jsx
// Tách thành nhiều context nhỏ — consumer chỉ re-render khi context nó subscribe thay đổi
const UserContext = createContext(null);
const PostsContext = createContext(null);
const ThemeContext = createContext(null);

function AppProvider({ children }) {
  return (
    <ThemeProvider>
      <UserProvider>
        <PostsProvider>
          {children}
        </PostsProvider>
      </UserProvider>
    </ThemeProvider>
  );
}
```

---

**FIX 2 — Tách State và Dispatch context:**

```jsx
// State thay đổi thường xuyên, dispatch không bao giờ thay đổi
// → components chỉ cần dispatch không bị re-render khi state thay đổi
const UserStateContext = createContext(null);
const UserDispatchContext = createContext(null);

function UserProvider({ children }) {
  const [state, dispatch] = useReducer(userReducer, initialState);

  return (
    <UserStateContext.Provider value={state}>
      <UserDispatchContext.Provider value={dispatch}>
        {children}
      </UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
}

// Component chỉ cần dispatch — không re-render khi user data thay đổi
function LogoutButton() {
  const dispatch = useContext(UserDispatchContext); // Stable reference!
  return <button onClick={() => dispatch({ type: 'LOGOUT' })}>Logout</button>;
}

// Component cần state — chỉ re-render khi user data thay đổi
function UserAvatar() {
  const { user } = useContext(UserStateContext);
  return <img src={user?.avatar} />;
}
```

---

**FIX 3 — useMemo để stable value:**

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  // Không tạo object mới mỗi render trừ khi theme thay đổi
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

**Khi nào KHÔNG nên dùng Context:**
- Data thay đổi rất thường xuyên (như mouse position) → dùng pub/sub hoặc Zustand/Jotai
- Nhiều components khác nhau cần subscribe → xem xét state management library
- Context lồng nhau quá nhiều → khó debug

---

## 11. useRef

**[Junior → Mid]**

**EN:** Explain `useRef`. What's the difference between using it as a DOM ref vs a mutable value? What is `forwardRef`?

**VI:** Giải thích `useRef`. Sự khác biệt giữa dùng nó như DOM ref và mutable value? `forwardRef` là gì?

### Trả lời

`useRef` trả về một object `{ current: initialValue }`. Giá trị `.current` có thể thay đổi mà **không gây re-render**.

**Dùng 1: DOM ref**

```jsx
function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus(); // Truy cập DOM node trực tiếp
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

// Các use case DOM ref phổ biến:
// - Focus management
// - Trigger animations
// - Integrate với third-party DOM libraries (D3, Chart.js)
// - Đo kích thước element (getBoundingClientRect)
// - Scroll to element
```

**Dùng 2: Mutable value (không trigger re-render)**

```jsx
// Lưu previous value
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current; // Giá trị từ render trước
}

// Lưu stable reference không cần re-render
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null); // Lưu interval ID

  const start = () => {
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };

  const stop = () => {
    clearInterval(intervalRef.current);
  };

  // Track mount status để tránh setState sau unmount
  const isMountedRef = useRef(true);
  useEffect(() => {
    return () => { isMountedRef.current = false; };
  }, []);
}
```

**forwardRef — truyền ref từ parent xuống DOM element bên trong custom component:**

```jsx
// Vấn đề: ref không tự động truyền qua props như các prop thường
function Parent() {
  const ref = useRef(null);
  return <CustomInput ref={ref} />; // ref KHÔNG tự động work
}

// FIX: dùng forwardRef
const CustomInput = forwardRef(function CustomInput({ label, ...props }, ref) {
  return (
    <div>
      <label>{label}</label>
      <input ref={ref} {...props} /> {/* ref được forward xuống đây */}
    </div>
  );
});

// Bây giờ parent có thể truy cập DOM input
function Parent() {
  const inputRef = useRef(null);
  return (
    <>
      <CustomInput label="Name" ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus Input</button>
    </>
  );
}

// useImperativeHandle — expose API có chọn lọc thay vì toàn bộ DOM node
const FancyInput = forwardRef(function FancyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ''; },
    // Chỉ expose những method cần thiết, không expose toàn bộ DOM node
  }));

  return <input ref={inputRef} {...props} />;
});
```

---

## 12. Controlled vs Uncontrolled

**[Junior]**

**EN:** What is the difference between controlled and uncontrolled components?

**VI:** Sự khác biệt giữa controlled và uncontrolled components là gì?

### Trả lời

| | Controlled | Uncontrolled |
|--|------------|--------------|
| Data source | React state | DOM (ref) |
| Cập nhật | Qua setState | DOM tự quản lý |
| Truy cập value | Từ state | Qua `ref.current.value` |
| Validation | Real-time, dễ | Submit-time, khó hơn |
| Reset | `setValues({})` | `ref.current.value = ''` |

```jsx
// Controlled — React là "single source of truth"
function ControlledForm() {
  const [name, setName] = useState('');

  return (
    <form onSubmit={() => console.log(name)}>
      <input
        value={name}           // React kiểm soát value
        onChange={e => setName(e.target.value)}
      />
    </form>
  );
}

// Uncontrolled — DOM tự quản lý, React chỉ đọc khi cần
function UncontrolledForm() {
  const nameRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(nameRef.current.value); // Đọc khi submit
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={nameRef}
        defaultValue="Alice" // Initial value, không controlled
      />
    </form>
  );
}
```

**Khi nào dùng Uncontrolled:**
- Form đơn giản chỉ cần value lúc submit
- File inputs (`<input type="file">`) — luôn phải là uncontrolled
- Tích hợp với non-React code
- Performance: form có rất nhiều fields

**Khi nào dùng Controlled:**
- Cần real-time validation
- Cần conditional rendering dựa trên input value
- Cần instant feedback
- Cần enforce format (phone number, currency)

---

## 13. React.lazy + Suspense

**[Mid]**

**EN:** How does `React.lazy` work with `Suspense`? Explain code splitting and loading boundaries.

**VI:** `React.lazy` hoạt động với `Suspense` như thế nào? Giải thích code splitting và loading boundaries.

### Trả lời

**Code splitting** là kỹ thuật chia bundle JavaScript thành các chunk nhỏ hơn, chỉ load khi cần — giảm thời gian tải trang ban đầu.

```jsx
// Không có code splitting — tất cả được bundle vào 1 file
import HeavyDashboard from './HeavyDashboard';
import AdminPanel from './AdminPanel';
import Analytics from './Analytics';

// Với React.lazy — mỗi component là 1 chunk riêng, load khi cần
const HeavyDashboard = React.lazy(() => import('./HeavyDashboard'));
const AdminPanel = React.lazy(() => import('./AdminPanel'));
const Analytics = React.lazy(() => import('./Analytics'));

// Suspense bọc lazy components và hiển thị fallback khi đang load
function App() {
  return (
    <Router>
      <Suspense fallback={<PageSpinner />}>
        <Routes>
          <Route path="/dashboard" element={<HeavyDashboard />} />
          <Route path="/admin" element={<AdminPanel />} />
          <Route path="/analytics" element={<Analytics />} />
        </Routes>
      </Suspense>
    </Router>
  );
}
```

**Multiple Suspense boundaries — loading granularity:**

```jsx
// Coarse-grained: Toàn bộ trang load cùng lúc
function Page() {
  return (
    <Suspense fallback={<FullPageSpinner />}>
      <Header />
      <MainContent />
      <Sidebar />
    </Suspense>
  );
}

// Fine-grained: Từng section load độc lập
function Page() {
  return (
    <>
      <Header /> {/* Load ngay */}
      <Suspense fallback={<ContentSkeleton />}>
        <MainContent />
      </Suspense>
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
    </>
  );
}
```

**Preloading — load trước khi cần:**

```jsx
// Preload khi user hover vào button
const AdminPanel = React.lazy(() => import('./AdminPanel'));

function NavLink() {
  const preload = () => import('./AdminPanel'); // Trigger load ngay

  return (
    <Link to="/admin" onMouseEnter={preload}>
      Admin
    </Link>
  );
}
```

**Kết hợp với Error Boundary:**

```jsx
function LazyPage({ path }) {
  const Component = React.lazy(() => import(path));

  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Spinner />}>
        <Component />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## 14. Error Boundaries

**[Mid]**

**EN:** What are Error Boundaries? How do you implement them with the class component pattern and the `react-error-boundary` library?

**VI:** Error Boundaries là gì? Cách implement với class component và thư viện `react-error-boundary`?

### Trả lời

**Error Boundary** là component bắt JavaScript errors trong component tree con, log chúng, và hiển thị fallback UI thay vì crash toàn bộ app.

**Lưu ý:** Error boundaries chỉ bắt được errors trong:
- Render method
- Lifecycle methods
- Constructor

**KHÔNG bắt được:** event handlers, async code, server-side rendering, errors trong chính error boundary.

```jsx
// Class component — cách duy nhất implement Error Boundary (không có hook tương đương)
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    // Cập nhật state để render fallback UI
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log error sang reporting service
    console.error('Error caught:', error, errorInfo);
    logErrorToService(error, errorInfo.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div>
          <h2>Something went wrong.</h2>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary fallback={<div>App crashed!</div>}>
      <Router>
        <Routes>
          <Route path="/" element={
            <ErrorBoundary fallback={<div>Home page error</div>}>
              <HomePage />
            </ErrorBoundary>
          } />
        </Routes>
      </Router>
    </ErrorBoundary>
  );
}
```

**react-error-boundary — library (recommended):**

```jsx
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary';

// Simple fallback component
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre style={{ color: 'red' }}>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// Usage với onError callback
function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => logToSentry(error, info)}
      onReset={() => {
        // Reset application state khi user click "Try again"
        queryClient.clear();
      }}
    >
      <MainApp />
    </ErrorBoundary>
  );
}

// useErrorBoundary hook — throw error từ async code vào boundary gần nhất
function DataFetcher({ id }) {
  const { showBoundary } = useErrorBoundary();

  useEffect(() => {
    fetchData(id)
      .then(setData)
      .catch(error => showBoundary(error)); // Throw vào Error Boundary
  }, [id]);
}

// withErrorBoundary HOC
const SafeComponent = withErrorBoundary(RiskyComponent, {
  FallbackComponent: ErrorFallback,
});
```

---

## 15. React 18

**[Mid → Senior]**

**EN:** What are the new features in React 18? Explain `useTransition`, `useDeferredValue`, `useId`, and automatic batching.

**VI:** Các tính năng mới trong React 18 là gì? Giải thích `useTransition`, `useDeferredValue`, `useId`, và automatic batching.

### Trả lời

**1. Automatic Batching:**

```jsx
// React 17 — chỉ batch updates trong event handlers
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 17: 2 lần re-render
  // React 18: 1 lần re-render (automatic batching)
}, 1000);

// Để opt-out batching trong React 18 (hiếm khi cần)
import { flushSync } from 'react-dom';

flushSync(() => setCount(c => c + 1)); // Render ngay
flushSync(() => setFlag(f => !f));     // Render ngay
```

**2. useTransition — mark updates là non-urgent:**

```jsx
function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    // Urgent — cập nhật input ngay lập tức
    setQuery(e.target.value);

    // Non-urgent — có thể bị interrupt nếu user tiếp tục gõ
    startTransition(() => {
      const filtered = heavySearch(allData, e.target.value);
      setResults(filtered);
    });
  };

  return (
    <>
      <input value={query} onChange={handleSearch} />
      {isPending ? <Spinner /> : <ResultsList results={results} />}
    </>
  );
}
```

**3. useDeferredValue — defer rendering của expensive child:**

```jsx
function App() {
  const [text, setText] = useState('');
  // deferredText "lags behind" text — cho phép input luôn responsive
  const deferredText = useDeferredValue(text);

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      {/* SlowList chỉ re-render với deferredText, không phải mỗi keystroke */}
      <SlowList text={deferredText} />
    </>
  );
}

// useDeferredValue vs useTransition:
// - useTransition: bạn kiểm soát code nào là "urgent" vs "non-urgent"
// - useDeferredValue: defer một VALUE từ bên ngoài (prop hoặc không kiểm soát được setter)
```

**4. useId — generate stable unique IDs:**

```jsx
// Vấn đề: ID không đồng nhất giữa server và client render (SSR hydration mismatch)
// BAD
function Input({ label }) {
  const id = Math.random(); // Khác nhau mỗi render ❌
  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} />
    </>
  );
}

// GOOD — useId đảm bảo stable, unique ID, consistent giữa server/client
function Input({ label }) {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} />
    </>
  );
}

// Có thể tạo nhiều IDs liên quan từ 1 useId call
function PasswordField() {
  const id = useId();
  return (
    <>
      <label htmlFor={`${id}-input`}>Password</label>
      <input id={`${id}-input`} type="password" aria-describedby={`${id}-hint`} />
      <p id={`${id}-hint`}>Must be at least 8 characters</p>
    </>
  );
}
```

---

## 16. Portals

**[Mid]**

**EN:** What are React Portals? What are their use cases (Modal, Tooltip, Dropdown)?

**VI:** React Portals là gì? Các use cases phổ biến (Modal, Tooltip, Dropdown)?

### Trả lời

**Portals** cho phép render children vào một DOM node nằm ngoài hierarchy của parent component — nhưng vẫn giữ nguyên React context và event bubbling.

```jsx
// Cú pháp cơ bản
ReactDOM.createPortal(child, domNode)

// Modal với Portal — render vào document.body thay vì nơi gọi
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  return ReactDOM.createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <button className="modal-close" onClick={onClose}>×</button>
        {children}
      </div>
    </div>,
    document.body // Render vào body, tránh z-index/overflow issues
  );
}

// Usage — Modal nằm trong DOM body nhưng events vẫn bubble lên App
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div style={{ overflow: 'hidden' }}> {/* overflow: hidden không ảnh hưởng Modal */}
      <button onClick={() => setShowModal(true)}>Open Modal</button>
      <Modal isOpen={showModal} onClose={() => setShowModal(false)}>
        <h2>Modal Content</h2>
        <p>This renders in document.body!</p>
      </Modal>
    </div>
  );
}
```

**Tooltip với Portal:**

```jsx
function Tooltip({ children, content }) {
  const [visible, setVisible] = useState(false);
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const triggerRef = useRef(null);

  const showTooltip = () => {
    const rect = triggerRef.current.getBoundingClientRect();
    setPosition({ top: rect.bottom + 8, left: rect.left });
    setVisible(true);
  };

  return (
    <>
      <span
        ref={triggerRef}
        onMouseEnter={showTooltip}
        onMouseLeave={() => setVisible(false)}
      >
        {children}
      </span>
      {visible && ReactDOM.createPortal(
        <div
          style={{
            position: 'fixed',
            top: position.top,
            left: position.left,
            background: '#333',
            color: '#fff',
            padding: '4px 8px',
            borderRadius: 4,
            zIndex: 9999,
          }}
        >
          {content}
        </div>,
        document.body
      )}
    </>
  );
}
```

**Tại sao cần Portal?**
- `overflow: hidden` trên parent cắt mất dropdown/tooltip
- `z-index` stacking context gây modal bị che khuất
- CSS transform trên parent làm `position: fixed` không hoạt động đúng
- Portal giải quyết tất cả vì nó render ở DOM level khác

---

## 17. HOC vs Render Props vs Hooks

**[Mid → Senior]**

**EN:** Explain the evolution from Higher Order Components to Render Props to Hooks. What are the trade-offs?

**VI:** Giải thích sự tiến hóa từ Higher Order Components → Render Props → Hooks. Các đánh đổi là gì?

### Trả lời

Cả 3 là patterns để **tái sử dụng logic** trong React.

**1. Higher Order Components (HOC) — circa 2015-2017:**

```jsx
// HOC — function nhận component, trả về component mới với thêm props/logic
function withUser(WrappedComponent) {
  return function WithUserComponent(props) {
    const [user, setUser] = useState(null);

    useEffect(() => {
      fetchCurrentUser().then(setUser);
    }, []);

    return <WrappedComponent user={user} {...props} />;
  };
}

// Usage
const UserProfile = withUser(Profile);
const UserDashboard = withUser(Dashboard);

// Vấn đề:
// - Wrapper hell (nhiều HOC lồng nhau → khó debug)
// - Props collision (2 HOC cùng inject prop cùng tên)
// - Không rõ props đến từ HOC nào trong DevTools
```

**2. Render Props — circa 2017-2018:**

```jsx
// Render Props — component nhận function là prop, gọi function đó trong render
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };

  handleMouseMove = (e) => {
    this.setState({ x: e.clientX, y: e.clientY });
  };

  render() {
    return (
      <div onMouseMove={this.handleMouseMove}>
        {/* Gọi render prop với data */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Usage
<MouseTracker render={({ x, y }) => (
  <div>Mouse: {x}, {y}</div>
)} />

// Vấn đề:
// - Callback hell khi nhiều render props lồng nhau
// - Verbose, khó đọc
// - Performance: function mới mỗi render
```

**3. Custom Hooks — từ React 16.8 (2019):**

```jsx
// Hooks — share logic mà không thêm component nodes vào tree
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handler = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
  }, []);

  return position;
}

// Usage — clean, flat, dễ compose
function Component() {
  const { x, y } = useMousePosition();
  return <div>Mouse: {x}, {y}</div>;
}
```

**So sánh:**

| | HOC | Render Props | Hooks |
|--|-----|--------------|-------|
| Wrapper hell | Có | Có | Không |
| Props collision | Có | Không | Không |
| Compose | Khó | Khó | Dễ (gọi nhiều hooks) |
| Debug | Khó | Khó | Dễ hơn |
| Static typing | Khó | OK | Tốt |
| Dùng bên ngoài render | Không | Không | Có |

**Khi nào vẫn dùng HOC:**
- Cần wrap component với additional DOM structure
- Tích hợp với code cũ dùng HOC pattern
- Class components (không dùng hooks được)

---

## 18. Performance Optimization

**[Senior]**

**EN:** What are the key React performance optimization techniques? How do you profile and identify bottlenecks?

**VI:** Các kỹ thuật tối ưu performance React quan trọng là gì? Cách profiling và tìm bottlenecks?

### Trả lời

**Profiling với React DevTools:**

```jsx
// Bật Profiler trong React DevTools:
// 1. Mở DevTools → tab "Profiler"
// 2. Click record → interact với app → stop
// 3. Xem flamegraph: màu cam/đỏ = render chậm
// 4. Xem "Ranked chart" để thấy component nào tốn thời gian nhất

// Programmatic profiling
import { Profiler } from 'react';

function onRenderCallback(
  id,          // "id" prop của Profiler
  phase,       // "mount" hoặc "update"
  actualDuration,   // Thời gian render commit
  baseDuration,     // Thời gian nếu không memoize
  startTime,
  commitTime
) {
  console.log({ id, phase, actualDuration });
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <MainContent />
    </Profiler>
  );
}
```

**Các kỹ thuật tối ưu chính:**

```jsx
// 1. Tránh re-render không cần thiết
const MemoizedList = React.memo(ExpensiveList);
const stableCallback = useCallback(fn, [deps]);
const expensiveValue = useMemo(() => compute(data), [data]);

// 2. Virtualize danh sách dài (react-window / react-virtual)
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}

// 3. Code splitting theo route và component
const Dashboard = React.lazy(() => import('./Dashboard'));

// 4. Tránh prop drilling với Context (nhưng cẩn thận với performance)
// → dùng Zustand/Jotai cho global state thay đổi thường

// 5. State colocation — giữ state gần nơi dùng nhất
// BAD: tất cả state ở App root → toàn bộ app re-render
// GOOD: state trong component cần nó

// 6. Tránh anonymous functions và object literals trong JSX
// BAD
<Component onClick={() => handleClick(id)} style={{ color: 'red' }} />
// GOOD
const handleClickId = useCallback(() => handleClick(id), [id]);
const style = useMemo(() => ({ color: 'red' }), []);
<Component onClick={handleClickId} style={style} />

// 7. Batch state updates (React 18 automatic, hoặc unstable_batchedUpdates trong React 17)
// 8. Lazy initial state
const [state] = useState(() => expensiveComputation()); // Chỉ chạy 1 lần
```

**Checklist performance:**
1. Dùng React DevTools Profiler để tìm bottleneck trước khi tối ưu
2. Kiểm tra renders không cần thiết với "Highlight updates" trong DevTools
3. Bundle size: dùng `webpack-bundle-analyzer` hoặc `next-bundle-analyzer`
4. Lazy load routes và heavy components
5. Virtualize danh sách > 100 items
6. Tối ưu images (WebP, lazy loading, responsive sizes)
7. Dùng Web Workers cho CPU-intensive tasks

---

## 19. Strict Mode

**[Mid]**

**EN:** What does `React.StrictMode` do? Why does it double-invoke certain functions?

**VI:** `React.StrictMode` làm gì? Tại sao nó gọi một số function hai lần?

### Trả lời

`React.StrictMode` là tool để phát hiện potential problems trong app. **Chỉ chạy trong development mode**, không ảnh hưởng production build.

```jsx
// Enable StrictMode
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

**Điều StrictMode làm:**

**1. Double-invoke các function để phát hiện side effects:**

React gọi 2 lần:
- Function component body (render)
- `useState`, `useMemo`, `useReducer` initializers
- `setState` updater functions
- `getDerivedStateFromProps`

```jsx
// Nếu component này log 2 lần trong dev → bình thường với StrictMode
function MyComponent() {
  console.log('render'); // Log 2 lần trong dev, 1 lần trong prod

  const [state] = useState(() => {
    console.log('init'); // Log 2 lần trong dev
    return expensiveCompute();
  });
}
```

**Tại sao double-invoke?** Để phát hiện side effects trong render phase. Render phase phải pure — không được có side effects (vì Concurrent Mode có thể render nhiều lần trước khi commit).

```jsx
// BUG — side effect trong render phase
function BadComponent() {
  // Nếu StrictMode double-invoke, count sẽ là 2 thay vì 1 → phát hiện bug
  count++; // ❌ Side effect trong render
  return <div>{count}</div>;
}
```

**2. Warn về deprecated APIs:**
- `findDOMNode`
- Legacy string refs
- Legacy Context API
- `componentWillMount`, `componentWillUpdate`, `componentWillReceiveProps`

**3. Detect unexpected side effects trong effects:**

Từ React 18, StrictMode mount → unmount → remount components trong dev để đảm bảo effects có cleanup đúng:

```jsx
// StrictMode sẽ chạy: mount → cleanup → mount
// Để test cleanup function hoạt động đúng
useEffect(() => {
  const sub = subscribe(channel);
  return () => sub.unsubscribe(); // Phải có cleanup
}, [channel]);
```

---

## 20. Concurrent Mode

**[Senior]**

**EN:** What does Concurrent Mode enable? Explain `startTransition` in the context of concurrent rendering.

**VI:** Concurrent Mode cho phép gì? Giải thích `startTransition` trong context của concurrent rendering.

### Trả lời

**Concurrent Mode** là tập hợp các tính năng cho phép React render nhiều versions của UI cùng lúc, pause và resume work, và prioritize updates.

**Trước Concurrent Mode (Legacy/Blocking Mode):**
- Render là synchronous — một khi bắt đầu, không thể dừng
- Long renders chặn browser → UI bị jank
- Không thể ưu tiên updates quan trọng (user input) hơn updates ít quan trọng (data fetch)

**Với Concurrent Mode (React 18 default với `createRoot`):**

```jsx
// Enable Concurrent Mode
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);

// startTransition — đánh dấu update là "non-urgent"
import { startTransition, useTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      // Non-urgent: React có thể interrupt render này
      // nếu user click tab khác trước khi render xong
      setTab(nextTab);
    });
  }

  return (
    <>
      <TabButton onClick={() => selectTab('about')}>About</TabButton>
      <TabButton onClick={() => selectTab('posts')}>Posts (slow)</TabButton>
      <TabButton onClick={() => selectTab('contact')}>Contact</TabButton>

      {/* isPending: React đang render version mới trong background */}
      <Suspense fallback={<Spinner />}>
        <div style={{ opacity: isPending ? 0.6 : 1 }}>
          {tab === 'about' && <AboutTab />}
          {tab === 'posts' && <PostsTab />} {/* Expensive component */}
          {tab === 'contact' && <ContactTab />}
        </div>
      </Suspense>
    </>
  );
}
```

**Key concurrent features:**

| Feature | Mô tả |
|---------|-------|
| `startTransition` | Mark update là non-urgent, có thể interrupt |
| `useDeferredValue` | Defer render của expensive subtree |
| Automatic batching | Batch tất cả updates (không chỉ event handlers) |
| Suspense for data | Suspend component khi data chưa ready |
| Streaming SSR | Stream HTML từ server, hydrate từng phần |

**Concurrent rendering flow:**

```
User types in input:
1. React bắt đầu render (Transition — non-urgent)
2. User gõ thêm → high-priority input update
3. React DỪNG transition render
4. Render input update (urgent) trước
5. Tiếp tục transition render từ đầu với giá trị mới
```

**Điều quan trọng:** Transitions phải pure — React có thể render chúng nhiều lần. Không đặt side effects bên trong `startTransition`.

---

---

# REACT 19

---

### 21. [Mid] React 19 — Tổng quan các thay đổi lớn

**EN:** What are the major new features in React 19?

**VI:** Các tính năng lớn mới trong React 19 là gì?

**Trả lời:**

React 19 released chính thức tháng 12/2024. Các thay đổi lớn nhất:

| Tính năng | Mô tả |
|---|---|
| **Actions** | Async transitions — xử lý pending/error/optimistic tự động |
| **useActionState** | Hook quản lý state của form action |
| **useFormStatus** | Hook đọc trạng thái của form đang submit |
| **useOptimistic** | Optimistic UI updates trong async transitions |
| **`use()` hook** | Đọc Promise và Context trong render, kể cả trong điều kiện |
| **Server Actions** | Async functions chạy trên server, gọi từ client |
| **ref as prop** | Không cần `forwardRef` nữa, ref là prop bình thường |
| **ref cleanup** | Return cleanup function từ ref callback |
| **Document Metadata** | `<title>`, `<meta>` trực tiếp trong component |
| **Asset preloading** | `preloadFont`, `preinit`, `prefetchDNS` APIs |
| **Error handling** | `onCaughtError`, `onUncaughtError` trên root |
| **Hydration** | Smarter recovery, không re-render toàn bộ subtree khi mismatch |

---

### 22. [Mid] Actions và useActionState

**EN:** What are Actions in React 19? How does `useActionState` work?

**VI:** Actions trong React 19 là gì? `useActionState` hoạt động như thế nào?

**Trả lời:**

**Actions** là async functions được truyền vào transitions. React tự động quản lý pending state, error handling, và optimistic updates.

```tsx
// TRƯỚC React 19 — tự quản lý pending/error:
function OldForm() {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleSubmit(e: FormEvent) {
    e.preventDefault();
    setIsPending(true);
    setError(null);
    try {
      await submitData(formData);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  }
}

// SAU React 19 — useActionState tự lo:
import { useActionState } from "react";

async function submitAction(prevState: State, formData: FormData) {
  const name = formData.get("name") as string;

  if (!name) {
    return { error: "Tên không được để trống", data: null };
  }

  try {
    const result = await saveUser({ name });
    return { error: null, data: result };
  } catch (err) {
    return { error: err.message, data: null };
  }
}

function NewForm() {
  const [state, action, isPending] = useActionState(submitAction, {
    error: null,
    data: null
  });
  // state     — kết quả return từ submitAction
  // action    — function truyền vào form action prop
  // isPending — true khi đang submit

  return (
    <form action={action}>
      <input name="name" />
      {state.error && <p style={{ color: "red" }}>{state.error}</p>}
      {state.data && <p>Đã lưu: {state.data.name}</p>}
      <button type="submit" disabled={isPending}>
        {isPending ? "Đang lưu..." : "Lưu"}
      </button>
    </form>
  );
}
```

**Lưu ý:** `useActionState` đổi tên từ `useFormState` (React DOM) — import từ `"react"` thay vì `"react-dom"`.

---

### 23. [Mid] useFormStatus

**EN:** What is `useFormStatus`? How is it different from `isPending` in `useActionState`?

**VI:** `useFormStatus` là gì? Khác `isPending` của `useActionState` như thế nào?

**Trả lời:**

`useFormStatus` đọc trạng thái của `<form>` **cha gần nhất** — không cần pass props xuống.

```tsx
import { useFormStatus } from "react-dom";

// SubmitButton nằm BÊN TRONG form — tự biết form đang submit hay không
function SubmitButton({ label }: { label: string }) {
  const { pending, data, method, action } = useFormStatus();
  // pending — form cha đang submit
  // data    — FormData đang được submit
  // method  — "get" | "post"
  // action  — URL hoặc function

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Đang xử lý..." : label}
    </button>
  );
}

function LoginForm() {
  const [state, action] = useActionState(loginAction, null);

  return (
    <form action={action}>
      <input name="email" type="email" />
      <input name="password" type="password" />
      {state?.error && <p>{state.error}</p>}
      <SubmitButton label="Đăng nhập" /> {/* không cần truyền isPending qua prop */}
    </form>
  );
}

// SO SÁNH:
// useActionState isPending — trạng thái của action function đó
// useFormStatus pending    — trạng thái của form CHA gần nhất
//
// useFormStatus hữu ích khi SubmitButton là component tách biệt,
// không muốn drill props xuống nhiều tầng
```

---

### 24. [Mid] useOptimistic

**EN:** How does `useOptimistic` work? When should you use it?

**VI:** `useOptimistic` hoạt động như thế nào? Khi nào nên dùng?

**Trả lời:**

`useOptimistic` hiển thị **optimistic state** ngay lập tức khi async action đang chạy. Khi action xong, tự revert về giá trị thực hoặc giữ lại nếu thành công.

```tsx
import { useOptimistic, useActionState } from "react";

interface Message {
  id: string;
  text: string;
  sending?: boolean;
}

function MessageThread({ messages }: { messages: Message[] }) {
  // useOptimistic(actualState, updateFn)
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (currentMessages: Message[], newText: string) => [
      ...currentMessages,
      { id: crypto.randomUUID(), text: newText, sending: true }
    ]
  );

  const [, sendAction, isPending] = useActionState(
    async (_: unknown, formData: FormData) => {
      const text = formData.get("text") as string;
      addOptimisticMessage(text);  // hiển thị ngay
      await sendMessage(text);     // gửi server
    },
    null
  );

  return (
    <div>
      <ul>
        {optimisticMessages.map(msg => (
          <li key={msg.id} style={{ opacity: msg.sending ? 0.6 : 1 }}>
            {msg.text}
            {msg.sending && <span> (đang gửi...)</span>}
          </li>
        ))}
      </ul>
      <form action={sendAction}>
        <input name="text" placeholder="Nhắn tin..." />
        <button type="submit" disabled={isPending}>Gửi</button>
      </form>
    </div>
  );
}

// FLOW:
// 1. User submit → addOptimisticMessage chạy ngay → message mờ xuất hiện
// 2. sendMessage() gửi lên server
// 3a. Thành công → server data thay thế optimistic state
// 3b. Thất bại   → optimistic tự động revert về messages cũ
```

---

### 25. [Senior] `use()` Hook

**EN:** What is the `use()` hook in React 19? How is it different from `useContext` and `await`?

**VI:** Hook `use()` trong React 19 là gì? Khác `useContext` và `await` như thế nào?

**Trả lời:**

`use()` là hook có thể đọc **Promise** và **Context** ngay trong render. Điểm đặc biệt: **có thể gọi trong điều kiện (if/loop)** — các hooks khác không làm được.

```tsx
import { use, Suspense } from "react";

// ===== Dùng với Context =====
const ThemeContext = createContext<"light" | "dark">("light");

// Có thể gọi trong điều kiện — useContext không làm được điều này:
function ConditionalTheme({ showThemed }: { showThemed: boolean }) {
  if (showThemed) {
    const theme = use(ThemeContext); // ✅ OK trong if
    return <ThemedComponent theme={theme} />;
  }
  return <DefaultComponent />;
}

// ===== Dùng với Promise — tích hợp với Suspense =====
function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);
  // Chưa resolve → suspend → Suspense fallback hiển thị
  // Reject       → Error Boundary bắt
  return <div>{user.name}</div>;
}

// Parent tạo promise và pass xuống (không await):
function UserPage({ id }: { id: string }) {
  const userPromise = fetchUser(id); // tạo promise, không await

  return (
    <ErrorBoundary fallback={<p>Lỗi tải user</p>}>
      <Suspense fallback={<Skeleton />}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}

// ===== SO SÁNH =====
// useContext(ctx)         — chỉ Context, không dùng trong điều kiện
// use(ctx)                — Context, CÓ THỂ dùng trong điều kiện
// use(promise)            — đọc Promise, suspend khi chưa resolve
// await trong Server Comp — chỉ dùng được trong Server Component (Next.js)
```

---

### 26. [Senior] Server Actions

**EN:** What are Server Actions in React 19? How do they work with forms?

**VI:** Server Actions trong React 19 là gì? Chúng hoạt động với forms như thế nào?

**Trả lời:**

Server Actions là async functions chạy **trên server**, gọi được từ client components — không cần tạo API route thủ công.

```tsx
// ===== Định nghĩa Server Action =====
// app/actions/user.ts
"use server"; // directive này biến file/function thành server-only

export async function createUser(formData: FormData) {
  const name = formData.get("name") as string;
  const email = formData.get("email") as string;

  // Chạy trên SERVER — access DB trực tiếp:
  const user = await db.user.create({ data: { name, email } });

  revalidatePath("/users"); // invalidate cache
  return { success: true, user };
}

// ===== Dùng với HTML form (progressive enhancement) =====
// Hoạt động dù JavaScript bị disabled!
function CreateUserForm() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Tạo</button>
    </form>
  );
}

// ===== Dùng với useActionState để có state feedback =====
function CreateUserFormEnhanced() {
  const [state, action, isPending] = useActionState(createUser, null);

  return (
    <form action={action}>
      <input name="name" />
      <input name="email" type="email" />
      {state?.error && <p>{state.error}</p>}
      <button disabled={isPending}>
        {isPending ? "Đang tạo..." : "Tạo"}
      </button>
    </form>
  );
}

// ===== Gọi từ event handler =====
"use client";
import { deleteUser } from "@/actions/user";

function DeleteButton({ userId }: { userId: string }) {
  return <button onClick={() => deleteUser(userId)}>Xóa</button>;
}

// ===== Closure trong Server Action =====
async function ProductPage({ product }: { product: Product }) {
  async function addToCart() {
    "use server";
    await db.cart.add({ productId: product.id }); // capture từ closure
    revalidatePath("/cart");
  }

  return (
    <form action={addToCart}>
      <button type="submit">Thêm vào giỏ</button>
    </form>
  );
}

// ⚠️ SECURITY: Server Actions bị expose như public API endpoints
// → Luôn validate input và kiểm tra authorization trên server
```

---

### 27. [Mid] ref as prop — Bỏ forwardRef

**EN:** How did ref handling change in React 19?

**VI:** Cách xử lý ref thay đổi như thế nào trong React 19?

**Trả lời:**

```tsx
// TRƯỚC React 19 — bắt buộc forwardRef:
const OldInput = forwardRef<HTMLInputElement, InputProps>(
  function Input({ label, ...props }, ref) {
    return (
      <label>
        {label}
        <input ref={ref} {...props} />
      </label>
    );
  }
);

// SAU React 19 — ref là prop bình thường:
function NewInput({ label, ref, ...props }: InputProps & { ref?: Ref<HTMLInputElement> }) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}

// Dùng y chang nhau:
const inputRef = useRef<HTMLInputElement>(null);
<NewInput ref={inputRef} label="Tên" />

// CLEANUP FUNCTION cho ref (mới trong React 19):
function MeasureDiv() {
  return (
    <div
      ref={(node) => {
        if (!node) return;

        const observer = new ResizeObserver(handleResize);
        observer.observe(node);

        // Return cleanup — chạy khi unmount, thay thế useEffect + useRef:
        return () => observer.disconnect();
      }}
    />
  );
}

// forwardRef vẫn hoạt động (deprecated, chưa bị xóa)
// Nên migrate dần sang ref-as-prop pattern
```

---

### 28. [Mid] Document Metadata và Asset Preloading

**EN:** How does React 19 handle `<title>`, `<meta>` tags and asset preloading?

**VI:** React 19 xử lý `<title>`, `<meta>` và asset preloading như thế nào?

**Trả lời:**

```tsx
// Trước đây cần react-helmet hoặc next/head
// React 19 — built-in, tự động hoist lên <head>:

function ProductPage({ product }: { product: Product }) {
  return (
    <div>
      <title>{product.name} | Shop</title>
      <meta name="description" content={product.description} />
      <meta property="og:title" content={product.name} />
      <meta property="og:image" content={product.imageUrl} />
      <link rel="canonical" href={`https://shop.com/products/${product.id}`} />

      <h1>{product.name}</h1>
    </div>
  );
}

// Stylesheet async loading:
function BlogPost({ post }: { post: Post }) {
  return (
    <div>
      {/* Load stylesheet cho component này, không block render */}
      <link rel="stylesheet" href="/css/blog.css" precedence="default" />

      <article>{post.content}</article>
    </div>
  );
}

// Resource hints API:
import { prefetchDNS, preconnect, preload, preinit } from "react-dom";

function App() {
  preconnect("https://api.example.com");           // kết nối trước
  prefetchDNS("https://cdn.example.com");          // DNS lookup trước
  preload("/fonts/main.woff2", { as: "font" });    // preload font
  preinit("/scripts/analytics.js", { as: "script" }); // load + execute script

  return <div>...</div>;
}
// React tự deduplicate — gọi nhiều lần vẫn chỉ tạo 1 tag
```

---

### 29. [Senior] React 18 vs React 19 — So sánh tổng quan

**EN:** Summarize the key differences between React 18 and React 19.

**VI:** Tóm tắt sự khác biệt chính giữa React 18 và React 19.

**Trả lời:**

| Tính năng | React 18 | React 19 |
|---|---|---|
| Form handling | Tự quản lý isPending/error | `useActionState`, `useFormStatus` |
| Optimistic UI | Tự implement với useState | `useOptimistic` built-in |
| Promise in render | Không hỗ trợ trực tiếp | `use(promise)` + Suspense |
| Context in conditions | ❌ `useContext` không được | ✅ `use(ctx)` được |
| ref forwarding | Bắt buộc `forwardRef` | ref là prop thường |
| ref cleanup | ❌ Không có | ✅ Return cleanup function |
| Document metadata | Cần `next/head` / `react-helmet` | Built-in `<title>`, `<meta>` |
| Asset preloading | Manual `<link rel="preload">` | `preload`, `preinit`, `preconnect` APIs |
| Server Actions | Chỉ trong Next.js framework | React core spec |
| Error handling | `onRecoverableError` | + `onCaughtError`, `onUncaughtError` |
| Hydration mismatch | Re-render toàn bộ subtree | Smarter recovery |
| Error log | Duplicate messages | Single clear message |
| `useFormState` | `react-dom` | Đổi thành `useActionState` từ `react` |
| `ReactDOM.render` | Deprecated | Bị xóa hoàn toàn |

**Migration:**

```bash
npm install react@19 react-dom@19
```

```tsx
// Các breaking changes cần sửa:

// 1. ReactDOM.render đã bị xóa:
// ❌ ReactDOM.render(<App />, root);
// ✅ createRoot(root).render(<App />);

// 2. useFormState → useActionState:
// ❌ import { useFormState } from "react-dom";
// ✅ import { useActionState } from "react";

// 3. PropTypes đã bị xóa khỏi React package:
// ❌ import PropTypes from "react"; // không còn
// ✅ npm install prop-types (nếu vẫn cần)
```

---

*Tài liệu này được cập nhật cho React 19 (released 12/2024). Câu hỏi được sắp xếp từ Junior đến Senior theo từng topic.*
