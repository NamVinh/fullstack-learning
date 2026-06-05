# Web Performance — Interview Questions (Junior → Senior)

---

## 1. Core Web Vitals

**Level:** [Junior] / [Mid]

**EN:** Explain the three Core Web Vitals: LCP, INP, and CLS. What are the thresholds for "Good", "Needs Improvement", and "Poor"?

**VI:** Giải thích 3 Core Web Vitals: LCP, INP, và CLS. Ngưỡng "Good", "Needs Improvement", "Poor" là bao nhiêu?

**Trả lời:**

Core Web Vitals là các metrics do Google định nghĩa để đo user experience thực tế (không chỉ technical metrics). Chúng ảnh hưởng đến Google Search ranking.

| Metric | Đo gì | Good | Needs Improvement | Poor |
|--------|--------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | Loading — khi nào nội dung chính xuất hiện | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | Interactivity — độ trễ response UI (thay thế FID) | ≤ 200ms | 200ms – 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | Visual Stability — content có nhảy không | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

```ts
// Đo Core Web Vitals trong code
import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals';

function sendToAnalytics({ name, value, id, delta }: Metric) {
  fetch('/api/vitals', {
    method: 'POST',
    body: JSON.stringify({ name, value, id, delta, url: window.location.href }),
    headers: { 'Content-Type': 'application/json' },
  });
}

// Đăng ký listeners
onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
onFCP(sendToAnalytics);  // First Contentful Paint
onTTFB(sendToAnalytics); // Time to First Byte

// Next.js App Router built-in
// app/layout.tsx
export function reportWebVitals(metric: NextWebVitalsMetric) {
  if (metric.label === 'web-vital') {
    console.log(metric); // { id, name, startTime, value, label }
  }
}
```

---

## 2. LCP Optimization

**Level:** [Mid] / [Senior]

**EN:** What causes poor LCP? How do you optimize it through preloading, image optimization, and eliminating render-blocking resources?

**VI:** Nguyên nhân LCP kém? Cách tối ưu qua preloading, image optimization, và loại bỏ render-blocking resources?

**Trả lời:**

**LCP element thường là:** hero image, `<h1>` text, video poster, large background image.

```html
<!-- ==============================
     1. PRELOAD critical resources
     ============================== -->

<!-- Preload LCP image — đây là tối ưu quan trọng nhất -->
<link rel="preload" as="image" href="/hero.webp"
      imagesrcset="/hero-400.webp 400w, /hero-800.webp 800w, /hero-1200.webp 1200w"
      imagesizes="(max-width: 768px) 100vw, 50vw" />

<!-- Preload critical fonts -->
<link rel="preload" as="font" type="font/woff2"
      href="/fonts/inter-var.woff2" crossorigin />

<!-- Preload critical CSS (nếu không inline) -->
<link rel="preload" as="style" href="/critical.css" />

<!-- ==============================
     2. Image optimization
     ============================== -->

<!-- Dùng modern formats + srcset + sizes + lazy loading cho non-LCP -->
<img
  src="/hero.webp"
  srcset="/hero-400.webp 400w, /hero-800.webp 800w, /hero-1200.webp 1200w"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 600px"
  alt="Hero image"
  width="1200"
  height="630"
  fetchpriority="high"   <!-- hint browser đây là LCP candidate -->
  decoding="async"
  <!-- KHÔNG dùng loading="lazy" cho LCP image -->
/>

<!-- Non-LCP images: lazy loading -->
<img src="/article-image.webp" loading="lazy" alt="..." width="800" height="400" />

<!-- ==============================
     3. Eliminate render-blocking resources
     ============================== -->

<!-- Render-blocking CSS: inline critical CSS, defer non-critical -->
<style>
  /* Critical CSS inlined — fold content */
  body { margin: 0; font-family: system-ui; }
  .hero { min-height: 60vh; }
</style>

<!-- Defer non-critical CSS -->
<link rel="preload" as="style" href="/non-critical.css"
      onload="this.rel='stylesheet'" />
<noscript><link rel="stylesheet" href="/non-critical.css" /></noscript>

<!-- Scripts: async hoặc defer -->
<script src="/analytics.js" async></script>     <!-- non-critical, load ASAP, execute khi load -->
<script src="/app.js" defer></script>            <!-- execute sau khi HTML parsed -->
<script src="/critical.js"></script>             <!-- blocking — tránh nếu có thể -->
```

```ts
// Next.js Image component — tự động optimize
import Image from 'next/image';

// LCP image — priority=true → preload + fetchpriority=high
function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero"
      width={1200}
      height={630}
      priority          // preload this image
      quality={85}      // balance quality vs size
      placeholder="blur" // blur-up effect trong khi load
      blurDataURL="data:image/jpeg;base64,/9j/..."
    />
  );
}

// ==============================
// 4. Server response time (TTFB)
// ==============================

// Caching ở server — giảm TTFB
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, unknown>({ max: 500, ttl: 1000 * 60 * 5 });

async function getCachedData(key: string, fetcher: () => Promise<unknown>) {
  if (cache.has(key)) return cache.get(key);

  const data = await fetcher();
  cache.set(key, data);
  return data;
}

// Next.js ISR (Incremental Static Regeneration) — pre-render + revalidate
// app/products/[id]/page.tsx
export async function generateStaticParams() {
  const products = await getTopProducts(); // pre-render top products
  return products.map((p) => ({ id: p.id }));
}

export const revalidate = 3600; // revalidate every hour
```

---

## 3. INP Optimization

**Level:** [Senior]

**EN:** What causes poor INP (Interaction to Next Paint)? How do you break up long tasks, use Web Workers, and the Scheduler API?

