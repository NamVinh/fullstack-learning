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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Can you name the three Core Web Vitals and what each one measures?"**

**Model Answer:**
"Sure. LCP — Largest Contentful Paint — measures loading performance: how long it takes for the main content to appear on screen. The 'good' threshold is under 2.5 seconds. INP — Interaction to Next Paint — replaced FID and measures responsiveness: the delay between a user interaction like a click or key press and when the browser actually paints the next frame. Good is under 200 milliseconds. CLS — Cumulative Layout Shift — measures visual stability: how much the page layout unexpectedly jumps around during loading. A good CLS score is 0.1 or less. These three are important because Google uses them as ranking signals and they directly map to how users experience a page."

**Trả lời (Tiếng Việt):**
"Được. LCP — Largest Contentful Paint — đo hiệu suất loading: thời gian để nội dung chính hiển thị trên màn hình. Ngưỡng 'good' là dưới 2.5 giây. INP — Interaction to Next Paint — thay thế FID, đo khả năng phản hồi: độ trễ giữa lúc người dùng tương tác như click hoặc nhấn phím và khi trình duyệt paint frame tiếp theo. Good là dưới 200 milliseconds. CLS — Cumulative Layout Shift — đo độ ổn định giao diện: mức độ layout trang bị nhảy bất ngờ khi loading. CLS tốt là 0.1 hoặc thấp hơn. Ba chỉ số này quan trọng vì Google dùng chúng làm tín hiệu ranking và chúng phản ánh trực tiếp trải nghiệm thực tế của người dùng."

---

**Q2 (Mid): "Your LCP score is 4.2 seconds. Walk me through how you'd investigate and fix that."**

**Model Answer:**
"I'd start by identifying what the LCP element actually is — open Chrome DevTools or PageSpeed Insights and they'll tell you which element triggered LCP. It's usually a hero image, an h1, or a large background image.

Then I'd look at the four common causes. First, slow server response — high TTFB means the HTML took too long to arrive. I'd check if the server is using caching. Second, render-blocking resources — CSS or scripts blocking the browser from painting. I'd inline critical CSS and add `defer` to scripts. Third, the LCP resource itself loading slowly — for images, I'd add `fetchpriority='high'` on the LCP image and a `<link rel='preload'>` in the head so the browser discovers it earlier. I'd also make sure I'm serving modern formats like WebP or AVIF. Fourth, if it's a client-rendered app, the LCP might not be in the initial HTML at all — SSR or SSG would fix that.

In Next.js I'd use `<Image priority />` which handles the preload and fetchpriority automatically."

**Trả lời (Tiếng Việt):**
"Tôi sẽ bắt đầu bằng cách xác định LCP element thực sự là gì — mở Chrome DevTools hoặc PageSpeed Insights và chúng sẽ chỉ ra element nào trigger LCP. Thường là hero image, h1, hoặc background image lớn.

Sau đó tôi sẽ xem xét bốn nguyên nhân phổ biến. Thứ nhất, server response chậm — TTFB cao nghĩa là HTML mất quá lâu để đến. Tôi sẽ kiểm tra server có dùng caching không. Thứ hai, render-blocking resources — CSS hoặc script đang chặn trình duyệt paint. Tôi sẽ inline critical CSS và thêm `defer` vào scripts. Thứ ba, resource LCP tự nó load chậm — với image, tôi sẽ thêm `fetchpriority='high'` trên LCP image và `<link rel='preload'>` trong head để trình duyệt phát hiện sớm hơn. Tôi cũng đảm bảo dùng các format hiện đại như WebP hoặc AVIF. Thứ tư, nếu là client-rendered app, LCP có thể không có trong HTML ban đầu — SSR hoặc SSG sẽ giải quyết điều đó.

Trong Next.js tôi sẽ dùng `<Image priority />` tự động xử lý preload và fetchpriority."

---

**Q3 (Mid): "What is CLS and how do you fix layout shifts caused by images and web fonts?"**

**Model Answer:**
"CLS measures how much content unexpectedly moves around during page load, which is frustrating for users trying to click things. The score accumulates every unexpected shift.

For images, the fix is simple: always set explicit `width` and `height` attributes. The browser then reserves space before the image loads, so nothing shifts. For responsive images I use `aspect-ratio` in CSS to maintain the right proportions.

For fonts, the issue is that the browser renders fallback text first, then swaps to the custom font when it loads, and the two fonts are usually different sizes. The best approach is `font-display: optional` which skips the swap entirely if the font hasn't loaded in time. If I need the custom font, I use `size-adjust`, `ascent-override`, and `descent-override` in CSS to make the fallback font match the metrics of the custom font as closely as possible — so when the swap happens, there's minimal visual shift.

The other big CLS source is injecting content above the fold after load, like cookie banners. The fix is to reserve space for them from the start, or use a fixed/sticky overlay so they don't push content down."

**Trả lời (Tiếng Việt):**
"CLS đo mức độ nội dung bị dịch chuyển bất ngờ trong quá trình page load, gây khó chịu cho người dùng khi đang cố click vào thứ gì đó. Score tích lũy qua mỗi lần shift bất ngờ.

Với images, fix rất đơn giản: luôn đặt explicit `width` và `height` attributes. Trình duyệt sẽ reserve space trước khi image load, nên không có gì bị shift. Với responsive images tôi dùng `aspect-ratio` trong CSS để duy trì đúng tỷ lệ.

