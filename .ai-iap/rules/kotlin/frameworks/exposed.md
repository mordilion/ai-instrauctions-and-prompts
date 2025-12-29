# Exposed ORM Framework

> **Scope**: Lightweight SQL library for Kotlin with DSL and DAO APIs
> **Applies to**: Kotlin files using Exposed
> **Extends**: kotlin/architecture.md, kotlin/code-style.md

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use transactions for all database operations
> **ALWAYS**: Use suspend functions with `dbQuery { }` for async operations
> **ALWAYS**: Define tables as objects extending `Table` or `IntIdTable`
> **ALWAYS**: Use `DatabaseFactory.init()` to setup connection
> **ALWAYS**: Use HikariCP for connection pooling in production
> 
> **NEVER**: Perform database operations outside transactions
> **NEVER**: Block threads with synchronous queries (use `dbQuery`)
> **NEVER**: Skip table schema creation (`SchemaUtils.create`)
> **NEVER**: Use string literals for column names
> **NEVER**: Expose database exceptions to API layer

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| **DSL API** | Type-safe SQL queries | `select`, `insert`, `update` |
| **DAO API** | Object-oriented approach | Entity classes |
| **Transactions** | All DB operations | `transaction { }` |
| **dbQuery** | Async operations | Coroutines, non-blocking |

## Core Patterns

### Database Setup

```kotlin
object DatabaseFactory {
    fun init() {
        val config = HikariConfig().apply {
            jdbcUrl = "jdbc:postgresql://localhost:5432/db"
            driverClassName = "org.postgresql.Driver"
            username = "user"
            password = "password"
            maximumPoolSize = 10
        }
        Database.connect(HikariDataSource(config))
        transaction { SchemaUtils.create(Users, Orders) }
    }
}

suspend fun <T> dbQuery(block: () -> T): T =
    withContext(Dispatchers.IO) {
        transaction { block() }
    }
```

### Table Definitions

```kotlin
object Users : Table("users") {
    val id = long("id").autoIncrement()
    val name = varchar("name", 100)
    val email = varchar("email", 100).uniqueIndex()
    val createdAt = timestamp("created_at").defaultExpression(CurrentTimestamp())
    override val primaryKey = PrimaryKey(id)
}

object Orders : Table("orders") {
    val id = long("id").autoIncrement()
    val userId = long("user_id").references(Users.id, onDelete = ReferenceOption.CASCADE)
    val total = decimal("total", 10, 2)
    override val primaryKey = PrimaryKey(id)
}
```

### DSL Queries (CRUD)

```kotlin
class UserRepository {
    suspend fun findAll(): List<User> = dbQuery {
        Users.selectAll().map { toUser(it) }
    }
    
    suspend fun findById(id: Long): User? = dbQuery {
        Users.select { Users.id eq id }
            .mapNotNull { toUser(it) }
            .singleOrNull()
    }
    
    suspend fun create(name: String, email: String): User = dbQuery {
        val id = Users.insert {
            it[Users.name] = name
            it[Users.email] = email
        } get Users.id
        User(id, name, email)
    }
    
    suspend fun update(id: Long, name: String): Boolean = dbQuery {
        Users.update({ Users.id eq id }) {
            it[Users.name] = name
        } > 0
    }
    
    suspend fun delete(id: Long): Boolean = dbQuery {
        Users.deleteWhere { Users.id eq id } > 0
    }
}
```

### Joins & Relationships

```kotlin
suspend fun getUserWithOrders(userId: Long): UserWithOrders? = dbQuery {
    (Users innerJoin Orders)
        .select { Users.id eq userId }
        .map { row ->
            UserWithOrders(
                user = toUser(row),
                orders = toOrder(row)
            )
        }
        .singleOrNull()
}
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **No Transaction** | Direct `Users.select()` | `transaction { Users.select() }` | Data consistency |
| **Blocking Calls** | `transaction { }` | `dbQuery { }` | Thread blocking |
| **No Connection Pool** | Direct `Database.connect()` | HikariCP | Performance |
| **No Schema Creation** | Skip `SchemaUtils.create` | Create schema | Missing tables |

### Anti-Pattern: No Transaction (DATA CORRUPTION)

```kotlin
// ❌ WRONG: No transaction
suspend fun createUser(name: String): User {
    Users.insert {
        it[Users.name] = name
    }  // Not in transaction!
}

// ✅ CORRECT: In transaction
suspend fun createUser(name: String): User = dbQuery {
    val id = Users.insert {
        it[Users.name] = name
    } get Users.id
    User(id, name)
}
```

## AI Self-Check (Verify BEFORE generating Exposed code)

- [ ] All operations in transactions?
- [ ] Using dbQuery for async operations?
- [ ] Tables defined as objects?
- [ ] HikariCP for connection pooling?
- [ ] Schema created with SchemaUtils?
- [ ] Using DSL operators (eq, like, etc.)?
- [ ] Proper error handling?
- [ ] No string literals for columns?
- [ ] Joins for relationships?
- [ ] Non-blocking operations?

## Key Features

| Feature | Purpose | Keywords |
|---------|---------|----------|
| **DSL API** | Type-safe queries | `select`, `where`, `join` |
| **Transactions** | Consistency | `transaction { }` |
| **SchemaUtils** | DDL operations | `create`, `drop`, `createMissingTablesAndColumns` |
| **References** | Foreign keys | `references`, `onDelete` |
| **Async** | Non-blocking | `dbQuery`, coroutines |

## Best Practices

**MUST**:
- Transactions for all operations
- dbQuery for async
- HikariCP connection pooling
- Schema creation
- Object-based table definitions

**SHOULD**:
- Use DSL API for queries
- Proper indexes
- Cascade deletes
- Error handling

**AVOID**:
- Operations outside transactions
- Blocking synchronous calls
- Direct database connections
- String literals
- N+1 queries