**VI:** Nguyên nhân INP kém? Cách break up long tasks, dùng Web Workers và Scheduler API?

**Trả lời:**

INP đo độ trễ từ khi user interact (click, type, touch) đến khi browser render frame tiếp theo. Nguyên nhân chính: **long tasks blocking main thread**.

```ts
// ==============================
// VẤNĐỀ: Long tasks block main thread
// ==============================

// XẤU — blocking task 500ms → INP sẽ tệ
function processLargeDataset(items: Item[]) {
  return items.map(item => expensiveTransform(item)); // blocking
}

// ==============================
// 1. Break up long tasks với scheduler
// ==============================

// Dùng setTimeout(0) để yield về main thread
async function processInChunks<T>(
  items: T[],
  processor: (item: T) => void,
  chunkSize = 50
): Promise<void> {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    chunk.forEach(processor);

    // Yield control → browser có thể handle user interactions giữa các chunks
    await new Promise((resolve) => setTimeout(resolve, 0));
  }
}

// Scheduler API (Chrome 94+) — ưu tiên user interactions hơn background tasks
async function processWithScheduler(items: Item[]) {
  for (const item of items) {
    await scheduler.postTask(() => processItem(item), {
      priority: 'background', // user-blocking | user-visible | background
    });
  }
}

// isInputPending — tạm dừng nếu user đang interact
async function processRespectingInput(items: Item[]) {
  let i = 0;
  while (i < items.length) {
    // Check nếu có pending user input
    if (navigator.scheduling?.isInputPending()) {
      await new Promise((resolve) => setTimeout(resolve, 0)); // yield
      continue;
    }

    processItem(items[i]);
    i++;

    // Yield mỗi 5ms
    if (performance.now() % 5 < 1) {
      await new Promise((resolve) => setTimeout(resolve, 0));
    }
  }
}

// ==============================
// 2. Web Workers — heavy computation off main thread
// ==============================

// worker.ts
self.onmessage = (event) => {
  const { items } = event.data;

  // Heavy computation trong worker — không block UI
  const result = items.map((item: Item) => heavyTransform(item));

  self.postMessage({ result });
};

// main.ts
const worker = new Worker(new URL('./worker.ts', import.meta.url), { type: 'module' });

async function processWithWorker(items: Item[]): Promise<ProcessedItem[]> {
  return new Promise((resolve, reject) => {
    worker.onmessage = (event) => resolve(event.data.result);
    worker.onerror = (error) => reject(error);
    worker.postMessage({ items });
  });
}

// React hook cho Web Worker
function useWorker<TInput, TOutput>(workerUrl: string) {
  const workerRef = useRef<Worker | null>(null);

  useEffect(() => {
    workerRef.current = new Worker(workerUrl, { type: 'module' });
    return () => workerRef.current?.terminate();
  }, [workerUrl]);

  const process = useCallback((data: TInput): Promise<TOutput> => {
    return new Promise((resolve, reject) => {
      if (!workerRef.current) return reject(new Error('Worker not ready'));

      workerRef.current.onmessage = (e) => resolve(e.data);
      workerRef.current.onerror = reject;
      workerRef.current.postMessage(data);
    });
  }, []);

  return process;
}

// ==============================
// 3. Debounce/Throttle input handlers
// ==============================
import { useCallback } from 'react';
import { useDebouncedCallback } from 'use-debounce';

function SearchInput() {
  // Không search sau mỗi keystroke — debounce 300ms
  const handleSearch = useDebouncedCallback(async (value: string) => {
    const results = await searchAPI(value);
    setResults(results);
  }, 300);

  return (
    <input
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}

// ==============================
// 4. useTransition — mark updates as non-urgent
// ==============================
import { useState, useTransition } from 'react';

function FilterableList({ items }: { items: Item[] }) {
  const [filter, setFilter] = useState('');
  const [isPending, startTransition] = useTransition();

  const filteredItems = items.filter(item =>
    item.name.toLowerCase().includes(filter.toLowerCase())
  );

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;

    // Update input ngay lập tức (urgent)
    setFilter(value);

    // Mark filter computation as non-urgent
    // React có thể interrupt để xử lý user interactions khác
    startTransition(() => {
      // expensive re-render
    });
  }

  return (
    <>
      <input onChange={handleChange} value={filter} />
      {isPending ? <Spinner /> : <ItemList items={filteredItems} />}
    </>
  );
}
```

---

## 4. CLS Optimization

**Level:** [Mid]

**EN:** What causes CLS (Cumulative Layout Shift)? How do you fix it with image dimensions, font loading, and avoiding content injection?

**VI:** Nguyên nhân CLS? Cách fix bằng image dimensions, font loading, và tránh content injection?

**Trả lời:**

CLS đo tổng tất cả layout shifts không mong muốn trong suốt vòng đời trang.

```html
<!-- ==============================
     1. Always set image dimensions
     ============================== -->

<!-- XẤU — browser không biết cần reserve bao nhiêu space -->
<img src="/photo.jpg" alt="Photo" />

<!-- TỐT — browser reserve exact space, không có shift -->
<img src="/photo.jpg" alt="Photo" width="800" height="600" />

<!-- Responsive với aspect ratio preserved -->
<style>
  .responsive-img {
    width: 100%;
    height: auto;
    aspect-ratio: 16 / 9; /* browser reserve đúng không gian */
  }
</style>
<img class="responsive-img" src="/video-thumbnail.jpg" alt="..." width="1280" height="720" />
```

