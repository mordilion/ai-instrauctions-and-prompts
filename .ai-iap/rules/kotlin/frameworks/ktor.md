# Ktor Framework

> **Scope**: Apply these rules when working with Ktor applications
> **Applies to**: Kotlin files in Ktor projects
> **Extends**: kotlin/architecture.md, kotlin/code-style.md
> **Precedence**: Framework rules OVERRIDE Kotlin rules for Ktor-specific patterns

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use suspend functions for all I/O operations
> **ALWAYS**: Use content negotiation for JSON (kotlinx.serialization)
> **ALWAYS**: Use routing DSL for endpoints
> **ALWAYS**: Use dependency injection (Koin recommended)
> **ALWAYS**: Use typed routes with type-safe routing
> 
> **NEVER**: Use blocking I/O in route handlers
> **NEVER**: Skip content negotiation setup
> **NEVER**: Use global mutable state
> **NEVER**: Forget to install plugins (ContentNegotiation, etc.)
> **NEVER**: Skip authentication for protected routes

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| Routing DSL | Define endpoints | `routing { get("/") {} }` |
| Content Negotiation | JSON/XML serialization | `install(ContentNegotiation)` |
| Type-safe Routing | Type-safe URLs | Resources, typed routes |
| Koin DI | Dependency injection | `inject()`, modules |
| Suspend Functions | Async operations | `suspend fun`, coroutines |

## Core Patterns

### Application Setup
```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json(Json {
            prettyPrint = true
            isLenient = true
        })
    }
    
    install(StatusPages) {
        exception<Throwable> { call, cause ->
            call.respondText("Error: ${cause.message}", status = HttpStatusCode.InternalServerError)
        }
    }
    
    routing {
        userRoutes()
    }
}
```

### Routing
```kotlin
fun Route.userRoutes() {
    route("/users") {
        get {
            val users = userService.getAll()
            call.respond(users)
        }
        
        get("/{id}") {
            val id = call.parameters["id"]?.toIntOrNull()
                ?: return@get call.respond(HttpStatusCode.BadRequest)
            val user = userService.getById(id)
                ?: return@get call.respond(HttpStatusCode.NotFound)
            call.respond(user)
        }
        
        post {
            val user = call.receive<CreateUserRequest>()
            val created = userService.create(user)
            call.respond(HttpStatusCode.Created, created)
        }
    }
}
```

### Data Classes with Serialization
```kotlin
import kotlinx.serialization.Serializable

@Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String
)

@Serializable
data class CreateUserRequest(
    val name: String,
    val email: String
)
```

### Dependency Injection (Koin)
```kotlin
val appModule = module {
    single { UserRepository() }
    single { UserService(get()) }
}

fun Application.module() {
    install(Koin) {
        modules(appModule)
    }
    
    routing {
        get("/users") {
            val userService by inject<UserService>()
            call.respond(userService.getAll())
        }
    }
}
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Blocking I/O** | Regular functions for DB/HTTP | `suspend fun` | Blocks coroutines |
| **No ContentNegotiation** | Manual JSON parsing | `install(ContentNegotiation)` | No auto-serialization |
| **Global State** | `val db = Database()` at top level | Dependency injection | Not thread-safe |
| **Missing Plugins** | Forget to install features | Install all needed plugins | Features don't work |
| **String Routes Only** | All routes as strings | Type-safe routing | No compile-time safety |

### Anti-Pattern: Blocking I/O (KILLS PERFORMANCE)
```kotlin
// ❌ WRONG - Blocking I/O
get("/users") {
    val users = database.query("SELECT * FROM users")  // Blocks!
    call.respond(users)
}

// ✅ CORRECT - Suspend function
get("/users") {
    val users = database.queryAsync("SELECT * FROM users")  // Non-blocking
    call.respond(users)
}
```

### Anti-Pattern: No Content Negotiation
```kotlin
// ❌ WRONG - Manual JSON
get("/users") {
    val users = userService.getAll()
    val json = Json.encodeToString(users)  // Manual!
    call.respondText(json, ContentType.Application.Json)
}

// ✅ CORRECT - Auto-serialization
install(ContentNegotiation) {
    json()
}

get("/users") {
    val users = userService.getAll()
    call.respond(users)  // Auto-serialized to JSON
}
```

## AI Self-Check (Verify BEFORE generating Ktor code)

- [ ] Using suspend functions for I/O?
- [ ] ContentNegotiation installed and configured?
- [ ] Routing DSL for endpoints?
- [ ] @Serializable on data classes?
- [ ] Dependency injection (Koin)?
- [ ] StatusPages for error handling?
- [ ] Type-safe routing where appropriate?
- [ ] No blocking I/O?
- [ ] Authentication configured for protected routes?
- [ ] Following Ktor conventions?

## Plugins

```kotlin
install(CallLogging) {
    level = Level.INFO
}

install(CORS) {
    anyHost()
    allowHeader(HttpHeaders.ContentType)
}

install(Authentication) {
    jwt("auth-jwt") {
        verifier(JWTVerifier())
        validate { credential ->
            if (credential.payload.audience.contains("app"))
                JWTPrincipal(credential.payload)
            else null
        }
    }
}
```

## Type-Safe Routing

```kotlin
@Resource("/users")
class Users {
    @Resource("{id}")
    class Id(val parent: Users = Users(), val id: Int)
}

fun Route.userRoutes() {
    get<Users> {
        call.respond(userService.getAll())
    }
    
    get<Users.Id> { user ->
        call.respond(userService.getById(user.id))
    }
}
```

## Testing

```kotlin
@Test
fun testGetUsers() = testApplication {
    client.get("/users").apply {
        assertEquals(HttpStatusCode.OK, status)
        assertTrue(bodyAsText().contains("users"))
    }
}
```

## Key Features

| Feature | Purpose | Plugin |
|---------|---------|--------|
| Routing | Define endpoints | Built-in |
| ContentNegotiation | Serialization | `ContentNegotiation` |
| Authentication | Auth/authz | `Authentication` |
| WebSockets | Real-time | `WebSockets` |
| StatusPages | Error handling | `StatusPages` |

## Key Libraries

- **kotlinx.serialization**: JSON serialization
- **Koin**: Dependency injection
- **Exposed**: SQL database
- **ktor-client**: HTTP client
