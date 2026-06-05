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
