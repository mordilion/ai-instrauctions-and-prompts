# TypeScript Security

> **Scope**: TypeScript-specific security practices (frontend and backend)
> **Extends**: general/security.md
> **Applies to**: *.ts, *.tsx files

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use strict type checking (`strict: true` in tsconfig.json)
> **ALWAYS**: Sanitize user input before rendering in DOM
> **ALWAYS**: Use Content Security Policy for web applications
> **ALWAYS**: Validate environment variables at startup
> **NEVER**: Use `dangerouslySetInnerHTML` without sanitization (React)
> **NEVER**: Disable TypeScript strict mode
> **NEVER**: Use `eval()` or `Function()` constructor with user input
> **NEVER**: Trust data from localStorage/sessionStorage

## 1. Frontend Security (React/Angular/Vue)

### XSS Prevention

❌ **WRONG - Direct HTML insertion**:
```typescript
// React
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// Angular
<div [innerHTML]="userInput"></div>

// Vanilla JS
element.innerHTML = userInput
```

✅ **CORRECT - Sanitized or escaped**:
```typescript
// React - Use text content
<div>{userInput}</div>  // Automatically escaped

// React - Sanitize if HTML needed
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />

// Angular - Sanitize
import { DomSanitizer } from '@angular/platform-browser'
const safe = sanitizer.sanitize(SecurityContext.HTML, userInput)
```

### Content Security Policy (CSP)

```typescript
// ✅ CORRECT - Strict CSP
const helmet = require('helmet')

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'nonce-{random}'"],
    styleSrc: ["'self'", "'nonce-{random}'"],
    imgSrc: ["'self'", "data:", "https:"],
    connectSrc: ["'self'", "https://api.yourapp.com"],
    fontSrc: ["'self'"],
    objectSrc: ["'none'"],
    mediaSrc: ["'self'"],
    frameSrc: ["'none'"]
  }
}))
```

### Client-Side Storage

```typescript
// ❌ WRONG - Sensitive data in localStorage
localStorage.setItem('token', jwtToken)  // Accessible to XSS!
localStorage.setItem('password', password)  // NEVER!

// ✅ CORRECT - Use HttpOnly cookies for tokens
// Server sets cookie:
res.cookie('token', jwtToken, {
  httpOnly: true,     // No JS access
  secure: true,       // HTTPS only
  sameSite: 'strict'  // CSRF protection
})

// ✅ OK - Non-sensitive data in localStorage
localStorage.setItem('theme', 'dark')
localStorage.setItem('language', 'en')
```

### URL Handling

```typescript
// ❌ WRONG - Open redirect
const redirectUrl = req.query.redirect as string
res.redirect(redirectUrl)  // Can redirect to malicious site!

// ✅ CORRECT - Validate redirect URL
const allowedDomains = ['yourapp.com', 'staging.yourapp.com']
const url = new URL(redirectUrl)
if (allowedDomains.includes(url.hostname)) {
  res.redirect(redirectUrl)
} else {
  res.redirect('/home')
}
```

## 2. Backend Security (Node.js/Express/NestJS)

### Environment Variables

```typescript
// ✅ CORRECT - Validate env vars at startup
import { z } from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  API_KEY: z.string().min(20),
  PORT: z.string().regex(/^\d+$/),
  NODE_ENV: z.enum(['development', 'staging', 'production'])
})

const env = envSchema.parse(process.env)

// ❌ WRONG - No validation
const dbUrl = process.env.DATABASE_URL!  // Might be undefined!
```

### SQL Injection Prevention

```typescript
// ❌ WRONG - String concatenation
const query = `SELECT * FROM users WHERE email = '${email}'`
db.query(query)

// ✅ CORRECT - Parameterized query
const query = 'SELECT * FROM users WHERE email = ?'
db.query(query, [email])

// ✅ CORRECT - ORM (TypeORM, Prisma)
const user = await userRepository.findOne({ where: { email } })
const user = await prisma.user.findUnique({ where: { email } })
```

### Input Validation

```typescript
// ✅ CORRECT - Using Zod
import { z } from 'zod'

const createUserSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(12).max(128),
  name: z.string().min(2).max(100).regex(/^[a-zA-Z\s]+$/),
  age: z.number().int().min(18).max(120)
})

app.post('/users', async (req, res) => {
  const data = createUserSchema.parse(req.body)  // Throws if invalid
  const user = await createUser(data)
  res.json(user)
})

// ✅ CORRECT - Using class-validator (NestJS)
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator'

export class CreateUserDto {
  @IsEmail()
  @MaxLength(255)
  email: string

  @IsString()
  @MinLength(12)
  @MaxLength(128)
  password: string
}
```

