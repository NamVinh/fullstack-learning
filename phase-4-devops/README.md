# Phase 4 — DevOps & Infrastructure cơ bản

**Thời gian**: 3–4 tuần | **Mục tiêu**: Code của mình chạy ở đâu và như thế nào

---

## Linux & Command Line

---

### Navigation & File Management

- [ ] Sống sót được trên server Linux không có GUI

**Học:**
1. 📖 [Linux Command Line Basics — Ubuntu Docs](https://ubuntu.com/tutorials/command-line-for-beginners)
2. 📖 [chmod explained — Linux permissions](https://www.guru99.com/file-permissions.html)

**Exercises:**

🛠️ Làm trọn bộ bài tập này trên terminal (không dùng GUI):
```bash
# 1. Tạo cấu trúc thư mục
mkdir -p ~/practice/{logs,config,scripts}

# 2. Tạo file và thao tác
echo "Hello World" > ~/practice/config/app.conf
cat ~/practice/config/app.conf
cp ~/practice/config/app.conf ~/practice/config/app.conf.bak

# 3. Permissions — hiểu rwxrwxrwx
ls -la ~/practice/
chmod 755 ~/practice/scripts    # owner: rwx, group: r-x, others: r-x
chmod 600 ~/practice/config/app.conf  # chỉ owner đọc/ghi

# 4. Find và grep
grep -r "Hello" ~/practice/
find ~/practice/ -name "*.conf" -newer ~/practice/config/app.conf
```

🛠️ Giải thích: `755` có nghĩa là gì? Tại sao `.env` file nên dùng `600`?

---

### Process Management & SSH

- [ ] Quản lý processes và kết nối server

**Học:**
1. 📖 [Linux Process Management](https://www.digitalocean.com/community/tutorials/process-management-in-linux)
2. 📖 [SSH Essentials — DigitalOcean](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)

**Exercises:**

🛠️ Process management:
```bash
# Xem processes đang chạy
ps aux | grep node
top  # hoặc htop nếu đã cài

# Chạy Node.js app ở background
node app.js &
echo "PID: $!"        # Lưu PID
kill -9 <PID>         # Dừng process

# Xem process nào đang dùng port 3000
lsof -i :3000
```

🛠️ SSH key setup:
```bash
# Tạo SSH key pair (nếu chưa có)
ssh-keygen -t ed25519 -C "your@email.com"

# Copy public key lên server (hoặc GitHub)
cat ~/.ssh/id_ed25519.pub
# Paste vào GitHub Settings → SSH Keys
```

---

## Nginx

---

### Reverse Proxy & SSL

- [ ] Nginx đứng trước Node.js app — rất phổ biến ở production

**Học:**
1. 📖 [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
2. 📖 [How to Configure Nginx as Reverse Proxy — DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04)
3. 📖 [Let's Encrypt with Nginx — Certbot](https://certbot.eff.org/instructions?os=ubuntufocal&certtype=nginx)

**Exercises:**

🛠️ Cấu hình Nginx reverse proxy cơ bản:
```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name myapp.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
```

🛠️ Thêm SSL với Let's Encrypt:
```bash
# Trên VPS Ubuntu
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d myapp.example.com
# Certbot tự sửa nginx config + setup auto-renewal
```

---

## Docker

---

### Dockerfile cho Node.js

- [ ] Đóng gói app để chạy ở mọi môi trường

**Học:**
1. 📖 [Dockerfile Best Practices — Docker Docs](https://docs.docker.com/develop/dev-best-practices/)
2. 📖 [Multi-stage builds — Docker Docs](https://docs.docker.com/build/building/multi-stage/)
3. 📖 [Node.js Docker Best Practices — nodejs.org](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)

**Exercises:**

🛠️ Viết Dockerfile tối ưu cho Node.js app — từ naive đến production-ready:

**Version 1 (naive):**
```dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "src/index.js"]
```

**Version 2 (tối ưu hơn):**
```dockerfile
# Multi-stage: build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # Chỉ production deps

COPY . .
RUN npm run build  # TypeScript compile

# Runtime stage — image nhỏ hơn
FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node  # Không chạy với root
CMD ["node", "dist/index.js"]
```

So sánh kích thước image của Version 1 và Version 2: `docker images`.

🛠️ Layer caching — giải thích tại sao thứ tự này quan trọng:
```dockerfile
# ✅ Đúng: copy package.json TRƯỚC, code SAU
COPY package*.json ./
RUN npm install
COPY src/ ./src/

# ❌ Sai: mỗi lần code thay đổi đều npm install lại
COPY . .
RUN npm install
```

---

### Docker Compose

- [ ] Chạy cả stack (app + DB + Redis) với 1 lệnh

**Học:**
1. 📖 [Docker Compose — Getting Started](https://docs.docker.com/compose/gettingstarted/)
2. 📖 [Networking in Docker Compose](https://docs.docker.com/compose/networking/)

**Exercises:**

🛠️ Viết `docker-compose.yml` cho full stack:
```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:password@postgres:5432/mydb
      REDIS_URL: redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - api

volumes:
  postgres_data:
  redis_data:
```

🛠️ Test:
```bash
docker compose up -d
docker compose logs -f api     # Xem logs
docker compose exec postgres psql -U postgres mydb  # Kết nối vào DB
docker compose down -v         # Dừng và xóa volumes
```

---

## CI/CD với GitHub Actions

---

### Viết Pipeline

- [ ] Automation: mỗi khi push code, tự test + deploy

**Học:**
1. 📖 [GitHub Actions — Quickstart](https://docs.github.com/en/actions/quickstart)
2. 📖 [GitHub Actions — Using environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)

**Exercises:**

🛠️ Viết pipeline hoàn chỉnh `.github/workflows/deploy.yml`:
```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/testdb

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Chỉ deploy khi push vào main

    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Deploy to server
        # SSH vào server và pull image mới
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /app
            docker compose pull
            docker compose up -d --no-deps api
```

---

## Monitoring & Logging

---

### Structured Logging với Pino

- [ ] `console.log` không đủ ở production — cần log có thể search và filter

**Học:**
1. 📖 [Pino — Getting Started](https://getpino.io/#/?id=quick-start)
2. 📖 [Structured Logging — why it matters](https://www.sumologic.com/glossary/structured-logging/)

**Exercises:**

🛠️ Thay tất cả `console.log` trong project bằng Pino:
```ts
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  // Development: human-readable
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined, // Production: JSON
});

// Request logger middleware
app.use((req, res, next) => {
  req.log = logger.child({ requestId: req.id });
  req.log.info({ method: req.method, url: req.url }, 'Request received');
  next();
});

// Thay console.error
try {
  await doSomething();
} catch (err) {
  logger.error({ err, userId: req.user?.id }, 'Failed to do something');
  // Chú ý: truyền object TRƯỚC string — Pino convention
}
```

🛠️ Query logs sau:
```bash
# Chạy app → redirect logs vào file
node app.js | tee app.log

# Search logs
cat app.log | grep '"level":50'          # errors only
cat app.log | jq 'select(.level >= 40)'  # warn + error
cat app.log | jq 'select(.requestId == "abc123")'  # trace 1 request
```

---

### Sentry — Error Tracking

- [ ] Biết lỗi production trước khi user báo cáo

**Học:**
1. 📖 [Sentry — Node.js Quickstart](https://docs.sentry.io/platforms/javascript/guides/node/)
2. 📖 [Sentry — Next.js Guide](https://docs.sentry.io/platforms/javascript/guides/nextjs/)

**Exercises:**

🛠️ Setup Sentry cho cả API và Next.js app:
```ts
// API: src/instrument.ts (import ở đầu tiên)
import * as Sentry from '@sentry/node';
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // Chỉ track 10% transactions ở production
});
```

```ts
// Trong error handler
app.use((err, req, res, next) => {
  Sentry.captureException(err, {
    user: { id: req.user?.id, email: req.user?.email },
    extra: { requestId: req.id },
  });
  res.status(500).json({ error: 'Internal server error' });
});
```

Tạo thử 1 lỗi có ý → xem nó xuất hiện trong Sentry dashboard.

---

## Project Phase 4

Containerize và deploy project Phase 3:

1. **Dockerfile**: viết multi-stage build cho cả `apps/api` và `apps/web`
2. **docker-compose.yml**: full stack local (api + web + postgres + redis + nginx)
3. **GitHub Actions**: pipeline CI test → build → push Docker Hub → deploy
4. **Nginx**: reverse proxy, SSL (dùng Certbot nếu có domain)
5. **Sentry**: setup cho cả API và Next.js app
6. **Logging**: replace console.log bằng Pino, xem log qua `docker compose logs`

**Mục tiêu cuối**: Mỗi khi `git push main` → tự động test + deploy. Zero manual steps.
