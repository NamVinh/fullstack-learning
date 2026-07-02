# Phase 15 — Advanced Architecture

Architecture patterns giúp code dễ maintain, test, và scale khi project lớn lên. Phase này cover những patterns quan trọng nhất cho Node.js backends: Repository Pattern, Clean Architecture, và gRPC cho microservices.

---

## Repository Pattern

Repository Pattern tách data access logic (database queries) khỏi business logic (service). Lợi ích: dễ test (mock repository), dễ swap database, separation of concerns.

```js
// repositories/user.repository.js
// Interface — định nghĩa contract
class UserRepository {
  async findById(id) { throw new Error('Not implemented') }
  async findByEmail(email) { throw new Error('Not implemented') }
  async create(data) { throw new Error('Not implemented') }
  async update(id, data) { throw new Error('Not implemented') }
  async delete(id) { throw new Error('Not implemented') }
  async findMany(options = {}) { throw new Error('Not implemented') }
}

// repositories/prisma-user.repository.js
// Concrete implementation với Prisma
const { prisma } = require('../db/prisma')

class PrismaUserRepository extends UserRepository {
  async findById(id) {
    return prisma.user.findUnique({ where: { id } })
  }

  async findByEmail(email) {
    return prisma.user.findUnique({ where: { email } })
  }

  async create(data) {
    return prisma.user.create({ data })
  }

  async update(id, data) {
    return prisma.user.update({ where: { id }, data })
  }

  async delete(id) {
    return prisma.user.delete({ where: { id } })
  }

  async findMany({ page = 1, limit = 10, where = {}, orderBy = { createdAt: 'desc' } } = {}) {
    const [data, total] = await Promise.all([
      prisma.user.findMany({
        where,
        orderBy,
        skip: (page - 1) * limit,
        take: limit
      }),
      prisma.user.count({ where })
    ])
    return { data, total, page, limit, pages: Math.ceil(total / limit) }
  }
}

// services/user.service.js
// Business logic — không biết về database
const bcrypt = require('bcrypt')
const { NotFoundError, ConflictError } = require('../errors')

class UserService {
  constructor(userRepository) {  // inject dependency
    this.userRepository = userRepository
  }

  async createUser({ name, email, password }) {
    const existing = await this.userRepository.findByEmail(email)
    if (existing) throw new ConflictError('Email already registered')

    const passwordHash = await bcrypt.hash(password, 12)
    const user = await this.userRepository.create({ name, email, password: passwordHash })

    const { password: _, ...userWithoutPassword } = user
    return userWithoutPassword
  }

  async getUserById(id) {
    const user = await this.userRepository.findById(id)
    if (!user) throw new NotFoundError(`User ${id}`)
    const { password, ...safeUser } = user
    return safeUser
  }

  async updateUser(id, updates) {
    await this.getUserById(id)  // verify exists
    if (updates.password) {
      updates.password = await bcrypt.hash(updates.password, 12)
    }
    return this.userRepository.update(id, updates)
  }

  async deleteUser(id) {
    await this.getUserById(id)  // verify exists
    await this.userRepository.delete(id)
  }
}

// Wiring — dependency injection tay (hoặc dùng DI container)
const userRepository = new PrismaUserRepository()
const userService = new UserService(userRepository)
module.exports = { userService }

// Testing — inject mock repository
const mockRepo = {
  findByEmail: jest.fn().mockResolvedValue(null),
  create: jest.fn().mockResolvedValue({ id: 1, email: 'a@b.com' })
}
const testService = new UserService(mockRepo)
```

---

## Clean Architecture

Clean Architecture (Robert Martin) đặt business logic ở trung tâm, UI và databases ở ngoài. **Dependency Rule**: dependencies chỉ trỏ vào — outer layers depend on inner layers, không bao giờ ngược lại.

```
┌─────────────────────────────────────────┐
│            Infrastructure               │  ← DB, HTTP, Redis, Email
│  ┌───────────────────────────────────┐  │
│  │         Application               │  │  ← Use Cases, Services
│  │  ┌─────────────────────────────┐  │  │
│  │  │         Domain              │  │  │  ← Entities, Business Rules
│  │  │  (pure JS, zero imports)    │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

```
src/
├── domain/                    ← CORE — zero external dependencies
│   ├── entities/
│   │   ├── user.js           ← User entity với business rules
│   │   └── post.js
│   ├── errors/
│   │   └── domain-errors.js  ← Domain-specific errors
│   └── repositories/
│       └── user.repository.js  ← Interface (abstract)
│
├── application/               ← Use cases, orchestration
│   ├── use-cases/
│   │   ├── create-user.use-case.js
│   │   ├── get-user.use-case.js
│   │   └── update-user.use-case.js
│   └── services/
│       └── auth.service.js
│
└── infrastructure/            ← Concrete implementations
    ├── database/
    │   ├── prisma-user.repository.js
    │   └── prisma.client.js
    ├── http/
    │   ├── express-app.js
    │   └── routes/
    │       └── user.routes.js
    ├── cache/
    │   └── redis-cache.js
    └── container.js           ← Dependency injection wiring
```

```js
// domain/entities/user.js
// Pure business logic — no database, no HTTP, no external libs
class User {
  constructor({ id, name, email, passwordHash, role = 'user', createdAt = new Date() }) {
    this.id = id
    this.name = name
    this.email = email
    this.passwordHash = passwordHash
    this.role = role
    this.createdAt = createdAt
  }

  // Business rules
  isAdmin() {
    return this.role === 'admin'
  }

