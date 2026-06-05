# Next.js — Interview Questions (Junior → Senior)

> Tổng hợp câu hỏi phỏng vấn Next.js từ Junior đến Senior.
> Câu hỏi song ngữ (EN/VI), trả lời chi tiết bằng tiếng Việt, code ví dụ bằng tiếng Anh.

---

## Table of Contents

1. [Next.js là gì? Tại sao dùng thay vì CRA/Vite SPA?](#1-nextjs-la-gi)
2. [Pages Router vs App Router](#2-pages-router-vs-app-router)
3. [SSR vs SSG vs ISR vs CSR — giải thích chi tiết](#3-ssr-ssg-isr-csr)
4. [getStaticProps / getServerSideProps / getStaticPaths (Pages Router)](#4-data-fetching-pages-router)
5. [Server Components vs Client Components](#5-server-vs-client-components)
6. [Data Fetching trong App Router](#6-data-fetching-app-router)
7. [next/image — tối ưu hóa hình ảnh](#7-next-image)
8. [next/font — tự host fonts](#8-next-font)
9. [Middleware — authentication, A/B testing, i18n](#9-middleware)
10. [API Routes vs Route Handlers](#10-api-routes-route-handlers)
11. [Caching trong App Router — 4 layers](#11-caching)
12. [Streaming và Suspense — loading.tsx, error.tsx](#12-streaming-suspense)
13. [next/dynamic — dynamic imports](#13-next-dynamic)
14. [Environment Variables](#14-env-variables)
15. [Deployment và Optimization](#15-deployment)

---

## 1. Next.js là gì?

**[Junior]**

**EN:** What is Next.js and why would you use it over a traditional CRA or Vite SPA?

**VI:** Next.js là gì và tại sao dùng nó thay vì CRA hoặc Vite SPA thông thường?

### Trả lời

**Next.js** là một React framework full-stack do Vercel phát triển, cung cấp nhiều tính năng out-of-the-box:

| Tính năng | CRA / Vite SPA | Next.js |
|-----------|----------------|---------|
| Rendering | CSR only | CSR, SSR, SSG, ISR |
| Routing | Cần thư viện (React Router) | File-based, built-in |
| Data fetching | Client-side only | Server + Client |
| SEO | Kém (HTML rỗng ban đầu) | Tốt (HTML đầy đủ từ server) |
| Image optimization | Tự làm | `next/image` built-in |
| Code splitting | Manual | Automatic per page |
| API routes | Cần server riêng | Built-in Route Handlers |
| Bundle optimization | Basic | Advanced (turbopack) |
| Font optimization | Tự làm | `next/font` built-in |

**Khi nào nên dùng Next.js:**
- Cần SEO tốt (landing page, blog, e-commerce)
- Cần First Contentful Paint (FCP) nhanh
- Cần render ở server (data phụ thuộc vào user, real-time)
- Cần API và frontend trong cùng một project
- Cần ISR (cập nhật nội dung mà không deploy lại)

**Khi nào SPA vẫn phù hợp:**
- Internal dashboard không cần SEO
- App đơn giản, team nhỏ, không cần server
- Progressive Web App

---

## 2. Pages Router vs App Router

**[Junior → Mid]**

**EN:** What are the differences between the Pages Router and App Router in Next.js? Compare folder structure and features.

**VI:** Sự khác biệt giữa Pages Router và App Router trong Next.js? So sánh cấu trúc thư mục và tính năng.

### Trả lời

**Pages Router** (Next.js 12 trở về trước, vẫn được support):

```
pages/
├── _app.tsx          # Global layout, wraps tất cả pages
├── _document.tsx     # Custom HTML document
├── index.tsx         # Route: /
├── about.tsx         # Route: /about
├── blog/
│   ├── index.tsx     # Route: /blog
│   └── [slug].tsx    # Route: /blog/:slug (dynamic)
└── api/
    └── users.ts      # API route: /api/users
```

**App Router** (Next.js 13.4+, stable từ 14, recommended):

```
app/
├── layout.tsx        # Root layout (bắt buộc)
├── page.tsx          # Route: /
├── loading.tsx       # Loading UI cho route /
├── error.tsx         # Error UI cho route /
├── not-found.tsx     # 404 UI
├── about/
│   └── page.tsx      # Route: /about
├── blog/
│   ├── layout.tsx    # Layout chỉ cho /blog và con
│   ├── page.tsx      # Route: /blog
│   └── [slug]/
│       └── page.tsx  # Route: /blog/:slug
└── api/
    └── users/
        └── route.ts  # Route Handler: /api/users
```

**So sánh tính năng:**

| Tính năng | Pages Router | App Router |
|-----------|-------------|------------|
| Server Components | Không | Mặc định |
| Layouts | `_app.tsx` (1 level) | Nested layouts |
| Data fetching | `getServerSideProps` / `getStaticProps` | `fetch` trong Server Component |
| Streaming | Không | Có (Suspense-based) |
| Caching | Ít granular | 4 cache layers |
| Concurrent features | Hạn chế | Full support |
| Loading states | Manual | `loading.tsx` file |
| Error handling | Custom `_error.tsx` | `error.tsx` file |
| Metadata | `<Head>` component | `metadata` export |

```tsx
// Pages Router — Metadata
import Head from 'next/head';

export default function Page() {
  return (
    <>
      <Head>
        <title>My Page</title>
        <meta name="description" content="Description" />
      </Head>
      <div>Content</div>
    </>
  );
}

// App Router — Metadata (static)
export const metadata = {
  title: 'My Page',
  description: 'Description',
};

export default function Page() {
  return <div>Content</div>;
}

// App Router — Metadata (dynamic)
export async function generateMetadata({ params }) {
  const product = await fetchProduct(params.id);
  return {
    title: product.name,
    description: product.description,
    openGraph: {
      images: [product.image],
    },
  };
}
```

---

## 3. SSR vs SSG vs ISR vs CSR

**[Junior → Mid]**

**EN:** Explain the four rendering strategies in Next.js: SSR, SSG, ISR, and CSR. When should you use each?

**VI:** Giải thích 4 chiến lược rendering trong Next.js: SSR, SSG, ISR, và CSR. Khi nào dùng mỗi loại?

### Trả lời

**1. CSR — Client-Side Rendering (SPA behavior):**

HTML ban đầu rỗng, JavaScript chạy ở browser rồi render UI.

```tsx
// App Router — CSR với 'use client' + useEffect
'use client';
import { useEffect, useState } from 'react';

export default function UserDashboard() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch('/api/user').then(r => r.json()).then(setUser);
  }, []);

  if (!user) return <Skeleton />;
  return <div>Hello, {user.name}</div>;
}
```

**2. SSG — Static Site Generation (build time):**

HTML được tạo lúc build, serve từ CDN — cực kỳ nhanh.

```tsx
// App Router — SSG (mặc định cho dynamic params)
// app/blog/[slug]/page.tsx

// Nếu có generateStaticParams → SSG
// Nếu không → dynamic rendering (mỗi request)
export async function generateStaticParams() {
  const posts = await fetchAllPosts();
  return posts.map(post => ({ slug: post.slug }));
}

export default async function BlogPost({ params }) {
  const post = await fetchPost(params.slug);
  return <article>{post.content}</article>;
}
```

```tsx
// Pages Router — SSG
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  return {
    props: { post },
  };
}

export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: false, // 404 cho paths không có trong list
  };
}

export default function BlogPost({ post }) {
  return <article>{post.content}</article>;
}
```

**3. SSR — Server-Side Rendering (per request):**

HTML được tạo ở server cho mỗi request.

```tsx
// App Router — SSR (opt-in với noStore hoặc dynamic = 'force-dynamic')
import { unstable_noStore as noStore } from 'next/cache';

export default async function UserPage() {
  noStore(); // Opt-out khỏi cache → SSR behavior
  const user = await fetchCurrentUser(); // Chạy ở server mỗi request
  return <div>Hello, {user.name}! Time: {new Date().toISOString()}</div>;
}

// Hoặc
export const dynamic = 'force-dynamic'; // Force SSR cho entire route
```

```tsx
// Pages Router — SSR
export async function getServerSideProps(context) {
  const { req, res, params, query } = context;
  const user = await fetchUser(req.cookies.token);
  return {
    props: { user },
  };
}

export default function UserPage({ user }) {
  return <div>Hello, {user.name}!</div>;
}
```

**4. ISR — Incremental Static Regeneration:**

Kết hợp tốt nhất của SSG và SSR: static nhưng có thể revalidate theo thời gian.

```tsx
// App Router — ISR với revalidate
export default async function ProductPage({ params }) {
  const product = await fetch(`/api/products/${params.id}`, {
    next: { revalidate: 3600 }, // Revalidate sau 3600 giây (1 giờ)
  }).then(r => r.json());

  return <ProductDetails product={product} />;
}

// On-demand revalidation (khi CMS publish content)
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request) {
  const { secret, path, tag } = await request.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  if (path) revalidatePath(path);
  if (tag) revalidateTag(tag);

  return Response.json({ revalidated: true });
}
```

```tsx
// Pages Router — ISR
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return {
    props: { product },
    revalidate: 3600, // Revalidate sau 1 giờ
  };
}
```

**Bảng so sánh khi nào dùng:**

| Chiến lược | Khi nào dùng | Ví dụ |
|------------|--------------|-------|
| **CSR** | Data riêng tư, không cần SEO, real-time | Dashboard, admin panel, chat |
| **SSG** | Content tĩnh, ít thay đổi, cần SEO | Blog, docs, landing page marketing |
| **SSR** | Data cá nhân hóa mỗi request, cần SEO | User profile, search results, cart |
| **ISR** | Content thay đổi thỉnh thoảng, cần SEO | E-commerce catalog, news site |

---

## 4. Data Fetching — Pages Router

**[Junior → Mid]**

**EN:** Explain `getStaticProps`, `getServerSideProps`, and `getStaticPaths` in the Pages Router.

**VI:** Giải thích `getStaticProps`, `getServerSideProps`, và `getStaticPaths` trong Pages Router.

### Trả lời

**`getStaticProps` — chạy lúc build time:**

```tsx
// pages/products/[id].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Product {
  id: string;
  name: string;
  price: number;
}

interface Props {
  product: Product;
}

export const getStaticPaths: GetStaticPaths = async () => {
  const products = await fetchAllProducts();

  return {
    paths: products.map(p => ({ params: { id: p.id } })),
    // fallback: false → 404 nếu path không trong list
    // fallback: true → render fallback UI trong khi generate
    // fallback: 'blocking' → SSR cho paths không trong list (không fallback UI)
    fallback: 'blocking',
  };
};

export const getStaticProps: GetStaticProps<Props> = async ({ params }) => {
  try {
    const product = await fetchProduct(params!.id as string);

    if (!product) {
      return { notFound: true }; // → 404 page
    }

    return {
      props: { product },
      revalidate: 60, // ISR: revalidate sau 60 giây
    };
  } catch (error) {
    return {
      redirect: {
        destination: '/products',
        permanent: false,
      },
    };
  }
};

export default function ProductPage({ product }: Props) {
  return <div>{product.name} — ${product.price}</div>;
}
```

**`getServerSideProps` — chạy mỗi request:**

```tsx
// pages/profile.tsx
import { GetServerSideProps } from 'next';
import { getSession } from 'next-auth/react';

export const getServerSideProps: GetServerSideProps = async (context) => {
  const { req, res, params, query, resolvedUrl } = context;

  // Đọc cookie, session
  const session = await getSession({ req });

  if (!session) {
    return {
      redirect: {
        destination: `/login?callbackUrl=${resolvedUrl}`,
        permanent: false,
      },
    };
  }

  const user = await fetchUserData(session.user.id);

  return {
    props: {
      user,
      // Lưu ý: data phải JSON-serializable
      // Dates cần convert sang string
      lastLogin: user.lastLogin.toISOString(),
    },
  };
};
```

**`getStaticPaths` — `fallback` options:**

```tsx
// fallback: false — 404 cho tất cả paths không được pre-generate
// Dùng khi: số lượng pages nhỏ, biết hết trước
export async function getStaticPaths() {
  return { paths: [...], fallback: false };
}

// fallback: true — hiển thị fallback UI ngay, generate page ở background
// Dùng khi: nhiều pages, muốn UX tốt cho paths mới
export default function Page({ product }) {
  const router = useRouter();
  if (router.isFallback) return <Spinner />; // Fallback UI
  return <div>{product.name}</div>;
}

// fallback: 'blocking' — chờ generate xong (như SSR lần đầu), cache lại sau
// Dùng khi: muốn SEO tốt, không muốn fallback UI
export async function getStaticPaths() {
  return { paths: [...], fallback: 'blocking' };
}
```

---

## 5. Server Components vs Client Components

**[Mid → Senior]**

**EN:** What is the difference between Server Components and Client Components? Explain the composition pattern, the "use client" boundary, and what you can/cannot do in each.

**VI:** Sự khác biệt giữa Server Components và Client Components? Giải thích composition pattern, "use client" boundary, và những gì có thể/không thể làm trong mỗi loại.

### Trả lời

**Server Components (RSC)** — mặc định trong App Router:
- Render ở server, HTML gửi xuống client
- Có thể access server-side resources trực tiếp (DB, file system, env vars)
- Bundle size 0 (không ship JavaScript xuống client)
- KHÔNG thể dùng: browser APIs, useState, useEffect, event handlers

**Client Components** — opt-in với `'use client'`:
- Render ở client (và server lần đầu để prerender)
- Có thể dùng hooks, event handlers, browser APIs
- Ship JavaScript xuống browser
- KHÔNG thể access server resources trực tiếp

```tsx
// app/dashboard/page.tsx — Server Component (mặc định)
// Không có 'use client' directive

async function DashboardPage() {
  // ✅ Trực tiếp query database — không cần API route!
  const stats = await db.query('SELECT COUNT(*) FROM users');

  // ✅ Access env vars (server-only, không bị expose ra client)
  const apiKey = process.env.INTERNAL_API_KEY;

  // ✅ Gọi async operations ở top level
  const [users, revenue] = await Promise.all([
    fetchUsers(),
    fetchRevenue(),
  ]);

  return (
    <div>
      <StatsCard stats={stats} />
      {/* Pass Server Component data xuống Client Component */}
      <InteractiveChart data={revenue} />
      <UserTable users={users} />
    </div>
  );
}
```

```tsx
// app/components/InteractiveChart.tsx — Client Component
'use client'; // Boundary — tất cả imports/children cũng là client (trừ RSC passed as children)

import { useState, useEffect } from 'react';
import { Chart } from 'chart.js';

export function InteractiveChart({ data }) {
  const [selectedPeriod, setSelectedPeriod] = useState('month');
  const chartRef = useRef(null);

  // ✅ Hooks, event handlers, browser APIs — OK trong Client Component
  useEffect(() => {
    const chart = new Chart(chartRef.current, { data });
    return () => chart.destroy();
  }, [data]);

  return (
    <div>
      <button onClick={() => setSelectedPeriod('week')}>Week</button>
      <button onClick={() => setSelectedPeriod('month')}>Month</button>
      <canvas ref={chartRef} />
    </div>
  );
}
```

**Composition Pattern — Server Component làm "children" của Client Component:**

```tsx
// ✅ Pattern đúng: truyền RSC như children vào Client Component
// app/layout.tsx (Server Component)
import { Modal } from './Modal'; // Client Component
import { UserProfile } from './UserProfile'; // Server Component

export default function Layout({ children }) {
  return (
    <Modal> {/* Client Component */}
      <UserProfile /> {/* Server Component — được phép! */}
    </Modal>
  );
}

// app/Modal.tsx
'use client';
export function Modal({ children }) {
  const [isOpen, setIsOpen] = useState(false);
  // children (RSC) không bị "converted" thành Client Component
  // chúng vẫn chạy ở server
  return isOpen ? <div className="modal">{children}</div> : null;
}

// ❌ Pattern sai: import RSC trực tiếp trong Client Component
'use client';
import { ServerComponent } from './ServerComponent'; // ❌ Bị convert thành Client!
```

**Bảng so sánh:**

| | Server Component | Client Component |
|--|-----------------|-----------------|
| `'use client'` directive | Không có | Cần có |
| useState / useEffect | ❌ | ✅ |
| Event handlers | ❌ | ✅ |
| Browser APIs | ❌ | ✅ |
| Database access | ✅ | ❌ |
| File system | ✅ | ❌ |
| Server env vars | ✅ | ❌ |
| async/await top-level | ✅ | ❌ |
| JavaScript bundle | 0 | Có |
| SEO | ✅ | ✅ (prerendered) |

---

## 6. Data Fetching — App Router

**[Mid → Senior]**

**EN:** How does data fetching work in the App Router? Explain `fetch` with cache options, revalidation, and cache tags.

**VI:** Data fetching trong App Router hoạt động như thế nào? Giải thích `fetch` với cache options, revalidation, và cache tags.

### Trả lời

Trong App Router, data fetching được thực hiện thẳng trong Server Components. Next.js extend `fetch` API với caching options.

**Cache options:**

```tsx
// 1. cache: 'force-cache' (default) — SSG behavior
// Cache vĩnh viễn, không revalidate trừ khi explicitly invalidate
async function StaticData() {
  const data = await fetch('https://api.example.com/static', {
    cache: 'force-cache', // Mặc định nếu không chỉ định
  });
  return data.json();
}

// 2. cache: 'no-store' — SSR behavior
// Không cache, fetch lại mỗi request
async function DynamicData() {
  const data = await fetch('https://api.example.com/live', {
    cache: 'no-store',
  });
  return data.json();
}

// 3. next.revalidate — ISR behavior
// Cache, revalidate sau N giây
async function ISRData() {
  const data = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 }, // Revalidate sau 1 giờ
  });
  return data.json();
}

// 4. next.tags — On-demand revalidation bằng tags
async function TaggedData() {
  const data = await fetch('https://api.example.com/posts', {
    next: {
      revalidate: 3600,
      tags: ['posts'], // Tag để revalidate theo demand
    },
  });
  return data.json();
}
```

**Request Deduplication — automatic:**

```tsx
// Trong cùng một render tree, cùng một fetch request chỉ thực hiện 1 lần
// Kể cả nếu nhiều components gọi cùng URL

async function fetchUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

// Component A
async function UserName({ id }) {
  const user = await fetchUser(id); // Fetch lần 1
  return <span>{user.name}</span>;
}

// Component B (cùng render)
async function UserAvatar({ id }) {
  const user = await fetchUser(id); // Được deduplicated — không fetch lại!
  return <img src={user.avatar} />;
}
```

**On-demand revalidation:**

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const { secret, type, value } = await request.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ message: 'Invalid secret' }, { status: 401 });
  }

  if (type === 'tag') {
    revalidateTag(value); // Revalidate tất cả fetches có tag này
  } else if (type === 'path') {
    revalidatePath(value); // Revalidate specific path
  }

  return Response.json({ revalidated: true, now: Date.now() });
}

// Gọi từ CMS webhook khi content thay đổi:
// POST /api/revalidate
// { "secret": "xxx", "type": "tag", "value": "posts" }
```

**Parallel Data Fetching — tránh request waterfall:**

```tsx
// ❌ Sequential — chậm, mỗi fetch đợi cái trước
async function Page({ params }) {
  const user = await fetchUser(params.id);          // Đợi
  const posts = await fetchUserPosts(params.id);    // Đợi user xong mới fetch
  const followers = await fetchFollowers(params.id); // Đợi tiếp
  // Total: time(user) + time(posts) + time(followers)
}

// ✅ Parallel — nhanh hơn
async function Page({ params }) {
  const [user, posts, followers] = await Promise.all([
    fetchUser(params.id),
    fetchUserPosts(params.id),
    fetchFollowers(params.id),
  ]);
  // Total: max(time(user), time(posts), time(followers))
}
```

---

## 7. next/image

**[Junior → Mid]**

**EN:** How does `next/image` work? What optimizations does it provide? Explain lazy loading, layout options, and blur placeholder.

**VI:** `next/image` hoạt động như thế nào? Nó cung cấp những tối ưu hóa gì? Giải thích lazy loading, layout options, và blur placeholder.

### Trả lời

`next/image` tự động tối ưu hóa images:

- **Format conversion**: Tự động serve WebP/AVIF nếu browser hỗ trợ
- **Resize**: Chỉ serve kích thước cần thiết cho từng device
- **Lazy loading**: Mặc định chỉ load khi vào viewport
- **Prevent CLS**: Giữ chỗ (placeholder) để tránh layout shift
- **CDN caching**: Cache images tự động

```tsx
import Image from 'next/image';

// Basic usage — local image (dimensions tự động)
import profilePic from './profile.png';

function Avatar() {
  return (
    <Image
      src={profilePic}
      alt="Profile picture"
      // width và height không cần với local import
    />
  );
}

// Remote image — cần width, height, và config
function ProductImage({ url }) {
  return (
    <Image
      src={url}
      alt="Product"
      width={400}
      height={300}
      quality={75}          // Default: 75, range: 1-100
      priority={false}      // Default: false (lazy). Set true cho LCP image
    />
  );
}

// fill — image fill toàn bộ parent container
function HeroBanner() {
  return (
    <div style={{ position: 'relative', width: '100%', height: '500px' }}>
      <Image
        src="/hero.jpg"
        alt="Hero banner"
        fill
        style={{ objectFit: 'cover' }} // cover, contain, fill
        sizes="100vw"
      />
    </div>
  );
}

// sizes — hint cho browser về kích thước image ở các breakpoints
function ResponsiveImage() {
  return (
    <Image
      src="/photo.jpg"
      alt="Photo"
      fill
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      // Browser dùng sizes để chọn đúng srcset
    />
  );
}

// Blur placeholder — UX tốt hơn khi loading
function ImageWithBlur() {
  return (
    // Local images: placeholder="blur" tự động
    <Image src={localImage} alt="..." placeholder="blur" />
  );
}

// Remote images cần blurDataURL
function RemoteImageWithBlur() {
  return (
    <Image
      src="https://example.com/photo.jpg"
      alt="Photo"
      width={800}
      height={600}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgAB..." // base64 tiny image
    />
  );
}
```

**Config cho remote images (bắt buộc):**

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        port: '',
        pathname: '/images/**',
      },
      {
        protocol: 'https',
        hostname: '**.cloudinary.com',
      },
    ],
  },
};
```

**Khi nào dùng `priority={true}`:**

```tsx
// Ảnh LCP (Largest Contentful Paint) — ảnh hiển thị đầu tiên above the fold
// VD: hero banner, product image trên fold đầu tiên
<Image src="/hero.jpg" alt="Hero" priority={true} fill />
// Không lazy load, preload ngay → cải thiện LCP score
```

---

## 8. next/font

**[Junior → Mid]**

**EN:** Why does `next/font` matter? How does it work for self-hosting fonts?

**VI:** Tại sao `next/font` quan trọng? Nó hoạt động như thế nào để self-host fonts?

### Trả lời

**Vấn đề khi dùng Google Fonts thông thường:**
- Request đến Google servers → latency, privacy concerns
- Font file block rendering → CLS (Cumulative Layout Shift)
- FOUT (Flash of Unstyled Text) khi font chưa load

**`next/font` giải quyết:**
- **Self-host fonts** tự động — download font lúc build, serve từ server của bạn
- **Zero layout shift** — CSS `size-adjust` đảm bảo fallback font có cùng kích thước
- **Privacy** — không request nào đến Google từ browser
- **Performance** — font được serve cùng domain, cache tối ưu

```tsx
// app/layout.tsx — Google Fonts (tự động self-hosted)
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',        // 'auto' | 'block' | 'swap' | 'fallback' | 'optional'
  variable: '--font-inter', // CSS variable để dùng trong CSS/Tailwind
  weight: ['400', '500', '600', '700'],
  preload: true,
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>
        {children}
      </body>
    </html>
  );
}
```

```css
/* globals.css — dùng CSS variable */
body {
  font-family: var(--font-inter), sans-serif;
}

code {
  font-family: var(--font-roboto-mono), monospace;
}
```

```tsx
// Tailwind CSS config
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)'],
        mono: ['var(--font-roboto-mono)'],
      },
    },
  },
};
```

```tsx
// Local fonts — tự upload font file
import localFont from 'next/font/local';

const myFont = localFont({
  src: [
    {
      path: './fonts/MyFont-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './fonts/MyFont-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
  ],
  variable: '--font-my-font',
});
```

---

## 9. Middleware

**[Mid → Senior]**

**EN:** What is Next.js Middleware? Show examples for authentication, A/B testing, i18n, and rate limiting.

**VI:** Next.js Middleware là gì? Cho ví dụ về authentication, A/B testing, i18n, và rate limiting.

### Trả lời

**Middleware** chạy ở Edge (trước khi request đến server/static files), cho phép rewrite, redirect, modify headers, hoặc return response sớm.

```typescript
// middleware.ts (root của project, cùng cấp app/)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Middleware logic ở đây
  return NextResponse.next();
}

// Chỉ chạy middleware cho các paths này
export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
    // Loại trừ static files, chỉ áp dụng cho pages
  ],
};
```

**1. Authentication Middleware:**

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { verifyJWT } from './lib/auth';

const PUBLIC_PATHS = ['/', '/login', '/register', '/about'];

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Bỏ qua public paths
  if (PUBLIC_PATHS.some(path => pathname.startsWith(path))) {
    return NextResponse.next();
  }

  const token = request.cookies.get('auth-token')?.value;

  if (!token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('callbackUrl', pathname);
    return NextResponse.redirect(loginUrl);
  }

  try {
    const payload = await verifyJWT(token);

    // Kiểm tra role-based access
    if (pathname.startsWith('/admin') && payload.role !== 'admin') {
      return NextResponse.redirect(new URL('/403', request.url));
    }

    // Inject user info vào request header
    const requestHeaders = new Headers(request.headers);
    requestHeaders.set('x-user-id', payload.userId);

    return NextResponse.next({ request: { headers: requestHeaders } });
  } catch {
    const response = NextResponse.redirect(new URL('/login', request.url));
    response.cookies.delete('auth-token');
    return response;
  }
}
```

**2. A/B Testing Middleware:**

```typescript
// middleware.ts
export function middleware(request: NextRequest) {
  // Kiểm tra bucket hiện tại
  const bucket = request.cookies.get('ab-bucket')?.value;

  // Assign bucket ngẫu nhiên nếu chưa có
  const assignedBucket = bucket ?? (Math.random() < 0.5 ? 'control' : 'variant');

  // Rewrite URL để serve variant khác nhau
  let response: NextResponse;

  if (request.nextUrl.pathname === '/landing') {
    const newUrl = request.nextUrl.clone();
    newUrl.pathname = assignedBucket === 'variant' ? '/landing-v2' : '/landing';
    response = NextResponse.rewrite(newUrl);
  } else {
    response = NextResponse.next();
  }

  // Set cookie để user giữ cùng bucket
  if (!bucket) {
    response.cookies.set('ab-bucket', assignedBucket, {
      maxAge: 60 * 60 * 24 * 30, // 30 ngày
      httpOnly: true,
    });
  }

  return response;
}
```

**3. i18n Middleware:**

```typescript
// middleware.ts
const SUPPORTED_LOCALES = ['en', 'vi', 'ja'];
const DEFAULT_LOCALE = 'en';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Check if pathname already has locale
  const hasLocale = SUPPORTED_LOCALES.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );

  if (hasLocale) return NextResponse.next();

  // Detect locale từ Accept-Language header
  const acceptLanguage = request.headers.get('accept-language') ?? '';
  const preferredLocale = SUPPORTED_LOCALES.find(
    locale => acceptLanguage.includes(locale)
  ) ?? DEFAULT_LOCALE;

  // Redirect sang URL có locale
  request.nextUrl.pathname = `/${preferredLocale}${pathname}`;
  return NextResponse.redirect(request.nextUrl);
}

export const config = {
  matcher: ['/((?!api|_next|.*\\..*).*)'],
};
```

**4. Rate Limiting (Edge với Redis/Upstash):**

```typescript
// middleware.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'), // 10 requests/10 giây
});

export async function middleware(request: NextRequest) {
  // Chỉ rate limit API routes
  if (!request.nextUrl.pathname.startsWith('/api')) {
    return NextResponse.next();
  }

  const ip = request.ip ?? request.headers.get('x-forwarded-for') ?? 'anonymous';
  const { success, limit, remaining, reset } = await ratelimit.limit(ip);

  if (!success) {
    return new NextResponse('Too Many Requests', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
        'Retry-After': Math.ceil((reset - Date.now()) / 1000).toString(),
      },
    });
  }

  return NextResponse.next();
}
```

---

## 10. API Routes vs Route Handlers

**[Junior → Mid]**

**EN:** What is the difference between API Routes (Pages Router) and Route Handlers (App Router)? How do you handle different HTTP methods?

**VI:** Sự khác biệt giữa API Routes (Pages Router) và Route Handlers (App Router)? Cách xử lý các HTTP methods khác nhau?

### Trả lời

**Pages Router — API Routes:**

```typescript
// pages/api/users/[id].ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;

  switch (req.method) {
    case 'GET':
      try {
        const user = await fetchUser(id as string);
        if (!user) return res.status(404).json({ error: 'User not found' });
        return res.status(200).json(user);
      } catch (error) {
        return res.status(500).json({ error: 'Internal server error' });
      }

    case 'PUT':
      try {
        const updated = await updateUser(id as string, req.body);
        return res.status(200).json(updated);
      } catch (error) {
        return res.status(400).json({ error: 'Bad request' });
      }

    case 'DELETE':
      await deleteUser(id as string);
      return res.status(204).end();

    default:
      res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
      return res.status(405).json({ error: 'Method not allowed' });
  }
}

// Config
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '4mb',
    },
  },
};
```

**App Router — Route Handlers:**

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

// Named exports cho từng HTTP method — cleaner, type-safe
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const { id } = params;

  try {
    const user = await fetchUser(id);
    if (!user) {
      return NextResponse.json({ error: 'User not found' }, { status: 404 });
    }
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const updated = await updateUser(params.id, body);
  return NextResponse.json(updated);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await deleteUser(params.id);
  return new NextResponse(null, { status: 204 });
}

// Streaming response
export async function POST(request: NextRequest) {
  const { prompt } = await request.json();

  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();
      for await (const chunk of generateText(prompt)) {
        controller.enqueue(encoder.encode(chunk));
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  });
}
```