Với fonts, vấn đề là trình duyệt render fallback text trước, rồi swap sang custom font khi load xong, và hai font thường có kích thước khác nhau. Cách tốt nhất là `font-display: optional` — bỏ qua việc swap hoàn toàn nếu font chưa load kịp. Nếu cần custom font, tôi dùng `size-adjust`, `ascent-override`, và `descent-override` trong CSS để làm cho fallback font khớp metrics của custom font — khi swap xảy ra, visual shift là tối thiểu.

Nguồn CLS lớn khác là inject content phía trên fold sau khi load, như cookie banner. Fix là reserve space cho chúng từ đầu, hoặc dùng fixed/sticky overlay để không đẩy content xuống."

---

**Q4 (Senior): "What replaced FID, and why? How does INP differ from FID in what it measures?"**

**Model Answer:**
"FID — First Input Delay — was replaced by INP, Interaction to Next Paint, because FID had a significant limitation: it only measured the input delay of the very first user interaction on the page. If that happened to be a lightweight click, you'd get a great FID score even if every subsequent interaction was sluggish.

INP is more comprehensive. It samples all interactions throughout the entire page lifecycle — clicks, key presses, taps — and reports the worst one, or close to it. It measures the full duration from when the user interacts to when the browser paints the visual response, not just the queueing delay.

The root cause of poor INP is long tasks on the main thread. If a user clicks a button while JavaScript is running a 500ms task, the browser can't respond until that task finishes. The fixes involve breaking up long tasks using `setTimeout(0)` to yield to the browser, using `useTransition` in React to mark state updates as non-urgent, or offloading heavy computation to Web Workers."

**Trả lời (Tiếng Việt):**
"FID — First Input Delay — bị thay thế bởi INP, Interaction to Next Paint, vì FID có hạn chế đáng kể: nó chỉ đo input delay của interaction đầu tiên của người dùng trên trang. Nếu đó là một click nhẹ, bạn sẽ có FID score tốt dù mọi interaction sau đó đều chậm chạp.

INP toàn diện hơn. Nó sample tất cả interaction trong suốt vòng đời của trang — clicks, key presses, taps — và report lần tệ nhất, hoặc gần với lần tệ nhất. Nó đo toàn bộ thời gian từ lúc người dùng tương tác đến khi trình duyệt paint phản hồi trực quan, không chỉ là queueing delay.

Nguyên nhân gốc của INP kém là long tasks trên main thread. Nếu người dùng click button trong khi JavaScript đang chạy task 500ms, trình duyệt không thể phản hồi cho đến khi task đó hoàn thành. Cách fix bao gồm chia nhỏ long tasks bằng `setTimeout(0)` để nhường quyền cho trình duyệt, dùng `useTransition` trong React để đánh dấu state update là không khẩn cấp, hoặc offload heavy computation sang Web Workers."

---

**Q5 (Senior): "You're measuring Core Web Vitals with the web-vitals library. How do you send this data to your analytics and why use sendBeacon instead of fetch?"**

**Model Answer:**
"I import the metric callbacks from the web-vitals library — `onLCP`, `onINP`, `onCLS` and so on — and pass them a handler function. That handler receives a metric object with the name, value, rating, and a unique id.

For sending to analytics, I use `navigator.sendBeacon` when available, and fall back to `fetch` with `keepalive: true`. The reason for sendBeacon is that web vitals like LCP and CLS are often finalized when the user is navigating away from the page. Regular fetch requests can be cancelled by the browser when the page unloads. sendBeacon is designed specifically for this — it queues a small POST request and guarantees delivery even during page unload, without blocking the navigation. The keepalive fetch option is a reasonable fallback for browsers that don't support sendBeacon."

**Trả lời (Tiếng Việt):**
"Tôi import các metric callback từ web-vitals library — `onLCP`, `onINP`, `onCLS` và các cái khác — và truyền vào một handler function. Handler đó nhận metric object với name, value, rating, và unique id.

Để gửi lên analytics, tôi dùng `navigator.sendBeacon` khi có thể, và fallback về `fetch` với `keepalive: true`. Lý do dùng sendBeacon là web vitals như LCP và CLS thường được finalize khi người dùng đang rời trang. Các request fetch thông thường có thể bị trình duyệt cancel khi trang unload. sendBeacon được thiết kế đặc biệt cho trường hợp này — nó queue một POST request nhỏ và đảm bảo delivery ngay cả khi trang đang unload, mà không chặn navigation. Fetch với keepalive là fallback hợp lý cho các trình duyệt không hỗ trợ sendBeacon."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is code splitting and why would you use it?"**

**Model Answer:**
"Code splitting is the practice of dividing your JavaScript bundle into smaller chunks that load on demand rather than all at once. By default, a bundler like Webpack or Vite will produce one large bundle containing everything. If a user just visits your home page, they don't need the code for the settings page, the admin panel, or a PDF export feature — but without code splitting, they'd download all of it anyway.

With code splitting, each route or heavy component gets its own chunk file. The browser only downloads the code it needs for the current view, and lazily loads the rest when the user navigates to that part of the app. This improves initial load time significantly, especially on mobile or slow connections."

