# Manual Testing with Cursor

Test your rules directly in Cursor without API keys, Ollama, or automation scripts.

---

## ✅ **Why Manual Testing with Cursor?**

- ✅ **No setup required** - You're already using Cursor
- ✅ **No API keys** - Uses your Cursor subscription
- ✅ **No installation** - No Ollama, no dependencies
- ✅ **Interactive** - See AI responses in real-time
- ✅ **Best quality** - Cursor uses latest Claude/GPT-4
- ⚠️ **Manual effort** - You evaluate results yourself

---

## 🚀 **How to Test Manually**

### Step 1: Load Your Rules into Cursor

Your rules are **already loaded** via `.cursor/rules/*.mdc` files! Cursor automatically applies them.

**Verify** (in Cursor Composer):
1. Open Composer (`Cmd/Ctrl + I`)
2. Type: "What coding standards should I follow?"
3. Cursor should reference your rules

---

### Step 2: Run Test Prompts

Use the test prompts from `.ai-iap/TEST_PROMPTS.md`:

#### **Test 1: Spring Boot Service** (Java)

**Prompt in Cursor Composer**:
```
Create a Spring Boot service class that uses UserRepository to fetch a user by ID.
The service should handle the case where the user is not found.
```

**What to Check** ✅:
- Uses `@RequiredArgsConstructor` (NOT `@Autowired` on fields)
- Has `@Transactional(readOnly = true)`
- Has `private final UserRepository`
- Returns `UserDto` (NOT `User` entity)
- Uses `orElseThrow()` (NOT `.get()`)

