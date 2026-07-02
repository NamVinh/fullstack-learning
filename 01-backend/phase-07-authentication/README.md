# Phase 7 — Authentication & Authorization

Authentication (authn) xác nhận **bạn là ai**, Authorization (authz) xác nhận **bạn được làm gì**. Đây là phần quan trọng nhất về security — implement sai sẽ dẫn đến data breach nghiêm trọng.

---

## Password Hashing với bcrypt

**KHÔNG BAO GIỜ** lưu plain text password. Dùng bcrypt — adaptive hash function với salt, chậm có chủ ý để resist brute-force:

```js
const bcrypt = require('bcrypt')
const SALT_ROUNDS = 12  // 10-12 là standard; cao hơn = chậm hơn = an toàn hơn

// Hash password khi register
const passwordHash = await bcrypt.hash(plainTextPassword, SALT_ROUNDS)
// Lưu passwordHash vào DB, KHÔNG lưu plainTextPassword

// Verify password khi login
const isValid = await bcrypt.compare(plainTextPassword, passwordHash)
// bcrypt.compare tự handle salt extraction từ hash
if (!isValid) throw new UnauthorizedError('Invalid credentials')

// Đừng leak timing information — compare luôn đủ thời gian dù password sai
```

**Tại sao SALT_ROUNDS = 12?**
- 10: ~100ms per hash (minimum acceptable)
- 12: ~400ms per hash (recommended production)
- 14: ~1.5s per hash (nếu security cực kỳ quan trọng)

---

## JWT (JSON Web Token)

JWT là self-contained token chứa claims (payload) — không cần server state để verify.

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiIsImlhdCI6MTcwMDAwMDAwMCwiZXhwIjoxNzAwMDAwOTAwfQ.SIGNATURE
```

```js
const jwt = require('jsonwebtoken')

// Sign — tạo access token (short-lived)
const accessToken = jwt.sign(
  {
    userId: user.id,      // payload — đừng đặt sensitive data
    role: user.role,
    email: user.email
  },
  process.env.JWT_SECRET,         // secret key
  { expiresIn: '15m' }           // 15 minutes — ngắn để giảm risk nếu bị leak
)

// Sign refresh token (long-lived, lưu ID vào DB)
const refreshToken = jwt.sign(
  { userId: user.id, tokenId: crypto.randomUUID() },
  process.env.JWT_REFRESH_SECRET,
  { expiresIn: '7d' }
)

// Verify
try {
  const payload = jwt.verify(token, process.env.JWT_SECRET)
  // payload.userId, payload.role, payload.exp (Unix timestamp)
  // payload.iat (issued at)
} catch (err) {
  if (err instanceof jwt.TokenExpiredError) {
    throw new UnauthorizedError('Token expired')
  }
  if (err instanceof jwt.JsonWebTokenError) {
    throw new UnauthorizedError('Invalid token')
  }
  throw err
}
```

---

## Access Token + Refresh Token Flow

```
Client                              Server
  │── POST /auth/login ──────────────►│
  │   { email, password }             │  verify password
  │◄── { accessToken (15m),  ─────────│  store refreshToken ID in DB
  │       refreshToken (7d) }         │
  │
  │── GET /api/data ────────────────►│
  │   Authorization: Bearer <access>  │  verify JWT
  │◄── 200 { data } ─────────────────│
  │
  │  (access expired after 15m)
  │── POST /auth/refresh ───────────►│
  │   { refreshToken }               │  verify refresh token
  │                                   │  check token ID exists in DB (not revoked)
  │◄── { accessToken (new 15m) } ────│
  │
  │── POST /auth/logout ────────────►│
  │   { refreshToken }               │  delete token ID from DB
  │◄── 204 No Content ───────────────│
```

---

## Auth Middleware

```js
const jwt = require('jsonwebtoken')

// Authentication — verify token và attach user to request
const authenticate = async (req, res, next) => {
  // Lấy token từ Authorization header
  const authHeader = req.headers.authorization
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' })
  }

  const token = authHeader.split(' ')[1]

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    req.user = payload  // { userId, role, email, exp, iat }
    next()
  } catch (err) {
    if (err instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ error: 'Token expired' })
    }
    return res.status(401).json({ error: 'Invalid token' })
  }
}

// Authorization — RBAC (Role-Based Access Control)
const authorize = (...roles) => (req, res, next) => {
  if (!req.user) return res.status(401).json({ error: 'Not authenticated' })
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Insufficient permissions' })
  }
  next()
}

// ABAC (Attribute-Based Access Control) — fine-grained
const authorizeOwner = (getResourceOwnerId) => async (req, res, next) => {
  const ownerId = await getResourceOwnerId(req)
  if (req.user.role === 'admin') return next()  // admin bypass
  if (req.user.userId !== ownerId) {
    return res.status(403).json({ error: 'Not your resource' })
  }
  next()
}

