# Expanding the Test Suite

How to add more tests from `.ai-iap/TEST_PROMPTS.md` to the automated test runner.

---

## 📋 Available Test Cases (from TEST_PROMPTS.md)

Currently, the test runner implements **5 of 15** test cases. Here are the remaining 10:

### ✅ Implemented (5/15)

1. ✅ Spring Boot Service (Test 1)
2. ✅ React Component (Test 2)
3. ✅ ASP.NET Core Controller (Test 3)
4. ✅ FastAPI Endpoint (Test 4)
5. ✅ Next.js Server Component (Test 5)

### 🔨 Ready to Implement (10/15)

6. **Django View** - Class-based views, serializers, permissions
7. **Laravel Controller** - Action classes, validation, resources
8. **SwiftUI View** - @State, @Binding, MVVM patterns
9. **Jetpack Compose Screen** - ViewModel, State hoisting
10. **NestJS Service** - Dependency injection, decorators
11. **Express.js Route** - Async/await, error handling
12. **Spring Data JPA Repository** - Derived queries, @Query
13. **Entity Framework Query** - LINQ, Include(), AsNoTracking()
14. **React Hook** - Custom hooks, dependency array
15. **Python Async Function** - async/await, asyncio patterns

---

## 🔧 Adding a New Test

### Step 1: Define Test in `test-ai.js`

Open `.github/scripts/test-ai.js` and add to `testDefinitions`:

```javascript
'django': {
  id: 'test-6-django',
  name: 'Django Class-Based View',
  language: 'python',
  framework: 'django',
  prompt: `Create a Django class-based view that lists all users with pagination. Use Django REST Framework serializers.`,
  expectedPatterns: [
    'class.*ListAPIView',
    'serializer_class\\s*=',
    'queryset\\s*=',
    'pagination_class',
    'permission_classes',
    'UserSerializer'
  ],
  forbiddenPatterns: [
    'def\\s+get\\(self',  // Should use ListAPIView, not custom get()
    '\\.objects\\.all\\(\\)(?!.*select_related)',  // Missing select_related for performance
    'return\\s+Response\\('  // ListAPIView handles response
  ],
  rules: [
    'rules/general/persona.md',
    'rules/general/architecture.md',
    'rules/general/code-style.md',
    'rules/python/architecture.md',
    'rules/python/code-style.md',
    'rules/python/frameworks/django.md'
  ]
},
```

### Step 2: Add to Test Suites

```javascript
const testSuites = {
  critical: ['spring-boot', 'react', 'aspnet'],
  all: ['spring-boot', 'react', 'aspnet', 'fastapi', 'nextjs', 'django'],  // Add here
  'django': ['django']  // Add individual suite
};
```

### Step 3: Test Locally

```bash
cd .github/scripts
node test-ai.js --provider openai --model gpt-4 --test-suite django
```

### Step 4: Review Results

Check `test-results/*.json` for:
- Score (should be ≥90)
- Expected patterns found/missing
- Forbidden patterns avoided/found

### Step 5: Iterate on Patterns

If score is low:
1. Check which patterns are missing → strengthen rules
2. Check which forbidden patterns are found → add more `> **NEVER**` directives
3. Re-run test to verify improvement

---

## 📖 Test Pattern Examples

### Example 1: Laravel Controller Test

```javascript
'laravel': {
  id: 'test-7-laravel',
  name: 'Laravel Controller',
  language: 'php',
  framework: 'laravel',
  prompt: `Create a Laravel controller that stores a new user with validation.`,
  expectedPatterns: [
    'class\\s+\\w+Controller',
    'public\\s+function\\s+store',
    'StoreUserRequest',  // Form Request for validation
    'UserResource',      // API Resource for output
    'action->execute',   // Action class for logic
    'return\\s+new\\s+UserResource'
  ],
  forbiddenPatterns: [
    'User::create\\(',              // Logic in controller
    '\\$request->all\\(\\)',        // No validation
    'return\\s+\\$user(?!.*Resource)'  // Returning model instead of resource
  ],
  rules: [
    'rules/general/persona.md',
    'rules/general/architecture.md',
    'rules/general/code-style.md',
    'rules/php/architecture.md',
    'rules/php/code-style.md',
    'rules/php/frameworks/laravel.md'
  ]
}
```

### Example 2: SwiftUI View Test

```javascript
'swiftui': {
  id: 'test-8-swiftui',
  name: 'SwiftUI View',
  language: 'swift',
  framework: 'swiftui',
  prompt: `Create a SwiftUI view that displays a user profile with edit functionality. Use MVVM pattern.`,
  expectedPatterns: [
    'struct\\s+\\w+View:\\s+View',
    '@StateObject\\s+.*ViewModel',
    '@State\\s+private\\s+var',
    '@Binding\\s+var',
    'var\\s+body:\\s+some\\s+View',
    '\\.onAppear'
  ],
  forbiddenPatterns: [
    '@ObservedObject\\s+.*viewModel(?!:)',  // Should be @StateObject for owner
    '@State.*let\\s',                        // @State must be var, not let
    'DispatchQueue\\.main\\.async'          // Should use @MainActor
  ],
  rules: [
    'rules/general/persona.md',
    'rules/general/architecture.md',
    'rules/swift/architecture.md',
    'rules/swift/code-style.md',
    'rules/swift/frameworks/swiftui.md'
  ]
}
```

### Example 3: Jetpack Compose Test