**Example Good Response**:
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserService {
    private final UserRepository userRepository;
    
    public UserDto getUser(Long id) {
        return userRepository.findById(id)
            .map(UserMapper::toDto)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

**Score Yourself**:
- All 5 checks pass → 100% ✅
- 4/5 checks pass → 80% ⚠️
- 3/5 checks pass → 60% ❌

---

#### **Test 2: React Component** (TypeScript)

**Prompt in Cursor Composer**:
```
Create a React component that fetches and displays a user by ID.
The component should show a loading state while fetching and handle the case where the user is not found.
```

**What to Check** ✅:
- Functional component (NOT class component)
- PascalCase name (e.g., `UserProfile`)
- Uses `useState` and `useEffect`
- `useEffect` has `userId` in dependency array (NOT empty)
- Shows loading state
- Handles user not found

**Example Good Response**:
```tsx
interface UserProfileProps {
  userId: string;
}

const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId]);  // ← userId MUST be here
  
  if (loading) return <Loading />;
  if (!user) return <NotFound />;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

**Score Yourself**:
- All 6 checks pass → 100% ✅
- 5/6 checks pass → 83% ✅
- 4/6 checks pass → 67% ⚠️
- <4 checks pass → ❌

---

#### **Test 3: ASP.NET Core Controller** (C#)

**Prompt in Cursor Composer**:
```
Create an ASP.NET Core controller with a GET endpoint that fetches a user by ID.
Return 404 if the user is not found.
```

**What to Check** ✅:
- Has `[ApiController]` attribute
- Has `[HttpGet("{id}")]` on method
- Returns `ActionResult<UserDto>`
- Method is `async Task`
- Uses `await`
- Returns `NotFound()` for missing user
- Injects service (NOT DbContext directly)

**Example Good Response**:
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetUser(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
            return NotFound();
        
        return user;
    }
}
```

**Score Yourself**:
- All 7 checks pass → 100% ✅
- 6/7 checks pass → 86% ✅
- 5/7 checks pass → 71% ⚠️
- <5 checks pass → ❌

---

### Step 3: Record Your Results

Create a simple table:

```
Manual Test Results - [Date]

| Test | Score | Issues Found |
|------|-------|--------------|
| Spring Boot | 100% ✅ | None |
| React | 83% ✅ | Missing userId in deps initially, fixed after second attempt |
| ASP.NET | 71% ⚠️ | Used DbContext directly, need to add service layer |

Overall: 85% - Rules mostly working, need to strengthen service layer guidance
```

---

## 📋 **Full Test Suite** (from TEST_PROMPTS.md)

### Critical Tests (Recommended)
1. ✅ Spring Boot Service (Java)
2. ✅ React Component (TypeScript)
3. ✅ ASP.NET Core Controller (C#)

### Additional Tests (Optional)
4. FastAPI Endpoint (Python)
5. Next.js Server Component (TypeScript)
6. Django Class-Based View (Python)
7. Laravel Controller (PHP)
8. SwiftUI View (Swift)
9. Jetpack Compose Screen (Kotlin)
10. NestJS Service (TypeScript)

See `.ai-iap/TEST_PROMPTS.md` for all 15 test prompts.

---

## 🎯 **Tips for Manual Testing**

### 1. **Use Cursor Composer** (Not Chat)

Composer applies rules more consistently than Chat:
- Open Composer: `Cmd/Ctrl + I`
- Paste test prompt
- Review generated code

### 2. **Test in Clean Workspace**

Create a temporary folder for testing:
```bash
mkdir test-cursor-rules
cd test-cursor-rules
# Open in Cursor
```

This ensures rules are applied without project-specific context.

### 3. **Compare with Expected Patterns**

Use the "Expected Output" from `TEST_PROMPTS.md` as reference:
- Does it match the pattern?
- Are required elements present?
- Are forbidden elements absent?

### 4. **Iterate on Failed Tests**

If Cursor generates wrong code:
1. **Identify the issue** (e.g., used field injection instead of constructor)
2. **Strengthen the rule** (add more explicit `> **NEVER**` directive)
3. **Re-test** with same prompt
4. **Verify improvement**

### 5. **Test Across Languages**

Test at least one prompt per language you use:
- Java → Spring Boot
- TypeScript → React
- C# → ASP.NET Core
- Python → FastAPI
- PHP → Laravel
- Swift → SwiftUI
- Kotlin → Jetpack Compose

---

## 📊 **Evaluation Criteria**

### Pass Thresholds

| Score | Rating | Action |
|-------|--------|--------|
| **90-100%** | ✅ Excellent | Rules working great |
| **80-89%** | ✅ Good | Minor tweaks needed |
| **70-79%** | ⚠️ Fair | Strengthen weak areas |
| **<70%** | ❌ Needs Work | Review and rewrite rules |

### Cross-Language Consistency

Test the same concept across languages:

**Example: Constructor Injection**

Ask Cursor to create a service in:
- Java (Spring Boot) → Should use `@RequiredArgsConstructor`
- C# (ASP.NET) → Should use constructor injection
- TypeScript (NestJS) → Should use constructor injection
- Python (FastAPI) → Should use Dependency Injection

All should follow the same pattern (avoid field injection).

---

## 🔄 **Manual vs Automated Testing**

| Feature | Manual (Cursor) | Automated (Scripts) |
|---------|-----------------|---------------------|
| **Setup** | None ✅ | API keys or Ollama |
| **Speed** | Slow (5-10 min/test) | Fast (30 sec/test) |
| **Cost** | Included in Cursor | $0-$0.09/test |
| **Quality Check** | Manual evaluation | Automated scoring |
| **Iteration** | Interactive | Batch processing |
| **Best For** | 1-5 tests | 10+ tests, CI/CD |

**Recommendation**:
- **Manual testing** for initial rule development (you're doing it anyway in Cursor)
- **Automated testing** for validation, regression testing, CI/CD

---

## 💡 **Workflow: Manual First, Then Automate**

```
Phase 1: Manual Testing (Cursor)
↓
Write rules → Test in Cursor Composer → Iterate → Refine
↓
Phase 2: Automated Testing (Scripts)
↓
Run automated tests → Verify consistency → Set up CI/CD
```

**Why this works**:
- Manual testing lets you see AI reasoning
- Automated testing ensures consistency across AIs
- Together: Best of both worlds

---

## 🎓 **Example Session**

### 1. Test Spring Boot

```
Me: Create a Spring Boot service class that uses UserRepository 
    to fetch a user by ID. The service should handle the case 
    where the user is not found.

Cursor: [Generates code with @Service, @RequiredArgsConstructor, etc.]

Me: ✅ Perfect! Using constructor injection, returning DTO, proper error handling.
    Score: 100%
```

### 2. Test React

```
Me: Create a React component that fetches and displays a user by ID.
    The component should show a loading state while fetching and 
    handle the case where the user is not found.

Cursor: [Generates functional component but forgets userId in deps]

Me: ⚠️ Missing userId in useEffect dependency array.
    
    "Please add userId to the useEffect dependency array"

Cursor: [Updates code with [userId] in deps]

Me: ✅ Now it's correct!
    Score: 83% (needed one correction)
```

### 3. Test ASP.NET

```
Me: Create an ASP.NET Core controller with a GET endpoint 
    that fetches a user by ID. Return 404 if the user is not found.

Cursor: [Generates controller but uses DbContext directly]

Me: ❌ Using DbContext directly in controller instead of service layer.
    
    Rule needs strengthening. Updated rule:
    > **NEVER**: Inject DbContext directly into controllers. Always use a service layer.

Cursor: [Regenerates with service injection]

Me: ✅ Better!
    Score: 71% → 100% after rule update
```

**Result**: Identified weak rule, strengthened it, retested successfully.

---

## 🆘 **Troubleshooting**

### Issue: Cursor not following rules

**Solution**:
1. Check `.cursor/rules/` files are present
2. Restart Cursor
3. Use Composer (not Chat)
4. Add more explicit directives (`> **ALWAYS**`, `> **NEVER**`)

### Issue: Inconsistent results

**Solution**:
1. Test in clean workspace (no project context)
2. Use exact prompts from `TEST_PROMPTS.md`
3. Compare with multiple generations (run same prompt 2-3 times)

### Issue: Too time-consuming

**Solution**:
Switch to automated testing:
- Install Ollama (free, unlimited tests)
- Or use OpenAI API ($0.09/test, instant)

---

## 📚 **Resources**

- **Test Prompts**: `.ai-iap/TEST_PROMPTS.md` (15 complete test cases)
- **Automated Testing**: `QUICK_START_AI_TESTING.md`
- **Rules Location**: `.ai-iap/rules/`
- **Your Cursor Rules**: `.cursor/rules/*.mdc`

---

## ✅ **Quick Checklist**

- [ ] Open Cursor Composer (`Cmd/Ctrl + I`)
- [ ] Test Spring Boot prompt
- [ ] Check all 5 criteria (constructor injection, @Transactional, DTO, etc.)
- [ ] Test React prompt
- [ ] Check all 6 criteria (functional component, hooks, deps, etc.)
- [ ] Test ASP.NET prompt
- [ ] Check all 7 criteria ([ApiController], async, DTO, etc.)
- [ ] Record scores
- [ ] Identify weak areas
- [ ] Strengthen rules
- [ ] Re-test

---

**Manual testing is perfect for rule development in Cursor!**

When you're ready for automated testing, regression testing, or CI/CD → Use the scripts with Ollama (free) or OpenAI API (paid).

🎯 **Best of both worlds**: Develop rules manually in Cursor, validate automatically with scripts.

