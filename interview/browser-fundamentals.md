# Browser Fundamentals & Web Rendering — Interview Questions (Junior → Senior)

> Tập trung vào CƠ CHẾ hoạt động bên trong, không chỉ là định nghĩa.
> Dành cho Senior FE Engineer — feedback: "hiểu rõ cơ chế hoạt động FE, HTML, CSS, JS, hiểu cách web render hoạt động như thế nào"

---

## Table of Contents

1. [Critical Rendering Path](#1-critical-rendering-path)
2. [Reflow vs Repaint vs Composite](#2-reflow-vs-repaint-vs-composite)
3. [script defer vs async vs normal](#3-script-defer-vs-async-vs-normal)
4. [DOM và CSSOM](#4-dom-và-cssom)
5. [Event System — Bubbling, Capturing, Delegation](#5-event-system--bubbling-capturing-delegation)
6. [CSS Cascade, Specificity, Inheritance](#6-css-cascade-specificity-inheritance)
7. [Box Model](#7-box-model)
8. [Stacking Context và z-index](#8-stacking-context-và-z-index)
9. [JavaScript Engine — V8](#9-javascript-engine--v8)
10. [Event Loop (deep dive)](#10-event-loop-deep-dive)
11. [Browser Storage](#11-browser-storage)
12. [HTTP và Network](#12-http-và-network)
13. [Web APIs — RAF, IntersectionObserver, MutationObserver, ResizeObserver](#13-web-apis--raf-intersectionobserver-mutationobserver-resizeobserver)
14. [HTML Semantics và Accessibility](#14-html-semantics-và-accessibility)
15. [Browser Rendering Optimization — Practical](#15-browser-rendering-optimization--practical)

---

## 1. Critical Rendering Path

### Q1.1
**Level: [Junior]**
**EN:** What happens step by step when a browser receives HTML from a server?
**VI:** Điều gì xảy ra từng bước khi trình duyệt nhận được HTML từ server?

**Trả lời:**

Critical Rendering Path (CRP) là chuỗi các bước mà browser thực hiện để chuyển đổi HTML, CSS, JS thành pixels trên màn hình.

```
Bytes → Characters → Tokens → Nodes → DOM
```

**Quy trình chi tiết:**

```
SERVER                          BROWSER
  |                               |
  |--- HTML bytes --------------->|
  |                          [Tokenizer]
  |                         <html> token
  |                         <head> token
  |                         <body> token
  |                               |
  |                          [Tree Builder]
  |                         DOM nodes được tạo
  |                         incrementally
  |                               |
  |--- CSS bytes (link) -------->|
  |                          [CSS Parser]
  |                         CSSOM construction
  |                         (BLOCKS rendering!)
  |                               |
  |                    [Render Tree = DOM + CSSOM]
  |                               |
  |                          [Layout/Reflow]
  |                         Calculate geometry
  |                               |
  |                           [Paint]
  |                         Fill pixels
  |                               |
  |                         [Composite]
  |                         GPU layer merge
  |                               |
  |                          [Screen!]
```

**5 bước chính:**

1. **Bytes → DOM**: Browser download HTML bytes → decode thành characters → tokenize (lexer) → parse thành DOM nodes. Quá trình này **incremental** — browser bắt đầu render DOM ngay khi nhận được từng chunk, không cần đợi toàn bộ HTML.

2. **CSS → CSSOM**: Tương tự HTML nhưng **không incremental** — CSSOM phải được xây dựng hoàn toàn trước khi render vì CSS có cascade (rule sau có thể override rule trước).

3. **Render Tree**: Kết hợp DOM + CSSOM. Chỉ bao gồm các node **visible** — `display: none` bị loại, nhưng `visibility: hidden` vẫn có trong Render Tree. Pseudo-elements như `::before`, `::after` được thêm vào Render Tree dù không có trong DOM.

4. **Layout (Reflow)**: Tính toán kích thước và vị trí chính xác của mỗi element. Output là "box model" của mỗi node. Đây là bước tốn kém nhất.

5. **Paint → Composite**: Paint điền pixels cho mỗi layer. Composite gộp các layer lại bằng GPU.

---

### Q1.2
**Level: [Mid]**
**EN:** Why does CSS block rendering but HTML parsing doesn't? Explain the mechanism.
**VI:** Tại sao CSS block rendering nhưng HTML parsing thì không? Giải thích cơ chế.

**Trả lời:**

**HTML không block rendering vì:**
- DOM được xây dựng **incrementally** — mỗi token được parse xong là thêm vào DOM ngay
- Browser có thể render "partial DOM" — người dùng thấy content dần dần
- Đây là thiết kế có chủ ý để cải thiện perceived performance

**CSS block rendering vì:**
- CSSOM phải **hoàn chỉnh 100%** trước khi build Render Tree
- Lý do: CSS có tính **cascade** — rule ở cuối file có thể override rule ở đầu
- Nếu render với CSSOM chưa đầy đủ → FOUC (Flash Of Unstyled Content)
- Browser chọn delay render hơn là show unstyled content

```
HTML Parsing (non-blocking):
Time: 0ms ----[chunk1]----[chunk2]----[chunk3]----
DOM:          [partial]  [partial]   [complete]
Render:       [starts]   [updates]   [final]

CSS Parsing (blocking):
Time: 0ms ----[downloading CSS]----[parsing CSS]----
CSSOM:                                             [complete]
Render:                                            [STARTS HERE]
```

**CSS cũng block JavaScript execution:**
```html
<link rel="stylesheet" href="style.css">  <!-- CSS đang download -->
<script>
  // Script này KHÔNG chạy cho đến khi style.css tải xong
  // Vì script có thể query CSSOM (getComputedStyle)
  document.querySelector('.box').style.color; // cần CSSOM
</script>
```

---

### Q1.3
**Level: [Mid]**
**EN:** How does JavaScript block HTML parsing? What do defer and async solve?
**VI:** Làm thế nào JavaScript block HTML parsing? defer và async giải quyết vấn đề gì?

**Trả lời:**

Script thông thường (`<script src="app.js">`) **block parser** vì:
1. Parser gặp `<script>` tag → dừng lại
2. Browser download script file
3. JavaScript engine thực thi script (có thể modify DOM)
4. Chỉ sau khi script chạy xong, parser mới tiếp tục

Lý do: JS có thể gọi `document.write()` — ghi nội dung mới vào document, thay đổi DOM đang được parse. Nếu parse tiếp trong khi JS chạy sẽ tạo race condition.

```
Normal script (blocking):
HTML: =====[STOP]====================[resume]======>
JS:              [download][execute]

defer:
HTML: =======================================[done]>
JS:        [download----------][execute after DOM]

async:
HTML: =======[STOP]=============[resume]===========>
JS:        [download--][execute]
```

---

## 2. Reflow vs Repaint vs Composite

### Q2.1
**Level: [Junior]**
**EN:** What is the difference between Reflow, Repaint, and Composite? Which is most expensive?
**VI:** Sự khác biệt giữa Reflow, Repaint và Composite là gì? Cái nào tốn kém nhất?

**Trả lời:**

```
PIPELINE: Layout → Paint → Composite

Reflow  → Paint  → Composite   (most expensive — rebuild everything downstream)
          Paint  → Composite   (Repaint — skip layout)
                   Composite   (only composite — cheapest, GPU only)
```

| Bước | Tên khác | Trigger bởi | Chi phí |
|------|----------|-------------|---------|
| Layout | Reflow | Thay đổi geometry | Rất cao |
| Paint | Repaint | Thay đổi visual (không layout) | Trung bình |
| Composite | - | transform, opacity | Thấp (GPU) |

**Các CSS property trigger từng bước:**

| Property | Reflow | Repaint | Composite only |
|----------|--------|---------|---------------|
| width, height | YES | YES | NO |
| top, left, margin | YES | YES | NO |
| padding, border | YES | YES | NO |
| background-color | NO | YES | NO |
| color | NO | YES | NO |
| box-shadow | NO | YES | NO |
| transform | NO | NO | YES |
| opacity | NO | NO | YES |
| filter | NO | NO | YES* |

*filter tạo stacking context nhưng browser modern thường composite-only

**Tại sao `transform` chỉ trigger composite?**
- `transform` không thay đổi flow của document
- Browser promote element lên **own compositing layer** (GPU texture)
- GPU apply matrix transform mà không cần CPU recalculate layout
- `opacity` tương tự — GPU thay đổi alpha của texture

**Tại sao `top/left` trigger reflow?**
- `top/left` trên `position: absolute/relative` thay đổi geometry
- Browser phải recalculate layout của element đó và potentially các siblings
- Sau layout xong → paint lại → composite

---

### Q2.2
**Level: [Mid]**
**EN:** Explain will-change: transform — what does it do internally, and what are its tradeoffs?
**VI:** Giải thích will-change: transform hoạt động bên trong như thế nào, và tradeoffs của nó?

**Trả lời:**

`will-change: transform` là hint cho browser biết element này **sắp** được transform. Browser sẽ:

1. **Promote element lên composite layer ngay** (không đợi animation bắt đầu)
2. Upload element như một GPU texture trước
3. Khi animation xảy ra, GPU xử lý — không cần CPU/main thread

```css
/* KHÔNG có will-change */
.box {
  transition: transform 0.3s; /* browser promote ON DEMAND khi transition bắt đầu */
}
/* Có thể có lag ở frame đầu tiên */

/* CÓ will-change */
.box {
  will-change: transform; /* browser promote NGAY BÂY GIỜ */
  transition: transform 0.3s; /* GPU sẵn sàng, không lag */
}
```

**Tradeoffs:**

| Pro | Con |
|-----|-----|
| Eliminate jank ở animation start | Memory tăng (mỗi layer = GPU texture = VRAM) |
| GPU animation smooth | Tạo stacking context mới (z-index issues) |
| Off-main-thread | Nếu dùng quá nhiều → GPU memory pressure |

**Khi nào dùng:**
```css
/* GOOD: element sẽ animate */
.modal { will-change: transform; }

/* BAD: dùng cho toàn trang */
* { will-change: transform; } /* memory disaster */

/* GOOD: add/remove via JS khi cần */
element.addEventListener('mouseenter', () => {
  element.style.willChange = 'transform';
});
element.addEventListener('animationend', () => {
  element.style.willChange = 'auto'; /* release GPU memory */
});
```

**`transform: translateZ(0)` hack:**
Đây là cách cũ trước khi có `will-change`. Buộc browser tạo composite layer bằng cách apply 3D transform không có tác dụng thực sự. Cách hiện đại là dùng `will-change`.

---

### Q2.3
**Level: [Mid]**
**EN:** Why should you use requestAnimationFrame instead of setTimeout for animations?
**VI:** Tại sao nên dùng requestAnimationFrame thay vì setTimeout cho animations?

**Trả lời:**

```javascript
// BAD: setTimeout
function animate() {
  element.style.left = (parseInt(element.style.left) + 1) + 'px';
  setTimeout(animate, 16); // 16ms ≈ 60fps
}
// Vấn đề:
// 1. Timer không sync với display refresh rate
// 2. Tab ẩn vẫn chạy → lãng phí CPU
// 3. Nếu main thread bận, timer bị delay → jank
// 4. Gây reflow vì đọc/ghi layout properties

// GOOD: requestAnimationFrame
function animate() {
  element.style.transform = `translateX(${position}px)`;
  position++;
  requestAnimationFrame(animate); // browser tự schedule
}
requestAnimationFrame(animate);
// Ưu điểm:
// 1. Sync với display refresh (60Hz = 16.7ms, 120Hz = 8.3ms)
// 2. Tab ẩn → browser pause RAF callbacks (tiết kiệm CPU/battery)
// 3. Browser có thể batch RAF callbacks trong cùng frame
// 4. Chạy trước paint → changes được apply đúng frame
```

**RAF trong Event Loop:**
```
Event Loop iteration:
1. [Macrotask] (setTimeout, I/O, etc.)
2. [All Microtasks] (Promise.then)
3. [RAF callbacks] ← đây
4. [Layout]
5. [Paint]
6. [Composite]
```

RAF callbacks chạy **sau microtasks nhưng trước paint** — đây là window lý tưởng để thay đổi DOM/styles vì browser sẽ batch chúng vào frame tiếp theo.

---

### Q2.4
**Level: [Senior]**
**EN:** How do you use Chrome DevTools to identify reflow/repaint bottlenecks?
**VI:** Làm thế nào dùng Chrome DevTools để xác định reflow/repaint bottlenecks?

**Trả lời:**

**Performance Tab:**
1. Record → interact với page → Stop
2. Tìm màu **purple** (Layout/Reflow) và **green** (Paint) trong flame chart
3. Long purple bars = reflow đang chạy lâu → cần optimize
4. "Layout Shift" events → CLS issues

**Rendering Panel** (More Tools → Rendering):
- `Paint flashing`: highlight vùng đang được repaint (màu xanh lá) — xem có repaint quá nhiều không
- `Layout Shift Regions`: highlight CLS
- `FPS Meter`: xem frame rate real-time

**Layers Panel** (More Tools → Layers):
- Xem tất cả composite layers
- Highlight từng layer để xem lý do nó được promote
- Memory usage của mỗi layer

**Performance Monitor** (More Tools → Performance Monitor):
- CPU Usage, Layout/s, Style Recalcs/s — real-time metrics

```javascript
// Trong code: đánh dấu để dễ tìm trong DevTools
performance.mark('animation-start');
// ... animation code ...
performance.mark('animation-end');
performance.measure('my-animation', 'animation-start', 'animation-end');
```

---

## 3. script defer vs async vs normal

### Q3.1
**Level: [Junior]**
**EN:** What is the difference between `<script>`, `<script defer>`, and `<script async>`?
**VI:** Sự khác biệt giữa `<script>`, `<script defer>`, và `<script async>` là gì?

**Trả lời:**

**Timeline diagrams:**

```
Legend: H = HTML parsing, D = Script download, E = Script execute
        | = pause/block

NORMAL <script src="app.js">:
H H H H | D D D D D | E E E | H H H H H H
         ^-- parser stops     ^-- parser resumes
Browser hoàn toàn block. Tệ nhất cho performance.

DEFER <script defer src="app.js">:
H H H H H H H H H H H H H H | D D D D D | E E E
          [download parallel]             ^-- after DOM parsed
D D D D D (download starts immediately, parallel with parsing)
Scripts thực thi THEO THỨ TỰ sau khi DOM parsed xong.

ASYNC <script async src="app.js">:
H H H H H H | D D D | E E E | H H H H H H
              [download parallel]
                              ^-- execute ngay khi download xong
Thứ tự KHÔNG đảm bảo. Có thể block parser khi execute.
```

**So sánh chi tiết:**

| | Normal | defer | async |
|---|--------|-------|-------|
| Download | Chặn parsing | Song song | Song song |
| Execute khi | Download xong | DOM parsed xong | Download xong |
| Thứ tự | Đúng thứ tự | Đúng thứ tự | Không đảm bảo |
| Block parsing | Có | Không | Chỉ khi execute |
| DOMContentLoaded | Trước | Sau (scripts chạy trước DCL) | Không ảnh hưởng |

**type="module":**
```html
<script type="module" src="app.js"></script>
```
- **defer by default** — không block parsing
- Chạy trong strict mode
- Có module scope (không leak global)
- Chỉ execute một lần dù include nhiều lần

**Khi nào dùng gì:**
```html
<!-- Scripts cần DOM, có dependencies, ở cuối <head> -->
<script defer src="jquery.js"></script>
<script defer src="app.js"></script>  <!-- jQuery chạy trước đảm bảo -->

<!-- Scripts độc lập, không cần DOM, không có dependencies -->
<script async src="analytics.js"></script>
<script async src="ads.js"></script>

<!-- Modern JS modules -->
<script type="module" src="app.js"></script>
```

---

## 4. DOM và CSSOM

### Q4.1
**Level: [Junior]**
**EN:** What is the DOM? What types of Nodes exist in the DOM tree?
**VI:** DOM là gì? Có những loại Node nào trong DOM tree?

**Trả lời:**

**DOM (Document Object Model)** là representation dạng cây (tree) của HTML document trong memory. Browser tạo DOM từ HTML và cho phép JavaScript tương tác.

**DOM Tree Structure:**
```
Document
├── DocumentType (<!DOCTYPE html>)
└── Element: <html>
    ├── Element: <head>
    │   ├── Element: <title>
    │   │   └── Text: "My Page"
    │   └── Element: <link>
    └── Element: <body>
        ├── Element: <h1>
        │   └── Text: "Hello"
        ├── Comment: <!-- nav comment -->
        └── Element: <p>
            ├── Text: "Hello "
            ├── Element: <strong>
            │   └── Text: "World"
            └── Text: "!"
```

**Node Types (nodeType value):**

| nodeType | Tên | Ví dụ |
|----------|-----|-------|
| 1 | Element | `<div>`, `<p>`, `<span>` |
| 3 | Text | Text content bên trong tag |
| 8 | Comment | `<!-- comment -->` |
| 9 | Document | `document` object |
| 10 | DocumentType | `<!DOCTYPE html>` |
| 11 | DocumentFragment | Fragment container |

```javascript
const el = document.querySelector('p');
console.log(el.nodeType);        // 1
console.log(el.firstChild.nodeType); // 3 (text node)
console.log(el.nodeName);        // "P"
console.log(el.nodeValue);       // null (elements have no nodeValue)
```

---

### Q4.2
**Level: [Mid]**
**EN:** What is the CSSOM and why does it block rendering?
**VI:** CSSOM là gì và tại sao nó block rendering?

**Trả lời:**

**CSSOM (CSS Object Model)** là tree structure biểu diễn tất cả CSS styles, được build song song với DOM nhưng theo cơ chế khác.

**CSSOM Tree:**
```css
body { font-size: 16px; color: black; }
p    { font-size: 14px; }
.blue { color: blue; }
```

```
CSSOM:
body
├── font-size: 16px
├── color: black
└── p (inherits + overrides)
    ├── font-size: 14px  (override)
    └── color: black     (inherited)
        └── .blue (when class applied)
            └── color: blue (override)
```

**Tại sao CSSOM phải complete trước khi render:**

CSS cascade có nghĩa là một rule cuối file có thể override rule đầu file:

```css
/* Line 1 */ p { color: red; }
/* Line 100 */ p { color: blue; }  /* override! */
```

Nếu browser render sau khi parse được đến line 1, user thấy đỏ → rồi thấy xanh = FOUC.

Browser quyết định: thà delay render còn hơn show unstyled/wrongly-styled content.

**CSSOM + DOM = Render Tree:**
```
DOM Tree          CSSOM Tree
[html]            [html styles]
  [head]            [body: font-size:16px]
  [body]              [p: color:blue]
    [p]      +              =    Render Tree
    [div]                        [body]
                                   [p] (visible, with styles)
                                   [div] (visible, with styles)
                                 * display:none nodes EXCLUDED
                                 * ::before/::after INCLUDED
                                 * <head>, <script> EXCLUDED
```

---

### Q4.3
**Level: [Mid]**
**EN:** Why is direct DOM manipulation expensive? How does DocumentFragment help?
**VI:** Tại sao DOM manipulation trực tiếp tốn kém? DocumentFragment giúp gì?

**Trả lời:**

**Vấn đề với DOM manipulation trực tiếp:**

Mỗi lần thay đổi DOM có thể trigger:
- Style recalculation (CSSOM update)
- Layout (reflow)
- Paint (repaint)
- Composite

```javascript
// BAD: Mỗi appendChild trigger potential reflow
const list = document.getElementById('list');
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  list.appendChild(li); // 1000 reflows!
}

// GOOD: DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li); // append vào fragment (off-DOM, no reflow)
}
list.appendChild(fragment); // 1 reflow duy nhất!
```

**DocumentFragment là "off-DOM container":**
- Tồn tại trong memory, không phải trong DOM tree
- Thao tác với fragment không trigger reflow/repaint
- Khi append fragment vào DOM → chỉ fragment's children được insert (không phải fragment itself)
- Sau khi append, fragment trở nên empty

**Batch DOM reads và writes:**
```javascript
// BAD: Read/Write xen kẽ → nhiều layout recalculations (layout thrashing)
elements.forEach(el => {
  const height = el.offsetHeight;  // READ → trigger layout
  el.style.height = height + 10 + 'px'; // WRITE → invalidate layout
  // vòng lặp tiếp theo READ → trigger layout AGAIN
});

// GOOD: Batch reads trước, writes sau
const heights = elements.map(el => el.offsetHeight); // all READs
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // all WRITEs
}); // chỉ 1 layout recalculation
```

---

### Q4.4
**Level: [Senior]**
**EN:** How does Virtual DOM work and what problem does it solve?
**VI:** Virtual DOM hoạt động như thế nào và nó giải quyết vấn đề gì?

**Trả lời:**

**Virtual DOM (VDOM)** là một lightweight JavaScript representation của DOM tree thực. React, Vue sử dụng VDOM để minimize direct DOM manipulation.

**Quy trình:**

```
State Change
     |
     v
New VDOM tree (cheap — just JS objects)
     |
     v
Diff với Old VDOM (reconciliation algorithm)
     |
     v
Calculate minimal DOM operations needed
     |
     v
Apply ONLY those changes to real DOM (batched)
```

**VDOM là JS object đơn giản:**
```javascript
// Thay vì:
<div class="box" id="main">
  <p>Hello</p>
</div>

// VDOM là:
{
  type: 'div',
  props: { className: 'box', id: 'main' },
  children: [
    {
      type: 'p',
      props: {},
      children: ['Hello']
    }
  ]
}
```

**Diffing Algorithm (React's reconciliation):**
```
Old VDOM:          New VDOM:
<ul>               <ul>
  <li>A</li>         <li>A</li>  ← same, skip
  <li>B</li>         <li>C</li>  ← changed, update text
  <li>C</li>         <li>D</li>  ← changed, update text
</ul>              </ul>

With key prop:
<li key="a">A</li>   → matched by key, no change
<li key="b">B</li>   → not found in new tree → DELETE
<li key="c">C</li>   → matched, no change
                     → <li key="d">D</li> is new → INSERT
```

**Tại sao VDOM nhanh hơn naive DOM manipulation:**
- Tính toán diff trong JS (cheap) → chỉ apply changes tối thiểu vào DOM (expensive)
- Batching: nhiều state changes → một DOM update
- Tuy nhiên: VDOM không phải lúc nào cũng nhanh hơn manual fine-grained updates (Svelte, Solid.js compile-away VDOM)

---

## 5. Event System — Bubbling, Capturing, Delegation

### Q5.1
**Level: [Junior]**
**EN:** What are the three phases of event propagation?
**VI:** Ba giai đoạn của event propagation là gì?

**Trả lời:**

```
Event propagation cho click vào <button>:

    document
        |
    [1. CAPTURING PHASE] — từ trên xuống
        |
      <html>
        |
      <body>
        |
      <div>
        |
     <button>  ← [2. TARGET PHASE] — tại element
        |
      <div>
        |
      <body>
        |
      <html>
        |
    document
    [3. BUBBLING PHASE] — từ dưới lên
```

**Ví dụ thực tế:**

```html
<div id="parent">
  <button id="child">Click me</button>
</div>
```

```javascript
document.getElementById('parent').addEventListener('click', (e) => {
  console.log('parent - bubbling');
}, false); // false = bubbling phase (default)

document.getElementById('parent').addEventListener('click', (e) => {
  console.log('parent - capturing');
}, true); // true = capturing phase

document.getElementById('child').addEventListener('click', (e) => {
  console.log('child - target');
});

// Khi click button, output:
// "parent - capturing"  ← capturing xảy ra trước
// "child - target"
// "parent - bubbling"
```

**Các method control propagation:**

| Method | Tác dụng |
|--------|----------|
| `e.stopPropagation()` | Dừng propagation, nhưng các listener khác cùng element vẫn chạy |
| `e.stopImmediatePropagation()` | Dừng propagation VÀ dừng tất cả listener còn lại trên element hiện tại |
| `e.preventDefault()` | Ngăn default browser behavior (form submit, link navigate) |

```javascript
// stopPropagation vs stopImmediatePropagation
button.addEventListener('click', (e) => {
  e.stopPropagation(); // parent không nhận event
  console.log('listener 1'); // VẪN chạy
});
button.addEventListener('click', (e) => {
  console.log('listener 2'); // VẪN chạy (vì cùng element)
});

// stopImmediatePropagation:
button.addEventListener('click', (e) => {
  e.stopImmediatePropagation();
  console.log('listener 1'); // chạy
});
button.addEventListener('click', (e) => {
  console.log('listener 2'); // KHÔNG chạy
});
```

---

### Q5.2
**Level: [Mid]**
**EN:** What is event delegation and why is it more performant for large lists?
**VI:** Event delegation là gì và tại sao nó hiệu quả hơn cho large lists?

**Trả lời:**

**Event delegation** là pattern: thay vì gán listener cho mỗi child element, gán **một listener duy nhất** cho parent và dùng bubbling để handle events.

```javascript
// BAD: 1000 listeners cho 1000 items
const items = document.querySelectorAll('.list-item');
items.forEach(item => {
  item.addEventListener('click', handleClick); // 1000 listeners!
});

// GOOD: Event delegation — 1 listener
document.getElementById('list').addEventListener('click', (e) => {
  const item = e.target.closest('.list-item');
  if (item) {
    handleClick(item);
  }
});
```

**Tại sao delegation performant hơn:**

1. **Memory**: 1 listener vs N listeners — tiết kiệm memory đáng kể
2. **Dynamic elements**: Items thêm vào sau không cần attach listener mới — parent đã handle
3. **Faster initialization**: Không cần loop qua 1000 items để attach listeners

```javascript
// Dynamic elements — delegation xử lý tự động
const list = document.getElementById('list');

list.addEventListener('click', (e) => {
  if (e.target.matches('.list-item')) {
    console.log('clicked:', e.target.textContent);
  }
});

// Thêm item mới — KHÔNG cần attach listener
const newItem = document.createElement('div');
newItem.className = 'list-item';
newItem.textContent = 'New Item';
list.appendChild(newItem); // delegation tự handle click này
```

**e.target vs e.currentTarget:**
```javascript
list.addEventListener('click', (e) => {
  console.log(e.target);         // Element người dùng thực sự click
  console.log(e.currentTarget);  // Element có listener (list trong trường hợp này)
});
```

---

### Q5.3
**Level: [Mid]**
**EN:** What does the `passive` option in addEventListener do? When should you use it?
**VI:** Option `passive` trong addEventListener làm gì? Khi nào nên dùng?

**Trả lời:**

`passive: true` báo cho browser biết listener **sẽ không gọi** `preventDefault()`. Browser có thể dùng thông tin này để **optimize scroll performance**.

**Vấn đề không có passive:**
```
User scrolls (touch event)
     |
     v
Browser muốn scroll ngay
     |
     v
Browser phải CHỜ touchstart/touchmove listener chạy xong
     |
     v
Check: listener có gọi preventDefault() không?
     |
     v
Chỉ sau đó mới scroll
→ Scroll lag lên đến 200ms!
```

**Với passive: true:**
```
User scrolls (touch event)
     |
     v
Browser scroll NGAY (vì biết không có preventDefault)
     |          |
     v          v
Listener chạy  Scroll diễn ra
(parallel)     (no wait)
→ Smooth scroll!
```

```javascript
// Scroll handler — không cần preventDefault
window.addEventListener('scroll', handleScroll, { passive: true });

// Touch handler — smooth scrolling
element.addEventListener('touchstart', handleTouch, { passive: true });
element.addEventListener('touchmove', handleMove, { passive: true });

// Khi KHÔNG dùng passive (cần preventDefault):
element.addEventListener('touchstart', (e) => {
  e.preventDefault(); // Ngăn scroll — custom scroll behavior
  // Không thể dùng passive: true ở đây
});
```

**Các options của addEventListener:**
```javascript
element.addEventListener('click', handler, {
  capture: false,  // false = bubbling phase (default)
  once: true,      // listener tự remove sau lần đầu tiên fire
  passive: true,   // không gọi preventDefault — optimize scroll
  signal: abortController.signal // AbortController để remove listener
});

// once: true thay thế pattern cũ:
function handleOnce(e) {
  doSomething();
  element.removeEventListener('click', handleOnce);
}
element.addEventListener('click', handleOnce);
```

---

## 6. CSS Cascade, Specificity, Inheritance

### Q6.1
**Level: [Junior]**
**EN:** How does CSS specificity work? How do you calculate it?
**VI:** CSS specificity hoạt động như thế nào? Tính toán nó ra sao?

**Trả lời:**

**Specificity** là cơ chế browser dùng để quyết định CSS rule nào "thắng" khi có conflict. Được biểu diễn dạng vector **(a, b, c, d)**:

```
(a, b, c, d)
 |  |  |  |
 |  |  |  └── d: element selectors và pseudo-elements (::before)
 |  |  └───── c: class, attribute, pseudo-class selectors
 |  └──────── b: ID selectors
 └─────────── a: inline styles
```

**Tính toán ví dụ:**

| Selector | Inline | IDs | Classes/Attrs | Elements | Score |
|----------|--------|-----|---------------|----------|-------|
| `p` | 0 | 0 | 0 | 1 | (0,0,0,1) |
| `.nav-item` | 0 | 0 | 1 | 0 | (0,0,1,0) |
| `#nav` | 0 | 1 | 0 | 0 | (0,1,0,0) |
| `#nav .item` | 0 | 1 | 1 | 0 | (0,1,1,0) |
| `#nav .item a` | 0 | 1 | 1 | 1 | (0,1,1,1) |
| `style="color:red"` | 1 | 0 | 0 | 0 | (1,0,0,0) |
| `!important` | — | — | — | — | Beats all |

**So sánh:**
```css
/* (0,0,0,1) vs (0,0,1,0): class wins */
p          { color: red; }
.highlight { color: blue; } /* WINS */

/* (0,1,0,0) vs (0,0,10,0): ID wins, bất kể bao nhiêu classes */
#main      { color: red; }  /* WINS */
.a.b.c.d.e.f.g.h.i.j { color: blue; } /* 10 classes nhưng vẫn thua */
```

**Cascade Order (khi specificity bằng nhau):**
```
1. User agent styles (browser defaults)
2. User styles (browser settings)
3. Author styles (your CSS)
4. Author !important
5. User !important
6. User agent !important
```

**Trong cùng author styles, khi specificity bằng nhau → rule sau thắng (source order):**
```css
.box { color: red; }
.box { color: blue; } /* WINS — same specificity, comes later */
```

---

### Q6.2
**Level: [Mid]**
**EN:** What is the specificity of :is(), :where(), and :has()?
**VI:** :is(), :where(), và :has() có specificity như thế nào?

**Trả lời:**

**`:is()`** — specificity = specificity của selector mạnh nhất trong list:
```css
:is(#id, .class, div) p { color: red; }
/* specificity = (0,1,0,1) — do #id là mạnh nhất trong :is() list */

/* Useful để viết gọn mà không mất specificity */
:is(h1, h2, h3) > span { color: blue; }
/* thay cho: */
h1 > span, h2 > span, h3 > span { color: blue; }
```

**`:where()`** — specificity = ZERO (0,0,0,0) luôn luôn:
```css
:where(#id, .class, div) p { color: red; }
/* specificity = (0,0,0,1) — :where() đóng góp 0 */

/* Dùng cho design system/reset styles — dễ override */
:where(h1, h2, h3) { margin: 0; } /* specificity 0 — dễ override */
```

**`:has()`** — specificity = specificity của selector bên trong:
```css
div:has(> p.highlight) { border: 1px solid blue; }
/* specificity = (0,0,1,1) — .highlight (0,0,1,0) + div (0,0,0,1) */

/* Use cases: parent selector! */
.card:has(.badge) { padding-right: 2rem; } /* card có badge → thêm padding */
form:has(:invalid) { border-color: red; }  /* form có field invalid */
```

**`:not()`** — specificity = specificity của selector trong :not():
```css
div:not(.special) { color: red; }
/* specificity: div (0,0,0,1) + .special (0,0,1,0) = (0,0,1,1) */
```

---

### Q6.3
**Level: [Mid]**
**EN:** Which CSS properties are inherited by default? How do CSS custom properties (variables) behave with inheritance?
**VI:** Những CSS property nào được inherit mặc định? CSS custom properties hoạt động thế nào với inheritance?

**Trả lời:**

**Inherited properties** (truyền từ parent xuống children tự động):
- Typography: `font-family`, `font-size`, `font-weight`, `font-style`, `font-variant`
- Text: `color`, `text-align`, `text-indent`, `line-height`, `letter-spacing`, `word-spacing`
- List: `list-style`, `list-style-type`
- Table: `border-collapse`, `border-spacing`
- Misc: `cursor`, `visibility`, `direction`

**Non-inherited properties** (không tự truyền):
- Box model: `margin`, `padding`, `border`, `width`, `height`
- Positioning: `position`, `top`, `left`, `right`, `bottom`, `z-index`
- Background: `background-*`
- Flex/Grid: `display`, `flex-*`, `grid-*`

```css
/* Override inheritance */
.child {
  color: inherit;    /* bắt buộc inherit dù non-inherited */
  color: initial;    /* reset về browser default */
  color: unset;      /* inherited prop: inherit; non-inherited: initial */
  color: revert;     /* reset về user-agent stylesheet */
}
```

**CSS Custom Properties và Inheritance:**

CSS variables (`--variable-name`) **LUÔN LUÔN inherited** theo mặc định:

```css
:root {
  --primary-color: blue;
  --font-size: 16px;
}

.parent {
  --button-color: red; /* chỉ available trong .parent và descendants */
}

.child {
  color: var(--primary-color); /* blue — inherited từ :root */
  color: var(--button-color);  /* red — inherited từ .parent */
}

/* Override tại component level */
.dark-theme {
  --primary-color: lightblue; /* override cho subtree này */
}
```

**CSS variables vs preprocessor variables (SASS $var):**
```scss
/* SASS — compile time, không có inheritance */
$color: blue;
.component { color: $color; } /* compiled thành static blue */

/* CSS vars — runtime, có inheritance, reactive */
:root { --color: blue; }
.component { color: var(--color); }
/* Có thể thay đổi qua JavaScript: */
document.documentElement.style.setProperty('--color', 'red');
/* Tất cả elements dùng --color sẽ update ngay! */
```

---

## 7. Box Model

### Q7.1
**Level: [Junior]**
**EN:** Explain the CSS Box Model. What does box-sizing: border-box change?
**VI:** Giải thích CSS Box Model. box-sizing: border-box thay đổi gì?

**Trả lời:**

```
content-box (default):

+--------------------------------------------+
|              MARGIN                         |
|   +------------------------------------+    |
|   |           BORDER                  |    |
|   |   +----------------------------+  |    |
|   |   |        PADDING             |  |    |
|   |   |   +--------------------+   |  |    |
|   |   |   |                    |   |  |    |
|   |   |   |      CONTENT       |   |  |    |
|   |   |   |  width × height    |   |  |    |
|   |   |   |                    |   |  |    |
|   |   |   +--------------------+   |  |    |
|   |   |        PADDING             |  |    |
|   |   +----------------------------+  |    |
|   |           BORDER                  |    |
|   +------------------------------------+    |
|              MARGIN                         |
+--------------------------------------------+

Kích thước thực tế = width + padding*2 + border*2
```

**content-box vs border-box:**
```css
/* content-box (default): width chỉ tính content */
.box {
  box-sizing: content-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total width: 200 + 20*2 + 5*2 = 250px */
}

/* border-box: width bao gồm padding + border */
.box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* Total width: 200px (content = 200 - 40 - 10 = 150px) */
}
```

**Best practice — reset toàn bộ:**
```css
*, *::before, *::after {
  box-sizing: border-box; /* dễ tính toán layout hơn nhiều */
}
```

---

### Q7.2
**Level: [Mid]**
**EN:** When does margin collapse happen? How do you prevent it?
**VI:** Margin collapse xảy ra khi nào? Làm thế nào để ngăn nó?

**Trả lời:**

**Margin collapse** xảy ra khi **vertical margins** của các elements "gộp" lại thành một — bằng giá trị margin lớn nhất (không phải tổng).

**Trường hợp 1: Adjacent siblings**
```css
.box1 { margin-bottom: 30px; }
.box2 { margin-top: 20px; }
/* Khoảng cách thực tế = max(30, 20) = 30px, KHÔNG phải 50px */
```

**Trường hợp 2: Parent-child (khi không có separation)**
```html
<div class="parent">   <!-- margin-top: 20px -->
  <div class="child">  <!-- margin-top: 30px -->
  </div>
</div>
<!-- parent's margin-top = max(20, 30) = 30px -->
<!-- child's margin-top không tạo khoảng cách với parent! -->
```

**Trường hợp 3: Empty elements**
```css
.empty {
  margin-top: 20px;
  margin-bottom: 30px;
}
/* Hai margin collapse → effective margin = 30px */
```

**Ngăn margin collapse:**
```css
/* 1. Thêm border hoặc padding vào parent */
.parent { padding-top: 1px; }    /* ngăn parent-child collapse */
.parent { border-top: 1px solid transparent; }

/* 2. Tạo new Block Formatting Context (BFC) */
.parent { overflow: hidden; }    /* tạo BFC */
.parent { display: flow-root; }  /* cách hiện đại, tạo BFC */
.parent { display: flex; }       /* flex container không collapse margins */
.parent { display: grid; }

/* 3. Dùng gap thay margin trong flex/grid */
.container {
  display: flex;
  flex-direction: column;
  gap: 20px; /* KHÔNG bị margin collapse */
}
```

**Lưu ý: Margin collapse CHỈ xảy ra theo chiều dọc (vertical), KHÔNG xảy ra:**
- Với `display: flex` hoặc `display: grid` children
- Với `float` elements
- Với `position: absolute/fixed`
- Với horizontal margins

---

### Q7.3
**Level: [Mid]**
**EN:** What is the difference between inline, block, and inline-block display?
**VI:** Sự khác biệt giữa inline, block, và inline-block display là gì?

**Trả lời:**

```
BLOCK:
[====================] ← takes full width
[====================]
[====================]
- Starts on new line
- Takes full available width
- Can set width, height, margin, padding (all directions)
- Examples: div, p, h1-h6, ul, li, section

INLINE:
[text][span][link] → flows with text
- Does NOT start on new line
- Width = content width only (cannot set width/height)
- Vertical margin/padding: renders but doesn't affect layout
- Examples: span, a, strong, em, img

INLINE-BLOCK:
[button][button][button] → flows like inline
but each has box model like block
- Flows with text (like inline)
- Can set width, height, margin, padding (like block)
- Examples: button, input, img (technically inline-block)
```

```css
/* Ví dụ thực tế */
span {
  display: inline;
  width: 100px;    /* IGNORED — inline cannot set width */
  height: 50px;    /* IGNORED */
  margin-top: 20px; /* does not push elements above/below */
}

span {
  display: inline-block;
  width: 100px;    /* WORKS */
  height: 50px;    /* WORKS */
  margin-top: 20px; /* WORKS — pushes elements */
}
```

---

## 8. Stacking Context và z-index

### Q8.1
**Level: [Mid]**
**EN:** What is a stacking context and what creates it?
**VI:** Stacking context là gì và gì tạo ra nó?

**Trả lời:**

**Stacking context** là một "không gian 3D" độc lập — các elements bên trong được xếp chồng theo z-axis, nhưng **không thể thoát ra ngoài** context của parent.

**Stacking context được tạo bởi:**

```css
/* 1. Root element (html) luôn là stacking context */

/* 2. position + z-index khác auto */
.box { position: relative; z-index: 1; }
.box { position: absolute; z-index: 1; }
.box { position: fixed; }     /* z-index auto cũng tạo stacking context */
.box { position: sticky; }    /* tương tự */

/* 3. Opacity < 1 */
.box { opacity: 0.99; } /* tạo stacking context! */

/* 4. Transform khác none */
.box { transform: translateX(0); }

/* 5. Filter */
.box { filter: blur(0px); } /* vẫn tạo stacking context! */

/* 6. will-change với bất kỳ property nào */
.box { will-change: transform; }

/* 7. isolation: isolate */
.box { isolation: isolate; } /* explicitly tạo stacking context */

/* 8. Flex/Grid children với z-index */
.flex-child { z-index: 1; } /* trong flex/grid container */

/* 9. clip-path, mask, mix-blend-mode, etc. */
```

**Stacking order trong một context:**
```
Lowest z-index (renders first, under everything):
1. Background và borders của stacking context element
2. Negative z-index children (z-index: -1)
3. Block-level flow children (no z-index)
4. Float elements
5. Inline-level flow children
6. z-index: 0 positioned children
7. Positive z-index children (highest z-index = on top)
```

---

### Q8.2
**Level: [Senior]**
**EN:** Why does z-index: 9999 sometimes not work? How do you debug stacking context issues?
**VI:** Tại sao z-index: 9999 đôi khi không hoạt động? Làm thế nào debug stacking context issues?

**Trả lời:**

**z-index chỉ so sánh trong cùng stacking context:**

```html
<div class="parent-a" style="position:relative; z-index:1">
  <div class="child-a" style="position:relative; z-index:9999">
    <!-- z-index 9999 nhưng chỉ trong parent-a's context -->
  </div>
</div>

<div class="parent-b" style="position:relative; z-index:2">
  <div class="child-b" style="position:relative; z-index:1">
    <!-- z-index 1 nhưng trong parent-b có z-index:2 -->
  </div>
</div>

<!-- child-b (z-index:1 trong parent-b:2) sẽ ĐÈ LÊN child-a (z-index:9999 trong parent-a:1) -->
<!-- Vì parent-b (z-index:2) > parent-a (z-index:1) -->
```

**Ví dụ common bug:**
```css
/* Modal không hiện trên top nav */
.top-nav {
  position: fixed;
  z-index: 100;
  /* nav này tạo stacking context! */
}

.modal-trigger {
  position: relative;
  z-index: 1; /* nằm TRONG nav's stacking context */
}

.modal {
  position: fixed;
  z-index: 9999; /* vẫn nằm TRONG nav's context → bị nav che */
}

/* Fix: đưa modal ra ngoài nav, hoặc render vào document.body */
```

**React Portal giải quyết vấn đề này:**
```javascript
// Modal được render vào document.body — thoát khỏi nested stacking context
ReactDOM.createPortal(
  <Modal />,
  document.body
);
```

**Debug stacking context:**
1. **Chrome DevTools → Elements → 3D View**: Xem layers theo chiều sâu
2. **DevTools → Layers panel**: Xem composite layers
3. **Kiểm tra:** Inspect element → Computed tab → tìm `z-index`, `position`, `opacity`, `transform`
4. **Code approach:** Dùng `isolation: isolate` để explicitly tạo stacking context thay vì dùng side effects như opacity

```javascript
// Debug script: tìm tất cả stacking contexts
const allElements = document.querySelectorAll('*');
allElements.forEach(el => {
  const style = getComputedStyle(el);
  const isStackingContext =
    style.zIndex !== 'auto' && style.position !== 'static' ||
    style.opacity !== '1' ||
    style.transform !== 'none' ||
    style.filter !== 'none' ||
    style.isolation === 'isolate';

  if (isStackingContext) {
    console.log(el, { zIndex: style.zIndex, position: style.position });
  }
});
```

---

## 9. JavaScript Engine — V8

### Q9.1
**Level: [Mid]**
**EN:** How does the V8 JavaScript engine compile and optimize JavaScript?
**VI:** V8 JavaScript engine compile và optimize JavaScript như thế nào?

**Trả lời:**

```
JS Source Code
      |
      v
  [Parser]
  - Tokenizer (lexical analysis)
  - Create AST (Abstract Syntax Tree)
      |
      v
  [Ignition] — Interpreter
  - Converts AST → Bytecode
  - Starts executing immediately
  - Collects "type feedback" (profiling)
      |
      v
  [TurboFan] — Optimizing Compiler
  - Hot functions (called many times) → machine code
  - Speculative optimization based on type feedback
  - Deoptimize if assumptions wrong
      |
      v
  Machine Code (native CPU instructions)
```

**Hidden Classes — tại sao object shape consistency quan trọng:**

```javascript
// BAD: objects có shape khác nhau
function Point(x, y) {
  this.x = x;
  this.y = y;
}
const p1 = new Point(1, 2);
p1.z = 3; // thêm property sau → tạo hidden class mới!

// V8 tạo hidden classes:
// HC0: {} (empty)
// HC1: {x}
// HC2: {x, y}  ← p1 sau constructor
// HC3: {x, y, z} ← p1 sau thêm z
// Mỗi shape thay đổi → inline cache miss → slower

// GOOD: consistent shape
function Point(x, y, z) {
  this.x = x;
  this.y = y;
  this.z = z; // khai báo hết trong constructor
}
// Tất cả Points có cùng HC2: {x, y, z}
// V8 optimize được!
```

**Inline Caches (IC):**
```javascript
// V8 "ghi nhớ" type của property access
function getX(obj) {
  return obj.x; // lần đầu: lookup slow
}

getX({x: 1}); // V8 nhớ: obj có shape HC1, x ở offset 0
getX({x: 2}); // cache hit! trực tiếp lấy offset 0
getX({x: 3}); // cache hit! same shape

getX({x: 1, y: 2}); // cache MISS! khác shape → slow, deoptimize
```

---

### Q9.2
**Level: [Senior]**
**EN:** Explain V8's garbage collection. What causes memory leaks?
**VI:** Giải thích garbage collection của V8. Những gì gây ra memory leaks?

**Trả lời:**

**V8 GC — Generational Model:**

```
Heap Memory:
+------------------+------------------------+
|   Young Space    |      Old Space          |
|   (New Gen)      |      (Old Gen)          |
|                  |                         |
| Nursery | Inter- | Large  | Code   | Map   |
|         | mediate| Object | Space  | Space |
|  (1MB)  |  (1MB) |        |        |       |
+------------------+------------------------+

"Most objects die young" — generational hypothesis
```

**Minor GC (Scavenger) — Young Space:**
- Chạy thường xuyên, nhanh (< 1ms thường)
- Dùng **Cheney's algorithm** (semi-space copying)
- Objects sống sót → copy sang "Intermediate" space
- Sống sót lần 2 → promote lên Old Space

**Major GC (Mark-Sweep-Compact) — Old Space:**
- Chạy ít thường xuyên hơn, có thể tốn 100ms+
- **Mark**: bắt đầu từ "roots" (global, stack, builtins), traverse graph, đánh dấu reachable objects
- **Sweep**: duyệt heap, free unmarked objects
- **Compact**: defragment memory (không phải lúc nào cũng chạy)

**Incremental marking**: V8 chia marking thành nhiều chunks nhỏ, xen kẽ với JavaScript execution → giảm pause time.

**Common Memory Leaks:**

```javascript
// 1. Forgotten global variables
function leak() {
  leakedVar = 'hello'; // không có var/let/const → global!
}

// 2. Closures giữ references không cần thiết
function createLeak() {
  const largeData = new Array(1000000).fill('data');
  return function() {
    // Closure giữ largeData mặc dù không dùng
    return 42;
  };
}
const leakFunc = createLeak();
// largeData không bao giờ được GC vì closure reference nó

// 3. Event listeners không được remove
function addHandler() {
  const element = document.getElementById('btn');
  const data = fetchLargeData();
  element.addEventListener('click', () => {
    console.log(data); // data bị giữ bởi listener
  });
  // Nếu element bị remove nhưng listener không bị remove → leak
}
// Fix:
const controller = new AbortController();
element.addEventListener('click', handler, { signal: controller.signal });
controller.abort(); // remove tất cả listeners với signal này

// 4. Timers không được clear
function startPolling() {
  const data = getHeavyData();
  setInterval(() => {
    process(data); // data bị giữ bởi timer
  }, 1000);
  // Nếu không clearInterval → data và closure sống mãi mãi
}

// 5. Detached DOM nodes
let elements = [];
function addElement() {
  const el = document.createElement('div');
  document.body.appendChild(el);
  elements.push(el); // giữ reference trong array
}
function removeElement() {
  const el = elements.pop();
  document.body.removeChild(el); // remove khỏi DOM
  // el vẫn còn trong elements array! → detached DOM node
  // GC không thể collect vì JS vẫn giữ reference
}

// 6. WeakMap/WeakSet — giải pháp cho cache với objects
const cache = new WeakMap(); // key phải là object
function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveCompute(obj);
  cache.set(obj, result);
  return result;
}
// Khi obj không còn reference nào khác → WeakMap entry tự bị GC
// Không leak dù obj bị remove khỏi nơi khác
```

**Debug memory leaks với Chrome DevTools:**
1. Memory tab → Take Heap Snapshot → làm action → Take another snapshot → Compare
2. Performance tab → record với Memory checkbox → xem memory timeline
3. Tìm "Detached HTMLElement" trong heap snapshot

---

## 10. Event Loop (deep dive)

### Q10.1
**Level: [Mid]**
**EN:** Explain the JavaScript Event Loop. What is the exact execution order?
**VI:** Giải thích JavaScript Event Loop. Thứ tự thực thi chính xác là gì?

**Trả lời:**

```
JavaScript Runtime Architecture:

+------------------+     +------------------+
|    CALL STACK    |     |    WEB APIs      |
|                  |     | setTimeout       |
| main()           |     | fetch()          |
| foo()            |     | DOM events       |
| bar()            |     | requestAnim...   |
|                  |     | MutationObserver |
+------------------+     +------------------+
         |                        |
         |                        |
         v                        v
+------------------+     +------------------+
| MICROTASK QUEUE  |     |  MACROTASK QUEUE |
| (Priority Queue) |     |  (Task Queue)    |
|                  |     |                  |
| Promise.then()   |     | setTimeout cb    |
| queueMicrotask() |     | setInterval cb   |
| MutationObserver |     | I/O callbacks    |
|                  |     | MessageChannel   |
+------------------+     +------------------+
         |                        |
         +------------------------+
                    |
               EVENT LOOP
         (checks queues, moves to stack)
```

**Thứ tự thực thi CHÍNH XÁC:**

```
1. Chạy hết synchronous code (Call Stack)
2. Chạy HẾT microtasks trong Microtask Queue
   (kể cả microtasks được thêm vào trong quá trình chạy microtasks)
3. Render (nếu cần — trước render, RAF callbacks chạy)
4. Lấy MỘT macrotask từ Macrotask Queue và chạy
5. Quay lại bước 2 (chạy hết microtasks sau mỗi macrotask)
6. Repeat
```

**Ví dụ thực tế:**
```javascript
console.log('1 - sync');

setTimeout(() => console.log('2 - macro'), 0);

Promise.resolve().then(() => {
  console.log('3 - micro');
  Promise.resolve().then(() => console.log('4 - micro nested'));
});

queueMicrotask(() => console.log('5 - microtask'));

console.log('6 - sync');

// OUTPUT:
// 1 - sync          (sync)
// 6 - sync          (sync)
// 3 - micro         (microtask)
// 4 - micro nested  (microtask — added during microtask processing)
// 5 - microtask     (microtask)
// 2 - macro         (macrotask — after ALL microtasks done)
```

---

### Q10.2
**Level: [Senior]**
**EN:** Which APIs produce microtasks vs macrotasks? Where does requestAnimationFrame fit?
**VI:** Những API nào tạo ra microtasks vs macrotasks? requestAnimationFrame nằm ở đâu?

**Trả lời:**

**Microtask producers:**
| API | Notes |
|-----|-------|
| `Promise.then/catch/finally` | Most common |
| `queueMicrotask(fn)` | Explicit microtask scheduling |
| `MutationObserver` callbacks | DOM change callbacks |
| `async/await` | Await point = Promise.then |

**Macrotask producers:**
| API | Notes |
|-----|-------|
| `setTimeout(fn, delay)` | Minimum delay ≥ 4ms trong nested calls |
| `setInterval(fn, delay)` | Repeating |
| `MessageChannel.postMessage` | Faster than setTimeout (no 4ms minimum) |
| `setImmediate(fn)` | Node.js only, before I/O |
| I/O callbacks | File reads, network, etc. |
| `requestIdleCallback` | When browser is idle |

**requestAnimationFrame — đặc biệt:**
```
RAF không phải macrotask hay microtask — nó là "animation frame callback"

Event Loop với RAF:
┌─────────────────────────────────────────────────────┐
│ 1. Macrotask (e.g., setTimeout callback)            │
│ 2. ALL Microtasks                                   │
│ 3. [RAF callbacks] ← chạy ở đây, trước paint       │
│ 4. Layout & Paint & Composite (if needed)           │
│ 5. → next iteration                                 │
└─────────────────────────────────────────────────────┘
```

```javascript
// Thứ tự với RAF:
setTimeout(() => console.log('1 - setTimeout'), 0);
requestAnimationFrame(() => console.log('2 - RAF'));
Promise.resolve().then(() => console.log('3 - Promise'));
console.log('4 - sync');

// Output (gần đúng — RAF phụ thuộc vào vsync):
// 4 - sync
// 3 - Promise    (microtask, trước macrotask)
// 1 - setTimeout (macrotask)
// 2 - RAF        (animation frame — thường sau macrotask, trước paint)
// [paint frame]
```

**Long Tasks và UI blocking:**
```javascript
// Long task > 50ms block toàn bộ UI
// User không thể click, scroll, type trong thời gian này

// BAD: synchronous heavy computation
function heavyWork() {
  let sum = 0;
  for (let i = 0; i < 1e9; i++) sum += i; // blocks 1+ giây!
  return sum;
}

// GOOD: Break into chunks với scheduler
function processInChunks(items) {
  let index = 0;

  function processChunk() {
    const deadline = performance.now() + 10; // 10ms budget

    while (index < items.length && performance.now() < deadline) {
      processItem(items[index++]);
    }

    if (index < items.length) {
      // Yield về event loop, tiếp tục trong macrotask tiếp theo
      setTimeout(processChunk, 0);
      // Hoặc dùng scheduler.postTask() nếu available
    }
  }

  processChunk();
}

// Scheduler API (Chrome 94+):
scheduler.postTask(() => heavyWork(), { priority: 'background' });
```

---

## 11. Browser Storage

### Q11.1
**Level: [Junior]**
**EN:** What are the different browser storage options? When would you use each?
**VI:** Các lựa chọn lưu trữ của browser là gì? Khi nào dùng mỗi loại?

**Trả lời:**

**Comparison Table:**

| | Cookie | localStorage | sessionStorage | IndexedDB | Cache API |
|--|--------|--------------|----------------|-----------|-----------|
| Capacity | 4KB | 5-10MB | 5-10MB | Hundreds MB | Hundreds MB |
| Persistent | Có (nếu có expiry) | Có | Không (tab) | Có | Có |
| Sent với HTTP | Có (auto) | Không | Không | Không | Không |
| API | Synchronous | Synchronous | Synchronous | Asynchronous | Asynchronous |
| Accessible | JS + Server | JS only | JS only | JS only | Service Worker |
| Scope | Origin + path | Origin | Origin + tab | Origin | Origin |

**Cookie:**
```javascript
// Set cookie
document.cookie = "username=John; expires=Thu, 18 Dec 2025 12:00:00 UTC; path=/";
document.cookie = "theme=dark; max-age=86400; SameSite=Strict; Secure";

// HttpOnly cookies: KHÔNG accessible từ JS — only server!
// Server set: Set-Cookie: sessionId=abc123; HttpOnly; Secure

// SameSite values:
// Strict: chỉ gửi khi navigate từ cùng site
// Lax: gửi khi top-level navigation (default mới)
// None: gửi mọi request (cần Secure)
```

**localStorage:**
```javascript
// Synchronous — block main thread (nhưng thường đủ nhanh cho nhỏ)
localStorage.setItem('user', JSON.stringify({ name: 'John' }));
const user = JSON.parse(localStorage.getItem('user'));
localStorage.removeItem('user');
localStorage.clear();

// Storage event (cross-tab communication!)
window.addEventListener('storage', (e) => {
  console.log(e.key, e.newValue, e.oldValue);
  // Fires in OTHER tabs khi localStorage changes
});
```

**IndexedDB:**
```javascript
// Asynchronous, transactional, can store complex data
const request = indexedDB.open('MyDB', 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  const store = db.createObjectStore('users', { keyPath: 'id' });
  store.createIndex('name', 'name', { unique: false });
};

request.onsuccess = (event) => {
  const db = event.target.result;
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  store.add({ id: 1, name: 'John', age: 30 });
};

// Thường dùng với wrapper library: idb, Dexie.js
import { openDB } from 'idb';
const db = await openDB('MyDB', 1, {
  upgrade(db) {
    db.createObjectStore('users', { keyPath: 'id' });
  },
});
await db.add('users', { id: 1, name: 'John' });
```

**Cache API (Service Worker):**
```javascript
// Offline support, PWA
const CACHE_NAME = 'v1';

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(['/index.html', '/styles.css', '/app.js']);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request); // cache first
    })
  );
});
```

---

## 12. HTTP và Network

### Q12.1
**Level: [Mid]**
**EN:** What are the key differences between HTTP/1.1, HTTP/2, and HTTP/3?
**VI:** Sự khác biệt chính giữa HTTP/1.1, HTTP/2, và HTTP/3 là gì?

**Trả lời:**

**HTTP/1.1 Problems:**
```
Request 1: GET /style.css   ──────────► [server]
Response 1:                 ◄─────────── [server]
Request 2: GET /script.js   ──────────► [server]  ← WAIT
Response 2:                 ◄───────────
Request 3: GET /image.png   ──────────►
Response 3:                 ◄───────────

Head-of-line blocking: Phải đợi response 1 xong mới gửi request 2.
Workaround: Browser mở 6 TCP connections per domain → domain sharding (anti-pattern now)
```

**HTTP/2 — Multiplexing:**
```
One TCP connection, multiple streams:

Connection:
  Stream 1: GET /style.css  ──► ◄─ Response (interleaved)
  Stream 2: GET /script.js  ──► ◄─ Response (interleaved)
  Stream 3: GET /image.png  ──► ◄─ Response (interleaved)

Tất cả parallel trên 1 connection! No head-of-line blocking.

Header Compression (HPACK):
HTTP/1.1: Headers được gửi full mỗi request (nhiều repeat)
HTTP/2: Headers được nén + delta encoding (chỉ gửi phần thay đổi)

Server Push:
Server có thể push /style.css khi client request /index.html
trước khi client biết nó cần file đó.
```

**HTTP/3 — QUIC (UDP-based):**
```
HTTP/2 vẫn có TCP HOL blocking:
Nếu một packet bị mất → toàn bộ connection phải chờ retransmit

HTTP/3 dùng QUIC (Quick UDP Internet Connections):
- Chạy trên UDP thay vì TCP
- Mỗi stream độc lập → packet loss chỉ ảnh hưởng stream đó
- 0-RTT handshake (subsequent connections không cần full TLS handshake)
- Connection migration (chuyển từ WiFi sang 4G mà không reconnect)
```

**So sánh:**
| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|--|----------|--------|--------|
| Protocol | TCP | TCP | QUIC/UDP |
| Multiplexing | No (workaround: 6 connections) | Yes (1 connection) | Yes |
| HOL Blocking | Yes | At TCP level | No |
| Header Compression | No | HPACK | QPACK |
| Server Push | No | Yes | Yes |
| 0-RTT | No | No | Yes |

---

### Q12.2
**Level: [Mid]**
**EN:** Explain browser caching mechanisms — Cache-Control, ETag, and how 304 works.
**VI:** Giải thích cơ chế browser caching — Cache-Control, ETag, và 304 hoạt động thế nào?

**Trả lời:**

**Cache-Control directives:**

```
Server Response Headers:
Cache-Control: max-age=3600          → cache 1 giờ
Cache-Control: no-cache              → LUÔN revalidate với server (không có nghĩa là no cache!)
Cache-Control: no-store              → KHÔNG cache gì cả (sensitive data)
Cache-Control: public                → có thể cache ở CDN/proxy
Cache-Control: private               → chỉ browser cache (không cache ở CDN)
Cache-Control: immutable             → file sẽ không bao giờ thay đổi
Cache-Control: stale-while-revalidate=60 → dùng cache cũ trong khi revalidate ngầm
```

**ETag mechanism:**

```
First Request:
Browser: GET /style.css
Server:  200 OK
         ETag: "abc123"          ← hash của file content
         Cache-Control: max-age=86400
         [file content]

Cache expires after 24 hours:
Browser: GET /style.css
         If-None-Match: "abc123" ← gửi ETag đã có

Case 1 — File không thay đổi:
Server:  304 Not Modified        ← NO body! chỉ headers
         ETag: "abc123"          ← có thể refresh cache
         ← browser dùng cached version, tiết kiệm bandwidth

Case 2 — File đã thay đổi:
Server:  200 OK
         ETag: "xyz789"          ← ETag mới
         [new file content]      ← full response
```

**Cache strategy cho static assets:**

```
index.html: Cache-Control: no-cache
            (cần revalidate để nhận JS/CSS mới)

app.[hash].js: Cache-Control: max-age=31536000, immutable
               (content-hashed filename → safe to cache forever)

fonts: Cache-Control: max-age=31536000, immutable
```

**CORS Preflight:**
```
Khi browser gửi "non-simple" cross-origin request (DELETE, PUT, custom headers):

Browser: OPTIONS /api/data        ← Preflight
         Origin: https://example.com
         Access-Control-Request-Method: DELETE
         Access-Control-Request-Headers: Authorization

Server:  200 OK
         Access-Control-Allow-Origin: https://example.com
         Access-Control-Allow-Methods: GET, POST, DELETE
         Access-Control-Allow-Headers: Authorization
         Access-Control-Max-Age: 86400 ← cache preflight result

Browser: DELETE /api/data         ← Actual request
         Origin: https://example.com
         Authorization: Bearer token
```

---

## 13. Web APIs — RAF, IntersectionObserver, MutationObserver, ResizeObserver

### Q13.1
**Level: [Mid]**
**EN:** How does IntersectionObserver work? What are its use cases?
**VI:** IntersectionObserver hoạt động thế nào? Những use cases của nó là gì?

**Trả lời:**

**IntersectionObserver** theo dõi khi một element "intersect" (giao nhau) với viewport hoặc ancestor element khác — **không dùng scroll events**.

**Cách hoạt động:**
- Observer chạy off-main-thread khi có thể
- Chỉ fire callback khi element's intersection ratio vượt qua `threshold`
- Async — không block scroll, không trigger layout

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    console.log({
      target: entry.target,
      isIntersecting: entry.isIntersecting,
      intersectionRatio: entry.intersectionRatio, // 0.0 → 1.0
      boundingClientRect: entry.boundingClientRect,
      rootBounds: entry.rootBounds,
    });
  });
}, {
  root: null,          // null = viewport
  rootMargin: '0px',   // expand/shrink root bounds (như CSS margin)
  threshold: [0, 0.5, 1.0] // fire khi 0%, 50%, 100% visible
});

observer.observe(element);
observer.unobserve(element); // stop observing
observer.disconnect();       // stop all

// Use case 1: Lazy loading images
const lazyImages = document.querySelectorAll('img[data-src]');
const imageObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;    // swap placeholder với real image
      imageObserver.unobserve(img); // stop observing sau khi loaded
    }
  });
}, { rootMargin: '100px' }); // load 100px trước khi vào viewport

lazyImages.forEach(img => imageObserver.observe(img));

// Use case 2: Infinite scroll
const sentinel = document.getElementById('load-more-sentinel');
const scrollObserver = new IntersectionObserver((entries) => {
  if (entries[0].isIntersecting) {
    loadMoreItems();
  }
});
scrollObserver.observe(sentinel);

// Use case 3: Analytics — track visible time
let visibleStart;
const analyticsObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      visibleStart = Date.now();
    } else if (visibleStart) {
      const duration = Date.now() - visibleStart;
      trackEvent('element_visible', { duration });
      visibleStart = null;
    }
  });
}, { threshold: 0.5 }); // 50% visible để tính là "seen"
```

---

### Q13.2
**Level: [Mid]**
**EN:** What are MutationObserver and ResizeObserver? Give practical use cases.
**VI:** MutationObserver và ResizeObserver là gì? Cho ví dụ thực tế.

**Trả lời:**

**MutationObserver** — theo dõi DOM changes:

```javascript
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    if (mutation.type === 'childList') {
      console.log('Children added:', mutation.addedNodes);
      console.log('Children removed:', mutation.removedNodes);
    }
    if (mutation.type === 'attributes') {
      console.log('Attribute changed:', mutation.attributeName);
      console.log('Old value:', mutation.oldValue);
    }
    if (mutation.type === 'characterData') {
      console.log('Text changed:', mutation.target.textContent);
    }
  });
});

