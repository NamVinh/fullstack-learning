# CSS & Styling — Interview Questions (Junior → Senior)

---

## 1. Box Model

**Level:** [Junior]

**EN:** Explain the CSS box model. What is the difference between `box-sizing: content-box` and `box-sizing: border-box`?

**VI:** Giải thích CSS box model. Sự khác biệt giữa `box-sizing: content-box` và `box-sizing: border-box` là gì?

**Trả lời:**

Box model mô tả cách trình duyệt tính toán kích thước và khoảng cách của một phần tử. Mỗi phần tử được bao bọc bởi 4 lớp: **content → padding → border → margin**.

- `content-box` (mặc định): `width`/`height` chỉ áp dụng cho vùng content. Padding và border được cộng thêm ra ngoài.
- `border-box`: `width`/`height` bao gồm cả padding và border. Đây là cách tính trực quan hơn và được dùng phổ biến trong production.

```css
/* content-box: tổng chiều rộng = 200 + 20 + 4 = 224px */
.box-content {
  box-sizing: content-box; /* mặc định */
  width: 200px;
  padding: 10px;
  border: 2px solid #000;
}

/* border-box: tổng chiều rộng = đúng 200px */
.box-border {
  box-sizing: border-box;
  width: 200px;
  padding: 10px;
  border: 2px solid #000;
  /* content width thực tế = 200 - 20 - 4 = 176px */
}

/* Best practice: áp dụng toàn cục */
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

---

## 2. Flexbox

**Level:** [Junior] / [Mid]

**EN:** Explain the main Flexbox properties. How do you implement common layouts like centering, a sidebar layout, and a navbar using Flexbox?

**VI:** Giải thích các thuộc tính Flexbox chính. Làm thế nào để triển khai các layout phổ biến như căn giữa, sidebar layout, và navbar bằng Flexbox?

**Trả lời:**

Flexbox là một chiều (1D) — layout theo hàng hoặc cột. Container có `display: flex` kiểm soát các flex items bên trong.

**Thuộc tính của Flex Container:**
- `flex-direction`: `row | column | row-reverse | column-reverse`
- `justify-content`: căn chỉnh trên **main axis** — `flex-start | center | flex-end | space-between | space-around | space-evenly`
- `align-items`: căn chỉnh trên **cross axis** — `stretch | center | flex-start | flex-end | baseline`
- `flex-wrap`: `nowrap | wrap | wrap-reverse`
- `gap`: khoảng cách giữa các items

**Thuộc tính của Flex Item:**
- `flex`: shorthand cho `flex-grow flex-shrink flex-basis`
- `align-self`: ghi đè `align-items` cho item đó
- `order`: thay đổi thứ tự hiển thị

```css
/* 1. Perfect centering */
.center-both {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

/* 2. Navbar: logo trái, nav giữa, actions phải */
.navbar {
  display: flex;
  align-items: center;
  padding: 0 1rem;
  gap: 1rem;
}

.navbar .logo {
  margin-right: auto; /* đẩy các phần tử còn lại sang phải */
}

.navbar .actions {
  margin-left: auto;
}

/* Cách khác dùng justify-content */
.navbar-v2 {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 3. Sidebar layout: sidebar cố định, main content chiếm phần còn lại */
.layout {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  flex: 0 0 260px; /* không grow, không shrink, width cố định 260px */
  background: #f4f4f4;
}

.main-content {
  flex: 1; /* flex-grow: 1 — chiếm toàn bộ không gian còn lại */
  padding: 1rem;
}

/* 4. Card grid với wrap */
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.card {
  flex: 1 1 280px; /* grow, shrink, min width 280px */
  max-width: 400px;
}

/* 5. Equal height columns */
.columns {
  display: flex;
  align-items: stretch; /* mặc định — tất cả cùng chiều cao */
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Can you explain the difference between Flexbox and Grid? When would you choose one over the other?"**

**Model Answer:**
"Sure. The key difference is that Flexbox is one-dimensional — you lay things out in either a row or a column. Grid is two-dimensional, so you can control both rows and columns at the same time.

I reach for Flexbox when I'm aligning items along a single axis — things like a navbar, a row of buttons, or centering something vertically and horizontally. It's really natural for those cases.

I switch to Grid when I need to manage the overall page structure, or when I have a two-dimensional layout like a card grid where I care about both column widths and row heights. Grid also shines when I need items to span across multiple columns or rows.

A simple rule I follow: if I'm thinking about one direction, Flexbox. If I'm thinking about rows and columns together, Grid."

**Trả lời (Tiếng Việt):**
"Điểm khác biệt mấu chốt là Flexbox một chiều — bạn sắp xếp theo hàng hoặc cột. Grid hai chiều, nên bạn kiểm soát cả hàng lẫn cột cùng lúc.

Tôi dùng Flexbox khi căn chỉnh items theo một trục — navbar, hàng nút, hoặc căn giữa theo chiều dọc và ngang. Rất tự nhiên cho những trường hợp đó.

Tôi chuyển sang Grid khi cần quản lý cấu trúc tổng thể trang, hoặc khi có layout hai chiều như card grid mà tôi quan tâm đến cả chiều rộng cột lẫn chiều cao hàng. Grid cũng tỏa sáng khi items cần span qua nhiều cột hay hàng.

Quy tắc đơn giản tôi hay dùng: nếu nghĩ theo một hướng thì Flexbox. Nếu nghĩ theo cả hàng lẫn cột cùng lúc thì Grid."

---

**Q2 (Mid): "How would you build a responsive card grid that wraps automatically without any media queries?"**

**Model Answer:**
"My go-to for this is CSS Grid with `repeat(auto-fit, minmax())`. Something like:

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
}
```

What this does: `auto-fit` creates as many columns as will fit. Each column is at least 280px wide, and they all grow equally to fill the available space. As the viewport shrinks, columns automatically drop to the next row. No media queries needed.

The subtle difference between `auto-fit` and `auto-fill` is that `auto-fit` collapses empty columns to zero, so if you only have two cards, they'll stretch to fill the row. `auto-fill` would leave ghost columns. For card grids I almost always want `auto-fit`."

**Trả lời (Tiếng Việt):**
"Cách tôi hay dùng nhất là CSS Grid với `repeat(auto-fit, minmax())`. Kiểu như:

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
}
```

Cơ chế hoạt động: `auto-fit` tạo bao nhiêu cột vừa khít. Mỗi cột rộng tối thiểu 280px và chúng đều grow đều để lấp đầy không gian. Khi viewport co lại, các cột tự động xuống hàng. Không cần media query.

Sự khác biệt tinh tế giữa `auto-fit` và `auto-fill` là `auto-fit` thu gọn các cột rỗng về không, nên nếu chỉ có hai card, chúng sẽ kéo dãn lấp đầy hàng. `auto-fill` để lại các cột ma. Với card grid tôi hầu như luôn dùng `auto-fit`."

---

**Q3 (Mid): "How would you implement a classic Holy Grail layout — header, footer, sidebar, main content — using CSS Grid?"**

**Model Answer:**
"Grid is perfect for this. I'd use `grid-template-areas` because it makes the layout very readable:

```css
.page {
  display: grid;
  grid-template-areas:
    'header  header'
    'sidebar main'
    'footer  footer';
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

header { grid-area: header; }
aside  { grid-area: sidebar; }
main   { grid-area: main; }
footer { grid-area: footer; }
```

Then for mobile I'd just redefine the template areas in a media query to stack everything into one column. The named areas make it really clear at a glance what the layout looks like."

**Trả lời (Tiếng Việt):**
"Grid rất thích hợp cho layout này. Tôi dùng `grid-template-areas` vì nó giúp layout rất dễ đọc:

```css
.page {
  display: grid;
  grid-template-areas:
    'header  header'
    'sidebar main'
    'footer  footer';
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

header { grid-area: header; }
aside  { grid-area: sidebar; }
main   { grid-area: main; }
footer { grid-area: footer; }
```

Sau đó trên mobile tôi chỉ cần redefine template areas trong media query để stack mọi thứ thành một cột. Các named areas giúp nhìn vào là biết ngay layout trông như thế nào."

---

**Q4 (Senior): "When would you mix Flexbox and Grid in the same component?"**

**Model Answer:**
"All the time, actually. They're not competitors — they complement each other really well.

A common pattern: I use Grid for the overall page structure and section layouts, because I need two-dimensional control. Then inside a card component, I use Flexbox to align the icon, title, and text along one axis. Or in a navbar, I use Flexbox to handle the spacing between logo, links, and the action button — because it's all one row and I want `space-between` semantics.

The mental model I have: Grid manages the big picture layout, Flexbox handles the internal arrangement of a component's children. The moment I catch myself fighting one tool to do something the other is built for, I switch."

**Trả lời (Tiếng Việt):**
"Rất thường xuyên, thực ra. Chúng không phải đối thủ của nhau — chúng bổ sung cho nhau rất tốt.

Pattern phổ biến: tôi dùng Grid cho cấu trúc tổng thể trang và section layouts, vì cần kiểm soát hai chiều. Còn bên trong card component, tôi dùng Flexbox để căn chỉnh icon, tiêu đề, và text theo một trục. Hoặc trong navbar, tôi dùng Flexbox để xử lý khoảng cách giữa logo, links, và action button — vì tất cả chỉ một hàng và tôi muốn semantics `space-between`.

Mental model của tôi: Grid quản lý big picture layout, Flexbox xử lý sắp xếp nội bộ trong children của một component. Khi nào thấy đang 'chiến đấu' với một tool để làm điều mà tool kia được sinh ra để làm, tôi chuyển sang."

---

---

## 3. CSS Grid

**Level:** [Mid] / [Senior]

**EN:** Explain CSS Grid. What is the difference between `auto-fill` and `auto-fit`? How do you use `minmax()` and `grid-template-areas`?

**VI:** Giải thích CSS Grid. Sự khác biệt giữa `auto-fill` và `auto-fit` là gì? Cách dùng `minmax()` và `grid-template-areas`?

**Trả lời:**

CSS Grid là hai chiều (2D) — layout theo cả hàng lẫn cột cùng lúc. Phù hợp cho page layouts phức tạp.

```css
/* Cơ bản */
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 1fr; /* 3 cột */
  grid-template-rows: auto;
  gap: 1rem;
}

/* auto-fill vs auto-fit */
/*
  auto-fill: tạo nhiều cột nhất có thể, KỂ CẢ cột rỗng (empty tracks)
  auto-fit: tạo nhiều cột nhất có thể, CÁC cột rỗng sẽ bị collapse về 0
  => Khi có ít items, auto-fit sẽ kéo dãn items, auto-fill giữ khoảng trống
*/
.auto-fill {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}

.auto-fit {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

/* minmax(min, max): cột không nhỏ hơn min, không lớn hơn max */
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  /* Tự động tạo số cột phù hợp, mỗi cột min 250px max 1fr */
}

/* grid-template-areas: layout trực quan dùng tên */
.page-layout {
  display: grid;
  grid-template-areas:
    "header  header  header"
    "sidebar main    main"
    "sidebar footer  footer";
  grid-template-columns: 240px 1fr 1fr;
  grid-template-rows: 60px 1fr 80px;
  min-height: 100vh;
  gap: 0;
}

.page-header { grid-area: header; }
.page-sidebar { grid-area: sidebar; }
.page-main   { grid-area: main; }
.page-footer { grid-area: footer; }

/* Span nhiều ô */
.featured-item {
  grid-column: 1 / 3;       /* span từ line 1 đến line 3 */
  grid-row: span 2;          /* span 2 hàng */
}

/* Responsive với media query */
@media (max-width: 768px) {
  .page-layout {
    grid-template-areas:
      "header"
      "main"
      "sidebar"
      "footer";
    grid-template-columns: 1fr;
    grid-template-rows: auto;
  }
}
```

---

## 4. Positioning

**Level:** [Junior] / [Mid]

**EN:** Explain the 5 CSS position values: `static`, `relative`, `absolute`, `fixed`, `sticky`. When do you use each?

**VI:** Giải thích 5 giá trị của CSS position. Khi nào dùng từng loại?

**Trả lời:**

| Giá trị | Nằm trong flow? | Tham chiếu | Khi nào dùng |
|---------|----------------|------------|-------------|
| `static` | Có | N/A | Mặc định |
| `relative` | Có | Chính nó | Dịch chuyển nhẹ, tạo stacking context |
| `absolute` | Không | Ancestor có `position != static` | Tooltips, dropdowns, overlays |
| `fixed` | Không | Viewport | Navbar cố định, back-to-top button |
| `sticky` | Có (đến khi dính) | Scroll container | Table header, section navigation |

```css
/* relative: dịch chuyển so với vị trí bình thường, KHÔNG ảnh hưởng flow */
.relative-example {
  position: relative;
  top: 10px;  /* dịch xuống 10px so với vị trí gốc */
  left: 20px;
}

/* absolute: thoát khỏi flow, định vị theo ancestor gần nhất có position */
.parent {
  position: relative; /* tạo positioning context */
  width: 300px;
  height: 200px;
}

.badge {
  position: absolute;
  top: -8px;
  right: -8px;
  /* định vị theo .parent */
}

/* fixed: cố định theo viewport */
.sticky-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
}

/* sticky: ở trong flow, trở thành fixed khi scroll đến ngưỡng offset */
.section-heading {
  position: sticky;
  top: 60px; /* dính ở 60px từ top */
  background: white;
  z-index: 10;
}

/* Tooltip pattern dùng relative + absolute */
.tooltip-container {
  position: relative;
  display: inline-block;
}

.tooltip {
  position: absolute;
  bottom: calc(100% + 8px); /* phía trên element */
  left: 50%;
  transform: translateX(-50%); /* căn giữa theo chiều ngang */
  white-space: nowrap;
  pointer-events: none;
}
```

---

## 5. CSS Specificity

**Level:** [Junior] / [Mid]

**EN:** How is CSS specificity calculated? What are "specificity wars" and how do you avoid them?

**VI:** CSS specificity được tính như thế nào? "Specificity wars" là gì và làm thế nào để tránh?

**Trả lời:**

Specificity được tính theo hệ 3 số `(A, B, C)`:
- **A**: inline styles → 1000 điểm
- **B**: ID selectors `#id` → 100 điểm
- **C**: class `.class`, attribute `[attr]`, pseudo-class `:hover` → 10 điểm
- **D**: element `div`, pseudo-element `::before` → 1 điểm
- `!important` ghi đè tất cả (nên tránh)

```css
/* Specificity calculation examples */
p                      /* 0,0,0,1 */
.text                  /* 0,0,1,0 */
#title                 /* 0,1,0,0 */
p.text                 /* 0,0,1,1 */
#nav .item:hover       /* 0,1,1,1 */
style=""               /* 1,0,0,0 */

/* Specificity wars — vấn đề phổ biến */
/* File A viết: */
#sidebar .widget .title { color: blue; }  /* 0,1,2,0 */

/* File B muốn override nhưng phải leo thang: */
#sidebar .widget .title.special { color: red; } /* 0,1,3,0 */
/* Rồi lại cần override tiếp... → specificity leo thang không kiểm soát */

/* Giải pháp: dùng class-based selectors thấp specificity */
/* BEM giúp tránh vấn đề này */
.widget__title { color: blue; }           /* 0,0,1,0 */
.widget__title--special { color: red; }   /* 0,0,1,0 — cùng specificity, rule sau thắng */

/* Nếu PHẢI override (third-party library), dùng :where() để specificity = 0 */
:where(.widget) .title { color: blue; }   /* 0,0,1,0 thay vì cao hơn */

/* :is() giữ specificity cao nhất trong danh sách */
:is(#id, .class) p { }  /* specificity = 1,0,0,1 (do #id) */

/* !important — chỉ dùng khi thực sự cần (utility classes, override 3rd party) */
.u-hidden { display: none !important; }
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "How does CSS specificity work? Can you give me an example of a specificity conflict?"**

**Model Answer:**
"Specificity is basically CSS's way of deciding which rule wins when two rules target the same element. It's calculated with three tiers: IDs are the most powerful, then classes and pseudo-classes, then plain elements.

A quick way I think about it: inline styles beat IDs, IDs beat classes, classes beat elements. And `!important` overrides everything — but that's a last resort.

So if I have `#nav .item { color: blue }` and `.item { color: red }`, the first rule wins because it has an ID in the selector, which gives it a higher score even though the second rule comes later in the file.

The problem people run into is what's called 'specificity wars' — one dev writes a high-specificity rule to override another, then the next person has to go even higher. It gets out of hand quickly."

**Trả lời (Tiếng Việt):**
"Specificity về cơ bản là cách CSS quyết định rule nào thắng khi hai rules cùng target một element. Nó được tính theo ba bậc: IDs mạnh nhất, rồi đến classes và pseudo-classes, rồi đến plain elements.

Cách tôi hay nghĩ: inline styles thắng IDs, IDs thắng classes, classes thắng elements. Và `!important` override tất cả — nhưng đó là lựa chọn cuối cùng.

Ví dụ, nếu có `#nav .item { color: blue }` và `.item { color: red }`, rule đầu thắng vì nó có một ID trong selector, cho nó điểm cao hơn dù rule thứ hai xuất hiện sau trong file.

Vấn đề người ta hay gặp là cái gọi là 'specificity wars' — một dev viết rule specificity cao để override rule khác, rồi người tiếp theo phải đi còn cao hơn. Rất nhanh mất kiểm soát."

---

**Q2 (Mid): "What's the cascade, and how is it different from specificity?"**

**Model Answer:**
"They're related but distinct. Specificity is the calculation engine — it determines which rule is 'stronger' based on the selectors used.

The cascade is the broader algorithm the browser uses to resolve conflicts. It considers origin — whether the style comes from the browser's default stylesheet, the user's preferences, or the author's stylesheet. Author styles beat user agent styles by default. Then within the same origin, it looks at specificity. And if specificity ties, the rule that comes later in the source wins.

So the full decision order is: importance (`!important`) → origin → specificity → source order. Specificity is just one piece of that."

**Trả lời (Tiếng Việt):**
"Chúng liên quan nhưng khác nhau. Specificity là engine tính toán — nó xác định rule nào 'mạnh hơn' dựa trên selectors được dùng.

Cascade là thuật toán tổng quát hơn mà browser dùng để giải quyết conflicts. Nó xét đến origin — style đến từ default stylesheet của browser, preferences của user, hay stylesheet của tác giả. Author styles thắng user agent styles theo mặc định. Rồi trong cùng origin, nó nhìn vào specificity. Và nếu specificity hòa, rule xuất hiện sau trong source thắng.

Vậy thứ tự quyết định đầy đủ là: importance (`!important`) → origin → specificity → source order. Specificity chỉ là một phần trong đó."

---

**Q3 (Mid): "How do you avoid specificity wars in a real codebase?"**

**Model Answer:**
"The main strategy is to keep specificity consistently low. If most of your selectors are single class selectors, you almost never have a conflict — and when you do, source order resolves it cleanly.

BEM helps a lot here because you end up with classes like `.card__title` that have specificity of just one class. No nesting needed.

Two newer CSS features I really like for this: `:where()` — it wraps a selector but gives it zero specificity, which is great for base styles or utility overrides. And `:is()` for grouping selectors without climbing specificity.

I also try to avoid ID selectors in stylesheets entirely. IDs are fine in HTML for anchoring or JavaScript, but in CSS they make specificity management much harder."

**Trả lời (Tiếng Việt):**
"Chiến lược chính là giữ specificity thấp nhất quán. Nếu hầu hết selectors của bạn là single class selectors, bạn hầu như không bao giờ có conflict — và khi có, source order giải quyết sạch sẽ.

BEM giúp rất nhiều ở đây vì bạn sẽ có các class như `.card__title` với specificity chỉ là một class. Không cần nesting.

Hai CSS features mới tôi rất thích cho việc này: `:where()` — nó bọc selector nhưng cho nó zero specificity, rất hay cho base styles hoặc utility overrides. Và `:is()` để group selectors mà không leo thang specificity.

Tôi cũng cố tránh ID selectors trong stylesheets hoàn toàn. IDs ổn trong HTML để anchor hoặc JavaScript, nhưng trong CSS chúng làm việc quản lý specificity khó hơn nhiều."

---

**Q4 (Senior): "When is using `!important` actually acceptable?"**

**Model Answer:**
"There are a few legitimate cases. Utility classes are the classic one — if you have a `.u-hidden { display: none !important }`, the whole point is that it should always win regardless of context. That's intentional and documented.

The other case is overriding third-party styles. If you're integrating a library that uses very high specificity selectors and you can't touch its CSS, `!important` may be the only practical option.

What I try to avoid is using `!important` as a quick fix when I don't understand why a rule isn't applying. That's almost always a specificity problem that should be solved by refactoring the selectors, not escalating. And if I do use it, I always leave a comment explaining why."

**Trả lời (Tiếng Việt):**
"Có vài trường hợp hợp lệ. Utility classes là ví dụ điển hình — nếu có `.u-hidden { display: none !important }`, ý đồ là nó phải luôn thắng bất kể context. Đó là cố ý và được documented.

Trường hợp khác là override third-party styles. Nếu bạn đang tích hợp một thư viện dùng selectors specificity rất cao và không thể chạm vào CSS của nó, `!important` có thể là lựa chọn thực tế duy nhất.

Điều tôi cố tránh là dùng `!important` như một quick fix khi không hiểu tại sao rule không apply. Đó hầu như luôn là vấn đề specificity nên được giải quyết bằng cách refactor selectors, không phải leo thang. Và nếu tôi có dùng, tôi luôn để lại comment giải thích tại sao."

---

---

## 6. CSS Custom Properties (Variables)

**Level:** [Mid]

**EN:** What are CSS custom properties? How do they differ from Sass variables? How do you use them for dynamic theming?

**VI:** CSS custom properties là gì? Chúng khác với biến Sass như thế nào? Cách dùng để tạo dynamic theming?

**Trả lời:**

| | CSS Custom Properties | Sass Variables |
|--|----------------------|---------------|
| Thời điểm resolve | Runtime (browser) | Compile time |
| Có thể thay đổi | Có (JS, media query) | Không |
| Cascade & inherit | Có | Không |
| DevTools | Hiển thị giá trị thực | Không thấy được |

```css
/* Khai báo: dùng dấu -- ở trước tên */
:root {
  /* Design tokens */
  --color-primary: #3b82f6;
  --color-primary-dark: #1d4ed8;
  --color-text: #111827;
  --color-bg: #ffffff;

  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 2rem;

  --radius-md: 0.5rem;
  --font-size-base: 1rem;
}

/* Sử dụng với var() */
.button {
  background: var(--color-primary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
  color: white;
}

/* Fallback value */
.card {
  color: var(--card-text, var(--color-text, #000)); /* nested fallback */
}

/* Dynamic theming — dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-text: #f9fafb;
    --color-bg: #111827;
    --color-primary: #60a5fa;
  }
}

/* Hoặc class-based theming (controlled bởi JS) */
[data-theme="dark"] {
  --color-text: #f9fafb;
  --color-bg: #111827;
}

/* Thay đổi bằng JavaScript */
/* document.documentElement.style.setProperty('--color-primary', '#ef4444'); */

/* Component-level override — CSS variables cascade */
.card {
  --color-primary: #10b981; /* override chỉ trong scope của .card */
}

.card .button {
  background: var(--color-primary); /* dùng giá trị #10b981 */
}

/* Tính toán với calc() */
:root {
  --base-font-size: 16px;
}

h1 {
  font-size: calc(var(--base-font-size) * 2.5); /* 40px */
}
```

---

## 7. SASS/SCSS

**Level:** [Mid]

**EN:** Explain the main SCSS features: variables, nesting, mixins, functions, `@extend`, partials, and the difference between `@use` and `@import`.

**VI:** Giải thích các tính năng chính của SCSS: variables, nesting, mixins, functions, `@extend`, partials, và sự khác biệt giữa `@use` và `@import`.

**Trả lời:**

```scss
// ============================================
// 1. VARIABLES — compile-time, có kiểu dữ liệu
// ============================================
$color-primary: #3b82f6;
$spacing-unit: 8px;
$font-stack: 'Inter', sans-serif;

// ============================================
// 2. NESTING — phản ánh HTML structure
// ============================================
.nav {
  display: flex;
  gap: $spacing-unit * 2;

  &__item {         // & = parent selector → .nav__item
    padding: $spacing-unit;
    color: #333;

    &:hover {       // .nav__item:hover
      color: $color-primary;
    }

    &--active {     // .nav__item--active (BEM modifier)
      font-weight: 700;
      color: $color-primary;
    }
  }
}

// ============================================
// 3. MIXINS — reusable blocks, có thể nhận arguments
// ============================================
@mixin flex-center($direction: row) {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: $direction;
}

@mixin responsive($breakpoint) {
  @if $breakpoint == 'md' {
    @media (min-width: 768px) { @content; }
  } @else if $breakpoint == 'lg' {
    @media (min-width: 1024px) { @content; }
  }
}

@mixin truncate($lines: 1) {
  @if $lines == 1 {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  } @else {
    display: -webkit-box;
    -webkit-line-clamp: $lines;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
}

.hero {
  @include flex-center(column);
  min-height: 100vh;

  @include responsive('md') {
    flex-direction: row;
  }
}

.card__title {
  @include truncate(2);
}

// ============================================
// 4. FUNCTIONS — trả về một giá trị
// ============================================
@function rem($px, $base: 16) {
  @return #{$px / $base}rem;
}

@function spacing($multiplier) {
  @return $spacing-unit * $multiplier;
}

.heading {
  font-size: rem(32);         // 2rem
  margin-bottom: spacing(3);  // 24px
}

// ============================================
// 5. @EXTEND — chia sẻ styles (dùng cẩn thận)
// ============================================
%button-base {  // placeholder selector, không compile nếu không được extend
  display: inline-flex;
  align-items: center;
  padding: spacing(1) spacing(2);
  border-radius: 4px;
  cursor: pointer;
  border: none;
  font-size: rem(14);
}

.button-primary {
  @extend %button-base;
  background: $color-primary;
  color: white;
}

.button-secondary {
  @extend %button-base;
  background: transparent;
  border: 1px solid $color-primary;
  color: $color-primary;
}

// ============================================
// 6. PARTIALS & @USE vs @IMPORT
// ============================================
// @import (cũ, deprecated): load toàn bộ vào global scope
//   - Gây ra namespace pollution
//   - Variables có thể bị override
//   - Không có encapsulation

// @use (mới, khuyến nghị): module system
//   - Mỗi file là một module độc lập
//   - Dùng namespace để tránh xung đột
//   - Chỉ load một lần dù import nhiều nơi

// _variables.scss, _mixins.scss (prefix _ = partial, không compile thành file riêng)

// main.scss
// @use 'variables' as vars;
// @use 'mixins' as m;
//
// .component {
//   color: vars.$color-primary;  // phải dùng namespace
//   @include m.flex-center();
// }
//
// @use 'variables' as *;  // dùng * để không cần namespace (cẩn thận)
```

---

## 8. CSS Modules

**Level:** [Mid]

**EN:** How do CSS Modules work? Explain `:local`, `:global`, `composes`. Why is it better than global CSS?

**VI:** CSS Modules hoạt động như thế nào? Giải thích `:local`, `:global`, `composes`. Tại sao nó tốt hơn global CSS?

**Trả lời:**

CSS Modules là một hệ thống tự động **scope** tên class theo từng file, tránh xung đột tên toàn cục. Build tool (Vite, Webpack) sẽ transform tên class thành unique hash.

```css
/* Button.module.css */

/* Mặc định tất cả selectors đều là :local */
.button {
  /* Được transform thành: .Button_button__xK3mP (ví dụ) */
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  border-radius: 4px;
}

.primary {
  background: #3b82f6;
  color: white;
}

/* :global — opt-out khỏi scoping, áp dụng global */
:global(.third-party-class) {
  margin: 0;
}

:global {
  /* Tất cả selectors bên trong đều global */
  body { margin: 0; }
  * { box-sizing: border-box; }
}

/* composes — kế thừa styles từ class khác (tương tự @extend trong SCSS) */
.button-primary {
  composes: button;         /* kế thừa từ .button trong file này */
  background: #3b82f6;
  color: white;
}

.button-danger {
  composes: button;
  composes: danger from './colors.module.css'; /* kế thừa từ file khác */
  background: #ef4444;
}
```

```tsx
// Button.tsx — import như object
import styles from './Button.module.css';

function Button({ variant = 'default', children }) {
  return (
    <button
      className={`${styles.button} ${variant === 'primary' ? styles.primary : ''}`}
    >
      {children}
    </button>
  );
}

// Dùng với clsx/classnames
import clsx from 'clsx';

function Button({ variant, disabled, children }) {
  return (
    <button
      className={clsx(styles.button, {
        [styles.primary]: variant === 'primary',
        [styles.disabled]: disabled,
      })}
    >
      {children}
    </button>
  );
}
```

**Tại sao tốt hơn global CSS:**
1. **No naming conflicts** — class names được hash, tránh `.button` của component A ghi đè `.button` của component B
2. **Dead code elimination** — dễ biết class nào không dùng
3. **Explicit dependencies** — import rõ ràng thay vì implicit global
4. **Refactoring safe** — đổi tên class không lo ảnh hưởng nơi khác

---

## 9. CSS-in-JS vs CSS Modules vs Tailwind

**Level:** [Senior]

**EN:** Compare styled-components (CSS-in-JS), CSS Modules, and Tailwind CSS. What are the tradeoffs of each approach?

**VI:** So sánh styled-components, CSS Modules, và Tailwind CSS. Tradeoffs của từng cách?

**Trả lời:**

| | styled-components | CSS Modules | Tailwind CSS |
|--|------------------|-------------|--------------|
| Runtime overhead | Có (JS inject CSS) | Không | Không |
| Bundle size | Lớn hơn | Nhỏ | Nhỏ (purged) |
| Dynamic styles | Rất dễ (props) | Hạn chế | Hạn chế |
| TypeScript DX | Tốt với themeProvider | Tốt | Tốt với intellisense |
| Learning curve | Trung bình | Thấp | Trung bình |
| Co-location | Cao nhất | Cao | Cao |

```tsx
// ===========================
// 1. styled-components (CSS-in-JS)
// ===========================
import styled from 'styled-components';

const Button = styled.button<{ $variant?: 'primary' | 'secondary' }>`
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 0.875rem;

  /* Dynamic styles dựa vào props — điểm mạnh nhất */
  background: ${({ $variant }) =>
    $variant === 'primary' ? '#3b82f6' : 'transparent'};
  color: ${({ $variant }) =>
    $variant === 'primary' ? 'white' : '#3b82f6'};

  &:hover {
    opacity: 0.9;
  }
`;

// Usage
<Button $variant="primary">Submit</Button>

// ===========================
// 2. CSS Modules
// ===========================
// Button.module.css
.button { padding: 0.5rem 1rem; }
.primary { background: #3b82f6; color: white; }
.secondary { background: transparent; color: #3b82f6; }

// Button.tsx
import styles from './Button.module.css';
<button className={`${styles.button} ${styles[variant]}`}>Submit</button>

// ===========================
// 3. Tailwind CSS
// ===========================
// Utility classes — không cần viết CSS
function Button({ variant = 'primary', children }) {
  const variantClasses = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600',
    secondary: 'bg-transparent text-blue-500 border border-blue-500 hover:bg-blue-50',
  };

  return (
    <button
      className={`inline-flex items-center px-4 py-2 rounded text-sm font-medium
                  transition-colors ${variantClasses[variant]}`}
    >
      {children}
    </button>
  );
}

// Tailwind v4 với cva (class-variance-authority) — best practice
import { cva } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center px-4 py-2 rounded text-sm font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-blue-500 text-white hover:bg-blue-600',
        secondary: 'border border-blue-500 text-blue-500 hover:bg-blue-50',
        ghost: 'text-gray-600 hover:bg-gray-100',
      },
      size: {
        sm: 'px-3 py-1.5 text-xs',
        md: 'px-4 py-2 text-sm',
        lg: 'px-6 py-3 text-base',
      },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
);
```

**Khi nào chọn gì:**
- **styled-components**: Khi cần dynamic styles phức tạp theo props, theme phức tạp, design system với nhiều variants
- **CSS Modules**: Khi muốn CSS thuần, không overhead runtime, team quen CSS truyền thống
- **Tailwind**: Khi muốn tốc độ phát triển nhanh, design system dựa trên tokens, ít context switching

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "Can you walk me through the tradeoffs between CSS Modules, styled-components, and Tailwind? When would you pick each one?"**

**Model Answer:**
"Sure. They solve the same core problem — avoiding global CSS conflicts — but in very different ways.

CSS Modules give you plain CSS files where class names get hashed automatically at build time. Zero runtime overhead, works with any CSS feature, and it's familiar to anyone who knows CSS. The tradeoff is that dynamic styling based on props requires manually constructing class name strings, which gets verbose.

styled-components is CSS-in-JS — you write CSS inside template literals in your JavaScript. The big win is that dynamic styles based on props are completely natural. The cost is a JavaScript runtime that generates and injects stylesheets at render time, which adds bundle weight and can cause style recalculation issues at scale.

Tailwind is utility-first — you compose styles using predefined class names directly in the HTML. The developer experience is fast once you know the utility names, and the production CSS is tiny because unused utilities get purged. The downside is that class names in JSX get long and there's a learning curve.

My personal default is Tailwind for new projects because of the speed and the small production bundle. CSS Modules for teams that prefer traditional CSS. styled-components when the design system needs a lot of dynamic prop-driven variants."

**Trả lời (Tiếng Việt):**
"Được. Chúng giải quyết cùng một vấn đề cốt lõi — tránh global CSS conflicts — nhưng theo những cách rất khác nhau.

CSS Modules cho bạn plain CSS files mà class names được hash tự động tại build time. Zero runtime overhead, hoạt động với bất kỳ CSS feature nào, và quen thuộc với bất kỳ ai biết CSS. Tradeoff là dynamic styling dựa trên props đòi hỏi phải tự construct class name strings, trở nên verbose.

styled-components là CSS-in-JS — bạn viết CSS bên trong template literals trong JavaScript. Điểm mạnh lớn là dynamic styles dựa trên props hoàn toàn tự nhiên. Chi phí là JavaScript runtime sinh và inject stylesheets tại render time, thêm bundle weight và có thể gây style recalculation issues ở scale.

Tailwind là utility-first — bạn compose styles dùng predefined class names trực tiếp trong HTML. Developer experience nhanh khi đã quen utility names, và production CSS rất nhỏ vì unused utilities được purge. Nhược điểm là class names trong JSX dài và có learning curve.

Mặc định cá nhân của tôi là Tailwind cho các project mới vì tốc độ và production bundle nhỏ. CSS Modules cho các team thích CSS truyền thống. styled-components khi design system cần nhiều dynamic prop-driven variants."

---

**Q2 (Senior): "What are the performance implications of CSS-in-JS at scale?"**

**Model Answer:**
"The main concern is runtime cost. Libraries like styled-components and Emotion generate stylesheets dynamically in JavaScript. Every time a component renders with new props, it may produce new CSS rules and inject them into the document.

With a large number of unique prop combinations, the stylesheet can grow significantly. And because this happens in JavaScript during rendering, it adds to the work the main thread has to do — which can hurt time-to-interactive on slower devices.

Server-side rendering adds another dimension. CSS-in-JS requires a critical CSS extraction step during SSR to avoid flash of unstyled content. That's extra complexity in the server render pipeline.

The modern alternatives that try to address this are zero-runtime CSS-in-JS tools like vanilla-extract, Linaria, or Panda CSS. They generate static CSS at build time using TypeScript or tagged template literals, so you get the developer experience of CSS-in-JS without the runtime cost. Tailwind is essentially a zero-runtime approach too."

**Trả lời (Tiếng Việt):**
"Mối lo chính là runtime cost. Các thư viện như styled-components và Emotion sinh stylesheets động trong JavaScript. Mỗi lần component render với props mới, nó có thể tạo ra CSS rules mới và inject chúng vào document.

Với lượng lớn prop combinations unique, stylesheet có thể grow đáng kể. Và vì điều này xảy ra trong JavaScript trong quá trình rendering, nó thêm việc cho main thread phải làm — có thể ảnh hưởng time-to-interactive trên thiết bị chậm hơn.

Server-side rendering thêm một chiều nữa. CSS-in-JS đòi hỏi một bước extraction critical CSS trong quá trình SSR để tránh flash of unstyled content. Đó là thêm complexity trong server render pipeline.

Các alternatives hiện đại cố giải quyết điều này là zero-runtime CSS-in-JS tools như vanilla-extract, Linaria, hoặc Panda CSS. Chúng sinh static CSS tại build time dùng TypeScript hoặc tagged template literals, nên bạn có developer experience của CSS-in-JS mà không có runtime cost. Tailwind về bản chất cũng là zero-runtime approach."

---

**Q3 (Senior): "If you inherited a large codebase with inconsistent CSS — some global stylesheets, some CSS Modules, some inline styles — how would you approach fixing it?"**

**Model Answer:**
"Incrementally, not all at once. A big rewrite of all the CSS at once is a high-risk move that rarely ends well.

I'd first audit what's there — identify where the actual pain points are. Usually it's specific components where styles are leaking or overrides are fighting. I'd prioritize fixing those rather than trying to standardize everything immediately.

For new components, I'd establish a clear standard — probably CSS Modules or Tailwind depending on the team's preference — and enforce it with a linter. Existing components I'd migrate when they need to be changed anyway, not as a separate task.

Global stylesheets I'd leave mostly in place initially, just making sure they're organized and documented. I'd focus on stopping the spread of bad patterns in new code before trying to erase the history in old code."

**Trả lời (Tiếng Việt):**
"Từng bước, không phải tất cả cùng một lúc. Rewrite toàn bộ CSS cùng lúc là một move rủi ro cao và hiếm khi kết thúc tốt.

Đầu tiên tôi sẽ audit những gì đang có — xác định đâu là điểm đau thực sự. Thường là các components cụ thể mà styles đang bị leak hoặc overrides đang chiến đấu nhau. Tôi ưu tiên fix những cái đó thay vì cố chuẩn hóa mọi thứ ngay lập tức.

Với components mới, tôi sẽ thiết lập một chuẩn rõ ràng — có thể CSS Modules hoặc Tailwind tùy preference của team — và enforce bằng linter. Components cũ tôi sẽ migrate khi chúng cần thay đổi anyway, không phải như một task riêng biệt.

Global stylesheets tôi sẽ để nguyên ban đầu, chỉ đảm bảo chúng được tổ chức và documented. Tôi tập trung vào việc ngăn lan rộng các bad patterns trong code mới trước khi cố xóa lịch sử trong code cũ."

---

---

## 10. Responsive Design

**Level:** [Mid]

**EN:** Explain mobile-first design, breakpoint strategy, `clamp()`, and container queries. How do they differ from media queries?

**VI:** Giải thích mobile-first design, chiến lược breakpoints, `clamp()`, và container queries.

**Trả lời:**

**Mobile-first**: Viết styles cho mobile trước, dùng `min-width` để mở rộng lên. Ngược lại với desktop-first (`max-width`).

```css
/* Mobile-first approach */
.card {
  /* Mobile styles mặc định */
  display: flex;
  flex-direction: column;
  padding: 1rem;
  font-size: 0.875rem;
}

@media (min-width: 640px) {   /* sm */
  .card { padding: 1.5rem; }
}

@media (min-width: 768px) {   /* md */
  .card {
    flex-direction: row;
    font-size: 1rem;
  }
}

@media (min-width: 1024px) {  /* lg */
  .card { padding: 2rem; }
}

/* ================================
   clamp(min, preferred, max)
   Responsive typography không cần media queries
================================ */
h1 {
  /* min: 1.5rem, max: 3rem, tỉ lệ với viewport width */
  font-size: clamp(1.5rem, 4vw, 3rem);
}

.container {
  /* Padding responsive: min 1rem, max 3rem */
  padding: clamp(1rem, 5vw, 3rem);
}

/* Fluid spacing */
.section {
  margin-block: clamp(2rem, 8vw, 6rem);
}

/* ================================
   Container Queries (CSS @container)
   Style dựa trên kích thước CONTAINER, không phải viewport
   => Thích hợp hơn cho component-driven development
================================ */

/* 1. Đặt tên container */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* 2. Style dựa trên container width */
@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
    gap: 1rem;
  }

  .card__image {
    width: 200px;
    flex-shrink: 0;
  }
}

/* Tại sao container queries tốt hơn media queries cho components:
   - Card có thể ở sidebar (hẹp) hoặc main area (rộng) — cùng 1 viewport width
   - Media query không phân biệt được context
   - Container query giải quyết vấn đề này hoàn toàn
*/

/* Breakpoint strategy — Tailwind-inspired */
:root {
  --bp-sm: 640px;
  --bp-md: 768px;
  --bp-lg: 1024px;
  --bp-xl: 1280px;
  --bp-2xl: 1536px;
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "What is mobile-first design and why is it preferred over desktop-first?"**

**Model Answer:**
"Mobile-first means you write your baseline CSS for the smallest screens and use `min-width` media queries to progressively enhance the layout as the viewport gets larger. Desktop-first is the opposite — you start with a wide layout and use `max-width` queries to scale down.

The reason mobile-first is preferred is partly practical and partly philosophical. Practically, CSS works by inheriting and overriding — if you start small, you're adding styles as you go, which tends to be simpler than removing or resetting styles for smaller screens.

Philosophically, it forces you to prioritize content. On mobile you can't hide everything behind hamburger menus or collapse half the page — you have to decide what actually matters. That discipline usually makes for better UX on all screen sizes."

**Trả lời (Tiếng Việt):**
"Mobile-first nghĩa là bạn viết CSS baseline cho màn hình nhỏ nhất và dùng `min-width` media queries để progressive enhance layout khi viewport lớn hơn. Desktop-first ngược lại — bạn bắt đầu với wide layout và dùng `max-width` queries để scale down.

Lý do mobile-first được ưa dùng là vừa thực tế vừa về triết lý. Về thực tế, CSS hoạt động bằng cách kế thừa và override — nếu bắt đầu nhỏ, bạn thêm styles khi đi, thường đơn giản hơn là xóa hay reset styles cho màn hình nhỏ hơn.

Về triết lý, nó buộc bạn phải ưu tiên content. Trên mobile bạn không thể ẩn mọi thứ sau hamburger menus hay collapse nửa trang — bạn phải quyết định điều gì thực sự quan trọng. Kỷ luật đó thường tạo ra UX tốt hơn trên mọi kích thước màn hình."

---

**Q2 (Mid): "What is the difference between media queries and container queries? When would you use each?"**

**Model Answer:**
"Media queries respond to the viewport size. Container queries respond to the size of a parent element. That distinction is huge for component-driven development.

The problem with media queries for components is that a card can appear in a wide main content area or a narrow sidebar depending on where you place it — same viewport, completely different context. With media queries, there's no way for the card to know which situation it's in.

Container queries solve this. You mark the parent as a container with `container-type: inline-size`, and the card styles itself based on how wide that container actually is. The same card component can be horizontal when it has room and vertical when it's squeezed — without any media query hacks.

I use media queries for page-level layout decisions and global typography scaling. I use container queries for reusable components that need to be context-aware."

**Trả lời (Tiếng Việt):**
"Media queries phản hồi kích thước viewport. Container queries phản hồi kích thước của một parent element. Sự khác biệt đó rất lớn cho component-driven development.

Vấn đề với media queries cho components là một card có thể xuất hiện ở một main content area rộng hoặc một sidebar hẹp tùy vào nơi bạn đặt nó — cùng viewport, context hoàn toàn khác nhau. Với media queries, không có cách nào để card biết mình đang trong tình huống nào.

Container queries giải quyết điều này. Bạn đánh dấu parent là container với `container-type: inline-size`, và card tự style dựa trên chiều rộng thực tế của container đó. Cùng một card component có thể nằm ngang khi có chỗ và đứng thẳng khi bị squeeze — không cần media query hacks.

Tôi dùng media queries cho các quyết định layout tầm trang và global typography scaling. Tôi dùng container queries cho các reusable components cần nhận biết context."

---

**Q3 (Mid): "How does `clamp()` work and how does it replace media queries for typography?"**

**Model Answer:**
"Clamp takes three values: a minimum, a preferred value, and a maximum. The preferred value is usually viewport-relative, like `4vw`. The browser picks the preferred value but clamps it between the min and max.

So `font-size: clamp(1rem, 4vw, 2.5rem)` means: never smaller than 1rem, never larger than 2.5rem, but scale smoothly with the viewport in between.

Before clamp you'd need a couple of media query breakpoints to handle typography scaling. With clamp it's one line and it scales continuously rather than jumping at breakpoints. I use it heavily for headings and section padding.

The limitation is you're tied to viewport units in the preferred value — you can't base it on a container's width. For that I'd still use media queries or container queries."

**Trả lời (Tiếng Việt):**
"Clamp nhận ba giá trị: minimum, preferred value, và maximum. Preferred value thường là viewport-relative, kiểu như `4vw`. Browser chọn preferred value nhưng clamp nó giữa min và max.

Vậy `font-size: clamp(1rem, 4vw, 2.5rem)` nghĩa là: không bao giờ nhỏ hơn 1rem, không bao giờ lớn hơn 2.5rem, nhưng scale mượt mà theo viewport ở giữa.

Trước clamp bạn cần vài media query breakpoints để xử lý typography scaling. Với clamp chỉ một dòng và nó scale liên tục thay vì nhảy tại breakpoints. Tôi dùng nó nhiều cho headings và section padding.

Hạn chế là bạn bị buộc vào viewport units trong preferred value — không thể base nó trên chiều rộng container. Cho điều đó tôi vẫn dùng media queries hoặc container queries."

---

**Q4 (Senior): "How do you decide on a breakpoint strategy for a project?"**

**Model Answer:**
"I try to base breakpoints on the content, not on specific device sizes. The device landscape is too fragmented to target individual screen widths reliably.

In practice I start with a mobile layout and widen the viewport until something looks broken or awkward — that's where I add a breakpoint. I don't pre-declare six breakpoints and force my content into them.

For the value of those breakpoints, I usually align with Tailwind's defaults — 640, 768, 1024, 1280 — because they're close enough to content breakpoints for most layouts, and they're familiar to any developer who's worked with modern CSS frameworks.

I also try to minimize the number of breakpoints. Between `clamp()` for fluid scaling, CSS Grid with `auto-fit/minmax` for adaptive layouts, and container queries for components, a lot of responsive behavior happens without any breakpoints at all."

**Trả lời (Tiếng Việt):**
"Tôi cố base breakpoints theo content, không phải theo device sizes cụ thể. Landscape thiết bị quá phân mảnh để target từng screen widths riêng lẻ một cách đáng tin.

Trong thực tế tôi bắt đầu với mobile layout và mở rộng viewport cho đến khi có gì đó trông gãy hoặc awkward — đó là nơi tôi thêm breakpoint. Tôi không khai báo trước sáu breakpoints và ép content vào chúng.

Về giá trị của những breakpoints đó, tôi thường align với defaults của Tailwind — 640, 768, 1024, 1280 — vì chúng gần đủ với content breakpoints cho hầu hết layouts, và quen thuộc với bất kỳ developer nào đã làm việc với modern CSS frameworks.

Tôi cũng cố minimize số lượng breakpoints. Giữa `clamp()` cho fluid scaling, CSS Grid với `auto-fit/minmax` cho adaptive layouts, và container queries cho components, rất nhiều responsive behavior xảy ra mà không cần breakpoint nào cả."

---

---

## 11. CSS Animations

**Level:** [Mid] / [Senior]

**EN:** What is the difference between `transition` and `animation`? How do `will-change` and `transform` affect performance? What is the FLIP technique?

**VI:** Sự khác biệt giữa `transition` và `animation`? `will-change` và `transform` ảnh hưởng đến performance như thế nào?

**Trả lời:**

| | `transition` | `animation` |
|--|------------|-------------|
| Kích hoạt | Cần trigger (hover, class change) | Tự chạy hoặc theo trigger |
| Keyframes | Không (chỉ A → B) | Có, nhiều bước |
| Loop | Không | Có (`iteration-count`) |
| Control (pause/resume) | Không | Có |
| Use case | Hover effects, state changes | Loaders, attention, complex motion |

```css
/* transition — chuyển đổi smooth khi state thay đổi */
.button {
  background: #3b82f6;
  transform: scale(1);
  /* property | duration | easing | delay */
  transition: background 0.2s ease, transform 0.15s ease;
}

.button:hover {
  background: #1d4ed8;
  transform: scale(1.03);
}

/* transition tất cả (cẩn thận — có thể animate những property không mong muốn) */
.card {
  transition: all 0.3s ease; /* không khuyến nghị */
}

/* animation — keyframes */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50%       { transform: scale(1.05); }
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.modal {
  animation: fadeInUp 0.3s ease forwards;
}

.loader {
  animation: spin 1s linear infinite;
}

/* ================================
   Performance: transform vs position
================================ */
/* XẤU — kích hoạt layout reflow (tốn kém) */
.bad-animation {
  transition: left 0.3s ease, top 0.3s ease; /* thay đổi layout */
}

/* TỐT — chỉ composite layer, không reflow */
.good-animation {
  transition: transform 0.3s ease; /* GPU-accelerated */
}

/* Properties chỉ trigger composite (nhanh nhất): transform, opacity */
/* Properties trigger paint: color, background, box-shadow */
/* Properties trigger layout: width, height, margin, padding, top, left */

/* ================================
   will-change — hint browser trước
================================ */
.animated-element {
  /* Báo browser chuẩn bị GPU layer trước */
  will-change: transform, opacity;
}

/* QUAN TRỌNG: Chỉ dùng will-change khi thực sự cần
   - Không dùng will-change: all
   - Xóa sau khi animation kết thúc (qua JS)
   - Dùng quá nhiều tốn bộ nhớ GPU
*/

/* Thay thế an toàn hơn: tạo stacking context */
.promote-layer {
  transform: translateZ(0); /* hack cũ */
  /* hoặc */
  isolation: isolate;
}

/* prefers-reduced-motion — accessibility */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What's the difference between a CSS transition and a CSS animation?"**

**Model Answer:**
"A transition is reactive — it smoothly animates between two states when something changes, like a hover or a class being toggled. You define start and end states and the browser fills in the middle. It can only go from A to B.

An animation with `@keyframes` is proactive — it runs on its own and you define multiple steps. You can loop it, pause it, run it in reverse. It's for things like loading spinners, pulsing effects, or entrance animations that play automatically.

Practically: if something changes based on user interaction, I use transitions. If I need something to run on its own, loop, or have more than two states, I use `@keyframes`."

**Trả lời (Tiếng Việt):**
"Transition phản ứng — nó animate mượt mà giữa hai trạng thái khi có gì đó thay đổi, như hover hoặc class được toggle. Bạn định nghĩa trạng thái đầu và cuối rồi browser điền phần giữa. Nó chỉ đi từ A đến B.

Animation với `@keyframes` chủ động — nó chạy tự mình và bạn định nghĩa nhiều bước. Bạn có thể loop nó, tạm dừng nó, chạy ngược lại. Dùng cho những thứ như loading spinners, pulsing effects, hay entrance animations chạy tự động.

Thực tế: nếu có gì đó thay đổi dựa trên tương tác người dùng, tôi dùng transitions. Nếu cần gì đó chạy tự mình, loop, hoặc có nhiều hơn hai trạng thái, tôi dùng `@keyframes`."

---

**Q2 (Mid): "Why is it better to animate `transform` and `opacity` rather than properties like `width` or `top`?"**

**Model Answer:**
"It comes down to how the browser renders things. When you change a layout property like `width`, `height`, `top`, or `margin`, the browser has to recalculate the layout for that element and potentially everything around it — that's called a reflow. Then it repaints. Both of those are expensive operations that happen on the main thread.

`transform` and `opacity` are special. The browser can handle them entirely on the GPU through the compositor thread, skipping layout and paint entirely. That's why they're smooth even when the main thread is busy.

So animating `transform: translateX(20px)` instead of `left: 20px` is a significant performance difference — especially noticeable on mobile or lower-end hardware."

**Trả lời (Tiếng Việt):**
"Vấn đề là cách browser render mọi thứ. Khi bạn thay đổi layout property như `width`, `height`, `top`, hay `margin`, browser phải tính lại layout cho element đó và có thể mọi thứ xung quanh nó — đó gọi là reflow. Rồi nó repaint. Cả hai đều là những operations tốn kém xảy ra trên main thread.

`transform` và `opacity` là đặc biệt. Browser có thể xử lý chúng hoàn toàn trên GPU thông qua compositor thread, bỏ qua layout và paint hoàn toàn. Đó là lý do tại sao chúng mượt mà ngay cả khi main thread đang bận.

Vậy animate `transform: translateX(20px)` thay vì `left: 20px` là một sự khác biệt performance đáng kể — đặc biệt rõ ràng trên mobile hoặc phần cứng cấp thấp hơn."

---

**Q3 (Mid): "What is `will-change` and when should you use it?"**

**Model Answer:**
"It's a hint to the browser that an element is going to be animated, so the browser can promote it to its own GPU layer in advance. This prevents the jank you sometimes get at the start of an animation when the browser hasn't prepared the layer yet.

But it's genuinely a hint — not a guarantee — and it has a cost. Keeping elements on a separate GPU layer uses more memory. If you put `will-change: transform` on every element, you can actually hurt performance.

I use it sparingly: only on elements that are about to animate in response to user interaction, and preferably I add and remove it via JavaScript right before and after the animation. I never put it in static CSS on everything."

**Trả lời (Tiếng Việt):**
"Đó là một gợi ý cho browser rằng một element sắp được animate, nên browser có thể đưa nó lên GPU layer riêng trước. Điều này ngăn jank bạn đôi khi thấy ở đầu animation khi browser chưa chuẩn bị layer.

Nhưng nó thực sự chỉ là gợi ý — không phải đảm bảo — và nó có chi phí. Giữ elements trên GPU layer riêng tốn thêm bộ nhớ. Nếu bạn đặt `will-change: transform` trên mọi element, bạn thực sự có thể làm giảm performance.

Tôi dùng nó một cách hạn chế: chỉ trên các elements sắp animate trong phản hồi tương tác người dùng, và tốt nhất là tôi thêm và xóa nó qua JavaScript ngay trước và sau animation. Tôi không bao giờ đặt nó trong static CSS trên mọi thứ."

---

**Q4 (Senior): "How do you handle animations for users who have motion sensitivity?"**

**Model Answer:**
"The CSS media feature `prefers-reduced-motion` is the right tool. When a user has enabled the 'reduce motion' setting on their OS, this media query matches and you can dial back or remove animations.

My preferred approach is opt-in rather than opt-out:

```css
@media (prefers-reduced-motion: no-preference) {
  .hero { animation: fadeInUp 0.6s ease both; }
}
```

This means animation only runs for users who haven't asked for reduced motion, rather than running for everyone and then being aggressively reset. It's a more respectful default.

For things like loaders that need some movement to communicate state, I'd replace a spinning animation with a pulsing opacity or a simpler, less vestibular animation rather than removing it entirely."

**Trả lời (Tiếng Việt):**
"CSS media feature `prefers-reduced-motion` là công cụ phù hợp. Khi người dùng đã bật cài đặt 'reduce motion' trên OS của họ, media query này match và bạn có thể giảm hoặc xóa animations.

Cách tiếp cận tôi ưa dùng là opt-in thay vì opt-out:

```css
@media (prefers-reduced-motion: no-preference) {
  .hero { animation: fadeInUp 0.6s ease both; }
}
```

Nghĩa là animation chỉ chạy cho người dùng chưa yêu cầu reduced motion, thay vì chạy cho mọi người rồi aggressively reset. Đó là mặc định respectful hơn.

Với những thứ như loaders cần một số chuyển động để communicate trạng thái, tôi sẽ thay spinning animation bằng pulsing opacity hoặc một animation đơn giản hơn, ít vestibular hơn thay vì xóa hoàn toàn."

---

**Q5 (Senior): "What is the FLIP animation technique?"**

**Model Answer:**
"FLIP stands for First, Last, Invert, Play. It's a technique for animating layout changes that would normally be expensive — like animating an element from one position on the page to a completely different position.

The idea is: First, record the element's initial position. Last, let it jump to the final position. Invert, apply a `transform` to make it visually appear at the starting position. Play, then animate the transform to identity (zero), which smoothly moves it to the final position.

The reason this performs well is that the actual layout change happens instantaneously — no animating `top` or `left`. The visual movement is driven entirely by `transform`, which is composited on the GPU.

The Web Animations API and libraries like Framer Motion make FLIP much easier to implement. Framer Motion's `layoutId` prop basically does this automatically when the same component moves between different DOM positions."

**Trả lời (Tiếng Việt):**
"FLIP viết tắt của First, Last, Invert, Play. Đó là kỹ thuật để animate layout changes thường sẽ tốn kém — như animate một element từ một vị trí trên trang sang một vị trí hoàn toàn khác.

Ý tưởng là: First, ghi lại vị trí ban đầu của element. Last, để nó nhảy sang vị trí cuối. Invert, áp dụng `transform` để nó visually xuất hiện ở vị trí bắt đầu. Play, rồi animate transform về identity (zero), mượt mà di chuyển nó sang vị trí cuối.

Lý do điều này perform tốt là actual layout change xảy ra tức thì — không animate `top` hay `left`. Chuyển động visual được driven hoàn toàn bởi `transform`, được composited trên GPU.

Web Animations API và các thư viện như Framer Motion làm FLIP dễ implement hơn nhiều. Prop `layoutId` của Framer Motion về cơ bản làm điều này tự động khi cùng một component di chuyển giữa các DOM positions khác nhau."

---

---

## 12. BEM Methodology

**Level:** [Junior] / [Mid]

**EN:** Explain BEM naming convention. What are Block, Element, and Modifier? Why is it useful?

**VI:** Giải thích BEM naming convention. Block, Element, Modifier là gì? Tại sao BEM hữu ích?

**Trả lời:**

BEM (Block__Element--Modifier) là một quy ước đặt tên class CSS giúp code có thể đọc, maintain, và tránh xung đột specificity.

- **Block**: Component độc lập (`card`, `nav`, `button`)
- **Element**: Phần tử thuộc Block, dùng `__` (`card__title`, `nav__item`)
- **Modifier**: Biến thể/trạng thái, dùng `--` (`button--primary`, `card--featured`)

```html
<!-- BEM HTML structure -->
<article class="card card--featured">
  <div class="card__image-wrapper">
    <img class="card__image" src="..." alt="..." />
  </div>

  <div class="card__body">
    <span class="card__tag">Technology</span>
    <h2 class="card__title">Article Title</h2>
    <p class="card__excerpt card__excerpt--truncated">...</p>
  </div>

  <footer class="card__footer">
    <button class="button button--primary button--sm">Read More</button>
    <button class="button button--ghost button--sm">Save</button>
  </footer>
</article>
```

```css
/* Block */
.card {
  border-radius: 8px;
  overflow: hidden;
  background: white;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

/* Block modifier */
.card--featured {
  border: 2px solid #3b82f6;
  box-shadow: 0 4px 12px rgba(59,130,246,0.2);
}

/* Elements */
.card__body {
  padding: 1.25rem;
}

.card__title {
  font-size: 1.125rem;
  font-weight: 600;
  margin: 0 0 0.5rem;
}

/* Element modifier */
.card__excerpt--truncated {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Button block — độc lập với card */
.button {
  display: inline-flex;
  align-items: center;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-weight: 500;
  transition: background 0.15s ease;
}

.button--primary {
  background: #3b82f6;
  color: white;
}

.button--ghost {
  background: transparent;
  color: #374151;
}

.button--sm {
  padding: 0.375rem 0.75rem;
  font-size: 0.8125rem;
}

/*
  Lợi ích BEM:
  1. Specificity thấp và nhất quán (0,0,1,0)
  2. Tự document — đọc class biết ngay mối quan hệ
  3. Không cần nesting sâu → dễ override
  4. Component portable — có thể move block sang context khác
*/
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is BEM and why would you use it over just writing regular class names?"**

**Model Answer:**
"BEM stands for Block, Element, Modifier. It's a naming convention for CSS classes where a Block is a standalone component like `card` or `button`, an Element is a part of that block separated by double underscores like `card__title`, and a Modifier is a variant separated by double dashes like `button--primary`.

The main reason to use it is to avoid class name collisions without resorting to deep selector nesting. If I just write `.title`, that could clash with any other `.title` anywhere. But `.card__title` is clearly scoped to the card component.

Another benefit is that every selector ends up with the same specificity — one class — which makes overrides predictable and consistent. No specificity wars."

**Trả lời (Tiếng Việt):**
"BEM viết tắt của Block, Element, Modifier. Đó là naming convention cho CSS classes mà Block là một standalone component như `card` hay `button`, Element là một phần của block đó được tách bằng double underscores như `card__title`, và Modifier là variant được tách bằng double dashes như `button--primary`.

Lý do chính để dùng nó là tránh class name collisions mà không cần deep selector nesting. Nếu tôi chỉ viết `.title`, nó có thể clash với bất kỳ `.title` nào ở đâu đó. Nhưng `.card__title` rõ ràng là scoped cho card component.

Một lợi ích nữa là mọi selector đều có cùng specificity — một class — giúp overrides có thể dự đoán và nhất quán. Không có specificity wars."

---

**Q2 (Mid): "How do you handle cases in BEM where an element needs to be inside another element? Like a card inside a grid inside a layout section?"**

**Model Answer:**
"This is a common point of confusion. The BEM rule is that you should never chain element selectors, so you wouldn't write `.layout__section__grid__card`. That gets unreadable fast.

Instead, you treat each level as a new Block if it can stand alone, or you just keep elements flat under their parent Block. So you might have a `.layout__section`, a `.card-grid` as its own block, and `.card` as another block. They're siblings in the HTML nesting sense, but each is an independent BEM block.

BEM describes the component structure, not the DOM depth. The double underscore means 'belongs to this block conceptually,' not 'is a child of this selector.'"

**Trả lời (Tiếng Việt):**
"Đây là điểm nhầm lẫn phổ biến. Quy tắc BEM là không bao giờ chain element selectors, nên bạn không viết `.layout__section__grid__card`. Trở nên không đọc được rất nhanh.

Thay vào đó, bạn treat mỗi cấp như một Block mới nếu nó có thể đứng độc lập, hoặc chỉ giữ elements phẳng dưới Block parent của chúng. Vậy bạn có thể có `.layout__section`, một `.card-grid` như block riêng, và `.card` như block khác. Chúng là siblings theo nghĩa HTML nesting, nhưng mỗi cái là một BEM block độc lập.

BEM mô tả component structure, không phải DOM depth. Double underscore nghĩa là 'thuộc về block này về mặt conceptual,' không phải 'là child của selector này.'"

---

**Q3 (Mid): "What are the limitations of BEM, and when might you not use it?"**

**Model Answer:**
"BEM works really well for server-rendered or plain CSS projects. The limitations show up when you're using CSS Modules or CSS-in-JS.

With CSS Modules, the build tool automatically scopes class names to the file, so the collision problem BEM solves is already solved for you. Using BEM on top of CSS Modules ends up being verbose without much payoff — you get class names like `Button_button__primary__abc123` which is harder to debug.

Similarly with styled-components or Emotion, scoping is handled by the library. In those contexts I'd lean toward shorter, descriptive class names.

I still find BEM valuable on large teams with global CSS, in design systems shared across projects, or in projects that need to be inspectable in plain HTML without React devtools."

**Trả lời (Tiếng Việt):**
"BEM hoạt động thực sự tốt cho server-rendered hoặc plain CSS projects. Hạn chế xuất hiện khi bạn dùng CSS Modules hoặc CSS-in-JS.

Với CSS Modules, build tool tự động scope class names theo file, nên vấn đề collision mà BEM giải quyết đã được giải quyết cho bạn rồi. Dùng BEM trên CSS Modules trở nên verbose mà không có nhiều payoff — bạn có class names như `Button_button__primary__abc123` khó debug hơn.

Tương tự với styled-components hoặc Emotion, scoping được xử lý bởi thư viện. Trong những contexts đó tôi sẽ lean toward shorter, descriptive class names.

Tôi vẫn thấy BEM có giá trị trên large teams với global CSS, trong design systems được chia sẻ qua các projects, hoặc trong các projects cần inspectable trong plain HTML mà không có React devtools."

---

**Q4 (Senior): "How do you scale BEM on a large team where different developers might interpret it differently?"**

**Model Answer:**
"Documentation and tooling are the main levers. I'd document conventions in a style guide — what counts as a block versus an element, when to create a modifier versus a new block, how to handle component composition.

On the tooling side, a linter like `stylelint-bem-pattern` can enforce the naming rules automatically in CI. That catches drift before it gets into a PR.

Beyond that, code review plays a role. If someone writes `.card__header__title`, that's a review comment — flatten it to `.card__title` or make `header` its own block.

The honest truth is BEM requires team buy-in. If people don't understand the why — keeping specificity flat and making relationships explicit — they'll fight it. I'd spend time explaining the motivation before enforcing the rules."

**Trả lời (Tiếng Việt):**
"Documentation và tooling là đòn bẩy chính. Tôi sẽ document conventions trong một style guide — cái gì được tính là block so với element, khi nào tạo modifier so với block mới, cách xử lý component composition.

Về tooling, một linter như `stylelint-bem-pattern` có thể tự động enforce naming rules trong CI. Điều đó bắt drift trước khi nó vào PR.

Ngoài ra, code review đóng vai trò. Nếu ai đó viết `.card__header__title`, đó là review comment — flatten nó thành `.card__title` hoặc làm `header` thành block riêng.

Sự thật thực tế là BEM đòi hỏi team buy-in. Nếu mọi người không hiểu lý do tại sao — giữ specificity phẳng và làm relationships rõ ràng — họ sẽ kháng cự. Tôi sẽ dành thời gian giải thích motivation trước khi enforce rules."

---

---

## 13. Accessibility in CSS

**Level:** [Mid] / [Senior]

**EN:** How does CSS affect accessibility? Explain `focus-visible`, `prefers-reduced-motion`, and `prefers-color-scheme`. What CSS patterns improve or hurt accessibility?

**VI:** CSS ảnh hưởng đến accessibility như thế nào? Giải thích `focus-visible`, `prefers-reduced-motion`, và `prefers-color-scheme`.

**Trả lời:**

```css
/* ================================
   1. focus-visible — focus indicator chỉ khi dùng keyboard
================================ */

/* XẤU: xóa outline hoàn toàn — ảnh hưởng keyboard users */
button:focus {
  outline: none; /* ĐỪNG LÀM THẾ NÀY */
}

/* TỐT: ẩn với mouse nhưng hiển thị với keyboard */
button:focus {
  outline: none; /* ẩn focus mặc định */
}

button:focus-visible {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
  /* Trình duyệt tự phân biệt mouse click vs keyboard Tab */
}

/* Custom focus ring đẹp hơn */
:focus-visible {
  outline: 2px solid #3b82f6;
  outline-offset: 3px;
  border-radius: 4px;
  box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.2);
}

/* ================================
   2. prefers-reduced-motion
================================ */

/* Người dùng với vestibular disorders, motion sensitivity */
@media (prefers-reduced-motion: reduce) {
  /* Tắt tất cả animations */
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* Cách tiếp cận tốt hơn: opt-in thay vì opt-out */
@media (prefers-reduced-motion: no-preference) {
  .hero {
    animation: fadeInUp 0.6s ease both;
  }

  .smooth-scroll {
    scroll-behavior: smooth;
  }
}

/* ================================
   3. prefers-color-scheme
================================ */
:root {
  --color-bg: #ffffff;
  --color-text: #111827;
  --color-border: #e5e7eb;
  --color-primary: #3b82f6;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0f172a;
    --color-text: #f1f5f9;
    --color-border: #334155;
    --color-primary: #60a5fa;
  }
}

/* ================================
   4. Các patterns accessibility khác
================================ */

/* Visually hidden nhưng vẫn accessible với screen reader */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Không dùng display:none hay visibility:hidden vì screen reader bỏ qua */

/* Color contrast — đủ tương phản WCAG AA: 4.5:1 (normal text), 3:1 (large text) */
.text-low-contrast {
  color: #9ca3af;       /* FAILS WCAG AA trên nền trắng */
}
.text-accessible {
  color: #4b5563;       /* PASSES WCAG AA */
}

/* Skip link — cho keyboard users nhảy qua nav */
.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  background: #3b82f6;
  color: white;
  padding: 0.5rem 1rem;
  z-index: 9999;
  border-radius: 0 0 4px 0;
}

.skip-link:focus {
  top: 0;
}

/* Hover cũng cần accessible — không dựa chỉ vào hover để show thông tin */
/* Dùng :hover AND :focus để tooltip, dropdown accessible với keyboard */
.tooltip-trigger:hover .tooltip,
.tooltip-trigger:focus .tooltip {
  display: block;
}

/* forced-colors — Windows High Contrast Mode */
@media (forced-colors: active) {
  .button {
    border: 2px solid ButtonText; /* dùng system colors */
  }
}
```
