# PHP Code Style

## General Rules

- **PSR-12** coding standard
- **Strict types** enabled
- **Type hints** everywhere

## Naming Conventions

```php
// PascalCase for classes
class UserService {}

// camelCase for methods and properties
public function getUserById(int $id): ?User {}
private string $userName;

// UPPER_SNAKE_CASE for constants
const MAX_LOGIN_ATTEMPTS = 3;
```

## Type Declarations

```php
declare(strict_types=1);

// Explicit types for parameters and return
function getUser(int $id): User {
    return $repository->find($id);
}

// Nullable types
function findUser(int $id): ?User {
    return $repository->findOrNull($id);
}

// Union types (modern PHP)
function processValue(int|string $value): void {}
```

## Classes

```php
// Use readonly properties (modern PHP)
final readonly class User {
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
    ) {}
}

// Constructor property promotion
class UserService {
    public function __construct(
        private readonly UserRepository $repository,
        private readonly LoggerInterface $logger,
    ) {}
}
```

## Best Practices

```php
// Use match over switch (modern PHP)
$result = match ($status) {
    'pending' => handlePending(),
    'approved' => handleApproved(),
    default => handleDefault(),
};

// Named arguments
$user = createUser(
    name: 'John',
    email: 'john@test.com'
);

// Null coalescing
$name = $user->name ?? 'Anonymous';

// Nullsafe operator
$email = $user?->profile?->email;
```
