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
