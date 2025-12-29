# NestJS Framework

> **Scope**: Apply these rules when working with NestJS applications
> **Applies to**: TypeScript files in NestJS projects
> **Extends**: typescript/architecture.md, typescript/code-style.md
> **Precedence**: Framework rules OVERRIDE TypeScript rules for NestJS-specific patterns

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use dependency injection (constructor injection required)
> **ALWAYS**: Use DTOs with class-validator for validation
> **ALWAYS**: Use Guards for authentication/authorization (NOT middleware)
> **ALWAYS**: Use Interceptors for response transformation (NOT manual)
> **ALWAYS**: Return exceptions via built-in exception filters
> 
> **NEVER**: Use `new` keyword for service instantiation (breaks DI)
> **NEVER**: Skip DTO validation (security risk)
> **NEVER**: Put business logic in controllers (use services)
> **NEVER**: Use Express middleware for auth (use Guards)
> **NEVER**: Return inconsistent response shapes (use Interceptors)

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| Controllers | HTTP endpoints | `@Controller()`, `@Get()`, `@Post()` |
| Services | Business logic | `@Injectable()`, injected via constructor |
| Guards | Auth, authorization | `@UseGuards()`, `CanActivate` |
| Interceptors | Response transformation | `@UseInterceptors()`, `NestInterceptor` |
| Pipes | Validation, transformation | `@UsePipes()`, `ValidationPipe` |
| DTOs | Request/response shapes | `class-validator`, `class-transformer` |

## Core Patterns

### Controller (Thin)
```typescript
import { Controller, Get, Post, Body, Param, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { CreateUserDto } from './dto/create-user.dto';
import { UserService } from './user.service';

@Controller('users')
@UseGuards(JwtAuthGuard)  // Apply guard to all routes
export class UserController {
  constructor(private readonly userService: UserService) {}  // DI
  
  @Get()
  findAll() {
    return this.userService.findAll();
  }
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id);
  }
  
  @Post()
  create(@Body() createUserDto: CreateUserDto) {  // DTO validation automatic
    return this.userService.create(createUserDto);
  }
}
```

### Service (Business Logic)
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}
  
  findAll(): Promise<User[]> {
    return this.userRepository.find();
  }
  
  async findOne(id: number): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) throw new NotFoundException(`User #${id} not found`);
    return user;
  }
  
  create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
}
```

### DTO with Validation
```typescript
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  name: string;
  
  @IsEmail()
  email: string;
  
  @IsString()
  @MinLength(8)
  password: string;
}
```

### Guard (Authentication)
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}
  
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) return false;
    
    try {
      const payload = this.jwtService.verify(token);
      request.user = payload;
      return true;
    } catch {
      return false;
    }
  }
}
```

### Module (Dependency Registration)
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { User } from './entities/user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService],  // Export for use in other modules
})
export class UserModule {}
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Manual Instantiation** | `new UserService()` | Inject via constructor | Breaks DI, no dependency resolution |
| **No DTO Validation** | Plain interfaces for requests | DTO classes with `class-validator` | Security vulnerability, no validation |
| **Business Logic in Controller** | Controller does DB queries | Service handles logic | Untestable, unmaintainable |
| **Middleware for Auth** | Express middleware | `@UseGuards(JwtAuthGuard)` | Not integrated with NestJS lifecycle |
| **Manual Exception Handling** | try-catch everywhere | Throw built-in exceptions | Inconsistent error responses |

### Anti-Pattern: Manual Instantiation (BREAKS DI)
```typescript
// ❌ WRONG - Manual instantiation
@Controller('users')
export class UserController {
  private userService = new UserService();  // BREAKS DI!
  
  @Get()
  findAll() {
    return this.userService.findAll();
  }
}

// ✅ CORRECT - Constructor injection
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}  // DI
  
  @Get()
  findAll() {
    return this.userService.findAll();
  }
}
```

### Anti-Pattern: No DTO Validation (SECURITY RISK)
```typescript
// ❌ WRONG - Interface, no validation
interface CreateUserDto {
  name: string;
  email: string;
}

@Post()
create(@Body() dto: CreateUserDto) {  // NO VALIDATION!
  return this.userService.create(dto);
}

// ✅ CORRECT - DTO class with validation
export class CreateUserDto {
  @IsString() @MinLength(2) name: string;
  @IsEmail() email: string;
}

@Post()
create(@Body() dto: CreateUserDto) {  // Automatic validation
  return this.userService.create(dto);
}
```

## AI Self-Check (Verify BEFORE generating NestJS code)

- [ ] Using dependency injection? (Constructor injection)
- [ ] DTOs with class-validator decorators?
- [ ] Controllers only call services? (No business logic)
- [ ] Guards for authentication/authorization?
- [ ] Throwing built-in exceptions? (NotFoundException, BadRequestException)
- [ ] Module imports/providers/exports correct?
- [ ] ValidationPipe enabled globally?
- [ ] Using TypeORM/Prisma decorators correctly?
- [ ] Interceptors for response transformation?
- [ ] Following NestJS conventions?

## Module Structure

```
src/
├── app.module.ts       # Root module
├── main.ts             # Bootstrap
├── users/
│   ├── user.module.ts
│   ├── user.controller.ts
│   ├── user.service.ts
│   ├── entities/
│   │   └── user.entity.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   └── guards/
│       └── jwt-auth.guard.ts
└── common/
    ├── filters/
    ├── interceptors/
    └── pipes/
```

## Global Setup

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,  // Strip unknown properties
    forbidNonWhitelisted: true,  // Throw on unknown properties
    transform: true,  // Auto-transform to DTO types
  }));
  
  await app.listen(3000);
}
bootstrap();
```

## Key Decorators

| Decorator | Purpose | Example |
|-----------|---------|---------|
| @Controller | Define controller | `@Controller('users')` |
| @Injectable | Mark as injectable | `@Injectable()` |
| @Get/@Post/@Put/@Delete | HTTP methods | `@Get(':id')` |
| @Body/@Param/@Query | Extract request data | `@Body() dto: CreateDto` |
| @UseGuards | Apply guard | `@UseGuards(AuthGuard)` |
| @UseInterceptors | Apply interceptor | `@UseInterceptors(TransformInterceptor)` |

## Key Libraries

- **@nestjs/typeorm**: TypeORM integration
- **@nestjs/jwt**: JWT authentication
- **class-validator**: DTO validation
- **class-transformer**: DTO transformation