**So sánh:**

| | API Routes | Route Handlers |
|--|------------|----------------|
| File naming | `pages/api/*.ts` | `app/api/*/route.ts` |
| HTTP method | `switch (req.method)` | Named exports (GET, POST...) |
| Request object | `NextApiRequest` | `NextRequest` (Web standard) |
| Response | `res.json()` | `NextResponse.json()` |
| Streaming | Hạn chế | ✅ Native support |
| Edge runtime | ❌ | ✅ |
| Web Crypto API | ❌ | ✅ |
| Cookies/Headers | `req.cookies` | `cookies()`, `headers()` từ next/headers |

---

## 11. Caching

**[Senior]**

**EN:** Explain the 4 caching layers in Next.js App Router: Request Memoization, Data Cache, Full Route Cache, and Router Cache.

**VI:** Giải thích 4 lớp cache trong Next.js App Router: Request Memoization, Data Cache, Full Route Cache, và Router Cache.

### Trả lời

Next.js App Router có 4 lớp cache hoạt động theo thứ tự, từ nhanh nhất đến chậm nhất:

```
Request → Router Cache → Full Route Cache → Data Cache → Request Memoization → Data Source
```

**1. Request Memoization (In-Memory, per-request):**

```tsx
// Trong một request, cùng fetch URL chỉ gọi 1 lần
// Tự động bởi React — không cần làm gì

async function getUser(id: string) {
  // Kể cả gọi hàm này 10 lần trong 1 render,
  // HTTP request chỉ xảy ra 1 lần
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

// Scope: 1 server request
// Cleared: sau mỗi request
// Chỉ áp dụng: GET fetch requests
```

