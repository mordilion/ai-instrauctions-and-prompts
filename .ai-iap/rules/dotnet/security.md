# .NET/C# Security

> **Scope**: .NET-specific security practices (ASP.NET Core, Web API, Blazor)
> **Extends**: general/security.md
> **Applies to**: *.cs, *.cshtml, *.razor files

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use Identity Framework for authentication
> **ALWAYS**: Enable HTTPS redirection in production
> **ALWAYS**: Use parameterized SQL or Entity Framework
> **ALWAYS**: Validate model state before processing
> **ALWAYS**: Use `[Authorize]` attribute on protected endpoints
> 
> **NEVER**: Disable request validation (`ValidateInput(false)`)
> **NEVER**: Return sensitive data in exceptions
> **NEVER**: Store passwords in plaintext
> **NEVER**: Trust user input (all input is malicious)
> **NEVER**: Disable HTTPS in production

## 1. Authentication (ASP.NET Core Identity)

### Setup Identity

```csharp
// ✅ CORRECT - Configure Identity
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>(options =>
    {
        // Password settings
        options.Password.RequireDigit = true;
        options.Password.RequiredLength = 12;
        options.Password.RequireNonAlphanumeric = true;
        options.Password.RequireUppercase = true;
        options.Password.RequireLowercase = true;
        
        // Lockout settings
        options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
        options.Lockout.MaxFailedAccessAttempts = 5;
        options.Lockout.AllowedForNewUsers = true;
        
        // User settings
        options.User.RequireUniqueEmail = true;
    })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
}
```

### Password Hashing

```csharp
// ✅ CORRECT - Identity handles hashing automatically
public class UserService
{
    private readonly UserManager<ApplicationUser> _userManager;
    
    public async Task<IdentityResult> CreateUserAsync(string email, string password)
    {
        var user = new ApplicationUser { UserName = email, Email = email };
        return await _userManager.CreateAsync(user, password);
        // Password automatically hashed with PBKDF2
    }
    
    public async Task<bool> ValidatePasswordAsync(ApplicationUser user, string password)
    {
        return await _userManager.CheckPasswordAsync(user, password);
    }
}

// ❌ WRONG - Manual hashing
public string HashPassword(string password)
{
    using var sha256 = SHA256.Create();
    var hash = sha256.ComputeHash(Encoding.UTF8.GetBytes(password));
    return Convert.ToBase64String(hash);  // Weak! No salt!
}
```

### JWT Authentication

```csharp
// ✅ CORRECT - Secure JWT configuration
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ValidIssuer = Configuration["Jwt:Issuer"],
                ValidAudience = Configuration["Jwt:Audience"],
                IssuerSigningKey = new SymmetricSecurityKey(
                    Encoding.UTF8.GetBytes(Configuration["Jwt:Secret"]))
            };
        });
}

// Generate token
public string GenerateJwtToken(ApplicationUser user)
{
    var claims = new[]
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.Role, user.Role)
        // NEVER include: Password, API keys, PII
    };
    
    var key = new SymmetricSecurityKey(
        Encoding.UTF8.GetBytes(_configuration["Jwt:Secret"]));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    
    var token = new JwtSecurityToken(
        issuer: _configuration["Jwt:Issuer"],
        audience: _configuration["Jwt:Audience"],
        claims: claims,
        expires: DateTime.UtcNow.AddMinutes(15),  // Short-lived
        signingCredentials: creds
    );
    
    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

## 2. Authorization

### Controller Authorization

```csharp
// ✅ CORRECT - Authorize attribute
[ApiController]
[Route("api/[controller]")]
[Authorize]  // Require authentication
public class UsersController : ControllerBase
{
    [HttpGet]
    [AllowAnonymous]  // Override for specific endpoint
    public IActionResult GetPublicData() { }
    
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]  // Role-based
    public async Task<IActionResult> DeleteUser(string id) { }
    
    [HttpPut("{id}")]
    [Authorize(Policy = "CanEditUser")]  // Policy-based
    public async Task<IActionResult> UpdateUser(string id, UpdateUserDto dto) { }
}

