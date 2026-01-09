---
title: Error Handling Patterns
category: Error Management
difficulty: intermediate
languages:
  - typescript
  - python
  - java
  - csharp
  - php
  - kotlin
  - swift
  - dart
tags:
  - errors
  - exceptions
  - error-boundaries
  - result-types
  - try-catch
updated: 2026-01-09
---

# Error Handling Patterns

> **Purpose**: Consistent error handling across all languages
>
> **When to use**: API errors, validation failures, external service failures, unexpected conditions

---

## TypeScript / JavaScript

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native try-catch** | Built-in | No install | Standard error handling |
| **Result types** | `neverthrow` | `npm install neverthrow` | Functional error handling |
| **Error boundaries (React)** | Built-in React | No install | React component errors |

### Native Try-Catch

```typescript
// Basic try-catch
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  if (error instanceof ValidationError) {
    throw new BadRequestError(error.message);
  }
  if (error instanceof DatabaseError) {
    throw new ServiceUnavailableError('Database connection failed');
  }
  // Log unknown errors
  logger.error('Unexpected error', { error, context: { userId, action } });
  throw error; // Re-throw unknown errors
}
```

### Custom Error Classes

```typescript
class AppError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public isOperational = true
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(404, `${resource} not found`);
  }
}

class ValidationError extends AppError {
  constructor(message: string, public fields?: Record<string, string>) {
    super(400, message);
  }
}

// Usage
throw new NotFoundError('User');
throw new ValidationError('Invalid input', { email: 'Invalid format' });
```

### Result Type (neverthrow)

```typescript
// Install: npm install neverthrow
import { Result, ok, err } from 'neverthrow';

function fetchUser(id: string): Result<User, Error> {
  try {
    const user = database.getUser(id);
    return ok(user);
  } catch (error) {
    return err(new Error('User not found'));
  }
}

// Usage
const result = fetchUser('123');
result.match(
  (user) => console.log(user),
  (error) => console.error(error)
);
```

### Error Boundary (React)

```typescript
// No installation needed - built into React
import React from 'react';

class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error: Error | null }
> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    logger.error('React Error:', { error, errorInfo });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

---

## Python

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native try-except** | Built-in | No install | Standard error handling |
| **Result type** | `result` | `pip install result` | Functional error handling |

### Native Try-Except

```python
# Basic try-except
try:
    result = risky_operation()
    return result
except ValidationError as e:
    raise BadRequestError(str(e))
except DatabaseError as e:
    logger.error(f"Database error: {e}")
    raise ServiceUnavailableError("Database connection failed")
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    raise
```

### Custom Exceptions

```python
class AppError(Exception):
    """Base exception for application errors"""
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

class NotFoundError(AppError):
    def __init__(self, resource: str):
        super().__init__(f"{resource} not found", status_code=404)

class ValidationError(AppError):
    def __init__(self, message: str, fields: dict = None):
        self.fields = fields or {}
        super().__init__(message, status_code=400)

# Usage
raise NotFoundError("User")
raise ValidationError("Invalid input", {"email": "Invalid format"})
```

### Context Manager

```python
from contextlib import contextmanager

@contextmanager
def handle_db_errors():
    try:
        yield
    except IntegrityError:
        raise ValidationError("Duplicate entry")
    except OperationalError:
        raise ServiceUnavailableError("Database unavailable")
    finally:
        # Cleanup code here
        pass

# Usage
with handle_db_errors():
    db.session.add(user)
    db.session.commit()
```

### Result Type (result library)

```python
# Install: pip install result
from result import Result, Ok, Err

def fetch_user(user_id: str) -> Result[User, str]:
    try:
        user = database.get_user(user_id)
        return Ok(user)
    except Exception as e:
        return Err(f"User not found: {str(e)}")

# Usage
result = fetch_user("123")
if result.is_ok():
    user = result.ok_value
else:
    error = result.err_value
```

---

## Java

### 📦 Dependencies

| Approach | Library | Maven/Gradle | Use Case |
|----------|---------|--------------|----------|
| **Native try-catch** | Built-in | No install | Standard error handling |
| **Vavr (Result)** | `io.vavr:vavr` | See below | Functional error handling |

### Native Try-Catch

```java
// Basic try-catch
try {
    Result result = riskyOperation();
    return result;
} catch (ValidationException e) {
    throw new BadRequestException(e.getMessage());
} catch (SQLException e) {
    logger.error("Database error", e);
    throw new ServiceUnavailableException("Database connection failed");
} catch (Exception e) {
    logger.error("Unexpected error", e);
    throw new InternalServerErrorException(e);
}
```

### Custom Exceptions

```java
public class AppException extends RuntimeException {
    private final int statusCode;
    private final boolean isOperational;

    public AppException(String message, int statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;
    }

