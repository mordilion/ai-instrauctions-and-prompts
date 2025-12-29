# API Documentation Process - Kotlin (OpenAPI/Swagger)

> **Purpose**: Auto-generate interactive API documentation

> **Tools**: SpringDoc ⭐ (Spring Boot), Ktor OpenAPI

---

## Phase 1: Spring Boot (Same as Java)

**Dependencies** (Gradle):
```kotlin
implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0")
```

**Annotate Controllers**:
```kotlin
@RestController
@RequestMapping("/api/users")
@Tag(name = "Users")
class UserController {
    
    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID")
    fun getUser(@PathVariable id: Long): User { }
}
```

> **Access**: http://localhost:8080/swagger-ui.html

---

## Phase 2: Ktor OpenAPI

**Install**:
```kotlin
implementation("io.github.smiley4:ktor-swagger-ui:$version")
```

**Configure**:
```kotlin
install(SwaggerUI) {
    swagger {
        swaggerUrl = "swagger-ui"
        forwardRoot = true
    }
    info {
        title = "My API"
        version = "1.0.0"
    }
}
```

---

## Phase 3: Security & Versioning

### 3.1 Document JWT Authentication (Spring Boot)

**Security Configuration**:
```kotlin
@Bean
fun customOpenAPI(): OpenAPI {
    return OpenAPI()
        .info(Info().title("My API").version("1.0"))
        .addSecurityItem(SecurityRequirement().addList("Bearer"))
        .components(Components()
            .addSecuritySchemes("Bearer", SecurityScheme()
                .type(SecurityScheme.Type.HTTP)
                .scheme("bearer")
                .bearerFormat("JWT")))
}
```

### 3.2 API Versioning

**URL-based**:
```kotlin
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "Users V1")
class UserControllerV1

@RestController
@RequestMapping("/api/v2/users")
@Tag(name = "Users V2")
class UserControllerV2
```

### 3.3 Ktor Security Documentation

**Configure JWT**:
```kotlin
install(SwaggerUI) {
    security {
        securityScheme("JWT") {
            type = HttpSecuritySchemeType.HTTP
            scheme = HttpAuthScheme.BEARER
            bearerFormat = "JWT"
        }
    }
}
```

---

## Phase 4: CI/CD Integration

> **ALWAYS**:
> - Generate OpenAPI spec during build
> - Validate with Spectral or swagger-cli
> - Export as artifact

**Gradle Task**:
```kotlin
tasks.register("generateOpenApi") {
    dependsOn("build")
    doLast {
        // Export OpenAPI JSON from running app or use springdoc plugin
        println("OpenAPI spec generated")
    }
}
```

---

## Best Practices

> **ALWAYS**:
> - Use data classes for request/response models
> - Add KDoc comments to endpoints (included in docs)
> - Document all HTTP status codes
> - Group endpoints with `@Tag`
> - Use `@Parameter` for path/query params

> **NEVER**:
> - Expose internal endpoints without security
> - Include tokens/passwords in examples
> - Skip error response documentation

---

## Troubleshooting

### Issue: Swagger UI not loading (Spring Boot)
- **Solution**: Check `springdoc.swagger-ui.enabled=true`, verify path

### Issue: Endpoints missing (Ktor)
- **Solution**: Ensure routes are registered before SwaggerUI plugin

### Issue: Security schemes not appearing
- **Solution**: Verify both security scheme and requirement configured

---

## AI Self-Check

- [ ] OpenAPI documentation configured
- [ ] Swagger UI accessible
- [ ] All endpoints documented with summaries
- [ ] JWT/OAuth security documented
- [ ] Request/response schemas defined
- [ ] Error responses documented (400, 401, 404, 500)
- [ ] API versioning configured (if needed)
- [ ] Try-it-out functionality works
- [ ] OpenAPI spec can be exported

---

**Process Complete** ✅

