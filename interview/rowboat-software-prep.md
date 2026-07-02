# Rowboat Software — Frontend Design Systems Engineer

> **Công ty:** Rowboat Software (Phoenix, AZ) — chi nhánh HCMC đang mở rộng  
> **Sản phẩm:** Online design editor (SaaS) — complex UI tương tự Canva/Figma  
> **Stack:** Vue.js + React, Figma, Storybook, CSS architecture  
> **Lưu ý:** Early hire → cơ hội lên team lead. JD rất nặng về CSS quality và Design System.

---

## Mục lục

1. [Company Context — Nói gì khi phỏng vấn](#1-company-context)
2. [Design System — Component Library](#2-design-system--component-library)
3. [CSS Architecture](#3-css-architecture)
4. [Responsive & Adaptive Design](#4-responsive--adaptive-design)
5. [Complex UI Patterns (Editor-specific)](#5-complex-ui-patterns-editor-specific)
6. [CSS Animations & Transitions](#6-css-animations--transitions)
7. [Figma to Code Workflow](#7-figma-to-code-workflow)
8. [Pixel-Perfect Implementation](#8-pixel-perfect-implementation)
9. [Accessibility & Semantic HTML](#9-accessibility--semantic-html)
10. [Cross-Browser Compatibility](#10-cross-browser-compatibility)
11. [Vue.js (nếu hỏi)](#11-vuejs-nếu-hỏi)
12. [Câu hỏi bạn nên hỏi ngược lại](#12-câu-hỏi-bạn-nên-hỏi-ngược-lại)

---

## 1. Company Context

**Biết về Rowboat trước khi vào phỏng vấn:**
- Sản phẩm là một **online design editor** dạng SaaS — think Canva, Figma, Adobe Express nhưng focus vào content management.
- HQ ở Phoenix, Arizona. HCMC branch đang expand — bạn là early hire → opportunity to grow into TL.
- Stack chính: **Vue.js** (legacy/primary) + **React** (đang dùng song song), design system components cần chạy cả 2.
- Họ tìm người **bridge giữa Design và Engineering** — không phải pure dev, phải có design sense.

**Cách mở đầu tự giới thiệu:**
> "Tôi đã đọc về sản phẩm của Rowboat — một online design editor với complex UI. Điều tôi thấy match nhất với experience của tôi là tôi vừa build end-to-end 70+ component design system tại Herond Labs, và tôi hiểu rõ sự khác biệt giữa một component library và một design system thực sự — tokens, composition patterns, visual regression pipeline..."

---

## 2. Design System — Component Library

### Q2.1
**Level: [Senior]**  
**EN:** What is the difference between a component library and a design system?  
**VI:** Sự khác biệt giữa component library và design system là gì?

**Trả lời:**

| | Component Library | Design System |
|---|---|---|
| **Scope** | Code only — React/Vue components | Code + Design + Documentation + Process |
| **Artifacts** | npm package, Storybook | Figma library + tokens + code + guidelines |
| **Design Tokens** | Không bắt buộc | Bắt buộc — single source of truth cho màu, spacing, typography |
| **Ownership** | Dev team | Cross-functional (Design + Engineering) |
| **Governance** | Ít formal hơn | Có review process, versioning, deprecation policy |

> Trong dự án thực tế: tôi build component library trước (70+ components), sau đó evolve nó thành design system bằng cách thêm Figma token pipeline, visual regression tests, và contribution guidelines.

---

### Q2.2
**Level: [Senior]**  
**EN:** How do you architect a scalable design system component — from atom to organism?  
**VI:** Bạn thiết kế một component trong design system như thế nào từ atom đến organism?

**Trả lời — Atomic Design pattern:**

```
Atoms → Molecules → Organisms → Templates → Pages
Button   ButtonGroup  Toolbar     EditorLayout  EditorPage
Icon     SearchBar    Sidebar     ModalShell    SettingsPage  
Input    InputGroup   Inspector   ...           ...
```

**Thực tế khi build:**

```tsx
// 1. Atom: Button — chỉ nhận primitive props
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'ghost' | 'danger';
  size: 'xs' | 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  leftIcon?: ReactNode;
  children: ReactNode;
  onClick?: () => void;
}

// 2. Molecule: ConfirmDialog — compose từ atoms
interface ConfirmDialogProps {
  title: string;
  description: string;
  onConfirm: () => void;
  onCancel: () => void;
  confirmVariant?: ButtonProps['variant'];
}

// 3. Token-driven: màu sắc không hard-code
const tokens = {
  color: {
    primary: { 500: '#3B82F6', 600: '#2563EB' },
    neutral: { 100: '#F3F4F6', 900: '#111827' },
  },
  spacing: { xs: '4px', sm: '8px', md: '16px', lg: '24px' },
  radius: { sm: '4px', md: '8px', lg: '12px', full: '9999px' },
};
```

---

### Q2.3
**Level: [Senior]**  
**EN:** How do you implement and manage design tokens in a design system?  
**VI:** Làm thế nào để implement và manage design tokens?

**Trả lời:**

```css
/* Bước 1: Khai báo tokens dưới dạng CSS Custom Properties */
:root {
  /* Color tokens */
  --color-primary-500: #3B82F6;
  --color-primary-600: #2563EB;
  --color-neutral-50: #F9FAFB;
  --color-neutral-900: #111827;
  
  /* Semantic tokens — dùng trong components */
  --color-bg-primary: var(--color-neutral-50);
  --color-text-primary: var(--color-neutral-900);
  --color-interactive: var(--color-primary-500);
  
  /* Spacing scale */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  
  /* Typography */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-weight-regular: 400;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  
  /* Border radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;
  
  /* Shadow */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
}

/* Bước 2: Dark mode bằng cách đổi semantic tokens */
[data-theme="dark"] {
  --color-bg-primary: #1F2937;
  --color-text-primary: #F9FAFB;
}
```

**Figma → Code automation (từ experience thực tế của tôi):**
```
Figma Variables → Style Dictionary → CSS Custom Properties + JS tokens
                                   → Tailwind config
                                   → TypeScript types
```

---

### Q2.4
**Level: [Senior]**  
**EN:** How do you set up visual regression testing in a design system CI pipeline?  
**VI:** Làm thế nào để setup visual regression testing trong CI?

**Trả lời:**

```yaml
# .github/workflows/visual-regression.yml
name: Visual Regression
on: [pull_request]

jobs:
  visual-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build-storybook
      - run: npx playwright test --project=chromium
        env:
          PLAYWRIGHT_UPDATE_SNAPSHOTS: false
```

```typescript
// playwright/visual-regression.spec.ts
import { test, expect } from '@playwright/test';

const COMPONENTS = ['Button', 'Input', 'Modal', 'Toolbar', 'Inspector'];

for (const component of COMPONENTS) {
  test(`${component} — visual regression`, async ({ page }) => {
    await page.goto(`/iframe.html?id=${component.toLowerCase()}--default`);
    await expect(page).toHaveScreenshot(`${component}-default.png`, {
      maxDiffPixelRatio: 0.02, // cho phép 2% pixel khác biệt
    });
  });
  
  // Test dark mode
  test(`${component} — dark mode`, async ({ page }) => {
    await page.goto(`/iframe.html?id=${component.toLowerCase()}--dark`);
    await page.emulateMedia({ colorScheme: 'dark' });
    await expect(page).toHaveScreenshot(`${component}-dark.png`);
  });
}
```

---

## 3. CSS Architecture

### Q3.1
**Level: [Mid/Senior]**  
**EN:** Compare CSS Modules, BEM, and CSS-in-JS. Which would you use in a design system for a Vue + React codebase?  
**VI:** So sánh CSS Modules, BEM, CSS-in-JS. Cái nào dùng cho design system Vue + React?

**Trả lời:**

| Approach | Pros | Cons | Khi nào dùng |
|---|---|---|---|
| **BEM** | Zero dependency, easy to share | Dễ conflict, verbose classnames | Multi-framework, vanilla projects |
| **CSS Modules** | Scoped, auto-unique classnames, works với cả Vue + React | Build pipeline cần support | ✅ Recommended cho dual Vue+React |
| **Tailwind** | Rapid UI, zero runtime | HTML verbose, hard to share tokens | App-level, không phải design system |
| **CSS-in-JS** | Fully dynamic, colocation | Runtime cost, SSR complexity | React-only, dynamic theming |

**Recommendation cho Rowboat (Vue + React shared design system):**

```scss
// CSS Modules + CSS Custom Properties = best of both worlds

// styles/Button.module.scss
.button {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  border-radius: var(--radius-md);
  font-weight: var(--font-weight-semibold);
  transition: background-color 150ms ease, box-shadow 150ms ease;
  cursor: pointer;
  
  /* Variants via data attributes — works in both Vue & React */
  &[data-variant="primary"] {
    background: var(--color-interactive);
    color: white;
  }
  
  &[data-variant="ghost"] {
    background: transparent;
    color: var(--color-text-primary);
    
    &:hover { background: var(--color-neutral-100); }
  }
  
  /* Sizes */
  &[data-size="sm"] { padding: var(--space-1) var(--space-3); font-size: var(--font-size-sm); }
  &[data-size="md"] { padding: var(--space-2) var(--space-4); font-size: var(--font-size-base); }
}
```

---

### Q3.2
**Level: [Senior]**  
**EN:** How do you structure CSS for a large application to prevent specificity wars and style leakage?  
**VI:** Làm thế nào để structure CSS cho app lớn, tránh specificity conflict?

**Trả lời — ITCSS (Inverted Triangle CSS):**

```
/styles
  /settings    → tokens, CSS custom properties
  /tools       → mixins, functions (nếu dùng SCSS)
  /generic     → reset, normalize, box-sizing
  /elements    → base HTML elements (a, p, h1-h6)
  /objects     → layout patterns (grid, container, media object)
  /components  → UI components (button, modal, toolbar)
  /utilities   → single-purpose helpers (.sr-only, .truncate)
```

**Rules:**
1. Không dùng `!important` trong components — chỉ dùng trong utilities layer
2. Không nest quá 3 levels sâu
3. Specificity thấp trong base, tăng dần
4. Semantic tokens → không hard-code color values trong component files

---

## 4. Responsive & Adaptive Design

### Q4.1
**Level: [Mid/Senior]**  
**EN:** What is the difference between responsive design and adaptive design?  
**VI:** Responsive design và adaptive design khác nhau như thế nào?

**Trả lời:**

| | Responsive | Adaptive |
|---|---|---|
| **Cơ chế** | Fluid layout — co giãn liên tục | Breakpoint cứng — serve different layout |
| **CSS** | `%`, `vw`, `clamp()`, media queries | Fixed breakpoints, có thể serve different HTML |
| **Complexity** | Đơn giản hơn | Phức tạp hơn, cần detect device |
| **Khi dùng** | Marketing sites, content pages | Complex apps (editor, dashboard) |

**Cho design editor như Rowboat — hybrid approach:**

```css
/* Fluid typography với clamp() */
:root {
  --font-size-heading: clamp(1.25rem, 2vw + 0.5rem, 1.875rem);
  --panel-width: clamp(240px, 20vw, 320px);
}

/* Container queries cho components */
.inspector-panel {
  container-type: inline-size;
  container-name: inspector;
}

@container inspector (min-width: 280px) {
  .property-row {
    flex-direction: row;
    justify-content: space-between;
  }
}

@container inspector (max-width: 280px) {
  .property-row {
    flex-direction: column;
  }
}

/* Adaptive: editor layout thay đổi hoàn toàn theo device */
@media (max-width: 768px) {
  .editor-layout {
    /* Mobile: toolbar bottom, canvas full screen, panels as drawers */
    grid-template-areas:
      "canvas"
      "toolbar";
    grid-template-rows: 1fr auto;
  }
}

@media (min-width: 769px) {
  .editor-layout {
    /* Desktop: side panels, top toolbar */
    grid-template-areas:
      "toolbar toolbar toolbar"
      "layers  canvas  inspector";
    grid-template-columns: var(--panel-width) 1fr var(--panel-width);
    grid-template-rows: 48px 1fr;
  }
}
```

---

### Q4.2
**Level: [Senior]**  
**EN:** When would you use container queries over media queries?  
**VI:** Khi nào dùng container queries thay vì media queries?

**Trả lời:**

- **Media queries**: layout respond to **viewport size** — dùng cho page-level layout, global breakpoints
- **Container queries**: component respond to **container size** — dùng khi component được embed ở nhiều chỗ khác nhau

```css
/* Vấn đề với media queries: */
/* Component Card bị break khi đặt trong sidebar (400px wide) trên desktop */
/* Vì media query chỉ biết viewport = 1440px, không biết container = 400px */

/* Giải pháp: container queries */
.card-container {
  container-type: inline-size;
}

.card {
  display: grid;
  grid-template-columns: 1fr; /* default: stacked */
}

@container (min-width: 400px) {
  .card {
    grid-template-columns: 80px 1fr; /* horizontal layout khi có đủ space */
  }
}
```

---

## 5. Complex UI Patterns (Editor-specific)

### Q5.1
**Level: [Senior]**  
**EN:** How do you build a resizable side panel (like in VS Code or Figma)?  
**VI:** Làm thế nào để build resizable side panel?

**Trả lời:**

```tsx
// ResizablePanel.tsx
const ResizablePanel = ({ defaultWidth = 280, minWidth = 200, maxWidth = 480, children }) => {
  const [width, setWidth] = useState(defaultWidth);
  const isResizing = useRef(false);
  const startX = useRef(0);
  const startWidth = useRef(0);

  const onMouseDown = (e: React.MouseEvent) => {
    isResizing.current = true;
    startX.current = e.clientX;
    startWidth.current = width;
    document.body.style.cursor = 'col-resize';
    document.body.style.userSelect = 'none'; // prevent text selection during drag
  };

  useEffect(() => {
    const onMouseMove = (e: MouseEvent) => {
      if (!isResizing.current) return;
      const delta = e.clientX - startX.current;
      const newWidth = Math.min(maxWidth, Math.max(minWidth, startWidth.current + delta));
      setWidth(newWidth);
    };

    const onMouseUp = () => {
      isResizing.current = false;
      document.body.style.cursor = '';
      document.body.style.userSelect = '';
    };

    document.addEventListener('mousemove', onMouseMove);
    document.addEventListener('mouseup', onMouseUp);
    return () => {
      document.removeEventListener('mousemove', onMouseMove);
      document.removeEventListener('mouseup', onMouseUp);
    };
  }, [minWidth, maxWidth]);

  return (
    <div style={{ width, flexShrink: 0, position: 'relative' }}>
      {children}
      <div
        className="resize-handle"
        onMouseDown={onMouseDown}
        style={{ position: 'absolute', right: 0, top: 0, bottom: 0, width: 4, cursor: 'col-resize' }}
        role="separator"
        aria-orientation="vertical"
        aria-valuenow={width}
        aria-valuemin={minWidth}
        aria-valuemax={maxWidth}
      />
    </div>
  );
};
```

---

### Q5.2
**Level: [Senior]**  
**EN:** How do you implement a contextual menu / right-click menu that stays within viewport bounds?  
**VI:** Làm sao implement contextual menu không bị cắt bởi viewport?

**Trả lời:**

```tsx
const ContextMenu = ({ x, y, items, onClose }) => {
  const menuRef = useRef<HTMLDivElement>(null);
  const [position, setPosition] = useState({ x, y });

  useLayoutEffect(() => {
    if (!menuRef.current) return;
    const menu = menuRef.current.getBoundingClientRect();
    const vw = window.innerWidth;
    const vh = window.innerHeight;

    setPosition({
      x: x + menu.width > vw ? x - menu.width : x,
      y: y + menu.height > vh ? y - menu.height : y,
    });
  }, [x, y]);

  return (
    <div
      ref={menuRef}
      style={{ position: 'fixed', left: position.x, top: position.y, zIndex: 9999 }}
      onMouseDown={(e) => e.stopPropagation()}
      role="menu"
    >
      {items.map((item) => (
        <div key={item.id} role="menuitem" onClick={() => { item.action(); onClose(); }}>
          {item.label}
        </div>
      ))}
    </div>
  );
};
```

---

### Q5.3
**Level: [Senior]**  
**EN:** How do you implement a focus trap for modals (accessibility requirement)?  
**VI:** Implement focus trap cho modal như thế nào?

**Trả lời:**

```tsx
const useFocusTrap = (containerRef: RefObject<HTMLElement>) => {
  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;

    const focusable = container.querySelectorAll<HTMLElement>(
      'a[href], button:not([disabled]), input:not([disabled]), textarea:not([disabled]), select:not([disabled]), [tabindex]:not([tabindex="-1"])'
    );
    const first = focusable[0];
    const last = focusable[focusable.length - 1];
    const previouslyFocused = document.activeElement as HTMLElement;

    first?.focus();

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;
      if (e.shiftKey) {
        if (document.activeElement === first) { e.preventDefault(); last?.focus(); }
      } else {
        if (document.activeElement === last) { e.preventDefault(); first?.focus(); }
      }
    };

    container.addEventListener('keydown', handleKeyDown);
    return () => {
      container.removeEventListener('keydown', handleKeyDown);
      previouslyFocused?.focus(); // restore focus khi modal đóng
    };
  }, [containerRef]);
};
```

---

## 6. CSS Animations & Transitions

### Q6.1
**Level: [Mid/Senior]**  
**EN:** When should you use CSS transitions vs CSS animations vs JavaScript animations?  
**VI:** Khi nào dùng CSS transition, CSS animation, hay JS animation?

**Trả lời:**

| | CSS Transition | CSS Animation (`@keyframes`) | JS Animation (WAAPI / GSAP) |
|---|---|---|---|
| **Trigger** | State change (hover, class toggle) | Runs automatically / on class add | Programmatic |
| **Complexity** | Simple A→B | Multi-step, loop | Complex sequences, physics |
| **Performance** | GPU accelerated (transform, opacity) | GPU accelerated | Depends on props |
| **Control** | Limited | Limited (pause/play) | Full control |

```css
/* ✅ Transition: sidebar slide in */
.sidebar {
  transform: translateX(-100%);
  transition: transform 250ms cubic-bezier(0.4, 0, 0.2, 1);
}
.sidebar.open { transform: translateX(0); }

/* ✅ @keyframes: loading spinner, skeleton pulse */
@keyframes skeleton-pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.4; }
}
.skeleton { animation: skeleton-pulse 1.5s ease-in-out infinite; }

/* ✅ @keyframes: modal backdrop fade + scale */
@keyframes modal-enter {
  from { opacity: 0; transform: scale(0.95) translateY(8px); }
  to   { opacity: 1; transform: scale(1)    translateY(0);   }
}
.modal { animation: modal-enter 200ms ease forwards; }

/* Respect user preference */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

### Q6.2
**Level: [Senior]**  
**EN:** Why should you only animate `transform` and `opacity` for performance?  
**VI:** Tại sao chỉ nên animate `transform` và `opacity`?

**Trả lời:**

Rendering pipeline của browser: **Style → Layout → Paint → Composite**

- Animate `width`, `height`, `top`, `left` → trigger **Layout** (reflow) → tốn nhất, toàn bộ DOM tính lại
- Animate `background-color`, `color`, `box-shadow` → trigger **Paint** → tốn vừa
- Animate `transform`, `opacity` → chỉ trigger **Composite** → GPU handle, không touch DOM → 60fps

```css
/* ❌ Gây reflow mỗi frame */
.bad { transition: width 300ms, left 300ms; }

/* ✅ Chỉ composite — GPU smooth */
.good { transition: transform 300ms cubic-bezier(0.4, 0, 0.2, 1); }

/* Ví dụ: di chuyển element */
/* ❌ */ .move-bad  { left: 100px; }
/* ✅ */ .move-good { transform: translateX(100px); }

/* Force GPU layer khi cần (dùng sparingly) */
.gpu-layer { will-change: transform; }
/* Hoặc: */ .gpu-layer { transform: translateZ(0); }
```

---

## 7. Figma to Code Workflow

### Q7.1
**Level: [Senior]**  
**EN:** Describe your workflow for converting a Figma design to production-quality code.  
**VI:** Mô tả workflow chuyển Figma design thành production code của bạn?

**Trả lời — từ experience thực tế:**

```
1. AUDIT FIGMA TRƯỚC KHI CODE
   ├── Kiểm tra design tokens: màu có consistent không, spacing có theo scale không
   ├── Check interactive states: hover, focus, active, disabled, loading, error
   ├── Check responsive frames: desktop / tablet / mobile
   └── Check edge cases: text overflow, empty states, long content

2. SETUP TOKENS TRƯỚC
   ├── Export Figma Variables → JSON via plugin (Tokens Studio / Variables Import Export)
   ├── Run Style Dictionary để gen CSS custom properties + TS types
   └── Commit tokens file vào repo

3. BUILD COMPONENT
   ├── Component shell với TypeScript interface
   ├── CSS với tokens (không hard-code values)
   ├── All states (hover, focus, disabled, loading)
   └── Responsive behavior

4. STORYBOOK STORY
   ├── Default story
   ├── All variants
   ├── Interaction tests (click, keyboard nav)
   └── A11y check (axe-core plugin)

5. VISUAL REGRESSION
   └── Playwright screenshot so với Figma reference
```

---

## 8. Pixel-Perfect Implementation

### Q8.1
**Level: [Senior]**  
**EN:** What tools and techniques do you use to achieve pixel-perfect implementation from design specs?  
**VI:** Tools và techniques nào để implement pixel-perfect?

**Trả lời:**

**Tools:**
- **Figma Dev Mode** (inspect spacing, font, colors chính xác)
- **PerfectPixel Chrome extension** — overlay Figma screenshot lên browser để so sánh
- **Playwright visual regression** — diff screenshots tự động
- **Storybook + Chromatic** — visual review cho từng story

**Techniques:**

```css
/* 1. Dùng đúng đơn vị */
/* Spacing: px cho fixed, rem cho scalable */
.button { padding: 8px 16px; }          /* fixed — match Figma px chính xác */
.body-text { font-size: 0.875rem; }     /* scalable với user browser font size */

/* 2. Text rendering */
.heading {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

/* 3. Line height cho typography precision */
/* Figma line-height: 24px trên font 16px → line-height: 1.5 */
/* Không dùng px cho line-height → sẽ không scale */

/* 4. Sub-pixel rendering issues */
/* Nếu element bị blurry: */
.crisp-render {
  transform: translateZ(0);
  backface-visibility: hidden;
}
```

---

## 9. Accessibility & Semantic HTML

### Q9.1
**Level: [Mid/Senior]**  
**EN:** What ARIA roles and attributes do complex UI components (toolbar, inspector panel) need?  
**VI:** ARIA roles và attributes cần thiết cho toolbar và inspector panel?

**Trả lời:**

```html
<!-- Toolbar: role="toolbar", keyboard nav với arrow keys -->
<div role="toolbar" aria-label="Formatting options">
  <button aria-pressed="true" aria-label="Bold">B</button>
  <button aria-pressed="false" aria-label="Italic">I</button>
  <separator role="separator" aria-orientation="vertical" />
  <button aria-label="Align left" aria-checked="true">⬅</button>
</div>

<!-- Inspector panel với collapsible sections -->
<section aria-labelledby="fill-heading">
  <h3 id="fill-heading">
    <button 
      aria-expanded="true" 
      aria-controls="fill-content"
      id="fill-trigger"
    >Fill</button>
  </h3>
  <div id="fill-content" role="region" aria-labelledby="fill-heading">
    <!-- content -->
  </div>
</section>

<!-- Modal -->
<div 
  role="dialog" 
  aria-modal="true" 
  aria-labelledby="dialog-title"
  aria-describedby="dialog-desc"
>
  <h2 id="dialog-title">Delete Layer</h2>
  <p id="dialog-desc">Are you sure? This action cannot be undone.</p>
</div>
```

---

## 10. Cross-Browser Compatibility

### Q10.1
**Level: [Mid/Senior]**  
**EN:** What are common cross-browser CSS issues and how do you handle them?  
**VI:** Các vấn đề CSS cross-browser phổ biến và cách xử lý?

**Trả lời:**

```css
/* 1. Scrollbar styling */
/* ✅ Modern — Chrome, Safari, Edge */
.scroll-container::-webkit-scrollbar { width: 6px; }
.scroll-container::-webkit-scrollbar-thumb { background: var(--color-neutral-300); border-radius: 3px; }
/* Firefox */
.scroll-container { scrollbar-width: thin; scrollbar-color: var(--color-neutral-300) transparent; }

/* 2. Gap trong Flexbox (Safari <14 không support gap với flex) */
/* Fallback: */
.flex-container > * + * { margin-left: 8px; }
/* Modern + override: */
@supports (gap: 8px) {
  .flex-container { gap: 8px; }
  .flex-container > * + * { margin-left: 0; }
}

/* 3. container-query (Safari 16+, Chrome 105+, Firefox 110+) */
@supports (container-type: inline-size) {
  .card-wrapper { container-type: inline-size; }
}

/* 4. aspect-ratio (cũ) */
/* Nếu cần support Safari <15: dùng padding-top hack */
.aspect-ratio-box {
  aspect-ratio: 16 / 9; /* modern */
}
/* Fallback: */
@supports not (aspect-ratio: 1) {
  .aspect-ratio-box::before {
    content: '';
    float: left;
    padding-top: 56.25%; /* 9/16 */
  }
}
```

---

## 11. Vue.js (nếu hỏi)

### Q11.1
**Level: [Mid]**  
**EN:** What are the key differences between Vue 3 Composition API and Options API?  
**VI:** Sự khác biệt giữa Vue 3 Composition API và Options API?

**Trả lời:**

```vue
<!-- Options API (Vue 2 style) -->
<script>
export default {
  data() {
    return { count: 0 };
  },
  computed: {
    doubled() { return this.count * 2; }
  },
  methods: {
    increment() { this.count++; }
  }
}
</script>

<!-- Composition API (Vue 3 — preferred) -->
<script setup>
import { ref, computed } from 'vue';

const count = ref(0);
const doubled = computed(() => count.value * 2);
const increment = () => count.value++;
</script>
```

**Composables (Vue equivalent của React hooks):**

```ts
// useResizablePanel.ts — reusable logic
import { ref, onMounted, onUnmounted } from 'vue';

export function useResizablePanel(defaultWidth = 280) {
  const width = ref(defaultWidth);
  // ... resize logic
  return { width };
}
```

---

## 12. Câu hỏi bạn nên hỏi ngược lại

> Cuối buổi phỏng vấn, hỏi những câu này — thể hiện bạn nghiêm túc và có strategic thinking:

1. **"The JD mentions both Vue.js and React — what is the current ratio of Vue vs React in the codebase? Is there a migration plan?"**
   → Cho thấy bạn quan tâm đến tech debt và planning.

2. **"What does success look like for this role in the first 3 months?"**
   → Show ambition và muốn deliver value nhanh.

3. **"Does the design system currently have a Figma component library that's in sync with the code? Or is that something we'd be building?"**
   → Show bạn đã nghĩ về design-engineering alignment.

4. **"What's the biggest visual inconsistency or CSS pain point in the product right now?"**
   → Show bạn ready để tackle real problems từ ngày đầu.

5. **"Since this is an early HCMC hire, how does collaboration work with the Phoenix team — async or real-time overlap?"**
   → Practical, shows you've thought about remote collaboration.

---

## Quick Reference — JD Match với CV của bạn

| JD Requirement | Your Experience |
|---|---|
| Design System ownership | ✅ 70+ component library Herond Labs |
| Figma → code pipeline | ✅ Figma token automation |
| Visual regression (Playwright) | ✅ Playwright CI pipeline |
| Storybook | ✅ Component docs |
| Pixel-perfect | ✅ Daily code reviews for visual quality |
| Vue.js | ⚠️ Learn basics — `ref`, `computed`, `v-model`, Composition API |
| React | ✅ 5 years experience |
| Responsive/adaptive | ✅ Mobile/tablet/desktop |
| Accessibility | ✅ Semantic HTML, ARIA (mention in interview) |
