# AI Compatibility Testing System

Automated testing system to verify that all AI models produce consistent, high-quality code when using your rule files.

---

## 🎯 **Purpose**

Ensure that GPT-4, Claude, Gemini, Codestral, and other AIs generate **architecturally equivalent** code that follows your rules.

**Success Criteria** (from `.cursor/rules/general.mdc`):
- ✅ Consistent code structure and architecture patterns
- ✅ Same security practices applied (OWASP Top 10, input validation)
- ✅ Same testing approaches (unit, integration, mocking)
- ✅ Same CI/CD best practices (version detection, caching)
- ✅ Not byte-for-byte identical, but architecturally equivalent

---

## 📦 **Installation**

```bash
cd .github/scripts
npm install
```

---

## 🚀 **Usage**

### Test GPT-4

```bash
node test-ai.js \
  --provider openai \
  --model gpt-4-turbo-preview \
  --test-suite critical

# Requires: OPENAI_API_KEY environment variable
```

### Test Claude

```bash
node test-ai.js \
  --provider anthropic \
  --model claude-3-5-sonnet-20241022 \
  --test-suite critical

# Requires: ANTHROPIC_API_KEY environment variable
```

### Test Gemini

```bash
node test-ai.js \
  --provider google \
  --model gemini-1.5-pro \
  --test-suite critical

# Requires: GOOGLE_API_KEY environment variable
```

### Test Codestral (Mistral)

```bash
node test-ai.js \
  --provider mistral \
  --model codestral-latest \
  --test-suite critical

# Requires: MISTRAL_API_KEY environment variable
```

---

## 🧪 **Available Test Suites**

| Suite | Tests | Description |
|-------|-------|-------------|
| **critical** | 3 tests | Spring Boot, React, ASP.NET Core (fastest, recommended for CI) |
| **all** | 5 tests | All available tests (more comprehensive) |
| **spring-boot** | 1 test | Spring Boot service only |
| **react** | 1 test | React component only |
| **aspnet** | 1 test | ASP.NET Core controller only |
| **fastapi** | 1 test | FastAPI endpoint only |
| **nextjs** | 1 test | Next.js server component only |

---

## 📊 **Test Scoring**

Each test is scored out of 100:
- **70 points**: Expected patterns found (constructor injection, DTOs, proper error handling, etc.)
- **30 points**: Forbidden patterns avoided (field injection, exposing entities, etc.)

**Pass threshold**: 90/100

**Overall pass rate**: Must be ≥90% to pass CI

---

## 📝 **Test Results**

Results are saved to `test-results/` directory:

```json
{
  "model": "gpt-4-turbo-preview",
  "provider": "openai",
  "testSuite": "critical",
  "totalTests": 3,
  "passed": 3,
  "failed": 0,
  "averageScore": 95,
  "passRate": 100,
  "tests": [
    {
      "testId": "test-1-spring-boot",
      "testName": "Spring Boot Service",
      "score": 95,
      "passed": true,
      "validation": {
        "expectedMatches": [...],
        "expectedMissing": [],
        "forbiddenFound": [],
        "forbiddenMissing": [...]
      }
    }
  ]
}
```

---

## 🔍 **What Gets Tested**

### Test 1: Spring Boot Service ⭐ CRITICAL

**Prompt**: Create a Spring Boot service class that uses UserRepository to fetch a user by ID.

**Expected Patterns**:
- ✅ `@RequiredArgsConstructor` (constructor injection, NOT field injection)
- ✅ `@Transactional(readOnly = true)`
- ✅ `private final UserRepository` (immutable dependencies)
- ✅ Returns `UserDto` (NOT entity)
- ✅ `orElseThrow` (proper error handling)

**Forbidden Patterns**:
- ❌ `@Autowired` on fields
- ❌ Returning `User` entity
- ❌ `.get()` without error handling

### Test 2: React Component ⭐ CRITICAL

**Prompt**: Create a React component that fetches and displays a user by ID.

**Expected Patterns**:
- ✅ Functional component (NOT class component)
- ✅ PascalCase component name
- ✅ `useEffect` with `userId` in dependency array
- ✅ Loading and error states

**Forbidden Patterns**:
- ❌ Class components
- ❌ Empty dependency array when should have `userId`
- ❌ camelCase component name

### Test 3: ASP.NET Core Controller ⭐ CRITICAL

**Prompt**: Create an ASP.NET Core controller with a GET endpoint.

**Expected Patterns**:
- ✅ `[ApiController]` attribute
- ✅ `ActionResult<UserDto>` return type
- ✅ `async Task` (async/await pattern)
- ✅ Returns DTOs (NOT entities)

**Forbidden Patterns**:
- ❌ Returning entities
- ❌ Try-catch in controller (should be in middleware)
- ❌ Direct database access (`_context.`)

### Test 4: FastAPI Endpoint

**Prompt**: Create a FastAPI endpoint that creates a new user.

**Expected Patterns**:
- ✅ Pydantic models with `EmailStr`
- ✅ `@app.post` decorator
- ✅ `response_model=` specified
- ✅ `status_code=201`
- ✅ `async def` (async functions)

### Test 5: Next.js Server Component

**Prompt**: Create a Next.js page that displays a list of users.