**Trả lời (Tiếng Việt):**
"Code splitting là kỹ thuật chia JavaScript bundle thành các chunk nhỏ hơn, chỉ load khi cần thay vì tất cả cùng một lúc. Mặc định, một bundler như Webpack hay Vite sẽ tạo ra một bundle lớn chứa tất cả mọi thứ. Nếu người dùng chỉ vào trang chủ, họ không cần code của trang settings, admin panel, hay tính năng export PDF — nhưng nếu không có code splitting, họ vẫn phải tải hết.

Với code splitting, mỗi route hoặc component nặng sẽ có chunk file riêng. Trình duyệt chỉ download code cần thiết cho view hiện tại, và lazily load phần còn lại khi người dùng điều hướng đến đó. Điều này cải thiện đáng kể initial load time, đặc biệt trên mobile hoặc kết nối chậm."

---

**Q2 (Mid): "How does React.lazy work under the hood? What does Suspense do while the chunk is loading?"**

**Model Answer:**
"React.lazy takes a function that calls a dynamic import — so `React.lazy(() => import('./Dashboard'))`. When the component is first rendered, React triggers that dynamic import which returns a Promise. React suspends the render of that component — meaning it throws the Promise — and the nearest Suspense boundary catches it. The fallback UI in the Suspense boundary is shown while the Promise is pending.

Once the chunk finishes downloading, React re-renders the suspended subtree with the now-available component. It's important to wrap lazy-loaded components in Suspense so you control what users see during the loading state rather than getting an unhandled error.

For routes, the most impactful pattern is route-level splitting — each page becomes a separate chunk so users only download code for pages they actually visit."

**Trả lời (Tiếng Việt):**
"React.lazy nhận một function gọi dynamic import — ví dụ `React.lazy(() => import('./Dashboard'))`. Khi component được render lần đầu, React trigger dynamic import đó trả về một Promise. React suspend render của component đó — nghĩa là nó throw Promise — và Suspense boundary gần nhất bắt nó. Fallback UI trong Suspense boundary được hiển thị trong khi Promise đang pending.

Khi chunk tải xong, React re-render subtree bị suspend với component đã có sẵn. Quan trọng là phải wrap lazy-loaded components trong Suspense để bạn kiểm soát được người dùng thấy gì trong loading state, thay vì bị lỗi unhandled.

Với routes, pattern có impact lớn nhất là route-level splitting — mỗi page là một chunk riêng để người dùng chỉ download code của các page họ thực sự truy cập."

---

**Q3 (Mid): "When would you use webpackPrefetch versus webpackPreload in a dynamic import comment?"**

**Model Answer:**
"These are webpack magic comments that give hints to the browser about how urgently a chunk is needed.

`webpackPreload` tells webpack to include the chunk in the parent chunk's network request — it loads in parallel with the current page. Use this for resources you are highly confident will be needed right away but couldn't be included in the initial bundle for some reason.

`webpackPrefetch` is much more common and appropriate for most lazy loading scenarios. It tells webpack to inject a `<link rel='prefetch'>` tag, and the browser will download that chunk during idle time. Use it for routes the user is likely to navigate to next — for example, if you're on a login page, you might prefetch the dashboard chunk so when the user logs in successfully, the chunk is already cached. The key difference is timing: preload is urgent and competes with the current page's resources, prefetch is low priority and happens when the browser is idle."

**Trả lời (Tiếng Việt):**
"Đây là các webpack magic comment cho trình duyệt biết độ ưu tiên cần thiết của một chunk.

`webpackPreload` nói với webpack để include chunk vào network request của parent chunk — nó load song song với trang hiện tại. Dùng khi bạn chắc chắn cao rằng resource đó sẽ cần ngay lập tức nhưng không thể include vào initial bundle vì lý do nào đó.

`webpackPrefetch` phổ biến hơn nhiều và phù hợp cho hầu hết các lazy loading scenario. Nó nói với webpack inject tag `<link rel='prefetch'>`, và trình duyệt sẽ download chunk đó trong thời gian idle. Dùng cho các route người dùng có khả năng navigate đến tiếp theo — ví dụ, nếu bạn đang ở trang login, bạn có thể prefetch dashboard chunk để khi người dùng đăng nhập thành công, chunk đó đã có trong cache. Sự khác biệt chính là timing: preload khẩn cấp và cạnh tranh với resources của trang hiện tại, prefetch có độ ưu tiên thấp và xảy ra khi trình duyệt rảnh."

---

**Q4 (Senior): "You have a large React app and the initial bundle is 800KB. Walk me through how you'd analyze and reduce it."**

**Model Answer:**
"First, I'd visualize what's in the bundle. For Vite I'd use the `rollup-plugin-visualizer` which generates a treemap showing each module's size. For Webpack I'd use `webpack-bundle-analyzer`. This gives me an immediate picture of what's large.

Looking at the treemap, I'd identify the biggest wins. Common culprits are libraries like Moment.js which is 67KB and can be replaced by date-fns or dayjs. Full Lodash import instead of cherry-picked imports or lodash-es. Chart libraries loaded in full when only a couple chart types are used.

Then I'd make sure route-based splitting is in place — each page as a separate lazy-loaded chunk. Heavy editor components, PDF libraries, or data visualization tools should be split off and only loaded when triggered.