**2. Data Cache (Persistent, cross-request):**

```tsx
// Kết quả fetch được cache trên server, persist giữa các requests
// Đây là cache mà next.revalidate, cache: 'no-store' affect

// SSG — cache mãi mãi
const data = await fetch('/api/data', { cache: 'force-cache' });

// ISR — cache với thời gian
const data = await fetch('/api/data', { next: { revalidate: 3600 } });

// SSR — không cache
const data = await fetch('/api/data', { cache: 'no-store' });

// Tagged cache — invalidate theo demand
const data = await fetch('/api/posts', {
  next: { tags: ['posts'], revalidate: 3600 },
});

// Scope: cross-request, persistent
// Stored: file system (server)
// Cleared: revalidateTag(), revalidatePath(), hoặc revalidate timeout
```

**3. Full Route Cache (HTML + RSC Payload, build time):**

```tsx
// Next.js cache toàn bộ rendered output của route
// Static routes (không dùng dynamic functions) được cache lúc build

// Opt out khỏi Full Route Cache:
export const dynamic = 'force-dynamic'; // Route không được cache
// Hoặc dùng dynamic functions:
import { cookies, headers } from 'next/headers';

export default async function Page() {
  const cookieStore = cookies(); // Dùng cookies → route trở thành dynamic
  const headersList = headers(); // Tương tự
  // ...
}

// Scope: cross-request, persistent
// Stored: file system
// Cleared: khi deploy, revalidatePath(), revalidateTag()
```

