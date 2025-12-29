# FastAPI Framework

> **Scope**: Apply these rules when working with FastAPI applications
> **Applies to**: Python files in FastAPI projects
> **Extends**: python/architecture.md, python/code-style.md
> **Precedence**: Framework rules OVERRIDE Python rules for FastAPI-specific patterns

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use Pydantic models for request/response validation
> **ALWAYS**: Use async def for I/O operations (database, HTTP calls)
> **ALWAYS**: Use dependency injection for shared resources (NOT globals)
> **ALWAYS**: Use HTTPException for error responses (standard format)
> **ALWAYS**: Type hint all parameters and return types
> 
> **NEVER**: Use sync functions for I/O operations (blocks event loop)
> **NEVER**: Use global state (breaks async, testing)
> **NEVER**: Skip Pydantic validation (security risk)
> **NEVER**: Return dict without Pydantic model (breaks type safety)
> **NEVER**: Use mutable default arguments

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| async def | I/O operations (always) | `async def`, `await` |
| Pydantic Models | Request/response validation | `BaseModel`, `Field()` |
| Depends() | Dependency injection | `Depends()`, shared resources |
| APIRouter | Route organization | `APIRouter()`, `include_router()` |
| BackgroundTasks | Async background jobs | `BackgroundTasks`, `add_task()` |

## Core Patterns

### Async Endpoint with Pydantic
```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, EmailStr, Field

app = FastAPI()

class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)

class UserResponse(BaseModel):
    id: int
    name: str
    email: EmailStr
    
    class Config:
        from_attributes = True  # For ORM models

@app.post("/users", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate) -> UserResponse:
    # Async database call
    db_user = await user_service.create(user)
    return UserResponse.from_orm(db_user)

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int) -> UserResponse:
    user = await user_service.get_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return UserResponse.from_orm(user)
```

### Dependency Injection
```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncSession:
    async with SessionLocal() as session:
        yield session

@app.get("/users")
async def get_users(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(User))
    return result.scalars().all()
```

### Router Organization
```python
# routers/users.py
from fastapi import APIRouter, Depends

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/")
async def list_users():
    return await user_service.get_all()

@router.post("/")
async def create_user(user: UserCreate):
    return await user_service.create(user)

# main.py
from routers import users

app.include_router(users.router)
```

### Background Tasks
```python
from fastapi import BackgroundTasks

def send_email(email: str, message: str):
    # Send email logic
    pass

@app.post("/users")
async def create_user(
    user: UserCreate,
    background_tasks: BackgroundTasks
):
    db_user = await user_service.create(user)
    background_tasks.add_task(send_email, user.email, "Welcome!")
    return db_user
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Sync I/O** | `def` for DB/HTTP calls | `async def` + `await` | Blocks event loop, kills performance |
| **No Pydantic** | Plain dict or no validation | Pydantic `BaseModel` | Security risk, no type safety |
| **Global State** | `db = Database()` at module level | `Depends(get_db)` | Breaks async, untestable |
| **Dict Return** | Return `dict` without model | `response_model=UserResponse` | No validation, breaks docs |
| **Mutable Defaults** | `def func(items=[]):` | `def func(items=None):` | Shared mutable state bug |

### Anti-Pattern: Sync I/O (BLOCKS EVENT LOOP)
```python
# ❌ WRONG - Sync function blocks event loop
@app.get("/users")
def get_users():  # Sync def
    users = db.query(User).all()  # Blocking!
    return users

# ✅ CORRECT - Async function
@app.get("/users")
async def get_users():  # async def
    users = await db.execute(select(User))  # Non-blocking
    return users.scalars().all()
```

### Anti-Pattern: No Pydantic Validation (SECURITY RISK)
```python
# ❌ WRONG - No validation
@app.post("/users")
async def create_user(name: str, email: str):  # No validation!
    return await user_service.create(name, email)

# ✅ CORRECT - Pydantic validation
class UserCreate(BaseModel):
    name: str = Field(..., min_length=2)
    email: EmailStr

@app.post("/users")
async def create_user(user: UserCreate):  # Automatic validation
    return await user_service.create(user)
```

### Anti-Pattern: Global State (BREAKS ASYNC)
```python
# ❌ WRONG - Global state
db = Database()  # Shared across all requests!

@app.get("/users")
async def get_users():
    return await db.query(User).all()  # Not thread-safe

# ✅ CORRECT - Dependency injection
async def get_db():
    async with SessionLocal() as session:
        yield session

@app.get("/users")
async def get_users(db: AsyncSession = Depends(get_db)):
    return await db.execute(select(User))
```

## AI Self-Check (Verify BEFORE generating FastAPI code)

- [ ] Using async def for all I/O operations?
- [ ] Pydantic models for requests/responses?
- [ ] Dependency injection with Depends()?
- [ ] HTTPException for errors?
- [ ] Type hints on all parameters and returns?
- [ ] response_model specified for endpoints?
- [ ] No global state (database, clients)?
- [ ] No mutable default arguments?
- [ ] Using APIRouter for organization?
- [ ] Background tasks for async operations?

## Error Handling

```python
from fastapi import HTTPException, status

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    user = await user_service.get_by_id(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    return user
```

## Authentication

```python
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    token = credentials.credentials
    user = await auth_service.verify_token(token)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@app.get("/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

## Key Features

| Feature | Purpose | Keywords |
|---------|---------|----------|
| Auto Docs | OpenAPI/Swagger UI | `/docs`, `/redoc` |
| Validation | Automatic with Pydantic | `BaseModel`, `Field()` |
| Dependency Injection | Reusable dependencies | `Depends()` |
| Background Tasks | Async job processing | `BackgroundTasks` |
| WebSockets | Real-time communication | `WebSocket` |

## Key Libraries

- **Pydantic**: Data validation, `BaseModel`, `Field()`
- **SQLAlchemy**: ORM, async support
- **httpx**: Async HTTP client
- **python-multipart**: File uploads
- **python-jose**: JWT tokens
