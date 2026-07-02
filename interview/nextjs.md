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
16. [Server Actions](#16-server-actions)
17. [Route Groups, Parallel Routes, và Intercepting Routes](#17-route-groups-parallel-routes-va-intercepting-routes)
18. [generateMetadata — Dynamic SEO và Open Graph](#18-generatemetadata-dynamic-seo-va-open-graph)
19. [Authentication — Middleware, NextAuth.js, và Session Management](#19-authentication-middleware-nextauthjs-va-session-management)
20. [next/link — Prefetching và Client-side Navigation](#20-nextlink-prefetching-va-client-side-navigation)
21. [next.config.js — Cấu Hình Quan Trọng](#21-nextconfigjs-cau-hinh-quan-trong)

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "A new Next.js project — do you use Pages Router or App Router?"**

**Model Answer:**
"For any new project today, I'd use App Router — it's been stable since Next.js 14 and it's the direction the Next.js team is investing in.

App Router gives you Server Components by default, which means better performance (less JavaScript to the browser), built-in streaming with Suspense, nested layouts that are much cleaner than wrapping everything in `_app.tsx`, and a more intuitive data fetching model using `async/await` directly in components.

The only case where I'd still reach for Pages Router is if I'm joining an existing project already on Pages Router and migration isn't justified — because both routers can coexist in the same app during migration."

**Trả lời (Tiếng Việt):**
"Với bất kỳ dự án mới nào hiện tại, tôi sẽ dùng App Router — nó đã ổn định từ Next.js 14 và đây là hướng mà team Next.js đang đầu tư phát triển.

App Router mặc định sử dụng Server Components, nghĩa là hiệu suất tốt hơn (ít JavaScript gửi về browser), hỗ trợ streaming tích hợp sẵn với Suspense, nested layouts gọn gàng hơn nhiều so với việc bọc mọi thứ trong `_app.tsx`, và mô hình data fetching trực quan hơn khi dùng `async/await` trực tiếp trong component.

Trường hợp duy nhất tôi vẫn dùng Pages Router là khi tham gia dự án đang chạy Pages Router mà việc migration không được justify — vì hai router có thể cùng tồn tại trong một app trong quá trình chuyển đổi."

---

**Q2 (Mid): "Explain the biggest difference in data fetching between Pages Router and App Router."**

**Model Answer:**
"In Pages Router, data fetching was 'page-level' — you wrote special functions like `getServerSideProps` or `getStaticProps` at the bottom of the page file, and Next.js called them before rendering. It was separate from the component itself.

In App Router, data fetching moved **into the components**. Any Server Component can be `async` and use `await` directly:

```tsx
// Pages Router — data fetching is separate from the component
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}
export default function Page({ data }) { ... } // receives data as props

// App Router — data fetching is inside the component
export default async function Page() {
  const data = await fetchData(); // async directly in component
  return <div>{data.title}</div>;
}
```

The App Router approach is more co-located and composable — each component in the tree can independently fetch its own data in parallel, rather than one big `getServerSideProps` at the top doing everything sequentially."

**Trả lời (Tiếng Việt):**
"Trong Pages Router, data fetching hoạt động ở cấp độ 'page' — bạn viết các function đặc biệt như `getServerSideProps` hay `getStaticProps` ở cuối file page, và Next.js gọi chúng trước khi render. Logic này hoàn toàn tách biệt khỏi component.

Trong App Router, data fetching chuyển **vào trong component**. Bất kỳ Server Component nào cũng có thể là `async` và dùng `await` trực tiếp.

Cách tiếp cận của App Router co-located và composable hơn — mỗi component trong cây có thể tự fetch data của mình song song, thay vì một `getServerSideProps` lớn ở trên cùng xử lý tuần tự."

---

**Q3 (Mid): "What are nested layouts and why are they useful?"**

**Model Answer:**
"Nested layouts are one of the most practical improvements in App Router. The idea is that different parts of your URL tree can have their own persistent layout that doesn't remount when child routes change.

Example: an admin dashboard where the sidebar and header stay mounted while the main content changes:

```
app/
├── layout.tsx         → Root layout (applies everywhere)
├── (auth)/
│   └── layout.tsx     → Auth layout: just centered login form
├── dashboard/
│   ├── layout.tsx     → Dashboard layout: sidebar + header (persists!)
│   ├── page.tsx       → /dashboard (overview)
│   ├── users/
│   │   └── page.tsx   → /dashboard/users
│   └── settings/
│       └── page.tsx   → /dashboard/settings
```

When user navigates from `/dashboard/users` to `/dashboard/settings`, the `dashboard/layout.tsx` does NOT remount — only the inner `page.tsx` swaps. This means:
- The sidebar stays in place without flash
- Any state in the layout (like 'active menu item') is preserved
- Better performance — no unnecessary DOM destruction/recreation

In Pages Router, you had one global `_app.tsx` and had to manually manage conditional layout logic. Nested layouts make this declarative and automatic."

**Trả lời (Tiếng Việt):**
"Nested layouts là một trong những cải tiến thực tế nhất của App Router. Ý tưởng là các phần khác nhau trong URL tree có thể có layout riêng, layout này không bị remount khi route con thay đổi.

Ví dụ: một admin dashboard mà sidebar và header vẫn được giữ nguyên trong khi nội dung chính thay đổi.

Khi người dùng điều hướng từ `/dashboard/users` sang `/dashboard/settings`, `dashboard/layout.tsx` KHÔNG bị remount — chỉ có `page.tsx` bên trong được hoán đổi. Điều này có nghĩa là sidebar giữ nguyên vị trí không bị flash, state trong layout được bảo toàn, và hiệu suất tốt hơn vì không phá hủy/tạo lại DOM không cần thiết.

Trong Pages Router, bạn chỉ có một `_app.tsx` toàn cục và phải tự quản lý logic layout theo điều kiện. Nested layouts giúp việc này trở nên declarative và tự động."

---

## 3. SSR vs SSG vs ISR vs CSR

**[Junior → Senior]**

**EN:** Explain the four rendering strategies in Next.js: SSR, SSG, ISR, and CSR. When should you use each?

**VI:** Giải thích 4 chiến lược rendering trong Next.js: SSR, SSG, ISR, và CSR. Khi nào dùng mỗi loại?

---

### 📖 Tên đầy đủ + Định nghĩa một câu

> Đọc phần này trước — hiểu tên rồi mới đọc chi tiết bên dưới.

| Viết tắt | Tên đầy đủ | Định nghĩa một câu | Dùng khi nào |
|---|---|---|---|
| **CSR** | **C**lient-**S**ide **R**endering — Render phía Client | Browser nhận HTML rỗng, JavaScript tải về rồi tự render UI | Dashboard sau đăng nhập, app cần nhiều tương tác, không cần SEO |
| **SSG** | **S**tatic **S**ite **G**eneration — Tạo trang tĩnh | HTML được tạo sẵn 1 lần lúc build, lưu trên CDN, serve cho mọi người | Blog, landing page, docs — content ít thay đổi, cần SEO |
| **ISR** | **I**ncremental **S**tatic **R**egeneration — Tái tạo tĩnh tăng dần | Giống SSG nhưng HTML tự động làm mới sau X giây, không cần deploy lại | E-commerce catalog, news — cần SEO nhưng data thỉnh thoảng thay đổi |
| **SSR** | **S**erver-**S**ide **R**endering — Render phía Server | Server tạo HTML mới cho từng request, có thể đọc cookie/session | Trang cá nhân hóa (my orders, profile), search results cần SEO |

**Câu hỏi cốt lõi để nhớ:** → **"HTML được tạo ra ở đâu và khi nào?"**

- CSR: trong **browser**, sau khi user vào trang
- SSG: trên **server**, **lúc build** (một lần duy nhất)
- ISR: trên **server**, **lúc build** + tự động làm mới định kỳ
- SSR: trên **server**, **mỗi lần có request** mới

---

### 🧠 Mental Model — Analogy Nhà Hàng

> Cách dễ nhớ nhất khi giải thích trong interview:

| Chiến lược | Analogy nhà hàng | Điểm khác biệt |
|---|---|---|
| **SSG** | Cơm hộp đóng sẵn từ sáng | Chuẩn bị từ trước (build time), ai vào cũng nhận được cùng hộp cơm đó |
| **ISR** | Cơm hộp đóng sẵn, nhưng cứ 1 tiếng làm mới lại | Vẫn nhanh, nhưng định kỳ cập nhật — không bao giờ quá cũ |
| **SSR** | Đầu bếp nấu theo order mỗi lần khách gọi | Mỗi request server phải làm việc, nhưng đồ ăn luôn mới nhất |
| **CSR** | Khách tự vào bếp tự nấu | Server không làm gì, browser tự fetch data và render |

---

### 📊 Timeline — HTML được tạo khi nào?

```
BUILD TIME          DEPLOY         REQUEST TIME           BROWSER
    │                  │                 │                    │
    │                  │                 │                    │
SSG:│──[generate HTML]─►│──[serve static]─►│──────────────────►│ (HTML sẵn rồi)
    │                  │                 │                    │
ISR:│──[generate HTML]─►│──[serve static]─►│──────────────────►│ (nhanh như SSG)
    │                  │   (sau 1h, bg   │                    │
    │                  │    regenerate)  │                    │
    │                  │                 │                    │
SSR:│                  │                 │──[server generates]►│ (mỗi request)
    │                  │                 │    HTML on-the-fly │
    │                  │                 │                    │
CSR:│                  │                 │──[empty HTML]──────►│──[fetch data]
    │                  │                 │                    │  [render UI]
```

---

### 1️⃣ SSG — Static Site Generation

**Khi nào HTML được tạo?** Lúc `next build` chạy — một lần duy nhất.

**Hoạt động như thế nào:**
1. Developer chạy `next build`
2. Next.js gọi `generateStaticParams()` (App Router) hoặc `getStaticPaths()` + `getStaticProps()` (Pages Router)
3. HTML file được tạo ra và lưu trên CDN/filesystem
4. Mọi user sau đó nhận đúng cái HTML đó — không server nào phải làm việc thêm

**Điểm mạnh:**
- Nhanh nhất có thể — serve từ CDN, không cần server compute
- Không có server = không có điểm fail

**Điểm yếu:**
- Nếu data thay đổi, phải build lại — không phù hợp với content động
- Build time tăng tuyến tính với số lượng pages

**Dùng khi:** Blog, documentation, landing page marketing, portfolio — những thứ không đổi thường xuyên

```tsx
// App Router — SSG
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetchAllPosts(); // Chạy LÚC BUILD
  return posts.map(post => ({ slug: post.slug }));
}

export default async function BlogPost({ params }) {
  const post = await fetchPost(params.slug); // Cũng chạy lúc build
  return <article>{post.content}</article>;
}
// → Next.js tạo ra /blog/post-1.html, /blog/post-2.html... tại build time
```

```tsx
// Pages Router — SSG
export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: false, // path không trong list → 404
    // fallback: 'blocking' → SSR cho path mới (không có trong list)
  };
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  return { props: { post } };
}
```

---

### 2️⃣ ISR — Incremental Static Regeneration

**Khi nào HTML được tạo?** Lúc build, nhưng sau đó **tự động regenerate** theo thời gian (hoặc on-demand).

**Hoạt động như thế nào:**
1. Build time: HTML được tạo như SSG
2. User vào → nhận HTML cũ (nhanh như CDN)
3. Nếu đã quá thời gian `revalidate` (ví dụ 1 tiếng):
   - Request đầu tiên sau khi hết hạn: user **vẫn nhận HTML cũ** (stale-while-revalidate)
   - Background: Next.js regenerate HTML mới
   - Request tiếp theo: nhận HTML mới

**Điểm quan trọng — Stale-While-Revalidate:**
```
User 1 (11:00) → nhận HTML của 10:00 (fresh)
User 2 (12:01) → nhận HTML của 10:00 (stale, expired), NHƯNG trigger background regenerate
User 3 (12:02) → nhận HTML mới nhất (fresh)
```

**Dùng khi:** E-commerce (giá sản phẩm, inventory), news site, dashboard metrics — data thay đổi nhưng không cần realtime 100%

```tsx
// App Router — ISR với time-based revalidation
export default async function ProductPage({ params }) {
  const product = await fetch(`https://api.store.com/products/${params.id}`, {
    next: { revalidate: 3600 }, // Tái tạo sau mỗi 1 tiếng
  }).then(r => r.json());

  return <ProductDetails product={product} />;
}

// App Router — On-demand ISR (khi CMS thay đổi content)
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const { secret, slug } = await request.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  revalidatePath(`/products/${slug}`); // Invalidate specific page
  // Hoặc: revalidateTag('products') để invalidate tất cả page có tag này

  return Response.json({ revalidated: true, at: new Date().toISOString() });
}
```

```tsx
// Pages Router — ISR
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return {
    props: { product },
    revalidate: 3600, // seconds — đây là điểm khác biệt vs SSG
  };
}
```

---

### 3️⃣ SSR — Server-Side Rendering

**Khi nào HTML được tạo?** Mỗi lần có **request mới** — server phải làm việc cho từng user.

**Hoạt động như thế nào:**
1. User mở URL
2. Request đến Next.js server
3. Server fetch data (từ DB, API...) dựa trên request context (cookies, headers, query params)
4. Server render HTML với data đó
5. HTML được gửi về browser — browser chỉ cần hiển thị, không cần fetch thêm

**Điểm mạnh:**
- Data luôn mới nhất — cực kỳ quan trọng khi dùng `req.cookies` để lấy data cá nhân hóa
- Vẫn có SEO vì server trả về HTML đầy đủ (khác với CSR)

**Điểm yếu:**
- Chậm hơn SSG/ISR — mỗi request server phải compute
- Time To First Byte (TTFB) cao hơn — user phải chờ server xử lý

**Dùng khi:** User-specific pages (dashboard cá nhân, order history, profile), search results cần SEO, pages cần data từ cookies/session

```tsx
// App Router — SSR
import { cookies } from 'next/headers';
import { unstable_noStore as noStore } from 'next/cache';

export default async function UserDashboard() {
  noStore(); // Quan trọng: opt-out cache → force SSR mỗi request

  const cookieStore = cookies();
  const token = cookieStore.get('auth-token')?.value;

  const user = await fetchUser(token); // Chạy TRÊN SERVER mỗi request
  const orders = await fetchUserOrders(user.id);

  return (
    <div>
      <h1>Xin chào, {user.name}!</h1>
      <OrderList orders={orders} />
    </div>
  );
}

// Hoặc dùng export const dynamic:
export const dynamic = 'force-dynamic'; // Tất cả fetch trong component này sẽ không cache
```

```tsx
// Pages Router — SSR
import { GetServerSideProps } from 'next';

export const getServerSideProps: GetServerSideProps = async (context) => {
  const { req, res, params, query } = context;

  // Có thể đọc cookies, headers từ req
  const token = req.cookies['auth-token'];
  const user = await fetchUser(token);

  if (!user) {
    return {
      redirect: { destination: '/login', permanent: false },
    };
  }

  return {
    props: { user },
    // Không có revalidate → không phải ISR
  };
};

export default function UserDashboard({ user }) {
  return <div>Xin chào, {user.name}!</div>;
}
```

---

### 4️⃣ CSR — Client-Side Rendering

**Khi nào HTML được tạo?** Trong **browser của user** — JavaScript tải về rồi tự render.

**Hoạt động như thế nào:**
1. Server gửi về HTML rỗng (chỉ có `<div id="root"></div>`)
2. Browser tải JavaScript bundle
3. JavaScript chạy → fetch data từ API
4. React render component với data nhận được

**Điểm mạnh:**
- Sau lần load đầu, navigation giữa các page rất nhanh (không reload)
- Phù hợp với app phức tạp, interactive cao

**Điểm yếu:**
- SEO kém — crawler thấy HTML rỗng
- Performance lần đầu chậm — user thấy loading spinner
- JavaScript phải tải xong mới thấy content

**Dùng khi:** Admin panel, user dashboard sau đăng nhập, real-time app (chat, live data) — những thứ không cần SEO

```tsx
// App Router — CSR bắt buộc dùng 'use client' + useEffect/fetch
'use client';
import { useEffect, useState } from 'react';

export default function UserDashboard() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/user/dashboard')
      .then(r => r.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <DashboardSkeleton />;
  return <Dashboard data={data} />;
}
```

```tsx
// Thực tế: dùng TanStack Query thay vì useEffect thô
'use client';
import { useQuery } from '@tanstack/react-query';

export default function UserDashboard() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', 'dashboard'],
    queryFn: () => fetch('/api/user/dashboard').then(r => r.json()),
  });

  if (isLoading) return <DashboardSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  return <Dashboard data={data} />;
}
```

---

### 📋 Bảng So Sánh Tổng Hợp

| | **SSG** | **ISR** | **SSR** | **CSR** |
|---|---|---|---|---|
| **HTML tạo khi nào** | Build time | Build time + background | Mỗi request | Trong browser |
| **Tốc độ (TTFB)** | ⚡⚡⚡ Nhanh nhất | ⚡⚡⚡ Gần như SSG | ⚡⚡ Trung bình | ⚡ Chậm nhất |
| **Data có thể realtime?** | ❌ Không | ⚠️ Gần realtime | ✅ Luôn mới | ✅ Fetch khi cần |
| **SEO** | ✅ Tốt | ✅ Tốt | ✅ Tốt | ❌ Kém |
| **Dùng cookies/session** | ❌ Không | ❌ Không | ✅ Có | ✅ Có (client-side) |
| **Server load** | Không (sau build) | Thấp | Cao | Không |
| **Ví dụ thực tế** | Blog, docs, marketing | E-commerce catalog | Search, user profile | Dashboard, chat |

---

### 🌳 Decision Tree — Chọn Strategy nào?

```
Cần SEO không?
├── KHÔNG → CSR (dashboard, admin, real-time app)
└── CÓ → Data thay đổi theo từng user không?
         ├── CÓ (cần cookies/session) → SSR
         └── KHÔNG → Data thay đổi thường xuyên không?
                     ├── Không bao giờ / rất hiếm → SSG
                     ├── Thỉnh thoảng (hàng giờ, hàng ngày) → ISR
                     └── Mọi lúc (realtime) → SSR