    public int getStatusCode() {
        return statusCode;
    }
}

public class NotFoundException extends AppException {
    public NotFoundException(String resource) {
        super(resource + " not found", 404);
    }
}

public class ValidationException extends AppException {
    private final Map<String, String> fields;

    public ValidationException(String message, Map<String, String> fields) {
        super(message, 400);
        this.fields = fields;
    }
}
```

### Try-with-Resources (Auto-close)

```java
// Automatically closes resources
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    
    ResultSet rs = stmt.executeQuery();
    return processResults(rs);
    
} catch (SQLException e) {
    logger.error("Database error", e);
    throw new DatabaseException("Query failed", e);
}
```

### Vavr (Result Type)

```xml
<!-- Maven: pom.xml -->
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.4</version>
</dependency>
```

```java
import io.vavr.control.Try;
import io.vavr.control.Either;

// Try monad
Try<User> result = Try.of(() -> database.getUser(userId));
result.onSuccess(user -> System.out.println(user))
      .onFailure(error -> logger.error("Error", error));

// Either type
Either<String, User> result = fetchUser(userId);
result.fold(
    error -> handleError(error),
    user -> handleSuccess(user)
);
```

---

## C# (.NET)

### 📦 Dependencies

| Approach | Library | NuGet | Use Case |
|----------|---------|-------|----------|
| **Native try-catch** | Built-in | No install | Standard error handling |
| **LanguageExt (Result)** | `LanguageExt.Core` | `dotnet add package LanguageExt.Core` | Functional error handling |

### Native Try-Catch

```csharp
// Basic try-catch
try
{
    var result = await RiskyOperationAsync();
    return result;
}
catch (ValidationException ex)
{
    throw new BadRequestException(ex.Message);
}
catch (DbException ex)
{
    _logger.LogError(ex, "Database error");
    throw new ServiceUnavailableException("Database connection failed");
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error");
    throw;
}
```

### Custom Exceptions

```csharp
public class AppException : Exception
{
    public int StatusCode { get; }
    public bool IsOperational { get; }

    public AppException(string message, int statusCode = 500, bool isOperational = true)
        : base(message)
    {
        StatusCode = statusCode;
        IsOperational = isOperational;
    }
}

public class NotFoundException : AppException
{
    public NotFoundException(string resource)
        : base($"{resource} not found", 404)
    {
    }
}

public class ValidationException : AppException
{
    public Dictionary<string, string> Fields { get; }

    public ValidationException(string message, Dictionary<string, string> fields = null)
        : base(message, 400)
    {
        Fields = fields ?? new Dictionary<string, string>();
    }
}
```

### Using Statement (Auto-dispose)

```csharp
// Automatically disposes resources
using (var connection = new SqlConnection(connectionString))
using (var command = new SqlCommand(sql, connection))
{
    try
    {
        await connection.OpenAsync();
        var result = await command.ExecuteReaderAsync();
        return ProcessResults(result);
    }
    catch (SqlException ex)
    {
        _logger.LogError(ex, "Database error");
        throw new DatabaseException("Query failed", ex);
    }
}

// Or with C# 8+ using declaration
using var connection = new SqlConnection(connectionString);
// Connection disposed at end of scope
```

### Result Type (LanguageExt)

```bash
# Install
dotnet add package LanguageExt.Core
```

```csharp
using LanguageExt;
using static LanguageExt.Prelude;

// Either type
Either<string, User> FetchUser(string id)
{
    try
    {
        var user = _database.GetUser(id);
        return Right<string, User>(user);
    }
    catch (Exception ex)
    {
        return Left<string, User>($"User not found: {ex.Message}");
    }
}

// Usage
var result = FetchUser("123");
result.Match(
    Left: error => HandleError(error),
    Right: user => HandleSuccess(user)
);
```

---

## PHP

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native try-catch** | Built-in | No install | Standard error handling |
| **Laravel exceptions** | Built-in Laravel | `composer require laravel/framework` | Laravel projects |

### Native Try-Catch

```php
<?php
// Basic try-catch
try {
    $result = riskyOperation();
    return $result;
} catch (ValidationException $e) {
    throw new BadRequestException($e->getMessage());
} catch (PDOException $e) {
    $logger->error('Database error: ' . $e->getMessage());
    throw new ServiceUnavailableException('Database connection failed');
} catch (Exception $e) {
    $logger->error('Unexpected error: ' . $e->getMessage());
    throw $e;
}
```

### Custom Exceptions

```php
<?php
class AppException extends Exception
{
    protected int $statusCode;
    protected bool $isOperational;

    public function __construct(
        string $message,
        int $statusCode = 500,
        bool $isOperational = true
    ) {
        parent::__construct($message);
        $this->statusCode = $statusCode;
        $this->isOperational = $isOperational;
    }

