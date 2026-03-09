# Phase 1 — Backend với Node.js + Express

**Thời gian**: 6–8 tuần | **Mục tiêu**: Tư duy BE, build REST API production-ready

## Checklist

### Node.js Core
- [ ] Node.js runtime vs browser JS
- [ ] Streams & Buffer
- [ ] EventEmitter pattern
- [ ] `fs`, `path`, `http` module
- [ ] `child_process` cơ bản

### Express.js
- [ ] Middleware pattern
- [ ] Routing, Router module hóa
- [ ] Error handling middleware
- [ ] Request validation (Zod)
- [ ] File upload (multer)

### Bảo mật BE
- [ ] CORS — cấu hình đúng
- [ ] Helmet.js
- [ ] Rate limiting
- [ ] Input sanitization / SQL Injection / XSS
- [ ] Password hashing: bcrypt / argon2
- [ ] JWT từ đầu (không dùng magic library)
- [ ] OAuth2 / OpenID Connect flow
- [ ] RBAC (Role-based access control)

### Environment & Configuration
- [ ] `.env` và `process.env`
- [ ] Secrets management — không commit secret vào git
- [ ] 12-Factor App methodology

### Practical skills
- [ ] Gửi email (Nodemailer + Resend)
- [ ] Webhook cơ bản

## Tài nguyên
- [Node.js Official Docs](https://nodejs.org/en/docs)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [Zod Documentation](https://zod.dev)
- [12factor.net](https://12factor.net)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

## Project: Blog API
```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
DELETE /auth/logout
GET    /posts          (pagination, filtering)
POST   /posts          (auth required)
PUT    /posts/:id      (auth + owner check)
DELETE /posts/:id      (auth + RBAC)
POST   /posts/:id/upload-image
```