```css
/* ==============================
   2. Font loading — FOUT (Flash of Unstyled Text)
   và FOIT (Flash of Invisible Text)
   ============================== */

/* font-display: swap — hiển thị fallback ngay, swap khi custom font load
   Có thể gây CLS khi font thay thế có kích thước khác */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-style: normal;
  font-display: swap;
}

/* Giảm CLS do font: dùng size-adjust để match fallback font */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-display: optional; /* không swap nếu font chưa load — ít CLS nhất */
}

/* Fallback font với gần giống metrics */
@font-face {
  font-family: 'Inter-fallback';
  src: local('Arial');
  size-adjust: 107%;         /* điều chỉnh size để match Inter */
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}

body {
  font-family: 'Inter', 'Inter-fallback', Arial, sans-serif;
}
```

```tsx
// ==============================
// 3. Avoid injecting content above existing content
// ==============================

// XẤU — banner inject làm push content xuống
function App() {
  const [showBanner, setShowBanner] = useState(false);

  useEffect(() => {
    checkIfUserShouldSeeBanner().then(setShowBanner);
  }, []);

  return (
    <div>
      {showBanner && <CookieBanner />}  {/* Xuất hiện và đẩy nội dung xuống → CLS */}
      <Header />
      <MainContent />
    </div>
  );
}

// TỐT — reserve space hoặc overlay
function App() {
  const [bannerReady, setBannerReady] = useState<boolean | null>(null);

  useEffect(() => {
    checkIfUserShouldSeeBanner().then(setBannerReady);
  }, []);

  return (
    <div>
      {/* Reserve space ngay từ đầu */}
      <div style={{ minHeight: bannerReady === null ? '60px' : bannerReady ? '60px' : '0' }}>
        {bannerReady && <CookieBanner />}
      </div>
      <Header />
      <MainContent />
    </div>
  );
}

// Hoặc dùng overlay/sticky bottom — không affect layout
function CookieConsent() {
  return (
    <div style={{
      position: 'fixed',
      bottom: 0,
      left: 0,
      right: 0,
      zIndex: 1000,
      // Overlay, không push content
    }}>
      <CookieBanner />
    </div>
  );
}

// ==============================
// 4. Skeleton screens thay vì loading spinners
// ==============================
function ProductCard({ productId }: { productId: string }) {
  const { data, isLoading } = useProduct(productId);

  if (isLoading) {
    // Skeleton giữ đúng kích thước → không có layout shift khi data load
    return (
      <div className="product-card" aria-hidden="true">
        <div className="skeleton" style={{ width: '100%', aspectRatio: '16/9' }} />
        <div className="skeleton" style={{ height: '1.2em', width: '70%', marginTop: '1rem' }} />
        <div className="skeleton" style={{ height: '1em', width: '40%', marginTop: '0.5rem' }} />
      </div>
    );
  }

  return (
    <div className="product-card">
      <img src={data.image} alt={data.name} width={640} height={360} />
      <h3>{data.name}</h3>
      <p>${data.price}</p>
    </div>
  );
}
```

---

## 5. Code Splitting

**Level:** [Mid]

**EN:** Explain route-level, component-level, and library code splitting. How does dynamic `import()` work?

**VI:** Giải thích route-level, component-level, và library code splitting. `import()` dynamic hoạt động như thế nào?

**Trả lời:**

Code splitting chia bundle thành các chunks nhỏ hơn, load on-demand thay vì load tất cả ngay từ đầu.

```tsx
// ==============================
// 1. Route-level splitting (Most impactful)
// ==============================

// React Router v6 + lazy
import { lazy, Suspense } from 'react';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Mỗi route = separate chunk, load khi navigate
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings  = lazy(() => import('./pages/Settings'));
const Reports   = lazy(() => import('./pages/Reports'));

// Prefetch trước khi user navigate
const AdminPanel = lazy(
  () => import(/* webpackPrefetch: true */ './pages/AdminPanel')
);

const router = createBrowserRouter([
  {
    path: '/dashboard',
    element: (
      <Suspense fallback={<PageSkeleton />}>
        <Dashboard />
      </Suspense>
    ),
  },
  {
    path: '/settings',
    element: (
      <Suspense fallback={<PageSkeleton />}>
        <Settings />
      </Suspense>
    ),
  },
]);

// Next.js App Router — tự động route splitting
// app/dashboard/page.tsx — tự động là separate chunk

// ==============================
// 2. Component-level splitting
// ==============================

// Heavy components — split khi không critical cho initial render
const RichTextEditor = lazy(() => import('./components/RichTextEditor'));
const DataVisualization = lazy(() => import('./components/DataVisualization'));
const PDFViewer = lazy(() => import('./components/PDFViewer'));

function ArticleEditor() {
  const [showEditor, setShowEditor] = useState(false);

  return (
    <div>
      <button onClick={() => setShowEditor(true)}>Edit Article</button>
      {showEditor && (
        <Suspense fallback={<div>Loading editor...</div>}>
          <RichTextEditor />
        </Suspense>
      )}
    </div>
  );
}

// ==============================
// 3. Dynamic import() — programmatic splitting
// ==============================

// Conditional imports
async function loadChartLibrary(type: 'line' | 'bar' | 'pie') {
  if (type === 'line') {
    const { LineChart } = await import('./charts/LineChart');
    return LineChart;
  }
  if (type === 'bar') {
    const { BarChart } = await import('./charts/BarChart');
    return BarChart;
  }
  const { PieChart } = await import('./charts/PieChart');
  return PieChart;
}

// Load heavy utility only when needed
async function exportToPDF(data: ReportData) {
  // jsPDF chỉ load khi user click Export
  const { jsPDF } = await import('jspdf');
  const doc = new jsPDF();
  // ...
}

// ==============================
// 4. Library splitting
// ==============================

// Vite config — manual chunks cho vendor splitting
// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunks — cached riêng, ít thay đổi hơn app code
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-tooltip'],
          'chart-vendor': ['recharts', 'd3'],
          'form-vendor': ['react-hook-form', 'zod', '@hookform/resolvers'],
          'query-vendor': ['@tanstack/react-query'],
        },
        // Hoặc để Rollup tự quyết định
        // manualChunks(id) {
        //   if (id.includes('node_modules')) {
        //     return 'vendor';
        //   }
        // },
      },
    },
    chunkSizeWarningLimit: 500, // warn nếu chunk > 500kb
  },
});
```