observer.observe(element, {
  childList: true,       // theo dõi thêm/xóa children
  attributes: true,      // theo dõi attribute changes
  characterData: true,   // theo dõi text content changes
  subtree: true,         // theo dõi tất cả descendants
  attributeOldValue: true,   // ghi lại old attribute value
  characterDataOldValue: true // ghi lại old text value
});

// Use case 1: Third-party DOM manipulation detection
const chatObserver = new MutationObserver((mutations) => {
  mutations.forEach(m => {
    m.addedNodes.forEach(node => {
      if (node.nodeType === 1 && node.matches('.chat-message')) {
        scrollToBottom();
        notifyNewMessage(node);
      }
    });
  });
});
chatObserver.observe(chatContainer, { childList: true });

// Use case 2: Custom element attribute watching
// (trước khi có Custom Elements API)
const attrObserver = new MutationObserver((mutations) => {
  if (mutations.some(m => m.attributeName === 'data-theme')) {
    applyTheme(element.dataset.theme);
  }
});
attrObserver.observe(element, { attributes: true });
```

**ResizeObserver** — theo dõi element size changes:

```javascript
const resizeObserver = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    console.log('New size:', width, height);

    // entry.borderBoxSize — including border
    // entry.contentBoxSize — content only
    // entry.devicePixelContentBoxSize — device pixels
  });
});

