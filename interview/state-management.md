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

---

*Tài liệu này được tối ưu cho phỏng vấn Frontend/Fullstack Engineer. Cập nhật lần cuối: 2025.*
