# Phase 9 — Testing

Testing đảm bảo code hoạt động đúng theo thời gian — khi refactor, thêm features, hoặc fix bugs. Node.js ecosystem có excellent testing tools: Vitest (nhanh, ESM native), Jest (popular), Supertest (HTTP integration), và Playwright (E2E).

---

## Testing Pyramid

```
           /\
          /E2E\           ← ít nhất, chậm nhất, tốn kém nhất
         /------\
        /  Integ \        ← vừa phải — test API endpoints
       /----------\
      /   Unit     \      ← nhiều nhất, nhanh nhất, rẻ nhất
     /--------------\
```

- **Unit**: test một function/class đơn lẻ, mock dependencies
- **Integration**: test nhiều components cùng nhau (API route + service + DB)
- **E2E**: test từ browser perspective, full stack

---

## Vitest — Unit Testing (Recommended)

Vitest nhanh hơn Jest, native ESM support, Vite ecosystem:

```bash
npm install -D vitest @vitest/coverage-v8
```

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

```js
// src/services/__tests__/user.service.test.js
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { UserService } from '../user.service'

// Mock module
vi.mock('bcrypt', () => ({
  hash: vi.fn().mockResolvedValue('$2b$12$hashedpassword'),
  compare: vi.fn().mockResolvedValue(true)
}))

describe('UserService', () => {
  let userService
  let mockUserRepo

  beforeEach(() => {
    // Fresh mock cho mỗi test
    mockUserRepo = {
      create: vi.fn(),
      findByEmail: vi.fn(),
      findById: vi.fn(),
      update: vi.fn(),
      delete: vi.fn()
    }
    userService = new UserService(mockUserRepo)
  })

  describe('createUser', () => {
    it('should hash password before saving', async () => {
      mockUserRepo.findByEmail.mockResolvedValue(null)  // email không tồn tại
      mockUserRepo.create.mockResolvedValue({ id: 1, email: 'a@b.com' })

      await userService.createUser({
        email: 'a@b.com',
        password: 'plaintext123'
      })

      expect(mockUserRepo.create).toHaveBeenCalledWith(
        expect.objectContaining({
          password: expect.stringMatching(/^\$2b\$/)  // bcrypt hash pattern
        })
      )
    })

    it('should throw ConflictError if email exists', async () => {
      mockUserRepo.findByEmail.mockResolvedValue({ id: 1, email: 'a@b.com' })

      await expect(
        userService.createUser({ email: 'a@b.com', password: 'password' })
      ).rejects.toThrow('Email already exists')
    })

    it('should return user without password', async () => {
      mockUserRepo.findByEmail.mockResolvedValue(null)
      mockUserRepo.create.mockResolvedValue({
        id: 1, email: 'a@b.com', name: 'Alice', password: 'hash'
      })

      const result = await userService.createUser({ email: 'a@b.com', password: 'pw' })

      expect(result).not.toHaveProperty('password')
    })
  })

  describe('validateLogin', () => {
    it('should return user on valid credentials', async () => {
      const mockUser = { id: 1, email: 'a@b.com', password: 'hash' }
      mockUserRepo.findByEmail.mockResolvedValue(mockUser)

      const result = await userService.validateLogin('a@b.com', 'correct')

      expect(result).toEqual(expect.objectContaining({ id: 1 }))
    })

    it('should throw on invalid password', async () => {
      mockUserRepo.findByEmail.mockResolvedValue({ id: 1, password: 'hash' })
      const { compare } = await import('bcrypt')
      compare.mockResolvedValueOnce(false)  // wrong password this time

      await expect(
        userService.validateLogin('a@b.com', 'wrong')
      ).rejects.toThrow('Invalid credentials')
    })
  })
})
```

---

## Jest (Alternative)

```bash
npm install -D jest @types/jest
```

```js
// Syntax rất similar với Vitest
// jest.mock thay vì vi.mock
// jest.fn() thay vì vi.fn()
// jest.spyOn() thay vì vi.spyOn()

jest.mock('../../db/prisma', () => ({
  user: {
    create: jest.fn(),
    findUnique: jest.fn()
  }
}))
```

---

## Supertest — HTTP Integration Testing

```bash
npm install -D supertest
```

```js
// tests/integration/users.test.js
import request from 'supertest'
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest'
import app from '../../src/app'
import { prisma } from '../../src/db/prisma'

// Dùng test database
beforeAll(async () => {
  await prisma.$connect()
})

afterAll(async () => {
  await prisma.$disconnect()
})

beforeEach(async () => {
  // Clean tables trước mỗi test
  await prisma.user.deleteMany()
})

describe('POST /api/users', () => {
  it('should create user and return 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        name: 'Alice',
        email: 'alice@test.com',
        password: 'Password123!'
      })
      .expect(201)

    expect(response.body).toMatchObject({
      name: 'Alice',
      email: 'alice@test.com'
    })
    expect(response.body).toHaveProperty('id')
    expect(response.body).not.toHaveProperty('password')
  })

  it('should return 400 on invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Bob', email: 'not-an-email', password: 'Password123!' })
      .expect(400)

    expect(response.body).toHaveProperty('error')
  })

  it('should return 409 on duplicate email', async () => {
    // Create user first
    await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@test.com', password: 'Password123!' })

    // Try to create again
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Alice2', email: 'alice@test.com', password: 'Password123!' })
      .expect(409)
  })
})

describe('GET /api/users/:id', () => {
  it('should return user by id', async () => {
    // Create user in DB first
    const user = await prisma.user.create({
      data: { name: 'Alice', email: 'alice@test.com', password: 'hash' }
    })

    const response = await request(app)
      .get(`/api/users/${user.id}`)
      .set('Authorization', `Bearer ${generateTestToken(user.id)}`)
      .expect(200)

    expect(response.body).toMatchObject({ id: user.id, name: 'Alice' })
  })

  it('should return 404 for non-existent user', async () => {
    await request(app)
      .get('/api/users/99999')
      .set('Authorization', `Bearer ${generateTestToken(1)}`)
      .expect(404)
  })
})
```