resizeObserver.observe(element);

// Use case 1: Responsive component (không phụ thuộc viewport width)
// Container Queries trước khi CSS hỗ trợ
const cardObserver = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    const { width } = entry.contentRect;
    const card = entry.target;
    card.classList.toggle('card--compact', width < 300);
    card.classList.toggle('card--expanded', width >= 600);
  });
});
document.querySelectorAll('.card').forEach(card => cardObserver.observe(card));

// Use case 2: Canvas resize
const canvasObserver = new ResizeObserver((entries) => {
  const { width, height } = entries[0].contentRect;
  canvas.width = width * devicePixelRatio;
  canvas.height = height * devicePixelRatio;
  redrawCanvas();
});
canvasObserver.observe(canvasContainer);
```

**PerformanceObserver — Web Vitals monitoring:**

```javascript
// Observe Core Web Vitals trong production
const observer = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    if (entry.entryType === 'largest-contentful-paint') {
      console.log('LCP:', entry.startTime);
    }
    if (entry.entryType === 'layout-shift') {
      if (!entry.hadRecentInput) {
        console.log('CLS contribution:', entry.value);
      }
    }
  }
});

observer.observe({ entryTypes: ['largest-contentful-paint', 'layout-shift'] });

// First Input Delay
const fidObserver = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    const fid = entry.processingStart - entry.startTime;
    console.log('FID:', fid);
  }
});
fidObserver.observe({ entryTypes: ['first-input'] });
```

---

## 14. HTML Semantics và Accessibility

### Q14.1
**Level: [Junior]**
**EN:** Why does semantic HTML matter? What are the key semantic elements?
**VI:** Tại sao semantic HTML quan trọng? Các semantic elements chính là gì?

**Trả lời:**

**Semantic HTML** — sử dụng HTML elements có ý nghĩa (meaning) đúng với nội dung, không chỉ vì style.

**3 lý do chính:**

1. **Accessibility (a11y)**: Screen readers hiểu page structure và announce đúng context. `<nav>` → "navigation region", `<button>` → "button, press Enter to activate"

2. **SEO**: Search engines weight content dựa trên semantic context. `<h1>` quan trọng hơn `<div class="heading">`. `<article>` signals standalone content.

3. **Maintainability**: Code tự document — `<header>`, `<main>`, `<footer>` rõ ràng hơn `<div id="top">`, `<div id="content">`, `<div id="bottom">`

**Cấu trúc semantic chuẩn:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Page Title</title>
</head>
<body>
  <header>                    <!-- Site/page header -->
    <nav aria-label="Main">   <!-- Navigation với accessible name -->
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </header>

  <main>                      <!-- ONE main per page — primary content -->
    <article>                 <!-- Self-contained, distributable content -->
      <header>
        <h1>Article Title</h1>
        <time datetime="2025-01-15">January 15, 2025</time>
        <address>By <a href="/author">John</a></address>
      </header>

      <section>               <!-- Thematic grouping, usually has heading -->
        <h2>Section Title</h2>
        <p>Content...</p>
      </section>

      <aside>                 <!-- Related but tangential content -->
        <h2>Related Articles</h2>
      </aside>
    </article>
  </main>

  <footer>
    <p><small>&copy; 2025 Company</small></p>
  </footer>
</body>
</html>
```

