# Frontend Tooling — Interview Questions (Junior → Senior)

---

## 1. Vite vs Webpack

**Level:** [Mid]

**EN:** Why is Vite faster than Webpack in development? Explain the differences in architecture: ESBuild/Rollup vs Webpack, native ESM dev server, and HMR speed.

**VI:** Tại sao Vite nhanh hơn Webpack trong development? Giải thích sự khác biệt kiến trúc: ESBuild/Rollup vs Webpack, native ESM dev server, và HMR speed.

**Trả lời:**

| | Vite | Webpack |
|--|------|---------|
| Dev server | Native ESM (no bundle) | Bundle trước khi serve |
| Cold start | Rất nhanh (dependencies pre-bundle với ESBuild) | Chậm (bundle toàn bộ) |
| HMR | Cực nhanh (chỉ invalidate module thay đổi) | Chậm hơn khi app lớn |
| Production build | Rollup | Webpack |
| Plugin ecosystem | Nhỏ hơn nhưng đủ | Rất lớn |
| Config complexity | Đơn giản | Phức tạp hơn |

```ts
// ==============================
// Vite Architecture
// ==============================

/*
  Vite dev server:
  1. Không bundle gì khi start — chỉ start server
  2. Browser request file → Vite transform on-demand (TypeScript, JSX, etc.)
  3. Native ESM: browser load modules trực tiếp qua <script type="module">
  4. Dependencies (node_modules): Pre-bundle 1 lần với ESBuild (Go) → rất nhanh

  HMR trong Vite:
  - Chỉ gửi module đã thay đổi qua WebSocket
  - Browser replace module đó, không reload toàn trang
  - State preserved
*/

// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [
    react({
      // Babel fast refresh — HMR cho React
      // babel: { plugins: [...] } — nếu cần Babel transforms
    }),
  ],

  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
    },
  },

  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },

  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
  },

  // Optimize deps được pre-bundled với ESBuild
  optimizeDeps: {
    include: ['react', 'react-dom', '@tanstack/react-query'],
  },
});

// ==============================
// Webpack — tại sao chậm hơn
// ==============================

/*
  Webpack dev server:
  1. Build toàn bộ dependency graph từ entry point
  2. Bundle tất cả modules → 1 hoặc vài bundles
  3. Chỉ sau khi bundle xong mới serve

  Với app lớn (1000+ modules):
  - Webpack cold start: 30-60+ giây
  - Vite cold start: < 1 giây (vì không bundle)

  HMR trong Webpack:
  - Invalidate module + những modules phụ thuộc vào nó
  - Với deep dependency chain, nhiều modules cần rebuild
  - Chậm dần khi app grow
*/
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "Why is Vite so much faster than Webpack in development?"**

**Model Answer:**
"The fundamental difference is that Vite doesn't bundle during development at all. When you start the dev server, Vite just starts it — nothing is compiled upfront. When the browser requests a file, Vite transforms it on demand and serves it as a native ES module. The browser handles the module graph itself.

Webpack takes the opposite approach: it builds a complete bundle before it can serve anything. With a large app, that means waiting 30 to 60 seconds on cold start before you see anything in the browser.

The other key is what Vite uses for dependencies. It pre-bundles `node_modules` once using ESBuild, which is written in Go and is 10 to 100 times faster than JavaScript-based tools. Those pre-bundled dependencies get cached and only re-processed if your lock file changes.

For HMR, Vite sends only the exact module that changed over a WebSocket. Webpack has to trace the whole dependency chain from that module upward, which gets slower as the app grows."

**Trả lời (Tiếng Việt):**
"Điểm khác biệt cơ bản là Vite không bundle trong quá trình development. Khi bạn khởi động dev server, Vite chỉ khởi động server — không có gì được compile trước. Khi browser request một file, Vite transform nó theo yêu cầu và serve nó như một native ES module. Browser tự xử lý module graph.

Webpack đi theo hướng ngược lại: nó build một bundle hoàn chỉnh trước khi serve được bất cứ thứ gì. Với một app lớn, nghĩa là chờ 30 đến 60 giây khi cold start trước khi thấy gì trong browser.

Điểm mấu chốt khác là Vite dùng gì cho dependencies. Nó pre-bundle `node_modules` một lần dùng ESBuild, viết bằng Go và nhanh hơn 10 đến 100 lần so với các JavaScript-based tools. Những pre-bundled dependencies đó được cache và chỉ reprocess nếu lock file thay đổi.

Với HMR, Vite chỉ gửi đúng module đã thay đổi qua WebSocket. Webpack phải trace toàn bộ dependency chain từ module đó ngược lên, dần chậm hơn khi app lớn dần."

---

**Q2 (Mid): "Vite uses ESBuild for development and Rollup for production builds. Why not use ESBuild for both?"**

**Model Answer:**
"This is a deliberate tradeoff. ESBuild is extremely fast but it has some gaps in its output optimization compared to Rollup — things like advanced code splitting strategies, plugin ecosystem depth, and some edge cases in tree shaking.

Rollup is specifically designed for library bundling and it produces very clean, optimized output. It has a mature plugin ecosystem and handles things like CSS code splitting and dynamic import optimization really well.

The reasoning is that in development, speed is the priority. In production, bundle quality and optimization matter more — and that's usually a one-time build, not something you're running constantly. So the tradeoff of a slightly slower production build for better output quality is worth it."

**Trả lời (Tiếng Việt):**
"Đây là một tradeoff có chủ đích. ESBuild cực kỳ nhanh nhưng có một số khoảng trống trong output optimization so với Rollup — những thứ như advanced code splitting strategies, chiều sâu plugin ecosystem, và một số edge cases trong tree shaking.

Rollup được thiết kế đặc biệt cho library bundling và nó tạo ra output rất clean, optimized. Nó có plugin ecosystem trưởng thành và xử lý những thứ như CSS code splitting và dynamic import optimization rất tốt.

Lý luận là: trong development, tốc độ là ưu tiên. Trong production, bundle quality và optimization quan trọng hơn — và đó thường là một lần build, không phải thứ bạn chạy liên tục. Vậy tradeoff của một production build chậm hơn đôi chút để có output quality tốt hơn là đáng."

---

**Q3 (Senior): "What are Vite's limitations compared to Webpack, and when might you still choose Webpack?"**

**Model Answer:**
"Vite's plugin ecosystem is smaller and younger than Webpack's. If you need a very specific loader or plugin that only exists for Webpack, that can be a blocker.

Webpack also supports more advanced Module Federation scenarios. The original Module Federation plugin is Webpack-specific, and while Vite has community plugins for it, they don't cover every use case.

Legacy browser support is another area. Vite targets modern browsers by default. If you need to support IE11 or very old browsers, Webpack with Babel has more mature tooling for that.

And Webpack has more granular control over code splitting and bundle optimization. For large apps with very specific caching and loading strategies, Webpack's configuration flexibility can be an advantage even if it comes with complexity.

That said, for the vast majority of new projects I'd start with Vite. You can always migrate to Webpack if you hit a specific wall."

**Trả lời (Tiếng Việt):**
"Plugin ecosystem của Vite nhỏ hơn và trẻ hơn của Webpack. Nếu bạn cần một loader hay plugin rất cụ thể chỉ tồn tại cho Webpack, đó có thể là một blocker.

Webpack cũng hỗ trợ các advanced Module Federation scenarios hơn. Plugin Module Federation gốc là Webpack-specific, và mặc dù Vite có community plugins cho nó, chúng không cover mọi use case.

Legacy browser support là một lĩnh vực nữa. Vite target modern browsers theo mặc định. Nếu bạn cần hỗ trợ IE11 hoặc các browsers rất cũ, Webpack với Babel có tooling trưởng thành hơn cho điều đó.

Và Webpack có kiểm soát chi tiết hơn về code splitting và bundle optimization. Với các apps lớn có caching và loading strategies rất cụ thể, flexibility cấu hình của Webpack có thể là lợi thế dù đi kèm complexity.

Dù vậy, với phần lớn các project mới tôi sẽ bắt đầu với Vite. Bạn luôn có thể migrate sang Webpack nếu gặp phải wall cụ thể nào đó."

---

**Q4 (Senior): "How does Vite handle environment variables differently from Webpack?"**

**Model Answer:**
"Both use a `.env` file system, but Vite has a specific security convention: only variables prefixed with `VITE_` are exposed to client-side code. Anything without that prefix stays on the server side only.

In the browser bundle, Vite replaces `import.meta.env.VITE_API_URL` with the actual string value at build time, similar to how Webpack's `DefinePlugin` replaces `process.env.VAR`. But Vite uses the import.meta.env namespace to align with the ES module spec rather than the Node.js `process.env` convention.

One practical difference: Webpack lets you use `process.env` everywhere because it has a shim. Vite doesn't polyfill `process.env` by default, so if you're migrating from Webpack you'll need to update any code that uses `process.env.VITE_*` to `import.meta.env.VITE_*`."

**Trả lời (Tiếng Việt):**
"Cả hai đều dùng hệ thống `.env` file, nhưng Vite có một security convention cụ thể: chỉ những variables có prefix `VITE_` mới được expose cho client-side code. Bất cứ thứ gì không có prefix đó chỉ ở phía server.

Trong browser bundle, Vite replace `import.meta.env.VITE_API_URL` bằng giá trị string thực tế tại build time, tương tự cách `DefinePlugin` của Webpack replace `process.env.VAR`. Nhưng Vite dùng namespace import.meta.env để align với ES module spec thay vì convention `process.env` của Node.js.

Một điểm khác biệt thực tế: Webpack cho bạn dùng `process.env` ở khắp nơi vì nó có một shim. Vite không polyfill `process.env` theo mặc định, nên nếu bạn đang migrate từ Webpack bạn sẽ cần update bất kỳ code nào dùng `process.env.VITE_*` thành `import.meta.env.VITE_*`."

---

---

## 2. Webpack Core Concepts

**Level:** [Mid]

**EN:** Explain Webpack's core concepts: entry, output, loaders, plugins, code splitting, and tree shaking.

**VI:** Giải thích các khái niệm cốt lõi của Webpack: entry, output, loaders, plugins, code splitting, tree shaking.

**Trả lời:**

```ts
// webpack.config.ts
import path from 'path';
import HtmlWebpackPlugin from 'html-webpack-plugin';
import MiniCssExtractPlugin from 'mini-css-extract-plugin';
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';
import TerserPlugin from 'terser-webpack-plugin';
import type { Configuration } from 'webpack';