```

---

### 🏢 Áp Dụng Vào Lumin PDF (Thực Tế)

Nếu Lumin PDF dùng Next.js, đây là cách mỗi trang sẽ được render:

| Page | Strategy | Lý do |
|---|---|---|
| Landing page (`/`) | SSG | Content tĩnh, cần SEO tốt |
| Pricing page (`/pricing`) | ISR (revalidate: 3600) | Giá thay đổi không thường xuyên, cần SEO |
| Help center / blog | SSG + ISR | Content do team viết, ít thay đổi |
| User dashboard (`/app`) | CSR | Sau đăng nhập, không cần SEO, nhiều tương tác |
| PDF Editor (`/edit/[id]`) | SSR → CSR hybrid | SSR cho initial load + auth check; CSR cho editor UI |
| Shared document (`/view/[id]`) | SSR | Cần kiểm tra permissions từ server mỗi request |

---

### 🎤 Mock Interview — Q&A

> **Cách dùng:** Đọc câu hỏi, tự trả lời bằng lời của bạn, rồi so sánh với đáp án mẫu bên dưới.

---

**Q1 (Junior): "Can you explain the difference between SSR and SSG?"**

> *Đây là câu mở đầu phổ biến nhất. Trả lời ngắn gọn, rõ ràng.*

**Model Answer:**
"Sure! The key difference is **when the HTML is generated**.

With SSG — Static Site Generation — the HTML is built once at build time. When a user visits the page, they get a pre-built HTML file served from a CDN. It's extremely fast but the content is fixed until you redeploy.

With SSR — Server-Side Rendering — the HTML is generated fresh on the server for every single request. This means you can include user-specific data, read cookies, or query a database based on who's visiting. The trade-off is higher latency since the server must compute the response each time.

Think of SSG like a pre-printed magazine — printed once, distributed to everyone. SSR is like a customized newspaper printed on demand for each reader."

**Trả lời (Tiếng Việt):**
"Sự khác biệt chính là **thời điểm HTML được tạo ra**.

Với SSG — Static Site Generation — HTML được build một lần tại thời điểm build. Khi người dùng truy cập trang, họ nhận được file HTML được build sẵn từ CDN. Cực kỳ nhanh nhưng nội dung cố định cho đến khi bạn redeploy.

Với SSR — Server-Side Rendering — HTML được tạo mới trên server cho mỗi request. Điều này cho phép bạn đưa vào dữ liệu cụ thể theo người dùng, đọc cookie, hoặc truy vấn database dựa trên người đang truy cập. Đánh đổi là latency cao hơn vì server phải tính toán response mỗi lần.

Hãy nghĩ SSG như một tạp chí in sẵn — in một lần, phát cho mọi người. SSR như tờ báo tùy chỉnh được in theo yêu cầu cho từng độc giả."

---

**Q2 (Junior): "What is ISR and how is it different from SSG?"**

**Model Answer:**
"ISR — Incremental Static Regeneration — is basically SSG with a built-in refresh mechanism.

With plain SSG, once a page is built, it stays that way until your next deployment. ISR adds a `revalidate` option where you specify how many seconds the cached page should be considered valid. After that window expires, the next request triggers a background regeneration while still serving the stale page. The subsequent request then gets the freshly generated page.

In code it's as simple as adding `next: { revalidate: 3600 }` to a fetch call in App Router.

The real-world benefit is you get CDN-level performance for most users, while the content updates automatically — without needing to redeploy. Perfect for e-commerce product pages where prices change occasionally but not in real-time."

**Trả lời (Tiếng Việt):**
"ISR — Incremental Static Regeneration — về cơ bản là SSG với cơ chế refresh tích hợp sẵn.

Với SSG thông thường, một khi trang đã được build thì nó sẽ như vậy mãi cho đến lần deploy tiếp theo. ISR thêm tùy chọn `revalidate` để bạn chỉ định số giây mà trang cached được coi là còn hợp lệ. Sau khi hết thời gian đó, request tiếp theo sẽ kích hoạt quá trình regeneration ở nền trong khi vẫn phục vụ trang cũ. Request sau đó sẽ nhận trang được tạo mới.

Trong code chỉ cần thêm `next: { revalidate: 3600 }` vào fetch call trong App Router là xong.

Lợi ích thực tế là bạn có hiệu suất CDN cho hầu hết người dùng, trong khi nội dung tự động cập nhật — không cần redeploy. Lý tưởng cho các trang product e-commerce khi giá thay đổi không thường xuyên nhưng không phải real-time."

---

**Q3 (Mid): "When would you choose SSR over ISR? Give me a concrete example."**

**Model Answer:**
"I'd choose SSR when the content is **personalized per user** or depends on request-specific data like cookies, auth tokens, or query parameters.

For example, at Herond Labs I worked on a user portal where we showed personalized data based on the authenticated user's session. There was no way to pre-generate that page because each user sees completely different content. We had to read the auth cookie server-side and query the database for that specific user.

ISR wouldn't work here because ISR generates one version of the page that all users see. There's no concept of 'user A sees this, user B sees that' in ISR.

The rule of thumb I use: if the page contains the words 'my' — my orders, my profile, my documents — it's almost always SSR or CSR, not SSG or ISR."

**Trả lời (Tiếng Việt):**
"Tôi sẽ chọn SSR khi nội dung được **cá nhân hóa theo từng người dùng** hoặc phụ thuộc vào dữ liệu đặc thù của request như cookie, auth token, hay query parameter.

Ví dụ, tại Herond Labs tôi từng làm một user portal hiển thị dữ liệu cá nhân hóa dựa trên session của người dùng đã đăng nhập. Không thể pre-generate trang đó vì mỗi người dùng thấy nội dung hoàn toàn khác nhau. Chúng tôi phải đọc auth cookie phía server và truy vấn database cho người dùng cụ thể đó.

ISR sẽ không hoạt động ở đây vì ISR tạo ra một phiên bản trang mà tất cả người dùng đều thấy. Không có khái niệm 'user A thấy cái này, user B thấy cái kia' trong ISR.

Quy tắc tôi dùng: nếu trang có chứa từ 'của tôi' — đơn hàng của tôi, hồ sơ của tôi, tài liệu của tôi — thì hầu như chắc chắn là SSR hoặc CSR, không phải SSG hay ISR."

---

**Q4 (Mid): "You mentioned CSR is bad for SEO. Why exactly? And when would you still use CSR despite that?"**

**Model Answer:**
"When a page uses CSR, the server sends back an essentially empty HTML document — just a `<div id='root'>` and a script tag. When Google's crawler visits the page, it initially sees that empty HTML. While modern Googlebot does execute JavaScript, it's slower and less reliable than indexing pre-rendered HTML.

The bigger issue is that other crawlers — social media preview scrapers for Open Graph tags, for instance — often don't execute JavaScript at all. So your LinkedIn or Twitter shares would show blank previews.

That said, CSR is absolutely the right choice for:
- **Authenticated experiences** — admin dashboards, user settings, anything behind a login wall that you don't want indexed anyway
- **Highly interactive apps** — PDF editors, canvas tools, real-time collaboration — where the interactivity is the product and SEO is irrelevant
- **Data that changes constantly** — live trading charts, real-time analytics

At Lumin PDF, the PDF editor itself is a perfect CSR use case. It's behind authentication, it's highly interactive, and there's nothing to index. But their marketing pages absolutely need SSG."

**Trả lời (Tiếng Việt):**
"Khi một trang dùng CSR, server trả về một document HTML gần như trống — chỉ là một `<div id='root'>` và một script tag. Khi Googlebot truy cập trang, ban đầu nó thấy HTML trống đó. Dù Googlebot hiện đại có thực thi JavaScript, nhưng nó chậm hơn và kém tin cậy hơn so với indexing HTML đã được render sẵn.

Vấn đề lớn hơn là các crawler khác — như social media preview scraper cho Open Graph tags chẳng hạn — thường không thực thi JavaScript. Vậy nên khi chia sẻ link trên LinkedIn hay Twitter sẽ hiển thị preview trống.

Tuy vậy, CSR hoàn toàn là lựa chọn đúng cho:
- **Trải nghiệm sau đăng nhập** — admin dashboard, cài đặt người dùng, bất kỳ thứ gì sau một login wall mà bạn không muốn được index
- **Ứng dụng highly interactive** — PDF editor, canvas tool, real-time collaboration — khi tính tương tác chính là sản phẩm và SEO không liên quan
- **Dữ liệu thay đổi liên tục** — biểu đồ trading live, analytics real-time

Tại Lumin PDF, bản thân PDF editor là use case CSR hoàn hảo. Nó sau tường đăng nhập, rất interactive, và không có gì để index. Nhưng các trang marketing của họ thì chắc chắn cần SSG."

---

**Q5 (Mid): "Explain the stale-while-revalidate pattern in ISR."**

**Model Answer:**
"This is one of my favorite features of ISR. It ensures the page is always fast, even when it's outdated.

Here's the sequence: Let's say a product page has `revalidate: 3600` — one hour.

- 10:00 AM — page is generated fresh.
- 11:00 AM — the revalidation window expires.
- 11:01 AM — User A visits. They **still get the 10:00 AM version** (stale but instant). In the background, Next.js starts regenerating the page.
- 11:02 AM — User B visits. Now they get the **freshly regenerated** page.

The key insight is that no user ever waits for a regeneration. The stale response is served immediately, and the regeneration happens asynchronously. This is why ISR feels as fast as plain SSG for the end user.

The trade-off: there's always a potential window where users see data that's up to `revalidate` seconds old. That's fine for product prices, but not fine for stock prices."

**Trả lời (Tiếng Việt):**
"Đây là một trong những tính năng tôi thích nhất của ISR. Nó đảm bảo trang luôn nhanh, kể cả khi đã lỗi thời.

Đây là chuỗi: Giả sử một trang product có `revalidate: 3600` — một tiếng.

- 10:00 SA — trang được tạo mới.
- 11:00 SA — cửa sổ revalidation hết hạn.
- 11:01 SA — User A truy cập. Họ **vẫn nhận bản 10:00 SA** (cũ nhưng tức thì). Phía sau, Next.js bắt đầu regenerate trang.
- 11:02 SA — User B truy cập. Bây giờ họ nhận được trang **vừa được regenerate**.

Điểm mấu chốt là không có người dùng nào phải chờ quá trình regeneration. Response cũ được phục vụ ngay lập tức, và việc regenerate diễn ra bất đồng bộ. Đó là lý do ISR cảm thấy nhanh như SSG thuần với người dùng cuối.

Đánh đổi: luôn có một khoảng thời gian mà người dùng có thể thấy dữ liệu cũ đến `revalidate` giây. Ổn thôi với giá sản phẩm, nhưng không ổn với giá cổ phiếu."

---

**Q6 (Mid): "In Next.js App Router, how do you explicitly control which rendering mode a page uses?"**

**Model Answer:**
"The App Router changed how you think about rendering modes. By default, components are **Server Components**, which means they run on the server — but they're not necessarily SSR.

Here's how to control it:

**For SSG** (the default for static routes):
```tsx
// Static export or no dynamic data reads — Next.js auto-detects this as SSG
export default async function Page() {
  const data = await fetch('https://api.example.com/static-data');
  // If this fetch has no cache: 'no-store', it gets cached at build time → SSG
}
```

**For ISR**:
```tsx
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }, // This fetch = ISR behavior
});
```

**For SSR** (force dynamic per-request rendering):
```tsx
export const dynamic = 'force-dynamic'; // Force entire route to SSR

