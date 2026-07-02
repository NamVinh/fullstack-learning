# Designveloper × Lumin PDF — Interview Prep

> **Công ty:** Designveloper (DSV) — HCMC, thành lập 2013  
> **Sản phẩm:** Lumin PDF — cloud document workflow platform, enterprise scale  
> **Stack:** React, Next.js, NestJS, PostgreSQL, Redis, AWS  
> **Điều kiện:** 35h/week, 2 remote days, MacBook, PVI health care 100M, 13th month salary  
> **HR yêu cầu:** Chuẩn bị self-introduction, experience, background, behavioral — **bằng tiếng Anh**

---

## Mục lục

**PHẦN 1 — TRƯỚC KHI VÀO PHỎNG VẤN**
1. [Company Research — học thuộc](#1-company-research)
2. [Self-Introduction — 3 phiên bản EN + VI](#2-self-introduction)
3. [Experience Walkthrough — kể story từng job](#3-experience-walkthrough)

**PHẦN 2 — BEHAVIORAL (soft skills)**
4. [Behavioral Questions — STAR format EN + VI](#4-behavioral-questions)
5. [Why Designveloper / Why Lumin PDF](#5-why-designveloper--why-lumin-pdf)

**PHẦN 3 — TECHNICAL (hard skills)**
6. [React.js — Core Concepts](#6-reactjs--core-concepts)
7. [Next.js — SSR/SSG/ISR/CSR](#7-nextjs--ssrssgisr)
8. [Performance Optimization](#8-performance-optimization)
9. [Unit Testing với Jest](#9-unit-testing-với-jest)
10. [REST API Integration](#10-rest-api-integration)
11. [CSS Animations & Responsive](#11-css-animations--responsive)
12. [State Management](#12-state-management)
13. [Git & GitFlow](#13-git--gitflow)

**PHẦN 4 — KẾT THÚC PHỎNG VẤN**
14. [Câu hỏi hỏi ngược lại](#14-câu-hỏi-hỏi-ngược-lại)
15. [Salary Expectation](#15-salary-expectation)
16. [Checklist ngày phỏng vấn](#16-checklist-ngày-phỏng-vấn)
17. [JD Match & Key Phrases](#17-jd-match--key-phrases)

---

# PHẦN 1 — TRƯỚC KHI VÀO PHỎNG VẤN

## 1. Company Research

> Interviewer **sẽ hỏi** "What do you know about us?" — học thuộc phần này.

### Designveloper (DSV)

| | |
|---|---|
| **Thành lập** | 2013, Ho Chi Minh City |
| **Loại** | Software development — outsourcing + own products |
| **Founder** | Hung Vo (CEO): *"To work hard is a method of personality training"*; Ha Truong (Co-founder): *"Be Thankful in All Things"* |
| **Dịch vụ** | AI-Powered Software, Custom Software, Web/Mobile App, VoIP, Cyber Security, UI/UX |
| **Sản phẩm nổi bật** | Lumin PDF, Walrus Education, Joyn'it, Bonux, Chloe Ting Fitness Hub |
| **Khách hàng** | Chủ yếu US/UK |
| **Tech** | React, React Native, Node.js, NestJS |
| **Culture** | Young, creative, fun. 35h/week, 2 remote days, MacBook, free food & drinks, team trips |
| **Rating** | 4.9/5 Glassdoor — ít overtime, môi trường tốt |

### Lumin PDF

| | |
|---|---|
| **Là gì** | Cloud-based document workflow platform — *"The new standard for agreements"* |
| **Core features** | PDF editing, annotation, digital eSignature (legally binding), AI tools (AgreementGen, PDF chat & summarize), real-time collaboration, mobile iOS/Android |
| **Enterprise users** | Spotify, Salesforce, PayPal, Uber, Costco |
| **Security** | SOC 2 compliant, bank-level encryption, audit trails, zero-trust architecture |
| **Integrations** | Google Drive, OneDrive, Dropbox, Slack, Salesforce, HubSpot, Monday.com, **Claude AI** |
| **Tech stack** | Frontend: **React** · Backend: Node.js, **NestJS**, MongoDB, PostgreSQL, Redis, Elasticsearch, AWS |
| **Client** | Max Ferguson (Founder, NitroLabs / LuminPDF) praised DSV's work quality |

**Key insight để nói trong phỏng vấn:**
> Lumin không phải tool đơn giản — đây là enterprise-grade agreement platform với AI, real-time collaboration, và SOC 2 compliance. Frontend phức tạp thực sự: PDF rendering (PDF.js + canvas), annotation overlays sync với canvas khi zoom/scroll, và responsive editor cho desktop + mobile.

---

## 2. Self-Introduction

> 3 phiên bản. **Luyện nói to, không đọc mắt.** Version B là quan trọng nhất.

---

### Version A — 30 giây (HR phone screen)

**English:**
> "Hi, I'm Nam Vinh, a Software Engineer based in Ho Chi Minh City with five years of experience in React, React Native, and TypeScript. Currently at Herond Labs, I'm building a privacy-focused browser ecosystem on Chromium — I own the Design System with 70-plus components, built Module Federation micro-frontends for independent deployment, and contributed to full-stack features including an OAuth2/OIDC identity platform and a NestJS-based admin portal. I'm excited about Lumin PDF because it's a product at real enterprise scale, and the kind of frontend complexity — PDF rendering, real-time collaboration, performance — is exactly the type of work I want to grow in."

**Tiếng Việt:**
> Tôi là Nam Vinh, Software Engineer HCMC, 5 năm React, React Native, TypeScript. Tại Herond Labs, tôi đang build privacy browser ecosystem trên Chromium — sở hữu Design System 70+ components, build Module Federation micro-frontends, và contribute full-stack gồm OAuth2/OIDC identity platform và NestJS admin portal. Hứng thú Lumin PDF vì scale enterprise và frontend complexity thực sự.

---

### Version B — 90 giây ✅ Standard cho technical round

**English:**
> "Sure, let me give you a quick overview. My name is Nam Vinh, and I've been a Software Engineer for five years, working primarily with React, React Native, and TypeScript.
>
> I started at Zens Technology, a software outsourcing company with a team of six. I worked across multiple client projects — including a React Native app for managing COVID-19 kit testing workflows — and built a solid foundation in React, Redux, GraphQL, and architecting shared component libraries from scratch.
>
> I then moved to Biso24, a digital transformation platform. Over two and a half years, I owned the codebase architecture for both the React Native mobile app and the web application, set up a Turborepo monorepo to share code and components across platforms, developed three core product modules end-to-end, and managed the full App Store and Google Play release cycle.
>
> Most recently, I joined Herond Labs as a Software Engineer on a privacy-focused browser ecosystem built on Chromium, with Web3 integration. There are a few highlights: I built the Design System of 70-plus components, live at storybook.herond.org, backed by Vitest and Playwright tests. I implemented Module Federation micro-frontends for the admin portal with NestJS and Prisma on the backend. And I contributed to the OAuth2/OIDC identity platform using Duende IdentityServer with Web3Auth wallet-based authentication.
>
> What draws me to Lumin PDF is the scale and complexity. A document platform trusted by Spotify and Salesforce, with AI features, real-time collaboration, and SOC 2 compliance — that's technically challenging, and the React and NestJS stack aligns directly with what I've been working with."

**Tiếng Việt (key structure):**
> - Zens: outsourcing, team 6, React/RN, COVID-19 app, shared component library từ đầu
> - Biso24: digital transformation, React Native + Web, Turborepo monorepo, 3 modules, App Store/Play Store
> - Herond Labs: privacy browser + Web3, Design System 70+, Module Federation micro-frontends, OAuth2/OIDC, NestJS Portal
> - Why Lumin: React + NestJS stack match, enterprise scale, frontend complexity

---

### Version C — 2-3 phút (khi interviewer muốn nghe thêm)

**English:**
> "Happy to share more. I'm Nam Vinh, five years as a Software Engineer focused on React, React Native, and TypeScript — and I've worked across three quite different environments.
>
> At Zens Technology, a software outsourcing company, I worked on multiple client projects across different domains. One of the more memorable ones was a React Native application to manage and accelerate COVID-19 kit testing workflows for a healthcare client. Working in outsourcing taught me to read unfamiliar codebases quickly, architect clean foundations that others can maintain without needing context from me, and communicate with both technical and non-technical stakeholders.
>
> At Biso24, I took on more ownership. It was a digital transformation platform serving business clients. I architected the foundation for both the React Native mobile app and the web application — the layout system, the shared component library, the Turborepo monorepo setup for code sharing across platforms. I developed three core product modules end-to-end and handled the full release cycle to App Store and Google Play. That was my first time owning the mobile release process.
>
> At Herond Labs, my current role, I'm a Software Engineer on a larger and more complex system — a privacy-focused browser built on Chromium, with Web3 wallet integration, an OAuth2/OIDC identity provider, a loyalty points platform, and an internal admin portal.
>
> The piece I'm most proud of is the Design System. I built it end-to-end: 70-plus production components in TypeScript with a Vite pipeline for ES and CJS output, tree-shaking, and automated CSS token generation from JSON design sources. It's documented in Storybook at storybook.herond.org, and every component has Vitest unit tests and Playwright browser tests to prevent visual regressions.
>
> I also built the micro-frontend architecture for the admin portal using Module Federation, and contributed to the backend — NestJS with Prisma ORM for the portal, and ASP.NET Core with CQRS and MediatR for the points and identity systems.
>
> What draws me to Lumin PDF: real enterprise scale, complex frontend problems I haven't fully solved before — PDF.js rendering, annotation layers synced to canvas, real-time collaboration — and an active AI roadmap. The React and NestJS stack is also a direct match, so I can be productive quickly."

**Tiếng Việt (3 điều học được từ 3 công ty):**
> - Zens → Adapt nhanh, kiến trúc clean foundation, communicate với stakeholders
> - Biso24 → Codebase ownership, Turborepo monorepo, mobile release cycle (App Store/Play)
> - Herond Labs → Design System end-to-end, Module Federation, full-stack (NestJS + ASP.NET Core)
> - Why Lumin: React + NestJS match, enterprise scale, PDF complexity chưa làm

---

## 3. Experience Walkthrough

> Dùng khi interviewer hỏi "Walk me through your experience" hoặc "Tell me about your current role."

### Herond Labs — nói nhiều nhất

**English:**
> "At Herond Labs, I'm a Software Engineer on a privacy-focused browser ecosystem built on Chromium — it includes a multi-chain Web3 wallet, an OAuth2/OIDC identity platform, a loyalty points system, and an admin portal. The team is 20-plus people.
>
> The piece I own most fully is the Design System — 70-plus components in TypeScript, bundled with Vite for ES and CJS output, with automated CSS token generation from JSON design sources. It's documented in Storybook at storybook.herond.org, and every component is tested with Vitest unit tests and Playwright browser tests to prevent regressions across product updates.
>
> I also built the browser UI pages — Welcome, New Tab, Wallet, Points — using React and Redux directly on Chromium's WebUI layer, shipping across macOS, Windows, Linux, and Android.
>
> On the architecture side, I implemented Module Federation micro-frontends for the admin portal so each team module deploys independently, with NestJS and Prisma ORM on the backend.
>
> I also contributed to backend systems — the identity platform using Duende IdentityServer with OAuth2/OIDC flows, social login, and Web3Auth; and the points system using ASP.NET Core with CQRS, MediatR, and Redis distributed locking."

---

### Biso24

**English:**
> "Biso24 was a digital transformation platform — providing businesses a ready-to-use solution with built-in cloud storage, domain management, and API integration, so they could modernize without infrastructure investment.
>
> I joined a team of ten and took on the codebase architecture early. I designed the foundation for both the React Native mobile app and the React web app — layout system, shared component library, coding conventions. I integrated and customized GlueStack UI to standardize the mobile developer experience, and set up a Turborepo monorepo so the web and mobile codebases could share types, utilities, and components cleanly.
>
> I developed three core product modules end-to-end and owned the full mobile release cycle — building, signing, and submitting to both App Store and Google Play."

---

### Zens Technology

**English:**
> "Zens was a software outsourcing company — a team of six, multiple client projects across different domains. One I remember clearly is a React Native app I built for a healthcare client to manage COVID-19 kit testing workflows. It was used in real operations, which made the quality expectations very concrete.
>
> Working in outsourcing taught me to get productive on unfamiliar codebases quickly, architect foundations that others can maintain without needing context from me, and communicate across technical and non-technical stakeholders."

---

# PHẦN 2 — BEHAVIORAL

## 4. Behavioral Questions

> **STAR format** — Situation, Task, Action, Result. Học structure, đừng học từng chữ.

---

### "Tell me about yourself."
→ Dùng Version B hoặc C từ Section 2.

---

### "What is your greatest strength?"

**English:**
> "My greatest strength is systematic thinking combined with an eye for quality. When I built the Design System, I didn't just write components — I thought about the token architecture, the Storybook docs strategy, the visual regression pipeline, and the contribution workflow for other teams. I care about systems that stay correct over time, not just features that ship on Friday. That's what I think separates a Senior engineer from someone who just completes tasks."

**Tiếng Việt:**
> Tư duy hệ thống + sense of quality. Khi build Design System, nghĩ về cả token architecture, Storybook docs, visual regression pipeline, contribution workflow — không chỉ viết components. Quan tâm đến systems stays correct over time.

---

### "What is your greatest weakness?"

**English:**
> "I can sometimes go too deep into architecture decisions — trying to make things perfect when a simpler solution would work fine. I've learned to set time-boxes: if I can't decide in an hour, I pick the simpler option, document the reasoning, and revisit later if needed. It's helped me ship faster without losing quality."

**Tiếng Việt:**
> Đôi khi over-engineer architecture. Đã học cách đặt time-box 1 tiếng: nếu không quyết định được, chọn cái đơn giản hơn, document lý do, review sau.

---

### "Tell me about a difficult technical challenge."

**English (STAR):**
> **S:** "At Herond Labs, we had a Design System serving four products simultaneously — and every time a component was updated, there was a risk of silently breaking the UI in one of the other products without anyone noticing until it hit production.
>
> **T:** I needed a way to catch unintentional visual regressions automatically before any PR could be merged.
>
> **A:** I set up Playwright browser tests for every component in the Design System, running in CI on every pull request. Each test renders the component in a real browser and compares it against a stored screenshot — if any pixel changes unexpectedly, the CI gate fails and the PR is blocked. The tricky part was making the tests stable: flaky tests from font rendering differences, anti-aliasing, and CI environment differences would cause false failures. I solved this by setting consistent viewport sizes, disabling animations in test mode, and using a pixel-diff threshold to tolerate sub-pixel rendering noise.
>
> **R:** In the first month, the pipeline caught regressions in six pull requests that would have shipped broken UI to production. The team's confidence in updating shared components went up significantly — engineers stopped being afraid to touch the Design System because they knew the tests would catch mistakes."

**Tiếng Việt (key points):**
> S: Design System cho 4 products — component update có thể break UI ở product khác mà không ai biết  
> T: Cần auto-catch visual regressions trước khi merge  
> A: Playwright screenshot tests trong CI, giải quyết flakiness (viewport cố định, disable animations, pixel-diff threshold)  
> R: Tháng đầu catch 6 regressions trước khi ship production. Team tự tin update shared components hơn

---

### "Tell me about a time you disagreed with a teammate."

**English (STAR):**
> **S:** "At Herond Labs, we were deciding between CSS-in-JS and CSS Modules for the Design System. A senior colleague strongly preferred CSS-in-JS.
>
> **T:** Make the right technical call for a system used across four products long-term.
>
> **A:** Instead of arguing, I prepared a written comparison covering runtime overhead, SSR hydration complexity in Next.js, and vendor lock-in — shared it in Slack before the meeting so everyone had context. In the meeting, I acknowledged CSS-in-JS is valid, then focused on our specific constraints: SSR performance and multi-product context.
>
> **R:** Team aligned on CSS Modules. The colleague appreciated data over opinion. Zero CSS architecture issues since launch."

**Tiếng Việt:**
> Không tranh luận trực tiếp — viết bản so sánh với data (runtime overhead, SSR complexity, lock-in), share trước meeting. Kết quả: team chọn CSS Modules, colleague appreciate vì có data.

---

### "Tell me about a time you had to learn something quickly."

**English (STAR):**
> **S:** "At Herond Labs, NestJS was listed as a bonus skill — but in my first month, a backend engineer was sick for two weeks during a critical deadline.
>
> **T:** Deliver two REST API endpoints and a Redis caching layer without backend support.
>
> **A:** I had Node.js knowledge but NestJS was new. Spent two evenings reading only the docs I needed — Controllers, Services, TypeORM, Redis cache module. Used the existing codebase as reference. Paired one hour with the team lead before he left to understand conventions.
>
> **R:** Delivered on time with unit tests. Backend engineer reviewed on return — said it followed conventions correctly. After that, NestJS tasks became a regular part of my role."

**Tiếng Việt:**
> Backend sick 2 tuần, cần deliver NestJS endpoints chưa biết.  
> 2 tối đọc docs (chỉ phần cần) + đọc codebase reference + pair 1h với TL.  
> Result: deliver đúng deadline, code review pass.

---

### "How do you handle feedback / code review?"

**English:**
> "I run daily code reviews myself, so I've thought about this from both sides. When I receive feedback, my default is to assume the reviewer is right and understand their reasoning before defending my approach. If I disagree, I ask a clarifying question — it usually turns out there's context I was missing. Engineers who take code review personally slow the whole team down. The goal is better code together."

**Tiếng Việt:**
> Default: assume reviewer đúng, hiểu reasoning trước khi defend. Nếu không đồng ý: hỏi clarifying question thay vì statement. Mục tiêu là better code, không phải defend cá nhân.

---

### "Describe your Agile experience."

**English:**
> "At Biso24, two-week sprints — daily standups, planning, refinement, retrospectives. I appreciated retros most — where we caught real issues like unclear ticket definitions causing rework. I'd bring specific data: 'three of our five rejected PRs this sprint were due to unclear acceptance criteria' — made discussions actionable. At Herond Labs we're async-first — decisions documented in Notion, not long sync meetings."

---

### "Where do you see yourself in 3 years?"

**English:**
> "In three years, I'd like to be a Tech Lead or Principal Frontend Engineer — someone who shapes architecture decisions, defines quality standards, and helps junior engineers grow. Lumin PDF is a great environment for that. Enterprise scale, complex document workflows, active AI roadmap — genuinely hard problems. I'm not looking for a comfortable role. I'm looking for one that challenges me."

---

### "Why are you leaving your current job?"

**English:**
> "I'm not leaving because of dissatisfaction — Herond Labs has been a great environment. I'm leaving because I've hit a natural transition point. The Design System is built, the CI/testing standards are in place, and the platform is in maintenance mode rather than heavy feature development. I want the next challenge — a product at larger scale with more complex frontend problems. Lumin PDF fits that exactly."

---

## 5. Why Designveloper / Why Lumin PDF

### "Why Designveloper?"

**English:**
> "Three reasons. First, track record — Designveloper has been shipping software since 2013 and has built products at real scale. Lumin PDF is the proof. That's engineering maturity, not a startup throwing code at the wall.
>
> Second, culture alignment. 35-hour week and two remote days signals the company trusts its engineers. I produce better work when I have space to think.
>
> Third, the stack is a direct match — React, Next.js, NestJS, PostgreSQL, Redis. I won't spend three months ramping up. I can focus on the product from day one."

---

### "Why Lumin PDF specifically?"

**English:**
> "Three specific reasons. First, technical complexity — PDF editing is genuinely hard frontend work. You're not working with standard DOM elements; you're dealing with canvas layers, PDF.js rendering, annotation overlays, keeping them synced at different zoom levels. Most developers never encounter that level of complexity.
>
> Second, scale — enterprise clients like Spotify and Salesforce mean performance decisions have real impact. A 200ms improvement in Time to Interactive matters at tens of millions of users.
>
> Third, the AI roadmap — AgreementGen, PDF chat, Claude integration — means the frontend will keep evolving in interesting directions. I want to be part of building that."

---

# PHẦN 3 — TECHNICAL

## 6. React.js — Core Concepts

### Q: Khi nào component re-render, cách prevent?

Component re-render khi: **state thay đổi, props thay đổi, context thay đổi, parent re-render**.

```tsx
// 1. React.memo — skip re-render nếu props không đổi
const AnnotationLayer = React.memo(({ annotations, pageNumber }) => {
  return (/* render */);
});

// 2. useMemo — memoize expensive calculation
const visibleAnnotations = useMemo(
  () => annotations.filter(a => a.page === currentPage),
  [annotations, currentPage]
);

// 3. useCallback — stable function reference
const handleAnnotationClick = useCallback((id: string) => {
  setSelectedId(id);
}, []);

// ❌ object inline → tạo mới mỗi render → AnnotationLayer re-render
<AnnotationLayer style={{ position: 'absolute' }} />

// ✅ move ra ngoài
const overlayStyle = { position: 'absolute' as const };
<AnnotationLayer style={overlayStyle} />
```

---

### Q: `useEffect` vs `useLayoutEffect`?

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Timing | Async, **sau** browser paint | Sync, **trước** browser paint |
| Blocking | Không block UI | Block paint cho đến khi xong |
| Dùng cho | Data fetch, subscriptions | DOM measurements, prevent layout flash |
| SSR | Safe | Warning — không chạy trên server |

```tsx
// useLayoutEffect: position tooltip trước khi user thấy
// Nếu dùng useEffect → tooltip flash vị trí sai trước khi correct
useLayoutEffect(() => {
  const anchor = anchorRef.current.getBoundingClientRect();
  const tooltip = tooltipRef.current.getBoundingClientRect();
  setPosition({
    top: anchor.bottom + 8,
    left: anchor.left + anchor.width / 2 - tooltip.width / 2,
  });
}, []);
```

---

### Q: Custom hook cho PDF page virtualization?

```tsx
// Chỉ render pages visible trong viewport + buffer
const usePdfVirtualization = (totalPages: number, pageHeight: number) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const [visibleRange, setVisibleRange] = useState({ start: 0, end: 5 });
  const BUFFER = 2;

  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;

    const handleScroll = () => {
      const { scrollTop, clientHeight } = container;
      const start = Math.max(0, Math.floor(scrollTop / pageHeight) - BUFFER);
      const end = Math.min(totalPages - 1, Math.ceil((scrollTop + clientHeight) / pageHeight) + BUFFER);
      setVisibleRange({ start, end });
    };

    container.addEventListener('scroll', handleScroll, { passive: true });
    return () => container.removeEventListener('scroll', handleScroll);
  }, [totalPages, pageHeight]);

  return { containerRef, visibleRange };
};
```

---

## 7. Next.js — SSR/SSG/ISR

### Q: Page nào của Lumin dùng strategy nào?

| Page | Strategy | Lý do |
|---|---|---|
| Landing/marketing | **SSG** | Static, SEO critical |
| Blog/help articles | **ISR** `revalidate: 3600` | Mostly static, update occasionally |
| PDF Editor (main app) | **CSR** `ssr: false` | Canvas-based, auth required |
| Dashboard (user files) | **SSR** | Per-user data, auth check server-side |
| Shared PDF viewer | **SSR** | Public URL, SEO + OG tags cần real data |

```tsx
// Public shared PDF — SSR
export default async function SharedPdfPage({ params }) {
  const pdf = await fetchPdfMetadata(params.id);
  if (!pdf.isPublic) notFound();
  return <PdfViewerShell initialData={pdf} />;
}

// PDF Editor — CSR (lazy load vì PDF.js ~700KB)
const PdfEditor = dynamic(() => import('@/components/PdfEditor'), {
  loading: () => <EditorSkeleton />,
  ssr: false,
});
```

---

### Q: Server Components vs Client Components?

| | Server Components | Client Components (`'use client'`) |
|---|---|---|
| Runs on | Server only | Server (init) + Client (hydration) |
| Bundle size | Không thêm JS client | Thêm JS vào client bundle |
| State/Effects | Không có hooks | Đầy đủ React hooks |
| Direct DB | ✅ | ❌ |
| Browser APIs | ❌ | ✅ |

```tsx
// Server Component — fetch trực tiếp, no useEffect
async function DashboardPage() {
  const files = await getUserFiles(); // server-side
  return (
    <div>
      <FileGrid files={files} />     {/* server component */}
      <UploadButton />               {/* client component */}
    </div>
  );
}

// Client Component
'use client';
export function UploadButton() {
  const [uploading, setUploading] = useState(false);
  return <button onClick={() => setUploading(true)}>Upload PDF</button>;
}
```

---

## 8. Performance Optimization

### Q: Optimize initial load cho Next.js app?

```tsx
// 1. Code splitting — lazy load nặng
const PdfEditor = dynamic(() => import('@/components/PdfEditor'), {
  loading: () => <EditorSkeleton />,
  ssr: false,
});

// 2. PDF.js lazy load — chỉ load khi user mở PDF
const loadPdfLib = () => import('pdfjs-dist'); // ~700KB

// 3. Bundle analysis
// ANALYZE=true npm run build → treemap

// 4. Font optimization
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'], display: 'swap' }); // zero layout shift
```

---

### Q: Infinite scroll với large list?

```tsx
// IntersectionObserver + React Query infinite
const useInfiniteScroll = (onLoadMore: () => void) => {
  const sentinelRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) onLoadMore(); },
      { threshold: 0.1 }
    );
    if (sentinelRef.current) observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [onLoadMore]);

  return sentinelRef;
};

const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['files'],
  queryFn: ({ pageParam = 0 }) => fetchFiles({ cursor: pageParam, limit: 20 }),
  getNextPageParam: (lastPage) => lastPage.nextCursor,
});

const sentinelRef = useInfiniteScroll(() => {
  if (hasNextPage && !isFetchingNextPage) fetchNextPage();
});
```

---

## 9. Unit Testing với Jest

### Q: Unit test cho component fetch data?

```tsx
// FileList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { server } from '@/mocks/server';
import { rest } from 'msw';

beforeEach(() => {
  server.use(
    rest.get('/api/files', (req, res, ctx) =>
      res(ctx.json({ files: [{ id: '1', name: 'report.pdf', size: 1024 }] }))
    )
  );
});

test('renders file list after fetching', async () => {
  render(<FileList />);
  expect(screen.getByRole('status')).toBeInTheDocument(); // loading spinner
  await waitFor(() => expect(screen.getByText('report.pdf')).toBeInTheDocument());
  expect(screen.getByText('1 KB')).toBeInTheDocument();
});

test('shows empty state', async () => {
  server.use(rest.get('/api/files', (req, res, ctx) => res(ctx.json({ files: [] }))));
  render(<FileList />);
  await waitFor(() => expect(screen.getByText(/no documents yet/i)).toBeInTheDocument());
});

test('shows error state', async () => {
  server.use(rest.get('/api/files', (req, res, ctx) => res(ctx.status(500))));
  render(<FileList />);
  await waitFor(() => expect(screen.getByText(/something went wrong/i)).toBeInTheDocument());
});
```

---

### Q: Mock vs Spy vs Stub?

| | Mô tả | Dùng khi |
|---|---|---|
| **Mock** | Thay thế hoàn toàn module/function | API calls, external services |
| **Spy** | Wrap function thực, track calls | Verify function được gọi đúng |
| **Stub** | Control return value cụ thể | Unit test với dependency output |

```ts
jest.mock('@/lib/analytics', () => ({ trackEvent: jest.fn() })); // mock

const consoleSpy = jest.spyOn(console, 'error').mockImplementation(() => {}); // spy
expect(consoleSpy).toHaveBeenCalledWith(expect.stringContaining('Upload failed'));
consoleSpy.mockRestore();

const getFileSizeMock = jest.fn().mockReturnValueOnce(1024).mockReturnValueOnce(2048); // stub
```

---

## 10. REST API Integration

### Q: Error handling + retry + loading states?

```tsx
// TanStack Query — industry standard
const { data, isLoading, isError } = useQuery({
  queryKey: ['file', fileId],
  queryFn: () => fetchFile(fileId),
  retry: 3,
  retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30000), // exponential backoff
  staleTime: 5 * 60 * 1000,
});

const uploadMutation = useMutation({
  mutationFn: async (file: File) => {
    const res = await fetch('/api/upload', {
      method: 'POST',
      body: new FormData(),
      signal: AbortSignal.timeout(30_000),
    });
    if (!res.ok) throw new Error(`Upload failed: ${res.status}`);
    return res.json();
  },
  onError: (error) => toast.error(error.message),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['files'] });
    toast.success('PDF uploaded successfully');
  },
});
```

---

### Q: File upload với progress bar?

```tsx
const useFileUpload = () => {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState<'idle' | 'uploading' | 'done' | 'error'>('idle');
  const abortRef = useRef<AbortController>();

  const upload = (file: File) => {
    setStatus('uploading');
    abortRef.current = new AbortController();

    return new Promise<void>((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      xhr.open('POST', '/api/upload');
      xhr.upload.onprogress = (e) => {
        if (e.lengthComputable) setProgress(Math.round((e.loaded / e.total) * 100));
      };
      xhr.onload = () => {
        if (xhr.status < 300) { setStatus('done'); resolve(); }
        else { setStatus('error'); reject(new Error(`${xhr.status}`)); }
      };
      xhr.onerror = () => { setStatus('error'); reject(new Error('Network error')); };
      abortRef.current.signal.addEventListener('abort', () => xhr.abort());
      const form = new FormData();
      form.append('file', file);
      xhr.send(form);
    });
  };

  return { upload, cancel: () => abortRef.current?.abort(), progress, status };
};
```

---

## 11. CSS Animations & Responsive

### Q: Tại sao chỉ animate `transform` và `opacity`?

Browser rendering pipeline: **Style → Layout → Paint → Composite**

- `width`, `height`, `top` → trigger **Layout** (reflow) — tốn nhất
- `background-color` → trigger **Paint**
- `transform`, `opacity` → chỉ **Composite** — GPU, không touch DOM → 60fps

```css
/* ❌ Gây reflow */
.bad { transition: left 300ms, width 300ms; }

/* ✅ GPU composite only */
.good { transition: transform 300ms cubic-bezier(0.4, 0, 0.2, 1); }

/* PDF page transition */
.pdf-page {
  position: absolute;
  inset: 0;
  transform: translateX(100%);
  opacity: 0;
  transition: transform 300ms cubic-bezier(0.4, 0, 0.2, 1), opacity 200ms ease;
}
.pdf-page.active { transform: translateX(0); opacity: 1; }

/* Respect reduced motion */
@media (prefers-reduced-motion: reduce) {
  .pdf-page { transition: opacity 150ms ease; transform: none !important; }
}
```

---

### Q: Mobile touch-first cho PDF editor?

```css
@media (max-width: 768px) {
  .editor-layout {
    display: grid;
    grid-template-rows: auto 1fr auto;
    grid-template-areas: "header" "canvas" "toolbar";
    height: 100dvh; /* tránh address bar overlap */
  }

  .toolbar {
    overflow-x: auto;
    scrollbar-width: none;
    padding: env(safe-area-inset-bottom); /* iPhone home bar */
  }

  /* Touch targets: minimum 44×44px (Apple HIG) */
  .toolbar-button {
    min-width: 44px;
    min-height: 44px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
```

---

## 12. State Management

### Q: State phân loại cho PDF editor?

```tsx
// 1. UI State (local) — component-level
useState() // modal open, hover, input value

// 2. Server State (TanStack Query)
useQuery()    // file list, user data
useMutation() // upload, save, delete

// 3. App State (Zustand) — cross-component
const useEditorStore = create<EditorStore>((set) => ({
  currentPage: 1,
  zoom: 1,
  selectedTool: 'select' as const,
  annotations: [] as Annotation[],
  selectedAnnotationId: null as string | null,

  setPage: (page) => set({ currentPage: page }),
  setZoom: (zoom) => set({ zoom: Math.max(0.5, Math.min(3, zoom)) }),
  addAnnotation: (annotation) =>
    set((s) => ({ annotations: [...s.annotations, annotation] })),
  updateAnnotation: (id, changes) =>
    set((s) => ({ annotations: s.annotations.map((a) => a.id === id ? { ...a, ...changes } : a) })),
  deleteAnnotation: (id) =>
    set((s) => ({ annotations: s.annotations.filter((a) => a.id !== id) })),
  selectAnnotation: (id) => set({ selectedAnnotationId: id }),
}));
```

---

## 13. Git & GitFlow

### Q: Git workflow trong team?

```bash
# Feature branch từ develop
git checkout -b feature/pdf-annotation-highlight develop

# Conventional Commits
git commit -m "feat(annotation): add highlight annotation tool"
git commit -m "fix(upload): handle network timeout on large files"
git commit -m "test(editor): add unit tests for zoom calculation"

# Rebase để keep history clean
git fetch origin && git rebase origin/develop

# Squash trước khi merge
git rebase -i HEAD~3

# Force push sau rebase (safe)
git push --force-with-lease
```

---

# PHẦN 4 — KẾT THÚC PHỎNG VẤN

## 14. Câu hỏi hỏi ngược lại

> Hỏi ít nhất 3 câu. Thể hiện bạn đã research và serious.

### Về sản phẩm
1. **"Lumin PDF's frontend — is it using Next.js with SSR, or primarily a client-side SPA? And is the team on App Router or Pages Router?"**
2. **"The AI features — AgreementGen, PDF chat — how are they integrated on the frontend? Streaming via SSE or WebSocket?"**
3. **"What's the biggest frontend performance challenge on Lumin right now — Core Web Vitals, bundle size, or something in the PDF rendering layer?"**

### Về team và process
4. **"How large is the frontend team on Lumin, and how is work structured — feature squads or a shared team?"**
5. **"What does the code review process look like? Are there automated CI gates — type checking, test coverage — before PRs can merge?"**
6. **"The JD mentions backend tasks as a bonus. In practice, how often does that come up, and which part of the NestJS backend would I likely touch?"**

### Về growth
7. **"What would the first 30, 60, and 90 days look like for someone in this role?"**

---

## 15. Salary Expectation

**English:**
> "Based on my five years of experience and the scope of this role on a product at Lumin's scale, I'm targeting around [X]–[Y] USD net per month. I'm open to discussion — the total package including benefits and remote flexibility matters too. Could you share the budget range for this position?"

**Tiếng Việt:**
> 5 năm kinh nghiệm + scope của role tại scale Lumin → target [X]–[Y] USD net/tháng. Flexible với total package. Bạn có thể share budget range không?

**Mức tham khảo (2026, HCMC):**
- Mid-Senior React (3-5 năm): $1,500–$2,500 gross
- Senior React (5+ năm): $2,000–$3,500 gross
- DSV đăng tuyển: up to $2,500 USD

---

## 16. Checklist ngày phỏng vấn

### Tối hôm trước
- [ ] Đọc lại Section 1 (company context) — DSV và Lumin PDF
- [ ] Luyện nói to Version B self-introduction ít nhất 3 lần
- [ ] Xem lại 5 behavioral answers theo STAR
- [ ] **Vào luminpdf.com, upload PDF, thử edit** — để nói "I tried the product yesterday and noticed..."
- [ ] Chuẩn bị 3 câu hỏi hỏi ngược lại

### Ngày phỏng vấn
- [ ] Vào trước 5-10 phút
- [ ] CV mở sẵn, tắt thông báo điện thoại
- [ ] Uống nước — giọng rõ hơn

### Trong phỏng vấn
- [ ] Không hiểu: **"Could you clarify what you mean by...?"**
- [ ] Cần suy nghĩ: **"That's a good question, let me think for a moment."**
- [ ] Không biết: **"I haven't used that specifically, but here's how I'd approach it..."**
- [ ] Mỗi câu trả lời kết bằng **result cụ thể** — đừng để câu trả lời lơ lửng

---

## 17. JD Match & Key Phrases

### JD Match

| JD Requirement | Your Experience |
|---|---|
| 1-3+ năm React | ✅ 5 năm React + React Native — over-qualify → negotiate salary cao hơn |
| Next.js SSR/SSG/ISR | ✅ 3 năm Next.js (skills section) |
| TypeScript | ✅ 5 năm |
| Jest unit testing | ✅ Vitest + Playwright ở Herond Labs (CI pipeline) |
| REST API integration | ✅ React Query, RTK Query, Apollo GraphQL |
| GraphQL (bonus) | ✅ Apollo GraphQL tại Zens + Biso24 |
| CSS animations, responsive | ✅ CSS Modules, SASS, responsive cho mobile + web |
| Git/GitFlow | ✅ GitHub Actions CI/CD tại Herond Labs |
| **Backend tasks (bonus)** | ✅ **NestJS (Portal) + ASP.NET Core (Points/Accounts)** |
| State management | ✅ Redux/RTK Query, Zustand, React Query |
| Mobile (bonus — Lumin có iOS/Android) | ✅ React Native 5 năm — BIG BONUS |

**Top 3 điểm bán mạnh nhất:**
1. **React 5 năm + React Native 5 năm** — họ tìm 1-3+ React → over-qualify mạnh. React Native = bonus vì Lumin có mobile app
2. **Backend thực tế: NestJS + ASP.NET Core** → "be able to handle backend tasks" không candidate nào có depth như vậy
3. **Design System ownership end-to-end** (70+ components, Storybook live, Playwright visual regression) → quality engineering culture rõ ràng

### Key Phrases tiếng Anh tự nhiên

| Tình huống | Phrase |
|---|---|
| Mở đầu | *"Sure, happy to walk you through that."* |
| Clarify câu hỏi | *"Just to make sure I understand — are you asking about...?"* |
| Trả lời technical | *"The way I think about this is..."* |
| Kể experience | *"A specific example that comes to mind is..."* |
| Nói trade-offs | *"The trade-off here is... and given our constraints, we chose..."* |
| Kể kết quả | *"The outcome was... which meant..."* |
| Hỏi ngược lại | *"I'd love to understand more about..."* |
| Kết thúc | *"Thank you — I really enjoyed this conversation. I'm genuinely excited about the role."* |

---

## Ôn tập nhanh — links đến core files

| Topic | File | Độ ưu tiên |
|---|---|---|
| React hooks, performance | [react.md](./react.md) | ⭐⭐⭐ Must |
| Next.js SSR/App Router | [nextjs.md](./nextjs.md) | ⭐⭐⭐ Must |
| Jest, RTL, MSW | [testing.md](./testing.md) | ⭐⭐⭐ Must |
| RTK Query, React Query, Zustand | [state-management.md](./state-management.md) | ⭐⭐ High |
| CSS responsive, animations | [css-styling.md](./css-styling.md) | ⭐⭐ High |
| Core Web Vitals, bundle opt | [performance.md](./performance.md) | ⭐⭐ High |
| TypeScript generics | [typescript.md](./typescript.md) | ⭐ Medium |