    public function getStatusCode(): int
    {
        return $this->statusCode;
    }
}

class NotFoundException extends AppException
{
    public function __construct(string $resource)
    {
        parent::__construct("$resource not found", 404);
    }
}

class ValidationException extends AppException
{
    private array $fields;

    public function __construct(string $message, array $fields = [])
    {
        parent::__construct($message, 400);
        $this->fields = $fields;
    }

    public function getFields(): array
    {
        return $this->fields;
    }
}
```

### Laravel Exception Handling

```php
<?php
// Laravel: app/Exceptions/Handler.php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    public function register()
    {
        $this->renderable(function (NotFoundException $e, $request) {
            return response()->json([
                'message' => $e->getMessage()
            ], 404);
        });

        $this->renderable(function (ValidationException $e, $request) {
            return response()->json([
                'message' => $e->getMessage(),
                'errors' => $e->getFields()
            ], 400);
        });
    }
}

// Usage in controller
throw new NotFoundException('User');
```

---

## Kotlin

### 📦 Dependencies

| Approach | Library | Gradle | Use Case |
|----------|---------|--------|----------|
| **Native try-catch** | Built-in | No install | Standard error handling |
| **Result type** | Built-in Kotlin | No install | Functional error handling |
| **Arrow (Either)** | `io.arrow-kt:arrow-core` | See below | Advanced functional programming |

### Native Try-Catch

```kotlin
// Basic try-catch
try {
    val result = riskyOperation()
    return result
} catch (e: ValidationException) {
    throw BadRequestException(e.message)
} catch (e: SQLException) {
    logger.error("Database error", e)
    throw ServiceUnavailableException("Database connection failed")
} catch (e: Exception) {
    logger.error("Unexpected error", e)
    throw InternalServerErrorException(e)
}
```

### Custom Exceptions

```kotlin
open class AppException(
    message: String,
    val statusCode: Int = 500,
    val isOperational: Boolean = true
) : RuntimeException(message)

class NotFoundException(resource: String) : AppException(
    message = "$resource not found",
    statusCode = 404
)

class ValidationException(
    message: String,
    val fields: Map<String, String> = emptyMap()
) : AppException(message, statusCode = 400)

// Usage
throw NotFoundException("User")
throw ValidationException("Invalid input", mapOf("email" to "Invalid format"))
```

### Result Type (Built-in Kotlin)

```kotlin
// Built into Kotlin - no installation needed
fun fetchUser(id: String): Result<User> {
    return try {
        val user = database.getUser(id)
        Result.success(user)
    } catch (e: Exception) {
        Result.failure(e)
    }
}

// Usage
when (val result = fetchUser("123")) {
    is Result.Success -> println(result.data)
    is Result.Failure -> handleError(result.exception)
}

// Or with fold
result.fold(
    onSuccess = { user -> handleSuccess(user) },
    onFailure = { error -> handleError(error) }
)
```

### Arrow (Either Type)

```kotlin
// build.gradle.kts
dependencies {
    implementation("io.arrow-kt:arrow-core:1.2.0")
}
```

```kotlin
import arrow.core.Either
import arrow.core.left
import arrow.core.right

sealed class ValidationError {
    data class InvalidEmail(val email: String) : ValidationError()
    data class InvalidAge(val age: Int) : ValidationError()
}

fun validateEmail(email: String): Either<ValidationError, String> {
    return if (email.contains("@")) {
        email.right()
    } else {
        ValidationError.InvalidEmail(email).left()
    }
}

// Usage
validateEmail("test@example.com").fold(
    { error -> handleError(error) },
    { email -> handleSuccess(email) }
)
```

---

## Swift

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native do-catch** | Built-in | No install | Standard error handling |
| **Result type** | Built-in Swift | No install | Functional error handling |

### Native Do-Catch

```swift
// Basic do-catch
do {
    let result = try riskyOperation()
    return result
} catch let error as ValidationError {
    throw BadRequestError(error.localizedDescription)
} catch let error as DatabaseError {
    logger.error("Database error: \(error)")
    throw ServiceUnavailableError("Database connection failed")
} catch {
    logger.error("Unexpected error: \(error)")
    throw error
}
```

### Custom Errors (Enum)

```swift
enum AppError: Error {
    case notFound(resource: String)
    case validation(message: String, fields: [String: String]?)
    case serviceUnavailable(message: String)
    case unauthorized
    case internalError(Error)
    
    var statusCode: Int {
        switch self {
        case .notFound: return 404
        case .validation: return 400
        case .serviceUnavailable: return 503
        case .unauthorized: return 401
        case .internalError: return 500
        }
    }
    
    var message: String {
        switch self {
        case .notFound(let resource):
            return "\(resource) not found"
        case .validation(let message, _):
            return message
        case .serviceUnavailable(let message):
            return message
        case .unauthorized:
            return "Unauthorized"
        case .internalError(let error):
            return error.localizedDescription
        }
    }
}

