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