### Authentication

```typescript
// ✅ CORRECT - Bcrypt password hashing
import bcrypt from 'bcrypt'

const SALT_ROUNDS = 12

async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS)
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash)
}

// ❌ WRONG - Weak hashing
const hash = crypto.createHash('md5').update(password).digest('hex')
```

### JWT Security

```typescript
// ✅ CORRECT - Secure JWT
import jwt from 'jsonwebtoken'

const JWT_SECRET = process.env.JWT_SECRET!
const JWT_EXPIRY = '15m'

interface TokenPayload {
  userId: string
  role: string
  // NEVER include: password, email, sensitive data
}

function createToken(payload: TokenPayload): string {
  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRY,
    algorithm: 'HS256'
  })
}

function verifyToken(token: string): TokenPayload {
  return jwt.verify(token, JWT_SECRET, {
    algorithms: ['HS256']
  }) as TokenPayload
}

// ❌ WRONG - No expiration, algorithm: 'none'
const token = jwt.sign({ userId }, JWT_SECRET, { algorithm: 'none' })
```

### Rate Limiting

```typescript
// ✅ CORRECT - Express rate limiter
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // Limit each IP to 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
  message: 'Too many requests, please try again later'
})

app.use('/api/', limiter)

// Stricter for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,  // Only 5 login attempts per 15 minutes
  skipSuccessfulRequests: true
})

app.post('/api/auth/login', authLimiter, loginHandler)
```

### CORS Configuration

```typescript
// ✅ CORRECT - Restrictive CORS
import cors from 'cors'

const allowedOrigins = [
  'https://yourapp.com',
  'https://staging.yourapp.com'
]

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error('Not allowed by CORS'))
    }
  },
  credentials: true,  // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}))

// ❌ WRONG - Open CORS
app.use(cors({ origin: '*' }))
```

## 3. TypeScript-Specific Security

### Strict Type Checking

```json
// ✅ CORRECT - tsconfig.json
{
  "compilerOptions": {
    "strict": true,                      // Enable all strict options
    "noImplicitAny": true,               // No implicit 'any'
    "strictNullChecks": true,            // Null safety
    "strictFunctionTypes": true,         // Function type checking
    "strictPropertyInitialization": true, // Class property initialization
    "noImplicitThis": true,              // 'this' type checking
    "alwaysStrict": true,                // Use 'use strict'
    "noUnusedLocals": true,              // Report unused variables
    "noUnusedParameters": true,          // Report unused parameters
    "noImplicitReturns": true,           // Report missing returns
    "noFallthroughCasesInSwitch": true   // Report switch fallthrough
  }
}
```

### Type Guards for Security

```typescript
// ✅ CORRECT - Type guards for runtime validation
function isValidUserId(id: unknown): id is string {
  return typeof id === 'string' && id.length > 0 && /^[a-zA-Z0-9-]+$/.test(id)
}

app.get('/users/:id', (req, res) => {
  const { id } = req.params
  
  if (!isValidUserId(id)) {
    return res.status(400).json({ error: 'Invalid user ID' })
  }
  
  // Safe to use 'id' here
  const user = await getUser(id)
  res.json(user)
})

// ❌ WRONG - Trusting route params
app.get('/users/:id', (req, res) => {
  const user = await getUser(req.params.id)  // No validation!
})
```

### Avoid `any` Type

```typescript
// ❌ WRONG - Using 'any'
function processData(data: any) {
  return data.someProperty  // No type safety!
}

// ✅ CORRECT - Proper typing
interface UserData {
  id: string
  email: string
  name: string
}

function processData(data: UserData) {
  return data.name  // Type-safe
}

// ✅ CORRECT - Unknown for truly unknown data
function processData(data: unknown) {
  if (typeof data === 'object' && data !== null && 'name' in data) {
    return (data as { name: string }).name
  }
  throw new Error('Invalid data')
}
```

## 4. Dependency Security

### Regular Updates

```bash
# Check for vulnerabilities
npm audit
npm audit fix

# Alternative: Snyk
npx snyk test
```

### Lock File Integrity

```bash
# Always commit package-lock.json or yarn.lock
git add package-lock.json
git commit -m "Update dependencies"

# Verify integrity
npm ci  # Uses lockfile, doesn't modify it
```

### Avoid Dangerous Packages

