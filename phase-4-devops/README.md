# Phase 4 — DevOps & Infrastructure cơ bản

**Thời gian**: 3–4 tuần | **Mục tiêu**: Code của mình chạy ở đâu và như thế nào

## Checklist

### Linux & Command Line
- [ ] Navigation, file management, permissions (chmod, chown)
- [ ] `grep`, `awk`, `sed`
- [ ] Process management: `ps`, `kill`, `top`, `htop`
- [ ] SSH, SCP, rsync
- [ ] Cron jobs
- [ ] systemd — quản lý service

### Nginx
- [ ] Nginx vs Apache
- [ ] Reverse proxy cho Node.js app
- [ ] SSL termination + Let's Encrypt (Certbot)
- [ ] Static file serving
- [ ] Load balancing
- [ ] Rate limiting

### Docker
- [ ] Container vs VM
- [ ] Dockerfile cho Node.js app
- [ ] Docker Compose: app + PostgreSQL + Redis + Nginx
- [ ] Image optimization (layer caching, multi-stage build)
- [ ] Docker networking
- [ ] Docker volumes

### Kubernetes (awareness)
- [ ] Tại sao Docker Compose không đủ ở production
- [ ] Pod, Deployment, Service, Ingress
- [ ] Đủ để làm việc với team DevOps

### CI/CD
- [ ] GitHub Actions pipeline
- [ ] lint → test → build → push Docker image → deploy
- [ ] Environment secrets trong CI/CD
- [ ] Branch protection rules

### Monitoring & Logging
- [ ] Structured logging với Pino hoặc Winston
- [ ] Log levels đúng: debug/info/warn/error
- [ ] Sentry — error tracking FE + BE
- [ ] Metrics cơ bản (response time, error rate)
- [ ] Uptime monitoring

### Cloud (chọn 1)
- [ ] AWS: EC2, S3, RDS, ECR, ECS, IAM
- [ ] **hoặc** GCP: Cloud Run, Cloud SQL, Cloud Storage
- [ ] **hoặc** Fly.io / Railway (dễ nhất để bắt đầu)

## Tài nguyên
- [Docker Deep Dive (Nigel Poulton)](https://www.nigelpoulton.com/books/) — ngắn, đủ dùng
- [DigitalOcean Nginx tutorials](https://www.digitalocean.com/community/tags/nginx)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Fly.io Docs](https://fly.io/docs/)
- [Sentry Docs](https://docs.sentry.io/)