// Or per-fetch:
import { unstable_noStore as noStore } from 'next/cache';
noStore(); // This specific render won't be cached
```

**For CSR**:
```tsx
'use client'; // This component runs in the browser
import { useEffect, useState } from 'react';
```

The mental model is: by default Next.js tries to cache everything (SSG). You opt-out of caching to get SSR behavior."

**Trả lời (Tiếng Việt):**
"App Router thay đổi cách bạn nghĩ về rendering mode. Mặc định, các component là **Server Components**, nghĩa là chúng chạy trên server — nhưng không nhất thiết là SSR.

Đây là cách kiểm soát:

**Cho SSG** (mặc định cho static routes): nếu fetch không có `cache: 'no-store'`, Next.js tự phát hiện và cache tại build time.

**Cho ISR**: thêm `next: { revalidate: 3600 }` vào fetch call.

**Cho SSR** (force dynamic per-request rendering): export `dynamic = 'force-dynamic'` để ép toàn bộ route dùng SSR, hoặc dùng `noStore()` cho từng fetch riêng lẻ.

**Cho CSR**: thêm `'use client'` để component chạy trong browser.

Mental model: mặc định Next.js cố cache mọi thứ (SSG). Bạn opt-out khỏi cache để có behavior SSR."

---

**Q7 (Senior): "In a large Next.js app, different pages might need different rendering strategies. How do you architect this?"**

**Model Answer:**
"Exactly — and this is where Next.js really shines. You can mix strategies at the route level, not just the app level.

Let me use an e-commerce example:

```
/                    → SSG (landing page, CDN-cached)
/products            → ISR revalidate:3600 (product catalog)
/products/[id]       → ISR revalidate:900 (product detail, price changes)
/search              → SSR (query-param driven, needs fresh results)
/account/orders      → SSR (requires auth cookie + user-specific data)
/account/dashboard   → CSR (highly interactive, no SEO needed)
/checkout            → SSR (session data, fraud checks)
/api/*               → API routes (separate concern)
```

The key principle: **optimize each route independently** based on its data freshness requirements and SEO needs.

A common mistake is defaulting to SSR for everything 'to be safe.' SSR adds server load and latency. If a page's data doesn't change per-user and can be stale for an hour, use ISR. You'll save significantly on infrastructure costs at scale.

For Lumin PDF specifically, I'd envision the marketing site using SSG/ISR, and the actual document editing app being a CSR SPA — possibly even a separate Next.js app or just a Vite SPA behind the authenticated route."

**Trả lời (Tiếng Việt):**
"Đúng vậy — và đây là lúc Next.js thực sự tỏa sáng. Bạn có thể kết hợp các chiến lược ở cấp độ route, không chỉ cấp app.

Để tôi dùng ví dụ e-commerce: trang chủ dùng SSG, danh mục sản phẩm dùng ISR revalidate mỗi giờ, trang chi tiết sản phẩm ISR mỗi 15 phút vì giá thay đổi, trang tìm kiếm dùng SSR vì phụ thuộc query parameter, trang đơn hàng và checkout dùng SSR vì cần auth và dữ liệu cá nhân, dashboard người dùng dùng CSR vì highly interactive và không cần SEO.

Nguyên tắc chính: **tối ưu từng route độc lập** dựa trên yêu cầu về độ tươi của dữ liệu và nhu cầu SEO.

Sai lầm phổ biến là mặc định dùng SSR cho mọi thứ 'cho an toàn.' SSR tốn server load và latency. Nếu dữ liệu của trang không thay đổi theo người dùng và có thể cũ đến một tiếng, hãy dùng ISR. Bạn sẽ tiết kiệm đáng kể chi phí hạ tầng khi scale.

Với Lumin PDF, tôi hình dung marketing site dùng SSG/ISR, và app chỉnh sửa tài liệu thực tế là một CSR SPA — thậm chí có thể là một Next.js app riêng hoặc Vite SPA sau route đã xác thực."

---

**Q8 (Senior): "What happens in ISR when generateStaticParams doesn't include a path, and a user visits it for the first time?"**

**Model Answer:**
"This depends on the `fallback` option in Pages Router, or the `dynamicParams` export in App Router.

**Pages Router:**
- `fallback: false` → 404 immediately
- `fallback: true` → shows a fallback UI while the page generates in the background (first request does SSR, result gets cached as static)
- `fallback: 'blocking'` → first request SSR-blocks until the page is generated, then caches it — no fallback UI shown

**App Router:**
```tsx
export const dynamicParams = true;  // default: first visit generates page and caches
export const dynamicParams = false; // = fallback: false, unknown paths → 404
```

In practice I prefer `fallback: 'blocking'` (Pages) or `dynamicParams: true` (App Router) for e-commerce. The first user to visit a newly added product page waits slightly longer, but after that it's CDN-fast for everyone. No need to generate all 10,000 product pages at build time."

**Trả lời (Tiếng Việt):**
"Điều này phụ thuộc vào tùy chọn `fallback` trong Pages Router, hoặc export `dynamicParams` trong App Router.

**Pages Router:**
- `fallback: false` → 404 ngay lập tức
- `fallback: true` → hiển thị fallback UI trong khi trang đang generate ở nền (request đầu tiên làm SSR, kết quả được cache thành static)
- `fallback: 'blocking'` → request đầu tiên SSR-block cho đến khi trang được tạo, sau đó cache lại — không hiển thị fallback UI

**App Router:** export `dynamicParams = true` (mặc định) để request đầu tiên tạo và cache trang, hoặc `dynamicParams = false` để path không biết trả về 404.

Trong thực tế tôi thích `fallback: 'blocking'` (Pages) hoặc `dynamicParams: true` (App Router) cho e-commerce. Người dùng đầu tiên truy cập trang product mới thêm vào sẽ chờ lâu hơn một chút, nhưng sau đó CDN-fast cho mọi người. Không cần generate tất cả 10,000 trang product lúc build time."

---

**Q9 (Senior): "What are the performance implications of choosing SSR for too many pages?"**

**Model Answer:**
"This is a real production concern. SSR has several costs that compound at scale:

1. **Server compute cost** — every page view requires CPU time. At 100 req/s with 50ms per render, you're burning 5 CPU-seconds per second. You need horizontal scaling or serverless functions.

2. **TTFB (Time To First Byte)** — users wait for the server to finish rendering before they see anything. If your DB query takes 200ms, your TTFB is at least 200ms. CDN-cached SSG pages are typically <20ms TTFB.

3. **Database load** — every SSR page view might hit your database. A viral article under SSR can DB-hammer you; under ISR, the same viral load hits CDN and your DB barely notices.

4. **Cold starts in serverless** — if you're on Vercel/Lambda, SSR pages can experience cold start latency on the first request after idle periods.

The architecture pattern I've used: aggressive ISR with on-demand revalidation. Most pages are ISR. When data changes (CMS publish, price update), we call `revalidatePath` or `revalidateTag` via a webhook. Users always get fast CDN responses, and data freshness is event-driven rather than time-based."

**Trả lời (Tiếng Việt):**
"Đây là mối lo thực sự trong môi trường production. SSR có một số chi phí cộng dồn khi scale:

1. **Chi phí compute server** — mỗi page view cần thời gian CPU. Với 100 req/s và 50ms mỗi render, bạn đang tốn 5 CPU-second mỗi giây. Bạn cần horizontal scaling hoặc serverless functions.

2. **TTFB (Time To First Byte)** — người dùng chờ server render xong mới thấy gì. Nếu DB query mất 200ms, TTFB của bạn ít nhất 200ms. Trang SSG được cache CDN thường có TTFB dưới 20ms.

3. **Database load** — mỗi SSR page view có thể hit database. Một bài viết viral dưới SSR có thể làm DB của bạn quá tải; dưới ISR, cùng lượng traffic đó vào CDN và DB hầu như không để ý.

4. **Cold start trong serverless** — nếu bạn dùng Vercel hay Lambda, các trang SSR có thể bị cold start latency ở request đầu tiên sau khoảng thời gian idle.

Pattern kiến trúc tôi đã dùng: ISR tích cực với on-demand revalidation. Hầu hết trang là ISR. Khi dữ liệu thay đổi (CMS publish, price update), chúng tôi gọi `revalidatePath` hoặc `revalidateTag` qua webhook. Người dùng luôn nhận response CDN nhanh, và độ tươi của dữ liệu được điều khiển bởi event thay vì thời gian."

---

**Q10 (Bonus — Applied): "In Lumin PDF, how would you handle the PDF viewing page? The URL is `/view/[documentId]`. Some documents are public, some require login."**

**Model Answer:**
"Great question — this is a hybrid scenario.

For **public documents**: I'd use SSR so the server can generate a proper Open Graph meta tag with the document title and thumbnail for social sharing. It needs to be server-rendered for SEO and link previews.

But the actual PDF viewer UI is highly interactive — pan, zoom, annotate, comment. That interactive layer is definitely CSR.

My architecture:

```tsx
// app/view/[documentId]/page.tsx
import { notFound, redirect } from 'next/navigation';

export default async function ViewDocumentPage({ params }) {
  // SSR: check document visibility + auth
  const doc = await fetchDocumentMeta(params.documentId);
  if (!doc) notFound();

  if (doc.visibility === 'private') {
    const session = await getSession();
    if (!session || !canAccess(session.userId, doc)) {
      redirect(`/login?next=/view/${params.documentId}`);
    }
  }

  // Return shell with metadata — actual viewer is Client Component
  return (
    <>
      <DocumentMetaTags doc={doc} />  {/* For SEO/OG */}
      <PDFViewerClient documentId={params.documentId} /> {/* 'use client' */}
    </>
  );
}
```

This way:
- The SSR layer handles auth checks and generates SEO metadata
- The Client Component handles the interactive PDF rendering
- The user gets the right HTML from the server (no SEO gap), but the editor UX is fully client-driven

This 'shell SSR + interactive CSR island' pattern is something I've applied at Herond Labs for the browser extension dashboard and it works well."

**Trả lời (Tiếng Việt):**
"Câu hỏi hay — đây là một tình huống hybrid.

Với **tài liệu công khai**: tôi sẽ dùng SSR để server có thể tạo Open Graph meta tag với tiêu đề và thumbnail cho việc chia sẻ mạng xã hội. Nó cần được server-render cho SEO và link preview.

Nhưng bản thân PDF viewer UI rất interactive — pan, zoom, annotate, comment. Layer interactive đó chắc chắn là CSR.

Kiến trúc của tôi: một Server Component async xử lý kiểm tra quyền truy cập và auth, đồng thời trả về `DocumentMetaTags` cho SEO. Component `PDFViewerClient` với `'use client'` xử lý phần render PDF tương tác.

Bằng cách này:
- Layer SSR xử lý kiểm tra auth và tạo SEO metadata
- Client Component xử lý việc render PDF tương tác
- Người dùng nhận HTML đúng từ server (không có SEO gap), nhưng UX editor hoàn toàn do client điều khiển

Pattern 'shell SSR + interactive CSR island' này là thứ tôi đã áp dụng tại Herond Labs cho browser extension dashboard và nó hoạt động rất tốt."

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

### 🎤 Mock Interview — Q&A

> **Tip:** Đây là topic hay bị hỏi ở Mid→Senior. Interviewer muốn thấy bạn hiểu WHY, không chỉ HOW.

---

**Q1 (Mid): "What is a React Server Component and why was it introduced?"**

**Model Answer:**
"React Server Components are components that run exclusively on the server and never ship any JavaScript to the browser.

Before RSC, the problem was: if you wanted data from a database, you had to either expose an API endpoint (extra round trip + security exposure) or use SSR functions like `getServerSideProps`. And the JavaScript for those data-fetching components still downloaded to the client, even though it did nothing there.

RSC solves this by letting you write a component that directly queries a database, reads a file, or calls an internal service — and the result is just HTML/JSON sent to the client. Zero JavaScript bundle for that component. The client downloads nothing.

The practical benefit: a product page that used to need an API route, a client fetch, loading state, and error state — can now be a simple async function:
```tsx
async function ProductPage({ params }) {
  const product = await db.products.findById(params.id); // Direct DB access
  return <ProductDetails product={product} />;
}
```
No API, no fetch, no loading state, no bundle cost."

**Trả lời (Tiếng Việt):**
"React Server Components là các component chạy hoàn toàn trên server và không bao giờ gửi JavaScript nào về browser.

Trước RSC, vấn đề là: nếu bạn muốn dữ liệu từ database, bạn phải hoặc expose một API endpoint (thêm round trip + rủi ro bảo mật) hoặc dùng SSR function như `getServerSideProps`. Và JavaScript của những component fetch dữ liệu đó vẫn tải về client, dù nó không làm gì ở đó.

RSC giải quyết vấn đề này bằng cách cho phép bạn viết component trực tiếp query database, đọc file, hoặc gọi internal service — và kết quả chỉ là HTML/JSON gửi về client. Không có JavaScript bundle cho component đó. Client không tải gì cả.

Lợi ích thực tế: một product page mà trước đây cần API route, client fetch, loading state và error state — giờ chỉ cần là một async function đơn giản truy cập thẳng vào database. Không API, không fetch, không loading state, không bundle cost."

---

**Q2 (Mid): "When should I use a Client Component vs a Server Component?"**

**Model Answer:**
"The decision rule is simple: **start with Server Component by default, and opt into Client Component only when you need browser-specific capabilities.**

Use a **Client Component** when you need:
- `useState`, `useReducer` — interactive state
- `useEffect` — side effects, subscriptions, browser APIs
- Event handlers (`onClick`, `onChange`) — user interaction
- Browser-only APIs (`window`, `document`, `localStorage`)
- Third-party libraries that haven't been updated for RSC

Use a **Server Component** when you need:
- Database or file system access
- Sensitive environment variables (API keys you can't expose)
- Reducing bundle size — heavy computation libraries stay on the server
- SEO-critical content that doesn't need interaction

A good mental model: Server Components are like the 'backend' of your frontend. They prepare the data and HTML. Client Components are the 'interactive shell' that handles user events.

In practice, most pages are a mix: the outer layout fetches data (Server Component), the interactive parts like a search bar or like button are Client Components."

**Trả lời (Tiếng Việt):**
"Quy tắc quyết định rất đơn giản: **mặc định bắt đầu bằng Server Component, và chỉ chuyển sang Client Component khi bạn cần khả năng của browser.**

Dùng **Client Component** khi cần:
- `useState`, `useReducer` — interactive state
- `useEffect` — side effects, subscriptions, browser API
- Event handler (`onClick`, `onChange`) — tương tác người dùng
- Browser-only API (`window`, `document`, `localStorage`)
- Thư viện bên thứ ba chưa được cập nhật cho RSC

Dùng **Server Component** khi cần:
- Truy cập database hoặc file system
- Biến môi trường nhạy cảm (API key không thể expose)
- Giảm bundle size — các thư viện tính toán nặng ở lại server
- Nội dung SEO-critical không cần tương tác

Mental model tốt: Server Components là phần 'backend' của frontend. Chúng chuẩn bị dữ liệu và HTML. Client Components là 'interactive shell' xử lý sự kiện người dùng.

Trong thực tế, hầu hết trang là sự kết hợp: layout ngoài fetch dữ liệu (Server Component), các phần tương tác như thanh tìm kiếm hay nút like là Client Components."

---

**Q3 (Mid): "What is the 'use client' boundary? What happens when you add it?"**

**Model Answer:**
"The `'use client'` directive is a boundary declaration. It tells Next.js: 'this file and everything it imports is a Client Component.'

The important part is: **the boundary propagates downward through imports.** If Component A has `'use client'` and imports Component B, then B also becomes a Client Component — even if it didn't have the directive itself.

```tsx
// A.tsx
'use client'; // Boundary declared here
import { B } from './B'; // B is now also a Client Component
import { C } from './C'; // C is also a Client Component
```

This is why you want to **push `'use client'` as deep as possible** in the component tree. If only a small button needs interactivity, don't put `'use client'` on the entire page — put it on just that button component. Everything else stays as Server Components.

The 'leaf strategy': identify the interactive leaf nodes in your tree and add `'use client'` there. Keep the root and branch components as Server Components to maximize server-side data fetching and minimize bundle size."

**Trả lời (Tiếng Việt):**
"Directive `'use client'` là một khai báo ranh giới. Nó nói với Next.js: 'file này và mọi thứ nó import là Client Component.'

Điểm quan trọng là: **ranh giới này lan truyền xuống qua các import.** Nếu Component A có `'use client'` và import Component B, thì B cũng trở thành Client Component — dù nó không có directive đó.

Đó là lý do bạn nên **đẩy `'use client'` xuống càng sâu càng tốt** trong cây component. Nếu chỉ một button nhỏ cần interactivity, đừng đặt `'use client'` lên toàn bộ trang — đặt nó trên chính component button đó thôi. Mọi thứ khác giữ là Server Components.

Chiến lược 'leaf': xác định các leaf node tương tác trong cây và thêm `'use client'` ở đó. Giữ các component gốc và nhánh là Server Components để tối đa hóa data fetching phía server và giảm thiểu bundle size."

---

**Q4 (Mid): "I have a Server Component that fetches user data. I want to pass that data to a Client Component that shows a like button. How do I do this?"**

**Model Answer:**
"Simple — pass the data as props. Server Components can pass serializable data (strings, numbers, objects, arrays) directly to Client Components:

```tsx
// ServerPage.tsx (Server Component)
async function ProductPage({ params }) {
  const product = await db.products.findById(params.id);
  
  return (
    <div>
      <h1>{product.name}</h1>
      {/* Pass server-fetched data to Client Component as props */}
      <LikeButton productId={product.id} initialLikes={product.likes} />
    </div>
  );
}

