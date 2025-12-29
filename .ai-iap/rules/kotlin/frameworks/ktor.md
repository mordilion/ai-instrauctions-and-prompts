# Ktor Framework

> **Scope**: Ktor applications
> **Applies to**: Kotlin files in Ktor projects
> **Extends**: kotlin/architecture.md, kotlin/code-style.md

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use suspend functions for I/O
> **ALWAYS**: Use content negotiation for JSON
> **ALWAYS**: Use routing DSL for endpoints
> **ALWAYS**: Use DI (Koin recommended)
> 
> **NEVER**: Use blocking I/O in handlers
> **NEVER**: Skip authentication for protected routes
> **NEVER**: Use global mutable state

## Core Patterns

### Setup

```kotlin
fun Application.module() {
    install(ContentNegotiation) { json() }
    install(StatusPages) {
        exception<Throwable> { call, cause ->
            call.respondText("Error: ${cause.message}", status = HttpStatusCode.InternalServerError)
        }
    }
    routing { userRoutes() }
}
```

### Routing

```kotlin
fun Route.userRoutes() {
    route("/users") {
        get {
            call.respond(userService.getAll())
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
            call.respond(HttpStatusCode.Created, userService.create(user))
        }
    }
}
```

### DI (Koin)

```kotlin
val appModule = module {
    single<UserRepository> { UserRepositoryImpl(get()) }
    single<UserService> { UserService(get()) }
}

fun Application.configureDI() {
    install(Koin) {
        modules(appModule)
    }
}

fun Route.userRoutesWithDI() {
    val userService by inject<UserService>()
    get("/users") {
        call.respond(userService.getAll())
    }
}
```

### Auth

```kotlin
install(Authentication) {
    jwt("auth-jwt") {
        verifier(JWK...)
        validate { credential ->
            if (credential.payload.getClaim("username").asString() != "") {
                JWTPrincipal(credential.payload)
            } else null
        }
    }
}

routing {
    authenticate("auth-jwt") {
        get("/protected") {
            val principal = call.principal<JWTPrincipal>()
            call.respondText("Hello ${principal?.payload?.getClaim("username")}")
        }
    }
}
```

## Common AI Mistakes

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| **Blocking I/O** | `Thread.sleep()` | `delay()` |
| **No Content Negotiation** | Manual JSON | `install(ContentNegotiation)` |
| **No DI** | `UserService()` | `inject<UserService>()` |
| **No Error Handling** | Ignore exceptions | `install(StatusPages)` |

### Anti-Pattern: Blocking I/O

```kotlin
// ❌ WRONG
get("/users") {
    Thread.sleep(1000)  // Blocks thread!
    call.respond(users)
}

// ✅ CORRECT
get("/users") {
    delay(1000)  // Suspends, non-blocking
    call.respond(users)
}
```

## AI Self-Check

- [ ] suspend functions for I/O?
- [ ] ContentNegotiation installed?
- [ ] Routing DSL used?
- [ ] DI configured (Koin)?
- [ ] Authentication for protected routes?
- [ ] StatusPages for error handling?
- [ ] No blocking operations?
- [ ] call.respond() for responses?

## Key Plugins

| Plugin | Purpose |
|--------|---------|
| ContentNegotiation | JSON/XML serialization |
| StatusPages | Error handling |
| Authentication | JWT, OAuth |
| Koin | Dependency injection |
| CORS | Cross-origin requests |

## Best Practices

**MUST**: suspend functions, Content Negotiation, routing DSL, DI
**SHOULD**: StatusPages, Authentication, type-safe routing
**AVOID**: Blocking I/O, global state, skipping auth
