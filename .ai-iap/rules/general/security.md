# Security Guidelines (General)

> **Scope**: Baseline security practices for ALL languages. Language-specific rules extend these.
> **Precedence**: Security rules ALWAYS apply - they CANNOT be overridden.

## CRITICAL REQUIREMENTS (AI: NEVER skip these)

> **ALWAYS**: Validate ALL user input at entry points (API, forms, file uploads)
> **ALWAYS**: Use parameterized queries/prepared statements for database access
> **ALWAYS**: Encode output based on context (HTML, SQL, JavaScript, URL)
> **ALWAYS**: Use HTTPS/TLS for ALL production traffic
> **ALWAYS**: Store secrets in environment variables or secure vaults
> 
> **NEVER**: Trust user input (all input is malicious until validated)
> **NEVER**: Concatenate user input into SQL queries (SQL injection risk)
> **NEVER**: Return sensitive data in error messages (stack traces, credentials)
> **NEVER**: Hardcode secrets, API keys, passwords in source code
> **NEVER**: Disable security features (CSRF protection, XSS filters, CORS)

## 1. Input Validation

### Validate at Boundaries
```
User Input → VALIDATE → Application Logic
External API → VALIDATE → Application Logic
File Upload → VALIDATE → Storage
```

### Validation Rules
- **Whitelist** over blacklist (allow known good, not block known bad)
- **Type checking**: Ensure correct data types
- **Length limits**: Enforce maximum lengths
- **Format validation**: Email, phone, dates, URLs
- **Range validation**: Numbers within acceptable ranges
- **Sanitization**: Remove dangerous characters AFTER validation

### Common Vulnerabilities

❌ **SQL Injection**:
```javascript
// WRONG - String concatenation
const query = `SELECT * FROM users WHERE id = ${userId}`
```

✅ **Use Parameterized Queries**:
```javascript
// CORRECT - Parameterized
const query = 'SELECT * FROM users WHERE id = ?'
db.query(query, [userId])
```

❌ **NoSQL Injection**:
```javascript
// WRONG - Direct object
db.users.find({ username: req.body.username })
```

✅ **Validate and Sanitize**:
```javascript
// CORRECT - Validated
const username = validator.isAlphanumeric(req.body.username)
db.users.find({ username })
```

## 2. Authentication

### Password Security
- **Hashing**: Use bcrypt, Argon2, or PBKDF2 (NEVER store plaintext)
- **Salt**: Unique salt per password (library handles automatically)
- **Minimum strength**: 12+ characters, complexity requirements
- **Rate limiting**: Prevent brute force (5 attempts per 15 minutes)

```javascript
// ✅ CORRECT - Bcrypt
const hashedPassword = await bcrypt.hash(password, 12)
const isValid = await bcrypt.compare(password, hashedPassword)

// ❌ WRONG - Weak hashing
const hashedPassword = md5(password)  // Broken, DO NOT USE
const hashedPassword = sha1(password)  // Weak, DO NOT USE
```

### Session Management
- **Secure cookies**: `HttpOnly`, `Secure`, `SameSite=Strict`
- **Session timeout**: 15-30 minutes idle timeout
- **Session invalidation**: Logout must destroy session server-side
- **CSRF tokens**: Required for state-changing operations

```javascript
// ✅ CORRECT - Secure session
res.cookie('sessionId', token, {
  httpOnly: true,      // No JavaScript access
  secure: true,        // HTTPS only
  sameSite: 'strict',  // CSRF protection
  maxAge: 1800000      // 30 minutes
})
```

### JWT Tokens
- **Algorithm**: Use RS256 or HS256 (NEVER 'none')
- **Expiration**: Short-lived (15 min access, 7 day refresh)
- **Secret rotation**: Rotate signing keys regularly
- **Payload**: NEVER include sensitive data (PII, passwords)

```javascript
// ✅ CORRECT - Secure JWT
const token = jwt.sign(
  { userId: user.id, role: user.role },  // Minimal payload
  process.env.JWT_SECRET,
  { 
    expiresIn: '15m',
    algorithm: 'HS256'
  }
)

// ❌ WRONG - Insecure JWT
const token = jwt.sign({ password: user.password }, 'hardcoded-secret')
```