const isDev = process.env.NODE_ENV === 'development';

const config: Configuration = {
  // ==============================
  // 1. ENTRY — điểm bắt đầu build dependency graph
  // ==============================
  entry: {
    main: './src/index.tsx',
    // Multiple entry points
    admin: './src/admin/index.tsx',
  },

  // ==============================
  // 2. OUTPUT — nơi emit bundles
  // ==============================
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isDev ? '[name].js' : '[name].[contenthash:8].js',
    chunkFilename: isDev ? '[name].chunk.js' : '[name].[contenthash:8].chunk.js',
    assetModuleFilename: 'assets/[hash][ext][query]',
    clean: true,   // xóa dist/ trước mỗi build
    publicPath: '/',
  },

  // ==============================
  // 3. LOADERS — transform files (không phải JS)
  // Webpack chỉ hiểu JS/JSON natively
  // ==============================
  module: {
    rules: [
      // TypeScript / JSX
      {
        test: /\.(ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', { targets: 'defaults' }],
              ['@babel/preset-react', { runtime: 'automatic' }],
              '@babel/preset-typescript',
            ],
          },
        },
      },

      // CSS / SCSS
      {
        test: /\.(css|scss)$/,
        use: [
          isDev ? 'style-loader' : MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: { modules: { auto: true } }, // CSS Modules khi file là *.module.css
          },
          'postcss-loader',
          'sass-loader',
        ],
      },

      // Images
      {
        test: /\.(png|jpg|jpeg|gif|svg|webp|avif)$/,
        type: 'asset',   // Webpack 5 Asset Modules
        parser: {
          dataUrlCondition: { maxSize: 8 * 1024 }, // < 8KB → inline base64
        },
      },

      // Fonts
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
        generator: { filename: 'fonts/[hash][ext]' },
      },
    ],
  },

  // ==============================
  // 4. PLUGINS — more powerful transforms, optimization, code injection
  // ==============================
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      favicon: './public/favicon.ico',
    }),

    new MiniCssExtractPlugin({
      filename: isDev ? '[name].css' : '[name].[contenthash:8].css',
    }),

    // Only in analysis mode
    ...(process.env.ANALYZE ? [new BundleAnalyzerPlugin()] : []),
  ],

  // ==============================
  // 5. OPTIMIZATION — code splitting + tree shaking
  // ==============================
  optimization: {
    minimize: !isDev,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: { drop_console: !isDev },
        },
      }),
    ],

    // Code splitting
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor chunk — stable, cached lâu
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          chunks: 'all',
          priority: 10,
        },
        // Common chunk — shared between entries
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },

    // Runtime chunk riêng — prevent vendor hash change khi app code thay đổi
    runtimeChunk: 'single',

    // Tree shaking: sideEffects flag trong package.json
    usedExports: true,
    sideEffects: true,
  },

  resolve: {
    extensions: ['.tsx', '.ts', '.js', '.jsx'],
    alias: { '@': path.resolve(__dirname, 'src') },
  },

  devServer: {
    port: 3000,
    historyApiFallback: true,  // SPA routing
    hot: true,
    compress: true,
    proxy: [{ context: ['/api'], target: 'http://localhost:8080' }],
  },

  devtool: isDev ? 'eval-cheap-module-source-map' : 'source-map',
  mode: isDev ? 'development' : 'production',
};

export default config;
```

---

## 3. Babel

**Level:** [Junior] / [Mid]

**EN:** What is Babel? Explain transpiling, `@babel/preset-env`, `@babel/preset-react`, and the difference between transpilation and polyfilling.

**VI:** Babel là gì? Giải thích transpiling, các presets, và sự khác biệt giữa transpilation và polyfilling.

**Trả lời:**

Babel là một JavaScript compiler: chuyển đổi code từ syntax mới (ES2022+, JSX, TypeScript) sang syntax browsers cũ có thể hiểu.

```ts
// babel.config.ts
import type { TransformOptions } from '@babel/core';

const config: TransformOptions = {
  presets: [
    // ==============================
    // @babel/preset-env — transpile modern JS → target browsers
    // ==============================
    [
      '@babel/preset-env',
      {
        // Tự động tính browsers cần support từ browserslist config
        targets: {
          browsers: ['> 0.5%', 'last 2 versions', 'not dead'],
          // hoặc
          esmodules: true,  // chỉ target browsers hỗ trợ ES modules
        },
        // useBuiltIns: tự động inject polyfills
        useBuiltIns: 'usage', // 'usage' | 'entry' | false
        corejs: 3,            // phiên bản core-js cho polyfills
        // usage: chỉ import polyfills được dùng trong code
        // entry: import tất cả polyfills cho target browsers (to lớn hơn)
        modules: false,  // giữ ES modules để tree shaking work
      },
    ],

    // ==============================
    // @babel/preset-react — JSX transform
    // ==============================
    [
      '@babel/preset-react',
      {
        runtime: 'automatic',  // Không cần import React trong mỗi file (React 17+)
        // runtime: 'classic' — cần import React từng file
        development: process.env.NODE_ENV === 'development',
      },
    ],

    // ==============================
    // @babel/preset-typescript — strip TypeScript types
    // ==============================
    [
      '@babel/preset-typescript',
      {
        isTSX: true,
        allExtensions: true,
      },
    ],
  ],

  plugins: [
    // Decorators (class decorators)
    ['@babel/plugin-proposal-decorators', { legacy: true }],

    // Class properties
    ['@babel/plugin-proposal-class-properties', { loose: true }],

    // React Hot Reload trong development
    ...(process.env.NODE_ENV === 'development' ? ['react-refresh/babel'] : []),
  ],
};

export default config;

/*
  ==============================
  Transpilation vs Polyfilling
  ==============================

  TRANSPILATION: chuyển đổi SYNTAX
  - Arrow functions → function expressions
  - Template literals → string concatenation
  - Destructuring → variable assignment
  - async/await → generator functions
  - JSX → React.createElement()
  - TypeScript → JavaScript

  POLYFILLING: thêm RUNTIME features còn thiếu
  - Array.prototype.includes → define nếu không tồn tại
  - Promise → implement nếu không có
  - fetch → implement nếu không có
  - Object.assign → implement nếu không có
  - Map, Set, Symbol → implement nếu không có

  Babel transpile syntax, không tự polyfill.
  core-js cung cấp polyfills.
  @babel/preset-env với useBuiltIns kết hợp cả hai.
*/

// .browserslistrc — định nghĩa target browsers
// > 0.5%              — usage > 0.5% globally
// last 2 versions     — last 2 major versions of each browser
// Firefox ESR         — latest Firefox Extended Support Release
// not dead            — browsers with official support
// not IE 11           — exclude IE 11
```

---

## 4. ESLint

**Level:** [Junior] / [Mid]

**EN:** How does ESLint work? Explain rules, `extends`, `plugins`, custom rules, `eslint-disable` comments, and CI integration.

**VI:** ESLint hoạt động như thế nào? Giải thích rules, extends, plugins, custom rules, và tích hợp CI.

**Trả lời:**

```ts
// eslint.config.mjs — Flat Config (ESLint v9+)
import js from '@eslint/js';
import typescript from '@typescript-eslint/eslint-plugin';
import typescriptParser from '@typescript-eslint/parser';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import importPlugin from 'eslint-plugin-import';
import jsxA11y from 'eslint-plugin-jsx-a11y';
import prettier from 'eslint-config-prettier';

export default [
  // Base JS rules
  js.configs.recommended,

  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parser: typescriptParser,
      parserOptions: {
        project: './tsconfig.json',
        ecmaFeatures: { jsx: true },
      },
    },
    plugins: {
      '@typescript-eslint': typescript,
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
      'import': importPlugin,
      'jsx-a11y': jsxA11y,
    },
    rules: {
      // TypeScript
      ...typescript.configs.recommended.rules,
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'off',
      '@typescript-eslint/no-floating-promises': 'error',

      // React Hooks
      ...reactHooks.configs.recommended.rules,
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',

      // Imports
      'import/order': ['error', {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
        'newlines-between': 'always',
        alphabetize: { order: 'asc', caseInsensitive: true },
      }],
      'import/no-cycle': 'error',           // circular imports
      'import/no-unused-modules': 'warn',

      // Accessibility
      'jsx-a11y/alt-text': 'error',
      'jsx-a11y/anchor-is-valid': 'error',
      'jsx-a11y/interactive-supports-focus': 'error',

      // General
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'prefer-const': 'error',
      'no-var': 'error',
      'eqeqeq': ['error', 'always'],
    },
  },

  // Disable style rules (Prettier handles formatting)
  prettier,

  // Test files — relax some rules
  {
    files: ['**/*.test.{ts,tsx}', '**/*.spec.{ts,tsx}'],
    rules: {
      '@typescript-eslint/no-explicit-any': 'off',
    },
  },

  // Ignore patterns
  {
    ignores: ['dist/**', 'node_modules/**', '*.config.js', 'coverage/**'],
  },
];

// ==============================
// eslint-disable comments
// ==============================

// Disable for entire file
/* eslint-disable @typescript-eslint/no-explicit-any */