**Sự khác biệt giữa các sectioning elements:**
| Element | Dùng khi |
|---------|----------|
| `<article>` | Content có thể stand-alone (blog post, product card, tweet) |
| `<section>` | Thematic grouping có heading (chapter, tab panel) |
| `<aside>` | Sidebar, pull quote, ads — related nhưng không essential |
| `<nav>` | Navigation links — major navigational blocks |
| `<header>` | Introductory content — site header hoặc article header |
| `<footer>` | Footer của section hoặc page |
| `<main>` | Primary content — chỉ một per page |

---

### Q14.2
**Level: [Mid]**
**EN:** When should you use ARIA vs native HTML? Explain tabindex values.
**VI:** Khi nào dùng ARIA vs native HTML? Giải thích các giá trị tabindex.

**Trả lời:**

**Rule: "No ARIA is better than bad ARIA"**

Native HTML > ARIA luôn luôn:

```html
<!-- BAD: div với ARIA (cần nhiều attributes để recreate native behavior) -->
<div role="button"
     tabindex="0"
     aria-pressed="false"
     aria-label="Toggle menu"
     onclick="toggle()"
     onkeydown="if(event.key==='Enter'||event.key===' ')toggle()">
  Menu
</div>

<!-- GOOD: native button (tất cả built-in!) -->
<button onclick="toggle()" aria-pressed="false" aria-label="Toggle menu">
  Menu
</button>
<!-- button: focusable by default, Enter/Space activate, role="button" tự động -->
```