---

## 6. Bundle Analysis

**Level:** [Mid]

**EN:** How do you analyze bundle size? What tools do you use to identify large dependencies?

**VI:** Làm thế nào phân tích bundle size? Tools nào dùng để identify large dependencies?

**Trả lời:**

```bash
# webpack-bundle-analyzer
npm install --save-dev webpack-bundle-analyzer

# Tạo stats.json
npx webpack --profile --json > stats.json

# Mở analyzer
npx webpack-bundle-analyzer stats.json
```

```ts
// Vite — rollup-plugin-visualizer
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'dist/stats.html',
      open: true,        // tự động mở browser
      gzipSize: true,    // hiển thị gzip size
      brotliSize: true,  // hiển thị brotli size
      template: 'treemap', // 'treemap' | 'sunburst' | 'network'
    }),
  ],
});

// Chạy: npm run build → mở stats.html tự động
```

```ts
// ==============================
// Analyze trước khi install package
// ==============================

// 1. bundlephobia.com — nhập package name, xem size + dependencies
// moment → 67.9KB gzip (LARGE)
// date-fns → 13.6KB gzip với tree shaking (MUCH BETTER)
// dayjs → 2.6KB gzip (SMALLEST)

// 2. packagephobia.com — install size + publish size

// ==============================
// Common large dependencies và alternatives
// ==============================

// Lodash — 70KB full
import _ from 'lodash'; // XẤU — import toàn bộ

import debounce from 'lodash/debounce'; // TỐT — per-method import
import throttle from 'lodash/throttle';

// Hoặc dùng lodash-es với tree shaking
import { debounce, throttle } from 'lodash-es';

// Moment.js → date-fns hoặc dayjs
import moment from 'moment'; // 67KB — TRÁNH
import { format, addDays } from 'date-fns'; // tree-shakable
import dayjs from 'dayjs'; // 2.6KB

// Chart.js full → chart.js với tree shaking
import Chart from 'chart.js/auto'; // 60KB — tránh
import { Chart, LineController, LineElement, PointElement, CategoryScale, LinearScale } from 'chart.js';
Chart.register(LineController, LineElement, PointElement, CategoryScale, LinearScale);

// ==============================
// Next.js Bundle Analyzer
// ==============================
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // next config
});

// package.json
// "analyze": "ANALYZE=true next build"
```

---

## 7. Image Optimization

**Level:** [Junior] / [Mid]

**EN:** Explain WebP/AVIF formats, `srcset`/`sizes`, lazy loading, and the blur-up technique.

**VI:** Giải thích WebP/AVIF, `srcset`/`sizes`, lazy loading, và blur-up technique.

**Trả lời:**

```html
<!-- ==============================
     1. Modern formats
     ============================== -->
<!--
  JPEG: Lossy, good for photos, wide support — baseline
  WebP: 25-35% smaller than JPEG, lossy/lossless, Chrome/Firefox/Safari (iOS 14+)
  AVIF: 50% smaller than JPEG, HDR support, broader adoption now
  PNG: Lossless, good for logos/icons with transparency

  Strategy: AVIF first (best), WebP fallback, JPEG/PNG final fallback
-->

<picture>
  <source srcset="/hero.avif" type="image/avif" />
  <source srcset="/hero.webp" type="image/webp" />
  <img src="/hero.jpg" alt="Hero" width="1200" height="630" fetchpriority="high" />
</picture>

<!-- ==============================
     2. srcset + sizes — responsive images
     ============================== -->
<!--
  srcset: danh sách images với kích thước (w descriptors)
  sizes: media conditions → kích thước image sẽ hiển thị
  Browser chọn image phù hợp nhất dựa trên viewport + DPR
-->

<img
  src="/product-800.jpg"
  srcset="
    /product-400.jpg  400w,
    /product-800.jpg  800w,
    /product-1200.jpg 1200w,
    /product-1600.jpg 1600w
  "
  sizes="
    (max-width: 640px)  100vw,
    (max-width: 1024px) 50vw,
    400px
  "
  alt="Product"
  width="800"
  height="600"
  loading="lazy"
/>
<!--
  Ví dụ calculation:
  - Viewport: 375px (mobile), DPR: 2
  - Từ sizes: 100vw → 375px
  - Cần image: 375 * 2 = 750px wide
  - Browser chọn: 800w image (gần nhất ≥ 750)
-->

<!-- ==============================
     3. Lazy loading
     ============================== -->

<!-- Native lazy loading -->
<img src="/below-fold.jpg" loading="lazy" alt="..." width="600" height="400" />

<!-- Intersection Observer (more control) -->
```