// LikeButton.tsx (Client Component)
'use client';
import { useState } from 'react';

export function LikeButton({ productId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  
  const handleLike = async () => {
    await fetch(`/api/products/${productId}/like`, { method: 'POST' });
    setLikes(prev => prev + 1);
  };
  
  return <button onClick={handleLike}>❤️ {likes}</button>;
}
```

The key constraint: **only serializable data can cross the Server→Client boundary.** You can't pass a function, a class instance, a Date object (it becomes a string), or a database connection. You pass plain data, not behavior."

**Trả lời (Tiếng Việt):**
"Đơn giản — truyền dữ liệu qua props. Server Components có thể truyền dữ liệu serializable (chuỗi, số, object, array) trực tiếp đến Client Components.

Server Component async fetch dữ liệu product từ database, rồi truyền `productId` và `initialLikes` xuống `LikeButton` là một Client Component. `LikeButton` nhận `initialLikes` qua props, khởi tạo `useState` với giá trị đó, rồi xử lý việc like thông qua API call.

Ràng buộc quan trọng: **chỉ dữ liệu serializable mới có thể vượt qua ranh giới Server→Client.** Bạn không thể truyền function, class instance, Date object (nó sẽ thành string), hoặc database connection. Bạn truyền dữ liệu thuần, không phải behavior."

---

**Q5 (Senior): "Explain the composition pattern where a Server Component is passed as `children` to a Client Component."**

**Model Answer:**
"This is one of the most important and often misunderstood patterns in RSC.

The rule is: **a Client Component cannot directly import a Server Component**, because importing it would pull it into the client bundle. But a Client Component **can** receive a Server Component as `children` props — because `children` is passed from the parent, which runs on the server.

```tsx
// ❌ Wrong: importing RSC inside Client Component
'use client';
import { ServerUserProfile } from './ServerUserProfile'; // ❌ This gets bundled as client code!

// ✅ Right: pass RSC as children from a Server Component parent
// app/page.tsx (Server Component — parent)
import { Modal } from '@/components/Modal'; // Client Component
import { UserProfile } from '@/components/UserProfile'; // Server Component

export default function Page() {
  return (
    <Modal>
      <UserProfile /> {/* This runs on the server! Not imported by Modal */}
    </Modal>
  );
}

// components/Modal.tsx (Client Component)
'use client';
export function Modal({ children }) {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <>
      <button onClick={() => setIsOpen(true)}>Open</button>
      {isOpen && <div className="modal">{children}</div>}
    </>
  );
}
```

Why does this work? Modal doesn't import UserProfile — it just receives `children` as a prop (already rendered HTML from the server). So Modal has no knowledge of the server-side code in UserProfile.

This pattern enables the best of both worlds: Modal handles interactivity (open/close state), while UserProfile fetches real user data directly from the database. Neither compromises the other."

**Trả lời (Tiếng Việt):**
"Đây là một trong những pattern quan trọng nhất và thường bị hiểu nhầm nhất trong RSC.

Quy tắc là: **một Client Component không thể trực tiếp import một Server Component**, vì import nó sẽ kéo nó vào client bundle. Nhưng một Client Component **có thể** nhận Server Component dưới dạng prop `children` — vì `children` được truyền từ component cha, thứ chạy trên server.

Cách đúng: trong Server Component cha (ví dụ `app/page.tsx`), bạn import cả `Modal` (Client Component) lẫn `UserProfile` (Server Component), rồi truyền `<UserProfile />` làm children của `Modal`. `Modal` không biết gì về code phía server trong `UserProfile` — nó chỉ nhận `children` là HTML đã render sẵn từ server.

Pattern này cho phép kết hợp tốt nhất của cả hai thế giới: Modal xử lý interactivity (trạng thái mở/đóng), còn UserProfile fetch dữ liệu người dùng thực trực tiếp từ database. Không cái nào ảnh hưởng đến cái kia."

---

**Q6 (Senior): "What can't you do with Server Components? Give me examples of pitfalls."**

**Model Answer:**
"Several important limitations:

**1. No browser APIs or hooks:**
```tsx
// ❌ This throws at runtime:
async function ServerComponent() {
  const width = window.innerWidth; // window doesn't exist on server
  const [count, setCount] = useState(0); // Hooks don't work in RSC
}
```

**2. Event handlers must go to Client Components:**
```tsx
// ❌ Event handlers are client-side by definition:
async function ServerButton() {
  return <button onClick={() => alert('hi')}>Click</button>; // ❌
}
```

**3. Context providers must be Client Components:**
```tsx
// ❌ Context uses useState internally:
// app/ThemeProvider.tsx — must be 'use client'
'use client';
export function ThemeProvider({ children }) {
  // Context.Provider requires client-side rendering
}
```

**4. Non-serializable data cannot cross the boundary:**
```tsx
// ❌ Cannot pass a function as prop from Server to Client:
async function Server() {
  const handler = () => console.log('hi'); // Non-serializable
  return <ClientComp onClick={handler} />; // ❌ Error
}
```

**5. Third-party libraries that use browser globals:**
```tsx
// If a library calls window or document at import time, it breaks RSC
// Solution: wrap it in a Client Component or use next/dynamic with ssr: false
```

The practical advice: if you get an error saying 'You're importing a component that needs X — it only works in a Client Component,' just add `'use client'` to that component or its wrapper."

**Trả lời (Tiếng Việt):**
"Có một số hạn chế quan trọng:

**1. Không có browser API hay hooks:** `window.innerWidth` không tồn tại trên server, và hooks như `useState` không hoạt động trong RSC.

**2. Event handler phải đặt trong Client Components:** các handler như `onClick` về bản chất là client-side, không thể đặt trong Server Component.

**3. Context provider phải là Client Components:** Context dùng `useState` bên trong, nên phải có `'use client'`.

**4. Dữ liệu non-serializable không thể vượt qua ranh giới:** không thể truyền function làm prop từ Server sang Client Component.

**5. Thư viện bên thứ ba dùng browser global:** nếu một thư viện gọi `window` hay `document` lúc import, nó sẽ break RSC. Giải pháp: bọc nó trong một Client Component hoặc dùng `next/dynamic` với `ssr: false`.

Lời khuyên thực tế: nếu bạn gặp lỗi 'You're importing a component that needs X — it only works in a Client Component,' chỉ cần thêm `'use client'` vào component đó hoặc wrapper của nó."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Senior): "Can you walk me through the 4 caching layers in Next.js App Router?"**

**Model Answer:**
"Next.js App Router has 4 caching layers, each operating at a different level. Think of them as a series of filters a request passes through, from fastest to slowest:

**Layer 1 — Request Memoization** (in-memory, per request):
This is React-level caching. If you call `fetch('https://api/user/1')` in 5 different components during a single server render, React automatically deduplicates them — only 1 actual HTTP request is made. It resets after every request, so it's purely about deduplication within a render cycle.

**Layer 2 — Data Cache** (persistent, cross-request):
This is Next.js's persistent cache for `fetch` responses, stored on the server filesystem. It persists between requests and even between deployments unless invalidated. You control it with `cache: 'no-store'` (skip cache), `next: { revalidate: 3600 }` (ISR), or `next: { tags: ['posts'] }` (tag-based invalidation).

**Layer 3 — Full Route Cache** (server filesystem, cross-request):
Next.js caches the fully rendered HTML + RSC payload of static routes. This happens at build time or after first request. It's cleared when you deploy, or when `revalidatePath()`/`revalidateTag()` is called.

**Layer 4 — Router Cache** (browser memory, per session):
Client-side cache of RSC payloads. When you navigate to `/about`, Next.js caches it in the browser. If you navigate away and come back, it uses the cache. Duration: static routes 5 minutes, dynamic routes 30 seconds. Invalidated by `router.refresh()`.

The request flow is: Browser Router Cache → Full Route Cache → Data Cache → Request Memoization → actual data source."

**Trả lời (Tiếng Việt):**
"Next.js App Router có 4 tầng cache, mỗi tầng hoạt động ở một cấp độ khác nhau. Hãy nghĩ chúng như một loạt bộ lọc mà request đi qua, từ nhanh nhất đến chậm nhất:

**Tầng 1 — Request Memoization** (in-memory, per request): Đây là cache ở cấp độ React. Nếu bạn gọi `fetch('https://api/user/1')` trong 5 component khác nhau trong một lần server render, React tự động deduplicate chúng — chỉ có 1 HTTP request thực sự được thực hiện. Nó reset sau mỗi request, nên chỉ để deduplication trong một render cycle.

**Tầng 2 — Data Cache** (persistent, cross-request): Đây là persistent cache của Next.js cho response từ `fetch`, lưu trên filesystem của server. Nó tồn tại giữa các request và thậm chí giữa các deployment trừ khi bị invalidate. Bạn kiểm soát nó với `cache: 'no-store'`, `next: { revalidate: 3600 }`, hoặc `next: { tags: ['posts'] }`.

**Tầng 3 — Full Route Cache** (server filesystem, cross-request): Next.js cache toàn bộ HTML đã render + RSC payload của static routes. Xảy ra lúc build time hoặc sau request đầu tiên. Bị xóa khi bạn deploy, hoặc khi `revalidatePath()`/`revalidateTag()` được gọi.

**Tầng 4 — Router Cache** (browser memory, per session): Cache phía client của RSC payload. Khi bạn điều hướng đến `/about`, Next.js cache nó trong browser. Nếu bạn thoát ra rồi quay lại, nó dùng cache. Thời gian sống: 5 phút cho static routes, 30 giây cho dynamic routes. Bị xóa bởi `router.refresh()`.

Luồng request: Browser Router Cache → Full Route Cache → Data Cache → Request Memoization → nguồn dữ liệu thực."

---

**Q2 (Senior): "After a user submits a form and updates some data, how do you make sure they see the fresh data?"**

**Model Answer:**
"This is the classic cache invalidation problem in Next.js. There are 3 main approaches depending on scope:

**Option 1 — `router.refresh()`** (Client Component after mutation):
```tsx
'use client';
import { useRouter } from 'next/navigation';

