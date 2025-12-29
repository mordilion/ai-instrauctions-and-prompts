# Kotlin Code Style

> **Scope**: All Kotlin files
> **Applies to**: *.kt files

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use `val` for immutable (default)
> **ALWAYS**: Use data classes for DTOs
> **ALWAYS**: Use named arguments for 3+ parameters
> **ALWAYS**: Use expression body for single expressions
> **ALWAYS**: Use null-safety operators (`?.`, `?:`)
> 
> **NEVER**: Use `var` unless mutation required
> **NEVER**: Use `!!` without justification
> **NEVER**: Ignore null-safety
> **NEVER**: Use Java-style getters/setters

## Naming

```kotlin
class UserService                          // PascalCase
fun calculateTotal(): BigDecimal           // camelCase
val userName: String                       // camelCase
const val MAX_ATTEMPTS = 3                 // UPPER_SNAKE_CASE
```

## Variables

```kotlin
// ✅ Immutable
val name = "John"
val items = listOf(1, 2, 3)

// ✅ Mutable when needed
var count = 0
count++
```

## Data Classes

```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
)
```

## Functions

```kotlin
// Expression body
fun add(a: Int, b: Int) = a + b

// Block body
fun processUser(user: User): Result {
    validate(user)
    return save(user)
}

// Named arguments (3+ params)
createUser(
    name = "John",
    email = "john@example.com",
    age = 30
)
```

## Null Safety

```kotlin
// Safe call
val length = name?.length

// Elvis operator
val result = value ?: default

// Let
user?.let { process(it) }

// !! only when justified
val id = user.id!!  // Comment: Guaranteed non-null by validation
```

## When Expression

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
}

when (result) {
    is Result.Success -> handle(result.data)
    is Result.Error -> log(result.message)
}
```

## Collections

```kotlin
// Immutable
val users = listOf(user1, user2)
val names = mapOf("a" to 1, "b" to 2)

// Mutable
val items = mutableListOf<Item>()
items.add(item)

// Functional
users.filter { it.isActive }
     .map { it.name }
     .sorted()
```

## Extension Functions

```kotlin
fun String.toTitleCase() =
    lowercase().replaceFirstChar { it.uppercase() }

val title = "hello world".toTitleCase()  // "Hello world"
```

## Scope Functions

| Function | Usage | Returns |
|----------|-------|---------|
| `let` | Transform value | Lambda result |
| `apply` | Configure object | Object |
| `also` | Side effects | Object |
| `run` | Execute block | Lambda result |

```kotlin
user?.let { processUser(it) }
User().apply { name = "John" }
list.also { log(it.size) }
```

## Common AI Mistakes

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| **Mutable by default** | `var` everywhere | `val` by default |
| **Force unwrap** | `user!!` | `user?.let { }` |
| **No data class** | Manual equals/hashCode | `data class` |
| **Java style** | `getName()` | `name` property |

## AI Self-Check

- [ ] `val` by default?
- [ ] Data classes for DTOs?
- [ ] Named arguments (3+ params)?
- [ ] Expression body for single expressions?
- [ ] Null-safety operators?
- [ ] No unnecessary `var`?
- [ ] No unjustified `!!`?
- [ ] Properties not getters/setters?
- [ ] Extension functions where appropriate?

## Best Practices

**MUST**: `val` default, data classes, null-safety, named arguments
**SHOULD**: Expression body, extension functions, sealed classes
**AVOID**: `var` by default, `!!`, Java patterns, mutable collections