```ts
// Intersection Observer lazy loading
function LazyImage({ src, alt, width, height, ...props }: ImgHTMLAttributes<HTMLImageElement> & { src: string }) {
  const imgRef = useRef<HTMLImageElement>(null);
  const [loaded, setLoaded] = useState(false);
  const [inView, setInView] = useState(false);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setInView(true);
          observer.disconnect();
        }
      },
      { rootMargin: '200px' } // preload khi cách viewport 200px
    );

    if (imgRef.current) observer.observe(imgRef.current);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={imgRef} style={{ width, height, background: '#f0f0f0' }}>
      {inView && (
        <img
          src={src}
          alt={alt}
          width={width}
          height={height}
          onLoad={() => setLoaded(true)}
          style={{ opacity: loaded ? 1 : 0, transition: 'opacity 0.3s' }}
          {...props}
        />
      )}
    </div>
  );
}

// ==============================
// 4. Blur-up technique (LQIP — Low Quality Image Placeholder)
// ==============================

// Generate tiny placeholder (10-20px wide) → base64
// Display blurred placeholder → load full image → fade in

function BlurUpImage({ src, placeholder, alt, width, height }: BlurUpImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);

  return (
    <div style={{ position: 'relative', overflow: 'hidden', width, height }}>
      {/* Blurred placeholder */}
      <img
        src={placeholder}  // tiny base64 image
        alt=""
        aria-hidden
        style={{
          position: 'absolute',
          inset: 0,
          width: '100%',
          height: '100%',
          objectFit: 'cover',
          filter: 'blur(20px)',
          transform: 'scale(1.1)', // hide blur edges
          opacity: isLoaded ? 0 : 1,
          transition: 'opacity 0.4s ease',
        }}
      />

      {/* Full quality image */}
      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        onLoad={() => setIsLoaded(true)}
        style={{
          width: '100%',
          height: '100%',
          objectFit: 'cover',
          opacity: isLoaded ? 1 : 0,
          transition: 'opacity 0.4s ease',
        }}
      />
    </div>
  );
}

// Next.js Image tự động làm tất cả những điều trên
import Image from 'next/image';

function OptimizedImage() {
  return (
    <Image
      src="/product.jpg"
      alt="Product"
      width={800}
      height={600}
      placeholder="blur"    // tự động generate blur placeholder
      blurDataURL="..."      // hoặc tự provide base64
      sizes="(max-width: 768px) 100vw, 50vw"
      loading="lazy"
    />
  );
}
```

---

## 8. Caching Strategies

**Level:** [Mid] / [Senior]

**EN:** Explain HTTP cache headers (`Cache-Control`, `ETag`), service worker cache strategies, and CDN caching.

**VI:** Giải thích HTTP cache headers, service worker cache strategies, và CDN caching.

**Trả lời:**

```ts
// ==============================
// 1. HTTP Cache Headers
// ==============================

// Express
app.use('/static', express.static('public', {
  maxAge: '1y',   // Cache-Control: max-age=31536000
  etag: true,
  lastModified: true,
}));

// Cache-Control directives:
// max-age=<seconds>  — time trong cache (mặc định private)
// s-maxage=<seconds> — time trong shared caches (CDN)
// public             — có thể cache bởi shared caches
// private            — chỉ cache trong browser (không CDN)
// no-cache           — phải revalidate với server trước khi dùng cache
// no-store           — không cache gì (sensitive data)
// immutable          — file không bao giờ thay đổi (versioned assets)
// stale-while-revalidate=<seconds> — dùng stale cache trong khi revalidate

// Versioned static assets (JS, CSS với content hash)
app.use('/assets', (req, res, next) => {
  // /assets/app.a1b2c3d4.js — content hash trong filename
  res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
  next();
}, express.static('dist/assets'));

// HTML — không cache hoặc cache ngắn
app.get('*', (req, res) => {
  res.setHeader('Cache-Control', 'no-cache, no-store, must-revalidate');
  // hoặc: 'public, max-age=0, must-revalidate'
  res.sendFile('dist/index.html');
});

// API responses
app.get('/api/products', (req, res) => {
  res.setHeader('Cache-Control', 'public, s-maxage=300, stale-while-revalidate=600');
  // CDN cache 5 phút, serve stale up to 10 phút trong khi revalidate
  res.json(products);
});

// ETag — conditional caching
app.get('/api/user/:id', async (req, res) => {
  const user = await getUser(req.params.id);
  const etag = `"${hashObject(user)}"`;

  // Client gửi If-None-Match: "etag-value"
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).send(); // Not Modified — không trả data
  }

  res.setHeader('ETag', etag);
  res.setHeader('Cache-Control', 'private, max-age=0, must-revalidate');
  res.json(user);
});

// ==============================
// 2. Service Worker Cache Strategies
// ==============================

// vite-plugin-pwa / workbox strategies

// Cache First: CDN images, fonts, versioned assets
// → Serve cache ngay, refresh background nếu network available

// Network First: API data, dynamic content
// → Thử network trước, fallback cache nếu offline

// Stale While Revalidate: Non-critical data
// → Serve cache ngay (fast), update cache từ network trong background

// workbox config
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      workbox: {
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/fonts\.googleapis\.com/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'google-fonts-stylesheets',
              expiration: { maxAgeSeconds: 60 * 60 * 24 * 365 },
            },
          },
          {
            urlPattern: /^https:\/\/api\.example\.com\/products/,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'api-products',
              networkTimeoutSeconds: 3,
              expiration: { maxEntries: 50, maxAgeSeconds: 300 },
            },
          },
          {
            urlPattern: /\.(?:png|jpg|jpeg|svg|gif|webp|avif)$/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'images',
              expiration: { maxEntries: 100, maxAgeSeconds: 30 * 24 * 60 * 60 },
            },
          },
        ],
      },
    }),
  ],
});
```

---

## 9. Critical Rendering Path

**Level:** [Mid] / [Senior]

**EN:** Explain the critical rendering path: HTML parsing, CSSOM, render tree, paint, and composite. What blocks rendering?

**VI:** Giải thích critical rendering path: HTML parsing, CSSOM, render tree, paint, và composite. Điều gì block rendering?

**Trả lời:**