async function handleSubmit() {
  await fetch('/api/update', { method: 'POST', body: formData });
  router.refresh(); // Invalidates Router Cache, refetches current page from server
}
```
This clears the Router Cache for the current route and re-fetches from the server. Fast and targeted.

**Option 2 — `revalidatePath()` / `revalidateTag()`** (Server Action):
```tsx
// app/actions.ts
'use server';
import { revalidatePath, revalidateTag } from 'next/cache';

export async function updateProduct(id: string, data: FormData) {
  await db.products.update(id, data);
  revalidatePath(`/products/${id}`); // Invalidate Data Cache + Full Route Cache for this path
  // OR: revalidateTag('products'); // Invalidate all routes tagged 'products'
}
```
Best practice for Server Actions — invalidates both the Data Cache and Full Route Cache.

**Option 3 — `cache: 'no-store'`** on sensitive fetch:
```tsx
// For data that must always be fresh (shopping cart, user session)
const cart = await fetch('/api/cart', { cache: 'no-store' });
```
This bypasses Data Cache entirely for that fetch.

My recommendation: use Server Actions with `revalidateTag()` for data mutations. Tag your fetches by resource type (e.g., `tags: ['products']`) and when a product is updated, invalidate the tag. This is the most precise and scalable approach."

**Trả lời (Tiếng Việt):**
"Đây là bài toán cache invalidation cổ điển trong Next.js. Có 3 cách tiếp cận chính tùy theo phạm vi:

**Option 1 — `router.refresh()`** (Client Component sau mutation): Gọi sau khi fetch update thành công. Cách này xóa Router Cache cho route hiện tại và re-fetch từ server. Nhanh và có phạm vi cụ thể.

**Option 2 — `revalidatePath()` / `revalidateTag()`** (Server Action): Trong Server Action, sau khi update database, gọi `revalidatePath` cho path cụ thể hoặc `revalidateTag` để invalidate tất cả route được tag. Đây là best practice cho Server Actions — invalidate cả Data Cache lẫn Full Route Cache.

**Option 3 — `cache: 'no-store'`** trên fetch nhạy cảm: Dùng cho dữ liệu luôn cần fresh như giỏ hàng hay session người dùng. Cách này bỏ qua Data Cache hoàn toàn cho fetch đó.

Khuyến nghị của tôi: dùng Server Actions với `revalidateTag()` cho data mutation. Tag các fetch của bạn theo loại resource (ví dụ `tags: ['products']`) và khi một product được update, invalidate tag. Đây là cách tiếp cận chính xác và scalable nhất."

---

**Q3 (Mid): "What's the difference between Data Cache and Router Cache? I always confuse these two."**

**Model Answer:**
"They're easy to confuse because both cache page data, but they live in completely different places:

**Data Cache** lives on the **server**. It's Next.js caching the raw response from `fetch()` calls. If 100 users visit the same product page, Next.js runs the data fetch once (or revalidates periodically), stores the result on disk, and serves all 100 users from that cached data. Users never know — they just get fast responses. This is what powers ISR.

**Router Cache** lives in the **browser**. It's the RSC payload (the rendered output) cached in the user's browser tab. If a user visits `/products`, then goes to `/about`, then clicks back — the browser serves `/products` from Router Cache without a server round trip. It's a navigation speed optimization.

Analogy:
- Data Cache = the restaurant's prep kitchen storing pre-made ingredients (server-side)
- Router Cache = the user's local clipboard remembering what they last looked at (browser-side)

The practical impact: if you update a product in the database and call `revalidateTag('products')`, that clears the Data Cache. But the user might still see stale data if their Router Cache hasn't expired yet. That's why sometimes you need both `revalidateTag()` on the server AND `router.refresh()` on the client."

**Trả lời (Tiếng Việt):**
"Dễ nhầm lẫn vì cả hai đều cache dữ liệu trang, nhưng chúng sống ở những nơi hoàn toàn khác nhau:

**Data Cache** nằm trên **server**. Đây là Next.js cache response từ `fetch()`. Nếu 100 người dùng truy cập cùng trang product, Next.js chỉ chạy data fetch một lần (hoặc revalidate định kỳ), lưu kết quả trên disk, và phục vụ tất cả 100 người dùng từ dữ liệu cache đó. Người dùng không biết — họ chỉ thấy response nhanh. Đây là thứ tạo nên ISR.

**Router Cache** nằm trong **browser**. Đây là RSC payload (kết quả render) được cache trong tab browser của người dùng. Nếu người dùng truy cập `/products`, rồi đi đến `/about`, rồi click back — browser phục vụ `/products` từ Router Cache mà không cần round trip đến server. Đây là tối ưu hóa tốc độ điều hướng.

Analogy:
- Data Cache = bếp chuẩn bị trước của nhà hàng lưu trữ nguyên liệu đã chế biến sẵn (phía server)
- Router Cache = clipboard cục bộ của người dùng ghi nhớ những gì họ vừa xem (phía browser)

Tác động thực tế: nếu bạn update một product trong database và gọi `revalidateTag('products')`, điều đó xóa Data Cache. Nhưng người dùng vẫn có thể thấy dữ liệu cũ nếu Router Cache của họ chưa hết hạn. Đó là lý do đôi khi bạn cần cả `revalidateTag()` trên server VÀ `router.refresh()` trên client."

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

---

## 16. Server Actions

**[Mid → Senior]**

**EN:** What are Server Actions? How do they work? How do you use them for form submissions and data mutations?

**VI:** Server Actions là gì? Cách hoạt động như thế nào? Dùng chúng cho form submission và data mutation như thế nào?

### Trả lời

**Server Actions** là các async function chạy trên server, có thể được gọi trực tiếp từ Client Components — **không cần tạo API route riêng**. Đây là tính năng cốt lõi của Next.js 13.4+ / 14, giúp đơn giản hóa đáng kể luồng data mutation.

**Cách tạo Server Action:**

```tsx
// Cách 1: Khai báo trực tiếp trong Server Component
// app/todos/page.tsx
async function createTodo(formData: FormData) {
  'use server'; // directive ở function level
  const title = formData.get('title') as string;
  await db.todo.create({ data: { title } });
  revalidatePath('/todos');
}

export default function TodoPage() {
  return (
    <form action={createTodo}>
      <input name="title" />
      <button type="submit">Add</button>
    </form>
  );
}

// Cách 2: File riêng (dùng 'use server' ở file level)
// app/actions/todo.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';
import { db } from '@/lib/db';

export async function createTodo(formData: FormData) {
  const title = formData.get('title') as string;
  if (!title) throw new Error('Title is required');
  await db.todo.create({ data: { title } });
  revalidatePath('/todos');
}

export async function deleteTodo(id: string) {
  await db.todo.delete({ where: { id } });
  revalidateTag('todos');
}
```

**Form với `useActionState` (React 19) — hiển thị pending state và error:**

```tsx
// app/todos/create-form.tsx
'use client';
import { useActionState } from 'react';
import { createTodo } from '@/app/actions/todo';

type State = { error?: string; success?: boolean };