On the bundler config side, I'd set up manual chunks to group stable vendor libraries like React and React Router into their own cache-friendly chunk. I'd also check the build targets — targeting modern ES2020+ means less polyfill code.

Finally I'd look at `sideEffects: false` in package.json for my own packages, which lets bundlers tree-shake more aggressively. After changes, I'd run the analyzer again and compare gzip sizes rather than raw sizes, since that's what actually travels over the network."

**Trả lời (Tiếng Việt):**
"Đầu tiên, tôi sẽ visualize những gì có trong bundle. Với Vite tôi dùng `rollup-plugin-visualizer` tạo ra treemap hiển thị kích thước của từng module. Với Webpack tôi dùng `webpack-bundle-analyzer`. Điều này cho tôi thấy ngay những gì to.

Nhìn vào treemap, tôi sẽ xác định những cơ hội tiết kiệm lớn nhất. Thủ phạm thường gặp là: Moment.js nặng 67KB có thể thay bằng date-fns hoặc dayjs. Import toàn bộ Lodash thay vì cherry-pick hoặc dùng lodash-es. Chart library load đầy đủ khi chỉ dùng một vài loại chart.

Sau đó tôi sẽ đảm bảo route-based splitting đã được thiết lập — mỗi page là một lazy-loaded chunk riêng. Các heavy editor component, PDF library, hay data visualization tool nên được tách ra và chỉ load khi cần.

Về phía bundler config, tôi sẽ setup manual chunks để gom các vendor library ổn định như React và React Router vào chunk riêng thân thiện với cache. Tôi cũng kiểm tra build targets — target ES2020+ trở lên có nghĩa là ít polyfill code hơn.

Cuối cùng tôi sẽ xem xét `sideEffects: false` trong package.json của các package riêng, cho phép bundler tree-shake mạnh hơn. Sau khi thay đổi, tôi sẽ chạy analyzer lại và so sánh gzip size thay vì raw size, vì đó là thứ thực sự truyền qua mạng."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior/Mid): "You found that moment.js is taking up a large portion of your bundle. How do you fix it?"**

**Model Answer:**
"Moment.js is a classic bundle size problem — the full library is around 67KB gzipped, and it pulls in all its locale data unless you configure it carefully.

The cleanest solution is to replace it entirely. For most use cases, date-fns is a much better choice because it's tree-shakable — you only import the specific functions you use, like `format` or `addDays`, and the bundler removes everything else. It ends up being maybe 5-10KB in practice. If I need an even lighter option, Day.js is about 2.6KB and has a plugin system.

If I absolutely can't replace Moment for some reason — maybe a legacy codebase — I'd use webpack's IgnorePlugin to prevent Moment from loading all locales, and then manually import only the locales I need. But the right answer is usually to migrate away from it."

**Trả lời (Tiếng Việt):**
"Moment.js là vấn đề bundle size kinh điển — thư viện đầy đủ khoảng 67KB gzipped, và kéo theo tất cả locale data trừ khi bạn cấu hình cẩn thận.

Giải pháp sạch nhất là thay thế hoàn toàn. Với hầu hết use case, date-fns là lựa chọn tốt hơn nhiều vì nó tree-shakable — bạn chỉ import các function cụ thể bạn dùng, như `format` hay `addDays`, và bundler loại bỏ tất cả phần còn lại. Thực tế có thể chỉ 5-10KB. Nếu cần option nhẹ hơn, Day.js khoảng 2.6KB và có plugin system.

Nếu vì lý do nào đó không thể thay Moment — có thể là codebase legacy — tôi sẽ dùng webpack's IgnorePlugin để ngăn Moment load tất cả locale, rồi manually import chỉ locale cần dùng. Nhưng câu trả lời đúng thường là migrate ra khỏi nó."

---

**Q2 (Mid): "What's the difference between the bundle tools you use to analyze bundle size before shipping?"**

**Model Answer:**
"There are a few layers. Before installing a new package, I'll check bundlephobia.com — you paste in a package name and it shows you the minified and gzipped size, plus the size of all its dependencies. This is great for making decisions like 'should I use this date library or that one.'

Once I'm building the actual app, for Vite projects I use `rollup-plugin-visualizer`. It hooks into the build and generates an interactive treemap at `dist/stats.html` showing every module and its contribution to each chunk, with gzip and brotli sizes. For webpack or Next.js apps I use `@next/bundle-analyzer` or `webpack-bundle-analyzer`.

The treemap view is the most useful because I can immediately see what's disproportionately large. If I see a big square for something unexpected, that's my first target."

**Trả lời (Tiếng Việt):**
"Có một vài lớp. Trước khi cài một package mới, tôi sẽ kiểm tra bundlephobia.com — bạn paste tên package và nó hiển thị kích thước minified và gzipped, cộng với kích thước của tất cả dependencies. Rất hữu ích để quyết định như 'nên dùng date library này hay cái kia.'

Khi đang build app thực tế, với Vite projects tôi dùng `rollup-plugin-visualizer`. Nó hook vào build và tạo ra interactive treemap tại `dist/stats.html` hiển thị mọi module và đóng góp của nó vào mỗi chunk, với gzip và brotli size. Với webpack hoặc Next.js apps tôi dùng `@next/bundle-analyzer` hoặc `webpack-bundle-analyzer`.

