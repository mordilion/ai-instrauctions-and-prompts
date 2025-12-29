# API Documentation Process - Swift (Vapor)

> **Purpose**: Auto-generate API documentation for Vapor applications

> **Tools**: VaporToOpenAPI, SwiftOpenAPI

---

## Phase 1: Manual OpenAPI

**Create OpenAPI Spec** (openapi.yaml):
```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
```

**Serve with Vapor**:
```swift
app.get("docs") { req in
    return req.fileio.streamFile(at: "openapi.yaml")
}
```

---

## Phase 2: Swagger UI

**Add Swagger UI** (static files):
- Download Swagger UI dist
- Serve from Public/swagger-ui/
- Point to openapi.yaml

**Route**:
```swift
app.get("api-docs") { req in
    return req.view.render("swagger-ui")
}
```

---

## Phase 3: Security & Authentication

### 3.1 Document JWT Authentication

**OpenAPI Spec**:
```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
security:
  - bearerAuth: []
```

**Vapor Route Documentation**:
```swift
// Document in comments or separate spec
// GET /api/users/{id}
// Security: Bearer token required
// Responses:
//   200: User found
//   401: Unauthorized
//   404: Not found
app.get("api", "users", ":id") { req -> User in
    // ...
}
```

### 3.2 API Versioning

**URL-based Versioning**:
```yaml
paths:
  /api/v1/users:
    get:
      summary: Get users (V1)
  /api/v2/users:
    get:
      summary: Get users (V2 - includes email field)
```

**Vapor Implementation**:
```swift
let v1 = app.grouped("api", "v1")
v1.get("users") { req in /* V1 logic */ }

let v2 = app.grouped("api", "v2")
v2.get("users") { req in /* V2 logic */ }
```

---

## Phase 4: CI/CD Integration

> **ALWAYS**:
> - Version control openapi.yaml
> - Validate spec in CI/CD
> - Generate client SDKs from spec

**Validate Spec**:
```bash
npx @openapitools/openapi-generator-cli validate -i openapi.yaml
```

**CI/CD Example**:
```yaml
- name: Validate OpenAPI Spec
  run: |
    npm install -g @openapitools/openapi-generator-cli
    openapi-generator-cli validate -i openapi.yaml
```

---

## Best Practices

> **ALWAYS**:
> - Keep OpenAPI spec in sync with routes
> - Document all HTTP status codes
> - Include request/response examples
> - Use `$ref` for reusable schemas
> - Document authentication requirements

> **NEVER**:
> - Hardcode secrets in examples
> - Skip documenting error responses
> - Forget to update spec when routes change

---

## Troubleshooting

### Issue: Swagger UI not loading
- **Solution**: Check static file serving, verify swagger-ui dist files present

### Issue: CORS errors in Try-it-out
- **Solution**: Configure CORS middleware in Vapor to allow swagger-ui origin

### Issue: Spec not validating
- **Solution**: Use online validator (swagger.io/tools/swagger-editor), check for syntax errors

### Issue: Want auto-generation from code
- **Solution**: Consider using VaporToOpenAPI package (experimental) or maintain spec manually

---

## AI Self-Check

- [ ] OpenAPI specification created (openapi.yaml)
- [ ] All endpoints documented with paths, methods, parameters
- [ ] Response schemas defined (200, 400, 401, 404, 500)
- [ ] JWT authentication documented in security schemes
- [ ] Swagger UI configured and accessible
- [ ] Examples provided for complex requests
- [ ] API versioning strategy documented
- [ ] OpenAPI spec validates without errors
- [ ] Spec version controlled in repository

---

**Process Complete** ✅