**4. Router Cache (Client-side, per-session):**

```tsx
// Cache RSC payload ở client memory trong browser session
// Prefetch và cache các routes khi hover link (next/link)
// Cho phép instant navigation back/forward

// Prefetching — Link tự động prefetch khi vào viewport
import Link from 'next/link';

// prefetch={true} — default cho static routes (prefetch full page)
// prefetch={false} — không prefetch
// prefetch={null} — default cho dynamic routes (chỉ prefetch loading.tsx)
<Link href="/dashboard" prefetch={true}>Dashboard</Link>

// Invalidate router cache
import { useRouter } from 'next/navigation';

function Component() {
  const router = useRouter();
  const handleAction = async () => {
    await updateData();
    router.refresh(); // Invalidate router cache, refetch từ server
  };
}

// Scope: 1 browser session
// Duration: static = 5 phút, dynamic = 30 giây
// Cleared: router.refresh(), revalidatePath(), revalidateTag(), cookies thay đổi
```

**Tóm tắt:**

| Cache | Nơi lưu | Thời gian sống | Cách xóa |
|-------|---------|----------------|----------|
| Request Memo | Server memory | 1 request | Tự động |
| Data Cache | File system | Vĩnh viễn (trừ revalidate) | revalidateTag/Path |
| Full Route Cache | File system | Đến khi deploy/revalidate | Deploy, revalidate |
| Router Cache | Browser memory | 30s-5 phút | router.refresh() |