// Disable for next line
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const data: any = legacyResponse;

// Disable for a block
/* eslint-disable no-console */
console.log('Debug information');
/* eslint-enable no-console */

// Inline disable
const result = eval(code); // eslint-disable-line no-eval

// ==============================
// Custom ESLint rule
// ==============================
// rules/no-hardcoded-colors.js
module.exports = {
  meta: {
    type: 'suggestion',
    docs: { description: 'Disallow hardcoded hex colors in JSX style props' },
    schema: [],
  },
  create(context) {
    return {
      JSXAttribute(node) {
        if (node.name.name !== 'style') return;

        // Check value for hex color patterns
        const value = node.value;
        if (value?.type === 'JSXExpressionContainer') {
          const text = context.getSourceCode().getText(value);
          const hexColorPattern = /#[0-9a-fA-F]{3,8}/;
          if (hexColorPattern.test(text)) {
            context.report({
              node,
              message: 'Use CSS variables instead of hardcoded colors',
            });
          }
        }
      },
    };
  },
};

// ==============================
// CI Integration
// ==============================
// package.json scripts
// "lint": "eslint src --ext .ts,.tsx",
// "lint:fix": "eslint src --ext .ts,.tsx --fix",
// "lint:ci": "eslint src --ext .ts,.tsx --max-warnings 0"

// GitHub Actions
// - name: Lint
//   run: npm run lint:ci
```

---

## 5. Prettier

**Level:** [Junior]

**EN:** What is the difference between Prettier (formatting) and ESLint (linting)? How do you resolve conflicts between them?

**VI:** Sự khác biệt giữa Prettier (formatting) và ESLint (linting)? Cách giải quyết conflicts giữa chúng?

**Trả lời:**

```ts
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "jsxSingleQuote": false,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "bracketSameLine": false,   // JSX closing bracket trên dòng riêng
  "arrowParens": "always",    // (x) => x thay vì x => x
  "endOfLine": "lf",
  "overrides": [
    {
      "files": "*.json",
      "options": { "printWidth": 200 }
    }
  ]
}

// .prettierignore
// dist/
// node_modules/
// *.min.js
// CHANGELOG.md

/*
  ESLint vs Prettier:
  - ESLint: tìm BUGs, BAD PATTERNS — no-unused-vars, exhaustive-deps, no-eval
  - Prettier: FORMAT CODE — spacing, quotes, semicolons, line length

  Vấn đề: Một số ESLint rules cũng format code (indent, quotes, semi)
  → Conflicts khi Prettier reformat khác với ESLint expect

  Giải pháp: eslint-config-prettier
  → Tắt tất cả ESLint format rules, chỉ để lint rules
*/

// Install
// npm install --save-dev prettier eslint-config-prettier

// eslint.config.mjs — prettier phải ở cuối (override format rules)
import prettier from 'eslint-config-prettier';

export default [
  // ... other configs
  prettier, // PHẢI ở cuối
];

// ==============================
// Husky + lint-staged — format on commit
// ==============================

// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml,yaml}": ["prettier --write"],
    "*.{css,scss}": ["prettier --write"]
  }
}

// .husky/pre-commit
// #!/bin/sh
// . "$(dirname "$0")/_/husky.sh"
// npx lint-staged

// Setup
// npm install --save-dev husky lint-staged
// npx husky init
// echo "npx lint-staged" > .husky/pre-commit
```

---

## 6. Turborepo

**Level:** [Senior]

**EN:** What is Turborepo? Explain task graph, local and remote caching, pipeline dependencies, and why it's fast.

**VI:** Turborepo là gì? Giải thích task graph, local/remote caching, pipeline dependencies, và tại sao nó nhanh?

**Trả lời:**

Turborepo là một build system cho monorepos, tối ưu bằng cách **cache task outputs** và chạy tasks song song.

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    // ==============================
    // Task definitions
    // ==============================

    "build": {
      // dependsOn: ["^build"] — phải build dependencies trước (topological order)
      "dependsOn": ["^build"],
      // outputs: files được cache
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"],
      // inputs: nếu những file này không thay đổi → dùng cache
      "inputs": ["src/**/*.ts", "src/**/*.tsx", "package.json", "tsconfig.json"]
    },

    "test": {
      "dependsOn": ["build"],    // cần build trước khi test
      "outputs": ["coverage/**"],
      "inputs": ["src/**", "tests/**"],
      "cache": true              // mặc định true
    },

    "lint": {
      // Không có dependsOn — có thể chạy bất kỳ lúc nào
      "outputs": [],
      "inputs": ["src/**/*.ts", "src/**/*.tsx", ".eslintrc*"]
    },

    "dev": {
      "cache": false,             // dev server không cache
      "persistent": true          // long-running task
    },

    "type-check": {
      "dependsOn": ["^build"],
      "outputs": [],
      "cache": true
    }
  }
}
```

```bash
# package.json scripts trong monorepo root
# "build": "turbo run build"
# "test": "turbo run test"
# "lint": "turbo run lint"
# "dev": "turbo run dev"

# Chạy chỉ packages bị affected bởi thay đổi
# turbo run build --filter=...[HEAD^1]

# Chạy specific package
# turbo run build --filter=@acme/web

# Dry run — xem task graph mà không execute
# turbo run build --dry=json
```

```ts
// ==============================
// Remote Caching — chia sẻ cache giữa team và CI
// ==============================

// Đăng nhập Vercel Remote Cache
// npx turbo login
// npx turbo link

// GitHub Actions với Remote Cache
// env:
//   TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
//   TURBO_TEAM: ${{ vars.TURBO_TEAM }}

// Khi CI chạy build:
// 1. Turbo hash inputs (source files, env vars, dependencies)
// 2. Kiểm tra remote cache
// 3. Cache hit → download outputs, không execute
// 4. Cache miss → execute, upload outputs

/*
  Tại sao Turborepo nhanh:
  1. Incremental builds: chỉ build packages thay đổi
  2. Content-based caching: hash của inputs, không phải timestamps
  3. Parallel execution: tasks không dependent chạy song song
  4. Remote cache sharing: team members dùng chung cache
  5. Tự động topological sort: đúng thứ tự build
*/

// Monorepo structure với Turborepo
// apps/
//   web/          (Next.js)
//   docs/         (Docusaurus)
//   mobile/       (React Native)
// packages/
//   ui/           (shared component library)
//   config/       (shared ESLint, TypeScript configs)
//   utils/        (shared utilities)

// packages/ui/package.json
{
  "name": "@acme/ui",
  "version": "0.0.1",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "dev": "tsup src/index.ts --format esm,cjs --dts --watch",
    "lint": "eslint src"
  }
}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "What problem does Turborepo solve, and why can't you just use `npm run build` across packages?"**

**Model Answer:**
"The core problem is that in a monorepo you have packages that depend on each other. If package B depends on package A, you need to build A before B. And if A has already been built and hasn't changed, you don't want to rebuild it at all.

Plain npm workspaces have no concept of this. You can run a script across all packages but there's no dependency ordering and no caching. Every run rebuilds everything.

Turborepo solves both problems. It understands the dependency graph between packages, so it automatically runs builds in the right order. And it caches task outputs based on a hash of the inputs — source files, environment, dependency versions. If none of those have changed, it skips the task entirely and restores the output from cache. On a CI server that can turn a 10-minute build into a 30-second one."

**Trả lời (Tiếng Việt):**
"Vấn đề cốt lõi là trong một monorepo bạn có các packages phụ thuộc vào nhau. Nếu package B phụ thuộc package A, bạn cần build A trước B. Và nếu A đã được build và không thay đổi, bạn không muốn rebuild nó.

npm workspaces thuần túy không có khái niệm này. Bạn có thể chạy script qua tất cả packages nhưng không có dependency ordering và không có caching. Mỗi lần chạy rebuild mọi thứ.

Turborepo giải quyết cả hai vấn đề. Nó hiểu dependency graph giữa các packages, nên nó tự động chạy builds theo đúng thứ tự. Và nó cache task outputs dựa trên hash của inputs — source files, environment, dependency versions. Nếu không có gì thay đổi, nó skip task hoàn toàn và restore output từ cache. Trên CI server điều đó có thể biến một build 10 phút thành 30 giây."

---

**Q2 (Senior): "How does Turborepo's remote caching work, and what are the security considerations?"**

**Model Answer:**
"Remote caching extends local caching to the team and CI. When a developer or CI job runs a build, the outputs get uploaded to a remote cache — in Turborepo's case, this is Vercel's cloud by default, but you can self-host it. The next time anyone runs that task with the same inputs, they get the cached outputs back instead of rerunning.

The inputs are hashed — source files, env variables listed as relevant, lock file, and so on. The hash identifies the cache entry. If the hash matches, you get a cache hit.

On security: you need to be thoughtful about what inputs you include. If a build embeds secrets like API keys, those should be in the inputs list so that different secrets produce different cache entries. Turborepo lets you specify `env` arrays in the pipeline to include environment variables in the hash. Without that, you could serve cached output that has the wrong secrets baked in."

**Trả lời (Tiếng Việt):**
"Remote caching mở rộng local caching ra cho cả team và CI. Khi một developer hay CI job chạy build, outputs được upload lên một remote cache — trong trường hợp Turborepo đây là cloud của Vercel theo mặc định, nhưng bạn có thể self-host. Lần tiếp theo ai đó chạy task đó với cùng inputs, họ nhận outputs đã cache thay vì rerun.

Inputs được hash — source files, env variables được liệt kê là relevant, lock file, và vân vân. Hash xác định cache entry. Nếu hash match, bạn có cache hit.

Về security: bạn cần thận trọng về inputs nào bạn include. Nếu một build nhúng secrets như API keys, những cái đó nên ở trong inputs list để các secrets khác nhau tạo ra cache entries khác nhau. Turborepo cho bạn chỉ định mảng `env` trong pipeline để include environment variables vào hash. Nếu không làm vậy, bạn có thể serve cached output có secrets sai baked in."

---

**Q3 (Senior): "What's the difference between `dependsOn: ['^build']` and `dependsOn: ['build']` in turbo.json?"**

**Model Answer:**
"The caret prefix means topological — 'run this task in all my dependencies first.' So `dependsOn: ['^build']` means: before I build, build everything that I depend on in the workspace.

`dependsOn: ['build']` without the caret means: run the build task in my own package before this task. So `test: { dependsOn: ['build'] }` means run build first, then test — in the same package.

A common configuration is: the `build` task has `dependsOn: ['^build']` — build my deps first, then build me. The `test` task has `dependsOn: ['build']` — build this package first, then test it. And `lint` has no dependsOn at all — it can run at any time in parallel with other tasks."

**Trả lời (Tiếng Việt):**
"Caret prefix nghĩa là topological — 'chạy task này trong tất cả dependencies của tôi trước.' Vậy `dependsOn: ['^build']` nghĩa là: trước khi tôi build, hãy build mọi thứ mà tôi phụ thuộc trong workspace.

`dependsOn: ['build']` không có caret nghĩa là: chạy build task trong chính package của tôi trước task này. Vậy `test: { dependsOn: ['build'] }` nghĩa là chạy build trước, rồi test — trong cùng package.

Cấu hình phổ biến là: task `build` có `dependsOn: ['^build']` — build deps của tôi trước, rồi build tôi. Task `test` có `dependsOn: ['build']` — build package này trước, rồi test nó. Và `lint` không có dependsOn — nó có thể chạy bất kỳ lúc nào song song với các tasks khác."

---

---

## 7. Nx

**Level:** [Senior]

**EN:** What is Nx? Explain the project graph, affected commands, generators, and executors. How does Nx compare to Turborepo?

**VI:** Nx là gì? Giải thích project graph, affected commands, generators, executors. Nx so với Turborepo?

**Trả lời:**

```bash
# Nx CLI commands
nx graph                          # visualize project graph trong browser
nx affected:build                 # chỉ build projects bị affected bởi changes
nx affected:test --base=main      # test từ main branch đến hiện tại
nx run @acme/web:build            # run target cho specific project
nx run-many --target=build --all  # build tất cả projects song song
nx generate @nx/react:component Button --project=ui  # generate component
```

```ts
// nx.json
{
  "$schema": "./node_modules/nx/schemas/nx-schema.json",
  "npmScope": "acme",
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "cache": true,
      "inputs": ["production", "^production"],
      "outputs": ["{projectRoot}/dist"]
    },
    "test": {
      "cache": true,
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"],
      "outputs": ["{projectRoot}/coverage"]
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json"
    ],
    "sharedGlobals": ["{workspaceRoot}/babel.config.json"]
  },
  "plugins": [
    "@nx/react",
    "@nx/node"
  ]
}