```
Critical Rendering Path:
1. HTML → DOM (Document Object Model)
2. CSS → CSSOM (CSS Object Model)
3. DOM + CSSOM → Render Tree (chỉ visible elements)
4. Layout (Reflow) — tính toán position/size
5. Paint — fill pixels
6. Composite — layer composition → screen

CSS blocking: CSSOM phải complete trước khi render tree → CSS block rendering
Script blocking: <script> block HTML parsing (trừ async/defer)
```

```html
<!-- ==============================
     Optimize Critical Rendering Path
     ============================== -->

<!-- 1. Inline critical CSS — không cần network round trip -->
<head>
  <style>
    /* CSS cho above-the-fold content */
    body { margin: 0; font-family: system-ui, sans-serif; }
    .hero { min-height: 60vh; background: #f0f0f0; }
    .nav { height: 60px; background: white; }
  </style>

  <!-- Non-critical CSS: preload + onload trick -->
  <link rel="preload" as="style" href="/full-styles.css"
        onload="this.onload=null;this.rel='stylesheet'" />
  <noscript><link rel="stylesheet" href="/full-styles.css" /></noscript>

  <!-- 2. Script: defer > async > blocking -->
  <!-- defer: execute after HTML parsed, before DOMContentLoaded, in order -->
  <script defer src="/app.js"></script>

  <!-- async: execute khi load, không đảm bảo thứ tự (analytics, ads) -->
  <script async src="/analytics.js"></script>

  <!-- Không cần defer nếu script ở cuối body -->
</head>

<!-- 3. Resource hints -->
<link rel="dns-prefetch" href="//fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link rel="preload" as="font" href="/fonts/inter.woff2" crossorigin type="font/woff2" />
```

```ts
// ==============================
// Paint vs Layout vs Composite
// ==============================

/*
  Layout (Reflow) — tốn kém nhất:
  Trigger bởi: width, height, margin, padding, top, left, display, position, font-size
  → Recompute toàn bộ layout tree

  Paint:
  Trigger bởi: color, background, box-shadow, border-color
  → Repaint pixels, không recompute layout

  Composite (cheapest):
  Trigger bởi: transform, opacity, filter (sometimes)
  → Chỉ compositing GPU layers
  → Không trigger layout hoặc paint

  Performance rule: Animate transform/opacity, không phải top/left/width/height
*/

// CSS cho smooth animations
const goodAnimation = css`
  /* Tốt: composite only */
  transition: transform 0.3s ease, opacity 0.3s ease;

  &:hover {
    transform: translateY(-4px);  /* không trigger layout */
    opacity: 0.9;                  /* composite only */
  }
`;

const badAnimation = css`
  /* Xấu: trigger layout */
  transition: top 0.3s ease, width 0.3s ease;

  &:hover {
    top: -4px;    /* trigger layout */
    width: 110%;  /* trigger layout */
  }
`;

// Forced synchronous layout (Layout thrashing)
// XẤU — read/write alternating trong loop
function badLayoutThrasher(elements: Element[]) {
  elements.forEach((el) => {
    const width = el.getBoundingClientRect().width; // READ — force layout
    el.style.height = width + 'px';                 // WRITE
    // Next iteration: getBoundingClientRect() triggers layout lại vì style changed
  });
}

// TỐT — batch reads then writes
function goodBatchedLayout(elements: Element[]) {
  // Batch all reads
  const widths = elements.map((el) => el.getBoundingClientRect().width);

  // Batch all writes
  elements.forEach((el, i) => {
    el.style.height = widths[i] + 'px';
  });
}

// FastDOM library tự động batch reads/writes
// import fastdom from 'fastdom';
// fastdom.measure(() => { const w = el.offsetWidth; });
// fastdom.mutate(() => { el.style.width = '100px'; });
```

---

## 10. Resource Hints

**Level:** [Mid]

**EN:** Explain `preload`, `prefetch`, `preconnect`, and `dns-prefetch`. What are the differences and when do you use each?

**VI:** Giải thích sự khác biệt giữa `preload`, `prefetch`, `preconnect`, và `dns-prefetch`. Khi nào dùng từng loại?

**Trả lời:**

| Hint | Thời điểm fetch | Priority | Use case |
|------|----------------|----------|----------|
| `preload` | Ngay lập tức | High | Critical resources cần trong page hiện tại |
| `prefetch` | Lúc browser idle | Lowest | Resources cần cho page tiếp theo |
| `preconnect` | Ngay lập tức (connection) | Medium | Connections đến third-party origins quan trọng |
| `dns-prefetch` | Ngay lập tức (DNS only) | Low | DNS lookup cho third-party origins ít quan trọng hơn |

```html
<!-- ==============================
     preload — "Tôi CẦN resource này sớm nhất có thể"
     ============================== -->
<!-- as attribute bắt buộc — browser xác định priority -->
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
<link rel="preload" href="/hero.webp" as="image" />
<link rel="preload" href="/critical.css" as="style" />
<link rel="preload" href="/app.js" as="script" />

<!-- Preload với media attribute -->
<link rel="preload" href="/hero-mobile.webp" as="image" media="(max-width: 640px)" />
<link rel="preload" href="/hero-desktop.webp" as="image" media="(min-width: 641px)" />

<!-- ==============================
     prefetch — "Tôi CÓ THỂ cần resource này cho navigation tiếp theo"
     ============================== -->
<!-- Load lúc idle, lưu cache cho lần dùng sau -->
<link rel="prefetch" href="/next-page-image.webp" />
<link rel="prefetch" href="/user-settings-chunk.js" />

<!-- Webpack magic comment -->
<!-- const Settings = lazy(() => import(/* webpackPrefetch: true */ './Settings')); -->

<!-- ==============================
     preconnect — "Tôi sẽ cần kết nối đến origin này"
     ============================== -->
<!-- Thực hiện DNS + TCP + TLS handshake trước -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<!-- crossorigin cho CORS connections (fonts) -->
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link rel="preconnect" href="https://api.example.com" />
<link rel="preconnect" href="https://cdn.analytics.com" />

<!-- ==============================
     dns-prefetch — "Có thể cần DNS lookup cho origin này"
     ============================== -->
<!-- Chỉ DNS lookup — ít cost hơn preconnect -->
<!-- Dùng cho origins less certain, hoặc browser không support preconnect -->
<link rel="dns-prefetch" href="//third-party.com" />
<link rel="dns-prefetch" href="//another-cdn.net" />

<!-- Kết hợp: preconnect cho quan trọng, dns-prefetch fallback -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="dns-prefetch" href="//fonts.googleapis.com" />
```