## 3. Authorization

### Check Permissions
```
1. Authenticate (who are you?)
2. Authorize (what can you do?)
3. Execute (perform action)
```

```typescript
// ✅ CORRECT - Check ownership
async deletePost(userId: string, postId: string) {
  const post = await getPost(postId)
  if (post.authorId !== userId) {
    throw new ForbiddenError('Not authorized')
  }
  await deletePost(postId)
}

// ❌ WRONG - No authorization check
async deletePost(postId: string) {
  await deletePost(postId)  // Anyone can delete any post!
}
```

### Access Control Patterns
- **Role-Based (RBAC)**: Users have roles (admin, user, guest)
- **Attribute-Based (ABAC)**: Access based on attributes (owner, department)
- **Policy-Based**: Complex rules (time-based, location-based)

## 4. Output Encoding

### Context-Aware Encoding
- **HTML context**: Encode `<`, `>`, `&`, `"`, `'`
- **JavaScript context**: JSON.stringify, escape quotes
- **URL context**: encodeURIComponent
- **SQL context**: Use parameterized queries (not encoding)

```javascript
// ✅ CORRECT - Context-aware encoding
const safeHTML = escapeHtml(userInput)
const safeJS = JSON.stringify(userInput)
const safeURL = encodeURIComponent(userInput)

// ❌ WRONG - No encoding (XSS vulnerability)
html = `<div>${userInput}</div>`  // XSS if userInput contains <script>
```

### Content Security Policy (CSP)
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'
```

## 5. Secrets Management

### Storage
- **Environment variables**: Development/staging
- **Secret vaults**: Production (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault)
- **Kubernetes secrets**: For K8s deployments
- **NEVER**: Commit secrets to version control

```bash
# ✅ CORRECT - Environment variables
DATABASE_URL=postgresql://...
API_KEY=sk_live_...

# ❌ WRONG - Hardcoded in code
const apiKey = 'sk_live_abc123def456'
```

### Rotation
- **API keys**: Rotate every 90 days
- **Database passwords**: Rotate every 90 days
- **JWT secrets**: Rotate every 30 days
- **Use managed rotation**: Cloud provider services

## 6. HTTPS/TLS

### Requirements
- **Production**: HTTPS only (redirect HTTP → HTTPS)
- **Development**: Can use HTTP locally
- **TLS version**: 1.2+ (1.3 preferred)
- **Certificate**: Valid, not self-signed

```nginx
# ✅ CORRECT - Force HTTPS
server {
  listen 80;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;
  ssl_protocols TLSv1.2 TLSv1.3;
  # ... SSL configuration
}
```

### Security Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

## 7. File Upload Security

### Validation
- **Type**: Check MIME type AND file extension
- **Size**: Enforce maximum file size (e.g., 5MB)
- **Content**: Scan for malware (ClamAV, cloud scanners)
- **Storage**: Store outside web root, serve via CDN

```javascript
// ✅ CORRECT - Validate file upload
const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
const maxSize = 5 * 1024 * 1024  // 5MB

if (!allowedTypes.includes(file.mimetype)) {
  throw new Error('Invalid file type')
}
if (file.size > maxSize) {
  throw new Error('File too large')
}

// Store with random name, not original filename
const filename = `${uuid()}.${extension}`
```

### Dangerous Files
- **Executable**: `.exe`, `.sh`, `.bat`, `.cmd`
- **Server-side**: `.php`, `.jsp`, `.asp`
- **Archives**: May contain malicious files

## 8. API Security

### Rate Limiting
- **Per user**: 100 requests/minute
- **Per IP**: 1000 requests/minute
- **Authentication endpoints**: 5 attempts/15 minutes

```javascript
// ✅ CORRECT - Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // Limit each IP to 100 requests
  message: 'Too many requests'
})
app.use('/api/', limiter)
```

### CORS Configuration
```javascript
// ✅ CORRECT - Strict CORS
app.use(cors({
  origin: ['https://yourapp.com'],  // Specific origins only
  credentials: true,                 // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE']
}))

