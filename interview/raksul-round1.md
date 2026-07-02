# RAKSUL Vietnam — Round 1 Interview Script

**Role:** Frontend Engineer (Banking)  
**Round:** 1 — Background Check & Culture Fit Assessment  
**Time:** 16:00 – 17:00, Friday June 26 2026  
**Language:** 100% English  
**Location:** 34th floor, Lim Tower 1, 9-11 Ton Duc Thang, District 1, HCMC

---

## Company Context

- Japanese SaaS company — transformed print & logistics industries in Japan (JPY 50B+ GMV)
- Now building **RAKSUL Bank** — challenger bank for corporate services in Japan, MVP completed end of 2025
- Vision: *"Better Systems, Better World"*
- Stack: Vue.js + TypeScript (Frontend), Ruby on Rails or TypeScript (Backend), AWS

---

## Interview Flow

| # | Topic | Time |
|---|-------|------|
| 1 | Self-introduction | ~5 min |
| 2 | Why RAKSUL / Why banking | ~10 min |
| 3 | Vue.js gap | ~10 min |
| 4 | Background deep-dive | ~15 min |
| 5 | Culture fit | ~15 min |
| 6 | Salary & logistics | ~5 min |

---

## PART 1 — Self Introduction

**Q: "Please introduce yourself."**

**EN:**
> "Hi, I'm Nam Vinh, a Frontend Engineer with 5 years of experience building production web applications. I've been working mainly with React and TypeScript — most recently at Herond Labs, where I lead frontend development for a browser ecosystem and loyalty platform, including building a shared design system with 70+ components and setting up automated visual regression testing in CI.
>
> Before that, I spent over 2 years at Biso24 building data-heavy SaaS dashboards, where I also drove a codebase refactor to TypeScript strict mode and introduced Micro-Frontend architecture.
>
> I'm drawn to RAKSUL because I want to work on a high-impact product in the fintech space — and the engineering culture here, especially the focus on ownership and improving systems, really aligns with how I think about engineering."

**VI:** Giới thiệu 5 năm kinh nghiệm React/TypeScript. Herond: design system 70+ components, visual regression CI. Biso24: SaaS dashboards, refactor TypeScript strict mode, Micro-Frontend. Lý do apply: muốn làm sản phẩm có impact lớn trong fintech, culture RAKSUL phù hợp cách làm việc.

---

## PART 2 — Why RAKSUL / Why Banking

**Q: "Why are you interested in RAKSUL specifically?"**

**EN:**
> "A few reasons. First, RAKSUL's mission — 'Better Systems, Better World' — isn't just a tagline. They've already transformed print and logistics industries in Japan and are now tackling banking. That's a company that actually delivers on big ideas.
>
> Second, the product itself — RAKSUL Bank is a challenger bank targeting corporate services in Japan. That means complex, high-stakes UI — multi-step workflows, real-time data, strict compliance requirements. That kind of product pushes engineers to write reliable, maintainable code, which is what I care about most.
>
> Third, the team here is globally distributed, English-first, and technically strong — exactly the environment I want to grow in."

**VI:** 3 lý do: (1) RAKSUL đã prove được họ transform được industry lớn. (2) Banking product phức tạp, đòi hỏi code chất lượng cao. (3) Team global, English-first, kỹ thuật mạnh.

---

**Q: "You don't have banking domain experience. Is that a concern?"**

**EN:**
> "Honestly, I see it as a ramp-up, not a blocker. Banking UX at its core is about trust, clarity, and zero-error workflows — the same engineering principles I already apply. I've built loyalty transaction systems, real-time dashboards, and complex multi-step forms with strict validation. The domain knowledge of banking regulations I can learn; the engineering habits I already have."

**VI:** Banking domain học được nhanh. Real-time transactions, complex forms với validation nghiêm ngặt, zero-error UI — đã làm rồi ở Herond và Biso24.

---

## PART 3 — Vue.js Gap (Câu hỏi quan trọng nhất)

**Q: "Our main frontend stack is Vue.js. You've been working with React. Are you comfortable switching?"**

**EN:**
> "Yes — and I want to be transparent about where I am. I haven't used Vue in production, but I've studied Vue 3's Composition API and I find it conceptually close to React hooks. The mental model — reactive state, component lifecycle, props/emits — is the same. The syntax is different, but not the thinking.
>
> In my experience, framework is the smallest part of what makes someone productive. The bigger things — component architecture, state management patterns, TypeScript, testing, performance optimization, code review discipline — I have all of those, and they transfer directly.
>
> I'd expect 2–4 weeks to be productive in Vue, and 2–3 months to be fully comfortable. I'm actively learning it right now."

**VI:** Chưa làm Vue production nhưng đã học Composition API — rất giống React hooks. Framework là phần nhỏ nhất; architecture, TypeScript, testing, performance transfer trực tiếp. 2–4 tuần productive, 2–3 tháng comfortable. Đang học ngay bây giờ.

---

**Q: "What do you know about Vue 3 vs React?"**

**EN:**
> "Both are component-based with a reactivity system. Vue 3's Composition API maps closely to React hooks — `ref`/`reactive` ≈ `useState`, `computed` ≈ `useMemo`, `watch` ≈ `useEffect`. Vue has the advantage of a built-in template syntax that's more explicit about reactivity, while React is more flexible but requires more discipline. Vue's single-file components keep template, script, and style together — which I actually like for maintainability."

**VI:** `ref/reactive` = `useState`, `computed` = `useMemo`, `watch` = `useEffect`. Single-file components gộp template + script + style — dễ maintain. Vue explicit hơn về reactivity, React flexible hơn nhưng cần discipline hơn.