**Khi nào DÙNG ARIA:**
1. Không có HTML element phù hợp (combobox, tree, tabpanel, alert)
2. Bổ sung thông tin cho custom widgets
3. Thay đổi dynamic state

```html
<!-- Tab component không có native HTML element -->
<div role="tablist" aria-label="Settings">
  <button role="tab"
          aria-selected="true"
          aria-controls="panel-1"
          id="tab-1">
    General
  </button>
  <button role="tab"
          aria-selected="false"
          aria-controls="panel-2"
          id="tab-2">
    Privacy
  </button>
</div>

<div role="tabpanel"
     id="panel-1"
     aria-labelledby="tab-1">
  General settings content
</div>

<!-- Live regions — announce dynamic changes -->
<div aria-live="polite" aria-atomic="true">
  <!-- Content thay đổi ở đây sẽ được announce bởi screen reader -->
  <p>3 results found</p>
</div>
<div aria-live="assertive">
  <!-- Interrupt user ngay — dùng cho errors quan trọng -->
  <p>Error: Session expired</p>
</div>
```

**tabindex values:**
```html
<!-- tabindex="0": thêm element vào tab order (tại vị trí tự nhiên trong DOM) -->
<div tabindex="0" role="button" onclick="...">Focusable div</div>

<!-- tabindex="-1": có thể focus bằng JS (element.focus()) nhưng không qua Tab -->
<div tabindex="-1" id="modal-content">
  <!-- Dùng cho focus management trong modals, drawers -->
</div>
<script>
  document.getElementById('modal-content').focus(); // OK
  // Tab key sẽ KHÔNG đến element này
</script>

<!-- tabindex="1+" (positive): AVOID — breaks natural tab order -->
<input tabindex="2"> <!-- BAD: rối tung tab order -->
<input tabindex="1"> <!-- Tab đến đây trước dù ở dưới trong DOM -->

<!-- Focus management cho modal -->
function openModal(modal) {
  modal.removeAttribute('hidden');
  const firstFocusable = modal.querySelector('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
  firstFocusable?.focus();

  // Trap focus trong modal
  modal.addEventListener('keydown', trapFocus);
}

function trapFocus(e) {
  if (e.key !== 'Tab') return;
  const focusables = [...modal.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  )].filter(el => !el.disabled);
  const first = focusables[0];
  const last = focusables[focusables.length - 1];

  if (e.shiftKey && document.activeElement === first) {
    e.preventDefault();
    last.focus();
  } else if (!e.shiftKey && document.activeElement === last) {
    e.preventDefault();
    first.focus();
  }
}
```

