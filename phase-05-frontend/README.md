# Phase 0.5 — Hoàn thiện FE

**Thời gian**: 3–4 tuần | Có thể học song song với Phase 1

---

## React nâng cao — Lấp lỗ hổng

---

### React Internals: Fiber, Reconciliation

- [ ] Hiểu tại sao React re-render — debug performance mới được

**Học:**
1. 📖 [React Fiber Architecture — acdlite](https://github.com/acdlite/react-fiber-architecture) — bản gốc, đọc overview
2. 📖 [Why React Re-Renders — Josh Comeau](https://www.joshwcomeau.com/react/why-react-re-renders/) — thực tế nhất

**Exercises:**

🛠️ Cài React DevTools → bật "Highlight updates when components render" → mở 1 app React của bạn → click mọi thứ → quan sát cái gì re-render không cần thiết.

🛠️ Giải thích tại sao component con này re-render mặc dù props không đổi:
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const style = { color: 'red' }; // object literal mỗi render
  return <Child style={style} />;
}
// Fix nó mà không dùng memo/useMemo
```

---

### React 18: Concurrent Features

- [ ] `useTransition` và `useDeferredValue` — không chỉ biết tên, phải biết khi nào dùng

**Học:**
1. 📖 [React 18 — New Hooks — Official Blog](https://react.dev/blog/2022/03/29/react-v18)
2. 📖 [useTransition — react.dev](https://react.dev/reference/react/useTransition)
3. 📖 [A Visual Guide to React Rendering — Alex Sidorenko](https://alexsidorenko.com/blog/react-render-always-rerenders/)

**Exercises:**

🛠️ Build một search input lọc danh sách 10,000 items:
- Version 1: không có gì → UI bị lag
- Version 2: `useDeferredValue` cho list → input vẫn responsive
- Version 3: `useTransition` để wrap filter operation

So sánh UX của 3 version. Giải thích sự khác biệt giữa `useTransition` và `useDeferredValue`.

---

### Performance: khi nào KHÔNG nên dùng memo

- [ ] Dùng `memo` sai còn tệ hơn không dùng

**Học:**
1. 📖 [Before You memo() — Dan Abramov](https://overreacted.io/before-you-memo/)
2. 📖 [When to useMemo and useCallback — Kent C. Dodds](https://kentcdodds.com/blog/usememo-and-usecallback)

**Exercises:**

🛠️ Với mỗi đoạn code dưới đây, quyết định: memo CÓ giúp ích không? Tại sao?

```jsx
// Case 1
const value = useMemo(() => 2 + 2, []);

// Case 2
const filteredList = useMemo(
  () => bigList.filter(item => item.active),
  [bigList]
);

// Case 3
const Child = memo(({ onClick }) => <button onClick={onClick}>Click</button>);
function Parent() {
  return <Child onClick={() => console.log('click')} />;
  // memo có hoạt động không? Fix nếu không.
}
```

---

### Custom Hooks nâng cao

- [ ] Hook = logic tái sử dụng — biết tách đúng chỗ

**Học:**
1. 📖 [Custom Hooks — react.dev](https://react.dev/learn/reusing-logic-with-custom-hooks)
2. 📖 [Thinking in React Hooks — Amelia Wattenberger](https://wattenberger.com/blog/react-hooks)

**Exercises:**

🛠️ Viết `useLocalStorage<T>(key, initialValue)` — state sync với localStorage, type-safe.

🛠️ Viết `useDebounce<T>(value, delay)` — trả về giá trị debounced.

🛠️ Viết `useFetch<T>(url)` trả về `{ data, loading, error }` — handle cleanup khi unmount (AbortController).

---

### Render Patterns: Compound Components & Headless

- [ ] Pattern này xuất hiện trong mọi UI library lớn (Radix, Headless UI)

**Học:**
1. 📖 [Compound Components — Kent C. Dodds](https://kentcdodds.com/blog/compound-components-with-react-hooks)
2. 📖 [Headless UI components](https://www.merrickchristensen.com/articles/headless-user-interface-components/)

**Exercises:**

🛠️ Build `<Tabs>` component theo pattern Compound Components:
```jsx
// API mong muốn:
<Tabs defaultValue="tab1">
  <Tabs.List>
    <Tabs.Trigger value="tab1">Tab 1</Tabs.Trigger>
    <Tabs.Trigger value="tab2">Tab 2</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="tab1">Content 1</Tabs.Content>
  <Tabs.Content value="tab2">Content 2</Tabs.Content>
</Tabs>
```

---

## Vue 3 — Học từ đầu

---

### Vue 3 Reactivity System

- [ ] Đây là điểm khác biệt lớn nhất giữa Vue và React — hiểu kỹ

**Học:**
1. 📖 [Vue 3 Official Guide — Reactivity Fundamentals](https://vuejs.org/guide/essentials/reactivity-fundamentals.html)
2. 📖 [Vue 3 — Computed Properties](https://vuejs.org/guide/essentials/computed.html)
3. 📖 [Vue 3 — Watchers](https://vuejs.org/guide/essentials/watchers.html)

**Exercises:**

🛠️ So sánh side-by-side với React — viết cùng 1 counter component:
```vue
<!-- Vue 3 version -->
<script setup>
import { ref, computed } from 'vue'
const count = ref(0)
const doubled = computed(() => count.value * 2)
</script>
```
```jsx
// React version
const [count, setCount] = useState(0)
const doubled = useMemo(() => count * 2, [count])
```
Giải thích: tại sao Vue dùng `.value` còn React không cần?

🛠️ Phân biệt khi nào dùng `ref` vs `reactive`. Viết ví dụ cho từng trường hợp.

🛠️ Phân biệt `watch` vs `watchEffect` — viết ví dụ cho từng trường hợp.

---

### Composition API: Components & Communication

- [ ] Props, emits, slots, provide/inject — Vue's way

**Học:**
1. 📖 [Vue 3 — Components Basics](https://vuejs.org/guide/essentials/component-basics.html)
2. 📖 [Vue 3 — Props](https://vuejs.org/guide/components/props.html)
3. 📖 [Vue 3 — Events (emits)](https://vuejs.org/guide/components/events.html)
4. 📖 [Vue 3 — Slots](https://vuejs.org/guide/components/slots.html)
5. 📖 [Vue 3 — Provide/Inject](https://vuejs.org/guide/components/provide-inject.html)

**Exercises:**

🛠️ Build `<Modal>` component với Vue 3:
```vue
<!-- Usage mong muốn -->
<Modal :open="isOpen" @close="isOpen = false">
  <template #header>Tiêu đề</template>
  <template #default>Nội dung</template>
  <template #footer>
    <button @click="isOpen = false">Đóng</button>
  </template>
</Modal>
```

🛠️ Dùng `provide/inject` để tạo Theme context (tương đương React Context):
```ts
// Trong parent: provide('theme', { primary: '#007bff' })
// Trong child bất kỳ: const theme = inject('theme')
```

---

### Vue Router 4 & Pinia

- [ ] Routing + State management — 2 thứ cần có trong mọi app Vue thực tế

**Học:**
1. 📖 [Vue Router 4 — Getting Started](https://router.vuejs.org/guide/)
2. 📖 [Vue Router — Navigation Guards](https://router.vuejs.org/guide/advanced/navigation-guards.html)
3. 📖 [Pinia — Getting Started](https://pinia.vuejs.org/getting-started.html)
4. 📖 [Pinia vs Vuex — tại sao chọn Pinia](https://pinia.vuejs.org/introduction.html#comparison-with-vuex)

**Exercises:**

🛠️ Setup Vue Router với:
- Public routes: `/`, `/login`
- Protected routes: `/dashboard` — redirect về `/login` nếu chưa đăng nhập
- Navigation guard kiểm tra auth token

🛠️ Tạo Pinia store `useAuthStore`:
```ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const isLoggedIn = computed(() => user.value !== null)
  async function login(email: string, password: string) { /* ... */ }
  function logout() { /* ... */ }
  return { user, isLoggedIn, login, logout }
})
```

---

### Vue 3 + TypeScript

- [ ] Vue 3 có TypeScript support tốt — nhưng cần biết cú pháp đúng

**Học:**
1. 📖 [Vue 3 — TypeScript with Composition API](https://vuejs.org/guide/typescript/composition-api.html)
2. 📖 [Vue 3 — Typed Component Props](https://vuejs.org/guide/typescript/composition-api.html#typing-component-props)

**Exercises:**

🛠️ Viết component `UserCard.vue` với typed props:
```ts
// Dùng cả 2 cách và hiểu sự khác nhau:

// Cách 1: defineProps với type literal
const props = defineProps<{
  user: { id: number; name: string; email: string }
  showAvatar?: boolean
}>()

// Cách 2: withDefaults
const props = withDefaults(defineProps<{...}>(), { showAvatar: true })
```

🛠️ Viết typed composable `useCounter(initialValue: number)` trả về `{ count: Ref<number>, increment: () => void, decrement: () => void }`.

---

### Nuxt.js 3 Overview

- [ ] Vue's Next.js — quan trọng cho fullstack với Vue stack

**Học:**
1. 📖 [Nuxt 3 — Introduction](https://nuxt.com/docs/getting-started/introduction)
2. 📖 [Nuxt 3 — Routing (file-based)](https://nuxt.com/docs/getting-started/routing)
3. 📖 [Nuxt 3 — Data Fetching](https://nuxt.com/docs/getting-started/data-fetching)
4. 📖 [Nuxt 3 — Server Routes](https://nuxt.com/docs/guide/directory-structure/server)

**Exercises:**

🛠️ Tạo Nuxt 3 app đơn giản:
```bash
npx nuxi@latest init my-nuxt-app
```
- Tạo trang `/posts` fetch data từ JSONPlaceholder
- Tạo trang `/posts/[id]` — dynamic route
- Tạo server API route `/api/hello` trả về JSON
- Thử `useFetch` vs `$fetch` — khác nhau chỗ nào?

---

## Build Tools: Vite

---

### Vite Deep Dive

- [ ] Biết tại sao nhanh — không phải chỉ "dùng Vite vì nó nhanh"

**Học:**
1. 📖 [Vite — Why Vite?](https://vitejs.dev/guide/why.html) — đọc hết, ngắn thôi
2. 📖 [Vite — Configuring Vite](https://vitejs.dev/config/)

**Exercises:**

🛠️ Tạo Vite project với React + TypeScript:
```bash
npm create vite@latest my-app -- --template react-ts
```
Sau đó cấu hình `vite.config.ts`:
```ts
export default defineConfig({
  resolve: {
    alias: { '@': path.resolve(__dirname, './src') }
  },
  server: {
    proxy: { '/api': 'http://localhost:3000' }
  }
})
```

🛠️ Bundle analysis — xem app của bạn nặng cỡ nào:
```bash
npm install -D rollup-plugin-visualizer
# Thêm vào vite.config.ts, build, xem report
```
Tìm 1 package nặng nhất và tìm cách thay thế hoặc lazy load.

---

## FE nâng cao

---

### Accessibility (a11y)

- [ ] Nhiều công ty yêu cầu — và là dấu hiệu của senior FE dev

**Học:**
1. 📖 [WebAIM — Introduction to Web Accessibility](https://webaim.org/intro/)
2. 📖 [MDN — ARIA basics](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/WAI-ARIA_basics)
3. 📖 [A11y Checklist — a11yproject.com](https://www.a11yproject.com/checklist/)

**Exercises:**

🛠️ Audit app của bạn:
1. Cài Lighthouse (có sẵn trong Chrome DevTools)
2. Chạy Accessibility audit
3. Fix 3 lỗi đầu tiên tìm thấy

🛠️ Test keyboard navigation: tắt chuột, chỉ dùng Tab và Enter — app có dùng được không?

---

### Core Web Vitals

- [ ] Đo thực tế, không chỉ đọc lý thuyết

**Học:**
1. 📖 [Web.dev — Core Web Vitals](https://web.dev/vitals/)
2. 📖 [Optimize LCP — web.dev](https://web.dev/articles/optimize-lcp)

**Exercises:**

🛠️ Đo app thực tế:
```bash
# Cài lighthouse CLI
npm install -g lighthouse
lighthouse https://your-app.vercel.app --output html --output-path report.html
open report.html
```

🛠️ Tìm cách cải thiện LCP (Largest Contentful Paint) trong 1 app React của bạn. Áp dụng ít nhất 1 optimization.

---

## Project Phase 0.5

Build cùng 1 app bằng **cả React lẫn Vue**:

**App**: Todo List với:
- Thêm/xóa/sửa todo
- Filter: All / Active / Completed
- Persist xuống localStorage
- Animate khi add/remove (CSS transition)

```
react-project/    ← React 18 + TypeScript + Vite + custom hooks
vue-project/      ← Vue 3 + TypeScript + Vite + Pinia
```

**Sau khi xong, so sánh:**

| Concept | React | Vue |
|---------|-------|-----|
| Local state | `useState` | `ref` |
| Computed | `useMemo` | `computed` |
| Side effects | `useEffect` | `watch` / `watchEffect` |
| Global state | Zustand | Pinia |
| Lifecycle on mount | `useEffect(fn, [])` | `onMounted` |
| Template logic | JSX | `v-if`, `v-for` |
| Two-way binding | Manually (value + onChange) | `v-model` |

Viết 1 đoạn ngắn (~200 từ) trong `notes/react-vs-vue.md` về điểm bạn thấy thú vị nhất khi so sánh 2 framework.
