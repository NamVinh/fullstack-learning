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
21. [React 19 — Tổng quan các thay đổi lớn](#21-react-19)
22. [Actions và useActionState](#22-actions-useactionstate)
23. [useFormStatus](#23-useformstatus)
24. [useOptimistic](#24-useoptimistic)
25. [`use()` Hook](#25-use-hook)
26. [Server Actions](#26-server-actions)
27. [ref as prop — Bỏ forwardRef](#27-ref-as-prop)
28. [Document Metadata và Asset Preloading](#28-document-metadata)
29. [React 18 vs React 19 — So sánh tổng quan](#29-react-18-vs-react-19)
30. [useLayoutEffect vs useEffect — Timing Difference](#30-uselayouteffect-vs-useeffect)
31. [Fragments — Grouping Without Extra DOM Nodes](#31-fragments)
32. [Conditional Rendering — Patterns and the && Gotcha](#32-conditional-rendering)
33. [Event Handling — Synthetic Events and Event Delegation](#33-event-handling)
34. [Compound Component Pattern — Context-Based Implicit State](#34-compound-component-pattern)
35. [Hydration — SSR to Client Handoff](#35-hydration)
36. [useSyncExternalStore — Subscribing to External Stores](#36-usesyncexternalstore)

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is the difference between `useState` and `useReducer`?"**

**Model Answer:**
"Both manage component state, but they suit different levels of complexity.

`useState` is the simple option — it gives you a value and a setter function. Perfect for a toggle, a counter, a form field:
```jsx
const [isOpen, setIsOpen] = useState(false);
```

`useReducer` is better when state has multiple related pieces that change together, or when the next state depends on the previous one in a complex way. You define a reducer — a pure function — that handles all the transitions:
```jsx
const [state, dispatch] = useReducer(cartReducer, initialState);
dispatch({ type: 'ADD_ITEM', payload: item });
```

My rule of thumb: if I find myself writing three or four related `useState` calls and the updates need to stay in sync, that's the signal to reach for `useReducer`."

**Trả lời (Tiếng Việt):**
"Cả hai đều dùng để quản lý state của component, nhưng phù hợp với các mức độ phức tạp khác nhau.

`useState` là lựa chọn đơn giản — nó trả về một giá trị và một hàm setter. Phù hợp cho toggle, counter, hay một ô input:
```jsx
const [isOpen, setIsOpen] = useState(false);
```

`useReducer` tốt hơn khi state có nhiều phần liên quan thay đổi cùng nhau, hoặc khi state tiếp theo phụ thuộc vào state trước theo cách phức tạp. Bạn định nghĩa một reducer — một pure function — xử lý tất cả các transition:
```jsx
const [state, dispatch] = useReducer(cartReducer, initialState);
dispatch({ type: 'ADD_ITEM', payload: item });
```

Nguyên tắc của mình: nếu thấy mình viết ba hay bốn `useState` liên quan và các lần cập nhật cần đồng bộ với nhau, đó là dấu hiệu nên chuyển sang dùng `useReducer`."

---

**Q2 (Mid): "When would you choose `useReducer` over `useState`? Give a real example."**

**Model Answer:**
"I reach for `useReducer` in a few situations.

The clearest case is a shopping cart or any form with multiple related fields — items, loading state, error state, and a computed total all need to update together based on the same actions. With `useState` I'd have four separate setters and I'd have to coordinate them carefully. With `useReducer`, each action type handles all the related mutations atomically:

```jsx
case 'ADD_ITEM':
  const newItems = [...state.items, action.payload];
  return { ...state, items: newItems, total: computeTotal(newItems) };
```

Another big reason: testability. A reducer is a pure function — I can test every state transition just by calling it with an input and checking the output, no component setup needed.

And the third reason is when I'm combining it with Context to replace a simple Redux store. The `dispatch` function from `useReducer` has a stable identity, so I can put it in a separate context from the state — components that only dispatch never re-render from state changes."

**Trả lời (Tiếng Việt):**
"Mình chọn `useReducer` trong một vài tình huống cụ thể.

Trường hợp rõ ràng nhất là giỏ hàng hoặc bất kỳ form nào có nhiều field liên quan — items, loading state, error state, và tổng tiền đều cần cập nhật cùng nhau dựa trên các action. Với `useState` mình sẽ có bốn setter riêng lẻ và phải phối hợp chúng cẩn thận. Với `useReducer`, mỗi action type xử lý tất cả các mutation liên quan một cách nguyên tử:

```jsx
case 'ADD_ITEM':
  const newItems = [...state.items, action.payload];
  return { ...state, items: newItems, total: computeTotal(newItems) };
```

Lý do lớn khác: khả năng test. Reducer là pure function — mình có thể test mọi state transition chỉ bằng cách gọi nó với input và kiểm tra output, không cần setup component.

Và lý do thứ ba là khi kết hợp với Context để thay thế một Redux store đơn giản. Hàm `dispatch` từ `useReducer` có identity ổn định, nên mình có thể đặt nó vào một context riêng với state — các component chỉ dispatch sẽ không bao giờ re-render khi state thay đổi."

---

**Q3 (Mid): "Can `useReducer` replace Redux entirely?"**

**Model Answer:**
"`useReducer` plus Context can replace Redux for simpler applications, yes. You get the same action-dispatch pattern, and combining it with Context means you don't need an external library.

But Redux still has advantages at scale: the Redux DevTools time-travel debugger is excellent, middleware like redux-thunk or redux-saga handles complex async flows cleanly, and there are mature patterns for things like normalized state and selectors with reselect.

With `useReducer` + Context there's also a performance concern — every time the context value changes, all consumers re-render unless you split state and dispatch into separate contexts. Redux's `useSelector` is more granular because it only re-renders a component when the specific slice it subscribes to changes.

So my answer is: for a mid-sized app without heavy async orchestration, `useReducer` + Context is fine and keeps dependencies lean. For a large app with lots of shared state and complex async logic, I'd still consider Zustand or Redux Toolkit."

**Trả lời (Tiếng Việt):**
"`useReducer` kết hợp Context có thể thay thế Redux cho những ứng dụng đơn giản hơn. Bạn có cùng pattern action-dispatch, và kết hợp với Context nghĩa là không cần thư viện ngoài.

Nhưng Redux vẫn có lợi thế khi ứng dụng lớn: Redux DevTools với tính năng time-travel debugging rất tốt, middleware như redux-thunk hay redux-saga xử lý async flow phức tạp rất sạch, và có nhiều pattern mature cho normalized state và selector với reselect.

Với `useReducer` + Context cũng có vấn đề performance — mỗi khi context value thay đổi, tất cả consumer đều re-render trừ khi bạn tách state và dispatch vào các context riêng. `useSelector` của Redux granular hơn vì nó chỉ re-render component khi slice cụ thể mà nó subscribe thay đổi.

Câu trả lời của mình: với app cỡ vừa không có async orchestration nặng, `useReducer` + Context ổn và giữ dependency gọn. Với app lớn có nhiều shared state và async logic phức tạp, mình vẫn cân nhắc Zustand hoặc Redux Toolkit."

---

**Q4 (Senior): "What is the `init` function (lazy initialization) in `useReducer` and when do you use it?"**

**Model Answer:**
"`useReducer` accepts an optional third argument — an `init` function — that computes the initial state lazily:

```jsx
function init(initialCount) {
  return { count: initialCount, step: 1 };
}

const [state, dispatch] = useReducer(reducer, initialCount, init);
```

Two reasons to use this.

First, if the initial state involves an expensive computation — like reading from localStorage or doing some heavy calculation — lazy initialization means it only runs once on mount, not on every render. Same benefit as the function form of `useState`.

Second, it's useful when you want to extract the initial state logic separately from the component, which makes it easier to reset state back to its initial value from inside the reducer:
```jsx
case 'RESET':
  return init(action.payload); // Reuse the same init logic
```

Without the `init` function, resetting state would mean duplicating the initial state definition."

**Trả lời (Tiếng Việt):**
"`useReducer` nhận một argument tùy chọn thứ ba — một hàm `init` — tính toán initial state một cách lazy:

```jsx
function init(initialCount) {
  return { count: initialCount, step: 1 };
}

const [state, dispatch] = useReducer(reducer, initialCount, init);
```

Có hai lý do để dùng cái này.

Thứ nhất, nếu initial state liên quan đến một phép tính tốn kém — như đọc từ localStorage hay làm một phép tính nặng — lazy initialization nghĩa là nó chỉ chạy một lần khi mount, không phải mỗi lần render. Giống với function form của `useState`.

Thứ hai, hữu ích khi bạn muốn tách logic initial state ra khỏi component, giúp dễ reset state về giá trị ban đầu từ bên trong reducer:
```jsx
case 'RESET':
  return init(action.payload); // Tái sử dụng logic init tương tự
```

Nếu không có hàm `init`, việc reset state sẽ phải duplicate định nghĩa initial state."

---

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

### 🎤 Mock Interview — useEffect Q&A

---

**Q1 (Junior): "What is `useEffect` and what can you use it for?"**

**Model Answer:**
"`useEffect` is a hook for performing side effects in function components. A side effect is anything that reaches outside of React's render flow — like fetching data from an API, setting up a WebSocket subscription, updating the document title, or starting a timer.

The key thing about `useEffect` is that it runs **after** the component renders — so it never blocks the browser from painting to the screen. You give it a function to run, and optionally an array of dependencies that tells React when to re-run it.

Most common uses I hit daily:
- Fetching data when a component mounts or a prop changes
- Subscribing to events (WebSocket, EventEmitter) with cleanup on unmount
- Syncing state to localStorage
- Integrating with third-party DOM libraries like chart.js"

**Trả lời (Tiếng Việt):**
"`useEffect` là hook để thực hiện side effect trong function component. Side effect là bất cứ thứ gì vươn ra ngoài luồng render của React — như fetch data từ API, setup WebSocket subscription, cập nhật document title, hay khởi động timer.

Điều quan trọng về `useEffect` là nó chạy **sau** khi component render — nên nó không bao giờ chặn browser vẽ lên màn hình. Bạn truyền cho nó một function để chạy, và tùy chọn một mảng dependencies cho React biết khi nào cần chạy lại.

Những use case phổ biến nhất mình dùng hằng ngày:
- Fetch data khi component mount hoặc prop thay đổi
- Subscribe vào event (WebSocket, EventEmitter) với cleanup khi unmount
- Sync state vào localStorage
- Tích hợp với thư viện DOM bên thứ ba như chart.js"

---

**Q2 (Mid): "Explain the dependency array in `useEffect`. What are the 3 modes?"**

**Model Answer:**
"There are 3 modes based on what you pass as the second argument:

**Mode 1 — No array: runs after every render**
```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}); // No deps array — runs after every single render
```
Rarely what you want. Can cause performance issues if the effect is expensive.

**Mode 2 — Empty array `[]`: runs once on mount**
```jsx
useEffect(() => {
  fetchInitialData();
  return () => cleanup(); // runs on unmount
}, []); // Empty array — runs only on mount
```
The equivalent of `componentDidMount`. Good for one-time setup.

**Mode 3 — With dependencies: runs when deps change**
```jsx
useEffect(() => {
  fetchUser(userId);
}, [userId]); // Re-runs whenever userId changes (and on mount)
```
Most common pattern. React compares deps with `Object.is()` — which is shallow equality. This is why objects and arrays as deps cause problems."

**Trả lời (Tiếng Việt):**
"Có 3 mode dựa vào những gì bạn truyền làm argument thứ hai:

**Mode 1 — Không có array: chạy sau mỗi lần render**
```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}); // Không có deps array — chạy sau mỗi lần render
```
Hiếm khi là thứ bạn muốn. Có thể gây vấn đề performance nếu effect tốn kém.

**Mode 2 — Array rỗng `[]`: chỉ chạy một lần khi mount**
```jsx
useEffect(() => {
  fetchInitialData();
  return () => cleanup(); // chạy khi unmount
}, []); // Array rỗng — chỉ chạy khi mount
```
Tương đương với `componentDidMount`. Tốt cho setup một lần.

**Mode 3 — Có dependencies: chạy khi deps thay đổi**
```jsx
useEffect(() => {
  fetchUser(userId);
}, [userId]); // Chạy lại mỗi khi userId thay đổi (và khi mount)
```
Pattern phổ biến nhất. React so sánh deps bằng `Object.is()` — là shallow equality. Đó là lý do objects và arrays làm deps gây ra vấn đề."

---

**Q3 (Mid): "What is a stale closure in `useEffect` and how do you fix it?"**

**Model Answer:**
"A stale closure happens when `useEffect` captures a variable from the component's render, but that variable later changes while the effect's closure still holds the old value.

Classic example:
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count); // STALE — always logs 0
      setCount(count + 1); // WRONG — always sets to 1
    }, 1000);
    return () => clearInterval(id);
  }, []); // Empty deps — closure captures count = 0 forever
}
```

**Fix 1 — Add `count` to deps** (but then interval restarts on every count change):
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]); // Restarts interval on every count change — not ideal
```

**Fix 2 — Use functional updater form** (best approach):
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(prev => prev + 1); // Reads current value, not stale closure
  }, 1000);
  return () => clearInterval(id);
}, []); // Empty deps is correct now — no stale closure
```

The functional form `setCount(prev => prev + 1)` always gets the latest state, so there's no stale closure issue."

**Trả lời (Tiếng Việt):**
"Stale closure xảy ra khi `useEffect` capture một biến từ render của component, nhưng biến đó sau đó thay đổi trong khi closure của effect vẫn giữ giá trị cũ.

Ví dụ kinh điển:
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count); // STALE — luôn log 0
      setCount(count + 1); // SAI — luôn set thành 1
    }, 1000);
    return () => clearInterval(id);
  }, []); // Deps rỗng — closure capture count = 0 mãi mãi
}
```

**Fix 1 — Thêm `count` vào deps** (nhưng interval sẽ restart mỗi lần count thay đổi):
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]); // Restart interval mỗi lần count thay đổi — không lý tưởng
```

**Fix 2 — Dùng functional updater form** (cách tốt nhất):
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(prev => prev + 1); // Đọc giá trị hiện tại, không phải stale closure
  }, 1000);
  return () => clearInterval(id);
}, []); // Deps rỗng là đúng rồi — không còn stale closure
```

Dạng functional `setCount(prev => prev + 1)` luôn lấy state mới nhất, nên không còn vấn đề stale closure nữa."

---

**Q4 (Senior): "How do you handle async operations in `useEffect`? What's the race condition problem?"**

**Model Answer:**
"You can't make `useEffect`'s callback `async` directly because React expects the cleanup function or `undefined` to be returned synchronously — not a Promise.

**Wrong approach:**
```jsx
useEffect(async () => {
  const data = await fetchUser(userId); // Works, but:
  setUser(data); // May cause memory leak if component unmounts before fetch completes
}, [userId]);
```

**Correct pattern:**
```jsx
useEffect(() => {
  let cancelled = false; // Cancellation flag

  async function loadUser() {
    const data = await fetchUser(userId);
    if (!cancelled) { // Only update state if component still mounted
      setUser(data);
    }
  }

  loadUser();

  return () => {
    cancelled = true; // Cleanup — cancel the pending operation
  };
}, [userId]);
```

**The race condition:** If `userId` changes from 1 to 2 quickly, two fetches start. If fetch for userId=2 completes before userId=1, the `cancelled` flag for the first fetch prevents stale data from overwriting the correct result.

In practice at Lumin PDF, for user document fetching I'd prefer TanStack Query which handles this automatically — but understanding the manual pattern shows you know the underlying problem."

**Trả lời (Tiếng Việt):**
"Bạn không thể làm callback của `useEffect` là `async` trực tiếp vì React expect cleanup function hoặc `undefined` được return đồng bộ — không phải Promise.

**Cách sai:**
```jsx
useEffect(async () => {
  const data = await fetchUser(userId); // Hoạt động, nhưng:
  setUser(data); // Có thể gây memory leak nếu component unmount trước khi fetch xong
}, [userId]);
```

**Pattern đúng:**
```jsx
useEffect(() => {
  let cancelled = false; // Cờ hủy

  async function loadUser() {
    const data = await fetchUser(userId);
    if (!cancelled) { // Chỉ cập nhật state nếu component vẫn còn mounted
      setUser(data);
    }
  }

  loadUser();

  return () => {
    cancelled = true; // Cleanup — hủy operation đang pending
  };
}, [userId]);
```

**Vấn đề race condition:** Nếu `userId` thay đổi từ 1 sang 2 nhanh, hai fetch cùng bắt đầu. Nếu fetch cho userId=2 hoàn thành trước userId=1, cờ `cancelled` của fetch đầu tiên ngăn data cũ ghi đè kết quả đúng.

Thực tế tại Lumin PDF, với việc fetch tài liệu người dùng mình sẽ ưu tiên TanStack Query vì nó xử lý việc này tự động — nhưng hiểu được pattern thủ công cho thấy bạn nắm được vấn đề cốt lõi."

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

### 🎤 Mock Interview — useMemo & useCallback Q&A

---

**Q1 (Mid): "What's the difference between `useMemo` and `useCallback`?"**

**Model Answer:**
"They both memoize things, but different things:
- `useMemo` memoizes a **value** — the result of a function call
- `useCallback` memoizes a **function** itself — its reference

```jsx
// useMemo — result is the memoized thing
const sortedList = useMemo(() => list.sort(compareFn), [list]);
// sortedList is an array (the computed result)

// useCallback — the function itself is the memoized thing
const handleClick = useCallback((id) => doSomething(id), []);
// handleClick is a function (stable reference across renders)
```

Actually, `useCallback(fn, deps)` is literally equivalent to `useMemo(() => fn, deps)` — just a more readable shorthand.

When to use which: `useMemo` for expensive computations (filtering, sorting, heavy math). `useCallback` for event handler functions that you pass to child components wrapped in `React.memo`."

**Trả lời (Tiếng Việt):**
"Cả hai đều memoize, nhưng memoize những thứ khác nhau:
- `useMemo` memoize một **giá trị** — kết quả của một function call
- `useCallback` memoize bản thân **function** — reference của nó

```jsx
// useMemo — kết quả là thứ được memoize
const sortedList = useMemo(() => list.sort(compareFn), [list]);
// sortedList là một array (kết quả tính toán)

// useCallback — bản thân function là thứ được memoize
const handleClick = useCallback((id) => doSomething(id), []);
// handleClick là một function (stable reference qua các lần render)
```

Thực ra, `useCallback(fn, deps)` hoàn toàn tương đương với `useMemo(() => fn, deps)` — chỉ là shorthand dễ đọc hơn.

Dùng cái nào: `useMemo` cho các tính toán tốn kém (filter, sort, heavy math). `useCallback` cho event handler function mà bạn truyền vào child component được wrap bởi `React.memo`."

---

**Q2 (Mid): "When should you NOT use `useMemo` and `useCallback`?"**

**Model Answer:**
"This is the more interesting question — most developers overuse them.

**Don't use `useMemo` for cheap computations:**
```jsx
// Anti-pattern — this memoization costs more than it saves
const doubled = useMemo(() => value * 2, [value]); // Just write value * 2!
```
The memoization itself has overhead: React needs to compare dependencies, store the previous value, and decide whether to re-run. For a simple multiplication, this is actually slower than just computing it.

**Don't use `useCallback` without `React.memo` on the child:**
```jsx
// Useless without React.memo — child re-renders anyway
function Parent() {
  const handler = useCallback(() => doSomething(), []); // Waste
  return <NormalChild onClick={handler} />; // NormalChild always re-renders
}
```
`useCallback` only helps if the child component uses `React.memo` — otherwise the stable function reference doesn't prevent re-renders.

**Rule of thumb:** Start without `useMemo`/`useCallback`. Add them only when you have a measured performance problem. Premature memoization makes code harder to read and can actually hurt performance for cheap operations."

**Trả lời (Tiếng Việt):**
"Đây là câu hỏi thú vị hơn — hầu hết developer lạm dụng chúng.

**Đừng dùng `useMemo` cho các tính toán đơn giản:**
```jsx
// Anti-pattern — memoization này tốn hơn lợi ích mang lại
const doubled = useMemo(() => value * 2, [value]); // Chỉ cần viết value * 2 thôi!
```
Bản thân việc memoize có overhead: React cần so sánh dependencies, lưu giá trị trước, và quyết định có chạy lại không. Với một phép nhân đơn giản, điều này thực ra còn chậm hơn tính trực tiếp.

**Đừng dùng `useCallback` nếu child không có `React.memo`:**
```jsx
// Vô nghĩa nếu không có React.memo — child vẫn re-render
function Parent() {
  const handler = useCallback(() => doSomething(), []); // Lãng phí
  return <NormalChild onClick={handler} />; // NormalChild luôn re-render
}
```
`useCallback` chỉ có tác dụng nếu child component dùng `React.memo` — nếu không thì stable function reference không ngăn được re-render.

**Nguyên tắc:** Bắt đầu không dùng `useMemo`/`useCallback`. Chỉ thêm vào khi bạn có vấn đề performance đã đo lường được. Premature memoization làm code khó đọc hơn và thực ra có thể làm chậm đối với các operation đơn giản."

---

**Q3 (Senior): "You have a component with 10,000 items that gets filtered. The filter is triggered by user typing in a search box. How do you optimize this?"**

**Model Answer:**
"Several layers of optimization:

**Layer 1 — `useMemo` for the filter computation:**
```jsx
const filteredItems = useMemo(
  () => items.filter(item =>
    item.name.toLowerCase().includes(searchTerm.toLowerCase())
  ),
  [items, searchTerm] // Only re-filter when items or searchTerm changes
);
```

**Layer 2 — Debounce the search input** to avoid filtering on every keystroke:
```jsx
const [inputValue, setInputValue] = useState('');
const debouncedSearch = useDebounce(inputValue, 300); // Custom hook

const filteredItems = useMemo(
  () => items.filter(item => item.name.includes(debouncedSearch)),
  [items, debouncedSearch]
);
```

**Layer 3 — Virtualize the list** if rendering 10k DOM nodes at once:
```jsx
import { FixedSizeList } from 'react-window';

<FixedSizeList height={600} itemCount={filteredItems.length} itemSize={50}>
  {({ index, style }) => (
    <div style={style}>
      <ItemRow item={filteredItems[index]} />
    </div>
  )}
</FixedSizeList>
```

**Layer 4 — `React.memo` on `ItemRow`** so individual rows don't re-render unless their item data changes.

At Lumin PDF, PDF page rendering is similar — you can't render 200 pages in the DOM. I'd use intersection observer to virtualize visible pages + `useMemo` for annotations per page."

**Trả lời (Tiếng Việt):**
"Nhiều lớp tối ưu:

**Lớp 1 — `useMemo` cho phép tính filter:**
```jsx
const filteredItems = useMemo(
  () => items.filter(item =>
    item.name.toLowerCase().includes(searchTerm.toLowerCase())
  ),
  [items, searchTerm] // Chỉ filter lại khi items hoặc searchTerm thay đổi
);
```

**Lớp 2 — Debounce search input** để tránh filter mỗi keystroke:
```jsx
const [inputValue, setInputValue] = useState('');
const debouncedSearch = useDebounce(inputValue, 300); // Custom hook

const filteredItems = useMemo(
  () => items.filter(item => item.name.includes(debouncedSearch)),
  [items, debouncedSearch]
);
```

**Lớp 3 — Virtualize danh sách** nếu render 10k DOM node cùng lúc:
```jsx
import { FixedSizeList } from 'react-window';

<FixedSizeList height={600} itemCount={filteredItems.length} itemSize={50}>
  {({ index, style }) => (
    <div style={style}>
      <ItemRow item={filteredItems[index]} />
    </div>
  )}
</FixedSizeList>
```

**Lớp 4 — `React.memo` trên `ItemRow`** để từng hàng không re-render trừ khi data của item đó thay đổi.

Tại Lumin PDF, render trang PDF cũng tương tự — bạn không thể render 200 trang vào DOM. Mình dùng intersection observer để virtualize các trang hiển thị + `useMemo` cho annotations theo từng trang."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is a custom hook and why would you create one?"**

**Model Answer:**
"A custom hook is just a JavaScript function whose name starts with `use` and that can call other hooks inside it. The naming convention is important — it's how React's linter knows to enforce the rules of hooks on it.

You create one when you have stateful logic that you want to reuse across multiple components. Instead of copy-pasting `useState` and `useEffect` into several places, you extract it into a hook once and use it everywhere.

A simple example: I had three different components that all needed to know if a value had changed since the previous render. I pulled that into a `usePrevious` hook:
```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; });
  return ref.current;
}
```
Now any component can call `usePrevious(someValue)` without knowing the implementation detail."

**Trả lời (Tiếng Việt):**
"Custom hook chỉ là một JavaScript function có tên bắt đầu bằng `use` và có thể gọi các hook khác bên trong. Convention đặt tên này quan trọng — đó là cách linter của React biết để áp dụng các rules of hooks lên nó.

Bạn tạo custom hook khi có stateful logic muốn tái sử dụng trên nhiều component. Thay vì copy-paste `useState` và `useEffect` vào nhiều chỗ, bạn trích xuất nó vào một hook một lần và dùng ở mọi nơi.

Một ví dụ đơn giản: mình có ba component khác nhau đều cần biết một giá trị có thay đổi kể từ lần render trước không. Mình tách nó thành hook `usePrevious`:
```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; });
  return ref.current;
}
```
Giờ bất kỳ component nào cũng có thể gọi `usePrevious(someValue)` mà không cần biết chi tiết implementation."

---

**Q2 (Mid): "Walk me through how you'd build a `useDebounce` hook."**

**Model Answer:**
"A debounce hook delays updating a value until the user stops changing it for a specified period. It's useful for search inputs — you don't want to fire an API call on every single keystroke.

The implementation uses `useState` to hold the debounced value and `useEffect` to manage a timer:

```jsx
function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer); // Cleanup cancels the previous timer
  }, [value, delay]);

  return debouncedValue;
}
```

The key insight is the cleanup function. Every time `value` changes, the effect cleans up the previous timer before setting a new one. So if the user types three characters quickly, only the timer for the last character actually fires — the earlier ones get cancelled.

Usage looks like this:
```jsx
const debouncedSearch = useDebounce(inputValue, 500);
useEffect(() => { if (debouncedSearch) fetchResults(debouncedSearch); }, [debouncedSearch]);
```"

**Trả lời (Tiếng Việt):**
"Debounce hook trì hoãn việc cập nhật giá trị cho đến khi người dùng ngừng thay đổi trong một khoảng thời gian nhất định. Hữu ích cho search input — bạn không muốn gọi API mỗi lần bấm phím.

Implementation dùng `useState` để giữ debounced value và `useEffect` để quản lý timer:

```jsx
function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer); // Cleanup hủy timer trước
  }, [value, delay]);

  return debouncedValue;
}
```

Insight quan trọng là cleanup function. Mỗi khi `value` thay đổi, effect dọn dẹp timer trước trước khi set cái mới. Vậy nếu người dùng gõ ba ký tự nhanh, chỉ timer của ký tự cuối mới thực sự chạy — những cái trước bị hủy.

Cách dùng trông như này:
```jsx
const debouncedSearch = useDebounce(inputValue, 500);
useEffect(() => { if (debouncedSearch) fetchResults(debouncedSearch); }, [debouncedSearch]);
```"

---

**Q3 (Mid): "What are the rules of hooks, and how do they apply to custom hooks?"**

**Model Answer:**
"There are two core rules:

**Rule 1 — Only call hooks at the top level.** Never inside loops, conditionals, or nested functions. This matters because React tracks hooks by their call order across renders — if a hook is sometimes skipped, the order changes and React loses track of which state belongs to which hook.

**Rule 2 — Only call hooks from React functions.** That means function components or other custom hooks — not regular JavaScript utility functions.

Custom hooks inherit both rules. Inside a custom hook I can call `useState`, `useEffect`, and other custom hooks, but they must always be at the top level of my hook, not conditionally.

The `eslint-plugin-react-hooks` package enforces both rules automatically. I always have it enabled — it catches mistakes like calling a hook inside a `if (condition)` block before they cause subtle runtime bugs."

**Trả lời (Tiếng Việt):**
"Có hai quy tắc cốt lõi:

**Quy tắc 1 — Chỉ gọi hook ở top level.** Không bao giờ bên trong loop, conditional, hay nested function. Điều này quan trọng vì React theo dõi hook theo thứ tự gọi qua các lần render — nếu một hook đôi khi bị bỏ qua, thứ tự thay đổi và React mất dấu state nào thuộc về hook nào.

**Quy tắc 2 — Chỉ gọi hook từ React function.** Tức là function component hoặc custom hook khác — không phải utility function JavaScript thông thường.

Custom hook kế thừa cả hai quy tắc. Bên trong custom hook mình có thể gọi `useState`, `useEffect`, và các custom hook khác, nhưng chúng phải luôn ở top level của hook, không phải có điều kiện.

Package `eslint-plugin-react-hooks` tự động enforce cả hai quy tắc. Mình luôn bật nó — nó bắt các lỗi như gọi hook bên trong `if (condition)` block trước khi chúng gây ra runtime bug khó phát hiện."

---

**Q4 (Senior): "How do you handle cleanup properly in a custom hook that wraps an async operation?"**

**Model Answer:**
"Cleanup is critical for async hooks because components can unmount before a fetch completes. If you update state on an unmounted component, React warns about a memory leak.

The pattern I use is a cancellation flag:

```jsx
function useAsync(asyncFn, deps) {
  const [state, setState] = useState({ data: null, loading: false, error: null });

  useEffect(() => {
    let cancelled = false;
    setState(s => ({ ...s, loading: true, error: null }));

    asyncFn()
      .then(data => { if (!cancelled) setState({ data, loading: false, error: null }); })
      .catch(error => { if (!cancelled) setState({ data: null, loading: false, error }); });

    return () => { cancelled = true; }; // Cleanup: prevent setState after unmount
  }, deps);

  return state;
}
```

For fetch calls specifically, I'd prefer `AbortController` because it actually cancels the network request rather than just ignoring the result:

```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal })
    .then(r => r.json())
    .then(setData)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [url]);
```

In practice for most projects I'd use TanStack Query instead of rolling this by hand, but understanding the cancellation pattern is what the library is doing under the hood."

**Trả lời (Tiếng Việt):**
"Cleanup rất quan trọng cho async hook vì component có thể unmount trước khi fetch hoàn thành. Nếu bạn cập nhật state trên component đã unmount, React cảnh báo về memory leak.

Pattern mình dùng là cancellation flag:

```jsx
function useAsync(asyncFn, deps) {
  const [state, setState] = useState({ data: null, loading: false, error: null });

  useEffect(() => {
    let cancelled = false;
    setState(s => ({ ...s, loading: true, error: null }));

    asyncFn()
      .then(data => { if (!cancelled) setState({ data, loading: false, error: null }); })
      .catch(error => { if (!cancelled) setState({ data: null, loading: false, error }); });

    return () => { cancelled = true; }; // Cleanup: ngăn setState sau khi unmount
  }, deps);

  return state;
}
```

Với fetch cụ thể, mình ưu tiên `AbortController` vì nó thực sự hủy network request thay vì chỉ bỏ qua kết quả:

```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal })
    .then(r => r.json())
    .then(setData)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [url]);
```

Thực tế với hầu hết project mình sẽ dùng TanStack Query thay vì tự viết, nhưng hiểu được cancellation pattern là những gì thư viện đó đang làm bên dưới."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "What are some of the new features in React 18?"**

**Model Answer:**
"React 18 introduced several things, but the three most practically impactful ones are:

First, **automatic batching**. In React 17, state updates inside `setTimeout`, Promises, or native event handlers weren't batched — each one triggered a separate re-render. React 18 batches them all automatically, so multiple state updates in async code now produce a single re-render by default.

Second, **`useTransition` and `useDeferredValue`** — two hooks for marking state updates as non-urgent so they don't block user input. These are part of the Concurrent Mode capabilities.

Third, **the new root API**. You now create your app with `ReactDOM.createRoot(container).render(<App />)` instead of `ReactDOM.render()`. This opt-in is what enables all the concurrent features — the old `render()` API still works but doesn't get concurrent capabilities.

There's also `useId` for generating stable IDs for accessibility, and Suspense improvements for server-side rendering streaming."

**Trả lời (Tiếng Việt):**
"React 18 giới thiệu nhiều thứ, nhưng ba cái có tác động thực tế nhất là:

Thứ nhất, **automatic batching**. Trong React 17, các state update bên trong `setTimeout`, Promise, hay native event handler không được batch — mỗi cái trigger một re-render riêng. React 18 batch tất cả tự động, nên nhiều state update trong async code giờ chỉ tạo ra một re-render duy nhất theo mặc định.

Thứ hai, **`useTransition` và `useDeferredValue`** — hai hook để đánh dấu state update là không khẩn cấp để chúng không block user input. Đây là một phần của khả năng Concurrent Mode.

Thứ ba, **root API mới**. Giờ bạn tạo app bằng `ReactDOM.createRoot(container).render(<App />)` thay vì `ReactDOM.render()`. Việc opt-in này là thứ kích hoạt tất cả concurrent features — API `render()` cũ vẫn hoạt động nhưng không có concurrent capabilities.

Còn có `useId` để tạo stable ID cho accessibility, và cải thiện Suspense cho server-side rendering streaming."

---

**Q2 (Mid): "Explain `useTransition`. What problem does it solve?"**

**Model Answer:**
"The problem it solves is keeping the UI responsive during expensive renders. Before `useTransition`, if a user typed in a search box and the results list was slow to render — say it had to filter 10,000 items — the input itself would feel laggy because React was busy rendering the results.

`useTransition` lets you tell React: 'this state update is non-urgent — prioritize user input over it.'

```jsx
const [isPending, startTransition] = useTransition();

function handleSearch(e) {
  setQuery(e.target.value); // Urgent — input updates immediately

  startTransition(() => {
    setResults(filterData(allData, e.target.value)); // Non-urgent — can be interrupted
  });
}
```

React renders the urgent input update first, then works on the transition in the background. If the user keeps typing, React can interrupt the transition render and start over with the new value. The `isPending` flag lets you show a loading indicator while the transition is in progress.

The key distinction from debouncing: debouncing delays the update, while `useTransition` starts the update immediately but gives it a lower priority."

**Trả lời (Tiếng Việt):**
"Vấn đề nó giải quyết là giữ UI responsive trong khi render tốn kém. Trước `useTransition`, nếu người dùng gõ vào search box và danh sách kết quả render chậm — ví dụ phải filter 10,000 item — bản thân ô input cũng cảm thấy lag vì React đang bận render kết quả.

`useTransition` cho phép bạn nói với React: 'state update này không khẩn cấp — ưu tiên user input hơn nó.'

```jsx
const [isPending, startTransition] = useTransition();

function handleSearch(e) {
  setQuery(e.target.value); // Khẩn cấp — input cập nhật ngay lập tức

  startTransition(() => {
    setResults(filterData(allData, e.target.value)); // Không khẩn cấp — có thể bị interrupt
  });
}
```

React render urgent input update trước, rồi làm việc với transition ở background. Nếu người dùng tiếp tục gõ, React có thể interrupt transition render và bắt đầu lại với giá trị mới. Flag `isPending` cho phép bạn hiển thị loading indicator trong khi transition đang diễn ra.

Điểm khác biệt quan trọng so với debouncing: debouncing trì hoãn update, còn `useTransition` bắt đầu update ngay lập tức nhưng với priority thấp hơn."

---

**Q3 (Mid): "What is the difference between `useTransition` and `useDeferredValue`?"**

**Model Answer:**
"They solve the same underlying problem — preventing expensive renders from blocking urgent updates — but from different angles.

`useTransition` is for when **you control the state setter**. You wrap the setState call in `startTransition`:
```jsx
startTransition(() => setResults(filter(data, query)));
```

`useDeferredValue` is for when **you receive the value as a prop or can't wrap the setter** — for example, if a third-party component triggers the update or the setter is in a parent you don't own:
```jsx
const deferredQuery = useDeferredValue(query); // query comes from props
// Pass deferredQuery to the expensive child — it lags behind query
```

In practice, `useTransition` is what I use most often because I'm usually the one calling setState. `useDeferredValue` is more useful at component boundaries where I only receive a value but don't control when it changes."

**Trả lời (Tiếng Việt):**
"Cả hai giải quyết cùng vấn đề cốt lõi — ngăn render tốn kém block urgent update — nhưng từ góc độ khác nhau.

`useTransition` dùng khi **bạn kiểm soát được setter**. Bạn wrap lời gọi setState vào `startTransition`:
```jsx
startTransition(() => setResults(filter(data, query)));
```

`useDeferredValue` dùng khi **bạn nhận giá trị như prop hoặc không thể wrap setter** — ví dụ nếu component bên thứ ba trigger update hoặc setter nằm ở parent mà bạn không sở hữu:
```jsx
const deferredQuery = useDeferredValue(query); // query đến từ props
// Truyền deferredQuery cho child tốn kém — nó lag phía sau query
```

Thực tế, `useTransition` là cái mình dùng thường xuyên nhất vì thường thì mình là người gọi setState. `useDeferredValue` hữu ích hơn ở ranh giới component khi mình chỉ nhận giá trị nhưng không kiểm soát khi nào nó thay đổi."

---

**Q4 (Senior): "What is Concurrent Mode and what does it actually enable in React 18?"**

**Model Answer:**
"Concurrent Mode is a rendering model where React can work on multiple versions of the UI simultaneously, pause work in progress, and prioritize more urgent updates over less urgent ones.

Before concurrent rendering, React's render was synchronous — once it started rendering a tree, it ran to completion. On slow devices with large component trees, this blocked the browser's main thread and made the UI feel unresponsive.

With Concurrent Mode enabled via `createRoot`, React can:

1. **Interrupt rendering** — if a high-priority update (like a user click) arrives while a low-priority render is in progress, React pauses the low-priority work and handles the urgent update first.

2. **Render in the background** — with `useTransition`, React keeps showing the old UI while preparing the new one behind the scenes. The user never sees an incomplete state.

3. **Re-use already-rendered work** — React can discard and restart renders if dependencies changed mid-way through.

The practical features you get from this are `useTransition`, `useDeferredValue`, better Suspense behavior (showing stale content instead of flashing back to a skeleton), and streaming SSR. All of these are built on top of the concurrent scheduler.

One important thing: concurrent rendering means your component render functions can run more than once before committing, so they must be pure — no side effects in the render body."

**Trả lời (Tiếng Việt):**
"Concurrent Mode là mô hình rendering trong đó React có thể làm việc trên nhiều phiên bản UI đồng thời, tạm dừng công việc đang tiến hành, và ưu tiên update khẩn cấp hơn những cái không khẩn cấp.

Trước concurrent rendering, render của React là đồng bộ — một khi bắt đầu render một cây component, nó chạy đến hoàn thành. Trên thiết bị chậm với component tree lớn, điều này chặn main thread của browser và làm UI cảm thấy không responsive.

Với Concurrent Mode được kích hoạt qua `createRoot`, React có thể:

1. **Interrupt rendering** — nếu một update priority cao (như click của người dùng) đến trong khi render priority thấp đang tiến hành, React tạm dừng công việc thấp priority và xử lý update khẩn cấp trước.

2. **Render ở background** — với `useTransition`, React giữ hiển thị UI cũ trong khi chuẩn bị cái mới phía sau. Người dùng không bao giờ thấy trạng thái chưa hoàn chỉnh.

3. **Tái sử dụng work đã render** — React có thể bỏ và restart render nếu dependency thay đổi giữa chừng.

Các tính năng thực tế bạn nhận được là `useTransition`, `useDeferredValue`, Suspense hoạt động tốt hơn (hiển thị nội dung cũ thay vì flash về skeleton), và streaming SSR. Tất cả được xây dựng trên concurrent scheduler.

Một điều quan trọng: concurrent rendering có nghĩa là render function của component có thể chạy nhiều hơn một lần trước khi commit, nên chúng phải pure — không có side effect trong render body."

---

**Q5 (Senior): "How does automatic batching in React 18 differ from React 17? When might you need to opt out?"**

**Model Answer:**
"In React 17, automatic batching only happened inside React synthetic event handlers. If you called multiple setStates inside a `setTimeout`, a Promise `.then()`, or a native DOM event listener, each setState triggered its own separate re-render.

React 18 batches everything by default — Promises, setTimeout, native events, and any async code all batch together into a single re-render.

```jsx
// React 17: 2 renders. React 18: 1 render.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
}, 1000);
```

The opt-out is `flushSync` from `react-dom`. You'd use it if you specifically need a synchronous re-render between two updates — for example, if you're reading a DOM measurement right after setting state:

```jsx
import { flushSync } from 'react-dom';

flushSync(() => setCount(count + 1)); // Forces a synchronous render immediately
const height = divRef.current.scrollHeight; // Read the DOM after the render
flushSync(() => setScrollTop(height));
```

I rarely need `flushSync` in practice — it's mostly for edge cases with third-party library integrations that depend on reading the DOM synchronously after a state update."

**Trả lời (Tiếng Việt):**
"Trong React 17, automatic batching chỉ xảy ra bên trong React synthetic event handler. Nếu bạn gọi nhiều setState bên trong `setTimeout`, Promise `.then()`, hay native DOM event listener, mỗi setState trigger một re-render riêng.

React 18 batch mọi thứ theo mặc định — Promise, setTimeout, native event, và bất kỳ async code nào đều được batch thành một re-render duy nhất.

```jsx
// React 17: 2 render. React 18: 1 render.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
}, 1000);
```

Cách opt-out là `flushSync` từ `react-dom`. Bạn dùng nó khi cần một re-render đồng bộ giữa hai update — ví dụ nếu bạn đang đọc DOM measurement ngay sau khi set state:

```jsx
import { flushSync } from 'react-dom';

flushSync(() => setCount(count + 1)); // Buộc render đồng bộ ngay lập tức
const height = divRef.current.scrollHeight; // Đọc DOM sau khi render
flushSync(() => setScrollTop(height));
```

Mình hiếm khi cần `flushSync` trên thực tế — chủ yếu cho edge case với tích hợp thư viện bên thứ ba phụ thuộc vào việc đọc DOM đồng bộ sau state update."

---

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "How do you identify performance problems in a React app?"**

**Model Answer:**
"I always measure before I optimize — otherwise I'm guessing. The first tool I reach for is React DevTools Profiler.

In the Profiler tab, I hit Record, interact with the slow part of the app, then stop. It gives me a flamegraph showing every component that rendered in each commit, color-coded by how long it took. Orange and red components are the slow ones. There's also a 'Ranked' chart that sorts components by render time so the worst offenders are obvious.

I also use the 'Highlight updates' feature in the Components tab — it flashes a colored border around every component that re-renders. If something is flashing when I haven't interacted with it at all, that's an unnecessary re-render I should investigate.

For bundle size, `webpack-bundle-analyzer` (or `next-bundle-analyzer` in Next.js) visualizes what's eating up my JavaScript bundle — it often reveals that a heavy library is being imported where a lighter alternative would work.

The general principle: measure first, find the actual bottleneck, then optimize. Premature optimization usually adds complexity without any real benefit."

**Trả lời (Tiếng Việt):**
"Mình luôn đo trước khi tối ưu — nếu không là đang đoán mò. Công cụ đầu tiên mình dùng là React DevTools Profiler.

Trong tab Profiler, mình bấm Record, tương tác với phần chậm của app, rồi dừng. Nó cho mình flamegraph hiển thị mọi component render trong mỗi commit, color-coded theo thời gian. Component màu cam và đỏ là các component chậm. Còn có chart 'Ranked' sắp xếp component theo thời gian render để những cái tệ nhất hiện rõ.

Mình cũng dùng tính năng 'Highlight updates' trong tab Components — nó flash viền màu quanh mọi component re-render. Nếu thứ gì đó flash mà mình không tương tác gì, đó là re-render không cần thiết cần điều tra.

Với bundle size, `webpack-bundle-analyzer` (hoặc `next-bundle-analyzer` trong Next.js) visualize những gì đang chiếm dung lượng JavaScript bundle — thường tiết lộ rằng một thư viện nặng đang được import trong khi một thư viện nhẹ hơn hoàn toàn đủ dùng.

Nguyên tắc chung: đo trước, tìm bottleneck thực sự, rồi mới tối ưu. Premature optimization thường thêm phức tạp mà không có lợi ích thực sự."

---

**Q2 (Mid): "A component is re-rendering too often. Walk me through how you'd fix it."**

**Model Answer:**
"I'd work through a checklist.

**Step 1 — Confirm it's actually a problem.** Open DevTools Profiler, find the component, see how long it takes. A fast re-render isn't worth optimizing.

**Step 2 — Check why it's re-rendering.** The most common cause is a parent re-rendering and passing new object/array/function references as props each time:
```jsx
// This creates a new object every render — child always re-renders
<Child style={{ color: 'red' }} />
```

**Step 3 — Apply `React.memo` to the child** if the props genuinely haven't changed:
```jsx
const Child = React.memo(function Child({ data, onClick }) { ... });
```

**Step 4 — Stabilize the props.** `React.memo` only works if the props pass a shallow equality check. For functions, I wrap them in `useCallback`. For objects or computed arrays, I use `useMemo`:
```jsx
const handleClick = useCallback(() => doThing(id), [id]);
const processedData = useMemo(() => transform(rawData), [rawData]);
```

**Step 5 — Check if state is too high up the tree.** If unrelated state lives at the root, every state change re-renders the whole tree. Moving state down to where it's actually needed (state colocation) is often the most impactful fix."

**Trả lời (Tiếng Việt):**
"Mình sẽ đi qua một checklist.

**Bước 1 — Xác nhận đây thực sự là vấn đề.** Mở DevTools Profiler, tìm component, xem mất bao lâu. Re-render nhanh không đáng tối ưu.

**Bước 2 — Kiểm tra tại sao nó re-render.** Nguyên nhân phổ biến nhất là parent re-render và truyền object/array/function reference mới mỗi lần:
```jsx
// Tạo object mới mỗi lần render — child luôn re-render
<Child style={{ color: 'red' }} />
```

**Bước 3 — Áp dụng `React.memo` cho child** nếu props thực sự không thay đổi:
```jsx
const Child = React.memo(function Child({ data, onClick }) { ... });
```

**Bước 4 — Stabilize props.** `React.memo` chỉ hoạt động nếu props pass shallow equality check. Với function, wrap chúng bằng `useCallback`. Với object hoặc computed array, dùng `useMemo`:
```jsx
const handleClick = useCallback(() => doThing(id), [id]);
const processedData = useMemo(() => transform(rawData), [rawData]);
```

**Bước 5 — Kiểm tra xem state có quá cao trong cây không.** Nếu state không liên quan nằm ở root, mọi state change đều re-render cả cây. Di chuyển state xuống nơi thực sự cần (state colocation) thường là fix có tác động lớn nhất."

---

**Q3 (Mid): "When would you use list virtualization and how does it work?"**

**Model Answer:**
"Virtualization is for when you have a very long list — typically more than a few hundred items — and rendering all of them to the DOM at once causes slowdowns.

The idea is simple: instead of rendering 10,000 list items, you only render the ones currently visible in the viewport plus a small buffer above and below. As the user scrolls, items are recycled — the ones that scroll out of view are replaced with new items from the other end.

```jsx
import { FixedSizeList } from 'react-window';

function BigList({ items }) {
  return (
    <FixedSizeList height={600} itemCount={items.length} itemSize={50} width="100%">
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>  // style sets position
      )}
    </FixedSizeList>
  );
}
```

`react-window` is my go-to library — it's lightweight. `react-virtual` (from TanStack) is more flexible for variable-height items and works well with TanStack Table.

The `style` prop from the render function is important — it sets the absolute position and height of each row so the scrollbar reflects the full list height even though only a subset of rows exists in the DOM."

**Trả lời (Tiếng Việt):**
"Virtualization dùng khi bạn có danh sách rất dài — thường hơn vài trăm item — và render tất cả vào DOM cùng lúc gây chậm.

Ý tưởng đơn giản: thay vì render 10,000 list item, bạn chỉ render những cái đang hiển thị trong viewport cộng một buffer nhỏ phía trên và dưới. Khi người dùng scroll, item được tái sử dụng — những cái scroll ra khỏi viewport được thay bằng item mới từ phía kia.

```jsx
import { FixedSizeList } from 'react-window';

function BigList({ items }) {
  return (
    <FixedSizeList height={600} itemCount={items.length} itemSize={50} width="100%">
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>  // style set vị trí
      )}
    </FixedSizeList>
  );
}
```

`react-window` là thư viện mình hay dùng — nhẹ. `react-virtual` (từ TanStack) linh hoạt hơn cho item có chiều cao khác nhau và hoạt động tốt với TanStack Table.

Prop `style` từ render function quan trọng — nó set vị trí tuyệt đối và chiều cao của mỗi hàng để scrollbar phản ánh chiều cao toàn bộ danh sách dù chỉ có một subset hàng tồn tại trong DOM."

---

**Q4 (Senior): "How does state colocation improve performance? Give a real-world example."**

**Model Answer:**
"State colocation means keeping state as close as possible to the components that actually use it, instead of lifting it unnecessarily high up the tree.

The performance benefit is direct: when state changes, React re-renders the component that owns the state and all its descendants. If you put state at the application root when only one small leaf component needs it, a state change re-renders the entire app tree.

A real example: imagine a search input in a sidebar. A common mistake is putting the search query in top-level application state because 'maybe we'll need it elsewhere later.' Every keystroke then re-renders the entire app.

Instead, keep that state in the sidebar component:
```jsx
// BAD — query in top-level state, whole app re-renders on every keystroke
function App() {
  const [searchQuery, setSearchQuery] = useState('');
  return <Layout><Sidebar query={searchQuery} onSearch={setSearchQuery} /><Main /></Layout>;
}

// GOOD — state lives in the component that owns it
function Sidebar() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <div>
      <SearchInput value={searchQuery} onChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
    </div>
  );
}
```

In the second version, `Main` never re-renders when the user types in the search box. This is often a bigger win than adding `React.memo` everywhere, because you're eliminating unnecessary work at the source rather than bailing out of it after it's already started."

**Trả lời (Tiếng Việt):**
"State colocation có nghĩa là giữ state càng gần những component thực sự dùng nó càng tốt, thay vì lift nó lên cao không cần thiết.

Lợi ích performance trực tiếp: khi state thay đổi, React re-render component sở hữu state và tất cả con cháu của nó. Nếu bạn đặt state ở application root trong khi chỉ một component leaf nhỏ cần nó, một state change sẽ re-render toàn bộ app tree.

Ví dụ thực tế: tưởng tượng một search input trong sidebar. Sai lầm phổ biến là đặt search query trong top-level application state vì 'có thể sau này sẽ cần ở chỗ khác.' Mỗi lần gõ phím sẽ re-render toàn bộ app.

Thay vào đó, giữ state đó trong sidebar component:
```jsx
// TỆ — query trong top-level state, cả app re-render mỗi keystroke
function App() {
  const [searchQuery, setSearchQuery] = useState('');
  return <Layout><Sidebar query={searchQuery} onSearch={setSearchQuery} /><Main /></Layout>;
}

// TỐT — state sống trong component sở hữu nó
function Sidebar() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <div>
      <SearchInput value={searchQuery} onChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
    </div>
  );
}
```

Trong phiên bản thứ hai, `Main` không bao giờ re-render khi người dùng gõ vào search box. Đây thường là win lớn hơn việc thêm `React.memo` khắp nơi, vì bạn loại bỏ công việc không cần thiết từ nguồn thay vì thoát ra sau khi nó đã bắt đầu."

---

**Q5 (Senior): "What are the trade-offs of using a global state library like Zustand versus Context API for performance?"**

**Model Answer:**
"The key performance difference is how granular subscriptions are.

With Context API, when the context value changes, every component that calls `useContext` for that context re-renders — even if it only cares about one field in the context object. You can mitigate this by splitting contexts or using `useMemo` on the value, but it requires careful manual structuring.

With Zustand or Jotai, components subscribe to specific slices of state:
```jsx
// Only re-renders when userName changes, not when userAvatar or userEmail change
const userName = useStore(state => state.user.name);
```

This is a significant advantage for large stores with many subscribers. Each component declares exactly what it depends on, and Zustand does the equality check automatically.

The trade-off is an extra dependency and a different mental model. For small-to-mid apps where the shared state is simple and not updated at high frequency, Context is perfectly fine and keeps the bundle lean. When I start seeing performance issues from context — components re-rendering when their relevant slice didn't change — that's when I'd migrate to Zustand or consider splitting the context more aggressively.

There's also the Zustand devtools integration which gives you Redux-style debugging, which is a developer experience win at scale."

**Trả lời (Tiếng Việt):**
"Sự khác biệt performance chính là granular subscriptions.

Với Context API, khi context value thay đổi, mọi component gọi `useContext` cho context đó đều re-render — dù nó chỉ quan tâm đến một field trong context object. Bạn có thể giảm thiểu bằng cách chia context hoặc dùng `useMemo` trên value, nhưng cần cấu trúc thủ công cẩn thận.

Với Zustand hay Jotai, component subscribe vào các slice cụ thể của state:
```jsx
// Chỉ re-render khi userName thay đổi, không phải khi userAvatar hay userEmail thay đổi
const userName = useStore(state => state.user.name);
```

Đây là lợi thế đáng kể cho store lớn với nhiều subscriber. Mỗi component khai báo chính xác nó phụ thuộc vào gì, và Zustand tự động kiểm tra equality.

Trade-off là thêm một dependency và mental model khác. Với app nhỏ đến vừa nơi shared state đơn giản và không cập nhật với tần suất cao, Context hoàn toàn ổn và giữ bundle gọn. Khi bắt đầu thấy vấn đề performance từ context — component re-render khi slice liên quan không thay đổi — đó là lúc mình migrate sang Zustand hoặc cân nhắc chia context tích cực hơn.

Còn có tích hợp Zustand devtools cho phép debug theo kiểu Redux, đây là win về developer experience ở quy mô lớn."

---

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

---

## 30. useLayoutEffect vs useEffect — Timing Difference

**[Mid → Senior]**

**EN:** What is the difference between `useLayoutEffect` and `useEffect`? When should you use one over the other?

**VI:** Sự khác biệt giữa `useLayoutEffect` và `useEffect` là gì? Khi nào nên dùng cái nào?

### Trả lời

Cả hai hook đều nhận cùng signature, nhưng **thời điểm chạy** khác nhau hoàn toàn:

```
Execution order:
─────────────────────────────────────────────────────────────────
  [React renders component]
        ↓
  [React commits DOM changes]
        ↓
  useLayoutEffect fires  ◄── SYNCHRONOUS, blocks browser paint
        ↓
  [Browser paints to screen]  ◄── user sees the update
        ↓
  useEffect fires        ◄── asynchronous, after paint
─────────────────────────────────────────────────────────────────
```

**`useLayoutEffect`** chạy **đồng bộ** sau khi React commit DOM nhưng **trước khi trình duyệt vẽ**. Điều này có nghĩa là bạn có thể đọc/ghi DOM mà người dùng sẽ không thấy trạng thái trung gian.

**`useEffect`** chạy **bất đồng bộ** sau khi trình duyệt đã vẽ xong. Đây là default và phù hợp với phần lớn trường hợp.

#### Khi nào dùng `useLayoutEffect`:

- **DOM measurements**: `getBoundingClientRect()`, `scrollHeight`, `offsetWidth`
- **Tooltip/popover positioning**: cần biết kích thước element để định vị
- **Synchronous DOM mutations before paint**: tránh flicker
- **Animations cần vị trí chính xác ban đầu**

#### Khi nào dùng `useEffect`:

- Data fetching
- Event subscriptions (websocket, resize listener)
- `localStorage` read/write
- Analytics / logging
- Bất cứ thứ gì không cần block paint

#### Ví dụ: Tooltip Positioning

```tsx
// BAD — dùng useEffect gây FLICKER:
// Component render với vị trí sai → browser vẽ → useEffect chạy → vị trí đúng → browser vẽ lại
function TooltipBad({ targetRef }: { targetRef: React.RefObject<HTMLElement> }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (targetRef.current && tooltipRef.current) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({ top: rect.bottom + 8, left: rect.left });
    }
  }, []);

  return (
    <div ref={tooltipRef} style={{ position: 'fixed', top: position.top, left: position.left }}>
      Tooltip content
    </div>
  );
}

// GOOD — dùng useLayoutEffect, KHÔNG có flicker:
// Component render → React commits DOM → useLayoutEffect chạy ngay → browser vẽ đúng 1 lần
function TooltipGood({ targetRef }: { targetRef: React.RefObject<HTMLElement> }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef<HTMLDivElement>(null);

  useLayoutEffect(() => {
    if (targetRef.current && tooltipRef.current) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({ top: rect.bottom + 8, left: rect.left });
    }
  }, []);

  return (
    <div ref={tooltipRef} style={{ position: 'fixed', top: position.top, left: position.left }}>
      Tooltip content
    </div>
  );
}
```

#### SSR Caveat — `useIsomorphicLayoutEffect`

`useLayoutEffect` **không chạy trên server** (SSR) và React sẽ warning:

```
Warning: useLayoutEffect does nothing on the server because its effect cannot be encoded into the server renderer's output format.
```

Fix: dùng `useIsomorphicLayoutEffect` — một custom hook chọn đúng hook dựa vào môi trường:

```tsx
import { useEffect, useLayoutEffect } from 'react';

// Trên server: useEffect (no-op safe)
// Trên client: useLayoutEffect (synchronous)
const useIsomorphicLayoutEffect =
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;

export default useIsomorphicLayoutEffect;

// Sử dụng:
function MyComponent() {
  const ref = useRef<HTMLDivElement>(null);

  useIsomorphicLayoutEffect(() => {
    // DOM measurement an toàn cả SSR lẫn client
    if (ref.current) {
      const { width } = ref.current.getBoundingClientRect();
      console.log('Width:', width);
    }
  }, []);

  return <div ref={ref}>Content</div>;
}
```

> Libraries như `react-use`, `usehooks-ts` đều export `useIsomorphicLayoutEffect` — bạn không cần tự viết.

---

### 🎤 Mock Interview — Q&A

**Q1: "What's the difference between useLayoutEffect and useEffect?"**

> `useEffect` runs asynchronously after the browser has painted — it's the default and safe for most side effects like data fetching or subscriptions. `useLayoutEffect` runs synchronously after React commits DOM changes but before the browser paints. This lets you read or mutate the DOM without the user seeing an intermediate state — critical for avoiding visual flicker in things like tooltip positioning.

**Q2: "When should you use useLayoutEffect vs useEffect? Give a real example."**

> Use `useLayoutEffect` when you need to measure the DOM and immediately update something based on that measurement. A classic case: a tooltip that needs to position itself relative to a target element. If you use `useEffect`, the tooltip renders at `(0,0)` first, then jumps to the correct position after paint — that's a visible flicker. With `useLayoutEffect`, the position calculation happens before paint, so the user only ever sees the correct position.

**Q3: "Why does useLayoutEffect cause a warning in SSR? How do you fix it?"**

> On the server there is no DOM, so `useLayoutEffect` has nothing to do — and it can't be serialized into server output. React warns about this. The fix is a `useIsomorphicLayoutEffect` custom hook: it's `useLayoutEffect` on the client and `useEffect` on the server. Since `useEffect` is a no-op on the server anyway, this makes the code safe in both environments. Many libraries like `react-use` already export this hook.

---

## 31. Fragments — Grouping Without Extra DOM Nodes

**[Junior → Mid]**

**EN:** What are React Fragments? When and why do you use them? What is the difference between `<>` and `<React.Fragment>`?

**VI:** React Fragment là gì? Khi nào và tại sao dùng chúng? `<>` khác `<React.Fragment>` như thế nào?

### Trả lời

React yêu cầu mỗi component trả về một **root element duy nhất**. Trước đây, developers hay bọc output trong một `<div>` thừa. **Fragment** giải quyết vấn đề này — nhóm nhiều elements lại mà **không thêm node nào vào DOM thật**.

#### Tại sao không dùng `<div>` wrapper?

Một số trường hợp div wrapper phá vỡ cấu trúc HTML hợp lệ:

```tsx
// BAD — div phá vỡ table structure (invalid HTML!)
function TableRows() {
  return (
    <div>   {/* ← div không được phép trong <tbody> */}
      <tr><td>Row 1, Col 1</td><td>Row 1, Col 2</td></tr>
      <tr><td>Row 2, Col 1</td><td>Row 2, Col 2</td></tr>
    </div>
  );
}

// GOOD — Fragment giữ nguyên structure
function TableRows() {
  return (
    <>
      <tr><td>Row 1, Col 1</td><td>Row 1, Col 2</td></tr>
      <tr><td>Row 2, Col 1</td><td>Row 2, Col 2</td></tr>
    </>
  );
}

// Sử dụng:
<table>
  <tbody>
    <TableRows />
  </tbody>
</table>
```

#### Cú pháp: `<>` vs `<React.Fragment>`

| | Short syntax `<>` | `<React.Fragment>` |
|---|---|---|
| Import cần | Không | `import React from 'react'` (hoặc named import) |
| Chấp nhận props | ❌ Không | ✅ Có (`key` prop) |
| Khi nào dùng | Grouping đơn giản | Khi cần `key` (render list) |

#### THE KEY GOTCHA — List rendering với Fragment

```tsx
// WRONG — <> không chấp nhận key prop:
function ItemList({ items }: { items: Item[] }) {
  return (
    <>
      {items.map((item) => (
        <>  {/* ❌ Lỗi: key không được chấp nhận ở đây */}
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </>
      ))}
    </>
  );
}

// CORRECT — dùng React.Fragment với key:
function ItemList({ items }: { items: Item[] }) {
  return (
    <dl>
      {items.map((item) => (
        <React.Fragment key={item.id}>  {/* ✅ key được chấp nhận */}
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

#### Các use cases phổ biến

```tsx
// 1. Flex/Grid children — tránh wrapper phá layout
function CardActions() {
  return (
    <>
      <button>Edit</button>
      <button>Delete</button>
      <button>Share</button>
    </>
  );
}

// 2. Conditional rendering — không muốn thêm DOM node
function UserInfo({ user }: { user: User }) {
  return (
    <>
      <h1>{user.name}</h1>
      {user.isPremium && <PremiumBadge />}
      <p>{user.bio}</p>
    </>
  );
}

// 3. Return nhiều elements từ custom hook render
function useDialogContent() {
  return (
    <>
      <DialogHeader />
      <DialogBody />
      <DialogFooter />
    </>
  );
}
```

---

### 🎤 Mock Interview — Q&A

**Q1: "What is a React Fragment and when would you use it?"**

> A Fragment lets you group multiple elements without adding an extra node to the DOM. You'd use it when you need to return multiple sibling elements from a component but a wrapper `div` would break the HTML structure — for example, returning `<tr>` elements from a `<tbody>`, or `<dt>`/`<dd>` pairs in a `<dl>`. It also helps keep the DOM clean in general.

**Q2: "What's the difference between `<>` and `<React.Fragment>`?"**

> They are functionally the same for simple grouping. The key difference is that `<>` is shorthand and accepts no props. `<React.Fragment>` is the explicit form and accepts the `key` prop. If you're rendering a list where each group needs a `key`, you must use `<React.Fragment key={...}>` — you cannot do `<> key={...}>`.

**Q3: "I'm rendering a list and need to add a key. How do I use Fragment with a key?"**

> You can't use the `<>` shorthand in that case. You need the explicit `<React.Fragment key={item.id}>` syntax. For example, if each list item renders two sibling elements like a `<dt>` and a `<dd>`, you'd wrap them in `<React.Fragment key={item.id}>` inside the `.map()` call. React needs the key to track which group changed during reconciliation.

---

## 32. Conditional Rendering — Patterns and the && Gotcha

**[Junior → Mid]**

**EN:** What are the different ways to conditionally render in React? What is the `&&` gotcha with falsy values?

**VI:** Có những cách nào để render có điều kiện trong React? Gotcha của `&&` với giá trị falsy là gì?

### Trả lời

React không có directive đặc biệt như `v-if` (Vue) hay `*ngIf` (Angular) — bạn dùng JavaScript thuần bên trong JSX.

#### 4 Pattern chính

**1. Ternary operator** — khi cần render một trong hai thứ:

```tsx
function UserGreeting({ isLoggedIn }: { isLoggedIn: boolean }) {
  return (
    <div>
      {isLoggedIn ? <UserDashboard /> : <LoginPrompt />}
    </div>
  );
}
```

**2. `&&` short-circuit** — khi chỉ render khi điều kiện true:

```tsx
function Notification({ message }: { message: string | null }) {
  return (
    <div>
      {message && <Alert>{message}</Alert>}
    </div>
  );
}
```

**3. Early return** — khi cần logic phức tạp hơn:

```tsx
function Profile({ user }: { user: User | null }) {
  if (!user) return <Skeleton />;
  if (user.isBanned) return <BannedMessage />;

  return <UserProfile user={user} />;
}
```

**4. Return `null`** — render hoàn toàn không có gì:

```tsx
function WarningBanner({ visible }: { visible: boolean }) {
  if (!visible) return null;
  return <div className="warning">Warning!</div>;
}
```

---

#### THE CRITICAL GOTCHA: `&&` với giá trị số 0

```tsx
// BUG — hiển thị "0" trên screen khi count = 0 !
function ItemCount({ count }: { count: number }) {
  return (
    <div>
      {count && <span>You have {count} items</span>}
      {/* Khi count = 0:
          - 0 là falsy → short-circuit → React nhận giá trị 0
          - React render số 0 như text!
          - User thấy "0" xuất hiện trên trang ← BUG */}
    </div>
  );
}
```

Tại sao? Vì `&&` trong JavaScript trả về **giá trị thực sự**, không phải boolean:
- `0 && <Component />` → trả về `0` (falsy value)
- React render `0` như string `"0"` — không phải `null`/`undefined`/`false` (những thứ React bỏ qua)

#### 3 cách fix:

```tsx
// FIX 1 — So sánh tường minh (recommended):
{count > 0 && <span>You have {count} items</span>}

// FIX 2 — Convert sang boolean:
{Boolean(count) && <span>You have {count} items</span>}

// FIX 3 — Double bang:
{!!count && <span>You have {count} items</span>}

// FIX 4 — Ternary (khi cần fallback):
{count ? <span>You have {count} items</span> : null}
```

> **Lưu ý**: Gotcha này cũng xảy ra với `array.length && <Component />` khi array rỗng → hiển thị `"0"`. Luôn dùng `array.length > 0 &&` hoặc `!!array.length &&`.

#### Khi nào dùng pattern nào?

| Pattern | Khi nào dùng |
|---|---|
| Ternary `? :` | Hai nhánh: render A hoặc B |
| `&&` | Một nhánh: render hoặc không (nhưng cẩn thận falsy!) |
| Early return | Multiple conditions, component phức tạp |
| Return `null` | Component chủ động ẩn hoàn toàn |

---

### 🎤 Mock Interview — Q&A

**Q1: "What are the different ways to conditionally render in React?"**

> There are four main patterns: ternary operator for rendering one of two things, `&&` short-circuit for rendering something or nothing, early `return` for complex conditions at the component level, and returning `null` to render nothing. React ignores `null`, `undefined`, and `false` in JSX — so returning `null` is the standard way to opt out of rendering.

**Q2: "Can you explain the `&&` gotcha with zero values? I've seen this bug in production."**

> Yes, this is a very common bug. When you write `{count && <Component />}`, JavaScript's `&&` returns the actual falsy value when the left side is falsy — not `false` the boolean. So when `count` is `0`, the expression evaluates to `0` — and React renders that as the text "0" on screen. React only skips rendering for `null`, `undefined`, and `false` — not for `0`. The fix is to use an explicit comparison: `{count > 0 && <Component />}`. Same issue with `{array.length && ...}` when the array is empty.

**Q3: "When would you use a ternary vs `&&` for conditional rendering?"**

> Use `&&` when you only have one branch — you either render something or nothing. Use ternary when you need to choose between two things. But if either operand could be `0` or another falsy non-boolean, I default to ternary or an explicit comparison to avoid the gotcha. For complex conditions with multiple branches, I prefer early returns in the component body over deeply nested ternaries — it keeps the JSX readable.

---

## 33. Event Handling — Synthetic Events and Event Delegation

**[Junior → Mid → Senior]**

**EN:** How does React handle events? What is a SyntheticEvent? How does event delegation work in React, and what changed in React 17?

**VI:** React xử lý events như thế nào? SyntheticEvent là gì? Event delegation hoạt động như thế nào trong React, và React 17 thay đổi gì?

### Trả lời

#### SyntheticEvent

React **không** attach listeners trực tiếp vào DOM elements. Thay vào đó, React tạo ra **SyntheticEvent** — một wrapper cross-browser normalize native DOM events:

- Cùng interface với native events (`preventDefault`, `stopPropagation`, `target`, `currentTarget`...)
- Hoạt động giống nhau trên mọi browser
- Trước React 17: SyntheticEvents được **pooled** (tái sử dụng để tiết kiệm memory) — sau React 17 đã bỏ pooling

```tsx
function Form() {
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();   // Ngăn browser reload trang
    console.log(e.target); // FormElement
    console.log(e.nativeEvent); // Truy cập native DOM Event
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

#### Event Delegation

Thay vì attach một listener cho **mỗi element**, React dùng **event delegation** — attach **một listener duy nhất** ở cấp cao hơn và xử lý tất cả events từ đó.

```
DOM Tree:
─────────────────────────────────────────────────
  document
    └── #root  ◄── React 17+: listener ở đây
          └── <App>
                ├── <Button onClick={...}>
                ├── <Input onChange={...}>
                └── <Link onClick={...}>
─────────────────────────────────────────────────
React 16 và trước: listener ở document (cấp cao hơn nữa)
React 17+: listener ở root container
─────────────────────────────────────────────────
```

**Lý do React 17 thay đổi từ `document` sang root container:**

Micro-frontends — nhiều React apps cùng tồn tại trên một trang. Nếu cả hai attach lên `document`, `stopPropagation` của một app có thể chặn events của app kia. Attaching vào root container riêng biệt giải quyết xung đột này.

#### `e.preventDefault()` vs `e.stopPropagation()`

```tsx
// preventDefault — ngăn hành vi mặc định của browser
function LoginForm() {
  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();  // Ngăn form reload trang
    // Xử lý submit bằng JS
  }
  return <form onSubmit={handleSubmit}>...</form>;
}

// stopPropagation — ngăn event bubble lên DOM tree
function Card({ onClick }: { onClick: () => void }) {
  return (
    <div onClick={onClick} className="card">
      <h2>Card Title</h2>
      {/* Delete button KHÔNG nên trigger card's onClick */}
      <button
        onClick={(e) => {
          e.stopPropagation(); // Ngăn click bubble lên div.card
          handleDelete();
        }}
      >
        Delete
      </button>
    </div>
  );
}
```

#### Passing arguments to event handlers

```tsx
// Pattern 1 — Arrow function inline (simple, đủ dùng):
function TodoList({ todos }: { todos: Todo[] }) {
  function handleDelete(id: string) {
    // xử lý delete
  }

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          {todo.text}
          <button onClick={() => handleDelete(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

// Pattern 2 — useCallback (khi cần tránh re-render children):
function TodoList({ todos }: { todos: Todo[] }) {
  const handleDelete = useCallback((id: string) => {
    // xử lý delete
  }, []);

  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} onDelete={handleDelete} />
      ))}
    </ul>
  );
}
```

---

### 🎤 Mock Interview — Q&A

**Q1: "What is a SyntheticEvent in React?"**

> SyntheticEvent is React's cross-browser wrapper around the native DOM event. It normalizes browser differences so you get the same interface everywhere. It has all the same methods as a native event — `preventDefault`, `stopPropagation`, `target`, etc. If you ever need the underlying native event, it's available as `e.nativeEvent`. In React 17+, the pooling of SyntheticEvents was removed, so you can safely use events asynchronously.

**Q2: "How does React's event delegation work, and why did they change it in React 17?"**

> Instead of attaching event listeners to each individual DOM element, React uses event delegation — it attaches a single listener at a higher level and handles all events there. Before React 17, that listener was at `document`. In React 17, it was moved to the root container element. The reason was micro-frontend support: when multiple React apps share a page, attaching to `document` meant one app's `stopPropagation` could interfere with another app's events. Scoping each app's listener to its own root container fixes that.

**Q3: "What's the difference between `e.preventDefault()` and `e.stopPropagation()`?"**

> `preventDefault` stops the browser's default behavior for that event — like preventing a form from submitting and reloading the page, or preventing a link from navigating. `stopPropagation` stops the event from bubbling up the DOM tree — it doesn't affect the browser's default behavior, just whether parent elements receive the event.

**Q4: "I have a card component that navigates on click, but it has a Delete button inside that should NOT navigate. How do I handle this?"**

> Call `e.stopPropagation()` on the Delete button's click handler. That prevents the click from bubbling up to the card's `onClick`. The card's navigation handler never fires, but the delete handler runs normally. The key insight is that both the button and the card will receive the click if you don't stop propagation — `stopPropagation` breaks that chain.

---

## 34. Compound Component Pattern — Context-Based Implicit State

**[Mid → Senior]**

**EN:** What is the Compound Component pattern in React? When would you use it, and how is it implemented?

**VI:** Compound Component pattern là gì? Khi nào dùng và cách implement như thế nào?

### Trả lời

**Compound Components** là một nhóm các components cùng nhau tạo thành một UI hoàn chỉnh, **chia sẻ state ngầm** thông qua Context. Consumer của pattern này được hưởng một API rõ ràng mà không cần biết cách state được quản lý bên trong.

#### Ví dụ thực tế: `<Tabs>` component

```tsx
// API mà consumer sử dụng — rõ ràng, composable:
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Trigger value="profile">Profile</Tabs.Trigger>
    <Tabs.Trigger value="settings">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="profile"><ProfileForm /></Tabs.Panel>
  <Tabs.Panel value="settings"><SettingsForm /></Tabs.Panel>
</Tabs>
```

So sánh với prop-drilling approach (kém linh hoạt hơn):

```tsx
// Không tốt — phải truyền tất cả qua props, khó customize:
<Tabs
  tabs={[
    { value: 'profile', label: 'Profile', content: <ProfileForm /> },
    { value: 'settings', label: 'Settings', content: <SettingsForm /> },
  ]}
/>
```

#### Implementation đầy đủ với TypeScript

```tsx
import React, { createContext, useContext, useState } from 'react';

// 1. Context type
interface TabsContextValue {
  activeTab: string;
  setActiveTab: (value: string) => void;
}

// 2. Tạo Context với undefined default (sẽ check trong hook)
const TabsContext = createContext<TabsContextValue | undefined>(undefined);

// 3. Custom hook để consume context — throw nếu dùng ngoài provider
function useTabsContext() {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error('useTabsContext must be used within <Tabs>');
  return ctx;
}

// 4. Root component — cung cấp state
interface TabsProps {
  defaultValue: string;
  children: React.ReactNode;
}

function Tabs({ defaultValue, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// 5. Sub-components — consume context ngầm
function TabsList({ children }: { children: React.ReactNode }) {
  return <div role="tablist" className="tabs__list">{children}</div>;
}

interface TabsTriggerProps {
  value: string;
  children: React.ReactNode;
}

function TabsTrigger({ value, children }: TabsTriggerProps) {
  const { activeTab, setActiveTab } = useTabsContext();
  const isActive = activeTab === value;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      className={`tabs__trigger ${isActive ? 'tabs__trigger--active' : ''}`}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

interface TabsPanelProps {
  value: string;
  children: React.ReactNode;
}

function TabsPanel({ value, children }: TabsPanelProps) {
  const { activeTab } = useTabsContext();
  if (activeTab !== value) return null;

  return (
    <div role="tabpanel" className="tabs__panel">
      {children}
    </div>
  );
}

// 6. Attach sub-components vào Tabs với namespace + displayName
Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Panel = TabsPanel;

// displayName giúp React DevTools hiển thị đúng tên
TabsList.displayName = 'Tabs.List';
TabsTrigger.displayName = 'Tabs.Trigger';
TabsPanel.displayName = 'Tabs.Panel';

export { Tabs };
```

#### TypeScript — Extend type để include sub-components

```tsx
// Cách type Tabs object có sub-components:
interface TabsComponent extends React.FC<TabsProps> {
  List: typeof TabsList;
  Trigger: typeof TabsTrigger;
  Panel: typeof TabsPanel;
}

const Tabs = (({ defaultValue, children }: TabsProps) => {
  // ... implementation
}) as TabsComponent;

Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Panel = TabsPanel;
```

#### So sánh với các pattern khác

| Pattern | Ưu điểm | Nhược điểm |
|---|---|---|
| Compound Components | Composable, clean API, consumer control | Cần Context, phức tạp hơn |
| HOC | Reuse logic, transparent | Wrapper hell, prop collisions |
| Render Props | Flexible | Verbose, JSX nesting sâu |
| Hooks | Simplest for logic reuse | Không handle layout composability |

---

### 🎤 Mock Interview — Q&A

**Q1: "What is the Compound Component pattern?"**

> It's a pattern where a group of components work together to form a complete UI, sharing implicit state through Context. The parent component owns the state and exposes it via a Context provider. Child components consume that context without the consumer needing to wire anything up explicitly. Think of how HTML's `<select>` and `<option>` work together — that's the mental model. In React, you'd implement it with `createContext` and a custom hook.

**Q2: "Implement a simple `<Tabs>` component using the compound component pattern."**

> You'd create a `TabsContext` with `activeTab` and `setActiveTab`. The root `<Tabs>` component holds state with `useState` and wraps children in `<TabsContext.Provider>`. Sub-components like `<Tabs.Trigger>` and `<Tabs.Panel>` consume the context via a `useTabsContext` hook. The trigger updates active tab on click; the panel renders only when its value matches `activeTab`. Finally, attach `Tabs.List`, `Tabs.Trigger`, and `Tabs.Panel` to the `Tabs` object for clean namespace access.

**Q3: "What's the advantage of compound components over passing everything as props?"**

> Flexibility and composability. When you pass everything as props (`tabs={[...]}`) you lock consumers into a fixed structure. With compound components, consumers can customize order, add extra elements between triggers, conditionally render panels, wrap a trigger in a tooltip — anything. The parent handles state, the children handle their own rendering. It's also easier to test each sub-component in isolation.

**Q4: "How do you expose compound components in TypeScript?"**

> You cast the root function as an interface that extends `React.FC<Props>` and adds properties for each sub-component. Then you attach the sub-components as properties on the function object. Adding `displayName` to each sub-component helps React DevTools show meaningful names in the component tree instead of anonymous functions.

---

## 35. Hydration — SSR to Client Handoff

**[Mid → Senior — Critical for Next.js roles]**

**EN:** What is hydration in React? What causes hydration mismatches, and how do you fix them?

**VI:** Hydration trong React là gì? Điều gì gây ra hydration mismatch và cách fix như thế nào?

### Trả lời

#### Hydration là gì?

Khi dùng SSR (Next.js, Remix...), server render component thành HTML string và gửi về browser. Người dùng thấy nội dung ngay lập tức. Nhưng HTML đó là **static** — không có event listeners, không có state.

**Hydration** là quá trình React chạy trên client, "attach" event listeners và internal state vào DOM đã tồn tại sẵn — **mà không tạo lại DOM từ đầu**.

```
SSR + Hydration Flow:
────────────────────────────────────────────────────────────
  [Server]
    React renders → HTML string → sent to browser
                                        ↓
  [Browser]
    Browser renders HTML immediately  ← user sees content fast
                                        ↓
    React bundle loads + executes
                                        ↓
    React "hydrates": walks the DOM, attaches event listeners,
    restores state — WITHOUT replacing existing HTML
                                        ↓
    App is now interactive             ← user can interact
────────────────────────────────────────────────────────────
```

#### Hydration Mismatch — Nguyên nhân

Mismatch xảy ra khi HTML server render **khác** với những gì React client sẽ render:

```tsx
// 1. Date.now() / Math.random() — khác nhau mỗi lần gọi:
function Timestamp() {
  return <span>Generated at: {Date.now()}</span>;
  // Server: 1703001234567  ≠  Client: 1703001234890 → MISMATCH
}

// 2. typeof window — chỉ có trên client:
function ClientOnlyBanner() {
  const isClient = typeof window !== 'undefined';
  return <div>{isClient ? 'Browser' : 'Server'}</div>;
  // Server: 'Server'  ≠  Client: 'Browser' → MISMATCH
}

// 3. localStorage / sessionStorage — không có trên server:
function ThemeProvider() {
  const theme = localStorage.getItem('theme') ?? 'light'; // 💥 crashes on server
  return <div data-theme={theme}>...</div>;
}

// 4. Browser extensions modifying DOM — ngoài tầm kiểm soát của bạn

// 5. Locale/timezone khác nhau giữa server và client
```

#### Cách React xử lý mismatch

| Version | Hành vi khi mismatch |
|---|---|
| React 17 | Throw error, destroy toàn bộ server HTML, re-render từ đầu → visible flash |
| React 18 | Selective hydration, attempt recovery, vẫn warn nhưng ít destructive hơn |
| React 19 | Smarter partial recovery, better error messages với diff |

#### Fix patterns

**Fix 1 — `useEffect` + mounted state** (component chỉ render client-side content sau hydration):

```tsx
function ClientOnlyTime() {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);  // Chạy sau hydration, chỉ trên client
  }, []);

  // Render placeholder khi chưa mounted (matches server output)
  if (!mounted) return <span>Loading...</span>;

  // Sau khi mounted, render client-only content
  return <span>Current time: {new Date().toLocaleTimeString()}</span>;
}
```

**Fix 2 — `suppressHydrationWarning`** (last resort, cho timestamps/IDs tự generate):

```tsx
function Timestamp() {
  return (
    <time suppressHydrationWarning>
      {new Date().toISOString()}
    </time>
  );
  // React bỏ qua mismatch trên element này — dùng cẩn thận!
}
```

**Fix 3 — `next/dynamic` với `ssr: false`** (Next.js — skip SSR hoàn toàn cho component):

```tsx
import dynamic from 'next/dynamic';

// Component này sẽ không được SSR — chỉ render trên client
const HeavyClientComponent = dynamic(
  () => import('./HeavyClientComponent'),
  { ssr: false }
);

function Page() {
  return (
    <div>
      <ServerRenderedContent />
      <HeavyClientComponent />  {/* Render sau hydration */}
    </div>
  );
}
```

**Fix 4 — `useIsomorphicLayoutEffect`** (đã đề cập ở section 30):

```tsx
// Thay vì useLayoutEffect (gây warning trên server), dùng:
const useIsomorphicLayoutEffect =
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;
```

#### Progressive Hydration với Streaming SSR (React 18+)

React 18 giới thiệu **streaming SSR** với multiple Suspense boundaries — server stream HTML theo từng phần, client hydrate từng phần độc lập:

```tsx
function Page() {
  return (
    <Layout>
      {/* Phần này hydrate trước — above the fold */}
      <HeroSection />

      {/* Phần này hydrate sau khi data ready */}
      <Suspense fallback={<ProductsSkeleton />}>
        <ProductList />  {/* Hydrate khi ready */}
      </Suspense>

      {/* Phần này hydrate cuối — below the fold */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments />
      </Suspense>
    </Layout>
  );
}
```

---

### 🎤 Mock Interview — Q&A

**Q1: "What is hydration in React/Next.js?"**

> Hydration is the process where React takes the static HTML rendered by the server and attaches event listeners and internal state to it on the client. The server sends HTML so the user sees content immediately, but that HTML is non-interactive. When the React bundle loads in the browser, React walks the existing DOM — rather than re-creating it — and "hydrates" it by binding all the event handlers and restoring component state. After hydration the app is fully interactive.

**Q2: "What causes hydration mismatches and how do you fix them?"**

> Mismatches happen when the HTML the server rendered differs from what React would render on the client. Common causes: using `Date.now()` or `Math.random()` which produce different values each call, reading `typeof window` or `localStorage` which don't exist on the server, or browser extensions that modify the DOM. Fixes depend on the cause: use a `mounted` state pattern with `useEffect` for browser-only content, use `suppressHydrationWarning` on an element for acceptable small differences like timestamps, or use `next/dynamic` with `ssr: false` to skip SSR entirely for a component.

**Q3: "You have a component that shows the current time — it works fine but throws a hydration error. How do you fix it?"**

> The time is different between server render and client render, causing a mismatch. Best fix: render a placeholder on the server and show the real time only after the component mounts. Use `const [mounted, setMounted] = useState(false)` and `useEffect(() => setMounted(true), [])`. Return a static placeholder when `!mounted`, and the actual time after mounting. This way, the server and initial client renders match exactly — the real time updates after hydration without any mismatch.

**Q4: "What changed in React 18 regarding hydration?"**

> React 18 introduced selective hydration and streaming SSR. Instead of waiting for the entire page's JavaScript to load before hydrating, React 18 can hydrate different parts of the page independently as their data becomes available, using multiple Suspense boundaries. If a user interacts with a not-yet-hydrated part of the page, React 18 prioritizes hydrating that part first. Also, mismatch recovery became smarter — React 18 tries to recover gracefully instead of destroying the entire server HTML tree on a mismatch.

---

## 36. useSyncExternalStore — Subscribing to External Stores

**[Senior]**

**EN:** What is `useSyncExternalStore`? What problem does it solve, and when would you use it directly?

**VI:** `useSyncExternalStore` là gì? Nó giải quyết vấn đề gì và khi nào bạn dùng nó trực tiếp?

### Trả lời

#### Vấn đề: "Tearing" trong Concurrent Mode

React 18's Concurrent Mode cho phép React **interrupt và resume** renders. Điều này có thể dẫn đến **tearing** — một vấn đề chỉ xảy ra với external mutable stores:

```
Tearing scenario:
────────────────────────────────────────────────────────────
  React bắt đầu render component tree
    → ComponentA reads store.value = 1  ✓
    → [React bị interrupt bởi urgent update]
    → External store changes: store.value = 2
    → React resumes render
    → ComponentB reads store.value = 2  ✓ (nhưng khác A!)
    → ComponentA và ComponentB hiển thị giá trị khác nhau
       cho cùng một data source → TEARING
────────────────────────────────────────────────────────────
```

Với React's built-in state (`useState`, `useReducer`), React biết khi nào state thay đổi và đảm bảo tính nhất quán. Nhưng với **external stores** (global variables, Redux store, browser APIs), React không biết.

#### `useEffect` + `useState` approach — Có vấn đề trong Concurrent Mode:

```tsx
// Pattern cũ — hoạt động đủ dùng nhưng có thể gây tearing:
function useWindowWidthOld() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}
// Trong Concurrent Mode, có thể có tearing giữa lần render đầu
// và khi effect chạy (sau paint)
```

#### `useSyncExternalStore` — Solution chính thức:

```tsx
import { useSyncExternalStore } from 'react';

function useSyncExternalStore<T>(
  subscribe: (onStoreChange: () => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T  // Optional — cho SSR
): T
```

**3 tham số:**
- `subscribe(callback)`: subscribe vào store, callback gọi khi store thay đổi, return unsubscribe function
- `getSnapshot()`: trả về giá trị hiện tại của store — **phải trả về cùng giá trị nếu store không thay đổi** (stable reference)
- `getServerSnapshot()`: optional, giá trị cho SSR

#### Ví dụ 1: `useWindowWidth`

```tsx
import { useSyncExternalStore } from 'react';

function subscribe(callback: () => void) {
  window.addEventListener('resize', callback);
  return () => window.removeEventListener('resize', callback);
}

function getSnapshot() {
  return window.innerWidth;
}

function getServerSnapshot() {
  return 0; // Reasonable default cho SSR (server không có window)
}

function useWindowWidth() {
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

// Sử dụng:
function ResponsiveComponent() {
  const width = useWindowWidth();
  return <div>{width >= 768 ? <DesktopLayout /> : <MobileLayout />}</div>;
}
```

#### Ví dụ 2: `useOnlineStatus`

```tsx
import { useSyncExternalStore } from 'react';

function subscribeOnline(callback: () => void) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  return useSyncExternalStore(
    subscribeOnline,
    () => navigator.onLine,    // client snapshot
    () => true                  // server snapshot (assume online)
  );
}

// Sử dụng:
function StatusBanner() {
  const isOnline = useOnlineStatus();
  return isOnline ? null : <div className="offline-banner">You are offline</div>;
}
```

#### Ví dụ 3: Simple custom store

```tsx
// Store đơn giản tự implement:
type Listener = () => void;

function createStore<T>(initialState: T) {
  let state = initialState;
  const listeners = new Set<Listener>();

  return {
    getState: () => state,
    setState: (newState: T | ((prev: T) => T)) => {
      state = typeof newState === 'function'
        ? (newState as (prev: T) => T)(state)
        : newState;
      listeners.forEach((l) => l());
    },
    subscribe: (listener: Listener) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    },
  };
}

const counterStore = createStore({ count: 0 });

function useCounterStore() {
  return useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getState
  );
}

// Sử dụng:
function Counter() {
  const { count } = useCounterStore();
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => counterStore.setState((s) => ({ count: s.count + 1 }))}>
        Increment
      </button>
    </div>
  );
}
```

#### Ai dùng `useSyncExternalStore`?

- **Zustand**, **Redux**, **Jotai**, **Valtio** — tất cả dùng hook này **internally** để đảm bảo tính nhất quán trong Concurrent Mode
- **Bạn** — khi subscribe trực tiếp vào browser APIs (window size, online status, matchMedia, history...)

> Trước React 18, các libraries dùng `useEffect` + `useState` pattern. Sau React 18, tất cả đã migrate sang `useSyncExternalStore` để tránh tearing.

---

### 🎤 Mock Interview — Q&A

**Q1: "Have you heard of `useSyncExternalStore`? What does it do?"**

> Yes. It's a React 18 hook for subscribing to external mutable stores in a way that's safe with Concurrent Mode. You provide three things: a `subscribe` function that registers a callback for when the store changes, a `getSnapshot` function that returns the current value, and an optional `getServerSnapshot` for SSR. React uses these to read the store synchronously during render, guaranteeing all components see the same value — preventing tearing.

**Q2: "What is the 'tearing' problem in React's Concurrent Mode?"**

> Tearing is when different parts of the UI show inconsistent data from the same source. In Concurrent Mode, React can pause a render mid-tree to handle a higher-priority update. If an external store mutates while React is paused, the resumed render reads the new value while earlier-rendered components used the old value. The UI ends up with two different values for the same data shown at the same time. `useSyncExternalStore` prevents this by reading the store synchronously and ensuring React sees a consistent snapshot throughout one render pass.

**Q3: "How does useSyncExternalStore relate to state management libraries like Zustand?"**

> All major state management libraries — Zustand, Redux (via react-redux), Jotai — use `useSyncExternalStore` internally. Their store is external mutable state outside of React's model. Before React 18, they used `useEffect` + `setState` patterns which were technically susceptible to tearing in Concurrent features. After React 18, they migrated to `useSyncExternalStore` to be concurrent-safe. As a developer, you'd use it directly when subscribing to browser APIs like window resize or online status — scenarios where you ARE the store author.

---

*Tài liệu này được cập nhật cho React 19 (released 12/2024) và React 18 Concurrent features. Câu hỏi được sắp xếp từ Junior đến Senior theo từng topic.*