---

## 15. Browser Rendering Optimization — Practical

### Q15.1
**Level: [Senior]**
**EN:** What is layout thrashing? How do you detect and fix it?
**VI:** Layout thrashing là gì? Làm thế nào để phát hiện và fix nó?

**Trả lời:**

**Layout thrashing** (hay Forced Synchronous Layout) xảy ra khi bạn xen kẽ reads và writes của DOM layout properties trong một loop, buộc browser phải recalculate layout nhiều lần thay vì một lần.

**Các properties trigger synchronous layout khi đọc:**
```javascript
// Các "layout-triggering" properties:
element.offsetTop, element.offsetLeft
element.offsetWidth, element.offsetHeight
element.offsetParent
element.clientTop, element.clientLeft
element.clientWidth, element.clientHeight
element.scrollTop, element.scrollLeft
element.scrollWidth, element.scrollHeight
element.getBoundingClientRect()
element.getClientRects()
window.getComputedStyle(element) // trong một số cases
window.scrollX, window.scrollY
window.innerWidth, window.innerHeight
```

**Ví dụ layout thrashing:**
```javascript
// BAD: Layout thrashing — recalculate layout N lần
const boxes = document.querySelectorAll('.box');

boxes.forEach(box => {
  const width = box.offsetWidth;         // READ → force layout
  box.style.width = (width * 2) + 'px'; // WRITE → invalidate layout
  // Next iteration: READ → force layout AGAIN (vì layout bị invalidate)
});

// N elements = N forced layouts!

// GOOD: Batch reads, then batch writes
const widths = [...boxes].map(box => box.offsetWidth); // all READs

boxes.forEach((box, i) => {
  box.style.width = (widths[i] * 2) + 'px'; // all WRITEs
});
// Chỉ 1 layout calculation

// BETTER: Dùng requestAnimationFrame
function updateBoxes() {
  // Read phase
  const widths = [...boxes].map(box => box.offsetWidth);

  requestAnimationFrame(() => {
    // Write phase (trong RAF, ngay trước paint)
    boxes.forEach((box, i) => {
      box.style.width = (widths[i] * 2) + 'px';
    });
  });
}

// BEST: FastDOM library
import fastdom from 'fastdom';

boxes.forEach(box => {
  fastdom.measure(() => {
    const width = box.offsetWidth; // queued read
    fastdom.mutate(() => {
      box.style.width = (width * 2) + 'px'; // queued write
    });
  });
});
// FastDOM tự batch và order reads/writes
```