// project.json (per project)
{
  "name": "@acme/web",
  "projectType": "application",
  "sourceRoot": "apps/web/src",
  "targets": {
    "build": {
      "executor": "@nx/vite:build",  // Nx executor
      "options": {
        "outputPath": "dist/apps/web"
      },
      "configurations": {
        "production": { "mode": "production" },
        "development": { "mode": "development" }
      }
    },
    "test": {
      "executor": "@nx/vite:test",
      "options": {
        "reportsDirectory": "../../coverage/apps/web"
      }
    }
  }
}

// ==============================
// Generator — scaffolding code
// ==============================
// generators.ts
import { Tree, formatFiles, generateFiles, names } from '@nx/devkit';
import * as path from 'path';

interface ComponentGeneratorSchema {
  name: string;
  project: string;
}

export async function componentGenerator(tree: Tree, options: ComponentGeneratorSchema) {
  const { className, fileName } = names(options.name);

  generateFiles(
    tree,
    path.join(__dirname, 'files'), // template files
    `libs/${options.project}/src/lib/${fileName}`,
    {
      name: options.name,
      className,
      fileName,
    }
  );

  await formatFiles(tree);
}

export default componentGenerator;

// ==============================
// Nx vs Turborepo
// ==============================

/*
  Turborepo:
  ✅ Đơn giản hơn, ít opinionated
  ✅ Fast setup, minimal config
  ✅ Tốt cho npm workspace monorepos
  ❌ Ít tích hợp framework
  ❌ Không có generators/schematics

  Nx:
  ✅ Powerful project graph visualization
  ✅ Generators (scaffolding code tự động)
  ✅ Executors (abstraction cho build tools)
  ✅ Plugin ecosystem (@nx/react, @nx/node, @nx/next)
  ✅ Affected commands based on project graph
  ✅ Remote cache (Nx Cloud)
  ❌ Học phức tạp hơn
  ❌ More opinionated structure

  Khi nào dùng Nx: Large enterprise monorepo, nhiều apps/libs,
                   cần generators, cần project graph visualization
  Khi nào dùng Turborepo: Nhỏ hơn, muốn ít config, ưu tiên simplicity
*/
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Senior): "What's the difference between Turborepo and Nx? How do you decide which to use?"**

**Model Answer:**
"Both are monorepo build systems built around the same core idea — task caching and dependency-aware execution. The main difference is scope and opinionation.

Turborepo is minimal and simple. You add a `turbo.json`, define your pipeline, and it works with whatever build tools and project structure you already have. It doesn't generate code or impose a folder structure. That makes it very easy to adopt incrementally.

Nx is a full platform. It has generators to scaffold new apps and libraries, executors to abstract build tools, a plugin system with deep integrations for frameworks like React, Next.js, and Node, and a visual project graph in the browser. It's more powerful but also more opinionated — it wants you to follow its conventions.

My rule of thumb: if you have an existing project and want faster CI with minimal changes, start with Turborepo. If you're starting a large greenfield monorepo and want automated code generation, consistent patterns, and rich tooling, Nx is worth the learning curve."

**Trả lời (Tiếng Việt):**
"Cả hai đều là monorepo build systems được xây dựng xung quanh cùng một ý tưởng cốt lõi — task caching và dependency-aware execution. Sự khác biệt chính là scope và opinionation.

Turborepo tối giản và đơn giản. Bạn thêm một `turbo.json`, định nghĩa pipeline của bạn, và nó hoạt động với bất kể build tools và project structure nào bạn đã có. Nó không generate code hay áp đặt folder structure. Điều đó làm nó rất dễ adopt incremental.

Nx là một platform đầy đủ. Nó có generators để scaffold new apps và libraries, executors để abstract build tools, plugin system với deep integrations cho các frameworks như React, Next.js, và Node, và visual project graph trong browser. Nó mạnh hơn nhưng cũng opinionated hơn — nó muốn bạn theo conventions của nó.

Quy tắc ngón tay cái của tôi: nếu bạn có project hiện tại và muốn CI nhanh hơn với ít thay đổi nhất, bắt đầu với Turborepo. Nếu bạn đang bắt đầu một monorepo greenfield lớn và muốn automated code generation, consistent patterns, và rich tooling, Nx xứng đáng với learning curve."

---

**Q2 (Senior): "How do Nx's affected commands work and why are they useful in CI?"**

**Model Answer:**
"Nx builds and maintains a project graph — a map of which packages depend on which. When you run `nx affected:build`, it compares the current branch against a base commit, figures out which files changed, traces those changes through the dependency graph, and only builds the packages that are affected — either directly changed or depend on something that changed.

In CI, this is valuable because most PRs only touch one or two packages, but a naive CI pipeline builds and tests everything. With affected commands, a PR that changes only the `ui` library triggers builds and tests for `ui` and anything that imports `ui`, but not unrelated packages. That can cut CI time dramatically in large monorepos.

The base commit is configurable — usually you'd compare against `main` for PRs or against the previous commit for main branch builds."

**Trả lời (Tiếng Việt):**
"Nx build và duy trì một project graph — một bản đồ về packages nào phụ thuộc vào packages nào. Khi bạn chạy `nx affected:build`, nó so sánh branch hiện tại với một base commit, tìm ra files nào đã thay đổi, trace những thay đổi đó qua dependency graph, và chỉ build các packages bị affected — hoặc trực tiếp thay đổi hoặc phụ thuộc vào thứ gì đó đã thay đổi.

Trong CI, điều này có giá trị vì hầu hết PRs chỉ chạm vào một hoặc hai packages, nhưng một CI pipeline naïve build và test mọi thứ. Với affected commands, một PR chỉ thay đổi thư viện `ui` kích hoạt builds và tests cho `ui` và bất cứ thứ gì import `ui`, nhưng không phải các packages không liên quan. Điều đó có thể cắt CI time đáng kể trong large monorepos.

Base commit có thể cấu hình được — thường bạn so sánh với `main` cho PRs hoặc với previous commit cho main branch builds."

---

---

## 8. Module Federation

**Level:** [Senior]

**EN:** What is Module Federation? Explain `exposes`, `remotes`, `shared` config, micro-frontend architecture, and runtime sharing.

**VI:** Module Federation là gì? Giải thích cấu hình `exposes`, `remotes`, `shared`, kiến trúc micro-frontend, và runtime sharing.

**Trả lời:**

Module Federation cho phép nhiều Webpack builds tách biệt **chia sẻ code ở runtime**, không phải compile time.