  canEdit(resourceOwnerId) {
    return this.isAdmin() || this.id === resourceOwnerId
  }

  toPublicProfile() {
    return { id: this.id, name: this.name, email: this.email, role: this.role }
  }
}

module.exports = User
```

```js
// application/use-cases/create-user.use-case.js
// Orchestrate: validate → hash → save → notify
class CreateUserUseCase {
  constructor({ userRepository, emailService, hashService }) {
    this.userRepository = userRepository
    this.emailService = emailService
    this.hashService = hashService
  }

  async execute({ name, email, password }) {
    // 1. Validate business rule
    const existing = await this.userRepository.findByEmail(email)
    if (existing) {
      throw new EmailAlreadyExistsError(email)
    }

    // 2. Hash password
    const passwordHash = await this.hashService.hash(password)

    // 3. Create entity
    const user = new User({ name, email, passwordHash })

    // 4. Persist
    const savedUser = await this.userRepository.save(user)

    // 5. Side effects (không blocking)
    this.emailService.sendWelcome(savedUser.email).catch(err =>
      console.error('Failed to send welcome email:', err)
    )

    return savedUser.toPublicProfile()
  }
}
```

---

## gRPC — Microservices Internal Communication

gRPC dùng Protocol Buffers (binary format) — nhanh hơn JSON, type-safe, auto-generate client/server code.

```bash
npm install @grpc/grpc-js @grpc/proto-loader
```

```protobuf
// protos/user.proto
syntax = "proto3";

package user;

service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc CreateUser (CreateUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
  rpc StreamUsers (ListUsersRequest) returns (stream User);
}

message GetUserRequest {
  int32 id = 1;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
  string password = 3;
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  string role = 4;
  string created_at = 5;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}
```

```js
// grpc-server.js
const grpc = require('@grpc/grpc-js')
const protoLoader = require('@grpc/proto-loader')
const path = require('path')

const packageDef = protoLoader.loadSync(
  path.join(__dirname, '../protos/user.proto'),
  { keepCase: true, longs: String, enums: String, defaults: true, oneofs: true }
)

const { user: userProto } = grpc.loadPackageDefinition(packageDef)

const userServiceImpl = {
  async GetUser({ request }, callback) {
    try {
      const user = await userRepository.findById(request.id)
      if (!user) return callback({ code: grpc.status.NOT_FOUND, message: 'User not found' })
      callback(null, user)
    } catch (err) {
      callback({ code: grpc.status.INTERNAL, message: err.message })
    }
  },

  async CreateUser({ request }, callback) {
    try {
      const user = await userService.createUser(request)
      callback(null, user)
    } catch (err) {
      callback({ code: grpc.status.ALREADY_EXISTS, message: err.message })
    }
  },

  StreamUsers({ request }, call) {
    // Server streaming — send users one by one
    userRepository.streamAll({ page: request.page, limit: request.limit })
      .on('data', (user) => call.write(user))
      .on('end', () => call.end())
      .on('error', (err) => call.destroy(err))
  }
}

const server = new grpc.Server()
server.addService(userProto.UserService.service, userServiceImpl)
server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
  server.start()
  console.log('gRPC server running on port 50051')
})
```

```js
// grpc-client.js (từ microservice khác)
const grpc = require('@grpc/grpc-js')
const protoLoader = require('@grpc/proto-loader')
const { promisify } = require('util')

const packageDef = protoLoader.loadSync('./protos/user.proto', {
  keepCase: true, longs: String, enums: String, defaults: true, oneofs: true
})
const { user: userProto } = grpc.loadPackageDefinition(packageDef)

const client = new userProto.UserService(
  'user-service:50051',
  grpc.credentials.createInsecure()
)

// Promisify unary calls
const getUser = promisify(client.GetUser.bind(client))
const createUser = promisify(client.CreateUser.bind(client))

// Usage
const user = await getUser({ id: 42 })
const newUser = await createUser({ name: 'Alice', email: 'alice@example.com', password: 'pw' })

// Server streaming
const stream = client.StreamUsers({ page: 1, limit: 100 })
stream.on('data', (user) => console.log('Received:', user))
stream.on('end', () => console.log('Stream ended'))
```

---

## gRPC vs REST

| | gRPC | REST |
|--|------|------|
| Format | Binary (protobuf) — nhỏ, nhanh | JSON/XML — human-readable |
| Contract | `.proto` file — strongly typed | OpenAPI (optional) |
| Streaming | Native (bidirectional) | SSE/WebSocket (separate) |
| Dùng khi | Microservices internal comms | Public APIs, browser clients |
| Browser support | Cần gRPC-Web proxy | Native |

---

## Bài tập thực hành

1. **Repository Pattern**: Refactor một Express app đang dùng Prisma trực tiếp trong routes → Repository + Service layers. Unit test service với mock repository.

2. **Clean Architecture**: Implement một feature (ví dụ: post creation) theo Clean Architecture — domain entity, use case, repository interface, infrastructure implementation.

3. **gRPC service**: Build `UserService` gRPC server với GetUser, CreateUser, ListUsers. Build client (có thể là Express app khác gọi vào). Test với gRPCurl.

4. **Dependency Injection**: Implement simple DI container thay vì manual wiring — register services, resolve dependencies tự động, handle singleton vs transient.

---

## Resources

- [Clean Architecture — Robert Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) — Original article
- [gRPC Node.js docs](https://grpc.io/docs/languages/node/) — Official guide
- [Node.js Design Patterns — Mario Casciaro](https://www.nodejsdesignpatterns.com) — Patterns book
