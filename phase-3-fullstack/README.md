# Phase 3 — Fullstack Integration

**Thời gian**: 4–5 tuần | **Mục tiêu**: Hiểu FE và BE hoạt động cùng nhau như thế nào

## Checklist

### API Design nâng cao
- [ ] REST best practices: versioning, pagination (cursor vs offset), filtering
- [ ] GraphQL cơ bản — nguyên lý, so sánh với REST
- [ ] tRPC — type-safe API với TypeScript monorepo
- [ ] Swagger/OpenAPI documentation
- [ ] Error response format chuẩn (RFC 7807)

### Real-time & WebSockets
- [ ] WebSocket protocol vs HTTP
- [ ] Socket.io — chat / notification
- [ ] Server-Sent Events (SSE)
- [ ] Long polling — tại sao deprecated

### Next.js
- [ ] App Router vs Pages Router
- [ ] React Server Components vs Client Components
- [ ] Server Actions
- [ ] SSR, SSG, ISR, PPR — khi nào dùng gì
- [ ] API Routes / Route Handlers
- [ ] Middleware
- [ ] Streaming & Suspense

### Fullstack Authentication
- [ ] Token storage: localStorage vs httpOnly cookie
- [ ] Refresh token rotation
- [ ] NextAuth / Auth.js
- [ ] Session management phía server
- [ ] CSRF protection

### File Storage
- [ ] Upload từ FE (presigned URL pattern)
- [ ] S3 / Cloudflare R2 / Supabase Storage
- [ ] Image optimization (CDN, Next.js Image)

### Monorepo & Code Sharing
- [ ] Turborepo — cấu trúc monorepo
- [ ] Share TypeScript types FE ↔ BE
- [ ] Share Zod schemas FE ↔ BE
- [ ] Shared ESLint/Prettier

### State Management
- [ ] Server state vs Client state
- [ ] TanStack Query — caching, stale time, invalidation, optimistic updates
- [ ] Zustand cho client state

### DX Tools
- [ ] MSW (Mock Service Worker) — mock API trong development
- [ ] Storybook basics

## Tài nguyên
- [Next.js Docs](https://nextjs.org/docs)
- [tRPC Docs](https://trpc.io/docs)
- [TanStack Query](https://tanstack.com/query/latest)
- [Turborepo](https://turbo.build/repo/docs)
- [Socket.io Docs](https://socket.io/docs/v4/)

## Project: Project Management Tool
- Boards, columns, tasks (Trello-like nhưng nhỏ)
- Real-time updates khi user khác thay đổi
- Auth đầy đủ + file attachments
- Monorepo: `apps/web` + `apps/api` + `packages/types`
- Deploy: Vercel + Railway/Fly.io