// Usage
app.get('/admin', authenticate, authorize('admin'), adminHandler)
app.delete('/posts/:id', authenticate, authorizeOwner(
  async (req) => {
    const post = await db.post.findById(req.params.id)
    return post?.authorId
  }
), deletePostHandler)
```

---

## Login và Register Endpoints

```js
const bcrypt = require('bcrypt')
const jwt = require('jsonwebtoken')
const crypto = require('crypto')

// Register
app.post('/auth/register', async (req, res, next) => {
  try {
    const { name, email, password } = req.body

    // Check email unique
    const exists = await db.user.findByEmail(email)
    if (exists) return res.status(409).json({ error: 'Email already registered' })

    // Hash password
    const passwordHash = await bcrypt.hash(password, 12)

    // Create user
    const user = await db.user.create({ name, email, password: passwordHash })

    res.status(201).json({
      id: user.id,
      name: user.name,
      email: user.email
    })
  } catch (err) {
    next(err)
  }
})

// Login
app.post('/auth/login', async (req, res, next) => {
  try {
    const { email, password } = req.body

    // Find user (include password field)
    const user = await db.user.findByEmail(email, { includePassword: true })

    // Constant-time comparison — đừng leak timing info
    if (!user || !(await bcrypt.compare(password, user.password))) {
      return res.status(401).json({ error: 'Invalid email or password' })
    }

    // Create tokens
    const accessToken = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    )

    const tokenId = crypto.randomUUID()
    const refreshToken = jwt.sign(
      { userId: user.id, tokenId },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: '7d' }
    )

    // Store refresh token ID in DB
    await db.refreshToken.create({
      userId: user.id,
      tokenId,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
    })

    res.json({ accessToken, refreshToken })
  } catch (err) {
    next(err)
  }
})

// Refresh
app.post('/auth/refresh', async (req, res, next) => {
  try {
    const { refreshToken } = req.body
    if (!refreshToken) return res.status(400).json({ error: 'Refresh token required' })

    let payload
    try {
      payload = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET)
    } catch {
      return res.status(401).json({ error: 'Invalid refresh token' })
    }

    // Check token ID exists in DB (not revoked)
    const storedToken = await db.refreshToken.findByTokenId(payload.tokenId)
    if (!storedToken || storedToken.revokedAt) {
      return res.status(401).json({ error: 'Refresh token revoked' })
    }

    // Issue new access token
    const user = await db.user.findById(payload.userId)
    const accessToken = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    )

    res.json({ accessToken })
  } catch (err) {
    next(err)
  }
})
```

---

## OAuth2 / OpenID Connect

Cho phép user đăng nhập bằng Google, GitHub, Facebook thay vì tạo account mới:

```
Authorization Code Flow:
1. User click "Login with Google"
2. App redirect → Google OAuth endpoint
3. User authorize → Google redirect back với ?code=xxx
4. App exchange code → access_token + id_token
5. App dùng id_token để lấy user info (email, name)
6. App tạo/update user trong DB → issue own JWT
```

**Dùng Passport.js:**
```js
const passport = require('passport')
const { Strategy: GoogleStrategy } = require('passport-google-oauth20')

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
  try {
    let user = await db.user.findByGoogleId(profile.id)
    if (!user) {
      user = await db.user.create({
        googleId: profile.id,
        email: profile.emails[0].value,
        name: profile.displayName
      })
    }
    done(null, user)
  } catch (err) {
    done(err)
  }
}))

app.get('/auth/google', passport.authenticate('google', { scope: ['email', 'profile'] }))
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    const token = jwt.sign({ userId: req.user.id }, process.env.JWT_SECRET, { expiresIn: '15m' })
    res.redirect(`https://myapp.com/callback?token=${token}`)
  }
)
```

---

## Sessions vs JWT

| | Sessions | JWT |
|--|---------|-----|
| Storage | Server-side (Redis/DB) | Client-side (localStorage/cookie) |
| Revocation | Dễ — xóa từ store ngay lập tức | Khó — phải dùng blacklist hoặc đợi expire |
| Scale | Cần shared store (Redis) | Stateless — scale dễ dàng |
| Payload size | Nhỏ (chỉ session ID) | Lớn hơn (chứa claims) |
| Dùng khi | Traditional web app, cần revoke tức thì | SPA, mobile, microservices |

---

## Bài tập thực hành

1. **Full auth flow**: Register, login, refresh token, logout. Dùng bcrypt + JWT. Test với httpie hoặc Postman.

2. **RBAC**: Implement routes với 3 roles: `guest` (read only), `user` (own resources), `admin` (all). Test unauthorized access.

3. **OAuth2**: Thêm "Login with GitHub" bằng Passport.js. Store GitHub profile data, handle case email đã tồn tại.

4. **Security audit**: Review auth implementation tìm lỗi: timing attacks, token storage, HTTPS-only cookies, CSRF protection.

---

## Resources

- [JWT.io](https://jwt.io) — JWT debugger và thư viện
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — Security best practices
- [Passport.js docs](https://www.passportjs.org/docs/) — Authentication strategies
