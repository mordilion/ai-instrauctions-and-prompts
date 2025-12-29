# AdonisJS MVC/Traditional Structure

> **Scope**: AdonisJS apps organized by technical layer (MVC pattern)
> **Precedence**: Overrides default AdonisJS folder organization
> **Use When**: Small-medium apps (<10 entities), simple domain logic, traditional MVC preference

## CRITICAL REQUIREMENTS

> **ALWAYS**: Organize by technical layer (controllers/, models/, services/)
> **ALWAYS**: Use dependency injection with `@inject()`
> **ALWAYS**: Validate in controllers before service calls
> **ALWAYS**: Keep business logic in services (NOT controllers)
> **ALWAYS**: Use path aliases (`#controllers`, `#models`, etc.)
> 
> **NEVER**: Put business logic in controllers
> **NEVER**: Access models directly from controllers (use services)
> **NEVER**: Skip validation in controllers
> **NEVER**: Use relative imports (use path aliases)

## Project Structure

```
app/
├── controllers/       # HTTP request handlers
├── models/           # Lucid ORM models
├── services/         # Business logic
├── validators/       # VineJS validators
├── middleware/       # HTTP middleware
├── exceptions/       # Custom exceptions
├── events/          # Domain events
├── listeners/       # Event handlers
└── utils/           # Utilities
config/              # Configuration
database/            # Migrations, seeders, factories
start/               # App bootstrap (routes.ts, kernel.ts, events.ts)
tests/
├── unit/            # Service/validator tests
└── functional/      # API integration tests
```

## Layer Organization

### Controllers → Services → Models

```typescript
// app/controllers/users_controller.ts
import { HttpContext } from '@adonisjs/core/http'
import { inject } from '@adonisjs/core'
import UserService from '#services/user_service'
import { createUserValidator } from '#validators/user_validator'

@inject()
export default class UsersController {
  constructor(protected userService: UserService) {}

  async store({ request, response }: HttpContext) {
    const payload = await request.validateUsing(createUserValidator)
    const user = await this.userService.create(payload)
    return response.created(user)
  }
}
```

### Services (Business Logic)

```typescript
// app/services/user_service.ts
import { inject } from '@adonisjs/core'
import User from '#models/user'
import hash from '@adonisjs/core/services/hash'
import emitter from '@adonisjs/core/services/emitter'

@inject()
export default class UserService {
  async create(data: { email: string; password: string; fullName: string }) {
    const hashedPassword = await hash.make(data.password)
    const user = await User.create({ ...data, password: hashedPassword })
    emitter.emit(new UserRegistered(user))
    return user
  }
}
```

### Models (Data Layer)

```typescript
// app/models/user.ts
import { BaseModel, column, hasMany } from '@adonisjs/lucid/orm'

export default class User extends BaseModel {
  @column({ isPrimary: true })
  declare id: number

  @column({ serializeAs: null })
  declare password: string

  @hasMany(() => Post)
  declare posts: HasMany<typeof Post>
}
```

## Routes (start/routes.ts)

```typescript
import router from '@adonisjs/core/services/router'
import { middleware } from '#start/kernel'

const UsersController = () => import('#controllers/users_controller')

// Protected routes
router.group(() => {
  router.resource('users', UsersController).apiOnly()
}).prefix('/api/v1').middleware(middleware.auth())
```

## Events & Listeners

```typescript
// app/events/user_registered.ts
export default class UserRegistered extends BaseEvent {
  constructor(public user: User) { super() }
}

// app/listeners/send_welcome_email.ts
@inject()
export default class SendWelcomeEmail {
  async handle(event: UserRegistered) {
    await this.emailService.sendWelcome(event.user.email)
  }
}

// start/events.ts
emitter.on(UserRegistered, [SendWelcomeEmail])
```

## Path Aliases (tsconfig.json)

```json
{
  "compilerOptions": {
    "paths": {
      "#controllers/*": ["./app/controllers/*"],
      "#models/*": ["./app/models/*"],
      "#services/*": ["./app/services/*"],
      "#validators/*": ["./app/validators/*"]
    }
  }
}
```

## Rules

| Rule | Why |
|------|-----|
| **Layer Separation** | Controllers → Services → Models |
| **Single Responsibility** | One class per file |
| **DI via Constructor** | Use `@inject()` decorator |
| **Validate Early** | In controllers, before service calls |
| **Business Logic in Services** | Keep controllers thin |
| **Events for Cross-Cutting** | Decouple side effects |

## When to Use

- ✅ Small to medium apps (<10 entities)
- ✅ Simple domain logic
- ✅ Single team
- ✅ Quick prototyping
- ✅ Traditional MVC preference
- ❌ Complex domain (use Modular structure)

## Testing

```typescript
// tests/functional/users.spec.ts
import { test } from '@japa/runner'

test.group('Users API', () => {
  test('can create user', async ({ client }) => {
    const response = await client.post('/api/v1/users').json({
      email: 'test@example.com',
      password: 'password123',
      fullName: 'Test User'
    })
    
    response.assertStatus(201)
    response.assertBodyContains({ email: 'test@example.com' })
  })
})
```

## Key Features

- **Flat Structure**: All controllers in one folder
- **Clear Layers**: Controllers, Services, Models
- **Path Aliases**: Clean imports with `#`
- **IoC Container**: Automatic dependency injection
- **Lucid ORM**: Active Record pattern
- **VineJS**: Schema-based validation
- **Events**: Decoupled side effects