export function CreateTodoForm() {
  const [state, formAction, isPending] = useActionState<State, FormData>(
    async (prevState, formData) => {
      try {
        await createTodo(formData);
        return { success: true };
      } catch (e) {
        return { error: (e as Error).message };
      }
    },
    {}
  );

  return (
    <form action={formAction}>
      <input name="title" disabled={isPending} placeholder="Todo title..." />
      {state.error && <p className="text-red-500">{state.error}</p>}
      {state.success && <p className="text-green-500">Created!</p>}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Add Todo'}
      </button>
    </form>
  );
}
```

**Server Action với authentication check:**

```tsx
// app/actions/post.ts
'use server';
import { auth } from '@/lib/auth';
import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  const session = await auth(); // đọc session từ cookies
  if (!session?.user) {
    redirect('/login'); // hoặc throw new Error('Unauthorized')
  }

  const title = formData.get('title') as string;
  await db.post.create({
    data: { title, authorId: session.user.id },
  });
  revalidatePath('/posts');
}
```

**Gọi Server Action từ event handler (không phải form):**

```tsx
'use client';
import { deletePost } from '@/app/actions/post';
import { useTransition } from 'react';

export function DeleteButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      onClick={() => startTransition(() => deletePost(postId))}
      disabled={isPending}
    >
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

**Optimistic UI với `useOptimistic`:**

```tsx
'use client';
import { useOptimistic, useTransition } from 'react';
import { toggleLike } from '@/app/actions/post';

export function LikeButton({ postId, initialLikes }: { postId: string; initialLikes: number }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    initialLikes,
    (state, increment: number) => state + increment
  );
  const [, startTransition] = useTransition();

  return (
    <button
      onClick={() => {
        startTransition(async () => {
          addOptimisticLike(1); // cập nhật UI ngay lập tức
          await toggleLike(postId); // thực hiện mutation thực sự
        });
      }}
    >
      ❤️ {optimisticLikes}
    </button>
  );
}
```

**Điểm quan trọng về bảo mật:**
- Server Actions tự động tạo một POST endpoint với **one-time token** — **CSRF protected by default**
- Nhưng vẫn cần: validate input (Zod), kiểm tra authentication, kiểm tra authorization (user có quyền sửa resource này không)
- Không expose sensitive data trong return value vì Client Component có thể nhận

**Progressive Enhancement:** `<form action={serverAction}>` hoạt động ngay cả khi **JavaScript bị tắt** trên trình duyệt.

### 🎤 Mock Interview — Q&A

**Q1 (Mid): "What is a Server Action and why was it introduced?"**

> Server Action là một async function có directive `'use server'`, chạy trên server nhưng có thể được gọi trực tiếp từ Client Component. Nó được giới thiệu để giải quyết vấn đề boilerplate khi xử lý mutations: trước đây phải tạo API route, fetch từ client, xử lý loading/error state thủ công. Server Actions đơn giản hóa flow này, hỗ trợ progressive enhancement (hoạt động không cần JS), và có CSRF protection tích hợp.

**Q2 (Mid): "How do Server Actions compare to API Routes for data mutations?"**

> Server Actions: không cần tạo URL riêng, tích hợp trực tiếp với form/button, CSRF protected tự động, tự động revalidate cache, progressive enhancement. API Routes: vẫn cần cho external clients (mobile app, third-party), public API, hoặc khi cần HTTP methods khác (GET). Với mutations nội bộ trong app Next.js, Server Actions là lựa chọn tốt hơn.

**Q3 (Mid): "How do you show loading state while a Server Action is running?"**

> Có 3 cách: (1) `useActionState` hook — trả về `isPending` boolean; (2) `useTransition` — wrap Server Action call trong `startTransition`, dùng `isPending`; (3) Với form thuần, dùng `useFormStatus` hook từ `react-dom` trong child component để đọc pending state của form parent.

**Q4 (Senior): "What are the security considerations for Server Actions?"**

> Server Actions an toàn hơn API routes vì: CSRF protection tích hợp (one-time token), không expose endpoint URL. Nhưng vẫn cần: (1) **Authentication check** — luôn verify session trong mỗi Server Action; (2) **Authorization** — kiểm tra user có quyền với resource cụ thể (không chỉ là logged in); (3) **Input validation** — dùng Zod hoặc tương tự để validate tất cả input; (4) **Rate limiting** — với sensitive actions; (5) Không return sensitive data vì client có thể đọc.

**Q5 (Senior): "How do you implement optimistic updates with Server Actions?"**

> Dùng `useOptimistic` hook từ React 19: (1) Khai báo `const [optimisticState, addOptimistic] = useOptimistic(initialState, updateFn)`; (2) Trong event handler, gọi `startTransition(() => { addOptimistic(newValue); await serverAction(); })`; (3) React sẽ hiển thị `optimisticState` ngay lập tức, khi Server Action hoàn thành sẽ revert về state thực. Nếu action thất bại, optimistic update tự động rollback.

---

## 17. Route Groups, Parallel Routes, và Intercepting Routes

**[Mid → Senior]**

**EN:** Explain Route Groups, Parallel Routes, and Intercepting Routes in the App Router.

**VI:** Giải thích Route Groups, Parallel Routes, và Intercepting Routes trong App Router.

### Trả lời

Ba tính năng nâng cao của App Router giúp tổ chức code và tạo các UI pattern phức tạp.

---

### Route Groups `(groupName)`

**What:** Folder được đặt trong dấu ngoặc đơn — tạo ra một nhóm logic **KHÔNG ảnh hưởng đến URL**.

**Why:** Áp dụng layout khác nhau cho các phần khác nhau của app, hoặc chỉ đơn giản là tổ chức code.

```
app/
├── (marketing)/
│   ├── layout.tsx        ← Marketing layout (header đơn giản)
│   ├── page.tsx          → URL: /
│   ├── about/page.tsx    → URL: /about
│   └── blog/page.tsx     → URL: /blog
│
├── (auth)/
│   ├── layout.tsx        ← Auth layout (centered card, no navbar)
│   ├── login/page.tsx    → URL: /login
│   └── register/page.tsx → URL: /register
│
└── (dashboard)/
    ├── layout.tsx        ← Dashboard layout (sidebar navigation)
    ├── dashboard/page.tsx → URL: /dashboard
    └── settings/page.tsx  → URL: /settings
```

```tsx
// app/(auth)/layout.tsx — Auth layout không có navbar
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        {children}
      </div>
    </div>
  );
}

// app/(dashboard)/layout.tsx — Dashboard với sidebar
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <main className="flex-1 overflow-auto p-6">{children}</main>
    </div>
  );
}
```

---

### Parallel Routes `@slot`

**What:** Render nhiều page trong CÙNG một layout đồng thời, mỗi slot có loading/error state độc lập.

**How:** Tạo folder với prefix `@`, nhận vào layout dưới dạng props.

```
app/dashboard/
├── layout.tsx
├── page.tsx
├── @analytics/
│   ├── page.tsx          ← Rendered cùng lúc với @team
│   └── loading.tsx       ← Loading riêng cho slot này
└── @team/
    ├── page.tsx
    └── error.tsx         ← Error boundary riêng
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode; // slot @analytics
  team: React.ReactNode;      // slot @team
}) {
  return (
    <div className="grid grid-cols-12 gap-4">
      <div className="col-span-12">{children}</div>
      <div className="col-span-8">{analytics}</div>
      <div className="col-span-4">{team}</div>
    </div>
  );
}
```

**`default.tsx`:** Fallback khi slot không có page khớp với route hiện tại (client-side navigation).

```tsx
// app/dashboard/@analytics/default.tsx
export default function AnalyticsDefault() {
  return <div>Select a dashboard item to see analytics.</div>;
}
```

---

### Intercepting Routes `(.)` notation

**What:** Intercept một route để render trong modal, trong khi URL thay đổi. Khi hard refresh, hiển thị full page.

**Notation:**
- `(.)path` — intercept route cùng level
- `(..)path` — intercept route một level trên
- `(...)path` — intercept từ root

**Classic use case:** Photo modal kiểu Instagram — click ảnh mở modal với URL của ảnh, nhưng vào trực tiếp URL đó hiển thị trang full.

```
app/
├── photos/
│   ├── page.tsx                    → /photos (gallery)
│   └── [id]/page.tsx               → /photos/123 (full page)
│
└── @modal/
    ├── default.tsx                 ← null (không có modal mặc định)
    └── (.)photos/
        └── [id]/page.tsx           → intercepts /photos/123 → modal
```

```tsx
// app/layout.tsx — Root layout nhận @modal slot
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html>
      <body>
        {children}
        {modal} {/* Modal overlay — chỉ render khi route bị intercept */}
      </body>
    </html>
  );
}

// app/@modal/(.)photos/[id]/page.tsx — Modal version
import { Modal } from '@/components/Modal';

export default async function PhotoModal({ params }: { params: { id: string } }) {
  const photo = await getPhoto(params.id);
  return (
    <Modal>
      <img src={photo.url} alt={photo.title} className="max-w-2xl" />
      <h2>{photo.title}</h2>
    </Modal>
  );
}

// app/photos/[id]/page.tsx — Full page version (khi hard refresh)
export default async function PhotoPage({ params }: { params: { id: string } }) {
  const photo = await getPhoto(params.id);
  return (
    <div>
      <img src={photo.url} alt={photo.title} />
      <h1>{photo.title}</h1>
    </div>
  );
}
```

**Cách hoạt động:**
- **Client-side navigation** (click từ gallery): Next.js nhận ra route bị intercept → render `@modal` slot
- **Hard refresh / direct URL**: Không có interception → render full page tại `photos/[id]/page.tsx`

### 🎤 Mock Interview — Q&A

**Q1 (Mid): "What are Route Groups and when do you use them?"**

> Route Groups là folder được đặt trong dấu ngoặc đơn `(groupName)` — không ảnh hưởng đến URL nhưng cho phép áp dụng layout khác nhau cho từng nhóm. Dùng khi: (1) Muốn layout riêng cho auth pages vs dashboard pages vs marketing pages; (2) Muốn tổ chức code theo feature mà không thay đổi URL structure; (3) Cần nhiều root layouts trong cùng app.

**Q2 (Mid): "Explain Parallel Routes. What problem do they solve?"**

> Parallel Routes cho phép render nhiều page đồng thời trong cùng một layout bằng cách tạo `@slot` folders. Vấn đề giải quyết: (1) Dashboard với nhiều panels độc lập — mỗi panel có loading/error state riêng; (2) Modal patterns — `@modal` slot render overlay mà không ảnh hưởng `children`; (3) Conditional rendering theo user role — show/hide slots dựa vào auth. So với chia component thông thường, mỗi slot có thể có `loading.tsx` và `error.tsx` riêng.

**Q3 (Senior): "How do Intercepting Routes work? Give a real-world example."**

> Intercepting Routes dùng `(.)` notation để "bắt" một route và render nó khác với khi vào trực tiếp. Ví dụ: photo gallery. Folder `@modal/(.)photos/[id]` intercept route `/photos/[id]`. Khi user click ảnh trong gallery (client nav), Next.js route `/photos/123` bị intercept → render modal với ảnh đó, URL thay đổi thành `/photos/123`. Khi user copy URL đó và mở tab mới (hard refresh), không có interception → render full photo page. Điều này cho UX tốt hơn (modal nhanh) mà vẫn shareable URL.

**Q4 (Senior): "How do you implement the Instagram-style modal that changes the URL but shows a modal?"**

> Cần kết hợp Parallel Routes + Intercepting Routes: (1) Tạo `@modal` slot trong root layout; (2) Tạo `@modal/(.)photos/[id]/page.tsx` — render Modal component; (3) Tạo `app/@modal/default.tsx` trả về `null` để không có modal khi không intercept; (4) Root layout nhận `modal` prop và render `{modal}` bên cạnh `{children}`; (5) Modal component dùng `useRouter().back()` để dismiss. Điểm mấu chốt: cùng URL `/photos/123` nhưng render khác nhau tùy cách navigate đến.

---

## 18. generateMetadata — Dynamic SEO và Open Graph

**[Junior → Mid]**

**EN:** How do you handle SEO metadata in Next.js App Router? Explain static and dynamic metadata, and how to generate Open Graph images.

**VI:** Xử lý SEO metadata trong Next.js App Router như thế nào? Giải thích static và dynamic metadata, và cách tạo Open Graph images.

### Trả lời

App Router cung cấp Metadata API mạnh mẽ để quản lý SEO tập trung và type-safe.

**1. Static Metadata — export const:**

```tsx
// app/layout.tsx — Root layout metadata (áp dụng cho toàn app)
import type { Metadata } from 'next';

export const metadata: Metadata = {
  metadataBase: new URL('https://myapp.com'), // BẮT BUỘC cho absolute URLs trong OG images
  title: {
    template: '%s | MyApp',   // trang con: "Product Name | MyApp"
    default: 'MyApp',         // khi trang con không set title
  },
  description: 'MyApp is the best platform for...',
  keywords: ['nextjs', 'react', 'web'],
  authors: [{ name: 'Nam Vinh' }],
  robots: {
    index: true,
    follow: true,
    googleBot: { index: true, follow: true },
  },
  openGraph: {
    type: 'website',
    siteName: 'MyApp',
    title: 'MyApp — Build Better',
    description: 'MyApp is the best...',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'MyApp' }],
  },
  twitter: {
    card: 'summary_large_image',
    site: '@myapp',
    creator: '@namvinh',
  },
};
```