---

## PART 4 — Background Deep-Dive

**Q: "Walk me through your most challenging project."**

**EN:**
> "At Herond Labs, I built a design system from scratch — 70+ components, used across 3–4 products simultaneously. The challenge wasn't just building the components; it was designing APIs that were flexible enough for every product's needs but strict enough to enforce consistency.
>
> I also automated Figma design token sync — so every design update auto-generated TypeScript constants and SCSS variables, eliminating manual drift between design and code. And I set up Playwright visual regression in CI, so every PR was blocked on unintended UI changes before merge.
>
> The real challenge was owning the full pipeline: design → code → test → CI → documentation — for a library that other engineers depended on daily."

**VI:** Design system 70+ components: thách thức là API vừa flexible vừa strict. Tự động hóa Figma token → TypeScript + SCSS. Playwright visual regression CI. Sở hữu toàn bộ pipeline.

---

**Q: "Tell me about a time you improved code quality on your team."**

**EN:**
> "At Biso24, I inherited a codebase that had grown organically without TypeScript strict mode. I proposed and led a gradual migration — starting with the most business-critical modules, adding strict typing incrementally over 3 months without blocking feature delivery. I also introduced ESLint rules and pre-commit hooks so issues were caught locally before CI. The result was a measurable drop in runtime errors and faster onboarding for new engineers because the types served as living documentation."

**VI:** Biso24: migrate sang TypeScript strict mode từng bước trong 3 tháng, không block feature delivery. ESLint + pre-commit hooks. Kết quả: ít runtime errors, onboard mới nhanh hơn.

---

## PART 5 — Culture Fit

**Q: "How do you handle code reviews — giving and receiving feedback?"**

**EN:**
> "I conduct daily code reviews and I try to make feedback specific and actionable, not vague. Instead of 'this is messy,' I'll say 'this component has 3 responsibilities — consider splitting out the data-fetching logic into a custom hook.' I always explain the why behind a suggestion, not just the what.
>
> Receiving feedback: I treat it as free knowledge. Even if I disagree, I engage with the reasoning first before pushing back."

**VI:** Feedback phải specific + lý do rõ ràng. Nhận feedback: treat as free knowledge, hiểu lý do trước khi phản bác.

---

**Q: "How do you work with product managers and designers?"**

**EN:**
> "I try to get involved early — before designs are finalized. It's much cheaper to flag a technical constraint at the wireframe stage than after the design is pixel-perfect. I also ask 'why' frequently — understanding the user's goal often reveals simpler technical solutions. At Herond, I worked directly with designers on our Figma token system, which tightened the feedback loop between design and code significantly."

**VI:** Vào sớm trước khi design finalized. Hỏi "why" để hiểu user goal. Ví dụ: Figma token system với designers ở Herond.

---

**Q: "RAKSUL is a Japanese company. What do you know about Japanese work culture and how do you fit?"**

**EN:**
> "Japanese engineering culture is known for attention to detail, quality over speed, and long-term thinking — which aligns well with how I work. I care about maintainability, documentation, and doing things right rather than just fast. I'm also comfortable with structured communication and clear ownership of responsibilities. That said, I appreciate that RAKSUL Vietnam operates in an English-first environment, so it's a blend of Japanese quality standards with international communication norms."

**VI:** Văn hóa Nhật: detail, quality-first, long-term thinking — match cách mình làm việc. Comfortable với giao tiếp có cấu trúc và ownership rõ ràng.

---

**Q: "Where do you see yourself in 2–3 years?"**

**EN:**
> "I want to grow into a role where I'm driving frontend architecture decisions at a product level — not just implementing features, but shaping how the engineering team builds and scales. At RAKSUL, with the banking product growing fast and the team expanding, I see a real opportunity to do that. I'm also interested in deepening my fintech domain knowledge — understanding compliance requirements, data integrity, and the engineering challenges specific to financial products."

**VI:** Drive frontend architecture decisions, không chỉ implement features. RAKSUL Banking đang grow nhanh → cơ hội rõ ràng. Muốn hiểu sâu fintech engineering.

---

## PART 6 — Salary & Logistics

**Q: "What is your expected salary?"**

**EN:**
> "Based on my 5 years of experience and current market rates for senior frontend roles in HCMC, I'm looking at around [INSERT NUMBER]. That said, I'm open to discussing based on the full package — learning opportunities and growth trajectory matter to me too."

---

**Q: "When can you start?"**

**EN:**
> "I'm currently employed, so I'd need to give 30 days notice. That means I could start from late July / early August. I can be flexible if there's a specific timeline you need."

---

## Questions to Ask the Interviewer

1. > "What does the frontend team's day-to-day look like — how do you balance feature delivery with tech debt and quality?"

2. > "RAKSUL Bank MVP was completed by end of 2025 — what's the main engineering focus for the frontend team right now, post-MVP?"

3. > "How does the Vietnam team collaborate with the Japan core team? What does that look like in practice?"

---

## Key Messages Cheat Sheet

| Topic | Key message |
|-------|-------------|
| Vue.js gap | Transparent + confident: học nhanh, framework không phải vấn đề |
| Banking domain | Không phải blocker, engineering principles transfer trực tiếp |
| Strongest card | Design system, testing culture, daily code reviews, TypeScript strict |
| Culture fit | Quality-first, ownership, Japanese work culture alignment |
| Salary | Chuẩn bị con số cụ thể trước khi vào |
