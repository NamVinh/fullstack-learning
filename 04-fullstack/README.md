# 04 — Fullstack Integration

> Đây là phần quan trọng nhất. FE + BE không phải 2 thứ riêng biệt — phải hiểu chúng nói chuyện với nhau như thế nào, auth xuyên suốt, và deploy thành 1 hệ thống hoàn chỉnh.

---

## Junior — Kết nối FE + BE cơ bản

### Next.js Full-stack

| Topic | Phải làm được |
|-------|---------------|
| App Router | File-based routing, layouts, loading/error UI |
| RSC vs Client Components | Khi nào `"use client"`, data fetching trong server component |
| Server Actions | Form mutation không cần API route, `revalidatePath` |
| API Route Handlers | `app/api/route.ts` — khi nào cần REST endpoints |
| SSR / SSG / ISR | Cấu hình per-page, `revalidate`, `dynamic` |
| Middleware | Edge middleware — auth check, redirect, header injection |

### Data Fetching

| Topic | Phải hiểu |
|-------|-----------|
| TanStack Query | `useQuery`, `useMutation`, `queryClient.invalidateQueries` |
| Stale time vs Cache time | Hiểu cache lifecycle, khi nào refetch |
| Optimistic updates | `onMutate`, rollback `onError` — UX trick quan trọng |
| Server state vs Client state | Phân biệt rõ — không dùng Zustand cho server state |

### Type-safe API

| Topic | Phải làm được |
|-------|---------------|
| tRPC | `router`, `procedure`, `useQuery`/`useMutation` từ client |
| Zod schemas dùng chung | Validate ở cả FE lẫn BE từ cùng 1 schema |
| Share TypeScript types | Monorepo — FE và BE dùng chung type definitions |
| OpenAPI / Swagger | Document REST API, auto-generate client types |

### Auth Fullstack

| Topic | Phải hiểu + implement |
|-------|----------------------|
| Token storage | `localStorage` vs `httpOnly cookie` — trade-offs bảo mật |
| CSRF protection | Khi dùng cookie-based auth, `SameSite=Strict/Lax` |
| Refresh token rotation | Implement đúng, handle race condition (multiple requests) |
| NextAuth / Auth.js | Providers, callbacks, session, JWT strategy |

**Checkpoint Junior**: Next.js app với tRPC, auth (NextAuth), TanStack Query, shared Zod types.

---

## Mid — Real-time, Files, Monorepo

### Real-time

| Topic | Phải biết khi nào dùng |
|-------|------------------------|
| WebSocket protocol | Persistent bidirectional connection — khác HTTP chỗ nào |
| Socket.io | `emit`, `on`, rooms, namespaces — implement chat/notification |
| Server-Sent Events (SSE) | One-way push — nhẹ hơn WebSocket, dùng khi chỉ cần server → client |
| Long polling | Tại sao đã bị bỏ, khi nào vẫn hữu dụng (fallback) |

### File Storage

| Topic | Phải làm được |
|-------|---------------|
| Presigned URL pattern | FE upload trực tiếp lên S3 — server không phải proxy file |
| AWS S3 | `PutObject`, `GetObject`, bucket policy, CORS config |
| Cloudflare R2 | S3-compatible, không tốn phí egress — alternative tốt |
| Image optimization | CDN, `next/image` internals, WebP conversion |
| File validation | Type checking (magic bytes, không chỉ extension), size limit |

### Monorepo

| Topic | Phải làm được |
|-------|---------------|
| Turborepo | Workspace setup, `turbo.json`, pipeline caching |
| Shared packages | `packages/types`, `packages/ui`, `packages/config` |
| Shared Zod schemas | Validate ở FE + BE từ cùng 1 source |
| Shared ESLint + Prettier | Config chỉ viết 1 lần, apply cho cả repo |

### Developer Experience

| Topic | Phải biết |
|-------|-----------|
| MSW (Mock Service Worker) | Mock API ở browser — phát triển FE độc lập với BE |
| Storybook | Document + test UI components independently |

**Checkpoint Mid**: Trello-like app hoàn chỉnh — real-time (Socket.io), file upload (S3), monorepo (Turborepo), auth đầy đủ.

---

## Resources

- [nextjs.org/docs](https://nextjs.org/docs)
- [trpc.io/docs](https://trpc.io/docs)
- [tanstack.com/query](https://tanstack.com/query)
- [authjs.dev](https://authjs.dev) — Auth.js (NextAuth v5)
- [turbo.build/repo/docs](https://turbo.build/repo/docs)
- Theo (t3.gg) YouTube — T3 Stack thực tế
