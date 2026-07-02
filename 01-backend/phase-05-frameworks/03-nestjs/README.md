# Phase 5.3 — NestJS

NestJS là framework TypeScript-first, opinionated, lấy cảm hứng từ Angular. Nó cung cấp Dependency Injection, Modules, Controllers, Services, Guards, Interceptors, và Pipes — một kiến trúc rõ ràng cho enterprise applications. Đây là lựa chọn hàng đầu cho large teams và complex domain logic.

---

## Core Concepts

```
┌─────────────────────────────────────┐
│           NestJS App                │
│                                     │
│  AppModule                          │
│    ├── UsersModule                  │
│    │     ├── UsersController        │  ← HTTP routing
│    │     ├── UsersService           │  ← Business logic
│    │     ├── UserRepository         │  ← Data access
│    │     └── UserEntity             │  ← Data model
│    ├── AuthModule                   │
│    └── DatabaseModule               │
└─────────────────────────────────────┘
```

**Module**: Unit của encapsulation — group controllers, services, providers
**Controller**: Handle HTTP requests, delegate business logic sang Service
**Service**: Business logic, injectable, testable
**Provider**: Anything injectable (service, repository, factory, helper)
**Guard**: Authentication/Authorization
**Interceptor**: Transform request/response, logging, caching
**Pipe**: Validation, transformation của input

---

## Project Structure

```
src/
├── main.ts                    ← bootstrap app
├── app.module.ts              ← root module
├── common/
│   ├── decorators/
│   ├── filters/               ← exception filters (error handling)
│   ├── guards/                ← auth guards
│   ├── interceptors/          ← logging, transform
│   └── pipes/                 ← validation
└── users/
    ├── users.module.ts
    ├── users.controller.ts
    ├── users.service.ts
    ├── users.repository.ts
    ├── dto/
    │   ├── create-user.dto.ts
    │   └── update-user.dto.ts
    └── entities/
        └── user.entity.ts
```

---

## Module

```typescript
// users/users.module.ts
import { Module } from '@nestjs/common'
import { TypeOrmModule } from '@nestjs/typeorm'
import { UsersController } from './users.controller'
import { UsersService } from './users.service'
import { User } from './entities/user.entity'

@Module({
  imports: [TypeOrmModule.forFeature([User])],  // Import entity
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService]  // Export nếu module khác cần dùng
})
export class UsersModule {}
```

---

## Controller

```typescript
// users/users.controller.ts
import {
  Controller, Get, Post, Put, Delete, Body, Param, Query,
  HttpCode, HttpStatus, UseGuards, ParseIntPipe, ValidationPipe
} from '@nestjs/common'
import { UsersService } from './users.service'
import { CreateUserDto } from './dto/create-user.dto'
import { UpdateUserDto } from './dto/update-user.dto'
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard'
import { RolesGuard } from '../common/guards/roles.guard'
import { Roles } from '../common/decorators/roles.decorator'

@Controller('users')
@UseGuards(JwtAuthGuard)  // apply guard to all routes in controller
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(@Query('page') page = 1, @Query('limit') limit = 10) {
    return this.usersService.findAll({ page: +page, limit: +limit })
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id)
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body(ValidationPipe) createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto)
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body(ValidationPipe) updateUserDto: UpdateUserDto
  ) {
    return this.usersService.update(id, updateUserDto)
  }

  @Delete(':id')
  @UseGuards(RolesGuard)
  @Roles('admin')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.remove(id)
  }
}
```

---

## Service

```typescript
// users/users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common'
import { InjectRepository } from '@nestjs/typeorm'
import { Repository } from 'typeorm'
import * as bcrypt from 'bcrypt'
import { User } from './entities/user.entity'
import { CreateUserDto } from './dto/create-user.dto'

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>
  ) {}

  async findAll({ page, limit }: { page: number; limit: number }) {
    const [users, total] = await this.userRepository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' }
    })
    return { data: users, total, page, limit }
  }

  async findOne(id: number): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } })
    if (!user) throw new NotFoundException(`User ${id} not found`)
    return user
  }

  async create(dto: CreateUserDto): Promise<User> {
    const existing = await this.userRepository.findOne({ where: { email: dto.email } })
    if (existing) throw new ConflictException('Email already registered')

    const passwordHash = await bcrypt.hash(dto.password, 12)
    const user = this.userRepository.create({ ...dto, password: passwordHash })
    return this.userRepository.save(user)
  }

  async update(id: number, dto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id)
    Object.assign(user, dto)
    return this.userRepository.save(user)
  }

  async remove(id: number): Promise<void> {
    const user = await this.findOne(id)
    await this.userRepository.remove(user)
  }
}
```

---

## DTO với class-validator

```typescript
// users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, IsOptional, IsInt, Min, Max } from 'class-validator'
import { Transform } from 'class-transformer'

export class CreateUserDto {
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  name: string

  @IsEmail()
  @Transform(({ value }) => value.toLowerCase())  // normalize
  email: string

  @IsString()
  @MinLength(8)
  @MaxLength(100)
  password: string

  @IsOptional()
  @IsInt()
  @Min(18)
  @Max(150)
  age?: number
}
```

---

## Guard — Authentication

```typescript
// auth/guards/jwt-auth.guard.ts
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common'
import { JwtService } from '@nestjs/jwt'
import { Request } from 'express'

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest<Request>()
    const token = this.extractToken(request)

    if (!token) throw new UnauthorizedException('No token provided')

    try {
      const payload = await this.jwtService.verifyAsync(token)
      request['user'] = payload
      return true
    } catch {
      throw new UnauthorizedException('Invalid or expired token')
    }
  }

  private extractToken(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? []
    return type === 'Bearer' ? token : undefined
  }
}
```

---

## Interceptor — Logging & Transform

```typescript
// common/interceptors/logging.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common'
import { Observable, tap } from 'rxjs'

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const req = context.switchToHttp().getRequest()
    const startTime = Date.now()

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - startTime
        console.log(`${req.method} ${req.url} - ${duration}ms`)
      })
    )
  }
}
```

---

## Main.ts — Bootstrap

```typescript
// main.ts
import { NestFactory } from '@nestjs/core'
import { ValidationPipe } from '@nestjs/common'
import { AppModule } from './app.module'

async function bootstrap() {
  const app = await NestFactory.create(AppModule)

  // Global validation pipe
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,       // strip unknown fields
    forbidNonWhitelisted: true,  // throw if unknown fields present
    transform: true        // auto-transform types
  }))

  app.setGlobalPrefix('api')
  app.enableCors()

  await app.listen(process.env.PORT || 3000)
}

bootstrap()
```

---

## Bài tập thực hành

1. **NestJS project**: Tạo project mới bằng `nest new`, implement `UsersModule` với CRUD. Dùng TypeORM + SQLite (dev) hoặc PostgreSQL.

2. **Auth module**: Implement authentication với Passport.js + JWT strategy — register, login, protected routes, refresh tokens.

3. **Exception filters**: Tạo global `HttpExceptionFilter` và `PrismaExceptionFilter` — handle Prisma unique constraint, not found errors thành proper HTTP responses.

4. **Testing**: Viết unit tests cho `UsersService` (mock repository), integration tests cho `UsersController` dùng `@nestjs/testing`.

---

## Resources

- [docs.nestjs.com](https://docs.nestjs.com) — Official NestJS documentation
- [NestJS — First Steps](https://docs.nestjs.com/first-steps) — Getting started
- [NestJS — Testing](https://docs.nestjs.com/fundamentals/testing) — Unit & integration testing