Treemap view hữu ích nhất vì tôi có thể thấy ngay cái gì quá lớn một cách bất thường. Nếu thấy ô vuông lớn cho thứ gì đó không mong đợi, đó là target đầu tiên của tôi."

---

**Q3 (Senior): "Explain tree shaking — what makes it work, and what can break it?"**

**Model Answer:**
"Tree shaking is the process where the bundler statically analyzes import and export statements and removes exports that are never imported anywhere — dead code elimination. The key requirement is ES Modules syntax: `import` and `export`. CommonJS `require()` calls are dynamic and evaluated at runtime, so the bundler can't know statically what's used.

What makes it work well: using named imports like `import { debounce } from 'lodash-es'` rather than default imports. The `sideEffects: false` field in a package's `package.json` tells the bundler it's safe to remove any module from that package that isn't directly imported — without it, bundlers have to be conservative and keep everything in case importing a module has a side effect like registering something globally.

What breaks it: CSS imports often have side effects — importing a CSS file causes it to be injected into the page, so `sideEffects: ['*.css']` protects those. Also, top-level function calls with side effects in a module — like `console.log()` at module level — make the bundler reluctant to remove that module. And circular dependencies can confuse static analysis."

**Trả lời (Tiếng Việt):**
"Tree shaking là quá trình bundler phân tích tĩnh các import và export statement rồi loại bỏ những export không bao giờ được import ở đâu cả — loại bỏ dead code. Yêu cầu chính là cú pháp ES Modules: `import` và `export`. Các lệnh gọi `require()` của CommonJS là dynamic và được evaluate lúc runtime, nên bundler không thể biết tĩnh những gì được dùng.

Điều làm nó hoạt động tốt: dùng named imports như `import { debounce } from 'lodash-es'` thay vì default imports. Field `sideEffects: false` trong `package.json` của package báo cho bundler biết an toàn để loại bỏ bất kỳ module nào từ package đó mà không được import trực tiếp — không có nó, bundler phải conservative và giữ tất cả để phòng trường hợp import module có side effect như đăng ký thứ gì đó globally.

Điều làm nó bị phá vỡ: CSS imports thường có side effects — import một file CSS khiến nó được inject vào trang, nên `sideEffects: ['*.css']` bảo vệ chúng. Ngoài ra, top-level function calls với side effects trong một module — như `console.log()` ở module level — khiến bundler ngần ngại loại bỏ module đó. Và circular dependencies có thể làm rối static analysis."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Why should you use WebP instead of JPEG for images on the web?"**

**Model Answer:**
"WebP is a modern image format that produces significantly smaller file sizes compared to JPEG — typically 25 to 35 percent smaller at comparable visual quality. Smaller files mean faster downloads, which directly improves page load time and LCP. WebP supports both lossy and lossless compression, and it also supports transparency like PNG, so it can replace both formats.

Browser support is now excellent — Chrome, Firefox, and Safari all support it. For the rare browser that doesn't, you use the `<picture>` element with a `<source>` for WebP and a regular `<img>` as the JPEG fallback. AVIF is even better — about 50 percent smaller than JPEG — but WebP is currently the safe baseline modern format."

**Trả lời (Tiếng Việt):**
"WebP là format image hiện đại tạo ra file nhỏ hơn đáng kể so với JPEG — thường 25 đến 35 phần trăm nhỏ hơn ở chất lượng hình ảnh tương đương. File nhỏ hơn có nghĩa là download nhanh hơn, cải thiện trực tiếp page load time và LCP. WebP hỗ trợ cả lossy lẫn lossless compression, và còn hỗ trợ transparency như PNG, nên có thể thay thế cả hai format.

Browser support hiện rất tốt — Chrome, Firefox và Safari đều hỗ trợ. Với trình duyệt hiếm không hỗ trợ, bạn dùng element `<picture>` với `<source>` cho WebP và `<img>` thông thường làm JPEG fallback. AVIF còn tốt hơn — khoảng 50 phần trăm nhỏ hơn JPEG — nhưng WebP hiện tại là format hiện đại an toàn nhất để dùng làm baseline."

---

**Q2 (Mid): "Explain what srcset and sizes do and why both are needed."**

**Model Answer:**
"They work together to let the browser pick the right image for the user's screen.

`srcset` gives the browser a list of available image files and their widths — for example, a 400px version, an 800px version, and a 1200px version. But the browser doesn't yet know how large the image will actually appear on screen, because that depends on CSS layout.

That's where `sizes` comes in. It tells the browser how wide the image will render under different viewport conditions, using media conditions. For example: on screens under 640px wide, the image takes up 100% of the viewport; on larger screens, it takes up 50% of the viewport.

With both pieces of information — available files and display size — the browser can calculate which image to request. It multiplies the display size by the device pixel ratio and picks the closest match from srcset. This means mobile users on a 2x screen don't download a 1200px image when a 400px image would look identical to them. Without `sizes`, the browser just guesses, usually downloading a larger image than needed."

**Trả lời (Tiếng Việt):**
"Hai thuộc tính này làm việc cùng nhau để trình duyệt chọn đúng image cho màn hình của người dùng.

`srcset` cung cấp cho trình duyệt danh sách các file image và chiều rộng của chúng — ví dụ, phiên bản 400px, 800px, và 1200px. Nhưng trình duyệt chưa biết image sẽ hiển thị lớn bao nhiêu trên màn hình, vì điều đó phụ thuộc vào CSS layout.

