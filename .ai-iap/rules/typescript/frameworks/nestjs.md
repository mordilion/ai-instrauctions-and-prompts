# NestJS Framework

> **Scope**: NestJS applications
> **Applies to**: TypeScript files in NestJS projects
> **Extends**: typescript/architecture.md, typescript/code-style.md

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use dependency injection (constructor)
> **ALWAYS**: Use DTOs with class-validator
> **ALWAYS**: Use Guards for auth/authorization
> **ALWAYS**: Use Interceptors for response transformation
> **ALWAYS**: Return exceptions via filters
> 
> **NEVER**: Use `new` for services (breaks DI)
> **NEVER**: Skip DTO validation
> **NEVER**: Put business logic in controllers
> **NEVER**: Use Express middleware for auth
> **NEVER**: Return inconsistent response shapes

## Core Patterns

### Controller (Thin)

```typescript
@Controller('users')
@UseGuards(JwtAuthGuard)
export class UserController {
  constructor(private readonly userService: UserService) {}
  
  @Get()
  findAll() {
    return this.userService.findAll()
  }
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.userService.create(dto)
  }
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id)
  }
}
```

### Service (Business Logic)

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.userRepository.find()
  }
  
  async create(dto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(dto)
    return this.userRepository.save(user)
  }
  
  async findOne(id: number): Promise<User> {
    const user = await this.userRepository.findOneBy({ id })
    if (!user) {
      throw new NotFoundException(`User ${id} not found`)
    }
    return user
  }
}
```

### DTO with Validation

```typescript
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator'

export class CreateUserDto {
  @IsEmail()
  email: string
  
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  name: string
  
  @IsString()
  @MinLength(8)
  password: string
}
```

### Guard (Auth)

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context)
  }
  
  handleRequest(err: any, user: any) {
    if (err || !user) {
      throw new UnauthorizedException()
    }
    return user
  }
}
```

### Interceptor (Transform Response)

```typescript
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        statusCode: context.switchToHttp().getResponse().statusCode,
        data,
        timestamp: new Date().toISOString()
      }))
    )
  }
}
```

### Exception Filter

```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp()
    const response = ctx.getResponse()
    const status = exception.getStatus()
    
    response.status(status).json({
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString()
    })
  }
}
```

### Module

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService]
})
export class UserModule {}
```

## Common AI Mistakes

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| **Manual Instantiation** | `new UserService()` | Constructor DI |
| **No Validation** | Plain objects | DTOs with class-validator |
| **Logic in Controller** | Business rules in controller | Move to service |
| **Middleware for Auth** | Express middleware | Guards |

### Anti-Pattern: No DI

```typescript
// ❌ WRONG
export class UserController {
  private userService = new UserService()  // Breaks DI!
}

// ✅ CORRECT
export class UserController {
  constructor(private readonly userService: UserService) {}
}
```

## AI Self-Check

- [ ] Constructor injection?
- [ ] DTOs with validation?
- [ ] Guards for auth?
- [ ] Interceptors for responses?
- [ ] Exception filters?
- [ ] Business logic in services?
- [ ] @Injectable() on services?
- [ ] Module imports/exports correct?
- [ ] No manual instantiation?
- [ ] Consistent response shapes?

## Key Decorators

| Decorator | Purpose |
|-----------|---------|
| `@Controller` | Define controller |
| `@Injectable` | Enable DI |
| `@UseGuards` | Apply guards |
| `@UseInterceptors` | Apply interceptors |
| `@Body/@Param/@Query` | Extract request data |

## Best Practices

**MUST**: DI, DTOs, Guards, Interceptors, Exception filters
**SHOULD**: ValidationPipe, TransformInterceptor, proper modules
**AVOID**: Manual instantiation, skipping validation, logic in controllers