// Usage
throw AppError.notFound(resource: "User")
throw AppError.validation(message: "Invalid input", fields: ["email": "Invalid format"])
```

### Result Type (Built-in Swift)

```swift
// Built into Swift - no installation needed
func fetchUser(id: String) -> Result<User, Error> {
    do {
        let user = try database.getUser(id)
        return .success(user)
    } catch {
        return .failure(error)
    }
}

// Usage
switch fetchUser(id: "123") {
case .success(let user):
    handleSuccess(user)
case .failure(let error):
    handleError(error)
}

// Or with map/flatMap
fetchUser(id: "123")
    .map { user in
        // Transform user
    }
    .mapError { error in
        // Transform error
    }
```

---

## Dart (Flutter)

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native try-catch** | Built-in | No install | Standard error handling |
| **Either type** | `dartz` | `flutter pub add dartz` | Functional error handling |

### Native Try-Catch

```dart
// Basic try-catch
try {
  final result = await riskyOperation();
  return result;
} on ValidationException catch (e) {
  throw BadRequestException(e.message);
} on DatabaseException catch (e) {
  logger.error('Database error: $e');
  throw ServiceUnavailableException('Database connection failed');
} catch (e, stackTrace) {
  logger.error('Unexpected error: $e\n$stackTrace');
  rethrow;
}
```

### Custom Exceptions

```dart
class AppException implements Exception {
  final String message;
  final int statusCode;
  final bool isOperational;

  AppException(
    this.message, {
    this.statusCode = 500,
    this.isOperational = true,
  });

  @override
  String toString() => 'AppException: $message (code: $statusCode)';
}

class NotFoundException extends AppException {
  NotFoundException(String resource)
      : super('$resource not found', statusCode: 404);
}

class ValidationException extends AppException {
  final Map<String, String>? fields;

  ValidationException(String message, {this.fields})
      : super(message, statusCode: 400);
}

// Usage
throw NotFoundException('User');
throw ValidationException('Invalid input', fields: {'email': 'Invalid format'});
```

### Either Type (dartz)

```dart
// Install: flutter pub add dartz
import 'package:dartz/dartz.dart';

abstract class Failure {
  String get message;
}

class NotFoundFailure extends Failure {
  final String resource;
  NotFoundFailure(this.resource);
  
  @override
  String get message => '$resource not found';
}

class ServerFailure extends Failure {
  @override
  String get message => 'Server error';
}

// Function returning Either
Future<Either<Failure, User>> fetchUser(String id) async {
  try {
    final user = await database.getUser(id);
    return Right(user);
  } on NotFoundException catch (e) {
    return Left(NotFoundFailure(e.message));
  } catch (e) {
    return Left(ServerFailure());
  }
}

// Usage
final result = await fetchUser('123');
result.fold(
  (failure) => handleError(failure),
  (user) => displayUser(user),
);
```

---

## Best Practices

✅ **DO**:
- Always log errors with sufficient context (user ID, request ID, timestamp)
- Use typed errors (custom classes/enums) for different scenarios
- Fail fast - validate early, throw immediately
- Wrap third-party errors in your own error types
- Never expose sensitive data in error messages (passwords, tokens, internal paths)
- Provide actionable messages - tell users what to do next
- Monitor error rates - set up alerts for spikes
- Test error paths - don't just test happy paths

❌ **DON'T**:
- Catch exceptions without logging
- Return generic "Error occurred" messages
- Swallow errors silently (`catch (e) {}`)
- Use exceptions for flow control
- Expose stack traces to users in production
- Log passwords or tokens in error messages
- Ignore error types - catch specific exceptions first

---

## Security Checklist

- [ ] Error messages don't expose internal system details
- [ ] Stack traces not shown to end users in production
- [ ] Sensitive data (passwords, tokens) never logged
- [ ] Error rates monitored for anomalies
- [ ] Generic errors for authentication failures (don't reveal user exists)
- [ ] Input validation errors don't expose validation logic
- [ ] Database errors don't expose schema information

---

## Quick Decision Tree

```
Error occurs
    ↓
Can you recover? ──YES──→ Handle and continue
    ↓ NO
Is it expected? ──YES──→ Throw custom error with context
    ↓ NO
Is it operational? ──YES──→ Log + throw with retry info
    ↓ NO
Programming bug ────────→ Log + throw + alert developers
```

---

## Error Response Format (APIs)

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User not found",
    "statusCode": 404,
    "timestamp": "2026-01-09T10:00:00Z",
    "path": "/api/users/123",
    "requestId": "abc-123-def",
    "fields": {
      "email": "Invalid email format"
    }
  }
}
```