Đây là lúc `sizes` phát huy tác dụng. Nó nói với trình duyệt image sẽ render rộng bao nhiêu ở các điều kiện viewport khác nhau, dùng media conditions. Ví dụ: trên màn hình dưới 640px, image chiếm 100% viewport; trên màn hình lớn hơn, nó chiếm 50% viewport.

Với cả hai thông tin — file available và display size — trình duyệt có thể tính toán image nào cần request. Nó nhân display size với device pixel ratio và chọn cái gần nhất từ srcset. Điều này có nghĩa là người dùng mobile trên màn hình 2x không download image 1200px khi image 400px trông giống hệt với họ. Không có `sizes`, trình duyệt chỉ đoán, thường download image lớn hơn mức cần."

---

**Q3 (Mid): "What is the blur-up technique and when does it improve perceived performance?"**

**Model Answer:**
"Blur-up, also called LQIP — Low Quality Image Placeholder — is a technique where you show a tiny, heavily blurred version of an image immediately, then fade in the full-resolution version once it loads.

The tiny placeholder might be a 10 to 20 pixel wide JPEG encoded as a base64 data URL, so it's inline in the HTML and requires no extra network request. It renders instantly. Then while the browser is downloading the actual image in the background, the user sees a blurred preview rather than an empty grey box. When the full image loads, you fade it in smoothly.

It improves perceived performance on below-the-fold images because it gives users visual context immediately — they can see the image composition before it fully loads. However, for the LCP image — the hero image at the top of the page — you should NOT use blur-up or lazy loading. That image needs `fetchpriority='high'` and a preload link so it loads as fast as possible. Blur-up is best suited for article images, product galleries, and anything below the initial viewport. Next.js Image with `placeholder='blur'` handles this automatically."

**Trả lời (Tiếng Việt):**
"Blur-up, còn gọi là LQIP — Low Quality Image Placeholder — là kỹ thuật hiển thị một phiên bản rất nhỏ, heavily blurred của image ngay lập tức, rồi fade in phiên bản full-resolution khi nó load xong.

Placeholder nhỏ có thể là JPEG rộng 10 đến 20 pixel được encode dưới dạng base64 data URL, nên nó inline trong HTML và không cần network request thêm. Nó render ngay lập tức. Sau đó trong khi trình duyệt download image thực tế ở background, người dùng thấy preview mờ thay vì hộp xám trống. Khi image đầy đủ load xong, bạn fade in mượt mà.

Kỹ thuật này cải thiện perceived performance cho các image below-the-fold vì nó cho người dùng visual context ngay lập tức — họ có thể thấy bố cục image trước khi nó load đầy đủ. Tuy nhiên, với LCP image — hero image ở đầu trang — bạn KHÔNG nên dùng blur-up hay lazy loading. Image đó cần `fetchpriority='high'` và preload link để load nhanh nhất có thể. Blur-up phù hợp nhất cho article images, product gallery, và bất cứ thứ gì below the initial viewport. Next.js Image với `placeholder='blur'` xử lý điều này tự động."

---

**Q4 (Junior/Mid): "What happens if you put loading='lazy' on your LCP image?"**

**Model Answer:**
"It would hurt your LCP score significantly. `loading='lazy'` tells the browser to defer loading the image until it's near the viewport. The browser intentionally delays discovering and fetching the image. But the LCP image is the most important visual element on the page — you want the browser to start fetching it as early as possible.

For the LCP image, you want the opposite approach: `fetchpriority='high'` to tell the browser to prioritize this image, and ideally a `<link rel='preload'>` in the `<head>` so the browser doesn't even need to parse the img tag before starting the download. Lazy loading should only be applied to images below the fold."

**Trả lời (Tiếng Việt):**
"Điều đó sẽ làm hỏng LCP score đáng kể. `loading='lazy'` nói với trình duyệt defer load image cho đến khi nó gần viewport. Trình duyệt cố ý trì hoãn việc phát hiện và fetch image. Nhưng LCP image là element visual quan trọng nhất trên trang — bạn muốn trình duyệt bắt đầu fetch nó càng sớm càng tốt.

Với LCP image, bạn muốn cách tiếp cận ngược lại: `fetchpriority='high'` để nói với trình duyệt ưu tiên image này, và lý tưởng là `<link rel='preload'>` trong `<head>` để trình duyệt không cần parse img tag trước khi bắt đầu download. Lazy loading chỉ nên được áp dụng cho các image below the fold."

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

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "When should you use React.memo and when is it a waste of time?"**

**Model Answer:**
"React.memo wraps a component so it skips re-rendering when its props haven't changed — it does a shallow equality check. It's useful when a parent re-renders frequently and the child's render is expensive, but the child's props don't actually change on every parent render.

The cases where it's a waste of time: components that are cheap to render anyway — you're adding overhead for no gain. Components that almost always receive different props — memo will do the comparison and then re-render anyway. And most importantly: components receiving inline object or array literals as props, or callback functions defined inline in the parent. An inline function like `onClick={() => doSomething()}` creates a new function reference every render, so the shallow comparison fails every time and memo does nothing.

For memo to actually work on a component that receives callbacks, the parent needs to wrap those callbacks in `useCallback`. Same with object props — you'd need `useMemo`. This is why you have to profile before adding memo — it's easy to add it thinking it helps but not actually fix the root cause of re-renders."