```ts
// ==============================
// HOST APPLICATION (shell)
// webpack.config.ts
// ==============================
import { ModuleFederationPlugin } from 'webpack/container';

// Host app — load remote modules
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',  // tên của app này

      // Consume từ remote apps
      remotes: {
        // 'remoteName': 'exposedName@url/remoteEntry.js'
        checkout: 'checkout@https://checkout.example.com/remoteEntry.js',
        catalog:  'catalog@https://catalog.example.com/remoteEntry.js',
        auth:     'auth@http://localhost:3001/remoteEntry.js',
      },

      // Shared dependencies — tránh duplicate
      shared: {
        react: {
          singleton: true,      // chỉ 1 instance
          requiredVersion: '^18.0.0',
          eager: false,          // lazy load (không bundle vào remoteEntry)
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: '^6.0.0',
        },
      },
    }),
  ],
};

// ==============================
// REMOTE APPLICATION (checkout MFE)
// webpack.config.ts
// ==============================
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'checkout',          // phải match với host's remote name

      // Expose modules cho host
      exposes: {
        './CheckoutPage': './src/pages/CheckoutPage',
        './CartSummary':  './src/components/CartSummary',
        './useCart':      './src/hooks/useCart',
      },

      // Filename của remote entry (mặc định remoteEntry.js)
      filename: 'remoteEntry.js',

      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};

// ==============================
// Sử dụng trong Host
// ==============================

// TypeScript declarations
declare module 'checkout/CheckoutPage' {
  import { ComponentType } from 'react';
  const CheckoutPage: ComponentType<{ orderId?: string }>;
  export default CheckoutPage;
}

declare module 'catalog/ProductList' {
  import { ComponentType } from 'react';
  const ProductList: ComponentType<{ categoryId: string }>;
  export default ProductList;
}

// Lazy load remote components
import { lazy, Suspense } from 'react';

// Remote components phải được lazy loaded
const RemoteCheckout = lazy(() => import('checkout/CheckoutPage'));
const RemoteCatalog  = lazy(() => import('catalog/ProductList'));

function App() {
  return (
    <Router>
      <Routes>
        <Route
          path="/checkout"
          element={
            <Suspense fallback={<PageSkeleton />}>
              <RemoteCheckout />
            </Suspense>
          }
        />
        <Route
          path="/catalog/:categoryId"
          element={
            <Suspense fallback={<PageSkeleton />}>
              <RemoteCatalog categoryId={params.categoryId} />
            </Suspense>
          }
        />
      </Routes>
    </Router>
  );
}

// ==============================
// Vite Module Federation (vite-plugin-federation)
// ==============================
import federation from '@originjs/vite-plugin-federation';

// vite.config.ts của host
export default defineConfig({
  plugins: [
    federation({
      name: 'host',
      remotes: {
        checkout: 'https://checkout.example.com/assets/remoteEntry.js',
      },
      shared: ['react', 'react-dom'],
    }),
  ],
});

// vite.config.ts của remote
export default defineConfig({
  plugins: [
    federation({
      name: 'checkout',
      filename: 'remoteEntry.js',
      exposes: {
        './CheckoutPage': './src/pages/CheckoutPage',
      },
      shared: ['react', 'react-dom'],
    }),
  ],
  build: { target: 'esnext', minify: false },
});
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Senior): "Can you explain what Module Federation is and the problem it solves?"**

**Model Answer:**
"Module Federation is a Webpack feature that lets multiple, separately deployed applications share code at runtime — not at build time. Before it existed, if you wanted to build micro-frontends you had to either bundle each team's code together in one build, or use iframes which are isolated but awkward.

With Module Federation, a host application can import a React component from a completely separate build that's running on a different server. The import happens over the network when the user navigates to that part of the app. Each team deploys their micro-frontend independently, and the host loads whatever version is currently deployed.

The practical benefit is organizational: large teams can own their slice of the application end-to-end, deploy independently, and not require a coordinated release. The cost is operational complexity — you now have distributed systems concerns in your frontend."

**Trả lời (Tiếng Việt):**
"Module Federation là một tính năng Webpack cho phép nhiều ứng dụng được deploy riêng biệt chia sẻ code ở runtime — không phải tại build time. Trước khi nó tồn tại, nếu bạn muốn build micro-frontends bạn phải hoặc bundle code của mỗi team lại trong một build, hoặc dùng iframes vốn isolated nhưng awkward.

Với Module Federation, một host application có thể import một React component từ một build hoàn toàn riêng biệt đang chạy trên một server khác. Import xảy ra qua network khi user navigate đến phần đó của app. Mỗi team deploy micro-frontend của họ độc lập, và host load bất kể phiên bản nào đang được deploy.

Lợi ích thực tế là về tổ chức: các large teams có thể sở hữu phần của họ trong ứng dụng end-to-end, deploy độc lập, và không cần một coordinated release. Chi phí là operational complexity — bạn giờ có distributed systems concerns trong frontend."

---

**Q2 (Senior): "What does the `shared` configuration do in Module Federation and why is it critical?"**

**Model Answer:**
"The `shared` config tells Module Federation which dependencies can be shared between the host and remotes rather than each loading their own copy.

React is the classic example. If you have a host app and three remote apps, and each loads its own copy of React, you'd have four instances of React running in the same browser tab. React requires a singleton — it doesn't work correctly with multiple instances because of how it manages fiber trees and hooks state.

By marking React as `shared: { singleton: true }`, Module Federation ensures only one copy is loaded. The host loads React, and the remotes use that same instance rather than fetching their own.

The `requiredVersion` field lets you specify compatibility constraints. If the host uses React 18 and a remote was built expecting React 17, you want that mismatch to fail loudly rather than silently loading the wrong version."

**Trả lời (Tiếng Việt):**
"`shared` config nói với Module Federation những dependencies nào có thể được share giữa host và remotes thay vì mỗi cái load bản copy riêng.

React là ví dụ điển hình. Nếu bạn có một host app và ba remote apps, và mỗi cái load React riêng, bạn sẽ có bốn instances của React chạy trong cùng browser tab. React yêu cầu singleton — nó không hoạt động đúng với multiple instances vì cách nó quản lý fiber trees và hooks state.

Bằng cách mark React là `shared: { singleton: true }`, Module Federation đảm bảo chỉ một bản copy được load. Host load React, và các remotes dùng cùng instance đó thay vì fetch của riêng chúng.

Field `requiredVersion` cho phép bạn chỉ định compatibility constraints. Nếu host dùng React 18 và một remote được build với kỳ vọng React 17, bạn muốn mismatch đó fail rõ ràng thay vì silently load phiên bản sai."

---

**Q3 (Senior): "What are the main challenges of a Module Federation micro-frontend architecture in production?"**

**Model Answer:**
"A few come up consistently.

First is versioning and compatibility. If the host expects a remote to have a certain API and the remote deploys a breaking change, things can break in production without any CI signal because the integration only exists at runtime. You need a solid contract testing strategy.

Second is shared state. If the host app has global state like auth context or a theme, remotes need access to it. You have to be deliberate about how that's shared — usually through the `shared` config for things like React context providers, or through custom events or a pub/sub mechanism.

Third is performance. Each remote adds a network roundtrip when it's first loaded. You need to think carefully about loading strategies, fallback UI during remote loading, and error boundaries for when a remote is unavailable.

And fourth is the operational overhead. You now have multiple deployment pipelines, multiple staging environments to coordinate, and debugging gets harder because the error might be in the host, the remote, or the interaction between them."

**Trả lời (Tiếng Việt):**
"Có một số vấn đề xuất hiện liên tục.

Đầu tiên là versioning và compatibility. Nếu host kỳ vọng một remote có một API nhất định và remote deploy một breaking change, mọi thứ có thể bị vỡ trong production mà không có CI signal nào vì integration chỉ tồn tại ở runtime. Bạn cần một chiến lược contract testing vững chắc.

Thứ hai là shared state. Nếu host app có global state như auth context hay theme, remotes cần access vào nó. Bạn phải cân nhắc kỹ về cách đó được chia sẻ — thường qua `shared` config cho những thứ như React context providers, hoặc qua custom events hay pub/sub mechanism.

Thứ ba là performance. Mỗi remote thêm một network roundtrip khi lần đầu được load. Bạn cần suy nghĩ cẩn thận về loading strategies, fallback UI trong khi remote đang load, và error boundaries cho khi remote không khả dụng.

Và thứ tư là operational overhead. Bây giờ bạn có multiple deployment pipelines, multiple staging environments cần coordinate, và debugging khó hơn vì lỗi có thể ở host, remote, hoặc tương tác giữa chúng."

---

**Q4 (Senior): "How does Module Federation differ from just lazy loading a separately deployed JavaScript file with `import()`?"**

**Model Answer:**
"A dynamic `import()` of a remote URL is actually how Module Federation works under the hood — the difference is what you get on top of that.

Module Federation gives you: shared dependency management so React isn't duplicated, a manifest system through `remoteEntry.js` so the host knows what modules are available, and build-time type safety if you set up TypeScript declarations.

With a raw dynamic import, you'd get the raw bundle with no deduplication — both apps would bundle their own React. You'd also have to manage the URL and module exports manually, with no framework for versioning or fallbacks.

Module Federation is essentially a runtime module system built on top of dynamic imports, with conventions and tooling for dependency sharing, discovery, and composition."

**Trả lời (Tiếng Việt):**
"Dynamic `import()` của một remote URL thực ra là cách Module Federation hoạt động ở bên dưới — điểm khác biệt là những gì bạn có thêm.

Module Federation cung cấp: shared dependency management để React không bị duplicate, một manifest system thông qua `remoteEntry.js` để host biết modules nào có sẵn, và build-time type safety nếu bạn setup TypeScript declarations.

Với một raw dynamic import, bạn sẽ có raw bundle không có deduplication — cả hai apps đều bundle React riêng của chúng. Bạn cũng phải quản lý URL và module exports thủ công, không có framework cho versioning hay fallbacks.

Module Federation về bản chất là một runtime module system được xây dựng trên dynamic imports, với conventions và tooling cho dependency sharing, discovery, và composition."

---

---

## 9. Storybook

**Level:** [Mid] / [Senior]

**EN:** How does Storybook support component-driven development? Explain stories, args, decorators, addons, and Chromatic integration.

**VI:** Storybook hỗ trợ component-driven development như thế nào? Giải thích stories, args, decorators, addons, và tích hợp Chromatic.

**Trả lời:**

```tsx
// ==============================
// Basic Story
// ==============================