// ❌ WRONG - No authorization
public class UsersController : ControllerBase
{
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteUser(string id)
    {
        // Anyone can delete any user!
    }
}
```

### Policy-Based Authorization

```csharp
// ✅ CORRECT - Custom authorization policy
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy("CanEditUser", policy =>
            policy.Requirements.Add(new CanEditUserRequirement()));
    });
    
    services.AddSingleton<IAuthorizationHandler, CanEditUserHandler>();
}

public class CanEditUserRequirement : IAuthorizationRequirement { }

public class CanEditUserHandler 
    : AuthorizationHandler<CanEditUserRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        CanEditUserRequirement requirement)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var httpContext = context.Resource as HttpContext;
        var routeUserId = httpContext?.Request.RouteValues["id"]?.ToString();
        
        // Check if user is editing their own profile or is admin
        if (userId == routeUserId || 
            context.User.IsInRole("Admin"))
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}
```

## 3. Input Validation

### Model Validation

```csharp
// ✅ CORRECT - Data annotations
public class CreateUserDto
{
    [Required]
    [EmailAddress]
    [MaxLength(255)]
    public string Email { get; set; }
    
    [Required]
    [MinLength(12)]
    [MaxLength(128)]
    [RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{12,}$")]
    public string Password { get; set; }
    
    [Required]
    [MinLength(2)]
    [MaxLength(100)]
    [RegularExpression(@"^[a-zA-Z\s]+$")]
    public string Name { get; set; }
    
    [Range(18, 120)]
    public int Age { get; set; }
}

[ApiController]
public class UsersController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserDto dto)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);  // Return validation errors
        }
        
        var user = await _userService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
}
```

### Custom Validation

```csharp
// ✅ CORRECT - Custom validation attribute
public class NoSqlInjectionAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(
        object? value, ValidationContext validationContext)
    {
        if (value is string str)
        {
            var dangerous = new[] { "';", "DROP", "DELETE", "INSERT", "UPDATE", "EXEC" };
            if (dangerous.Any(d => str.Contains(d, StringComparison.OrdinalIgnoreCase)))
            {
                return new ValidationResult("Invalid characters detected");
            }
        }
        
        return ValidationResult.Success;
    }
}

public class CreatePostDto
{
    [Required]
    [MaxLength(200)]
    [NoSqlInjection]
    public string Title { get; set; }
}
```

## 4. SQL Injection Prevention

### Entity Framework Core

```csharp
// ✅ CORRECT - EF Core (parameterized)
public async Task<User?> GetUserByEmailAsync(string email)
{
    return await _context.Users
        .Where(u => u.Email == email)
        .FirstOrDefaultAsync();
}

// ✅ CORRECT - Raw SQL with parameters
public async Task<List<User>> SearchUsersAsync(string searchTerm)
{
    return await _context.Users
        .FromSqlRaw(
            "SELECT * FROM Users WHERE Name LIKE {0}",
            $"%{searchTerm}%")
        .ToListAsync();
}

// ❌ WRONG - String interpolation
public async Task<List<User>> SearchUsersAsync(string searchTerm)
{
    return await _context.Users
        .FromSqlRaw($"SELECT * FROM Users WHERE Name LIKE '%{searchTerm}%'")
        .ToListAsync();
    // SQL INJECTION VULNERABILITY!
}
```

### Dapper

```csharp
// ✅ CORRECT - Parameterized query
public async Task<User?> GetUserAsync(string email)
{
    const string query = "SELECT * FROM Users WHERE Email = @Email";
    return await _connection.QueryFirstOrDefaultAsync<User>(
        query, 
        new { Email = email });
}

// ❌ WRONG - String concatenation
public async Task<User?> GetUserAsync(string email)
{
    var query = $"SELECT * FROM Users WHERE Email = '{email}'";
    return await _connection.QueryFirstOrDefaultAsync<User>(query);
    // SQL INJECTION!
}
```

## 5. XSS Prevention

### Razor Pages (Automatic Encoding)

```cshtml
@* ✅ CORRECT - Razor automatically encodes *@
<div>@Model.UserInput</div>
<input type="text" value="@Model.UserInput" />