**Trả lời (Tiếng Việt):**
"React.memo wrap một component để skip re-render khi props không thay đổi — nó thực hiện shallow equality check. Hữu ích khi parent re-render thường xuyên và render của child tốn kém, nhưng props của child thực sự không thay đổi trên mỗi lần parent render.

Các trường hợp lãng phí thời gian: component render rẻ — bạn đang thêm overhead mà không có lợi ích gì. Component gần như luôn nhận props khác nhau — memo sẽ so sánh rồi re-render lại. Và quan trọng nhất: component nhận inline object hay array literal làm props, hoặc callback function được định nghĩa inline trong parent. Inline function như `onClick={() => doSomething()}` tạo ra function reference mới mỗi lần render, nên shallow comparison fail mỗi lần và memo không có tác dụng gì.

Để memo thực sự hoạt động trên component nhận callback, parent cần wrap các callback đó trong `useCallback`. Tương tự với object props — bạn cần `useMemo`. Đó là lý do bạn phải profile trước khi thêm memo — dễ thêm vào nghĩ rằng nó giúp ích nhưng thực ra không giải quyết nguyên nhân gốc của re-renders."

---

**Q2 (Mid): "Explain the difference between useMemo and useCallback. When would you reach for each?"**

**Model Answer:**
"They're both about caching a value so it's not recreated on every render, but they cache different things.

`useCallback` caches a function. `useCallback(() => doSomething(id), [id])` returns the same function reference across renders as long as `id` doesn't change. You'd use this when passing a callback down to a memoized child component — if the function reference changes every render, memo on the child is useless.

`useMemo` caches the result of calling a function. `useMemo(() => items.filter(...).sort(...), [items, filter])` runs the computation and caches the result. You'd use it for expensive computations that run on every render — filtering and sorting a large list, building a complex data structure, doing heavy math.

Both have a cost — they add overhead for the memoization itself. The rule I follow is: don't add them preemptively. Add useCallback when a child component wrapped in memo is still re-rendering unnecessarily. Add useMemo when a profiler shows an expensive computation is running on renders where its inputs haven't changed."

**Trả lời (Tiếng Việt):**
"Cả hai đều về việc cache một giá trị để nó không bị tạo lại mỗi lần render, nhưng chúng cache những thứ khác nhau.

`useCallback` cache một function. `useCallback(() => doSomething(id), [id])` trả về cùng function reference qua các lần render miễn là `id` không thay đổi. Bạn dùng khi truyền callback xuống cho memoized child component — nếu function reference thay đổi mỗi lần render, memo trên child không có giá trị gì.

`useMemo` cache kết quả của việc gọi một function. `useMemo(() => items.filter(...).sort(...), [items, filter])` chạy computation và cache kết quả. Bạn dùng nó cho các expensive computation chạy trên mỗi lần render — filter và sort danh sách lớn, xây dựng data structure phức tạp, thực hiện heavy math.

Cả hai đều có chi phí — chúng thêm overhead cho chính việc memoization. Rule tôi theo: đừng thêm vào đề phòng. Thêm useCallback khi child component được wrap trong memo vẫn re-render không cần thiết. Thêm useMemo khi profiler cho thấy expensive computation đang chạy trên các lần render mà input của nó không thay đổi."

---

**Q3 (Senior): "You have a React app where a list of 10,000 items renders very slowly. What's your approach?"**

**Model Answer:**
"The first step is always to confirm what's slow with the React DevTools Profiler or Chrome Performance tab. But for 10,000 items, the answer almost certainly involves virtualization.

The fundamental problem is that rendering 10,000 DOM nodes — even simple ones — is inherently slow, and scrolling through that many nodes can drop frame rate because the browser has to composite all of them. Virtualization solves this by only rendering the items that are actually visible in the viewport, plus a small overscan buffer. With react-window or TanStack Virtual, you go from 10,000 DOM nodes to maybe 20 at any given time.

Beyond virtualization: I'd check whether the list items themselves are unnecessarily re-rendering. If a parent state update causes the whole list to re-render, wrapping each item in `React.memo` and ensuring the item data references are stable can help. I'd also check if the items are receiving callbacks — those need `useCallback`.

For images in the list, lazy loading with IntersectionObserver. For the data itself, if it's paginated or has infinite scroll, I'd use TanStack Query which handles caching and background refetching cleanly."

**Trả lời (Tiếng Việt):**
"Bước đầu tiên luôn là xác nhận điều gì chậm bằng React DevTools Profiler hoặc Chrome Performance tab. Nhưng với 10,000 items, câu trả lời hầu như chắc chắn liên quan đến virtualization.

Vấn đề cơ bản là render 10,000 DOM node — kể cả các node đơn giản — vốn đã chậm, và scroll qua nhiều node đó có thể làm giảm frame rate vì trình duyệt phải composite tất cả chúng. Virtualization giải quyết điều này bằng cách chỉ render các items thực sự hiển thị trong viewport, cộng thêm một overscan buffer nhỏ. Với react-window hoặc TanStack Virtual, bạn đi từ 10,000 DOM node xuống còn khoảng 20 vào bất kỳ lúc nào.