// components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { within, userEvent, expect } from '@storybook/test';
import { Button } from './Button';

// Meta — config cho tất cả stories trong file
const meta: Meta<typeof Button> = {
  title: 'Components/Button',   // organization trong sidebar
  component: Button,
  tags: ['autodocs'],            // auto-generate Docs tab

  // Global args cho tất cả stories
  args: {
    size: 'md',
    disabled: false,
  },

  // ArgTypes — controls UI trong Storybook
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost', 'danger'],
      description: 'Visual style variant',
      table: {
        defaultValue: { summary: 'primary' },
      },
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
    },
    onClick: { action: 'clicked' }, // log actions in Actions tab
  },

  // Decorators — wrap stories với context (providers)
  decorators: [
    (Story) => (
      <div style={{ padding: '2rem', background: '#f5f5f5' }}>
        <Story />
      </div>
    ),
  ],
};

export default meta;
type Story = StoryObj<typeof Button>;

// ==============================
// Stories
// ==============================
export const Primary: Story = {
  args: { variant: 'primary', children: 'Primary Button' },
};

export const Secondary: Story = {
  args: { variant: 'secondary', children: 'Secondary Button' },
};

export const Loading: Story = {
  args: {
    variant: 'primary',
    children: 'Loading...',
    disabled: true,
    loading: true,
  },
};

// Story với interaction test (Storybook 7+)
export const ClickInteraction: Story = {
  args: { variant: 'primary', children: 'Click Me' },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button', { name: 'Click Me' });

    await userEvent.click(button);
    await expect(button).toBeVisible();
  },
};

// ==============================
// Global Decorators (providers)
// ==============================
// .storybook/preview.tsx
import type { Preview } from '@storybook/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { MemoryRouter } from 'react-router-dom';
import '../src/styles/globals.css';

const queryClient = new QueryClient({
  defaultOptions: { queries: { retry: false, staleTime: Infinity } },
});

const preview: Preview = {
  decorators: [
    (Story) => (
      <QueryClientProvider client={queryClient}>
        <MemoryRouter>
          <Story />
        </MemoryRouter>
      </QueryClientProvider>
    ),
  ],
  parameters: {
    // Viewport addon — responsive testing
    viewport: {
      viewports: {
        mobile: { name: 'Mobile', styles: { width: '375px', height: '812px' } },
        tablet: { name: 'Tablet', styles: { width: '768px', height: '1024px' } },
        desktop: { name: 'Desktop', styles: { width: '1440px', height: '900px' } },
      },
      defaultViewport: 'desktop',
    },
    // Actions addon — log events
    actions: { argTypesRegex: '^on[A-Z].*' },
    // Background addon
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#0f172a' },
      ],
    },
  },
};

export default preview;

// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',  // docs, controls, actions, viewport, backgrounds
    '@storybook/addon-a11y',        // accessibility checks
    '@storybook/addon-interactions', // play functions
    '@chromatic-com/storybook',     // Chromatic integration
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: { autodocs: 'tag' },
};

export default config;
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "What is Storybook and how does it help with component development?"**

**Model Answer:**
"Storybook is a development environment for building UI components in isolation. You write stories — each story is a function that renders your component with a specific set of props. Storybook runs a separate dev server that shows you all your stories in a sidebar, so you can develop and review components without running your full application.

The main workflow benefit is isolation. If you're building a button component, you don't need to navigate to the specific page in your app where that button appears. You just open the button's story. That speeds up iteration a lot.

It also doubles as documentation. The stories catalog your component API with live examples, and the auto-docs feature can generate documentation pages from your TypeScript types. For design systems especially, it becomes the source of truth for what components exist and how they behave."

**Trả lời (Tiếng Việt):**
"Storybook là một development environment để build UI components trong isolation. Bạn viết stories — mỗi story là một function render component của bạn với một tập props cụ thể. Storybook chạy một dev server riêng biệt hiển thị tất cả stories của bạn trong một sidebar, nên bạn có thể develop và review components mà không cần chạy full application.

Lợi ích workflow chính là isolation. Nếu bạn đang build một button component, bạn không cần navigate đến trang cụ thể trong app nơi button đó xuất hiện. Bạn chỉ mở story của button. Điều đó tăng tốc iteration rất nhiều.

Nó cũng double as documentation. Các stories catalog component API của bạn với live examples, và tính năng auto-docs có thể generate documentation pages từ TypeScript types của bạn. Đặc biệt với design systems, nó trở thành source of truth cho những components nào tồn tại và chúng behave như thế nào."

---

**Q2 (Mid): "What's the difference between a decorator and an arg in Storybook?"**

**Model Answer:**
"Args are the inputs — they map directly to your component's props. When you define args in a story, they show up in the Controls panel in Storybook's UI, and you can change them interactively to explore how the component responds. They're also what gets passed to the component when the story renders.

Decorators are wrappers around stories. They're for adding context that a component needs to render correctly but isn't part of its props — things like a theme provider, a router context, a query client, or even just a padding wrapper to give the story visual breathing room.

You can define decorators at the story level, the file level in the meta object, or globally in `.storybook/preview.tsx`. The global level is where I put things like providers that every component needs — React Query client, router, design token theme."

**Trả lời (Tiếng Việt):**
"Args là inputs — chúng map trực tiếp đến props của component. Khi bạn định nghĩa args trong một story, chúng xuất hiện trong Controls panel trong Storybook UI, và bạn có thể thay đổi chúng tương tác để khám phá cách component phản hồi. Chúng cũng là thứ được truyền vào component khi story render.

Decorators là wrappers xung quanh stories. Chúng dùng để thêm context mà component cần để render đúng nhưng không phải là props — những thứ như theme provider, router context, query client, hoặc thậm chí chỉ một padding wrapper để story có visual breathing room.

Bạn có thể định nghĩa decorators ở story level, file level trong meta object, hoặc globally trong `.storybook/preview.tsx`. Global level là nơi tôi đặt những thứ như providers mà mọi component cần — React Query client, router, design token theme."

---

**Q3 (Senior): "How would you integrate Storybook into a CI pipeline for visual regression testing?"**

**Model Answer:**
"The standard approach is Chromatic. It captures screenshots of every story on each commit and compares them to the baseline. If anything changed visually — even by a pixel — it flags it for review in a UI where you can accept or reject the change.

The workflow is: push to a branch, Chromatic runs in CI and uploads your Storybook, screenshots every story, and reports back. If there are visual diffs, the CI check fails and you have to explicitly approve the changes in the Chromatic dashboard. Once you approve, that becomes the new baseline.

The practical benefit is catching unintentional visual regressions — a CSS change that breaks a component you weren't thinking about. It's not perfect; there are false positives from rendering differences or animation timing. But for a design system, it's a very valuable safety net.

Alternatives to Chromatic include Percy or self-hosted solutions using Playwright's screenshot comparison."

**Trả lời (Tiếng Việt):**
"Cách tiếp cận chuẩn là Chromatic. Nó capture screenshots của mọi story trên mỗi commit và so sánh chúng với baseline. Nếu có gì thay đổi về visual — dù chỉ một pixel — nó flag để review trong một UI nơi bạn có thể chấp nhận hoặc từ chối thay đổi.

Workflow là: push lên branch, Chromatic chạy trong CI và upload Storybook của bạn, screenshot mọi story, và báo cáo lại. Nếu có visual diffs, CI check fail và bạn phải explicitly approve các thay đổi trong Chromatic dashboard. Sau khi bạn approve, đó trở thành baseline mới.

Lợi ích thực tế là phát hiện visual regressions không có chủ ý — một CSS change phá vỡ một component bạn không nghĩ đến. Nó không hoàn hảo; có false positives từ rendering differences hoặc animation timing. Nhưng với một design system, đó là một safety net rất có giá trị.

Các alternatives cho Chromatic bao gồm Percy hoặc self-hosted solutions dùng Playwright's screenshot comparison."

---

**Q4 (Senior): "How do you handle stories for components that make API calls or depend on complex application state?"**

**Model Answer:**
"The key is mocking. I don't want stories to make real API calls — they'd be flaky, slow, and wouldn't work offline.

For components that use React Query or SWC, I configure the provider in a decorator with specific initial data. For components that directly call APIs, I use MSW — Mock Service Worker. Storybook has an MSW addon that lets you define request handlers per story. The story renders, the component makes its fetch call, and MSW intercepts it and returns the mock data you defined.

The advantage of MSW is that the mock is at the network level, so the component behaves exactly as it would with a real API — including loading states, error states, pagination. You can write separate stories for the loading state, the error state, and the success state by returning different MSW responses.

For Redux or Zustand state, I either provide a pre-configured store in a decorator or, for simpler cases, test the presentational component directly with mock data passed as props and write the integration tests separately."

**Trả lời (Tiếng Việt):**
"Chìa khóa là mocking. Tôi không muốn stories thực hiện real API calls — chúng sẽ flaky, chậm, và không hoạt động offline.

Với các components dùng React Query hoặc SWC, tôi cấu hình provider trong decorator với specific initial data. Với các components trực tiếp gọi APIs, tôi dùng MSW — Mock Service Worker. Storybook có MSW addon cho phép bạn định nghĩa request handlers cho từng story. Story render, component thực hiện fetch call, và MSW intercept nó và trả về mock data bạn đã định nghĩa.

Ưu điểm của MSW là mock ở network level, nên component behave chính xác như với real API — bao gồm loading states, error states, pagination. Bạn có thể viết separate stories cho loading state, error state, và success state bằng cách return các MSW responses khác nhau.