---

## 12. Streaming và Suspense

**[Mid → Senior]**

**EN:** How does streaming work in Next.js? Explain `loading.tsx`, `error.tsx`, and nested Suspense boundaries.

**VI:** Streaming hoạt động trong Next.js như thế nào? Giải thích `loading.tsx`, `error.tsx`, và nested Suspense boundaries.

### Trả lời

**Streaming** cho phép server gửi HTML theo từng phần (chunks) thay vì đợi toàn bộ trang render xong. Người dùng thấy nội dung sớm hơn.

```
Without Streaming:
Server ──────────────────────────► Client shows page
        (wait for all data)

With Streaming:
Server ──────► Client shows shell
      ──────────────► Client shows part A
      ──────────────────────────► Client shows part B
```

**`loading.tsx` — automatic Suspense boundary:**

```tsx
// app/dashboard/loading.tsx
// Tự động bọc page.tsx trong Suspense, hiển thị khi đang load

export default function DashboardLoading() {
  return (
    <div>
      <Skeleton className="h-8 w-48 mb-4" />
      <div className="grid grid-cols-3 gap-4">
        <Skeleton className="h-32" />
        <Skeleton className="h-32" />
        <Skeleton className="h-32" />
      </div>
    </div>
  );
}

// app/dashboard/page.tsx
// Khi loading.tsx exist → page được bọc trong Suspense tự động
export default async function Dashboard() {
  const data = await fetchSlowData(); // Đang fetch → loading.tsx hiển thị
  return <DashboardContent data={data} />;
}
```

