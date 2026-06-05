# 03 — Frontend Developer

> Bạn đã có 5 năm React/TypeScript. Phase này **lấp lỗ hổng** và đưa lên Senior FE level.
> Không học lại từ đầu — tập trung vào những gì bạn dùng mà chưa thực sự hiểu.

---

## Junior — Core Web (review nếu có gap)

### HTML

| Topic | Phải biết |
|-------|-----------|
| Semantic HTML | `<header>`, `<main>`, `<article>`, `<section>`, `<nav>`, `<aside>` — dùng đúng |
| Accessibility (a11y) | ARIA roles, `aria-label`, `aria-describedby`, keyboard navigation |
| Forms | `<input>` types, validation attributes, `<fieldset>`, `<label>` |

### CSS

| Topic | Phải làm được |
|-------|---------------|
| Flexbox + Grid | Tất cả properties, responsive layouts, auto-placement |
| Specificity | Tính điểm cascade, khi nào cần `!important` (và tại sao không nên) |
| CSS Variables | `--custom-props`, theming, responsive design tokens |
| Responsive | Mobile-first, media queries, `clamp()` |

### JavaScript Core

| Topic | Phải giải thích được |
|-------|---------------------|
| Event Loop | Call stack, microtask queue, macrotask queue — thứ tự thực thi |
| Prototype chain | `__proto__`, `Object.create`, class là syntactic sugar |
| Closure | Tại sao tồn tại, memory implications, common pitfall với loop |
| `this` | 4 binding rules: default, implicit, explicit (`call/apply/bind`), `new` |
| ES Modules | Named vs default export, dynamic `import()`, tree shaking |

### TypeScript

| Topic | Phải dùng được hàng ngày |
|-------|--------------------------|
| Generics | `<T>`, constraints `T extends`, multiple generics |
| Mapped Types | `[K in keyof T]`, `Partial`, `Required`, `Pick`, `Record` |
| Conditional Types | `T extends U ? X : Y`, `infer` keyword |
| Utility Types | `Omit`, `Exclude`, `Extract`, `ReturnType`, `Parameters` |
| Template Literal Types | `\`${string}-${number}\`` |

---

## Mid — Framework Depth + Vue 3

### React — Lấp lỗ hổng

| Topic | Phải hiểu thực sự |
|-------|-------------------|
| React Fiber | Reconciliation, diffing algorithm, commit phase |
| React 18 Concurrent | `startTransition`, `useTransition`, `useDeferredValue` — tại sao cần |
| Suspense | Với data fetching, lazy loading, Streaming SSR |
| `memo` / `useMemo` / `useCallback` | Khi nào **KHÔNG** nên dùng — over-optimization là bug |
| Custom hooks | Composability, separation of concerns |
| Error boundaries | Tại sao vẫn phải dùng class, đặt ở đâu trong tree |
| Render patterns | Compound components, render props, headless components |
| Strict Mode | Tại sao render 2 lần — detect unintended side effects |

### Vue 3 — Học từ đầu (Composition API)

| Topic | Phải hiểu sâu |
|-------|---------------|
| Reactivity system | `ref` vs `reactive` — proxy-based, khác React state chỗ nào |
| `computed()` | Lazy evaluation, caching, writable computed |
| `watch` vs `watchEffect` | Khi nào dùng cái nào, immediate, deep |
| Lifecycle hooks | `onMounted`, `onUnmounted`, `onBeforeUpdate` — timing |
| Composables | Tương đương custom hooks, naming `use*` |
| `defineProps` / `defineEmits` | Typed với TypeScript |
| Slots | Default, named, scoped slots (`v-slot`) |
| `provide` / `inject` | Dependency injection trong component tree |
| Teleport | Render ra ngoài DOM parent |
| Vue Router 4 | Dynamic routes, navigation guards, lazy loading |
| Pinia | `defineStore`, state, getters, actions |

### Build Tools

| Topic | Phải hiểu |
|-------|-----------|
| Vite | Tại sao nhanh: native ESM dev server + esbuild |
| Vite config | `plugins`, `resolve.alias`, `proxy`, `envPrefix` |
| Tree shaking | Named exports quan trọng hơn default — tại sao |
| Bundle analysis | `rollup-plugin-visualizer` — xem gì chiếm dung lượng |

**Checkpoint Mid**: Build cùng 1 Kanban app bằng React và Vue 3 — so sánh reactivity model của 2 framework.

---

## Senior — Performance, SSR, Modern CSS

### React 19

| Topic | Phải hiểu |
|-------|-----------|
| `use()` hook | Unwrap promise trong render, Suspense integration |
| Server Actions | Form mutations không cần API route |
| React Compiler | Tự động memoization — impact đến codebase |
| RSC vs Client Components | Khi nào `"use client"`, tại sao mặc định là server |

### Nuxt.js 3

| Topic | Phải làm được |
|-------|---------------|
| File-based routing | Tự động tạo route từ folder structure |
| Auto-imports | Components, composables không cần import thủ công |
| SSR / SSG / Hybrid | Cấu hình rendering mode per-route |
| Server routes | `server/api/` — full-stack trong 1 project |

### Performance

| Topic | Phải đo + tối ưu được |
|-------|----------------------|
| Core Web Vitals | LCP < 2.5s, CLS < 0.1, INP < 200ms — đo bằng Lighthouse |
| Code splitting | Route-based, component-level `lazy()` / `defineAsyncComponent()` |
| Bundle analysis | Identify và loại bỏ heavy dependencies |
| Image optimization | `srcset`, WebP/AVIF, lazy loading, `priority` |
| Font | `font-display: swap`, preload critical fonts |

### CSS Hiện đại

| Topic | Phải biết |
|-------|-----------|
| Container Queries | `@container` — responsive theo parent, không phải viewport |
| `@layer` | Quản lý specificity layers: reset, base, components, utilities |
| CSS Nesting | Native nesting (không cần Sass) |
| `:has()` selector | Chọn parent dựa trên children |

### PWA

| Topic | Phải biết |
|-------|-----------|
| Service Worker | Install → activate → fetch lifecycle |
| Cache strategies | Cache First, Network First, Stale While Revalidate |
| Web App Manifest | `manifest.json`, installable PWA |

---

## Resources

- [react.dev](https://react.dev) — React docs mới, hooks-first
- [vuejs.org/guide](https://vuejs.org/guide) — Vue 3 official (rất tốt)
- [nuxt.com/docs](https://nuxt.com/docs)
- [web.dev](https://web.dev) — Performance, Core Web Vitals (Google)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook)
- [The Missing Semester — Shell Tools](https://missing.csail.mit.edu/2020/shell-tools/)