---

## Test Database Setup

```js
// Test với SQLite in-memory (fast, no Docker needed)
// prisma/schema.test.prisma
datasource db {
  provider = "sqlite"
  url      = "file::memory:"
}

// vitest.config.js
export default {
  test: {
    globalSetup: './tests/setup.js',
    env: {
      DATABASE_URL: 'file::memory:'
    }
  }
}
```

```js
// Với testcontainers — real PostgreSQL trong Docker
import { PostgreSqlContainer } from '@testcontainers/postgresql'

let container
let testPrisma

beforeAll(async () => {
  container = await new PostgreSqlContainer('postgres:16-alpine').start()
  process.env.DATABASE_URL = container.getConnectionUri()

  testPrisma = new PrismaClient({ datasources: { db: { url: container.getConnectionUri() } } })
  await testPrisma.$executeRaw`CREATE TABLE ...`  // setup schema
})

afterAll(async () => {
  await testPrisma.$disconnect()
  await container.stop()
})
```

---

## E2E Testing với Playwright

```js
// tests/e2e/auth.spec.js
import { test, expect } from '@playwright/test'

test.describe('Authentication', () => {
  test('user can register and login', async ({ page }) => {
    // Register
    await page.goto('/register')
    await page.fill('[name=name]', 'Alice')
    await page.fill('[name=email]', 'alice@test.com')
    await page.fill('[name=password]', 'Password123!')
    await page.click('button[type=submit]')

    await expect(page).toHaveURL('/dashboard')
    await expect(page.locator('h1')).toContainText('Welcome, Alice')
  })

  test('shows error on invalid credentials', async ({ page }) => {
    await page.goto('/login')
    await page.fill('[name=email]', 'nonexistent@test.com')
    await page.fill('[name=password]', 'wrongpassword')
    await page.click('button[type=submit]')

    await expect(page.locator('[data-testid=error-message]'))
      .toContainText('Invalid credentials')
    await expect(page).toHaveURL('/login')  // stays on login page
  })
})
```

---

## Mocking Patterns

```js
// Spy on existing function
const hashSpy = vi.spyOn(bcrypt, 'hash').mockResolvedValue('mocked-hash')
expect(hashSpy).toHaveBeenCalledWith('plaintext', 12)
hashSpy.mockRestore()  // restore original

// Mock entire module
vi.mock('../email.service', () => ({
  sendEmail: vi.fn().mockResolvedValue({ messageId: 'test-123' }),
  sendWelcomeEmail: vi.fn().mockResolvedValue(true)
}))

// Mock with implementation
vi.mock('../config', () => ({
  default: {
    db: { url: 'test-db-url' },
    jwt: { secret: 'test-secret' }
  }
}))

// Timer mocks
vi.useFakeTimers()
vi.advanceTimersByTime(1000)  // advance time 1 second
vi.useRealTimers()
```

---

## Code Coverage

```bash
# Chạy coverage
npx vitest run --coverage

# Coverage report
# -------------------------------|---------|----------|---------|---------|
# File                           | % Stmts | % Branch | % Funcs | % Lines |
# -------------------------------|---------|----------|---------|---------|
# src/services/user.service.js   |   95.2  |   88.9   |   100   |   95.2  |
# src/middleware/auth.js         |   100   |   100    |   100   |   100   |
# -------------------------------|---------|----------|---------|---------|
```

Target: **>70% coverage cho business logic** (services, utils) — không cần 100% cho routes, config.

---

## Bài tập thực hành

1. **Unit test service**: Viết full unit tests cho `UserService` — mock repository, test tất cả methods, bao gồm edge cases (email conflict, user not found, invalid password).

2. **Integration test API**: Test tất cả CRUD endpoints với Supertest — 201/200/404/400/409 responses, auth required endpoints, pagination.

3. **Coverage report**: Đạt >70% coverage cho `src/services/`. Generate HTML report, identify uncovered branches và add tests.

4. **E2E test**: Dùng Playwright test user flow: register → login → create post → view post → logout. Run với headed mode để xem browser.

---

## Resources

- [Vitest docs](https://vitest.dev) — Test framework
- [Supertest GitHub](https://github.com/ladjs/supertest) — HTTP integration testing
- [Playwright docs](https://playwright.dev/docs/intro) — E2E testing