@* ❌ DANGEROUS - Raw HTML *@
<div>@Html.Raw(Model.UserInput)</div>

@* ✅ CORRECT - Sanitize if HTML needed *@
@using Ganss.Xss
@{
    var sanitizer = new HtmlSanitizer();
    var clean = sanitizer.Sanitize(Model.UserInput);
}
<div>@Html.Raw(clean)</div>
```

### Blazor

```csharp
// ✅ CORRECT - Blazor automatically encodes
<div>@UserInput</div>

// ❌ DANGEROUS - MarkupString (no sanitization)
<div>@((MarkupString)UserInput)</div>

// ✅ CORRECT - Sanitize first
@using Ganss.Xss
@code {
    private string UserInput { get; set; }
    
    private MarkupString SafeHtml =>
        (MarkupString)new HtmlSanitizer().Sanitize(UserInput);
}
<div>@SafeHtml</div>
```

## 6. HTTPS & Security Headers

### HTTPS Redirection

```csharp
// ✅ CORRECT - Program.cs
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    builder.Services.AddHttpsRedirection(options =>
    {
        options.RedirectStatusCode = StatusCodes.Status308PermanentRedirect;
        options.HttpsPort = 443;
    });
    
    builder.Services.AddHsts(options =>
    {
        options.MaxAge = TimeSpan.FromDays(365);
        options.IncludeSubDomains = true;
        options.Preload = true;
    });
}

var app = builder.Build();

if (app.Environment.IsProduction())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}
```

### Security Headers

```csharp
// ✅ CORRECT - Add security headers
public void Configure(IApplicationBuilder app)
{
    app.Use(async (context, next) =>
    {
        context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
        context.Response.Headers.Add("X-Frame-Options", "DENY");
        context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
        context.Response.Headers.Add("Referrer-Policy", "no-referrer");
        context.Response.Headers.Add(
            "Content-Security-Policy",
            "default-src 'self'; script-src 'self' 'nonce-{random}'");
        
        await next();
    });
}
```

## 7. CORS Configuration

```csharp
// ✅ CORRECT - Restrictive CORS
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("AllowedOrigins", builder =>
        {
            builder
                .WithOrigins(
                    "https://yourapp.com",
                    "https://staging.yourapp.com")
                .AllowAnyMethod()
                .AllowAnyHeader()
                .AllowCredentials();
        });
    });
}

public void Configure(IApplicationBuilder app)
{
    app.UseCors("AllowedOrigins");
}

// ❌ WRONG - Open CORS
services.AddCors(options =>
{
    options.AddPolicy("AllowAll", builder =>
    {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
});
```

## 8. Configuration & Secrets

### User Secrets (Development)

```bash
# Initialize user secrets
dotnet user-secrets init

# Set secrets
dotnet user-secrets set "Jwt:Secret" "your-secret-key-here"
dotnet user-secrets set "ConnectionStrings:Default" "Server=..."
```

```csharp
// ✅ CORRECT - Access secrets
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    
    public IConfiguration Configuration { get; }
    
    public void ConfigureServices(IServiceCollection services)
    {
        var jwtSecret = Configuration["Jwt:Secret"];
        var connectionString = Configuration.GetConnectionString("Default");
        
        // NEVER: var secret = "hardcoded-secret";
    }
}
```

### Azure Key Vault (Production)

```csharp
// ✅ CORRECT - Use Key Vault
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    var keyVaultUrl = builder.Configuration["KeyVault:Url"];
    builder.Configuration.AddAzureKeyVault(
        new Uri(keyVaultUrl),
        new DefaultAzureCredential());
}
```

## 9. Error Handling

```csharp
// ✅ CORRECT - Global exception handler
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();  // Detailed errors OK in dev
    }
    else
    {
        app.UseExceptionHandler("/error");  // Generic errors in production
    }
}