```javascript
'compose': {
  id: 'test-9-compose',
  name: 'Jetpack Compose Screen',
  language: 'kotlin',
  framework: 'compose',
  prompt: `Create a Jetpack Compose screen that displays a user list with pull-to-refresh. Use ViewModel and State hoisting.`,
  expectedPatterns: [
    '@Composable',
    'fun\\s+\\w+Screen',
    'val\\s+viewModel:\\s+.*ViewModel\\s+=\\s+hiltViewModel',
    'val\\s+\\w+\\s+by\\s+viewModel\\.\\w+\\.collectAsState',
    'LazyColumn',
    'PullRefreshIndicator'
  ],
  forbiddenPatterns: [
    'var\\s+.*by\\s+remember\\s*\\{\\s*mutableStateOf',  // Should be in ViewModel
    'viewModelScope\\.launch\\s*\\{(?!.*Dispatchers)',    // Missing dispatcher
    '\\.value\\s*='  // Mutating state directly instead of using ViewModel
  ],
  rules: [
    'rules/general/persona.md',
    'rules/general/architecture.md',
    'rules/kotlin/architecture.md',
    'rules/kotlin/code-style.md',
    'rules/kotlin/frameworks/compose.md'
  ]
}
```

---

## 🎯 Pattern Matching Tips

### Use Regex Patterns

```javascript
// Match @Service annotation
'@Service'

// Match constructor injection (final field)
'private\\s+final\\s+\\w+Repository'

// Match async function
'async\\s+def\\s+\\w+'

// Match useState with type
'useState<\\w+>'

// Match class extends
'class\\s+\\w+\\s+extends\\s+Component'
```

### Match Absence of Pattern

```javascript
// Forbidden: Direct field injection
forbiddenPatterns: [
  '@Autowired\\s+private\\s+'
]

// Forbidden: Returning entities
forbiddenPatterns: [
  'return.*User[^D]'  // Returns User not UserDto
]

// Forbidden: Sync function when should be async
forbiddenPatterns: [
  '^def\\s+(?!.*async)'  // def without async
]
```

### Match Context-Aware Patterns

```javascript
// Ensure userId is in dependency array
'userId.*\\]'

// Ensure orElseThrow is used
'\\.(orElseThrow|orElse)'

// Ensure proper error handling
'ActionResult<\\w+Dto>'
```

---

## 📊 Quality Thresholds

### Scoring

- **Expected patterns**: 70% of total score
- **Forbidden patterns**: 30% of total score

### Pass Criteria

- **Individual test**: ≥90/100
- **Overall pass rate**: ≥90% of tests
- **Cross-AI variance**: ≤10 points

### Example Calculation

Test has:
- 7 expected patterns
- 3 forbidden patterns

AI output:
- 6/7 expected patterns found → (6/7) * 70 = 60 points
- 3/3 forbidden patterns avoided → (3/3) * 30 = 30 points
- **Total**: 90/100 → **PASS** ✅

---

## 🔍 Debugging Failed Tests

### Step 1: Check Test Results

```bash
cat test-results/openai-gpt-4-*.json | jq '.tests[] | select(.passed == false)'
```

### Step 2: Review Missing Patterns

```json
{
  "expectedMissing": ["@Transactional(readOnly = true)"],
  "forbiddenFound": ["@Autowired"]
}
```

**Action**: AI didn't add `@Transactional` and used field injection

### Step 3: Strengthen Rules

In `.ai-iap/rules/java/frameworks/spring-boot.md`:

**Before**:
```markdown
- Use constructor injection for dependencies
```

**After**:
```markdown
> **ALWAYS**: Use constructor injection with `@RequiredArgsConstructor` and `private final` fields
> **NEVER**: Use field injection with `@Autowired` on fields
```

### Step 4: Re-test

```bash
node test-ai.js --provider openai --model gpt-4 --test-suite spring-boot
```

---

## 📈 Expanding Test Coverage

### Current Coverage (5 tests)

- ✅ Backend: Spring Boot, ASP.NET, FastAPI (3/8 languages)
- ✅ Frontend: React, Next.js (2/8 languages)
- ❌ Mobile: 0/3 (iOS, Android, Flutter)

### Recommended Next Tests (Prioritized)

**Priority 1: Mobile Coverage**
1. SwiftUI View (iOS)
2. Jetpack Compose Screen (Android)
3. Flutter Widget (Dart)

**Priority 2: Backend Coverage**
4. Django View (Python)
5. Laravel Controller (PHP)
6. NestJS Service (TypeScript/Node.js)

**Priority 3: Framework-Specific**
7. Spring Data JPA Repository
8. Entity Framework Query
9. Custom React Hook

**Priority 4: Advanced Patterns**
10. Python Async Function
11. Express.js Middleware
12. Kotlin Coroutines

---

## 🎓 Best Practices

1. **Start with critical tests**: Backend + Frontend (currently have 5)
2. **Cover all languages**: Ensure each of 8 languages has ≥1 test
3. **Test architectural patterns**: DI, layering, error handling, async
4. **Test common mistakes**: Field injection, exposing entities, missing await
5. **Keep patterns specific**: Too broad = false positives

---

## 📚 Resources

- **Test definitions source**: `.ai-iap/TEST_PROMPTS.md`
- **Rule files**: `.ai-iap/rules/`
- **Test runner**: `.github/scripts/test-ai.js`
- **Results analyzer**: `.github/scripts/analyze-results.js`

---

**Next**: Run `node test-ai.js --help` for all options

