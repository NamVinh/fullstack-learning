# Phase 12 — Deployment & Production

Deployment là quá trình đưa code từ development lên production environment. Production Node.js apps cần: containerization (Docker), process management (PM2), graceful shutdown, CI/CD pipeline, và proper environment configuration.

---

## Environment Configuration

```bash
# .env — KHÔNG commit vào git
PORT=3000
NODE_ENV=production
DATABASE_URL=postgresql://user:pass@db-host:5432/mydb
REDIS_URL=redis://redis-host:6379
JWT_SECRET=your-256-bit-secret-here
JWT_REFRESH_SECRET=another-256-bit-secret

# .env.example — COMMIT cái này
PORT=3000
NODE_ENV=development
DATABASE_URL=
REDIS_URL=redis://localhost:6379
JWT_SECRET=
JWT_REFRESH_SECRET=
```

```js
// Validate env vars khi app start
const { z } = require('zod')

const envSchema = z.object({
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 chars'),
  REDIS_URL: z.string().url().optional()
})

const env = envSchema.safeParse(process.env)
if (!env.success) {
  console.error('Invalid environment variables:')
  console.error(env.error.flatten().fieldErrors)
  process.exit(1)
}

module.exports = env.data
```

---

## PM2 — Process Manager

PM2 giữ Node.js app alive, restart khi crash, và hỗ trợ cluster mode:

```bash
# Install PM2 globally
npm install -g pm2

# Start app
pm2 start src/index.js --name api

# Cluster mode — 1 instance per CPU core
pm2 start src/index.js --name api --instances max --exec-mode cluster

# Xem logs
pm2 logs api
pm2 logs api --lines 100

# Monitor
pm2 monit

# Restart app (hard restart)
pm2 restart api

# Reload (zero-downtime rolling restart)
pm2 reload api

# Stop và delete
pm2 stop api
pm2 delete api

# Save process list và setup startup script
pm2 save
pm2 startup  # chạy lệnh được in ra để setup systemd/launchctl
```

### ecosystem.config.js

```js
module.exports = {
  apps: [{
    name: 'api',
    script: 'src/index.js',
    instances: 'max',          // hoặc số cụ thể như 4
    exec_mode: 'cluster',
    watch: false,               // KHÔNG watch ở production
    max_memory_restart: '1G',  // restart nếu memory > 1GB
    min_uptime: '5s',          // consider unhealthy nếu crash trong 5s đầu
    max_restarts: 10,          // tối đa 10 restarts

    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },

    // Logging
    out_file: '/var/log/api/out.log',
    error_file: '/var/log/api/error.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true
  }]
}

// Deploy
// pm2 start ecosystem.config.js --env production
// pm2 reload ecosystem.config.js --env production  // zero-downtime
```

---

## Docker — Containerization

### Dockerfile (Production)

```dockerfile
# Multi-stage build — image nhỏ hơn, không có dev dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production   # chỉ install production deps

# Build stage (nếu có TypeScript)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build              # compile TypeScript

# Production image
FROM node:20-alpine AS production
WORKDIR /app

# Security: chạy với non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./

USER nodejs

EXPOSE 3000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => r.statusCode === 200 ? process.exit(0) : process.exit(1))"

CMD ["node", "dist/index.js"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      target: production
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:password@db:5432/mydb
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes  # persistent storage
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/ssl:ro
    depends_on:
      - api
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:
```

---

## Graceful Shutdown

```js
// src/index.js
const express = require('express')
const { createServer } = require('http')
const { prisma } = require('./db/prisma')
const { redisClient } = require('./cache/redis')

const app = express()
// ... setup middleware, routes

const server = createServer(app)
server.listen(process.env.PORT || 3000)

// === GRACEFUL SHUTDOWN ===
let isShuttingDown = false

const shutdown = async (signal) => {
  if (isShuttingDown) return
  isShuttingDown = true

  console.log(`\nReceived ${signal} — starting graceful shutdown`)

  // Sau 30s, force exit nếu cleanup không xong
  const forceExit = setTimeout(() => {
    console.error('Forced exit after timeout')
    process.exit(1)
  }, 30000)

  // Stop accepting new connections
  server.close(async () => {
    try {
      // Cleanup resources
      await prisma.$disconnect()
      await redisClient.quit()

      clearTimeout(forceExit)
      console.log('Shutdown complete')
      process.exit(0)
    } catch (err) {
      console.error('Error during shutdown:', err)
      process.exit(1)
    }
  })
}

process.on('SIGTERM', () => shutdown('SIGTERM'))  // Docker stop, Kubernetes
process.on('SIGINT', () => shutdown('SIGINT'))    // Ctrl+C

// Health check endpoint (cho load balancer)
app.get('/health', (req, res) => {
  if (isShuttingDown) {
    return res.status(503).json({ status: 'shutting-down' })
  }
  res.json({ status: 'ok', uptime: process.uptime(), timestamp: new Date() })
})
```

---

## CI/CD với GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: testpassword
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        options: --health-cmd "redis-cli ping" --health-interval 10s
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:testpassword@localhost:5432/testdb

      - name: Run tests
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://postgres:testpassword@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret-key-at-least-32-characters

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build-and-push:
    name: Build Docker Image
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

  deploy:
    name: Deploy to Production
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            docker pull ghcr.io/${{ github.repository }}:latest
            docker-compose up -d --no-deps api
```

---

## Bài tập thực hành

1. **Docker setup**: Containerize một Express + PostgreSQL app. Viết Dockerfile multi-stage, docker-compose.yml với db + redis + api. Test locally.

2. **Graceful shutdown**: Implement complete graceful shutdown — stop accepting new connections, finish existing requests, close DB/Redis connections, verify với `curl` trong một terminal và `docker stop` trong terminal khác.

3. **GitHub Actions CI**: Setup CI pipeline cho project — chạy tests với real PostgreSQL service, check coverage threshold (>70%), fail build nếu tests fail.

4. **PM2 cluster**: Deploy app với PM2 cluster mode (4 instances), setup `ecosystem.config.js`, test zero-downtime reload với `pm2 reload`.

---

## Resources

- [PM2 docs](https://pm2.keymetrics.io/docs/) — Process manager
- [Docker docs — Node.js best practices](https://docs.docker.com/language/nodejs/) — Official guide
- [GitHub Actions docs](https://docs.github.com/en/actions) — CI/CD pipeline