**2. Dynamic Metadata — generateMetadata (async):**

```tsx
// app/products/[id]/page.tsx
import type { Metadata } from 'next';

type Props = {
  params: { id: string };
  searchParams: { [key: string]: string | string[] | undefined };
};

// generateMetadata được gọi trước khi render page
// Next.js deduplicates fetch — cùng fetch trong page() sẽ không gọi lại
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const product = await getProduct(params.id); // fetch data

  if (!product) {
    return { title: 'Product Not Found' };
  }

  return {
    title: product.name, // sẽ render thành "Product Name | MyApp" nhờ template
    description: product.description,
    openGraph: {
      title: product.name,
      description: product.description,
      images: [
        {
          url: product.imageUrl,
          width: 1200,
          height: 630,
          alt: product.name,
        },
      ],
    },
  };
}

export default async function ProductPage({ params }: Props) {
  const product = await getProduct(params.id);
  // ... render
}
```

**3. Dynamic OG Image với `next/og`:**

```tsx
// app/og/route.tsx (hoặc app/products/[id]/opengraph-image.tsx)
import { ImageResponse } from 'next/og';

export const runtime = 'edge'; // chạy trên Edge Runtime — nhanh hơn

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get('title') ?? 'Default Title';
  const description = searchParams.get('description') ?? '';

  return new ImageResponse(
    (
      <div
        style={{
          display: 'flex',
          flexDirection: 'column',
          width: '100%',
          height: '100%',
          backgroundColor: '#0f172a',
          padding: '60px',
          justifyContent: 'center',
        }}
      >
        <h1 style={{ color: 'white', fontSize: '60px', fontWeight: 'bold' }}>
          {title}
        </h1>
        <p style={{ color: '#94a3b8', fontSize: '30px' }}>{description}</p>
      </div>
    ),
    { width: 1200, height: 630 }
  );
}

// Trong generateMetadata — trỏ đến dynamic OG route
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const product = await getProduct(params.id);
  const ogImageUrl = `/og?title=${encodeURIComponent(product.name)}&description=${encodeURIComponent(product.description)}`;

  return {
    openGraph: {
      images: [{ url: ogImageUrl, width: 1200, height: 630 }],
    },
  };
}
```

**4. `sitemap.ts` và `robots.ts`:**

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const products = await getProducts();

  return [
    { url: 'https://myapp.com', lastModified: new Date(), changeFrequency: 'yearly', priority: 1 },
    { url: 'https://myapp.com/about', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.8 },
    ...products.map((p) => ({
      url: `https://myapp.com/products/${p.id}`,
      lastModified: p.updatedAt,
      changeFrequency: 'weekly' as const,
      priority: 0.6,
    })),
  ];
}

// app/robots.ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: '/private/' },
    sitemap: 'https://myapp.com/sitemap.xml',
  };
}
```

### 🎤 Mock Interview — Q&A

**Q1 (Junior): "How do you add SEO metadata to a Next.js App Router page?"**

> Có 2 cách: (1) **Static**: export `const metadata: Metadata = { title, description, openGraph, ... }` từ page.tsx hoặc layout.tsx; (2) **Dynamic**: export async function `generateMetadata({ params })` để fetch data và trả về metadata object. Metadata được kế thừa từ layout cha — trang con override trường cụ thể. Root layout thường set `metadataBase` và `title.template`.

**Q2 (Mid): "How do you dynamically generate metadata for a product page based on the product data?"**

> Dùng `generateMetadata` async function — nó chạy trên server trước khi render page, có thể fetch data. Quan trọng: Next.js **deduplicates** fetch requests — nếu cùng URL được fetch trong `generateMetadata` và trong `page()`, chỉ gọi một lần duy nhất (nhờ Request Memoization). Luôn handle trường hợp product không tồn tại và trả về fallback metadata.

**Q3 (Mid): "What is metadataBase and why is it needed?"**

> `metadataBase` là base URL của app, required khi dùng relative paths trong OG images. Không có `metadataBase`, Next.js không biết convert `/og-image.png` thành `https://myapp.com/og-image.png`. Set ở root layout. Nên dùng biến môi trường: `new URL(process.env.NEXT_PUBLIC_BASE_URL ?? 'http://localhost:3000')`.

**Q4 (Senior): "How do you generate dynamic Open Graph images in Next.js?"**

> Dùng `ImageResponse` từ `next/og` chạy trên Edge Runtime. Có 2 cách đặt file: (1) `app/og/route.tsx` — API route nhận query params; (2) `app/products/[id]/opengraph-image.tsx` — convention file, Next.js tự động serve. `ImageResponse` nhận JSX và config (width/height), render thành PNG dùng Satori (SVG engine). Hỗ trợ custom fonts với `options.fonts`. Cần `metadataBase` set đúng để OG crawler có thể fetch absolute URL.

---

## 19. Authentication — Middleware, NextAuth.js, và Session Management

**[Mid → Senior]**

**EN:** How do you implement authentication in Next.js? Explain the middleware-based protection pattern and Auth.js (NextAuth).

**VI:** Triển khai authentication trong Next.js như thế nào? Giải thích middleware-based protection pattern và Auth.js (NextAuth).

### Trả lời

Authentication trong Next.js có **2 layers bảo vệ** — mỗi layer phục vụ mục đích khác nhau.

**Layer 1 — Middleware (Route-level protection): redirect user chưa đăng nhập**

```tsx
// middleware.ts (root level, KHÔNG phải trong app/)
import { auth } from '@/lib/auth'; // Auth.js v5

export default auth((req) => {
  const isLoggedIn = !!req.auth;
  const isAuthRoute = req.nextUrl.pathname.startsWith('/login');
  const isProtectedRoute = req.nextUrl.pathname.startsWith('/dashboard')
    || req.nextUrl.pathname.startsWith('/settings');

  if (isProtectedRoute && !isLoggedIn) {
    const loginUrl = new URL('/login', req.nextUrl.origin);
    loginUrl.searchParams.set('callbackUrl', req.nextUrl.pathname);
    return Response.redirect(loginUrl);
  }

  if (isAuthRoute && isLoggedIn) {
    return Response.redirect(new URL('/dashboard', req.nextUrl.origin));
  }
});

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

**Layer 2 — Server Component / Server Action (Data-level protection): verify quyền với resource cụ thể**

```tsx
// app/dashboard/posts/[id]/edit/page.tsx
import { auth } from '@/lib/auth';
import { redirect, notFound } from 'next/navigation';

export default async function EditPostPage({ params }: { params: { id: string } }) {
  const session = await auth();
  // Middleware đảm bảo user đã login, nhưng cần check ownership
  if (!session) redirect('/login');

  const post = await db.post.findUnique({ where: { id: params.id } });
  if (!post) notFound();

  // Check authorization (không chỉ authentication)
  if (post.authorId !== session.user.id && session.user.role !== 'admin') {
    redirect('/dashboard?error=unauthorized');
  }

  return <EditPostForm post={post} />;
}
```

**Auth.js (NextAuth v5) Setup:**

```tsx
// lib/auth.ts — Configuration
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';
import Google from 'next-auth/providers/google';
import Credentials from 'next-auth/providers/credentials';
import { PrismaAdapter } from '@auth/prisma-adapter';
import { db } from '@/lib/db';
import bcrypt from 'bcryptjs';

export const { auth, handlers, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(db), // lưu session vào DB
  session: { strategy: 'jwt' }, // hoặc 'database'
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
    Google({
      clientId: process.env.GOOGLE_ID!,
      clientSecret: process.env.GOOGLE_SECRET!,
    }),
    Credentials({
      async authorize(credentials) {
        const user = await db.user.findUnique({
          where: { email: credentials.email as string },
        });
        if (!user || !user.password) return null;
        const isValid = await bcrypt.compare(credentials.password as string, user.password);
        return isValid ? user : null;
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) token.role = (user as any).role; // thêm role vào JWT
      return token;
    },
    async session({ session, token }) {
      if (token) session.user.role = token.role as string; // expose role tới client
      return session;
    },
  },
  pages: {
    signIn: '/login',
    error: '/auth/error',
  },
});

// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/lib/auth';
export const { GET, POST } = handlers;
```

**Đọc session trong các context khác nhau:**

```tsx
// 1. Server Component
import { auth } from '@/lib/auth';

export default async function ProfilePage() {
  const session = await auth();
  if (!session) return <div>Not logged in</div>;
  return <div>Hello, {session.user?.name}</div>;
}

// 2. Client Component
'use client';
import { useSession } from 'next-auth/react';

export function UserAvatar() {
  const { data: session, status } = useSession();
  if (status === 'loading') return <Skeleton />;
  if (!session) return <LoginButton />;
  return <img src={session.user?.image ?? ''} alt="avatar" />;
}

// 3. Server Action — bảo vệ mutation
'use server';
import { auth } from '@/lib/auth';

export async function deleteAccount() {
  const session = await auth();
  if (!session?.user) throw new Error('Unauthorized');
  await db.user.delete({ where: { id: session.user.id } });
}
```

**JWT vs Database Sessions:**

| | JWT (default) | Database |
|---|---|---|
| Storage | Token trong cookie | Session record trong DB |
| Lookup | Verify signature only | DB query mỗi request |
| Revoke | Khó (phải blacklist) | Dễ (xóa record) |
| Scale | Tốt (stateless) | Cần cẩn thận (N+1) |
| Role update | Cần re-login | Tức thì |

### 🎤 Mock Interview — Q&A

**Q1 (Mid): "How do you protect routes in Next.js so unauthenticated users get redirected?"**

> Dùng **Middleware** (`middleware.ts` ở root). Middleware chạy trước khi request đến page, có thể đọc session từ cookie và redirect. Với Auth.js v5, wrap middleware với `auth()` để tự động parse session. Set `matcher` config để chỉ chạy middleware trên các routes cần thiết, tránh chạy trên static assets. Quan trọng: Middleware bảo vệ route, nhưng vẫn cần check data-level authorization trong Server Component/Action.

**Q2 (Mid): "What is Auth.js (NextAuth) and how do you set it up?"**

> Auth.js (NextAuth v5) là thư viện authentication cho Next.js, hỗ trợ nhiều providers (OAuth, Email, Credentials). Setup gồm: (1) `lib/auth.ts` — config providers, adapter, callbacks, pages; (2) `app/api/auth/[...nextauth]/route.ts` — expose GET/POST handlers; (3) Wrap middleware với `auth()` cho route protection; (4) `SessionProvider` trong root layout cho Client Components. Database adapter (Prisma, Drizzle) lưu users/accounts/sessions.

**Q3 (Senior): "What's the difference between JWT sessions and database sessions in NextAuth?"**

> **JWT**: session được encode trong cookie, mỗi request chỉ cần verify signature — stateless, scale tốt. Nhược điểm: khó revoke (phải dùng blacklist), role changes không có hiệu lực ngay. **Database**: session được lưu trong DB, mỗi request query DB để verify — dễ revoke, role update tức thì. Nhược điểm: thêm DB roundtrip mỗi request, cần cẩn thận với performance. Recommendation: JWT cho hầu hết cases, Database khi cần security cao hoặc revocation ngay lập tức.

**Q4 (Senior): "How do you protect a Server Action so only authenticated users can call it?"**

> Luôn gọi `const session = await auth()` ngay đầu Server Action và throw error hoặc redirect nếu không có session. Không được tin tưởng vào Middleware hoàn toàn vì Server Actions có thể được gọi trực tiếp qua POST request. Pattern: (1) Check authentication (`session` tồn tại); (2) Check authorization (`session.user.id === resource.ownerId`); (3) Validate input với Zod; (4) Thực hiện mutation. Đây là "defense in depth" — nhiều layers bảo vệ.

---

## 20. next/link — Prefetching và Client-side Navigation

**[Junior → Mid]**

**EN:** How does `next/link` work? Explain its prefetching behavior and how it differs from a regular `<a>` tag.

**VI:** `next/link` hoạt động như thế nào? Giải thích prefetching behavior và sự khác biệt với `<a>` tag thông thường.

### Trả lời

`next/link` là component cốt lõi giúp navigation trong Next.js cảm giác tức thì.

**Khác biệt so với `<a>` tag thông thường:**

| | `<a href="...">` | `<Link href="...">` |
|---|---|---|
| Navigation | Full page reload | Client-side (SPA-style) |
| Prefetching | Không | Có (tự động khi vào viewport) |
| JS Bundle | Load lại tất cả | Chỉ load route mới |
| Scroll | Reset về top | Reset về top (có thể tắt) |
| History | Push state | Push state |

**Prefetching behavior — chi tiết quan trọng:**

```tsx
// app/components/Navigation.tsx
import Link from 'next/link';

