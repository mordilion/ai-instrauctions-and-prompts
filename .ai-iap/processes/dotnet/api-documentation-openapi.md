# API Documentation Process - .NET/C# (OpenAPI/Swagger)

> **Purpose**: Auto-generate interactive API documentation with Swashbuckle

> **Tool**: Swashbuckle.AspNetCore ⭐ (built-in with ASP.NET Core)

---

## Phase 1: Setup Swashbuckle

**Install** (if not already):
```bash
dotnet add package Swashbuckle.AspNetCore
```

**Configure** (Program.cs):
```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Version = "v1",
        Title = "My API",
        Description = "API for my application"
    });
    
    // JWT Auth
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT"
    });
});

app.UseSwagger();
app.UseSwaggerUI();
```

**Annotate Controllers**:
```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(User), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<User> GetUser(int id) { }
}
```

> **Access**: https://localhost:5001/swagger

---

## Phase 2: XML Comments

**Enable XML Documentation**:
```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

**Include in Swagger**:
```csharp
options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, "MyApi.xml"));
```

**Document with XML**:
```csharp
/// <summary>
/// Gets a user by ID
/// </summary>
/// <param name="id">User ID</param>
/// <returns>User object</returns>
[HttpGet("{id}")]
public ActionResult<User> GetUser(int id) { }
```

---

## Phase 3: Security & Versioning

### 3.1 Document Authentication

**JWT Configuration**:
```csharp
options.AddSecurityRequirement(new OpenApiSecurityRequirement
{
    {
        new OpenApiSecurityScheme
        {
            Reference = new OpenApiReference
            {
                Type = ReferenceType.SecurityScheme,
                Id = "Bearer"
            }
        },
        Array.Empty<string>()
    }
});
```

### 3.2 API Versioning

**Install**:
```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

**Configure**:
```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
});

options.SwaggerDoc("v1", new OpenApiInfo { Version = "v1", Title = "My API V1" });
options.SwaggerDoc("v2", new OpenApiInfo { Version = "v2", Title = "My API V2" });
```

### 3.3 Rate Limiting (ASP.NET Core 7+)

**Document**:
```csharp
[RateLimiting("fixed")]
[ProducesResponseType(StatusCodes.Status429TooManyRequests)]
public ActionResult<User> GetUser(int id) { }
```

---

## Phase 4: CI/CD Integration

> **ALWAYS**:
> - Generate OpenAPI JSON during build
> - Validate with Spectral or similar
> - Version control the spec
> - Export as build artifact

**Generate Spec**:
```bash
dotnet build
dotnet swagger tofile --output openapi.json bin/Debug/net8.0/MyApi.dll v1
```

---

## Best Practices

> **ALWAYS**:
> - Use XML comments for descriptions
> - Document all status codes with `[ProducesResponseType]`
> - Group endpoints with `[ApiExplorerSettings(GroupName = "...")]`
> - Add examples with `[SwaggerRequestExample]`

> **NEVER**:
> - Expose internal APIs in docs (use `[ApiExplorerSettings(IgnoreApi = true)]`)
> - Include sensitive data in examples
> - Skip documenting error responses

---

## Troubleshooting

### Issue: Swagger UI shows 404
- **Solution**: Ensure `app.UseSwagger()` before `app.UseSwaggerUI()`, check route template

### Issue: XML comments not appearing
- **Solution**: Verify `<GenerateDocumentationFile>true</GenerateDocumentationFile>` in .csproj, check file path in options

### Issue: JWT auth button not showing
- **Solution**: Add both `AddSecurityDefinition` and `AddSecurityRequirement`

---

## AI Self-Check

- [ ] Swashbuckle.AspNetCore installed and configured
- [ ] Swagger UI accessible at `/swagger`
- [ ] All endpoints documented with XML comments
- [ ] JWT authentication documented and testable
- [ ] Response types documented with `[ProducesResponseType]`
- [ ] Error responses documented (400, 401, 404, 500)
- [ ] API versioning configured (if multi-version)
- [ ] OpenAPI spec can be exported for CI/CD
- [ ] No warnings about missing XML comments

---

**Process Complete** ✅

