# 05 — DevOps

> Fullstack cần biết code chạy ở đâu và như thế nào. Không cần thành DevOps engineer — cần đủ để tự deploy, debug production, và không phá server.

---

## Junior — Linux, Docker, CI/CD cơ bản

### Linux & Bash

| Topic | Phải làm được |
|-------|---------------|
| Hệ thống file | `/etc`, `/var`, `/home`, `/tmp`, `/opt` — mỗi folder dùng để làm gì |
| User & Permissions | `useradd`, `passwd`, `sudo`, `chmod`, `chown`, `umask` |
| Process management | `ps aux`, `kill -9`, `top`/`htop`, `nohup`, `&` background |
| Networking | `curl`, `wget`, `netstat`/`ss`, `ping`, `traceroute`, `lsof -i` |
| Disk & Memory | `df -h`, `du -sh`, `free -m`, `iostat` |
| Logs | `journalctl -u`, `tail -f`, `grep` trong logs |
| Bash scripting | Variables, `if/else`, `for` loop, functions, exit codes, `$?` |
| Cron jobs | `crontab -e`, cron syntax `* * * * *`, cron.d |

### systemd

| Topic | Phải làm được |
|-------|---------------|
| Service management | `start`, `stop`, `restart`, `enable`, `disable`, `status` |
| Service file | Viết `.service` file cho Node.js app |
| Logs | `journalctl -u service-name -f` |

### Nginx

| Topic | Phải cấu hình được |
|-------|-------------------|
| Reverse proxy | Forward request từ port 80 → Node.js app port 3000 |
| SSL termination | Let's Encrypt + Certbot — HTTPS miễn phí |
| Static file serving | Serve build output trực tiếp từ Nginx |
| Rate limiting | `limit_req_zone`, `limit_req` |
| Gzip compression | `gzip on`, `gzip_types` |

### Docker

| Topic | Phải làm được |
|-------|---------------|
| Container vs VM | Process isolation vs OS isolation — tại sao container nhẹ hơn |
| Dockerfile | `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`, `ENTRYPOINT` |
| Build context | `.dockerignore` — không copy `node_modules` |
| Image layers | Layer caching — order instructions để tối ưu rebuild time |
| Multi-stage build | Build stage + production stage — giảm image size |
| Docker Compose | `services`, `volumes`, `networks`, `depends_on`, `env_file` |
| Stack Node.js app | app + PostgreSQL + Redis + Nginx trong 1 compose file |

### GitHub Actions (CI/CD)

| Topic | Phải viết được |
|-------|----------------|
| Workflow syntax | `on`, `jobs`, `steps`, `uses`, `run`, `env` |
| Pipeline cơ bản | lint → test → build → push Docker image |
| Secrets | `secrets.DOCKER_PASSWORD` — không hardcode credentials |
| Branch protection | Require CI pass trước khi merge |
| Deploy via SSH | `appleboy/ssh-action` — SSH vào server, pull + restart |

**Checkpoint Junior**: Containerize app từ Phase 04, GitHub Actions pipeline auto-deploy lên VPS khi merge vào main.

---

## Mid — Cloud, IaC, Monitoring

### AWS Core Services

| Service | Phải biết dùng |
|---------|----------------|
| EC2 | Launch instance, security groups, key pairs, elastic IP |
| VPC | Subnets (public/private), route tables, Internet Gateway |
| S3 | Buckets, policies, CORS, versioning, lifecycle rules |
| RDS | Managed PostgreSQL, Multi-AZ, automated backups |
| ECR | Push Docker images, lifecycle policies |
| ECS Fargate | Deploy containers không cần quản lý EC2 |
| Route53 | Hosted zones, A/CNAME records, health checks |
| IAM | Users, roles, policies — principle of least privilege |
| SES | Gửi email transactional |

### Infrastructure as Code

| Topic | Phải làm được |
|-------|---------------|
| Terraform cơ bản | `provider`, `resource`, `variable`, `output`, `data` |
| State management | `terraform.tfstate`, remote state (S3 + DynamoDB lock) |
| Modules | Tái sử dụng config, `module` block |
| Workflow | `init` → `plan` → `apply` → `destroy` |
| Không click-ops | Mọi infra thay đổi qua code, review qua PR |

### Ansible

| Topic | Phải làm được |
|-------|---------------|
| Inventory | Hosts file, groups |
| Playbooks | `tasks`, `handlers`, `variables`, `templates` |
| Modules | `apt`, `copy`, `template`, `service`, `docker_container` |
| Idempotency | Chạy nhiều lần → cùng kết quả |

### Monitoring & Observability

| Topic | Phải setup được |
|-------|----------------|
| Sentry | Error tracking FE + BE, source maps, alert rules |
| Structured logging | Pino/Winston với JSON format, correlation ID |
| Prometheus | `metrics` endpoint, `Counter`, `Gauge`, `Histogram` |
| Grafana | Dashboard từ Prometheus data, alert rules |
| Uptime monitoring | UptimeRobot — alert khi service down |

**Checkpoint Mid**: IaC cho toàn bộ stack (Terraform), Ansible playbooks để configure servers, Grafana dashboard + Sentry setup.

---

## Senior — Kubernetes, Security, SRE

### Kubernetes

| Topic | Phải hiểu |
|-------|-----------|
| Core objects | Pod, Deployment, Service, Ingress, ConfigMap, Secret |
| Networking | ClusterIP vs NodePort vs LoadBalancer, DNS trong cluster |
| Storage | PersistentVolume, PersistentVolumeClaim, StorageClass |
| Configuration | ConfigMap cho config, Secret cho credentials (base64) |
| Health checks | `livenessProbe`, `readinessProbe`, `startupProbe` |
| Resource management | `requests` và `limits` — CPU, memory |
| HPA | Horizontal Pod Autoscaler, metrics-based scaling |
| Rolling updates | `maxSurge`, `maxUnavailable`, rollback strategy |
| Managed K8s | EKS (AWS), GKE (GCP), AKS (Azure) — trade-offs |

### Advanced CI/CD

| Topic | Phải biết |
|-------|-----------|
| GitOps | ArgoCD / Flux — git là source of truth cho cluster state |
| Blue/Green deployment | Zero-downtime deployments |
| Canary releases | Route % traffic đến new version |
| Secret management | HashiCorp Vault, Sealed Secrets, AWS Secrets Manager |

### SRE Practices

| Topic | Phải hiểu |
|-------|-----------|
| SLO / SLI / SLA | Định nghĩa reliability targets, error budget |
| Incident management | Runbooks, post-mortems, blameless culture |
| Log aggregation | ELK Stack hoặc Grafana Loki |
| Distributed tracing | OpenTelemetry, Jaeger / Zipkin |
| Chaos engineering | Awareness — Chaos Monkey, fault injection |

---

## Resources

- **Docker Deep Dive** (Nigel Poulton) — ngắn, đủ dùng
- [roadmap.sh/devops](https://roadmap.sh/devops) — chi tiết từng bước
- [DigitalOcean Tutorials](https://www.digitalocean.com/community/tutorials) — Nginx, Linux, Docker practical
- [learn.hashicorp.com](https://developer.hashicorp.com/terraform/tutorials) — Terraform tutorials
- [Kubernetes docs](https://kubernetes.io/docs) — official, dùng làm reference
- TechWorld with Nana YouTube — Docker, K8s, DevOps từng bước