**Detect layout thrashing trong DevTools:**
1. Performance tab → Record → kiểm tra "Recalculate Style" và "Layout" events
2. Nếu thấy nhiều "Layout" events liên tiếp trong một script → layout thrashing
3. "Forced reflow" warning trong DevTools console (nếu bật)

---

### Q15.2
**Level: [Senior]**
**EN:** Explain CSS `contain` and `content-visibility`. How do they improve performance?
**VI:** Giải thích CSS `contain` và `content-visibility`. Chúng cải thiện performance như thế nào?

**Trả lời:**

**`contain` property** — cho browser biết subtree này **độc lập** với phần còn lại của page:

```css
/* contain values: */
.component {
  contain: layout;   /* Changes bên trong không ảnh hưởng layout bên ngoài */
  contain: paint;    /* Element không paint ngoài bounds của nó */
  contain: style;    /* CSS counters không affect ngoài element */
  contain: size;     /* Element size không phụ thuộc vào children */
  contain: strict;   /* = size layout paint style */
  contain: content;  /* = layout paint style (thường dùng nhất) */
}
```

**Ví dụ `contain: layout`:**
```css
/* Không có contain: */
.card {
  /* Nếu content bên trong thay đổi height →
     browser phải reflow toàn bộ page */
}

/* Với contain: layout: */
.card {
  contain: layout;
  /* Browser chỉ cần reflow bên trong .card
     Phần còn lại của page không bị ảnh hưởng */
}

/* Dùng cho widget/component độc lập */
.news-widget {
  contain: content; /* layout + paint + style */
}
```

