# API Documentation Process - Java (OpenAPI/Swagger)

> **Purpose**: Auto-generate interactive API documentation with SpringDoc

> **Tool**: springdoc-openapi ⭐ (Spring Boot 3+)

---

## Phase 1: Setup SpringDoc

**Dependencies** (Maven):
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.2.0</version>
</dependency>
```

**Configuration** (application.properties):
```properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
```

**Annotate Controllers**:
```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "Users", description = "User management APIs")
public class UserController {
    
    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    public ResponseEntity<User> getUser(@PathVariable Long id) { }
}
```

> **Access**: http://localhost:8080/swagger-ui.html

---

## Phase 2: Security Documentation

**Configure JWT**:
```java
@Configuration
public class OpenApiConfig {
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info().title("My API").version("1.0"))
            .addSecurityItem(new SecurityRequirement().addList("Bearer Auth"))
            .components(new Components()
                .addSecuritySchemes("Bearer Auth", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")));
    }
}
```

---

## Phase 3: Versioning & Additional Features

### 3.1 API Versioning

**Configure**:
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { }
```

**Document Multiple Versions**:
```java
@Bean
public GroupedOpenAPI publicApiV1() {
    return GroupedOpenAPI.builder()
        .group("v1")
        .pathsToMatch("/api/v1/**")
        .build();
}

@Bean
public GroupedOpenAPI publicApiV2() {
    return GroupedOpenAPI.builder()
        .group("v2")
        .pathsToMatch("/api/v2/**")
        .build();
}
```

### 3.2 Request/Response Examples

**Add Examples**:
```java
@Operation(summary = "Create user")
@ApiResponses({
    @ApiResponse(responseCode = "201", description = "User created",
        content = @Content(schema = @Schema(implementation = User.class),
            examples = @ExampleObject(value = "{\"id\":1,\"name\":\"John\"}")))
})
public ResponseEntity<User> createUser(@RequestBody User user) { }
```

### 3.3 Rate Limiting Documentation

**Document Headers**:
```java
@Operation(summary = "Get user", 
    description = "Rate limit: 100 requests/hour per IP")
@ApiResponse(responseCode = "429", description = "Too many requests")
public ResponseEntity<User> getUser(@PathVariable Long id) { }
```

---

## Phase 4: CI/CD Integration

> **ALWAYS**:
> - Export OpenAPI spec during build
> - Validate spec in CI/CD
> - Version control the spec file

**Maven Plugin** (generate spec):
```xml
<plugin>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-maven-plugin</artifactId>
    <version>1.4</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Access Spec**:
- JSON: http://localhost:8080/v3/api-docs
- YAML: http://localhost:8080/v3/api-docs.yaml

---

## Best Practices

> **ALWAYS**:
> - Use `@Tag` to group related endpoints
> - Document all parameters with `@Parameter`
> - Include realistic examples with `@ExampleObject`
> - Document error responses (400, 401, 403, 404, 500)
> - Use schemas for request/response bodies

> **NEVER**:
> - Expose internal/admin endpoints without security annotations
> - Include passwords or tokens in examples
> - Skip documenting deprecated endpoints (use `@Deprecated` + `@Operation(deprecated = true)`)

---

## Troubleshooting

### Issue: Swagger UI not loading
- **Solution**: Check `springdoc.swagger-ui.enabled=true` in application.properties

### Issue: Endpoints missing from docs
- **Solution**: Verify `@RestController` or `@Controller` + `@ResponseBody`, check path patterns

### Issue: Security not showing in Swagger UI
- **Solution**: Ensure both `SecurityRequirement` and `SecurityScheme` configured

### Issue: Want to hide specific endpoints
- **Solution**: Use `@Hidden` annotation on method or controller

---

## AI Self-Check

- [ ] springdoc-openapi dependency installed
- [ ] Swagger UI accessible at `/swagger-ui.html`
- [ ] All endpoints annotated with `@Operation`
- [ ] JWT security configured and testable
- [ ] Request/response schemas defined
- [ ] Error responses documented (400, 401, 404, 500)
- [ ] API versioning configured (if needed)
- [ ] Examples provided for complex requests
- [ ] OpenAPI spec accessible at `/v3/api-docs`

---

**Process Complete** ✅

