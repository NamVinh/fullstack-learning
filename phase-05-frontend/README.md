# Phase 0.5 — Hoàn thiện FE

**Thời gian**: 3–4 tuần | Có thể học song song với Phase 1

## Checklist

### React nâng cao — Lấp lỗ hổng
- [ ] React internals: Fiber, reconciliation, diffing algorithm
- [ ] React 18: concurrent rendering, `useTransition`, `useDeferredValue`, Suspense
- [ ] React 19: `use()` hook, Server Actions, React compiler
- [ ] Performance: biết khi nào KHÔNG dùng `memo`/`useMemo`/`useCallback`
- [ ] Custom hooks nâng cao — composition pattern
- [ ] Error boundaries — đặt ở đâu, khi nào cần
- [ ] Render patterns: compound components, headless components
- [ ] React strict mode — tại sao render 2 lần

### Vue 3 — Từ đầu
- [ ] Vue 3 vs React — mental model: reactivity system vs virtual DOM
- [ ] `ref`, `reactive`, `computed`, `watch`, `watchEffect`
- [ ] Composition API: `setup()`, lifecycle hooks
- [ ] `defineProps`, `defineEmits`, slots, provide/inject, teleport
- [ ] Vue Router 4: dynamic routes, navigation guards, lazy loading
- [ ] Pinia — state management
- [ ] Vue 3 + TypeScript: typed props, typed emits, typed composables
- [ ] Nuxt.js 3: SSR/SSG, file-based routing, server routes, auto-imports

### Build Tools
- [ ] Vite — tại sao nhanh (native ESM, esbuild)
- [ ] Vite config: plugins, aliases, env, proxy
- [ ] Bundle analysis: vite-bundle-visualizer
- [ ] Tree shaking và named exports

### FE nâng cao
- [ ] Accessibility (a11y): WCAG 2.1 AA, ARIA, keyboard navigation
- [ ] Core Web Vitals: đo lường LCP, CLS, INP thực tế
- [ ] CSS hiện đại: Container Queries, `@layer`, design tokens
- [ ] i18n: react-i18next / vue-i18n patterns

## Tài nguyên

**React:**
- [React Docs — react.dev](https://react.dev) — hoàn toàn mới, có nhiều advanced content
- [Making Sense of React Server Components](https://www.joshwcomeau.com/react/server-components/)
- [Why React Re-Renders](https://www.joshwcomeau.com/react/why-react-re-renders/)

**Vue 3:**
- [Vue.js Official Guide](https://vuejs.org/guide/introduction.html) — bắt đầu từ đây
- [Vue Mastery — Intro to Vue 3 (free)](https://www.vuemastery.com/courses/intro-to-vue-3/intro-to-vue3)
- [Nuxt 3 Docs](https://nuxt.com/docs)

**Build Tools:**
- [Vite Why](https://vitejs.dev/guide/why.html)

## Project
Build cùng 1 app (Todo + filter + pagination) bằng **cả React lẫn Vue**.

```
react-project/   ← React + TypeScript + Vite
vue-project/     ← Vue 3 + TypeScript + Vite
```

Mục tiêu: thấy rõ sự khác nhau về cách tư duy giữa 2 framework.

| Feature | React | Vue |
|---------|-------|-----|
| State | `useState` | `ref` / `reactive` |
| Computed | `useMemo` | `computed` |
| Side effects | `useEffect` | `watch` / `watchEffect` |
| Global state | Zustand | Pinia |
| Routing | React Router | Vue Router |

## Notes

> Vue 3 Composition API rất giống React hooks về tư duy.
> Điểm khác biệt lớn nhất: Vue **reactive by default** (mutate trực tiếp được),
> React phải tạo state mới (`setState`).