**Expected Patterns**:
- ✅ `async function` (Server Component)
- ✅ `await` for data fetching
- ✅ No `'use client'` directive
- ✅ No `useState` / `useEffect`

---

## 💰 **Cost Management**

### Estimated Costs (per test run)

| Provider | Model | Cost per Test | Critical Suite (3 tests) |
|----------|-------|---------------|--------------------------|
| OpenAI | GPT-4 Turbo | ~$0.03 | ~$0.09 |
| OpenAI | GPT-3.5 Turbo | ~$0.001 | ~$0.003 |
| Anthropic | Claude 3.5 Sonnet | ~$0.015 | ~$0.045 |
| Google | Gemini 1.5 Pro | ~$0.005 | ~$0.015 |
| Mistral | Codestral | ~$0.01 | ~$0.03 |

**Monthly Budget Estimate** (1 CI run per day):
- Critical suite only: ~$0.20/month
- All 5 tests: ~$0.35/month
- Testing all 4 providers: ~$1.20/month

### Cost Optimization Strategies

1. **Use Critical Suite Only**: Run only Spring Boot, React, ASP.NET tests
2. **Run on Schedule**: Weekly instead of every PR
3. **Use GPT-3.5 First**: Quick validation, then GPT-4 for detailed checks
4. **Sample Testing**: Test 1-2 AIs per run, rotate weekly
5. **Manual Triggers**: Only run when rules change

---

## 🔄 **CI/CD Integration**

See `../.github/workflows/ai-compatibility-tests.yml` (create this file)

---

## 📈 **Interpreting Results**

### ✅ Good Results (Pass)
```
Tests: 3
Passed: 3
Failed: 0
Average Score: 95/100
Pass Rate: 100%
```
**Action**: Rules are working consistently across AIs

### ⚠️ Mixed Results
```
Tests: 3
Passed: 2
Failed: 1
Average Score: 87/100
Pass Rate: 67%
```
**Action**: Investigate failed test, refine rules for that framework

### ❌ Poor Results (Fail)
```
Tests: 3
Passed: 1
Failed: 2
Average Score: 62/100
Pass Rate: 33%
```
**Action**: Rules unclear or conflicting. Review and strengthen directives.

---

## 🛠️ **Adding New Tests**

1. **Add to `test-ai.js`**:

```javascript
'laravel': {
  id: 'test-6-laravel',
  name: 'Laravel Controller',
  language: 'php',
  framework: 'laravel',
  prompt: `Create a Laravel controller that stores a new user with validation.`,
  expectedPatterns: [
    'class.*Controller',
    'public function store',
    'StoreUserRequest',
    'UserResource',
    'action->execute',
    'return new UserResource'
  ],
  forbiddenPatterns: [
    'User::create',  // Logic in controller
    '\\$request->all\\(\\)',  // No validation
  ],
  rules: [
    'rules/general/persona.md',
    'rules/general/architecture.md',
    'rules/php/architecture.md',
    'rules/php/frameworks/laravel.md'
  ]
}
```

2. **Add to test suites**:

```javascript
const testSuites = {
  critical: ['spring-boot', 'react', 'aspnet'],
  all: ['spring-boot', 'react', 'aspnet', 'fastapi', 'nextjs', 'laravel'],
  'laravel': ['laravel']
};
```

3. **Test it**:

```bash
node test-ai.js --provider openai --model gpt-4 --test-suite laravel
```

---

## 🐛 **Troubleshooting**

### Error: "API key not found"

```bash
# Set API keys in environment
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export GOOGLE_API_KEY="..."
export MISTRAL_API_KEY="..."
```

### Error: "Rule file not found"

Check that you're running from project root:
```bash
cd /path/to/ai-instructions-and-prompts
node .github/scripts/test-ai.js ...
```

### Score lower than expected

Check `test-results/*.json` for details:
- `expectedMissing`: Patterns AI should have generated but didn't
- `forbiddenFound`: Anti-patterns AI generated

**Action**: Strengthen rules with more explicit `> **ALWAYS**` / `> **NEVER**` directives

---

## 📚 **Related Files**

- **Test Definitions**: `.ai-iap/TEST_PROMPTS.md` (15 manual test cases)
- **Rules**: `.ai-iap/rules/` (the source of truth)
- **Config**: `.ai-iap/config.json` (framework/language definitions)

---

## 🎯 **Best Practices**

1. **Run locally first**: Test changes before committing
2. **Review failures**: Don't just re-run, understand why AI failed
3. **Iterate on rules**: Strengthen directives based on test failures
4. **Document patterns**: Add common failures to TEST_PROMPTS.md
5. **Version testing**: Test major AI model updates (GPT-4 → GPT-4 Turbo)

---

## 📊 **Success Metrics**

Your goal (from `.cursor/rules/general.mdc`):
> "All rules and processes **MUST** be understandable the same way for all AI's - The goal is to have the same result no matter which AI is used"

**Target Metrics**:
- ✅ Pass rate: ≥90% across all tested AIs
- ✅ Average score: ≥92/100
- ✅ Cross-AI variance: <10% (all AIs within 10 points of each other)

---

**Next Steps**: See main README for GitHub Actions integration