**`error.tsx` — automatic Error Boundary:**

```tsx
// app/dashboard/error.tsx
'use client'; // Error components phải là Client Component

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void; // Thử render lại
}) {
  useEffect(() => {
    // Log error sang reporting service
    logError(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// global-error.tsx — catch errors ở root layout
// app/global-error.tsx
'use client';
export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>Something went critically wrong!</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  );
}
```

**Nested Suspense — fine-grained streaming:**

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <div>
      {/* Header load ngay — không async */}
      <DashboardHeader />

      {/* Stats load riêng lẻ */}
      <Suspense fallback={<StatsSkeleton />}>
        <StatsCards />    {/* async — fetch data riêng */}
      </Suspense>

      <div className="grid grid-cols-2">
        {/* Hai components load độc lập */}
        <Suspense fallback={<ChartSkeleton />}>
          <RevenueChart />  {/* async */}
        </Suspense>

        <Suspense fallback={<TableSkeleton />}>
          <RecentOrders /> {/* async */}
        </Suspense>
      </div>
    </div>
  );
}

// Mỗi async component sẽ stream xuống khi data ready
// → User thấy skeleton, rồi dần dần thay bằng nội dung thật
async function StatsCards() {
  const stats = await fetchStats(); // Có thể chậm
  return <div>{stats.map(s => <StatCard key={s.id} stat={s} />)}</div>;
}

