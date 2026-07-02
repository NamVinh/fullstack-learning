# Phase 2 — npm & Package Management

npm (Node Package Manager) là hệ sinh thái package lớn nhất thế giới với hơn 2 triệu packages. Nắm vững npm giúp bạn quản lý dependencies hiệu quả, tránh những lỗi phổ biến về versioning, và hiểu cách tổ chức dự án Node.js đúng cách.

---

## npm Basics

```bash
# Khởi tạo project
npm init -y                    # tạo package.json với defaults

# Cài đặt packages
npm install express            # install + thêm vào dependencies
npm install -D vitest          # --save-dev: devDependencies
npm install -g nodemon         # global: có thể chạy từ bất kỳ đâu
npm install express@4.18.0     # version cụ thể
npm install express@latest     # latest version

# Xóa packages
npm uninstall express
npm uninstall -D vitest

# Update packages
npm update                     # update tất cả (theo semver constraints)
npm update express             # update cụ thể
npm outdated                   # xem packages nào có version mới hơn

# Xem packages đã cài
npm list --depth=0             # top-level packages
npm list express               # xem version của express

# Chạy scripts
npm run dev
npm test                       # shortcut cho npm run test
npm start                      # shortcut cho npm run start
```

---

## package.json — Cấu hình Project

```json
{
  "name": "my-api",
  "version": "1.0.0",
  "description": "REST API built with Node.js",
  "main": "src/index.js",
  "type": "module",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "build": "tsc",
    "db:migrate": "prisma migrate dev",
    "db:seed": "node prisma/seed.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "prisma": "^5.0.0"
  },
  "devDependencies": {
    "vitest": "^1.0.0",
    "nodemon": "^3.0.0",
    "@types/express": "^4.17.0"
  },
  "engines": {
    "node": ">=20.0.0",
    "npm": ">=10.0.0"
  },
  "keywords": ["api", "nodejs", "express"],
  "author": "Your Name <your@email.com>",
  "license": "MIT"
}
```

---

## Semantic Versioning (SemVer)

Format: `MAJOR.MINOR.PATCH`

| Thay đổi | Version bump |
|---------|-------------|
| Breaking changes (API cũ không còn hoạt động) | MAJOR: 1.x.x → 2.0.0 |
| New features, backward compatible | MINOR: 1.1.x → 1.2.0 |
| Bug fixes, backward compatible | PATCH: 1.1.1 → 1.1.2 |

```
Version ranges trong package.json:
^1.2.3  → >=1.2.3 <2.0.0  (caret — minor + patch updates OK)
~1.2.3  → >=1.2.3 <1.3.0  (tilde — patch updates only)
1.2.3   → exact: phải đúng version này
*       → any version (NGUY HIỂM — tránh dùng)
>=1.2.0 → bất kỳ version >= 1.2.0
1.2.x   → tương đương ~1.2.0

Ví dụ:
"express": "^4.18.0"  → sẽ cài 4.18.x hoặc 4.19.x, KHÔNG cài 5.0.0
"express": "~4.18.0"  → sẽ cài 4.18.x, KHÔNG cài 4.19.0
```

---

## package-lock.json

```
package.json         → khai báo version ranges (fuzzy)
package-lock.json    → lock exact version mỗi dependency và sub-dependency
```

- **Luôn commit** `package-lock.json` vào git
- `npm ci` dùng lock file (KHÔNG update packages) — dùng trong CI/CD
- `npm install` có thể update packages theo constraints

```bash
npm ci          # install từ lock file (deterministic, nhanh hơn trong CI)
npm install     # install và có thể update lock file
```

---

## .npmrc — Cấu hình npm

```ini
# .npmrc trong project root
registry=https://registry.npmjs.org/
save-exact=true              # luôn save exact version (không có ^ hay ~)
engine-strict=true           # fail nếu Node version không match engines field
```

---

## npx — Run Packages Without Installing

```bash
# Chạy package mà không cần cài global
npx create-react-app my-app
npx create-next-app my-next-app

# Chạy local package (thay vì ./node_modules/.bin/...)
npx prisma migrate dev
npx vitest

# Chạy version cụ thể
npx node@18 --version
```

---

## Alternatives to npm

| Tool | Điểm nổi bật | Khi nào dùng |
|------|-------------|-------------|
| **yarn** | Parallel installs, workspaces tốt, `.yarnrc.yml` | Monorepos, teams đã dùng yarn |
| **pnpm** | Symlinks — tiết kiệm disk (không duplicate packages), strict | Projects nhiều packages, tiết kiệm disk |
| **bun** | Siêu nhanh (Zig), all-in-one: runtime + bundler + test + package manager | Khi muốn tốc độ install tối đa |

```bash
# pnpm (ví dụ)
npm install -g pnpm
pnpm install
pnpm add express
pnpm run dev

# bun
curl -fsSL https://bun.sh/install | bash
bun install
bun add express
bun run dev
```

---

## Workspaces (Monorepo)

```json
// package.json ở root
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": ["packages/*", "apps/*"]
}
```

```bash
# Structure
my-monorepo/
├── package.json          ← root
├── packages/
│   ├── shared-utils/     ← shared code
│   └── ui-components/
└── apps/
    ├── api/             ← backend
    └── web/             ← frontend

# Run script trong specific workspace
npm run build --workspace=packages/shared-utils
npm run dev --workspace=apps/api
```

---

## Bài tập thực hành

1. **Project setup**: Tạo Node.js project từ scratch — `package.json`, scripts (start, dev, test, lint), `.gitignore`, `.nvmrc` (pin Node version), `.npmrc`.

2. **SemVer quiz**: Với dependency `"lodash": "^4.17.11"`, liệt kê 3 versions sẽ được cài và 3 versions sẽ KHÔNG được cài. Giải thích tại sao.

3. **Publish package**: Tạo npm package đơn giản (ví dụ: `date-formatter`) với README, exports đúng chuẩn, publish lên npm (tạo tài khoản npm nếu chưa có).

4. **Audit & update**: Trong một project có nhiều dependencies, chạy `npm audit`, `npm outdated`, fix vulnerabilities, update packages một cách an toàn.

---

## Resources

- [npm documentation](https://docs.npmjs.com) — Official npm docs
- [semver.npmjs.com](https://semver.npmjs.com) — Interactive SemVer calculator
- [pnpm documentation](https://pnpm.io) — pnpm guide