```ts
// Dynamic prefetch dựa trên user behavior
function prefetchOnHover(href: string) {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = href;
  document.head.appendChild(link);
}

// React: prefetch khi hover link
function PrefetchLink({ to, children, ...props }: LinkProps) {
  const handleMouseEnter = useCallback(() => {
    prefetchOnHover(to as string);
  }, [to]);

  return (
    <Link to={to} onMouseEnter={handleMouseEnter} {...props}>
      {children}
    </Link>
  );
}

// Quicklink library — tự động prefetch links in viewport
// import { listen } from 'quicklink';
// listen(); // prefetch all links when they enter viewport
```

---

## 11. Performance Monitoring

**Level:** [Mid] / [Senior]

**EN:** What tools do you use to monitor web performance? Explain Lighthouse, the Web Vitals library, RUM, and the Chrome DevTools Performance tab.

**VI:** Tools nào để monitor web performance? Lighthouse, Web Vitals library, RUM, và Chrome DevTools Performance tab?

**Trả lời:**

```ts
// ==============================
// 1. Web Vitals Library — RUM (Real User Monitoring)
// ==============================
import { onLCP, onINP, onCLS, onFCP, onTTFB, type Metric } from 'web-vitals';

interface VitalsReport {
  name: string;
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
  id: string;
  delta: number;
  navigationType: string;
}

function sendToRUM(metric: Metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
    rating: metric.rating,
    id: metric.id,
    delta: metric.delta,
    url: window.location.href,
    userAgent: navigator.userAgent,
    timestamp: Date.now(),
  });

  // Use sendBeacon để không block page navigation
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/vitals', body);
  } else {
    fetch('/api/vitals', { method: 'POST', body, keepalive: true });
  }
}

onLCP(sendToRUM);
onINP(sendToRUM);
onCLS(sendToRUM);
onFCP(sendToRUM);
onTTFB(sendToRUM);

// ==============================
// 2. Performance Observer API
// ==============================
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    if (entry.entryType === 'longtask') {
      console.warn(`Long task detected: ${entry.duration}ms`, entry);
    }

    if (entry.entryType === 'navigation') {
      const nav = entry as PerformanceNavigationTiming;
      console.log({
        ttfb: nav.responseStart - nav.requestStart,
        domContentLoaded: nav.domContentLoadedEventEnd - nav.startTime,
        load: nav.loadEventEnd - nav.startTime,
      });
    }
  });
});

observer.observe({ entryTypes: ['longtask', 'navigation', 'resource', 'paint'] });

// ==============================
// 3. Performance marks — custom metrics
// ==============================
// Đặt trong code để measure specific operations
performance.mark('search-start');

const results = await searchProducts(query);

performance.mark('search-end');
performance.measure('search-duration', 'search-start', 'search-end');

const measure = performance.getEntriesByName('search-duration')[0];
console.log(`Search took: ${measure.duration}ms`);

// React: measure component render
function SearchComponent() {
  useEffect(() => {
    performance.mark('search-mounted');
    return () => {
      performance.measure('search-unmount', 'search-mounted');
    };
  }, []);
}

/*
  Chrome DevTools Performance tab:
  1. Record page load hoặc user interaction
  2. Flame chart: call stack theo thời gian
  3. Long tasks: đỏ trên main thread
  4. Layout/Paint: màu xanh lá/xanh dương
  5. Network waterfall

  Key metrics để xem:
  - First Paint, FCP, LCP trong Summary
  - Long tasks (> 50ms) — nguyên nhân INP kém
  - Layout thrashing — read/write alternating
  - JavaScript execution time
*/
```

---

## 12. React-specific Performance

**Level:** [Mid] / [Senior]

**EN:** How do you profile and optimize React performance? Explain the Profiler, `why-did-you-render`, virtualization, and preventing unnecessary re-renders.

**VI:** Cách profile và optimize React performance? Giải thích Profiler, `why-did-you-render`, virtualization, và tránh re-renders không cần thiết.

**Trả lời:**