```typescript
// ❌ DANGEROUS - eval-like packages
const vm = require('vm')
const safeEval = require('safe-eval')  // Still risky

// ✅ SAFER - Use JSON for data, not code
const data = JSON.parse(userInput)
```

## 5. API Security

### Authentication Middleware

```typescript
// ✅ CORRECT - Auth middleware
import { Request, Response, NextFunction } from 'express'

interface AuthRequest extends Request {
  user?: { id: string; role: string }
}

async function authenticate(
  req: AuthRequest,
  res: Response,
  next: NextFunction
) {
  const token = req.headers.authorization?.replace('Bearer ', '')
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' })
  }
  
  try {
    const payload = verifyToken(token)
    req.user = payload
    next()
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' })
  }
}

// Use
app.get('/api/protected', authenticate, protectedHandler)
```

### Authorization

```typescript
// ✅ CORRECT - Role-based authorization
function authorize(...roles: string[]) {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' })
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Not authorized' })
    }
    
    next()
  }
}

// Use
app.delete('/api/users/:id', 
  authenticate, 
  authorize('admin'), 
  deleteUserHandler
)
```

## 6. Error Handling

```typescript
// ✅ CORRECT - Safe error handling
import { Request, Response, NextFunction } from 'express'

class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message)
  }
}

function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Log error with details
  logger.error('Error occurred', {
    error: error.message,
    stack: error.stack,
    url: req.url,
    method: req.method,
    ip: req.ip
  })
  
  // Return generic error to client
  if (error instanceof AppError) {
    res.status(error.statusCode).json({
      error: error.message
    })
  } else {
    res.status(500).json({
      error: 'Internal server error'  // Generic message
    })
  }
}

app.use(errorHandler)
```

## 7. File Upload Security

```typescript
// ✅ CORRECT - Secure file upload (using multer)
import multer from 'multer'
import path from 'path'
import crypto from 'crypto'

const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif']
const MAX_SIZE = 5 * 1024 * 1024  // 5MB

const storage = multer.diskStorage({
  destination: './uploads/',
  filename: (req, file, cb) => {
    // Random name to prevent path traversal
    const randomName = crypto.randomBytes(16).toString('hex')
    const ext = path.extname(file.originalname)
    cb(null, `${randomName}${ext}`)
  }
})

const upload = multer({
  storage,
  limits: { fileSize: MAX_SIZE },
  fileFilter: (req, file, cb) => {
    if (!ALLOWED_TYPES.includes(file.mimetype)) {
      cb(new Error('Invalid file type'))
    } else {
      cb(null, true)
    }
  }
})

app.post('/upload', upload.single('file'), (req, res) => {
  res.json({ filename: req.file?.filename })
})
```

## AI Self-Check

Before generating TypeScript code, verify:
- [ ] `strict: true` in tsconfig.json?
- [ ] No `dangerouslySetInnerHTML` without DOMPurify?
- [ ] All user input validated (Zod, class-validator)?
- [ ] Database queries parameterized (no string templates)?
- [ ] Passwords hashed with bcrypt (12+ rounds)?
- [ ] JWT tokens have expiration (`expiresIn`)?
- [ ] Environment variables validated at startup?
- [ ] Rate limiting on API endpoints?
- [ ] CORS configured restrictively (not `*`)?
- [ ] No secrets hardcoded in source?
- [ ] Error messages don't expose internals?
- [ ] File uploads validated (type, size)?
- [ ] Authentication middleware on protected routes?
- [ ] Authorization checks before actions?
- [ ] No `eval()` or `Function()` with user input?

## Testing Security

```typescript
// ✅ Security testing example (Jest)
describe('Authentication', () => {
  it('should reject invalid tokens', async () => {
    const response = await request(app)
      .get('/api/protected')
      .set('Authorization', 'Bearer invalid-token')
    
    expect(response.status).toBe(401)
  })
  
  it('should prevent SQL injection', async () => {
    const malicious = "admin'; DROP TABLE users--"
    const response = await request(app)
      .post('/api/login')
      .send({ username: malicious, password: 'test' })
    
    expect(response.status).not.toBe(500)  // Should handle gracefully
  })
  
  it('should rate limit requests', async () => {
    // Make 6 requests quickly
    for (let i = 0; i < 6; i++) {
      await request(app).post('/api/login').send({ ... })
    }
    
    const response = await request(app).post('/api/login')
    expect(response.status).toBe(429)  // Too many requests
  })
})
```

---

**Security is a continuous process, not a one-time task. Keep dependencies updated, monitor logs, and stay informed about new vulnerabilities.**

