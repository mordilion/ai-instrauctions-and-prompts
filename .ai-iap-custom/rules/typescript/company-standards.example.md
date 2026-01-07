# TypeScript Company Standards

> **Scope**: This applies to all TypeScript projects at [Your Company]
> **Type**: Custom Rule (extends core TypeScript standards)

---

## Company-Specific TypeScript Standards

### Import Organization

> **ALWAYS** organize imports in this order:

1. External dependencies (npm packages)
2. Internal shared libraries (`@company/...`)
3. Relative imports (`./ or ../`)

```typescript
// External
import { Injectable } from '@nestjs/common';
import * as express from 'express';

// Internal shared
import { Logger } from '@company/logger';
import { Config } from '@company/config';

// Relative
import { UserService } from './user.service';
import { UserDto } from '../dto/user.dto';
```

---

## Naming Conventions

> **ALWAYS** follow company naming conventions:

| Type | Convention | Example |
|------|-----------|---------|
| API Endpoints | `/api/v1/resource-name` | `/api/v1/user-profiles` |
| Database Tables | `snake_case` | `user_profiles` |
| TypeScript Classes | `PascalCase` | `UserProfileService` |
| Environment Variables | `UPPER_SNAKE_CASE` | `DATABASE_URL` |

---

## Required File Headers

> **ALWAYS** include file header for non-trivial files:

```typescript
/**
 * @file user.service.ts
 * @description User management service
 * @author Your Name <your.name@company.com>
 * @created 2026-01-07
 * @team Platform Engineering
 */
```

---

## Logging Standards

> **ALWAYS** use company logger:

```typescript
import { Logger } from '@company/logger';

export class UserService {
  private readonly logger = new Logger(UserService.name);

  async createUser(data: CreateUserDto) {
    this.logger.info('Creating user', { email: data.email });
    
    try {
      const user = await this.userRepository.create(data);
      this.logger.info('User created successfully', { userId: user.id });
      return user;
    } catch (error) {
      this.logger.error('Failed to create user', { error, data });
      throw error;
    }
  }
}
```

---

## Error Handling

> **ALWAYS** use company error classes:

```typescript
import { ApplicationError, ValidationError } from '@company/errors';

// Validation errors
if (!user) {
  throw new ValidationError('User not found', { userId });
}

// Application errors
throw new ApplicationError('Database connection failed', {
  code: 'DB_CONNECTION_ERROR',
  retryable: true
});
```

---

## API Response Format

> **ALWAYS** use standard response format:

```typescript
// Success response
{
  "success": true,
  "data": { /* ... */ },
  "meta": {
    "timestamp": "2026-01-07T10:30:00Z",
    "requestId": "abc-123"
  }
}

// Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": { "field": "email" }
  },
  "meta": {
    "timestamp": "2026-01-07T10:30:00Z",
    "requestId": "abc-123"
  }
}
```

---

## Database Migrations

> **ALWAYS** follow migration naming:

```
YYYYMMDDHHMMSS_descriptive_name.ts
20260107103000_add_user_profiles_table.ts
```

---

## Testing Standards

> **ALWAYS** include tests for:

- ✅ All public methods
- ✅ Error handling paths
- ✅ Edge cases (null, undefined, empty)
- ✅ Integration with external services

---

## AI Self-Check

Before marking code as complete, verify:

- [ ] File header present with correct team and author
- [ ] Imports organized (external → internal → relative)
- [ ] Company logger used (not console.log)
- [ ] Company error classes used
- [ ] API responses follow standard format
- [ ] Database queries use connection pool
- [ ] Sensitive data not logged
- [ ] Tests cover all public methods
- [ ] Migration follows naming convention
- [ ] No hardcoded secrets or credentials
- [ ] Code reviewed by team lead
- [ ] Documentation updated in Confluence

---

## Internal Resources

- **Confluence**: [Company TypeScript Standards](https://wiki.company.com/typescript)
- **Slack**: #platform-engineering
- **Repository**: [Company TypeScript Templates](https://github.com/company/typescript-templates)