```tsx
// ==============================
// 1. React DevTools Profiler
// ==============================
/*
  Record → Interact → Stop
  - Flamegraph: component tree, render time mỗi component
  - Ranked: sắp xếp theo render time
  - "Why did this render?": thay đổi props/state/context gây render

  Commit chứa màu xanh đậm = rendered, xám = không render
*/

// ==============================
// 2. why-did-you-render
// ==============================
// wdyr.ts (import đầu tiên trong index.tsx)
import React from 'react';

if (process.env.NODE_ENV === 'development') {
  const whyDidYouRender = require('@welldone-software/why-did-you-render');
  whyDidYouRender(React, {
    trackAllPureComponents: true,
    logOwnerReasons: true,
  });
}

// Mark specific component để track
function ExpensiveComponent({ data }: Props) {
  return <div>{/* ... */}</div>;
}

ExpensiveComponent.whyDidYouRender = true;

// ==============================
// 3. Prevent unnecessary re-renders
// ==============================

// memo — skip re-render nếu props không thay đổi (shallow comparison)
const UserCard = memo(function UserCard({ user, onSelect }: UserCardProps) {
  return (
    <div onClick={() => onSelect(user.id)}>
      {user.name}
    </div>
  );
});

// QUAN TRỌNG: callback props cần useCallback, objects cần useMemo
// Không dùng memo thì callback trong parent tạo ra function mới mỗi render → memo vô dụng

function UserList({ users }: { users: User[] }) {
  const [selectedId, setSelectedId] = useState<string | null>(null);

  // useCallback — stable reference
  const handleSelect = useCallback((id: string) => {
    setSelectedId(id);
  }, []); // empty deps — function không thay đổi

  return (
    <ul>
      {users.map((user) => (
        <UserCard key={user.id} user={user} onSelect={handleSelect} />
      ))}
    </ul>
  );
}

// useMemo — expensive computations
function FilteredList({ items, filter }: Props) {
  // Chỉ recompute khi items hoặc filter thay đổi
  const filteredItems = useMemo(
    () => items.filter((item) => item.name.includes(filter)).sort((a, b) => a.name.localeCompare(b.name)),
    [items, filter]
  );

  return <ul>{filteredItems.map((item) => <li key={item.id}>{item.name}</li>)}</ul>;
}

// Context — tránh re-render toàn bộ tree
// XẤU: 1 context chứa tất cả → mọi consumer re-render khi bất kỳ value nào thay đổi
const AppContext = createContext({ user: null, theme: 'light', cart: [] });

// TỐT: tách context theo tần suất thay đổi
const UserContext = createContext<User | null>(null);
const ThemeContext = createContext<'light' | 'dark'>('light');
const CartContext = createContext<CartItem[]>([]);

// ==============================
// 4. Virtualization — list ảo
// ==============================
// Chỉ render items trong viewport, không render toàn bộ list

// react-window
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }: { items: Item[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <ItemRow item={items[index]} />
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={80}    // height của mỗi row
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// TanStack Virtual (react-virtual) — flexible hơn, không cần container dimensions
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 80,
    overscan: 5, // render thêm 5 items ngoài viewport
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              transform: `translateY(${virtualItem.start}px)`,
              width: '100%',
            }}
          >
            <ItemRow item={items[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 13. Network Optimization

**Level:** [Mid] / [Senior]

**EN:** Explain HTTP/2 multiplexing, compression (gzip/brotli), tree shaking, and dead code elimination.

**VI:** Giải thích HTTP/2 multiplexing, compression gzip/brotli, tree shaking, và dead code elimination.

**Trả lời:**

```ts
// ==============================
// 1. HTTP/2 Multiplexing
// ==============================
/*
  HTTP/1.1: 1 request per connection → cần connection pooling, domain sharding
  HTTP/2: Multiplexing — nhiều requests trong 1 TCP connection

  Hệ quả cho performance strategy:
  - HTTP/1.1: bundle nhiều files thành 1 (giảm connections)
  - HTTP/2: nhiều small files OK (multiplexed), có thể split rộng hơn
  - Domain sharding (HTTP/1 hack) có hại với HTTP/2
  - HTTP/2 Server Push (deprecated in favor of preload hints)
*/

// Nginx HTTP/2 config
// server {
//   listen 443 ssl http2;
//   ssl_certificate /etc/ssl/cert.pem;
//   ssl_certificate_key /etc/ssl/key.pem;
// }

// ==============================
// 2. Compression
// ==============================

// Express compression
import compression from 'compression';

app.use(compression({
  level: 6,       // gzip compression level (1-9, 6 = balance)
  threshold: 1024, // chỉ compress files > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  },
}));

// Nginx với brotli (tốt hơn gzip ~15%)
// brotli on;
// brotli_comp_level 6;
// brotli_types text/html text/css application/javascript application/json;
// gzip on;
// gzip_vary on;
// gzip_types text/html text/css application/javascript application/json;

// Vite pre-compress assets
// vite.config.ts
import viteCompression from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    viteCompression({ algorithm: 'brotliCompress' }),  // .br files
    viteCompression({ algorithm: 'gzip' }),            // .gz files
  ],
});

// ==============================
// 3. Tree Shaking
// ==============================

/*
  Tree shaking: loại bỏ exports không được import (dead code elimination)
  Yêu cầu: ES Modules (import/export), không CommonJS (require)
  Rollup/Vite làm tốt, Webpack v4+ hỗ trợ
*/

// Tốt — named import → tree shaking works
import { debounce } from 'lodash-es';          // chỉ bundle debounce
import { format } from 'date-fns';             // chỉ bundle format

// Xấu — default import → có thể kéo cả library
import _ from 'lodash';                         // bundle toàn bộ lodash
import moment from 'moment';                    // bundle moment + locales

// Package.json sideEffects hint cho bundler
// package.json (library)
{
  "sideEffects": false,        // không có side effects → an toàn tree shake
  // "sideEffects": ["*.css"], // CSS files có side effects
}

// ==============================
// 4. Bundle Optimization
// ==============================

// vite.config.ts
export default defineConfig({
  build: {
    minify: 'terser', // hoặc 'esbuild' (nhanh hơn, ít compact hơn)
    terserOptions: {
      compress: {
        drop_console: true,    // xóa console.log trong production
        drop_debugger: true,
        pure_funcs: ['console.info', 'console.debug'],
      },
    },
    rollupOptions: {
      output: {
        // Xóa unused code
        treeshake: {
          moduleSideEffects: false,
          propertyReadSideEffects: false,
        },
      },
    },
    // Target modern browsers — nhỏ hơn do ít polyfills
    target: ['es2020', 'chrome80', 'firefox78', 'safari14'],
  },
});
```
