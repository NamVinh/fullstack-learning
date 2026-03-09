# Phase 3 — Fullstack Integration

**Thời gian**: 4–5 tuần | **Mục tiêu**: FE và BE không còn 2 thứ tách biệt

---

## Next.js Deep Dive

---

### App Router & Server Components

- [ ] Đây là tư duy mới nhất của React — hiểu đúng ngay từ đầu

**Học:**
1. 📖 [Next.js — App Router Introduction](https://nextjs.org/docs/app)
2. 📖 [Making Sense of React Server Components — Josh Comeau](https://www.joshwcomeau.com/react/server-components/) — bài hay nhất giải thích RSC
3. 📖 [Next.js — Server and Client Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)

**Exercises:**

🛠️ Tạo Next.js 14 app và thực hành phân biệt Server vs Client Component:
```tsx
// app/posts/page.tsx — Server Component (mặc định)
// Fetch data trực tiếp, không cần useEffect
async function PostsPage() {
  const posts = await fetch('https://jsonplaceholder.typicode.com/posts')
    .then(r => r.json());
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}

// app/posts/search.tsx — Client Component (có interactivity)
'use client';
function SearchBar({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('');
  // ...
}
```

🛠️ Vẽ sơ đồ: component nào là Server, cái nào là Client trong 1 trang typical (header, sidebar, feed, like button, comment form).

---

### SSR, SSG, ISR — Khi nào dùng gì

- [ ] Chọn sai rendering strategy → SEO kém hoặc performance kém

**Học:**
1. 📖 [Next.js — Rendering Strategies](https://nextjs.org/docs/app/building-your-application/rendering)
2. 📖 [When to use SSR vs SSG — Vercel Blog](https://vercel.com/blog/nextjs-server-side-rendering-vs-static-generation)

**Exercises:**

🛠️ Build 3 trang trong cùng app, mỗi trang dùng 1 strategy khác nhau:
```tsx
// 1. SSG — Blog post (content không đổi)
// Dùng generateStaticParams để pre-generate

// 2. SSR — Dashboard (data real-time, theo user)
// Fetch trong Server Component mà không cache

// 3. ISR — Product listing (cập nhật mỗi 60 giây)
fetch(url, { next: { revalidate: 60 } })
```

Inspect Network tab → thấy sự khác biệt trong response headers (`x-nextjs-cache: HIT/MISS/STALE`).

---

### Server Actions

- [ ] Form mutation không cần API route — tư duy mới

**Học:**
1. 📖 [Next.js — Server Actions and Mutations](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)

**Exercises:**

🛠️ Viết Server Action cho form tạo post — không cần `POST /api/posts`:
```tsx
// app/posts/new/page.tsx
async function createPost(formData: FormData) {
  'use server';
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // Validate
  // Lưu DB trực tiếp
  await prisma.post.create({ data: { title, content, authorId: ... } });

  // Revalidate cache
  revalidatePath('/posts');
  redirect('/posts');
}

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="content" />
      <button type="submit">Tạo post</button>
    </form>
  );
}
```

---

## Real-time với WebSockets

---

### WebSocket Protocol & Socket.io

- [ ] Build tính năng real-time — chat, notification, live update

**Học:**
1. 📖 [WebSocket — MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
2. 📖 [Socket.io — Get Started](https://socket.io/get-started/chat)
3. 📖 [WebSocket vs SSE — khi nào dùng gì](https://ably.com/blog/websockets-vs-sse)

**Exercises:**

🛠️ Build real-time notification system với Socket.io:

**Server (Node.js):**
```ts
import { Server } from 'socket.io';

io.on('connection', (socket) => {
  // User join vào room theo userId
  socket.on('join', (userId: string) => {
    socket.join(`user:${userId}`);
  });
});

// Khi có sự kiện (ví dụ: ai đó like bài viết)
function notifyUser(userId: string, notification: Notification) {
  io.to(`user:${userId}`).emit('notification', notification);
}
```

**Client (React):**
```tsx
'use client';
function useNotifications(userId: string) {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  useEffect(() => {
    const socket = io();
    socket.emit('join', userId);
    socket.on('notification', (notif) => setNotifications(prev => [notif, ...prev]));
    return () => { socket.disconnect(); };
  }, [userId]);
  return notifications;
}
```

🛠️ So sánh: Implement cùng tính năng bằng **SSE** (Server-Sent Events). Nhận xét: khi nào nên dùng SSE thay WebSocket?

---

## Authentication End-to-End

---

### Token Storage & Security

- [ ] Cookie vs localStorage — đây là câu hỏi phỏng vấn kinh điển

**Học:**
1. 📖 [Where to Store Tokens — Auth0](https://auth0.com/docs/secure/security-guidance/data-security/token-storage)
2. 📖 [CSRF Attack Prevention — OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

**Exercises:**

🛠️ Implement 2 strategies và so sánh:

**Strategy 1: httpOnly Cookie**
```ts
// Server: set cookie sau login
res.cookie('accessToken', token, {
  httpOnly: true,     // Không đọc được từ JS → chặn XSS
  secure: true,       // Chỉ HTTPS
  sameSite: 'strict', // CSRF protection
  maxAge: 15 * 60 * 1000,
});
```

**Strategy 2: localStorage + Bearer token**
```ts
// Client:
localStorage.setItem('token', accessToken);
// Mỗi request: Authorization: Bearer <token>
```

Viết bảng so sánh: XSS vulnerability / CSRF vulnerability / Mobile support / Complexity.

---

### NextAuth / Auth.js

- [ ] Library auth phổ biến nhất cho Next.js

**Học:**
1. 📖 [Auth.js — Getting Started](https://authjs.dev/getting-started)
2. 📖 [Auth.js — Credentials Provider](https://authjs.dev/getting-started/authentication/credentials)
3. 📖 [Auth.js — Database Session](https://authjs.dev/getting-started/session-management/database)

**Exercises:**

🛠️ Setup NextAuth với:
- Google OAuth provider
- Credentials provider (email/password)
- Prisma adapter (lưu session vào PostgreSQL)
- Protected routes với middleware

```ts
// middleware.ts
export { auth as middleware } from '@/auth';
export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

---

## Monorepo với Turborepo

---

### Share Types giữa FE và BE

- [ ] Đây là superpower của TypeScript fullstack — 1 type, 2 nơi dùng

**Học:**
1. 📖 [Turborepo — Getting Started](https://turbo.build/repo/docs/getting-started/create-new)
2. 📖 [Turborepo — Sharing Code](https://turbo.build/repo/docs/crafting-your-repository/creating-an-internal-package)

**Exercises:**

🛠️ Tạo monorepo structure:
```
my-app/
├── apps/
│   ├── web/          ← Next.js app
│   └── api/          ← Express app
├── packages/
│   ├── types/        ← Shared TypeScript types
│   │   └── src/index.ts
│   └── validators/   ← Shared Zod schemas
│       └── src/index.ts
├── turbo.json
└── package.json
```

🛠️ Trong `packages/validators`:
```ts
// packages/validators/src/index.ts
import { z } from 'zod';

export const createPostSchema = z.object({
  title: z.string().min(1).max(255),
  content: z.string().min(1),
  tags: z.array(z.string()).default([]),
});

export type CreatePostInput = z.infer<typeof createPostSchema>;
```

Dùng cùng schema này trong:
- `apps/api`: validate request body
- `apps/web`: validate form trước khi submit

---

## TanStack Query

---

### Caching, Invalidation, Optimistic Updates

- [ ] Không chỉ "fetch data" — TanStack Query là state manager cho server state

**Học:**
1. 📖 [TanStack Query — Quick Start](https://tanstack.com/query/latest/docs/framework/react/quick-start)
2. 📖 [TanStack Query — Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
3. 📖 [TanStack Query — Optimistic Updates](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates)

**Exercises:**

🛠️ Implement like button với optimistic update:
```tsx
function LikeButton({ postId, initialLiked }: Props) {
  const queryClient = useQueryClient();

  const { mutate: toggleLike } = useMutation({
    mutationFn: (liked: boolean) => api.post(`/posts/${postId}/like`, { liked }),

    // Optimistic update — UI thay đổi NGAY, không đợi API
    onMutate: async (newLiked) => {
      await queryClient.cancelQueries({ queryKey: ['post', postId] });
      const previous = queryClient.getQueryData(['post', postId]);

      queryClient.setQueryData(['post', postId], (old: Post) => ({
        ...old,
        liked: newLiked,
        likeCount: old.likeCount + (newLiked ? 1 : -1),
      }));

      return { previous };
    },

    // Rollback nếu API fail
    onError: (err, _, context) => {
      queryClient.setQueryData(['post', postId], context?.previous);
    },
  });
}
```

---

## MSW — API Mocking

- [ ] Phát triển FE khi BE chưa xong — không cần mock thủ công

**Học:**
1. 📖 [Mock Service Worker — Getting Started](https://mswjs.io/docs/getting-started)

**Exercises:**

🛠️ Setup MSW cho Next.js app:
```ts
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/posts', () => {
    return HttpResponse.json([
      { id: 1, title: 'Mock Post 1', author: 'Nam' },
      { id: 2, title: 'Mock Post 2', author: 'An' },
    ]);
  }),

  http.post('/api/posts', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 3, ...body }, { status: 201 });
  }),
];
```

Chạy app với `NODE_ENV=development` — tất cả `/api/*` calls được intercept bởi MSW, không cần server thật.

---

## Project Phase 3: Project Management Tool

**Mô tả**: Trello clone nhỏ — boards, columns, tasks

**Tech Stack:**
```
apps/web/   → Next.js 14 + App Router + TanStack Query + NextAuth
apps/api/   → Express + Prisma + PostgreSQL + Redis + Socket.io
packages/
  types/    → Shared TypeScript interfaces
  validators/ → Shared Zod schemas
```

**Features:**
- [ ] Auth với Google OAuth (NextAuth)
- [ ] Tạo/xóa boards
- [ ] Columns trong board (To Do / In Progress / Done)
- [ ] Drag-and-drop tasks giữa columns
- [ ] Real-time: khi 1 user move task, user khác thấy ngay (Socket.io)
- [ ] Optimistic updates cho drag-and-drop (UI update ngay, sync về sau)
- [ ] File attachment cho tasks (upload lên Cloudflare R2)

**Deploy:**
- `apps/web` → Vercel (free)
- `apps/api` → Railway hoặc Fly.io (free tier)
- Database → Neon PostgreSQL (free)
- Cache → Upstash Redis (free)