async function RevenueChart() {
  const data = await fetchRevenue(); // Fetch riêng
  return <Chart data={data} />;
}
```

---

## 13. next/dynamic

**[Mid]**

**EN:** How does `next/dynamic` work? When would you use `ssr: false`?

**VI:** `next/dynamic` hoạt động như thế nào? Khi nào dùng `ssr: false`?

### Trả lời

`next/dynamic` là wrapper của `React.lazy` với tích hợp SSR support và loading states cho Next.js.

```tsx
import dynamic from 'next/dynamic';

// Basic — code split, lazy load
const HeavyComponent = dynamic(() => import('../components/HeavyComponent'), {
  loading: () => <p>Loading...</p>,
});

// ssr: false — quan trọng cho client-only components
const Map = dynamic(() => import('../components/Map'), {
  ssr: false, // Không render ở server
  loading: () => <div className="map-placeholder">Loading map...</div>,
});

// Tại sao cần ssr: false?
// 1. Component dùng browser-only APIs (window, document, navigator)
// 2. Third-party library không support SSR (Leaflet, certain chart libraries)
// 3. Component access localStorage/sessionStorage trong render

// Ví dụ — Leaflet map sẽ crash ở server
'use client';
import { MapContainer, TileLayer } from 'react-leaflet'; // Dùng window

// ❌ Crash nếu import trực tiếp trong Server Component
// import Map from './Map';

// ✅ Dùng dynamic với ssr: false
const Map = dynamic(() => import('./Map'), { ssr: false });
```

```tsx
// Named exports
const { MyComponent } = dynamic(
  () => import('../components/MyComponent').then(mod => mod.MyComponent)
);

// Preload on hover — tương tự React.lazy preload
const Modal = dynamic(() => import('./Modal'));

function Button() {
  const [show, setShow] = useState(false);

  return (
    <>
      <button
        onMouseEnter={() => import('./Modal')} // Preload khi hover
        onClick={() => setShow(true)}
      >
        Open Modal
      </button>
      {show && <Modal />}
    </>
  );
}
```

**next/dynamic vs React.lazy:**

| | React.lazy | next/dynamic |
|--|------------|-------------|
| SSR | Không hỗ trợ | Có (opt-out với `ssr: false`) |
| Loading state | Dùng Suspense | Built-in `loading` prop |
| Named exports | Cần .then() | Tự động (với helper) |
| Environment | Client only | Server + Client |

---

## 14. Environment Variables

**[Junior]**

**EN:** How do environment variables work in Next.js? Explain `.env.local`, the `NEXT_PUBLIC_` prefix, and server vs client access.

**VI:** Environment variables hoạt động trong Next.js như thế nào? Giải thích `.env.local`, prefix `NEXT_PUBLIC_`, và truy cập từ server vs client.

### Trả lời

**File hierarchy (thứ tự ưu tiên từ cao xuống thấp):**

```
.env.local          # Luôn override, không commit git (chứa secrets)
.env.development    # Chỉ load trong development
.env.production     # Chỉ load trong production
.env                # Mặc định, tất cả environments
```

**Server-only vs Client-exposed:**

```bash
# .env.local

# Server-only — KHÔNG được gửi xuống browser
DATABASE_URL="postgresql://..."
API_SECRET_KEY="super-secret-key"
STRIPE_SECRET_KEY="sk_live_..."
INTERNAL_API_URL="http://internal-service:3000"

# Client-exposed — prefix NEXT_PUBLIC_ → ship xuống browser bundle
NEXT_PUBLIC_API_URL="https://api.example.com"
NEXT_PUBLIC_GOOGLE_MAPS_KEY="AIza..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_live_..."
```

```tsx
// Server Component / API Route — truy cập được tất cả
async function ServerComponent() {
  const dbUrl = process.env.DATABASE_URL;        // ✅ Có access
  const publicUrl = process.env.NEXT_PUBLIC_API_URL; // ✅ Có access
}

