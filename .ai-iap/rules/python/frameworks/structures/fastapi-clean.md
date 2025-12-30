# FastAPI Clean Architecture

> **Scope**: FastAPI with Clean Architecture (DDD-inspired)
> **Use When**: Complex domain, clear boundaries, high testability needs

## CRITICAL REQUIREMENTS

> **ALWAYS**: Separate domain, application, infrastructure, presentation layers
> **ALWAYS**: Domain entities are pure (no framework dependencies)
> **ALWAYS**: Use interfaces for external dependencies
> **ALWAYS**: Dependency rule: inner layers don't know outer layers
> 
> **NEVER**: Import infrastructure in domain
> **NEVER**: Put business logic in API layer
> **NEVER**: Skip repository interfaces

## Structure

```
src/
├── domain/              # Enterprise rules (pure Python)
│   ├── entities/       # Business objects
│   ├── repositories/   # Interfaces only
│   └── exceptions/
├── application/         # Use cases
│   ├── use_cases/
│   └── dto/
├── infrastructure/      # External interfaces
│   ├── database/
│   ├── services/
│   └── config/
└── presentation/        # API layer
    └── api/
```

## Core Patterns

### Domain Entity (Pure)

```python
# domain/entities/user.py
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: int | None
    email: str
    name: str
    created_at: datetime
    
    def change_email(self, new_email: str) -> None:
        if "@" not in new_email:
            raise ValueError("Invalid email")
        self.email = new_email
```

### Repository Interface

```python
# domain/repositories/user_repository.py
from abc import ABC, abstractmethod
from domain.entities.user import User

class UserRepository(ABC):
    @abstractmethod
    async def get_by_id(self, user_id: int) -> User | None:
        pass
    
    @abstractmethod
    async def save(self, user: User) -> User:
        pass
```

### Use Case

```python
# application/use_cases/users/create_user.py
from domain.entities.user import User
from domain.repositories.user_repository import UserRepository

class CreateUserUseCase:
    def __init__(self, user_repository: UserRepository):
        self.user_repository = user_repository
    
    async def execute(self, email: str, name: str) -> User:
        user = User(id=None, email=email, name=name, created_at=datetime.utcnow())
        return await self.user_repository.save(user)
```

### Repository Implementation

```python
# infrastructure/database/repositories/sqlalchemy_user_repository.py
from sqlalchemy.ext.asyncio import AsyncSession
from domain.repositories.user_repository import UserRepository
from domain.entities.user import User

class SQLAlchemyUserRepository(UserRepository):
    def __init__(self, session: AsyncSession):
        self.session = session
    
    async def get_by_id(self, user_id: int) -> User | None:
        result = await self.session.execute(
            select(UserModel).where(UserModel.id == user_id)
        )
        model = result.scalar_one_or_none()
        return model.to_entity() if model else None
    
    async def save(self, user: User) -> User:
        model = UserModel.from_entity(user)
        self.session.add(model)
        await self.session.commit()
        return model.to_entity()
```

### API Router

```python
# presentation/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends
from application.use_cases.users.create_user import CreateUserUseCase
from application.dto.user_dto import CreateUserRequest, UserResponse

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=201)
async def create_user(
    request: CreateUserRequest,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case)
):
    user = await use_case.execute(request.email, request.name)
    return UserResponse.from_entity(user)
```

## Common AI Mistakes

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| **Domain Depends on Infra** | Import SQLAlchemy in domain | Pure Python entities |
| **Logic in API** | Business rules in router | Use cases |
| **No Interfaces** | Direct DB access | Repository interface |
| **Wrong Direction** | Domain imports API | API imports domain |

## AI Self-Check

- [ ] Domain layer pure (no framework imports)?
- [ ] Repository interfaces in domain?
- [ ] Use cases in application layer?
- [ ] Dependency rule followed?
- [ ] DTOs for API boundary?
- [ ] No business logic in routers?
- [ ] Infrastructure implements interfaces?
- [ ] Testable without DB?

## Benefits

- ✅ Testable (mock repositories)
- ✅ Framework-independent business logic
- ✅ Clear boundaries
- ✅ Easy to change infrastructure

## When to Use

- ✅ Complex domain logic
- ✅ Long-term projects
- ✅ High testability requirements
- ❌ Simple CRUD apps (over-engineering)