[ApiController]
[ApiExplorerSettings(IgnoreApi = true)]
public class ErrorController : ControllerBase
{
    [Route("/error")]
    public IActionResult HandleError()
    {
        var context = HttpContext.Features.Get<IExceptionHandlerFeature>();
        var exception = context?.Error;
        
        // Log detailed error
        _logger.LogError(exception, "Unhandled exception occurred");
        
        // Return generic message to client
        return Problem(
            title: "An error occurred",
            statusCode: StatusCodes.Status500InternalServerError);
    }
}

// ❌ WRONG - Expose exception details
catch (Exception ex)
{
    return BadRequest(new { error = ex.Message, stack = ex.StackTrace });
    // Exposes internals!
}
```

## 10. File Upload Security

```csharp
// ✅ CORRECT - Secure file upload
[HttpPost("upload")]
public async Task<IActionResult> UploadFile(IFormFile file)
{
    var allowedTypes = new[] { ".jpg", ".png", ".gif" };
    var maxSize = 5 * 1024 * 1024;  // 5MB
    
    if (file == null || file.Length == 0)
        return BadRequest("No file uploaded");
    
    if (file.Length > maxSize)
        return BadRequest("File too large");
    
    var extension = Path.GetExtension(file.FileName).ToLowerInvariant();
    if (!allowedTypes.Contains(extension))
        return BadRequest("Invalid file type");
    
    // Generate safe filename
    var fileName = $"{Guid.NewGuid()}{extension}";
    var filePath = Path.Combine(_uploadPath, fileName);
    
    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await file.CopyToAsync(stream);
    }
    
    return Ok(new { fileName });
}
```

## 11. Rate Limiting (.NET 7+)

```csharp
// ✅ CORRECT - Built-in rate limiting
public void ConfigureServices(IServiceCollection services)
{
    services.AddRateLimiter(options =>
    {
        options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(
            context =>
            {
                return RateLimitPartition.GetFixedWindowLimiter(
                    partitionKey: context.User.Identity?.Name ?? context.Request.Headers.Host.ToString(),
                    factory: partition => new FixedWindowRateLimiterOptions
                    {
                        AutoReplenishment = true,
                        PermitLimit = 100,
                        Window = TimeSpan.FromMinutes(1)
                    });
            });
    });
}

public void Configure(IApplicationBuilder app)
{
    app.UseRateLimiter();
}

// Endpoint-specific
[EnableRateLimiting("auth")]
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] LoginDto dto) { }
```

## AI Self-Check

Before generating .NET code, verify:
- [ ] Identity Framework configured for authentication?
- [ ] Password requirements set (length, complexity)?
- [ ] `[Authorize]` attribute on protected endpoints?
- [ ] Model validation attributes on DTOs?
- [ ] Entity Framework used (not string concatenation for SQL)?
- [ ] HTTPS redirection enabled in production?
- [ ] HSTS configured?
- [ ] Security headers added (CSP, X-Frame-Options, etc.)?
- [ ] CORS configured restrictively (not `AllowAnyOrigin`)?
- [ ] Secrets in User Secrets/Key Vault (not hardcoded)?
- [ ] Global exception handler configured?
- [ ] File uploads validated (type, size)?
- [ ] Rate limiting configured?
- [ ] Razor pages auto-escape user input?
- [ ] No `@Html.Raw()` without sanitization?

## Testing Security

```csharp
// ✅ Security testing example (xUnit)
public class SecurityTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    
    [Fact]
    public async Task Protected_Endpoint_Requires_Authentication()
    {
        var client = _factory.CreateClient();
        
        var response = await client.GetAsync("/api/users");
        
        Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
    }
    
    [Fact]
    public async Task SQL_Injection_Is_Prevented()
    {
        var client = _factory.CreateClient();
        var malicious = "admin'; DROP TABLE Users--";
        
        var response = await client.GetAsync($"/api/users/{malicious}");
        
        Assert.NotEqual(HttpStatusCode.InternalServerError, response.StatusCode);
    }
}
```

---

**Security in .NET is built-in and comprehensive. Use the framework's features instead of rolling your own security.**