// Client Component — chỉ truy cập được NEXT_PUBLIC_
'use client';
function ClientComponent() {
  const publicUrl = process.env.NEXT_PUBLIC_API_URL; // ✅ OK
  const secretKey = process.env.API_SECRET_KEY;       // ❌ undefined! Không expose
}
```

**Type-safety với zod (best practice):**

```typescript
// lib/env.ts — validate env vars lúc startup
import { z } from 'zod';

const serverSchema = z.object({
  DATABASE_URL: z.string().url(),
  API_SECRET_KEY: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']),
});

const clientSchema = z.object({
  NEXT_PUBLIC_API_URL: z.string().url(),
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().startsWith('pk_'),
});

// Validate server env
export const serverEnv = serverSchema.parse({
  DATABASE_URL: process.env.DATABASE_URL,
  API_SECRET_KEY: process.env.API_SECRET_KEY,
  NODE_ENV: process.env.NODE_ENV,
});

// Validate client env
export const clientEnv = clientSchema.parse({
  NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY,
});

// Nếu thiếu env var → crash ngay lúc startup với error rõ ràng
```

**Sử dụng trong các contexts:**

```tsx
// Server Action — truy cập server env
'use server';
export async function createPayment(amount: number) {
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
  return stripe.paymentIntents.create({ amount, currency: 'usd' });
}

// Route Handler
export async function GET() {
  const data = await fetch(process.env.INTERNAL_API_URL + '/health');
  return Response.json(await data.json());
}

// Client Component
'use client';
function PaymentButton({ price }) {
  const stripe = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);
  // ...
}
```

---

## 15. Deployment và Optimization

**[Mid → Senior]**

**EN:** Explain Next.js deployment options, build output analysis, and key performance optimization strategies.

**VI:** Giải thích các tùy chọn deployment của Next.js, phân tích build output, và các chiến lược tối ưu performance chính.

### Trả lời

**Build output:**

```bash
npm run build
# Output:
# Route (app)                              Size     First Load JS
# ┌ ○ /                                   5.2 kB        87.5 kB
# ├ ○ /about                              1.1 kB        83.4 kB
# ├ ● /blog/[slug]                        3.4 kB        85.7 kB
# ├ ƒ /dashboard                          2.8 kB        85.1 kB
# └ ƒ /api/users/[id]                     0 B                0 B
#
# ○ Static  ● SSG (getStaticProps)  ƒ Dynamic (server-rendered)
```

**Deployment options:**

```
1. Vercel (recommended for Next.js)
   - Zero config deployment
   - Edge Network tự động
   - Preview deployments cho mỗi PR
   - Built-in analytics, logging

2. Docker / Self-hosted server
   - Cần Node.js runtime
   - next build → .next/ folder
   - next start để chạy production server

3. Static Export (chỉ SSG/CSR, không SSR/ISR)
   next.config.js: output: 'export'
   → Cho phép deploy lên bất kỳ static host (S3, GitHub Pages)

4. Edge Runtime (partial)
   - Middleware chạy ở Edge mặc định
   - Route Handlers có thể opt-in Edge runtime
```

**Bundle Analysis:**

```bash
# Cài đặt
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // next config
});

# Chạy
ANALYZE=true npm run build
# → Mở browser với interactive bundle visualization
```

**Key performance optimizations:**

```tsx
// 1. Image optimization
import Image from 'next/image';
<Image
  src="/hero.jpg"
  alt="Hero"
  priority // LCP image — preload
  fill
  sizes="100vw"
/>

// 2. Font optimization
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'], display: 'swap' });

// 3. Script optimization
import Script from 'next/script';
// strategy: 'beforeInteractive' | 'afterInteractive' | 'lazyOnload' | 'worker'
<Script src="https://analytics.js" strategy="lazyOnload" />

// 4. Code splitting
const HeavyFeature = dynamic(() => import('./HeavyFeature'), {
  ssr: false,
  loading: () => <Skeleton />,
});

// 5. Prefetching
<Link href="/dashboard" prefetch={true}>Dashboard</Link>

// 6. Server Components — giảm JS bundle
// Di chuyển logic vào Server Components → không ship JS xuống client

// 7. Partial Prerendering (PPR — Next.js 14+, experimental)
// Kết hợp static shell với dynamic content
export const experimental_ppr = true;

export default function Page() {
  return (
    <div>
      <StaticHeader />   {/* Prerendered tĩnh */}
      <Suspense fallback={<Skeleton />}>
        <DynamicFeed />  {/* Stream dynamically */}
      </Suspense>
    </div>
  );
}
```

**Performance checklist:**

```
Build time:
□ Phân tích bundle với @next/bundle-analyzer
□ Kiểm tra "First Load JS" — target < 100kB per route
□ Tree-shake unused imports

Images:
□ Dùng next/image cho tất cả images
□ Set priority cho LCP images
□ Dùng WebP/AVIF (tự động với next/image)
□ Đúng sizes prop cho responsive images

Fonts:
□ Dùng next/font thay vì Google Fonts trực tiếp
□ display: 'swap' để tránh FOIT

JavaScript:
□ Di chuyển logic sang Server Components khi có thể
□ Lazy load components nặng với next/dynamic
□ Code split theo route (tự động với App Router)

Caching:
□ Hiểu 4 cache layers và cấu hình phù hợp
□ Dùng revalidate cho ISR thay vì SSR khi có thể
□ Tags cho on-demand revalidation

Core Web Vitals (đo bằng Lighthouse / Vercel Analytics):
□ LCP (Largest Contentful Paint) < 2.5s
□ FID (First Input Delay) < 100ms
□ CLS (Cumulative Layout Shift) < 0.1
□ TTFB (Time to First Byte) < 800ms
```

**Monitoring và debugging:**

```tsx
// next.config.js — useful config
module.exports = {
  // Phân tích bundle
  experimental: {
    optimizePackageImports: ['@icons/pack', 'lodash'],
  },

  // Headers bảo mật
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
        ],
      },
    ];
  },

  // Redirects
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true, // 308, false = 307
      },
    ];
  },
};
```

---

*Tài liệu này được cập nhật cho Next.js 14 / 15 với App Router. Câu hỏi được sắp xếp từ Junior đến Senior theo từng topic.*
