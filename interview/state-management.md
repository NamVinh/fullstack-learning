# State Management — Interview Questions (Junior → Senior)

## Table of Contents

1. [State Management là gì?](#1-state-management-là-gì)
2. [Khi nào dùng gì?](#2-khi-nào-dùng-gì)
3. [Context API](#3-context-api)
4. [Context Performance Issues](#4-context-performance-issues)
5. [Redux Core Concepts](#5-redux-core-concepts)
6. [Redux Toolkit](#6-redux-toolkit)
7. [Redux Async — createAsyncThunk vs RTK Query](#7-redux-async--createasyncthunk-vs-rtk-query)
8. [RTK Query](#8-rtk-query)
9. [RTK Query Tags & Optimistic Updates](#9-rtk-query-tags--optimistic-updates)
10. [Zustand](#10-zustand)
11. [TanStack Query — Cơ bản](#11-tanstack-query--cơ-bản)
12. [TanStack Query — Nâng cao](#12-tanstack-query--nâng-cao)
13. [Recoil / Jotai — Atom-based State](#13-recoil--jotai--atom-based-state)
14. [So sánh tất cả giải pháp](#14-so-sánh-tất-cả-giải-pháp)

---

## 1. State Management là gì?

**[Junior]**

> **EN:** What is state management? What is the difference between local state, global state, server state, and client state?
>
> **VI:** State management là gì? Phân biệt local state, global state, server state và client state?

**Trả lời:**

State management là cách tổ chức và quản lý dữ liệu (state) trong ứng dụng — bao gồm nơi lưu trữ, cách cập nhật, và cách chia sẻ giữa các component.

**Phân loại:**

| Loại | Mô tả | Ví dụ | Công cụ phù hợp |
|------|--------|-------|-----------------|
| **Local state** | Chỉ dùng trong một component | toggle modal, input value | `useState`, `useReducer` |
| **Global state** | Chia sẻ giữa nhiều component không liên quan | user đang đăng nhập, theme, giỏ hàng | Context, Zustand, Redux |
| **Server state** | Dữ liệu fetch từ API, có vòng đời riêng (loading/error/stale) | danh sách sản phẩm, profile | React Query, RTK Query, SWR |
| **Client state** | Dữ liệu chỉ tồn tại ở client, không sync với server | bộ lọc UI, bước wizard | useState, Zustand |

**Lý do cần phân biệt:** Server state có những vấn đề riêng như caching, refetching, synchronization mà global state tool thông thường không giải quyết tốt. Dùng Redux để quản lý server state thường dẫn đến boilerplate khổng lồ và bug tinh vi.

```typescript
// Local state — chỉ dùng trong component này
function Modal() {
  const [isOpen, setIsOpen] = useState(false);
  return <button onClick={() => setIsOpen(true)}>Open</button>;
}

// Server state — React Query lo caching, loading, refetch
function ProductList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });
}

// Global client state — Zustand
const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));
```

---

## 2. Khi nào dùng gì?

**[Junior → Mid]**

> **EN:** How do you decide which state management solution to use? Walk me through your decision process.
>
> **VI:** Bạn quyết định dùng giải pháp state management nào trong từng tình huống như thế nào?

**Trả lời:**

Dùng decision tree sau để chọn công cụ phù hợp:

```
Dữ liệu này có phải fetch từ API không?
├── Có → Dùng TanStack Query / RTK Query / SWR
│         (server state: caching, loading, error, refetch)
└── Không → Dữ liệu này có cần chia sẻ giữa nhiều component không?
            ├── Không → useState / useReducer (local)
            └── Có → App có phức tạp không?
                      ├── Nhỏ / vừa → Context API + useReducer
                      └── Lớn / nhiều team → Zustand hoặc Redux Toolkit
                          ├── Cần DevTools mạnh, team lớn, nhiều middleware → Redux Toolkit
                          └── Cần nhẹ, API đơn giản, ít boilerplate → Zustand
```

**Nguyên tắc thực tế:**

1. **useState** — Trạng thái UI cục bộ (form, toggle, animation step)
2. **useReducer** — Logic phức tạp trong một component (nhiều actions liên quan)
3. **Context API** — Theme, locale, auth user — dữ liệu thay đổi ít, đọc nhiều
4. **Zustand** — Global UI state, shopping cart, filter state — API gọn, không cần Provider
5. **Redux Toolkit** — Enterprise app, nhiều team, cần audit trail, middleware phức tạp
6. **TanStack Query / RTK Query** — Bất kỳ thứ gì từ server

```typescript
// Ví dụ: cùng một feature "user profile" — chọn đúng tool

// Sai: dùng Redux cho server state
dispatch(fetchUser(userId)); // boilerplate, phải tự quản lý loading/error

// Đúng: React Query lo hết
const { data: user } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => api.getUser(userId),
  staleTime: 5 * 60 * 1000, // cache 5 phút
});

// Đúng: Zustand cho UI state "tab nào đang active"
const useProfileStore = create((set) => ({
  activeTab: 'info',
  setActiveTab: (tab) => set({ activeTab: tab }),
}));
```

---

## 3. Context API

**[Junior]**

> **EN:** Explain the Context API. How do you create and use context? What are common mistakes?
>
> **VI:** Giải thích Context API. Cách tạo và dùng context? Những lỗi thường gặp là gì?

**Trả lời:**

Context API cho phép truyền dữ liệu xuống component tree mà không cần props drilling. Gồm 3 phần chính: `createContext`, `Provider`, `useContext`.

**Pattern chuẩn:**

```typescript
// 1. Tạo context với type đầy đủ
interface AuthContextType {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// 2. Tạo Provider component
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (credentials: Credentials) => {
    const user = await authApi.login(credentials);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
    authApi.logout();
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// 3. Custom hook để dùng context — bắt buộc có null check
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// 4. Dùng trong component
function Header() {
  const { user, logout } = useAuth();
  return <button onClick={logout}>Hello {user?.name}</button>;
}
```

**Lỗi thường gặp:**

```typescript
// Lỗi 1: Không bọc component trong Provider
// → runtime error: "useAuth must be used within AuthProvider"

// Lỗi 2: Tạo context với giá trị mặc định là null mà không check
const ThemeContext = createContext(null); // unsafe
const { theme } = useContext(ThemeContext); // có thể crash

// Lỗi 3: Tạo value object mới mỗi lần render
function Provider({ children }) {
  const [count, setCount] = useState(0);
  // Mỗi lần Provider re-render, object mới được tạo → tất cả consumer re-render
  return (
    <Ctx.Provider value={{ count, setCount }}>
      {children}
    </Ctx.Provider>
  );
}

// Fix: useMemo
const value = useMemo(() => ({ count, setCount }), [count]);
```

---

## 4. Context Performance Issues

**[Mid]**

> **EN:** Why do all context consumers re-render when context value changes? How do you fix this?
>
> **VI:** Tại sao tất cả consumer re-render khi context value thay đổi? Cách khắc phục?

**Trả lời:**

**Vấn đề:** Context dùng referential equality để check xem value có thay đổi không. Khi bất kỳ phần nào của value thay đổi, **tất cả** component dùng `useContext` với context đó đều re-render, kể cả component không dùng phần đó.

```typescript
// Vấn đề: count thay đổi → cả UserProfile cũng re-render dù không dùng count
const AppContext = createContext({ count: 0, user: null, theme: 'dark' });

function Counter() {
  const { count } = useContext(AppContext); // re-render khi count thay đổi ✓
  return <div>{count}</div>;
}

function UserProfile() {
  const { user } = useContext(AppContext); // re-render khi count thay đổi ✗
  return <div>{user?.name}</div>;
}
```

**Giải pháp 1: Tách Context theo domain**

```typescript
// Chia nhỏ context — mỗi context chỉ chứa state liên quan
const CountContext = createContext(0);
const UserContext = createContext<User | null>(null);
const ThemeContext = createContext<'light' | 'dark'>('dark');

// Giờ Counter chỉ re-render khi count thay đổi
function Counter() {
  const count = useContext(CountContext);
  return <div>{count}</div>;
}
```

**Giải pháp 2: Tách State và Dispatch Context**

```typescript
// Pattern phổ biến với useReducer
type Action = { type: 'INCREMENT' } | { type: 'SET_USER'; user: User };

const StateContext = createContext<AppState | undefined>(undefined);
const DispatchContext = createContext<Dispatch<Action> | undefined>(undefined);

export function AppProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <DispatchContext.Provider value={dispatch}>
      <StateContext.Provider value={state}>
        {children}
      </StateContext.Provider>
    </DispatchContext.Provider>
  );
}

// Component chỉ cần dispatch → không re-render khi state thay đổi
function AddButton() {
  const dispatch = useContext(DispatchContext);
  return <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>;
}
```

**Giải pháp 3: useMemo cho value**

```typescript
function Provider({ children }) {
  const [user, setUser] = useState<User | null>(null);
  const [theme, setTheme] = useState<'light' | 'dark'>('dark');

  // Chỉ tạo object mới khi user hoặc theme thực sự thay đổi
  const value = useMemo(
    () => ({ user, theme, setUser, setTheme }),
    [user, theme]
  );

  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}
```

**Lưu ý quan trọng:** Nếu app có performance issue thực sự với Context, đây là dấu hiệu nên chuyển sang Zustand — vốn có built-in selector để chỉ subscribe vào phần state cần thiết.

---

## 5. Redux Core Concepts

**[Mid]**

> **EN:** Explain Redux core concepts: store, actions, reducers, dispatch, selectors, and unidirectional data flow.
>
> **VI:** Giải thích các khái niệm cốt lõi của Redux: store, actions, reducers, dispatch, selectors và luồng dữ liệu một chiều.

**Trả lời:**

Redux dựa trên 3 nguyên tắc: **Single source of truth** (một store duy nhất), **State is read-only** (chỉ thay đổi qua action), **Changes are made with pure functions** (reducer phải pure).

**Luồng dữ liệu một chiều (Unidirectional Data Flow):**

```
UI → dispatch(action) → reducer(state, action) → new state → UI re-render
```

**Các khái niệm:**

```typescript
// 1. ACTION — plain object mô tả "điều gì đã xảy ra"
const addToCart = (product: Product) => ({
  type: 'cart/addItem' as const,
  payload: product,
});

// 2. REDUCER — pure function: (state, action) → new state
// KHÔNG được: mutate state, gọi API, Math.random(), Date.now()
function cartReducer(state = initialState, action: CartAction): CartState {
  switch (action.type) {
    case 'cart/addItem':
      return {
        ...state,
        items: [...state.items, action.payload],
        total: state.total + action.payload.price,
      };
    case 'cart/removeItem':
      return {
        ...state,
        items: state.items.filter((item) => item.id !== action.payload),
      };
    default:
      return state;
  }
}

// 3. STORE — nơi lưu toàn bộ state của app
import { createStore } from 'redux';
const store = createStore(cartReducer);

// 4. DISPATCH — cách duy nhất để thay đổi state
store.dispatch(addToCart({ id: 1, name: 'Phone', price: 999 }));

// 5. SELECTOR — function để đọc state (có thể memoized với reselect)
const selectCartItems = (state: RootState) => state.cart.items;
const selectCartTotal = (state: RootState) => state.cart.total;

// Memoized selector với reselect
import { createSelector } from 'reselect';
const selectExpensiveItems = createSelector(
  selectCartItems,
  (items) => items.filter((item) => item.price > 100)
  // Chỉ tính lại khi items thay đổi
);

// 6. Trong component
function CartSummary() {
  const dispatch = useDispatch();
  const items = useSelector(selectCartItems);
  const total = useSelector(selectCartTotal);

  return (
    <div>
      <p>Total: {total}</p>
      <button onClick={() => dispatch(addToCart(product))}>Add</button>
    </div>
  );
}
```

---

## 6. Redux Toolkit

**[Mid]**

> **EN:** What is Redux Toolkit? Explain createSlice, configureStore, and why RTK is preferred over vanilla Redux.
>
> **VI:** Redux Toolkit là gì? Giải thích createSlice, configureStore và tại sao RTK được ưu tiên hơn Redux thuần?

**Trả lời:**

Redux Toolkit (RTK) là bộ công cụ chính thức để viết Redux một cách ngắn gọn và an toàn hơn. RTK tích hợp sẵn Immer (cho phép "mutate" state trực tiếp), createSlice (tạo actions + reducer cùng lúc), và nhiều utilities khác.

**Tại sao dùng RTK thay vì vanilla Redux:**

| Vấn đề với vanilla Redux | RTK giải quyết thế nào |
|--------------------------|------------------------|
| Quá nhiều boilerplate | `createSlice` tạo actions + reducer cùng lúc |
| Phải spread state thủ công | Immer cho phép mutate trực tiếp |
| Cấu hình store phức tạp | `configureStore` có sẵn Redux DevTools, thunk |
| Không có async built-in | `createAsyncThunk` có sẵn |

```typescript
import { createSlice, configureStore, PayloadAction } from '@reduxjs/toolkit';

// createSlice — tạo reducer + action creators cùng một chỗ
const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [] as CartItem[],
    total: 0,
  },
  reducers: {
    // Immer cho phép "mutate" trực tiếp — thực ra vẫn immutable ở bên dưới
    addItem(state, action: PayloadAction<CartItem>) {
      state.items.push(action.payload); // OK với Immer!
      state.total += action.payload.price;
    },
    removeItem(state, action: PayloadAction<string>) {
      const index = state.items.findIndex((item) => item.id === action.payload);
      if (index !== -1) {
        state.total -= state.items[index].price;
        state.items.splice(index, 1);
      }
    },
    clearCart(state) {
      state.items = [];
      state.total = 0;
    },
  },
});

// Action creators được tự động tạo từ reducers
export const { addItem, removeItem, clearCart } = cartSlice.actions;

// configureStore — tự động thêm DevTools, thunk middleware
const store = configureStore({
  reducer: {
    cart: cartSlice.reducer,
    // Thêm reducer khác ở đây
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(myCustomMiddleware),
});

// Type-safe hooks
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
```

**Immer mutation vs return — quan trọng:**

```typescript
reducers: {
  // Cách 1: Mutate trực tiếp với Immer (không return)
  addItem(state, action: PayloadAction<CartItem>) {
    state.items.push(action.payload); // Immer xử lý
  },

  // Cách 2: Return new state (kiểu cũ, vẫn hợp lệ)
  addItemOld(state, action: PayloadAction<CartItem>) {
    return {
      ...state,
      items: [...state.items, action.payload],
    };
  },

  // SAI: Vừa mutate vừa return → Immer không biết làm gì
  addItemWrong(state, action: PayloadAction<CartItem>) {
    state.items.push(action.payload);
    return state; // ← ĐỪNG làm thế này
  },
}
```

---

## 7. Redux Async — createAsyncThunk vs RTK Query

**[Mid → Senior]**

> **EN:** How do you handle async operations in Redux? Compare createAsyncThunk and RTK Query.
>
> **VI:** Xử lý async trong Redux như thế nào? So sánh createAsyncThunk và RTK Query?

**Trả lời:**

Redux thuần không hỗ trợ async. Thunk middleware (có sẵn trong RTK) cho phép dispatch function thay vì plain object.

**createAsyncThunk — cho async operations thủ công:**

```typescript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

// Tạo thunk
export const fetchProducts = createAsyncThunk(
  'products/fetchAll', // action type prefix
  async (filters: ProductFilters, { rejectWithValue }) => {
    try {
      const response = await productApi.getAll(filters);
      return response.data;
    } catch (error) {
      // rejectWithValue để truyền error xuống rejected case
      return rejectWithValue(error.response.data);
    }
  }
);

// Slice xử lý 3 trạng thái: pending, fulfilled, rejected
const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [] as Product[],
    status: 'idle' as 'idle' | 'loading' | 'succeeded' | 'failed',
    error: null as string | null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.status = 'loading';
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload as string;
      });
  },
});

// Dùng trong component
function ProductList() {
  const dispatch = useAppDispatch();
  const { items, status, error } = useAppSelector((s) => s.products);

  useEffect(() => {
    dispatch(fetchProducts({ category: 'electronics' }));
  }, [dispatch]);

  if (status === 'loading') return <Spinner />;
  if (status === 'failed') return <Error message={error} />;
  return <List items={items} />;
}
```

**So sánh createAsyncThunk vs RTK Query:**

| | createAsyncThunk | RTK Query |
|--|-----------------|-----------|
| Boilerplate | Nhiều (pending/fulfilled/rejected) | Rất ít |
| Caching | Tự làm | Tự động |
| Invalidation | Tự làm | Tag-based |
| Loading state | Tự quản lý trong slice | Tự động |
| Dùng khi nào | Async không phải API call (WebSocket, file) | CRUD API calls |

**Khi nào dùng createAsyncThunk:** Upload file, WebSocket events, business logic phức tạp không phải API request đơn thuần.

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "You have a React app with user authentication and a product list that changes often. Which state management tools would you pick?"**

**Model Answer:**
"I'd split it into two concerns. For the product list — that's server data, so I'd use TanStack Query or RTK Query. It handles caching, loading states, and background refetching automatically, which saves a ton of boilerplate. For auth — the logged-in user is global client state that rarely changes, so I'd use Zustand or even just Context. The key insight is that server state and client state have very different needs: server state goes stale, needs sync with the backend, and has loading/error lifecycles, while client state is just data you own on the frontend."

**Trả lời (Tiếng Việt):**
"Mình sẽ tách ra thành hai vấn đề riêng. Danh sách sản phẩm là dữ liệu từ server, nên mình dùng TanStack Query hoặc RTK Query — nó tự lo caching, loading state, background refetch, tiết kiệm rất nhiều boilerplate. Còn thông tin user đang đăng nhập thì đó là global client state, thay đổi ít, nên dùng Zustand hoặc thậm chí Context là đủ. Điểm mấu chốt là server state và client state có bản chất rất khác nhau: server state có thể bị stale, cần sync với backend, có vòng đời loading/error — trong khi client state là dữ liệu mình tự quản lý ở phía frontend."

---

**Q2 (Mid): "Walk me through how you decide between Zustand and Redux Toolkit for a new project."**

**Model Answer:**
"My first question is: how large is the team and how complex is the app? For a startup or a small-to-medium app, I'd reach for Zustand — the API is tiny, there's no Provider setup, and you get selectors out of the box so components only re-render when their slice of state actually changes. For an enterprise project with many teams, Redux Toolkit makes more sense: the Redux DevTools are best-in-class, the mental model of unidirectional data flow is well understood by most engineers, and you can layer on middleware like saga or observable if needed. That said, in 2024-2025 the most common stack I see is TanStack Query plus Zustand — you get the best tool for each job without the Redux overhead."

**Trả lời (Tiếng Việt):**
"Câu hỏi đầu tiên mình hỏi là: team có bao nhiêu người và app phức tạp đến đâu? Với startup hoặc app vừa và nhỏ, mình chọn Zustand — API gọn, không cần Provider, có selector sẵn để component chỉ re-render khi đúng phần state nó cần thay đổi. Với dự án enterprise nhiều team thì Redux Toolkit hợp lý hơn: Redux DevTools mạnh nhất thị trường, tư duy unidirectional data flow ai cũng hiểu, và có thể gắn thêm middleware như saga hay observable khi cần. Thực tế trong 2024-2025, stack mình hay thấy nhất là TanStack Query cộng với Zustand — mỗi công cụ làm đúng việc của nó mà không cần kéo theo overhead của Redux."

---

**Q3 (Senior): "A colleague says 'just use Redux for everything including API calls.' How do you respond?"**

**Model Answer:**
"I'd push back on that, but diplomatically. The problem is that server state has fundamentally different characteristics: it lives on the server, it can become stale, it needs refetching on window focus or network reconnect, and it needs cache invalidation. If you manage all of that with Redux, you end up writing a ton of boilerplate — pending/fulfilled/rejected cases, loading flags, error handling, manual invalidation logic. RTK Query or TanStack Query solve all of that for free. I'd suggest treating Redux as the tool for shared client-side state — UI state, user preferences, wizard steps — and using a dedicated server state library for anything that touches an API. The cognitive overhead is actually lower because each tool has a clear, focused purpose."

**Trả lời (Tiếng Việt):**
"Mình sẽ phản biện, nhưng nhẹ nhàng thôi. Vấn đề là server state có đặc điểm hoàn toàn khác: nó nằm trên server, có thể bị stale, cần refetch khi user focus lại tab hoặc mạng reconnect, và cần cache invalidation. Nếu quản lý hết bằng Redux thì phải viết một đống boilerplate — các case pending/fulfilled/rejected, loading flag, error handling, logic invalidate thủ công. RTK Query hay TanStack Query giải quyết tất cả cái đó miễn phí. Mình sẽ đề xuất dùng Redux cho shared client state — UI state, user preferences, wizard steps — còn bất kỳ thứ gì chạm vào API thì dùng thư viện server state chuyên dụng. Thực ra cognitive overhead còn thấp hơn vì mỗi tool có mục đích rõ ràng."

---

---

## 8. RTK Query

**[Mid → Senior]**

> **EN:** Explain RTK Query. How do you define an API with createApi? What is the difference between query and mutation endpoints?
>
> **VI:** Giải thích RTK Query. Cách định nghĩa API với createApi? Phân biệt query và mutation endpoints?

**Trả lời:**

RTK Query là data fetching và caching tool được tích hợp trong Redux Toolkit. Nó tự động quản lý: loading state, caching, background refetching, cache invalidation — không cần viết reducer hay thunk thủ công.

```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const productsApi = createApi({
  reducerPath: 'productsApi', // key trong Redux store
  baseQuery: fetchBaseQuery({
    baseUrl: '/api',
    prepareHeaders: (headers, { getState }) => {
      // Thêm auth token tự động
      const token = (getState() as RootState).auth.token;
      if (token) headers.set('authorization', `Bearer ${token}`);
      return headers;
    },
  }),
  tagTypes: ['Product', 'User'], // khai báo tag types trước
  endpoints: (builder) => ({
    // QUERY — dùng để GET data (có cache)
    getProducts: builder.query<Product[], ProductFilters>({
      query: (filters) => ({
        url: '/products',
        params: filters,
      }),
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'Product' as const, id })),
              { type: 'Product', id: 'LIST' },
            ]
          : [{ type: 'Product', id: 'LIST' }],
    }),

    getProductById: builder.query<Product, string>({
      query: (id) => `/products/${id}`,
      providesTags: (result, error, id) => [{ type: 'Product', id }],
    }),

    // MUTATION — dùng để POST/PUT/DELETE (không cache, invalidate tags)
    createProduct: builder.mutation<Product, Partial<Product>>({
      query: (newProduct) => ({
        url: '/products',
        method: 'POST',
        body: newProduct,
      }),
      invalidatesTags: [{ type: 'Product', id: 'LIST' }],
    }),

    updateProduct: builder.mutation<Product, { id: string; data: Partial<Product> }>({
      query: ({ id, data }) => ({
        url: `/products/${id}`,
        method: 'PUT',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Product', id }],
    }),
  }),
});

// Auto-generated hooks
export const {
  useGetProductsQuery,
  useGetProductByIdQuery,
  useCreateProductMutation,
  useUpdateProductMutation,
} = productsApi;

// Thêm vào store
const store = configureStore({
  reducer: {
    [productsApi.reducerPath]: productsApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(productsApi.middleware),
});
```

**Sử dụng trong component:**

```typescript
function ProductList() {
  // Query: tự động fetch, cache, và refetch
  const { data: products, isLoading, isFetching, error } = useGetProductsQuery(
    { category: 'electronics' },
    {
      pollingInterval: 30000, // refetch mỗi 30 giây
      skip: !isLoggedIn,       // bỏ qua nếu chưa login
    }
  );

  // Mutation: trả về [triggerFn, { isLoading, error }]
  const [createProduct, { isLoading: isCreating }] = useCreateProductMutation();

  const handleCreate = async (data: Partial<Product>) => {
    try {
      await createProduct(data).unwrap(); // .unwrap() để catch error
      toast.success('Created!');
    } catch (err) {
      toast.error('Failed to create');
    }
  };

  if (isLoading) return <Spinner />;
  return <List products={products} />;
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "What's the difference between a query endpoint and a mutation endpoint in RTK Query?"**

**Model Answer:**
"A query is for reading data — GET requests. RTK Query caches the response, and the same query key anywhere in the app shares that cache. By default data is considered stale immediately, so it refetches on mount if it's been a while, but you can set `staleTime` to control that. A mutation is for write operations — POST, PUT, DELETE. Mutations don't cache anything, and their job is usually to invalidate the cache of related queries so those queries refetch fresh data. In code, the hooks look different too: `useGetProductsQuery` returns data directly, while `useCreateProductMutation` gives you a trigger function you call imperatively."

**Trả lời (Tiếng Việt):**
"Query dùng để đọc dữ liệu — tức là GET request. RTK Query cache response lại, và bất kỳ chỗ nào trong app dùng cùng query key thì đều share chung cache đó. Mặc định data được coi là stale ngay lập tức, nên nó sẽ refetch khi mount nếu đã qua một thời gian, nhưng mình có thể set `staleTime` để kiểm soát. Còn mutation thì dùng cho các thao tác ghi — POST, PUT, DELETE. Mutation không cache gì hết, nhiệm vụ chính của nó thường là invalidate cache của các query liên quan để những query đó tự fetch lại data mới. Về cú pháp hook cũng khác: `useGetProductsQuery` trả data trực tiếp, còn `useCreateProductMutation` trả về một trigger function mà mình gọi thủ công khi cần."

---

**Q2 (Mid): "What does the `skip` option do in RTK Query and when would you use it?"**

**Model Answer:**
"The `skip` option prevents a query from running at all. The most common case is when you have a dependency — for example, you're fetching a user's orders, but you need the user's ID first, and that comes from another query or from auth state. You'd write something like `skip: !userId`, so the orders query just sits idle until the ID is available. Another use case is feature flags or permissions — if a user doesn't have access to a section, you skip the query entirely instead of fetching data they can't see. It's cleaner than having conditional hook calls, which React doesn't allow."

**Trả lời (Tiếng Việt):**
"Option `skip` ngăn query chạy hoàn toàn. Trường hợp phổ biến nhất là khi bạn có dependency — ví dụ đang fetch orders của user, nhưng cần có user ID trước, mà ID đó lại đến từ một query khác hoặc từ auth state. Thì viết `skip: !userId`, query orders cứ ngồi chờ đến khi có ID. Một trường hợp khác là feature flag hoặc phân quyền — nếu user không có quyền xem một section, thì skip query luôn thay vì fetch data họ không nhìn thấy được. Cách này gọn hơn nhiều so với việc gọi hook có điều kiện, vì React không cho phép conditional hook calls."

---

**Q3 (Senior): "How does RTK Query handle caching — what happens when two components request the same endpoint with the same arguments?"**

**Model Answer:**
"RTK Query deduplicates requests automatically. If two components mount at the same time and both call `useGetUserQuery('123')`, only one network request goes out. The response is stored in the Redux store under that endpoint and argument combination, and both components read from that shared cache. RTK Query also tracks how many subscribers are using a given cache entry. When the last subscriber unmounts, it starts a timer — by default 60 seconds — and if no new subscriber appears before the timer runs out, it garbage collects that cache entry. You can tune this per endpoint with the `keepUnusedDataFor` option. This whole mechanism means you can have components freely query the same data without worrying about N+1 request problems."

**Trả lời (Tiếng Việt):**
"RTK Query tự động deduplicate request. Nếu hai component mount cùng lúc và cả hai đều gọi `useGetUserQuery('123')`, thì chỉ có một network request được gửi đi thôi. Response được lưu trong Redux store theo tổ hợp endpoint và argument, và cả hai component đọc từ cache chung đó. RTK Query cũng theo dõi có bao nhiêu subscriber đang dùng một cache entry. Khi subscriber cuối cùng unmount, nó bắt đầu đếm ngược — mặc định là 60 giây — nếu không có subscriber mới trước khi hết giờ thì nó garbage collect cache entry đó. Mình có thể chỉnh thời gian này per endpoint bằng option `keepUnusedDataFor`. Cơ chế này cho phép component thoải mái query cùng data mà không lo vấn đề N+1 request."

---

**Q4 (Senior): "You call a mutation to update a product, but the list of products on screen doesn't refresh. What's wrong and how do you fix it?"**

**Model Answer:**
"Almost certainly a missing `invalidatesTags` on the mutation. RTK Query's cache invalidation is tag-based — you declare which tags a query `provides`, and which tags a mutation `invalidates`. If the mutation doesn't invalidate the right tag, the query cache stays untouched. The fix is to add `invalidatesTags` to the mutation that matches the `providesTags` on the list query. For a product update, you'd invalidate `{ type: 'Product', id }` for that specific product, and the list query refetches because it provides that tag. If you're adding a new product, you'd invalidate `{ type: 'Product', id: 'LIST' }` — that's the LIST pattern — to trigger a full list refetch."

**Trả lời (Tiếng Việt):**
"Gần như chắc chắn là thiếu `invalidatesTags` trong mutation. Cache invalidation của RTK Query hoạt động theo tag — mình khai báo query `provides` tag gì, và mutation `invalidates` tag gì. Nếu mutation không invalidate đúng tag thì query cache giữ nguyên, không refetch. Cách fix là thêm `invalidatesTags` vào mutation sao cho khớp với `providesTags` của list query. Với update product thì invalidate `{ type: 'Product', id }` của sản phẩm đó, list query sẽ tự refetch vì nó provides tag đó. Còn nếu thêm product mới thì invalidate `{ type: 'Product', id: 'LIST' }` — đó là LIST pattern — để kích hoạt refetch toàn bộ danh sách."

---

---

## 9. RTK Query Tags & Optimistic Updates

**[Senior]**

> **EN:** Explain RTK Query's tag system: providesTags, invalidatesTags, tag shapes, the LIST pattern, and optimistic updates with onQueryStarted.
>
> **VI:** Giải thích hệ thống tag của RTK Query: providesTags, invalidatesTags, shape của tag, LIST pattern và optimistic updates với onQueryStarted.

**Trả lời:**

**Tag system** là cơ chế cache invalidation của RTK Query. Khi mutation thành công, nó invalidate các tag, và các query cung cấp những tag đó sẽ tự động refetch.

**Tag shapes:**

```typescript
// Tag có thể là string hoặc object { type, id }
providesTags: ['Product']                         // tất cả Product
providesTags: [{ type: 'Product', id: 'LIST' }]  // danh sách Product
providesTags: [{ type: 'Product', id: '123' }]   // Product cụ thể id=123
```

**LIST Pattern — quan trọng:**

```typescript
endpoints: (builder) => ({
  getProducts: builder.query<Product[], void>({
    query: () => '/products',
    // Cung cấp tag cho từng item VÀ tag LIST chung
    providesTags: (result) =>
      result
        ? [
            // Tag cho từng sản phẩm cụ thể
            ...result.map(({ id }) => ({ type: 'Product' as const, id })),
            // Tag LIST đại diện cho "danh sách sản phẩm"
            { type: 'Product' as const, id: 'LIST' },
          ]
        : [{ type: 'Product' as const, id: 'LIST' }],
  }),

  createProduct: builder.mutation<Product, Partial<Product>>({
    query: (body) => ({ url: '/products', method: 'POST', body }),
    // Tạo mới → invalidate LIST (vì danh sách đã thay đổi)
    invalidatesTags: [{ type: 'Product', id: 'LIST' }],
  }),

  updateProduct: builder.mutation<Product, { id: string; data: Partial<Product> }>({
    query: ({ id, data }) => ({ url: `/products/${id}`, method: 'PUT', body: data }),
    // Cập nhật → chỉ invalidate item đó (không refetch toàn bộ list)
    invalidatesTags: (result, error, { id }) => [{ type: 'Product', id }],
  }),

  deleteProduct: builder.mutation<void, string>({
    query: (id) => ({ url: `/products/${id}`, method: 'DELETE' }),
    // Xóa → invalidate cả item đó lẫn LIST
    invalidatesTags: (result, error, id) => [
      { type: 'Product', id },
      { type: 'Product', id: 'LIST' },
    ],
  }),
})
```

**Optimistic Updates với onQueryStarted:**

```typescript
updateProduct: builder.mutation<Product, { id: string; data: Partial<Product> }>({
  query: ({ id, data }) => ({
    url: `/products/${id}`,
    method: 'PUT',
    body: data,
  }),
  // Cập nhật cache ngay lập tức, không đợi server
  async onQueryStarted({ id, data }, { dispatch, queryFulfilled }) {
    // 1. Cập nhật cache ngay (optimistic)
    const patchResult = dispatch(
      productsApi.util.updateQueryData('getProducts', undefined, (draft) => {
        const product = draft.find((p) => p.id === id);
        if (product) Object.assign(product, data);
      })
    );

    try {
      // 2. Đợi server response
      await queryFulfilled;
      // Thành công → cache đã đúng, không cần làm gì thêm
    } catch {
      // 3. Thất bại → rollback cache về trạng thái cũ
      patchResult.undo();
    }
  },
}),
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Senior): "Explain optimistic updates. Why do you update the cache before the server responds?"**

**Model Answer:**
"The goal is to make the UI feel instant. Instead of waiting for the network round-trip before showing the user's change, you apply it to the local cache immediately, and then either confirm it when the server succeeds or roll it back if it fails. The pattern in RTK Query's `onQueryStarted` is: first, snapshot the current cache so you can restore it, then apply the optimistic patch with `updateQueryData`, then await the server response, and in the catch block call `patchResult.undo()` to revert. It's a UX technique — most of the time the server succeeds, so users see a zero-latency response. The rollback covers the minority failure case."

**Trả lời (Tiếng Việt):**
"Mục tiêu là làm cho UI cảm giác phản hồi tức thì. Thay vì đợi network round-trip xong mới hiện thay đổi cho user, mình áp dụng nó vào local cache ngay lập tức, rồi hoặc là xác nhận khi server thành công, hoặc là rollback nếu thất bại. Pattern trong `onQueryStarted` của RTK Query là: đầu tiên snapshot cache hiện tại để có thể restore, rồi apply optimistic patch bằng `updateQueryData`, rồi await server response, và trong catch block gọi `patchResult.undo()` để revert. Đây là kỹ thuật UX — phần lớn thời gian server thành công, nên user thấy response zero-latency. Rollback chỉ xử lý trường hợp thất bại là thiểu số."

---

**Q2 (Senior): "When would you use the LIST pattern in `providesTags` vs just tagging by ID?"**

**Model Answer:**
"They serve different purposes. When you tag individual items by ID — `{ type: 'Product', id: '123' }` — you're enabling surgical invalidation. A mutation that updates product 123 only invalidates that one cache entry, not the whole list. But when you create or delete an item, the list itself has changed — the total count, the ordering, which items appear. For those operations you need to invalidate the `LIST` tag to force a full list refetch. The best practice is to have list queries provide both the LIST tag and per-item ID tags. That way update mutations can target just the changed item, but create and delete mutations can invalidate the LIST to get the fresh count and ordering."

**Trả lời (Tiếng Việt):**
"Hai cái này phục vụ mục đích khác nhau. Khi tag từng item theo ID — `{ type: 'Product', id: '123' }` — mình đang bật chế độ invalidation chính xác theo từng item. Một mutation update product 123 chỉ invalidate đúng cache entry đó, không đụng vào list. Nhưng khi tạo mới hoặc xóa một item, bản thân danh sách đã thay đổi — tổng số lượng, thứ tự, item nào xuất hiện. Với các operation đó thì cần invalidate tag `LIST` để buộc refetch toàn bộ list. Best practice là list query nên provide cả tag LIST lẫn tag ID từng item. Như vậy update mutation chỉ target đúng item bị thay đổi, còn create và delete mutation invalidate LIST để lấy lại số lượng và thứ tự mới nhất."

---

---

## 10. Zustand

**[Mid]**

> **EN:** Explain Zustand. How do you create a store, use middleware (devtools, persist, immer), and structure large stores with slices? Compare with Redux.
>
> **VI:** Giải thích Zustand. Cách tạo store, dùng middleware, tổ chức large store với slice pattern? So sánh với Redux?

**Trả lời:**

Zustand là thư viện state management nhỏ gọn (~1KB) với API cực kỳ đơn giản. Không cần Provider, không cần boilerplate.

**Cơ bản:**

```typescript
import { create } from 'zustand';

interface CartStore {
  items: CartItem[];
  total: number;
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  clearCart: () => void;
}

const useCartStore = create<CartStore>((set, get) => ({
  items: [],
  total: 0,

  addItem: (item) =>
    set((state) => ({
      items: [...state.items, item],
      total: state.total + item.price,
    })),

  removeItem: (id) => {
    const item = get().items.find((i) => i.id === id);
    set((state) => ({
      items: state.items.filter((i) => i.id !== id),
      total: state.total - (item?.price ?? 0),
    }));
  },

  clearCart: () => set({ items: [], total: 0 }),
}));

// Dùng trong component — chỉ subscribe vào phần cần thiết
function CartBadge() {
  // Selector: chỉ re-render khi items.length thay đổi
  const itemCount = useCartStore((state) => state.items.length);
  return <Badge count={itemCount} />;
}
```

**Middleware:**

```typescript
import { create } from 'zustand';
import { devtools, persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useStore = create<CartStore>()(
  devtools(        // Redux DevTools
    persist(       // Lưu vào localStorage
      immer(       // Cho phép mutate trực tiếp như Immer
        (set) => ({
          items: [],
          addItem: (item) =>
            set((state) => {
              state.items.push(item); // mutate trực tiếp với Immer
            }),
        })
      ),
      {
        name: 'cart-storage',
        storage: createJSONStorage(() => localStorage),
        partialize: (state) => ({ items: state.items }), // chỉ persist items
      }
    ),
    { name: 'CartStore' } // tên trong DevTools
  )
);
```

**Slice Pattern cho large store:**

```typescript
// Tách thành các slice riêng
import { StateCreator } from 'zustand';

interface CartSlice {
  items: CartItem[];
  addItem: (item: CartItem) => void;
}

interface UISlice {
  isCartOpen: boolean;
  toggleCart: () => void;
}

type StoreState = CartSlice & UISlice;

const createCartSlice: StateCreator<StoreState, [], [], CartSlice> = (set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
});

const createUISlice: StateCreator<StoreState, [], [], UISlice> = (set) => ({
  isCartOpen: false,
  toggleCart: () => set((state) => ({ isCartOpen: !state.isCartOpen })),
});

// Kết hợp các slice
const useStore = create<StoreState>()((...a) => ({
  ...createCartSlice(...a),
  ...createUISlice(...a),
}));
```

**Zustand vs Redux:**

| | Zustand | Redux Toolkit |
|--|---------|---------------|
| Boilerplate | Rất ít | Vừa phải |
| Bundle size | ~1KB | ~40KB |
| DevTools | Có (devtools middleware) | Tốt hơn |
| Middleware | Đơn giản | Mạnh hơn (saga, observable) |
| Learning curve | Thấp | Cao hơn |
| Phù hợp | App vừa, team nhỏ | Enterprise, team lớn |

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "What is Zustand and why would you choose it over Context API?"**

**Model Answer:**
"Zustand is a small state management library — about 1 kilobyte — built around a hook-based API. The main reason to choose it over Context is performance: with Context, any change to the context value re-renders all consumers, even components that only care about part of the state. Zustand uses a subscription model with selectors, so a component only re-renders when the specific slice of state it subscribes to actually changes. The API is also much simpler — no Provider wrapper, no createContext boilerplate. You just call `create` with your state and actions, then use the hook wherever you need it. I usually reach for Zustand when I have global client state that changes frequently enough to cause performance issues with Context."

**Trả lời (Tiếng Việt):**
"Zustand là thư viện state management nhỏ — khoảng 1 kilobyte — xây dựng xung quanh hook-based API. Lý do chính để chọn thay Context là performance: với Context, bất kỳ thay đổi nào trong context value đều re-render tất cả consumer, kể cả những component chỉ quan tâm đến một phần state. Zustand dùng subscription model với selector, nên component chỉ re-render khi đúng phần state nó subscribe thực sự thay đổi. API cũng đơn giản hơn nhiều — không cần Provider wrapper, không cần boilerplate createContext. Chỉ cần gọi `create` với state và actions, rồi dùng hook ở bất kỳ đâu. Mình thường chuyển sang Zustand khi có global client state thay đổi đủ thường xuyên để gây ra performance issue với Context."

---

**Q2 (Mid): "How does the `persist` middleware work in Zustand? What would you NOT persist?"**

**Model Answer:**
"The `persist` middleware wraps your store and serializes state to a storage backend — usually `localStorage` — after every update. On initialization, it reads from storage first, so state survives page refreshes. You configure it with a `name` key for the storage key and optionally a `partialize` function to control exactly which parts of state get persisted. For a shopping cart, I'd persist the items array. What I would NOT persist: loading states, error states, derived data, or anything that should always start fresh. Also be careful with anything that contains non-serializable values like functions or class instances — those break JSON serialization. And sensitive data like auth tokens in localStorage is a security consideration, though some apps do it anyway."

**Trả lời (Tiếng Việt):**
"Middleware `persist` bọc store của mình lại và serialize state vào một storage backend — thường là `localStorage` — sau mỗi lần update. Khi khởi tạo, nó đọc từ storage trước, nên state tồn tại được qua các lần refresh trang. Mình cấu hình với key `name` cho storage key và tùy chọn hàm `partialize` để kiểm soát chính xác phần nào của state được persist. Với shopping cart thì mình persist mảng items. Những thứ KHÔNG nên persist: loading state, error state, derived data, hay bất cứ thứ gì cần bắt đầu fresh mỗi lần. Cũng cẩn thận với những thứ chứa non-serializable value như function hay class instance — những cái đó sẽ làm vỡ JSON serialization. Và dữ liệu nhạy cảm như auth token trong localStorage là vấn đề bảo mật cần cân nhắc."

---

**Q3 (Mid): "How do you prevent unnecessary re-renders with Zustand selectors?"**

**Model Answer:**
"The key is to only subscribe to the exact piece of state you need. If a component only shows the cart item count, you write `const count = useStore(state => state.items.length)` rather than `const { items } = useStore()`. Zustand compares the return value of the selector on each state change — by default with strict equality. If the selector returns the same value, the component doesn't re-render. The gotcha is with selectors that return objects or arrays — since those are new references on every call, Zustand will re-render even if the contents are the same. For that case you either select primitive values, or pass a custom equality function like `shallow` from Zustand's utilities."

**Trả lời (Tiếng Việt):**
"Chìa khóa là chỉ subscribe vào đúng phần state mình cần. Nếu component chỉ hiển thị số lượng item trong cart, thì viết `const count = useStore(state => state.items.length)` thay vì `const { items } = useStore()`. Zustand so sánh giá trị trả về của selector sau mỗi lần state thay đổi — mặc định dùng strict equality. Nếu selector trả về cùng giá trị thì component không re-render. Điểm cần chú ý là với selector trả về object hoặc array — vì mỗi lần gọi sẽ tạo reference mới, Zustand sẽ re-render dù nội dung không đổi. Trường hợp đó thì hoặc là select primitive value, hoặc truyền custom equality function như `shallow` từ Zustand utilities."

---

---

## 11. TanStack Query — Cơ bản

**[Junior → Mid]**

> **EN:** Explain TanStack Query (React Query). How do useQuery and useMutation work? Explain queryKey patterns, staleTime vs gcTime, enabled flag, and select transform.
>
> **VI:** Giải thích TanStack Query. useQuery và useMutation hoạt động như thế nào? queryKey patterns, staleTime vs gcTime, enabled flag, và select transform?

**Trả lời:**

TanStack Query (trước là React Query) là thư viện quản lý server state. Nó tự động lo: caching, background refetching, loading/error states, deduplication của requests.

**useQuery:**

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function ProductDetail({ productId }: { productId: string }) {
  const { data, isLoading, isError, error, isFetching } = useQuery({
    // queryKey — mảng unique xác định cache entry
    // Thay đổi bất kỳ element nào → refetch tự động
    queryKey: ['product', productId],
    queryFn: () => productApi.getById(productId),

    // staleTime: bao lâu data được coi là "fresh" (không refetch)
    // Default: 0 → luôn stale → refetch khi component mount
    staleTime: 5 * 60 * 1000, // 5 phút

    // gcTime (trước là cacheTime): bao lâu cache được giữ khi không có subscriber
    // Default: 5 phút
    gcTime: 10 * 60 * 1000, // 10 phút

    // enabled: khi nào mới fetch
    enabled: !!productId, // chỉ fetch khi productId tồn tại

    // select: transform data trước khi trả về component
    select: (data) => ({
      ...data,
      formattedPrice: `$${data.price.toFixed(2)}`,
    }),
  });

  // isLoading: không có data VÀ đang fetch
  // isFetching: đang fetch (kể cả khi đã có data cũ — background refetch)
  if (isLoading) return <Spinner />;
  if (isError) return <Error message={error.message} />;

  return <div>{data.name} — {data.formattedPrice}</div>;
}
```

**queryKey patterns:**

```typescript
// Đơn giản
useQuery({ queryKey: ['users'], queryFn: fetchUsers });

// Với ID
useQuery({ queryKey: ['user', userId], queryFn: () => fetchUser(userId) });

// Với filters — filters thay đổi → refetch tự động
useQuery({
  queryKey: ['products', { category, page, sortBy }],
  queryFn: () => fetchProducts({ category, page, sortBy }),
});

// Nested
useQuery({
  queryKey: ['user', userId, 'orders'],
  queryFn: () => fetchUserOrders(userId),
});
```

**useMutation:**

```typescript
function CreateProductForm() {
  const queryClient = useQueryClient();

  const { mutate, mutateAsync, isPending, error } = useMutation({
    mutationFn: (data: Partial<Product>) => productApi.create(data),

    onSuccess: (newProduct) => {
      // Invalidate list query để refetch
      queryClient.invalidateQueries({ queryKey: ['products'] });
      toast.success('Created!');
    },

    onError: (error) => {
      toast.error(error.message);
    },

    onSettled: () => {
      // Chạy dù success hay error
      setIsFormOpen(false);
    },
  });

  // mutate: fire-and-forget
  const handleSubmit = (data) => mutate(data);

  // mutateAsync: có thể await và catch
  const handleSubmitAsync = async (data) => {
    try {
      const product = await mutateAsync(data);
      router.push(`/products/${product.id}`);
    } catch (err) {
      console.error(err);
    }
  };
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What problem does TanStack Query solve that useState doesn't?"**

**Model Answer:**
"When you fetch data with `useState` and `useEffect`, you end up writing a lot of repetitive code: set loading to true, try/catch, set data or error, set loading to false. And you don't get caching — every time a component mounts, it fires a fresh request. You don't get background refetching when the user switches tabs and comes back. You don't get deduplication when two components need the same data. TanStack Query handles all of that automatically. You just define a query key and a fetch function, and you get back `data`, `isLoading`, and `error` with all the lifecycle management handled. It also has concepts like `staleTime` — data within that window is considered fresh and won't trigger a refetch — and `gcTime` — how long unused data stays in the cache before being garbage collected."

**Trả lời (Tiếng Việt):**
"Khi fetch data bằng `useState` và `useEffect`, mình phải viết rất nhiều code lặp đi lặp lại: set loading thành true, try/catch, set data hoặc error, set loading thành false. Và không có caching — mỗi lần component mount là gửi request mới. Không có background refetching khi user chuyển tab rồi quay lại. Không có deduplication khi hai component cần cùng một data. TanStack Query xử lý tất cả những thứ đó tự động. Mình chỉ cần định nghĩa query key và fetch function, rồi nhận lại `data`, `isLoading`, `error` với toàn bộ lifecycle management được handle sẵn. Nó còn có `staleTime` — data trong khoảng thời gian đó được coi là fresh và không trigger refetch — và `gcTime` — bao lâu data không dùng thì bị garbage collect khỏi cache."

---

**Q2 (Mid): "Explain the difference between `isLoading` and `isFetching` in TanStack Query."**

**Model Answer:**
"`isLoading` is true only when there's no cached data AND a fetch is in progress — it's the initial loading state. `isFetching` is true any time a fetch is happening, including background refetches when you already have data. This distinction is useful in the UI: for the initial load you want to show a full loading skeleton, but for a background refetch you might just show a subtle spinner in the corner so the user can still interact with the existing data. A common pattern is `if (isLoading) return <Skeleton />` and then separately `{isFetching && <RefreshIndicator />}` in the component."

**Trả lời (Tiếng Việt):**
"`isLoading` chỉ true khi không có cached data VÀ đang fetch — đây là trạng thái loading lần đầu. `isFetching` thì true bất cứ khi nào đang có fetch diễn ra, kể cả background refetch khi đã có data rồi. Sự phân biệt này rất hữu ích trên UI: khi load lần đầu thì mình muốn hiện full loading skeleton, nhưng khi background refetch thì chỉ cần hiện một spinner nhỏ ở góc để user vẫn tương tác được với data cũ. Pattern phổ biến là `if (isLoading) return <Skeleton />` và riêng phần `{isFetching && <RefreshIndicator />}` trong component."

---

**Q3 (Mid): "How does the `queryKey` array work? Why does changing a value in the key cause a refetch?"**

**Model Answer:**
"The query key is TanStack Query's cache key. It can be an array with any serializable values. When the key changes — say a filter param or an ID — TanStack Query treats it as a completely different cache entry and fetches fresh data. This is intentional and powerful: you can put all your query dependencies in the key, and the fetching behavior becomes declarative. For example, `['products', { category, page }]` — whenever `category` or `page` changes, the query automatically refetches. You don't have to write any `useEffect` with a dependency array to trigger the fetch. The convention is to go from general to specific: resource type first, then IDs, then filters."

**Trả lời (Tiếng Việt):**
"Query key là cache key của TanStack Query. Nó là một mảng chứa bất kỳ serializable value nào. Khi key thay đổi — ví dụ filter param hay ID — TanStack Query coi đó là cache entry hoàn toàn khác và fetch data mới. Đây là thiết kế có chủ đích và rất mạnh: mình đặt tất cả dependency của query vào key, và hành vi fetching trở thành declarative. Ví dụ `['products', { category, page }]` — bất cứ khi nào `category` hoặc `page` thay đổi, query tự động refetch. Không cần viết `useEffect` với dependency array để trigger fetch nữa. Convention là đi từ chung đến cụ thể: loại resource trước, rồi đến ID, rồi đến filter."

---

**Q4 (Senior): "Walk me through how you'd implement optimistic updates with TanStack Query's `useMutation`."**

**Model Answer:**
"The pattern uses the three callbacks on `useMutation`. In `onMutate`, which runs before the mutation fires, you first cancel any pending refetches for that query so they don't overwrite your optimistic state. Then you snapshot the current cache with `getQueryData`. Then you call `setQueryData` to apply the change immediately. You return the snapshot as context. In `onError`, which receives that context, you call `setQueryData` again with the snapshot to roll back. In `onSettled`, you call `invalidateQueries` to trigger a fresh fetch from the server regardless of success or failure — this syncs the cache to the ground truth. The reason you need to cancel pending refetches first is to avoid a race condition where a stale refetch comes back and overwrites your optimistic update."

**Trả lời (Tiếng Việt):**
"Pattern này dùng ba callback trong `useMutation`. Trong `onMutate` — chạy trước khi mutation được gửi đi — đầu tiên mình cancel các refetch đang pending của query đó để chúng không overwrite optimistic state. Rồi snapshot cache hiện tại bằng `getQueryData`. Rồi gọi `setQueryData` để apply thay đổi ngay lập tức. Return snapshot đó làm context. Trong `onError` nhận được context đó, mình gọi `setQueryData` lại với snapshot để rollback. Trong `onSettled`, gọi `invalidateQueries` để trigger fresh fetch từ server bất kể thành công hay thất bại — cái này sync cache về ground truth. Lý do cần cancel pending refetch trước là để tránh race condition, khi refetch cũ trả về và overwrite mất optimistic update của mình."

---

---

## 12. TanStack Query — Nâng cao

**[Senior]**

> **EN:** Explain optimistic updates, dependent queries, and infinite scroll with useInfiniteQuery in TanStack Query.
>
> **VI:** Giải thích optimistic updates, dependent queries và infinite scroll với useInfiniteQuery trong TanStack Query.

**Trả lời:**

**Optimistic Updates:**

```typescript
function TodoList() {
  const queryClient = useQueryClient();

  const updateTodo = useMutation({
    mutationFn: (todo: Todo) => todoApi.update(todo),

    // onMutate: chạy trước khi mutationFn, cập nhật cache ngay
    onMutate: async (updatedTodo) => {
      // Cancel outgoing refetches để tránh overwrite
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // Snapshot cache hiện tại để rollback
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);

      // Cập nhật cache optimistically
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map((todo) =>
          todo.id === updatedTodo.id ? updatedTodo : todo
        ) ?? []
      );

      // Trả về context để dùng trong onError
      return { previousTodos };
    },

    onError: (err, updatedTodo, context) => {
      // Rollback về snapshot cũ
      queryClient.setQueryData(['todos'], context?.previousTodos);
    },

    onSettled: () => {
      // Refetch để sync với server
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

**Dependent Queries — fetch tuần tự:**

```typescript
function UserOrders({ userId }: { userId: string }) {
  // Fetch user trước
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  // Chỉ fetch orders khi đã có user và user có địa chỉ
  const { data: orders } = useQuery({
    queryKey: ['orders', user?.id],
    queryFn: () => fetchOrders(user!.id),
    enabled: !!user?.id && user.hasAddress, // dependent!
  });

  // Parallel queries với useQueries
  const results = useQueries({
    queries: productIds.map((id) => ({
      queryKey: ['product', id],
      queryFn: () => fetchProduct(id),
    })),
  });
}
```

**Infinite Scroll với useInfiniteQuery:**

```typescript
function InfiniteProductList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteQuery({
    queryKey: ['products', 'infinite'],
    queryFn: ({ pageParam }) =>
      productApi.getPage({ page: pageParam, limit: 20 }),

    initialPageParam: 1,

    // Xác định pageParam cho lần fetch tiếp theo
    getNextPageParam: (lastPage, allPages) => {
      // lastPage.hasMore ? allPages.length + 1 : undefined
      return lastPage.nextCursor ?? undefined;
    },
  });

  // data.pages là mảng của các page responses
  const allProducts = data?.pages.flatMap((page) => page.items) ?? [];

  // Dùng với Intersection Observer để auto-load
  const { ref: bottomRef } = useIntersectionObserver({
    onChange: (isIntersecting) => {
      if (isIntersecting && hasNextPage && !isFetchingNextPage) {
        fetchNextPage();
      }
    },
  });

  return (
    <div>
      {allProducts.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
      <div ref={bottomRef} />
      {isFetchingNextPage && <Spinner />}
    </div>
  );
}
```

---

## 13. Recoil / Jotai — Atom-based State

**[Mid — Awareness]**

> **EN:** What is atom-based state management? Give a brief overview of Recoil and Jotai.
>
> **VI:** Atom-based state management là gì? Tổng quan ngắn về Recoil và Jotai?

**Trả lời:**

Atom-based state management chia nhỏ state thành các "atom" — đơn vị state độc lập nhỏ nhất. Component chỉ subscribe vào atom nó cần, tránh re-render không cần thiết.

**Jotai (nhẹ hơn, được ưa chuộng hơn Recoil hiện nay):**

```typescript
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Định nghĩa atoms
const countAtom = atom(0);
const userAtom = atom<User | null>(null);

// Derived atom (computed)
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Async atom
const userProfileAtom = atom(async (get) => {
  const user = get(userAtom);
  if (!user) return null;
  return await fetchUserProfile(user.id);
});

// Writable derived atom
const uppercaseNameAtom = atom(
  (get) => get(userAtom)?.name.toUpperCase(),
  (get, set, newName: string) => {
    const user = get(userAtom);
    if (user) set(userAtom, { ...user, name: newName.toLowerCase() });
  }
);

// Trong component — chỉ re-render khi atom này thay đổi
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubleCount = useAtomValue(doubleCountAtom);

  return (
    <div>
      <p>{count} × 2 = {doubleCount}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
    </div>
  );
}
```

**Recoil (của Meta, hiện ít được maintain hơn):**

```typescript
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil';

const cartItemsAtom = atom<CartItem[]>({
  key: 'cartItems', // key phải unique
  default: [],
});

const cartTotalSelector = selector({
  key: 'cartTotal',
  get: ({ get }) => {
    const items = get(cartItemsAtom);
    return items.reduce((sum, item) => sum + item.price, 0);
  },
});
```

**Khi nào dùng atom-based:**
- App có nhiều state độc lập, granular
- Muốn tránh re-render không cần thiết mà không cần tách context
- Không cần middleware phức tạp như Redux

---

## 14. So sánh tất cả giải pháp

**[Senior]**

> **EN:** Compare all state management solutions. When would you choose each one?
>
> **VI:** So sánh tất cả giải pháp state management. Khi nào chọn cái nào?

**Trả lời:**

**Bảng so sánh:**

| | Context API | Zustand | Redux Toolkit | TanStack Query | Jotai |
|--|-------------|---------|---------------|----------------|-------|
| **Bundle size** | 0 (built-in) | ~1KB | ~40KB | ~13KB | ~3KB |
| **Boilerplate** | Thấp | Rất thấp | Vừa | Rất thấp | Rất thấp |
| **DevTools** | Không | Có (Redux DevTools) | Tốt nhất | Có | Có |
| **Server state** | Không | Không | RTK Query có | Chuyên dụng | Không |
| **Performance** | Re-render nhiều | Selector tốt | Selector tốt | Không liên quan | Tốt nhất |
| **Learning curve** | Thấp | Thấp | Cao | Vừa | Thấp |
| **Middleware** | Không | Có | Mạnh nhất | Không | Ít |
| **TypeScript** | Vừa | Tốt | Tốt | Xuất sắc | Tốt |

**Recommendation theo use case:**

```typescript
// 1. Nhỏ, prototype, không cần global state nhiều
// → useState + Context (hoặc không dùng gì thêm)

// 2. App vừa, cần global client state
// → Zustand cho client state + TanStack Query cho server state
import { create } from 'zustand';
import { useQuery } from '@tanstack/react-query';

// 3. Enterprise, nhiều team, cần audit trail
// → Redux Toolkit + RTK Query
import { createSlice, createApi } from '@reduxjs/toolkit';

// 4. App cần granular reactivity, nhiều state atoms
// → Jotai + TanStack Query
import { atom } from 'jotai';
import { useQuery } from '@tanstack/react-query';
```

**Stack phổ biến năm 2024-2025:**

```
Startup / App vừa:
  TanStack Query (server state) + Zustand (client state)

Enterprise:
  RTK Query (server state) + Redux Toolkit (client state) + Redux DevTools

Next.js / Server Components:
  React Server Components (server state) + Zustand / Jotai (client state)
```

**Nguyên tắc chọn:**

1. **Không over-engineer** — dùng tool đơn giản nhất đáp ứng nhu cầu
2. **Tách server state và client state** — luôn dùng tool chuyên dụng cho server state
3. **Đánh giá team** — Redux tốt hơn khi team lớn, nhiều người mới cần học codebase
4. **Đánh giá app hiện tại** — migration cost so với lợi ích

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "If you're starting a greenfield React app, what state management stack would you recommend and why?"**

**Model Answer:**
"For most apps today I'd start with TanStack Query for server state and Zustand for global client state. TanStack Query gives you caching, background sync, loading/error states, and optimistic updates without any setup beyond a QueryClient provider. Zustand covers shared UI state like sidebar open/closed, user preferences, or cart contents — it's about 1KB, no providers needed, and the selector API means components only re-render when their specific piece of state changes. I wouldn't reach for Redux unless the team is large and needs strong conventions, the app requires time-travel debugging heavily, or you're already invested in the RTK Query ecosystem. Context is fine for truly static things like theme or locale, but not for frequently changing data."

**Trả lời (Tiếng Việt):**
"Với hầu hết app hiện nay, mình sẽ bắt đầu với TanStack Query cho server state và Zustand cho global client state. TanStack Query cho mình caching, background sync, loading/error state, và optimistic update mà không cần setup gì nhiều ngoài QueryClient provider. Zustand lo shared UI state như sidebar mở/đóng, user preference, cart contents — khoảng 1KB, không cần provider, và selector API giúp component chỉ re-render khi đúng phần state của nó thay đổi. Mình sẽ không với tay lấy Redux trừ khi team lớn cần conventions chặt chẽ, app yêu cầu time-travel debugging nhiều, hoặc đã đầu tư vào RTK Query ecosystem rồi. Context thì ổn cho những thứ gần như tĩnh như theme hay locale, nhưng không phù hợp với data thay đổi thường xuyên."

---

**Q2 (Mid): "When does using React's built-in Context become a performance problem?"**

**Model Answer:**
"Context re-renders every consumer whenever the context value changes — and 'changes' means reference inequality. So if your provider creates a new object on every render, every consumer re-renders even if the actual data is the same. The bigger problem is granularity: if you put user, theme, cart, and notification state all in one context, a cart update re-renders components that only care about the theme. The fixes are: split contexts by domain, memoize the value object with `useMemo`, or if you're seeing real performance issues, switch to Zustand which has built-in selectors. Context is great for things like auth user or locale that barely change and are read everywhere — it's not great for frequently updating state."

**Trả lời (Tiếng Việt):**
"Context re-render tất cả consumer mỗi khi context value thay đổi — và 'thay đổi' ở đây là reference inequality. Nên nếu provider tạo object mới mỗi lần render, tất cả consumer đều re-render dù data thực sự không đổi. Vấn đề lớn hơn là độ granular: nếu nhét user, theme, cart, và notification state vào một context, thì một lần update cart sẽ re-render cả những component chỉ quan tâm đến theme. Cách fix là: tách context theo domain, memoize value object bằng `useMemo`, hoặc nếu đang gặp performance issue thực sự thì chuyển sang Zustand vốn có built-in selector. Context rất tốt cho những thứ như auth user hay locale gần như không thay đổi và được đọc ở khắp nơi — không phù hợp cho state thay đổi thường xuyên."

---

**Q3 (Senior): "Describe a real scenario where you'd use Redux Toolkit's `createAsyncThunk` over RTK Query."**

**Model Answer:**
"RTK Query is optimized for REST-style request/response patterns — you define an endpoint, it handles the lifecycle. But `createAsyncThunk` is better when the async operation isn't a simple HTTP request. For example: a multi-step file upload where you track progress and can cancel mid-upload, or a WebSocket message handler that updates state in response to server push events, or a complex sequence where you need to read from the current Redux state mid-operation and conditionally dispatch different actions. RTK Query has `onQueryStarted` for side effects, but for complex orchestration logic that involves multiple dispatches or reading state, `createAsyncThunk` with `getState` gives you more control. Another case: server-sent events or polling with custom backoff logic where you're managing the fetch lifecycle yourself."

**Trả lời (Tiếng Việt):**
"RTK Query được tối ưu cho pattern request/response kiểu REST — mình define endpoint, nó lo vòng đời. Nhưng `createAsyncThunk` tốt hơn khi async operation không phải là HTTP request đơn giản. Ví dụ: upload file nhiều bước có theo dõi progress và có thể cancel giữa chừng, hoặc WebSocket message handler cập nhật state khi server push event, hoặc một chuỗi phức tạp cần đọc Redux state giữa chừng và dispatch các action khác nhau tùy điều kiện. RTK Query có `onQueryStarted` cho side effect, nhưng với logic orchestration phức tạp cần nhiều dispatch hoặc đọc state, `createAsyncThunk` với `getState` cho mình nhiều control hơn. Trường hợp khác: server-sent event hay polling với custom backoff logic khi mình tự quản lý fetch lifecycle."

---

**Q4 (Senior): "How would you architect state management for a large Next.js app with both server components and client components?"**

**Model Answer:**
"The key insight with Next.js App Router is that server components run on the server and can fetch data directly — no need for any client-side fetching library for the initial data. So the architecture splits cleanly: for data that can be fetched on the server, I use React Server Components with direct database or API calls, and pass data down as props. For client-side interactivity — mutations, optimistic updates, real-time sync — I'd use TanStack Query with its Next.js prefetching support so the server-fetched data hydrates the client cache seamlessly. For global client state that's truly UI-only — things like modals, shopping cart, user preferences — I'd use Zustand with the `persist` middleware where needed. The goal is to push as much data fetching to the server as possible, and only bring in client-side state management where you actually need interactivity."

**Trả lời (Tiếng Việt):**
"Điểm mấu chốt với Next.js App Router là server component chạy trên server và có thể fetch data trực tiếp — không cần client-side fetching library cho initial data. Nên architecture tách ra khá rõ: với data có thể fetch trên server, mình dùng React Server Components với direct database hoặc API call, rồi pass data xuống qua props. Với client-side interactivity — mutation, optimistic update, real-time sync — mình dùng TanStack Query với prefetching support của Next.js để data fetch từ server hydrate client cache một cách liền mạch. Với global client state thuần UI — modal, shopping cart, user preference — mình dùng Zustand với `persist` middleware khi cần. Mục tiêu là đẩy càng nhiều data fetching lên server càng tốt, chỉ đưa client-side state management vào khi thực sự cần interactivity."

---

---

*Tài liệu này được tối ưu cho phỏng vấn Frontend/Fullstack Engineer. Cập nhật lần cuối: 2025.*