**`content-visibility: auto`** — browser skip rendering off-screen content:

```css
/* Killer performance feature cho long documents */
.article-section {
  content-visibility: auto;
  /* Browser sẽ:
     1. Skip layout, paint của section khi off-screen
     2. Tính toán "estimated size" từ contain-intrinsic-size
     3. Render section khi user scroll đến gần */

  contain-intrinsic-size: 0 500px; /* estimated height khi off-screen */
}

/* Real-world impact:
   - document.body với 1000 sections
   - Trước: render tất cả → chậm
   - Sau: chỉ render sections in/near viewport → up to 7x faster initial render */
```

```html
<!-- Ví dụ cho social media feed -->
<div class="feed">
  <article class="post" style="content-visibility: auto; contain-intrinsic-size: 0 200px;">
    <!-- Nội dung phức tạp -->
  </article>
  <article class="post" style="content-visibility: auto; contain-intrinsic-size: 0 200px;">
    <!-- Nội dung phức tạp -->
  </article>
  <!-- ... 1000 posts ... -->
</div>
```

**Layer promotion best practices:**
```css
/* Chỉ promote khi thực sự cần thiết */

/* OK — element sẽ animate */
.animated-modal {
  will-change: transform, opacity;
}
/* Remove sau khi animation xong */

/* OK — fixed header để smooth scroll */
.sticky-nav {
  position: fixed;
  /* Browser đã tạo composite layer cho fixed elements */
}

/* NOT OK — blanket promotion */
.card {
  transform: translateZ(0); /* hack không cần thiết → tốn GPU memory */
}

/* Prefer composite-only animations: */
@keyframes slide-in {
  from { transform: translateX(-100%); opacity: 0; }
  to   { transform: translateX(0);     opacity: 1; }
}
/* Chỉ dùng transform + opacity → chỉ composite → smooth 60fps */

/* AVOID animating layout-triggering properties: */
@keyframes bad-animation {
  from { left: -100px; } /* triggers reflow every frame! */
  to   { left: 0; }
}
/* Thay bằng: */
@keyframes good-animation {
  from { transform: translateX(-100px); } /* composite only */
  to   { transform: translateX(0); }
}
```

---

### Q15.3
**Level: [Senior]**
**EN:** Walk me through all the optimizations you'd apply to a page with a long product list that's scrolling slowly.
**VI:** Hướng dẫn tất cả các optimizations bạn sẽ áp dụng cho một trang có danh sách sản phẩm dài đang scroll chậm.

**Trả lời:**

**Bước 1: Diagnose với DevTools**
```
1. Open Performance tab → Record 5 giây scroll → Analyze
2. Tìm: Long red bars (long tasks > 50ms), paint flashing
3. Rendering tab: bật "Paint flashing" — xem màu xanh nhấp nháy quá nhiều không
4. Check Layers panel: quá nhiều composite layers không?
```

**Bước 2: Áp dụng theo thứ tự ưu tiên**

```javascript
// 1. Virtual scrolling — render chỉ visible items
// Dùng thư viện: react-window, tanstack-virtual

import { FixedSizeList } from 'react-window';

function ProductList({ products }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={products.length}
      itemSize={200}  // height mỗi item
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          <ProductCard product={products[index]} />
        </div>
      )}
    </FixedSizeList>
  );
}
// Thay vì render 10,000 DOM nodes → chỉ render ~15-20 visible items

// 2. Lazy load images với IntersectionObserver
const imgObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.removeAttribute('data-src');
      imgObserver.unobserve(img);
    }
  });
}, { rootMargin: '200px' }); // preload 200px trước

document.querySelectorAll('img[data-src]').forEach(img => {
  imgObserver.observe(img);
});
```

```css
/* 3. CSS contain cho mỗi product card */
.product-card {
  contain: content; /* layout + paint + style containment */
}

/* 4. content-visibility nếu không dùng virtual scroll */
.product-card {
  content-visibility: auto;
  contain-intrinsic-size: 0 200px; /* estimated height */
}

/* 5. Dùng transform thay top/left cho animations */
.product-card:hover {
  transform: translateY(-4px); /* composite only */
  /* KHÔNG dùng: top: -4px; (triggers reflow) */
}

/* 6. passive scroll listener */
```

```javascript
// 7. Event delegation cho buttons
document.getElementById('product-list').addEventListener('click', (e) => {
  const addToCart = e.target.closest('[data-add-to-cart]');
  if (addToCart) {
    const productId = addToCart.dataset.productId;
    cart.add(productId);
  }
}, { passive: true });
// Thay vì 1000 listeners → 1 listener

// 8. Debounce/throttle scroll handler nếu vẫn cần
import { throttle } from 'lodash';

const handleScroll = throttle(() => {
  updateScrollPosition();
}, 16); // max 60fps

window.addEventListener('scroll', handleScroll, { passive: true });
```

```html
<!-- 9. Image optimizations -->
<img
  src="product-sm.jpg"
  srcset="product-sm.jpg 400w, product-md.jpg 800w, product-lg.jpg 1200w"
  sizes="(max-width: 600px) 100vw, 300px"
  loading="lazy"
  decoding="async"
  width="300"
  height="200"  <!-- prevent layout shift -->
  alt="Product name"
>
```

```javascript
// 10. Batch DOM updates với DocumentFragment
function appendProducts(products) {
  const fragment = document.createDocumentFragment();
  products.forEach(product => {
    fragment.appendChild(createProductCard(product));
  });
  productList.appendChild(fragment); // 1 DOM operation
}

// 11. requestIdleCallback cho non-critical work
requestIdleCallback(() => {
  prefetchNextPage(); // preload khi user idle
  generateAnalyticsReport();
}, { timeout: 2000 });
```

**Expected improvements:**
- Virtual scrolling: từ 10,000 DOM nodes → ~20 → render time giảm 99%
- IntersectionObserver lazy load: giảm initial network requests
- CSS contain: giảm reflow scope
- Passive listeners: scroll không lag do event handler
- Event delegation: giảm memory, faster initialization

---

## Quick Reference Cheat Sheet

### CSS Properties by Rendering Impact
```
COMPOSITE ONLY (fastest — GPU):
  transform, opacity, filter*, will-change

REPAINT (medium):
  background, color, border-color, box-shadow,
  outline, visibility, text-decoration

REFLOW (slowest — avoid in animations):
  width, height, margin, padding, border-width
  top, left, bottom, right (when positioned)
  font-size, font-family, line-height
  display, position, float, overflow

*filter tạo stacking context nhưng thường composite
```

### Event Loop Quick Reference
```
Sync → Microtasks (ALL) → RAF → Paint → Macrotask (ONE) → repeat

Microtasks: Promise.then, queueMicrotask, MutationObserver, await
Macrotasks: setTimeout, setInterval, MessageChannel, I/O, setImmediate
```

### Specificity Quick Reference
```
Inline style:    (1,0,0,0) — beats everything except !important
ID selector:     (0,1,0,0) — #nav
Class selector:  (0,0,1,0) — .nav, [href], :hover
Element:         (0,0,0,1) — div, p, ::before
:where():        (0,0,0,0) — always zero
```

### Stacking Context Triggers
```
position + z-index (not auto)
opacity < 1
transform (not none)
filter (not none)
will-change
isolation: isolate
position: fixed/sticky
flex/grid children with z-index
```

---

> **Ghi chú học tập:** File này bao gồm toàn bộ spectrum từ Junior đến Senior. Ưu tiên hiểu SÂU về Critical Rendering Path, Event Loop, và Reflow/Repaint/Composite — đây là những topic interviewer hay hỏi deep nhất. Kết hợp lý thuyết với thực hành trong Chrome DevTools.