Với Redux hoặc Zustand state, tôi hoặc cung cấp pre-configured store trong decorator hoặc, trong trường hợp đơn giản hơn, test presentational component trực tiếp với mock data được truyền như props và viết integration tests riêng."

---

---

## 10. Docker for Frontend

**Level:** [Mid] / [Senior]

**EN:** How do you Dockerize a frontend application? Explain multi-stage builds, `.dockerignore`, and runtime environment variables.

**VI:** Cách Docker hóa frontend app? Giải thích multi-stage build, `.dockerignore`, và runtime environment variables.

**Trả lời:**

```dockerfile
# Dockerfile — Multi-stage build
# Stage 1: Build
FROM node:20-alpine AS builder

# Thiết lập working directory
WORKDIR /app

# Copy package files trước (leverage Docker layer caching)
# nếu chỉ code thay đổi, không cần reinstall dependencies
COPY package.json package-lock.json ./

# npm ci: install từ lock file chính xác, không thay đổi lock file
RUN npm ci --frozen-lockfile

# Copy source code
COPY . .

# Build args cho build-time env vars (ví dụ: API URL)
ARG VITE_API_URL=https://api.example.com
ENV VITE_API_URL=$VITE_API_URL

# Build production bundle
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine AS production

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built files từ builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy entrypoint script để inject runtime env vars
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /usr/share/nginx/html
USER appuser

EXPOSE 80

ENTRYPOINT ["/entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # SPA routing — tất cả routes → index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }

    # Gzip
    gzip on;
    gzip_types text/html text/css application/javascript application/json;
    gzip_min_length 1000;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
}
```

```bash
#!/bin/sh
# entrypoint.sh — inject runtime environment variables

# Vite env vars được hardcode tại build time
# Để có runtime variables, inject vào window object qua JSON

# Tạo runtime-config.js từ environment variables
cat > /usr/share/nginx/html/runtime-config.js << EOF
window.__RUNTIME_CONFIG__ = {
  API_URL: "${API_URL:-https://api.example.com}",
  FEATURE_FLAGS: "${FEATURE_FLAGS:-{}}",
  APP_VERSION: "${APP_VERSION:-unknown}"
};
EOF

# Execute CMD (nginx)
exec "$@"
```

```html
<!-- index.html — load runtime config trước app bundle -->
<head>
  <script src="/runtime-config.js"></script>
</head>
```

```ts
// src/config.ts — sử dụng runtime config
declare global {
  interface Window {
    __RUNTIME_CONFIG__?: {
      API_URL: string;
      FEATURE_FLAGS: string;
      APP_VERSION: string;
    };
  }
}

export const config = {
  apiUrl: window.__RUNTIME_CONFIG__?.API_URL ?? import.meta.env.VITE_API_URL ?? '',
  appVersion: window.__RUNTIME_CONFIG__?.APP_VERSION ?? 'dev',
};
```

```
# .dockerignore — tránh copy không cần thiết
node_modules/
dist/
.git/
.env
.env.*
*.md
.DS_Store
coverage/
.storybook/
stories/
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        VITE_API_URL: https://api.example.com
    ports:
      - "3000:80"
    environment:
      - API_URL=https://api.example.com
      - FEATURE_FLAGS={"darkMode":true}
    restart: unless-stopped

  # Development
  frontend-dev:
    image: node:20-alpine
    working_dir: /app
    volumes:
      - .:/app
      - /app/node_modules  # anonymous volume — tránh overwrite node_modules
    ports:
      - "3000:3000"
    command: npm run dev -- --host
    environment:
      - VITE_API_URL=http://localhost:8080
```

---

## 11. GitHub Actions

**Level:** [Mid] / [Senior]

**EN:** Explain GitHub Actions workflow syntax: jobs, steps, matrix builds, caching `node_modules`, and deploy on push.

**VI:** Giải thích GitHub Actions: workflow syntax, jobs/steps, matrix builds, cache node_modules, và deploy on push.

**Trả lời:**

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
  TURBO_TEAM: ${{ vars.TURBO_TEAM }}

# ==============================
# JOBS
# ==============================
jobs:
  # Job 1: Quality checks
  quality:
    name: Quality Checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # full history cho affected commands

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'  # cache node_modules tự động

      # Cache node_modules
      - name: Cache node_modules
        uses: actions/cache@v4
        id: cache-node-modules
        with:
          path: |
            node_modules
            ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        if: steps.cache-node-modules.outputs.cache-hit != 'true'
        run: npm ci --frozen-lockfile

      - name: Lint
        run: npm run lint:ci

      - name: Type check
        run: npm run type-check

  # Job 2: Tests
  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: quality  # chạy sau quality
    strategy:
      # Matrix builds — chạy song song
      matrix:
        node-version: [18, 20, 22]  # test trên nhiều Node versions
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - run: npm ci

      - name: Run tests
        run: npm run test:coverage
        env:
          CI: true

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info

  # Job 3: Build
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [quality, test]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci

      - name: Build
        run: npm run build
        env:
          VITE_API_URL: ${{ vars.VITE_API_URL }}

      # Cache build artifacts cho deploy job
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
          retention-days: 1

  # Job 4: E2E Tests
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      - run: npm ci

      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/

      - name: Run Playwright tests
        uses: microsoft/playwright-github-action@v1
        with:
          command: npx playwright test

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/

  # Job 5: Deploy (chỉ khi push to main)
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [build, e2e]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production  # requires manual approval
    steps:
      - uses: actions/checkout@v4

      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'

      - name: Post deployment notification
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "Deployment successful for ${{ github.sha }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### 🎤 Mock Interview — Q&A

---

**Q1 (Mid): "Can you explain what a GitHub Actions workflow is and how jobs and steps relate to each other?"**

**Model Answer:**
"A workflow is a YAML file in `.github/workflows/` that defines automated processes triggered by events — like a push to main or a pull request.

A workflow contains one or more jobs. Each job runs on a fresh virtual machine — a runner. By default, jobs run in parallel. If you want sequential execution, you use `needs` to declare dependencies between jobs.

Within a job, you have steps. Steps run sequentially on the same runner, sharing the filesystem. Each step is either a shell command with `run` or a reusable action from the marketplace with `uses`. Because steps share the runner, you can build something in one step and use that output in the next step.

The common pattern I use is: a quality job for linting and type checking, a test job that depends on quality, a build job that depends on tests, and a deploy job that only runs on main and depends on build passing."

**Trả lời (Tiếng Việt):**
"Workflow là một YAML file trong `.github/workflows/` định nghĩa automated processes được triggered bởi events — như push lên main hoặc pull request.

Một workflow chứa một hoặc nhiều jobs. Mỗi job chạy trên một fresh virtual machine — một runner. Theo mặc định, jobs chạy song song. Nếu bạn muốn sequential execution, bạn dùng `needs` để khai báo dependencies giữa các jobs.

Trong một job, bạn có steps. Steps chạy tuần tự trên cùng runner, chia sẻ filesystem. Mỗi step là hoặc một shell command với `run` hoặc một reusable action từ marketplace với `uses`. Vì steps chia sẻ runner, bạn có thể build gì đó trong một step và dùng output đó ở step tiếp theo.

Pattern phổ biến tôi dùng là: một quality job cho linting và type checking, một test job phụ thuộc vào quality, một build job phụ thuộc vào tests, và một deploy job chỉ chạy trên main và phụ thuộc vào build pass."

---

**Q2 (Mid): "How do you cache `node_modules` in GitHub Actions to speed up CI?"**

**Model Answer:**
"There are two approaches. The simpler one is using the built-in cache support in `actions/setup-node` — you just set `cache: 'npm'` and it handles it for you. It caches based on the lock file hash.

For more control, I use `actions/cache` directly. You specify a path to cache, a cache key, and restore keys. The key is usually something like `${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}` — so it's OS-specific and invalidates whenever the lock file changes. The restore keys are fallbacks for partial cache hits.

The pattern is to check if the cache was a hit, and only run `npm ci` if it wasn't:

```yaml
- uses: actions/cache@v4
  id: cache-node
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

- run: npm ci
  if: steps.cache-node.outputs.cache-hit != 'true'
```

This can save two to three minutes per job on a project with many dependencies."

**Trả lời (Tiếng Việt):**
"Có hai cách tiếp cận. Cách đơn giản hơn là dùng built-in cache support trong `actions/setup-node` — bạn chỉ cần set `cache: 'npm'` và nó tự xử lý. Nó cache dựa trên lock file hash.

Để kiểm soát nhiều hơn, tôi dùng `actions/cache` trực tiếp. Bạn chỉ định một path để cache, một cache key, và restore keys. Key thường là gì đó như `${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}` — nên nó OS-specific và invalidate khi lock file thay đổi. Restore keys là fallbacks cho partial cache hits.

Pattern là kiểm tra xem cache có phải là hit không, và chỉ chạy `npm ci` nếu không phải:

```yaml
- uses: actions/cache@v4
  id: cache-node
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

- run: npm ci
  if: steps.cache-node.outputs.cache-hit != 'true'
```

Điều này có thể tiết kiệm hai đến ba phút mỗi job trên một project với nhiều dependencies."

---

**Q3 (Senior): "How would you set up a CI pipeline that only runs tests for the packages changed in a PR?"**

**Model Answer:**
"This is where Turborepo or Nx become really valuable. With Turborepo, I'd run `turbo run test --filter=...[HEAD^1]` which uses the filter syntax to only include packages changed since the last commit and their dependents.

With Nx, `nx affected:test --base=origin/main` computes affected packages by comparing the branch to main.