export function Navigation() {
  return (
    <nav>
      {/* Static route: prefetch FULL content khi vào viewport (production only) */}
      <Link href="/about">About</Link>

      {/* Dynamic route: chỉ prefetch loading.tsx skeleton */}
      <Link href="/products/123">Product</Link>

      {/* Opt-out prefetch — hữu ích cho trang nặng hoặc ít dùng */}
      <Link href="/admin/reports" prefetch={false}>Reports</Link>

      {/* Force prefetch full content ngay cả cho dynamic route */}
      <Link href="/dashboard" prefetch={true}>Dashboard</Link>

      {/* Không scroll lên đầu khi navigate — hữu ích cho modal-like navigation */}
      <Link href="/gallery#section-2" scroll={false}>Gallery Section</Link>

      {/* Replace thay vì push — không tạo history entry mới */}
      <Link href="/step-2" replace>Next Step</Link>
    </nav>
  );
}
```

**Active link styling với `usePathname()`:**

```tsx
// app/components/NavLink.tsx
'use client';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { clsx } from 'clsx';

interface NavLinkProps {
  href: string;
  children: React.ReactNode;
  exact?: boolean;
}

export function NavLink({ href, children, exact = false }: NavLinkProps) {
  const pathname = usePathname();
  const isActive = exact ? pathname === href : pathname.startsWith(href);

  return (
    <Link
      href={href}
      className={clsx(
        'px-3 py-2 rounded-md text-sm font-medium transition-colors',
        isActive
          ? 'bg-blue-600 text-white'
          : 'text-gray-600 hover:bg-gray-100'
      )}
    >
      {children}
    </Link>
  );
}

// Usage
<NavLink href="/dashboard" exact>Dashboard</NavLink>
<NavLink href="/products">Products</NavLink>
```

**Programmatic navigation với `useRouter()`:**

```tsx
// app/components/SearchBar.tsx
'use client';
import { useRouter, useSearchParams } from 'next/navigation';
import { useTransition } from 'react';

export function SearchBar() {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();

  function handleSearch(query: string) {
    startTransition(() => {
      // Programmatic navigation — dùng khi navigation xảy ra từ event handler
      router.push(`/search?q=${encodeURIComponent(query)}`);
    });
  }

  function handleBack() {
    router.back(); // quay lại trang trước
  }

  function handleReplace() {
    router.replace('/home'); // replace history entry
  }

  return (
    <div>
      <input
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      {isPending && <span>Loading...</span>}
    </div>
  );
}
```

**Lỗi phổ biến cần tránh:**

```tsx
// ❌ SAI — Next.js 13+ không cần <a> bên trong <Link>
<Link href="/about">
  <a>About</a>  // <a> lồng vào <Link> → invalid HTML
</Link>

// ✅ ĐÚNG — Link tự render <a> tag
<Link href="/about">About</Link>

// ❌ SAI — dùng <a> cho internal links (mất prefetching, gây full reload)
<a href="/dashboard">Dashboard</a>

// ✅ ĐÚNG
<Link href="/dashboard">Dashboard</Link>

// ✅ <a> là ĐÚNG cho external links
<a href="https://github.com" target="_blank" rel="noopener noreferrer">
  GitHub
</a>
```

**Prefetching chỉ hoạt động trong production** (`next build && next start`). Trong development, không có prefetching để dễ debug.

### 🎤 Mock Interview — Q&A

**Q1 (Junior): "Why does navigation feel instant in Next.js? What is Link's prefetching?"**

> Navigation cảm giác tức thì nhờ 2 cơ chế: (1) **Client-side navigation** — `<Link>` không reload toàn trang, chỉ swap component theo route mới; (2) **Prefetching** — khi một `<Link>` xuất hiện trong viewport, Next.js tự động download JavaScript và data của route đó ở background. Khi user click, tài nguyên đã sẵn sàng → hiển thị ngay. Chỉ hoạt động trong production build.

**Q2 (Mid): "When would you use `router.push()` instead of `<Link>`?"**

> `router.push()` dùng khi navigation xảy ra từ logic code, không phải từ user click link: (1) Sau khi submit form thành công — redirect đến trang mới; (2) Sau khi login/logout; (3) Conditional navigation dựa vào kết quả API call; (4) Navigation từ keyboard shortcut hoặc gesture. Dùng `useTransition` để wrap `router.push()` và có `isPending` state trong quá trình navigation.

**Q3 (Mid): "What's the difference in prefetch behavior between static and dynamic routes?"**

> **Static routes** (không có dynamic segments, hoặc dùng `generateStaticParams`): prefetch toàn bộ content của page — khi click, trang hiển thị ngay lập tức. **Dynamic routes** (`/products/[id]`): chỉ prefetch `loading.tsx` skeleton — khi click, loading skeleton hiển thị ngay trong khi data được fetch. Lý do: không thể biết trước data của dynamic route. Có thể override với `prefetch={true}` để force prefetch full content (cẩn thận với performance) hoặc `prefetch={false}` để disable hoàn toàn.

---

## 21. next.config.js — Cấu Hình Quan Trọng

**[Mid → Senior]**

**EN:** What are the most important configuration options in next.config.js? Explain redirects, rewrites, headers, and Turbopack.

**VI:** Các cấu hình quan trọng nhất trong next.config.js là gì? Giải thích redirects, rewrites, headers, và Turbopack.

### Trả lời

`next.config.js` (hoặc `next.config.ts`) là file cấu hình chính của Next.js app.

**1. Redirects — chuyển hướng URL (SEO-friendly):**

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async redirects() {
    return [
      // 308 Permanent — SEO preserving, search engines cập nhật index
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
      // 307 Temporary — không ảnh hưởng SEO ranking
      {
        source: '/sale',
        destination: '/products?category=sale',
        permanent: false,
      },
      // Redirect toàn bộ www → non-www
      {
        source: '/:path*',
        has: [{ type: 'host', value: 'www.myapp.com' }],
        destination: 'https://myapp.com/:path*',
        permanent: true,
      },
    ];
  },
};
```

**2. Rewrites — proxy requests (URL không thay đổi trên browser):**

```js
async rewrites() {
  return [
    // API proxy — ẩn backend URL khỏi client
    // Browser thấy /api/users, request thực sự đến external-api.com
    {
      source: '/api/:path*',
      destination: 'https://internal-api.company.com/:path*',
    },
    // Proxy đến microservice riêng biệt
    {
      source: '/payments/:path*',
      destination: 'https://payments.myapp.com/:path*',
    },
    // A/B testing — route % traffic đến variant
    {
      source: '/landing',
      destination: '/landing-v2',
      // có thể thêm has: [{ type: 'cookie', key: 'ab-variant', value: 'b' }]
    },
  ];
},
```

**3. Headers — custom security headers:**

```js
async headers() {
  return [
    {
      source: '/(.*)',
      headers: [
        // Chống clickjacking
        { key: 'X-Frame-Options', value: 'DENY' },
        // Chống MIME-type sniffing
        { key: 'X-Content-Type-Options', value: 'nosniff' },
        // Strict HTTPS
        { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' },
        // Content Security Policy
        {
          key: 'Content-Security-Policy',
          value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; img-src 'self' data: https:;",
        },
        // Referrer policy
        { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
        // Permissions policy
        { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
      ],
    },
    // CORS cho API routes
    {
      source: '/api/:path*',
      headers: [
        { key: 'Access-Control-Allow-Origin', value: 'https://trusted-domain.com' },
        { key: 'Access-Control-Allow-Methods', value: 'GET, POST, PUT, DELETE, OPTIONS' },
      ],
    },
  ];
},
```

**4. Remote Images cho `next/image`:**

```js
images: {
  remotePatterns: [
    {
      protocol: 'https',
      hostname: 'res.cloudinary.com',
      port: '',
      pathname: '/myaccount/**',
    },
    {
      protocol: 'https',
      hostname: '*.amazonaws.com', // wildcard subdomain
    },
  ],
},
```

**5. Turbopack — Rust-based bundler thay thế Webpack:**

```bash
# Bật Turbopack cho dev server (Next.js 14+)
next dev --turbopack

# Hoặc trong package.json
"dev": "next dev --turbopack"
```

```js
// next.config.js — experimental Turbopack options (Next.js 15)
const nextConfig = {
  // Turbopack tự động bật với --turbopack flag
  // Cấu hình alias, loader nếu cần
};
```

**Turbopack vs Webpack:**

| | Webpack | Turbopack |
|---|---|---|
| Ngôn ngữ | JavaScript | Rust |
| Dev startup | ~3-5s | ~0.3s (10x faster) |
| HMR | ~500ms | ~50ms |
| Production build | Stable | Webpack (Turbopack production đang phát triển) |
| Config | Rất linh hoạt | Đang tích hợp dần |
| Ecosystem | Khổng lồ | Đang phát triển |

**6. Các cấu hình quan trọng khác:**

```js
const nextConfig = {
  // Docker-optimized build — tạo standalone folder
  output: 'standalone',

  // Transpile node_modules chưa được compile (ESM-only packages)
  transpilePackages: ['@acme/ui', 'some-esm-package'],

  // Expose env vars (chú ý: không dùng cho secrets)
  env: {
    APP_VERSION: process.env.npm_package_version,
  },

  // Deploy tại subpath (ví dụ: myapp.com/blog/...)
  basePath: '/blog',

  // Compress responses
  compress: true,

  // Tắt x-powered-by header (security)
  poweredByHeader: false,

  // Custom webpack config — ví dụ: thêm SVG loader
  webpack(config) {
    config.module.rules.push({
      test: /\.svg$/,
      use: ['@svgr/webpack'],
    });
    return config;
  },

  // Optimize imports từ large packages
  experimental: {
    optimizePackageImports: ['lucide-react', '@heroicons/react'],
  },
};

module.exports = nextConfig;
```

**Sự khác biệt Redirects vs Rewrites:**

| | Redirects | Rewrites |
|---|---|---|
| Browser URL | Thay đổi | **Không thay đổi** |
| HTTP status | 307 / 308 | 200 |
| SEO | Chuyển ranking | Routing trong bóng |
| Use case | URL đổi vĩnh viễn | Proxy, A/B, URL shortener |

### 🎤 Mock Interview — Q&A

**Q1 (Mid): "What is the difference between redirects and rewrites in next.config.js?"**

> **Redirects**: browser URL thay đổi — dùng 307 (temporary) hoặc 308 (permanent). Hữu ích khi URL cũ không còn tồn tại, cần chuyển SEO ranking. **Rewrites**: browser URL **không thay đổi** — Next.js internally route request đến destination khác. Dùng cho: proxy API requests để ẩn backend URL, A/B testing, pretty URLs trỏ đến internal routes. Về mặt kỹ thuật, rewrites xảy ra trên server trước khi response, transparent với client.

**Q2 (Mid): "How do you proxy API calls through Next.js to hide your backend URL?"**

> Dùng `rewrites()` trong `next.config.js`: set `source: '/api/:path*'` và `destination: 'https://internal-api.com/:path*'`. Client chỉ biết `/api/...`, không biết URL thực của backend. Lợi ích: (1) Ẩn backend URL khỏi browser network tab; (2) Tránh CORS issues vì request qua cùng origin; (3) Dễ thay đổi backend URL mà không cần update frontend code; (4) Có thể thêm auth headers ở Next.js layer trước khi forward.

**Q3 (Senior): "What is Turbopack and how is it different from Webpack?"**

> Turbopack là bundler được viết bằng Rust, thay thế Webpack trong Next.js dev workflow. Điểm khác biệt chính: (1) **Tốc độ**: dev startup nhanh hơn ~10x, HMR nhanh hơn ~10x nhờ Rust; (2) **Incremental compilation**: chỉ recompile những gì thực sự thay đổi — tính toán theo function level thay vì file level; (3) **Hiện tại**: Turbopack stable cho dev (`--turbopack`), production build vẫn dùng Webpack (Next.js 14/15). Turbopack không yêu cầu thay đổi code — là drop-in replacement, nhưng một số custom webpack loaders chưa được hỗ trợ.

**Q4 (Senior): "How do you configure security headers in next.config.js?"**

> Dùng `headers()` async function, return array của header objects với `source` (URL pattern) và `headers` array. Headers quan trọng nhất: `X-Frame-Options: DENY` (chống clickjacking), `X-Content-Type-Options: nosniff`, `Strict-Transport-Security` (HTTPS), `Content-Security-Policy` (XSS mitigation), `Referrer-Policy`. Có thể check score tại securityheaders.com. Lưu ý: CSP cần test kỹ vì quá strict sẽ break inline scripts/styles. Với App Router, có thể kết hợp headers trong `next.config.js` (static) và Middleware (dynamic, dựa vào request).

---

*Tài liệu này được cập nhật cho Next.js 14 / 15 với App Router. Câu hỏi được sắp xếp từ Junior đến Senior theo từng topic.*