Ngoài virtualization: tôi sẽ kiểm tra xem các list item có đang re-render không cần thiết không. Nếu parent state update khiến cả list re-render, wrap mỗi item trong `React.memo` và đảm bảo item data references ổn định có thể giúp ích. Tôi cũng kiểm tra xem items có nhận callback không — những cái đó cần `useCallback`.

Với images trong list, lazy loading bằng IntersectionObserver. Với data, nếu phân trang hoặc infinite scroll, tôi dùng TanStack Query xử lý caching và background refetching gọn gàng."

---

**Q4 (Senior): "Walk me through how you'd use the React Profiler to find a performance problem."**

**Model Answer:**
"I'd open React DevTools, go to the Profiler tab, click Record, interact with the part of the app that's slow — maybe clicking a button or typing in a search box — then stop the recording.

The Profiler gives me a Flamegraph view. Each bar represents a component render. The width represents time. Grey components didn't render at all. I'm looking for bars that are wide — those are components that took a long time to render — and for components that I didn't expect to render at all.

I'd switch to the Ranked view which sorts components by how long they spent rendering, so the worst offenders are at the top. I can click on any component to see why it rendered — the Profiler shows whether it was props, state, context, or hooks that changed.

If I see a component re-rendering with the same props every time, that's a signal to add `React.memo`. If I see a parent re-rendering hundreds of child components unnecessarily, that's a context or state architecture issue — the state might be too high up in the tree.

For deep investigation I'll also add `@welldone-software/why-did-you-render` in development — it logs to the console every time a component re-renders with the same props, which is almost always a bug."

**Trả lời (Tiếng Việt):**
"Tôi sẽ mở React DevTools, vào tab Profiler, click Record, tương tác với phần app đang chậm — có thể click button hoặc gõ vào search box — rồi dừng recording.

Profiler cung cấp Flamegraph view. Mỗi bar đại diện cho một component render. Chiều rộng đại diện cho thời gian. Component màu xám không render. Tôi đang tìm các bar rộng — đó là component mất nhiều thời gian render — và các component mà tôi không ngờ sẽ render.

Tôi sẽ chuyển sang Ranked view sắp xếp component theo thời gian render, để những thứ tệ nhất nằm trên cùng. Tôi có thể click vào bất kỳ component nào để xem tại sao nó render — Profiler hiển thị props, state, context, hay hook nào thay đổi.

Nếu thấy component re-render với cùng props mỗi lần, đó là tín hiệu thêm `React.memo`. Nếu thấy parent re-render hàng trăm child component không cần thiết, đó là vấn đề context hoặc state architecture — state có thể đang ở quá cao trong tree.

Để điều tra sâu hơn tôi cũng sẽ thêm `@welldone-software/why-did-you-render` trong development — nó log vào console mỗi lần component re-render với cùng props, gần như luôn là một bug."

---

**Q5 (Senior): "When should you use virtualization, and what are its tradeoffs?"**

**Model Answer:**
"Virtualization makes sense when you have a long list or grid where rendering all items simultaneously would be slow — typically anywhere above a few hundred items, but it depends on item complexity.

The tradeoffs are real though. First, accessibility can be harder — screen readers expect to navigate through all list items, but virtualized lists only have items in the DOM that are visible, so keyboard navigation and 'find on page' may not work without extra effort. Second, the implementation adds complexity — you need a scrollable container with fixed height, and items need predictable heights or you need to handle variable heights which is trickier. Third, some CSS tricks break — for example, you can't easily do `position: sticky` inside a virtualized list without special handling.

For most cases, libraries like `@tanstack/react-virtual` handle the hard parts. But for something like a short paginated list of 20 items per page, virtualization is overkill and I'd just paginate.

The alternative worth considering for read-only long documents is `content-visibility: auto` in CSS — the browser natively skips rendering off-screen sections without any JavaScript, though it doesn't give you the same degree of control as a virtualization library."

**Trả lời (Tiếng Việt):**
"Virtualization có ý nghĩa khi bạn có danh sách hoặc grid dài mà render tất cả items đồng thời sẽ chậm — thường là trên vài trăm items, nhưng phụ thuộc vào độ phức tạp của item.

Nhưng tradeoff là thực tế. Thứ nhất, accessibility có thể khó hơn — screen reader mong đợi điều hướng qua tất cả list items, nhưng virtualized list chỉ có items trong DOM mà đang hiển thị, nên keyboard navigation và 'find on page' có thể không hoạt động nếu không có nỗ lực thêm. Thứ hai, implementation thêm độ phức tạp — bạn cần scrollable container với chiều cao cố định, và items cần chiều cao có thể đoán trước, hoặc bạn phải xử lý variable heights phức tạp hơn. Thứ ba, một số CSS trick bị vỡ — ví dụ, không thể dễ dàng dùng `position: sticky` bên trong virtualized list mà không có xử lý đặc biệt.

Với hầu hết trường hợp, library như `@tanstack/react-virtual` xử lý phần khó. Nhưng với danh sách phân trang ngắn 20 items mỗi trang, virtualization là quá mức và tôi chỉ cần paginate.

Giải pháp thay thế đáng cân nhắc cho document dài chỉ đọc là `content-visibility: auto` trong CSS — trình duyệt natively bỏ qua render các section ngoài màn hình mà không cần JavaScript, dù không cho bạn cùng mức control như virtualization library."

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