// ❌ WRONG - Open CORS
app.use(cors({ origin: '*' }))  // Anyone can access!
```

### API Keys
- **Prefix**: Identify key type (`sk_live_`, `pk_test_`)
- **Transmission**: Header or query param (NEVER in URL path)
- **Rotation**: Provide rotation mechanism
- **Scoping**: Limit permissions per key

## 9. Error Handling

### Safe Error Messages
```javascript
// ✅ CORRECT - Generic error
catch (error) {
  logger.error('Database error', { error, userId })  // Log details
  res.status(500).json({ error: 'Internal server error' })  // Generic to user
}

// ❌ WRONG - Expose internals
catch (error) {
  res.status(500).json({ error: error.message })  // Exposes stack trace!
}
```

### Information Disclosure
- **Avoid**: Stack traces, database errors, file paths, versions
- **Production**: Generic error messages
- **Development**: Detailed errors OK (but disable in production)

## 10. Logging & Monitoring

### Security Logging
Log these events:
- Authentication attempts (success/failure)
- Authorization failures
- Input validation failures
- Session management events (create, destroy)
- Admin actions
- Security-relevant configuration changes

```javascript
// ✅ CORRECT - Security logging
logger.security({
  event: 'login_failure',
  userId: attempt.userId,
  ip: req.ip,
  timestamp: new Date(),
  reason: 'invalid_password'
})
```

### What NOT to Log
- **Passwords** (plaintext or hashed)
- **API keys, tokens**
- **Credit card numbers**
- **Personally Identifiable Information (PII)** (unless required)

## OWASP Top 10 Coverage

| Vulnerability | Prevention |
|---------------|------------|
| **Injection** | Parameterized queries, input validation |
| **Broken Auth** | Bcrypt, secure sessions, MFA |
| **Sensitive Data** | Encryption at rest/transit, HTTPS |
| **XML External Entities** | Disable XXE in parsers |
| **Broken Access Control** | Check permissions on every request |
| **Security Misconfiguration** | Secure defaults, harden configuration |
| **XSS** | Output encoding, CSP headers |
| **Insecure Deserialization** | Validate before deserialize, use JSON |
| **Known Vulnerabilities** | Keep dependencies updated |
| **Insufficient Logging** | Log security events, monitor alerts |

## AI Self-Check

Before generating code, verify:
- [ ] All user input validated at boundaries?
- [ ] Database queries parameterized (no string concatenation)?
- [ ] Passwords hashed with bcrypt/Argon2 (NEVER plaintext)?
- [ ] HTTPS enforced in production?
- [ ] Secrets stored in environment variables (not hardcoded)?
- [ ] Output encoded for correct context (HTML, JS, URL)?
- [ ] Authentication required for protected endpoints?
- [ ] Authorization checked before actions?
- [ ] Error messages don't expose sensitive data?
- [ ] Security headers configured (CSP, HSTS, etc.)?
- [ ] Rate limiting applied to API endpoints?
- [ ] File uploads validated (type, size, content)?
- [ ] CORS configured restrictively (not `*`)?
- [ ] Security events logged (auth, authz, failures)?
- [ ] Dependencies kept up-to-date?

## Security Testing

- **SAST**: Static code analysis (SonarQube, Snyk)
- **DAST**: Dynamic testing (OWASP ZAP, Burp Suite)
- **Dependency scanning**: npm audit, Dependabot, Snyk
- **Secret scanning**: git-secrets, TruffleHog
- **Penetration testing**: Professional security audit

## Compliance Considerations

- **GDPR**: Data protection, right to deletion, consent
- **HIPAA**: Healthcare data encryption, access logging
- **PCI-DSS**: Credit card data, encryption, network segmentation
- **SOC 2**: Security controls, audit trails, access management

---

**Remember**: Security is NOT optional. Every vulnerability is a potential breach. When in doubt, choose the more secure option.