Without those tools, you'd have to roll your own change detection. You could use `git diff --name-only origin/main HEAD` to get changed files, then map those to packages by checking which `package.json` directory they're under, then run tests only in those directories. It works but it's brittle — you also need to handle transitive dependencies manually.

In GitHub Actions I'd capture the affected packages as a job output in a first step, then use matrix strategy in the test job to run tests for each affected package in parallel."

**Trả lời (Tiếng Việt):**
"Đây là nơi Turborepo hoặc Nx trở nên thực sự có giá trị. Với Turborepo, tôi sẽ chạy `turbo run test --filter=...[HEAD^1]` dùng filter syntax để chỉ include các packages thay đổi kể từ commit cuối và các dependents của chúng.

Với Nx, `nx affected:test --base=origin/main` tính toán affected packages bằng cách so sánh branch với main.

Nếu không có những tools đó, bạn phải tự roll change detection. Bạn có thể dùng `git diff --name-only origin/main HEAD` để lấy changed files, rồi map những files đó sang packages bằng cách kiểm tra thư mục `package.json` nào chúng thuộc về, rồi chạy tests chỉ trong những thư mục đó. Nó hoạt động nhưng fragile — bạn cũng cần xử lý transitive dependencies thủ công.

Trong GitHub Actions tôi sẽ capture affected packages như một job output trong bước đầu tiên, rồi dùng matrix strategy trong test job để chạy tests cho mỗi affected package song song."

---

**Q4 (Senior): "What's the difference between `github.event_name` and `github.ref` and why do you need both when controlling deploy jobs?"**

**Model Answer:**
"They're checking different things. `github.ref` is the branch or tag the workflow is running on — like `refs/heads/main`. `github.event_name` is what triggered the workflow — `push`, `pull_request`, `workflow_dispatch`, etc.

You need both because a pull request targeting main and a direct push to main both have `github.ref` equal to `refs/heads/main`. But for a PR, the workflow runs on a temporary merge commit, not actually on main.

If you only check `github.ref == 'refs/heads/main'`, your deploy job could trigger on PR builds. Adding `github.event_name == 'push'` ensures it only runs on actual pushes to main, not PRs targeting main.

The canonical production deploy condition is:
```yaml
if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```"

**Trả lời (Tiếng Việt):**
"Chúng đang kiểm tra những thứ khác nhau. `github.ref` là branch hoặc tag workflow đang chạy trên — như `refs/heads/main`. `github.event_name` là thứ triggered workflow — `push`, `pull_request`, `workflow_dispatch`, v.v.

Bạn cần cả hai vì một pull request targeting main và một direct push lên main đều có `github.ref` bằng `refs/heads/main`. Nhưng với một PR, workflow chạy trên một temporary merge commit, không thực sự chạy trên main.

Nếu bạn chỉ kiểm tra `github.ref == 'refs/heads/main'`, deploy job của bạn có thể trigger trên PR builds. Thêm `github.event_name == 'push'` đảm bảo nó chỉ chạy trên actual pushes lên main, không phải PRs targeting main.

Điều kiện deploy production chuẩn là:
```yaml
if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```"

---

---

## 12. Package Managers

**Level:** [Mid]

**EN:** Compare npm, Yarn, and pnpm. Explain workspace support, lockfiles, and why pnpm uses hard links instead of symlinks.

**VI:** So sánh npm, Yarn, pnpm. Giải thích workspace support, lockfiles, và tại sao pnpm dùng hard links.

**Trả lời:**

| | npm | Yarn | pnpm |
|--|-----|------|------|
| Performance | Trung bình | Nhanh (PnP) | Nhanh nhất |
| Disk space | Nhiều (duplicate) | Nhiều (duplicate) | Ít nhất (hard links) |
| Workspaces | npm workspaces | yarn workspaces | pnpm workspaces |
| Lockfile | package-lock.json | yarn.lock | pnpm-lock.yaml |
| Strictness | Loose | Loose | Strict (phantom deps prevention) |
| Monorepo | Ổn | Tốt | Tốt nhất |

```bash
# ==============================
# pnpm — content-addressable store
# ==============================
# Tất cả packages được lưu trong ~/.pnpm-store
# Mỗi version chỉ lưu 1 lần (deduplication)
# node_modules/ dùng hard links đến store → không duplicate

# npm/yarn: project A và project B đều install lodash@4.17.21
# → 2 copies trên disk

# pnpm: cả 2 đều hard link đến 1 copy trong store
# → tiết kiệm disk space đáng kể

# pnpm setup
npm install -g pnpm

pnpm install               # install dependencies
pnpm add react             # add package
pnpm add -D typescript     # add dev dependency
pnpm remove lodash         # remove package
pnpm update react          # update to latest compatible
```

```yaml
# pnpm-workspace.yaml — monorepo config
packages:
  - 'apps/*'
  - 'packages/*'
  - '!**/test/**'
```

```json
// package.json root — workspace commands
{
  "scripts": {
    "build": "pnpm -r run build",              // -r = recursive (tất cả packages)
    "test": "pnpm -r run test",
    "dev:web": "pnpm --filter @acme/web dev",  // --filter = chỉ package đó
    "add:ui": "pnpm --filter @acme/web add @acme/ui",
    "changeset": "changeset",
    "version": "changeset version",
    "publish": "pnpm -r publish"
  }
}
```

```bash
# Yarn workspaces
yarn workspace @acme/web add react  # add to specific package
yarn workspaces foreach run build   # run in all packages

# Yarn Plug'n'Play (PnP) — không có node_modules
# .pnp.cjs thay thế node_modules
# Nhanh hơn, zero installs, nhưng cần editor support

# .yarnrc.yml
# nodeLinker: pnp  # enable PnP
# nodeLinker: node-modules  # traditional (compat mode)
```

```bash
# npm workspaces (v7+)
npm install --workspace=@acme/web  # install trong specific workspace
npm run build --workspaces         # run trong tất cả workspaces
npm run test -w @acme/ui           # shorthand
```

---

## 13. TypeScript Build Tooling

**Level:** [Senior]

**EN:** Compare `tsc`, `ts-jest`, `SWC`, and `esbuild` for TypeScript compilation. What are the speed and feature tradeoffs?

**VI:** So sánh `tsc`, `ts-jest`, `SWC`, và `esbuild` cho TypeScript compilation. Tradeoffs về tốc độ và features?

**Trả lời:**

| Tool | Speed | Type checking | Babel plugins | Use case |
|------|-------|--------------|---------------|----------|
| `tsc` | Chậm | Đầy đủ | Không | Type checking, tạo .d.ts files |
| `ts-jest` | Chậm | Có thể tắt | Không | Jest với TypeScript (legacy) |
| `SWC` | Rất nhanh (Rust) | Không | Có (SWC plugins) | Compiler drop-in replacement |
| `esbuild` | Cực nhanh (Go) | Không | Không | Bundler, dev server |
| `Babel` | Nhanh (JS) | Không | Có | Legacy, plugins ecosystem |

```ts
// ==============================
// 1. tsc — TypeScript compiler chính thức
// ==============================
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,         // generate .d.ts files
    "declarationMap": true,
    "sourceMap": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "skipLibCheck": true,
    "jsx": "react-jsx"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}

// package.json scripts
// "type-check": "tsc --noEmit"           — chỉ type check, không emit
// "build:types": "tsc --emitDeclarationOnly"  — chỉ emit .d.ts
// "build": "tsc"

// ==============================
// 2. ts-jest → @swc/jest (migration)
// ==============================

// Cũ: ts-jest (chậm)
// jest.config.ts
export default {
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
};

// Mới: @swc/jest (5-20x nhanh hơn)
export default {
  transform: {
    '^.+\\.tsx?$': ['@swc/jest', {
      jsc: {
        parser: { syntax: 'typescript', tsx: true, decorators: true },
        transform: { react: { runtime: 'automatic' } },
      },
    }],
  },
};

// ==============================
// 3. SWC — Speedy Web Compiler (Rust)
// ==============================
// Dùng làm drop-in replacement cho Babel
// Next.js dùng SWC mặc định từ v12

// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true
    },
    "transform": {
      "react": {
        "runtime": "automatic"
      }
    },
    "target": "es2020"
  },
  "module": {
    "type": "es6"
  }
}

// Vite dùng SWC với @vitejs/plugin-react-swc
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react-swc'; // SWC thay Babel

export default defineConfig({
  plugins: [react()],
  // Fast refresh với SWC
});

// ==============================
// 4. esbuild — Extremely fast (Go)
// ==============================
import { build, transform } from 'esbuild';

// Bundle
await build({
  entryPoints: ['src/index.tsx'],
  bundle: true,
  outfile: 'dist/bundle.js',
  format: 'esm',
  target: ['es2020', 'chrome80'],
  loader: {
    '.tsx': 'tsx',
    '.ts': 'ts',
    '.png': 'file',
    '.svg': 'text',
  },
  define: {
    'process.env.NODE_ENV': '"production"',
  },
  minify: true,
  sourcemap: true,
  // esbuild không type check — kết hợp với tsc
});

// Transform only (không bundle)
const result = await transform(typescriptCode, {
  loader: 'tsx',
  target: 'es2020',
});

// ==============================
// Recommended setup cho large project
// ==============================

// package.json
{
  "scripts": {
    // Type checking (tsc) — separate từ compilation
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch",

    // Fast compilation (esbuild/SWC) — không type check
    "build": "vite build",   // Vite/SWC/Rollup
    "dev": "vite",

    // Testing (SWC/ESBuild based)
    "test": "vitest",        // Vitest dùng Vite's pipeline (SWC/esbuild)

    // CI: cả type check VÀ build/test
    "ci": "npm run type-check && npm run build && npm test"
  }
}

// Vitest config — sử dụng Vite transform pipeline (rất nhanh)
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react-swc';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    globals: true,          // không cần import describe/it/expect
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov'],
    },
  },
});
```
