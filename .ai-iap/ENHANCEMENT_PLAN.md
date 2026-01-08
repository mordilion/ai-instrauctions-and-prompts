# On-Demand Process Enhancement Plan

## Goal

Make all 69 on-demand process files comprehensive and self-contained (like dotnet/test-implementation.md).

## Current Status

- ✅ **1 file complete**: dotnet/test-implementation.md (661 lines, comprehensive)
- ⏳ **68 files remaining**

## Architecture Decision

**Keep permanent rules lean (NO changes)**:
- Framework rules stay small
- No testing/logging additions
- Maintains 85% token savings

**Make on-demand comprehensive (ALL content)**:
- Include best practices
- Include setup instructions
- Include code examples
- Include everything in ONE prompt
- 500-700 lines per file is OK (only copied once)

## Pattern: Comprehensive On-Demand File

### Structure

```markdown
# [Process Name] - Copy This Prompt

> **Type**: One-time setup process
> **When to use**: [scenario]
> **Instructions**: Copy the entire prompt below

---

## 📋 Complete Self-Contained Prompt

```
========================================
[PROCESS NAME IN CAPS]
========================================

CONTEXT:
[What user is implementing]

CRITICAL REQUIREMENTS:
- ALWAYS detect version from [config file]
- ALWAYS match version in Docker/CI/CD
- NEVER [anti-patterns]
- Use team's Git workflow

========================================
TECH STACK SELECTION
========================================

[TOOL/FRAMEWORK 1] (choose one):

1. [Option A] ⭐ RECOMMENDED
   - [Why it's good]
   - [When to use]
   - [Pros]
   Install: [command]
   Example: [code]

2. [Option B]
   - [Why it's good]
   - [When to use]
   Install: [command]

3. [Option C]
   - [When to use]

[TOOL/FRAMEWORK 2] (choose one):
[Same structure]

========================================
INFRASTRUCTURE SETUP
========================================

DOCKERFILE ([filename]):
```[language]
[Complete example with version placeholders]
```

DOCKER-COMPOSE ([filename]):
```yaml
[Complete example]
```

CI/CD INTEGRATION ([platform]):
```yaml
[Complete example]
```

========================================
PHASE 1 - [NAME]
========================================

Objective: [What this phase achieves]

Steps:
1. [Detailed step]
   - [Sub-step]
   - [Sub-step with code example]
   
2. [Next step]
   - [Details]
   
[... all steps]

Deliverable: [What user gets]

Example [documentation]:
```
[Complete template]
```

========================================
PHASE 2 - [NAME]
========================================

[Same structure]

========================================
PHASE 3 - [NAME]
========================================

[Same structure]

========================================
PHASE 4 - [NAME] (Iterative)
========================================

For EACH component:
1. [Step]
2. [Step with example]
   ```[language]
   [Complete code example]
   ```
3. [Step]

[All iterative steps]

========================================
DOCUMENTATION
========================================

Create these files in process-docs/:

1. [FILE 1]:
```markdown
[Complete template]
```

2. [FILE 2]:
```markdown
[Complete template]
```

========================================
EXECUTION INSTRUCTIONS
========================================

START HERE:

1. Execute Phase 1
   - [Action]
   - [Action]
   - Report findings

2. Execute Phase 2
   - [Action]

[... all phases]

REMEMBER:
- [Key point]
- [Key point]

BEGIN: [Starting instruction]
```

---

## 📌 Quick Reference

**What this prompt does**: [Summary]
**When to use**: [Scenario]
**What you'll get**: [Outcomes]
```

### Key Elements

1. **One markdown code block with entire prompt**
2. **Comprehensive tech stack section** (tools, frameworks, comparisons)
3. **Infrastructure templates** (Docker, CI/CD, config files)
4. **All phases with complete steps** (not summaries)
5. **Code examples throughout** (not references)
6. **Documentation templates** (complete, not partial)
7. **Execution instructions** (clear start-to-finish)
8. **Quick reference** (TL;DR at end)

## Files to Update (68 total)

### Category 1: test-implementation (7 files)

Each needs comprehensive prompts with:
- Language-specific frameworks (JUnit/Jest/pytest/XCTest/PHPUnit/flutter_test)
- Assertion libraries
- Mocking frameworks
- Spring/Django/Express specific patterns
- Complete test examples
- Infrastructure setup

Files:
- [ ] java/test-implementation.md
- [ ] kotlin/test-implementation.md
- [ ] python/test-implementation.md
- [ ] typescript/test-implementation.md
- [ ] swift/test-implementation.md
- [ ] php/test-implementation.md
- [ ] dart/test-implementation.md

### Category 2: ci-cd-github-actions (8 files)

Each needs:
- Version detection for language
- Cache strategies
- Build/test/deploy pipeline examples
- Matrix testing
- Deployment strategies
- Platform alternatives (GitLab CI, Azure DevOps)

Files:
- [ ] dart/ci-cd-github-actions.md
- [ ] dotnet/ci-cd-github-actions.md
- [ ] java/ci-cd-github-actions.md
- [ ] kotlin/ci-cd-github-actions.md
- [ ] php/ci-cd-github-actions.md
- [ ] python/ci-cd-github-actions.md
- [ ] swift/ci-cd-github-actions.md
- [ ] typescript/ci-cd-github-actions.md

### Category 3: code-coverage (8 files)

Each needs:
- Language-specific coverage tools (JaCoCo/nyc/coverage.py/Slather)
- Configuration examples
- Threshold setup
- CI integration
- Report formats

Files:
- [ ] dart/code-coverage.md
- [ ] dotnet/code-coverage.md
- [ ] java/code-coverage.md
- [ ] kotlin/code-coverage.md
- [ ] php/code-coverage.md
- [ ] python/code-coverage.md
- [ ] swift/code-coverage.md
- [ ] typescript/code-coverage.md

### Category 4: docker-containerization (8 files)

Each needs:
- Multi-stage Dockerfile examples
- Language-specific base images
- docker-compose examples
- Production optimizations
- Security best practices

Files:
- [ ] dart/docker-containerization.md
- [ ] dotnet/docker-containerization.md
- [ ] java/docker-containerization.md
- [ ] kotlin/docker-containerization.md
- [ ] php/docker-containerization.md
- [ ] python/docker-containerization.md
- [ ] swift/docker-containerization.md
- [ ] typescript/docker-containerization.md

### Category 5: logging-observability (8 files)

Each needs:
- Structured logging patterns
- Logger library recommendations
- Correlation IDs
- Log levels
- ELK/Loki integration
- Alert configuration

Files:
- [ ] dart/logging-observability.md
- [ ] dotnet/logging-observability.md
- [ ] java/logging-observability.md
- [ ] kotlin/logging-observability.md
- [ ] php/logging-observability.md
- [ ] python/logging-observability.md
- [ ] swift/logging-observability.md
- [ ] typescript/logging-observability.md

### Category 6: linting-formatting (8 files)

Each needs:
- Linter recommendations (ESLint/Ruff/SwiftLint/etc.)
- Formatter recommendations
- Configuration examples
- Pre-commit hooks
- CI integration

Files:
- [ ] dart/linting-formatting.md
- [ ] dotnet/linting-formatting.md
- [ ] java/linting-formatting.md
- [ ] kotlin/linting-formatting.md
- [ ] php/linting-formatting.md
- [ ] python/linting-formatting.md
- [ ] swift/linting-formatting.md
- [ ] typescript/linting-formatting.md

### Category 7: security-scanning (8 files)

Each needs:
- Dependency scanning tools
- SAST tools
- Container scanning
- Configuration examples
- CI integration

Files:
- [ ] dart/security-scanning.md
- [ ] dotnet/security-scanning.md
- [ ] java/security-scanning.md
- [ ] kotlin/security-scanning.md
- [ ] php/security-scanning.md
- [ ] python/security-scanning.md
- [ ] swift/security-scanning.md
- [ ] typescript/security-scanning.md

### Category 8: api-documentation-openapi (7 files)

Each needs:
- OpenAPI/Swagger library recommendations
- Configuration examples
- Annotation/decorator patterns
- UI setup
- Auto-generation setup

Files:
- [ ] dotnet/api-documentation-openapi.md
- [ ] java/api-documentation-openapi.md
- [ ] kotlin/api-documentation-openapi.md
- [ ] php/api-documentation-openapi.md
- [ ] python/api-documentation-openapi.md
- [ ] swift/api-documentation-openapi.md
- [ ] typescript/api-documentation-openapi.md

### Category 9: authentication-jwt-oauth (7 files)

Each needs:
- JWT library recommendations
- OAuth setup
- Password hashing
- Token generation/validation
- RBAC patterns
- Security best practices

Files:
- [ ] dotnet/authentication-jwt-oauth.md
- [ ] java/authentication-jwt-oauth.md
- [ ] kotlin/authentication-jwt-oauth.md
- [ ] php/authentication-jwt-oauth.md
- [ ] python/authentication-jwt-oauth.md
- [ ] swift/authentication-jwt-oauth.md
- [ ] typescript/authentication-jwt-oauth.md

## Estimated Effort

- **Per file**: 15-20 minutes (creating comprehensive prompt)
- **68 files**: ~18-23 hours of work
- **Token cost**: 80-120k tokens (10-12% of budget)

## Recommended Approach

### Option A: Continue Now (Full Implementation)
- Systematically update all 68 files
- Complete in current session
- Requires 80-120k tokens
- 18-23 hours of systematic work

### Option B: Batch by Priority
- Update test-implementation first (7 files)
- Commit and review
- Continue with next batch

### Option C: Create Automation Scripts
- Create PowerShell/bash scripts to generate comprehensive prompts
- Use templates with language-specific substitutions
- Faster but less tailored

### Option D: Document Pattern for Later
- Commit current progress
- Create comprehensive guide
- User or next AI session completes remaining files

## Recommendation

**Option B - Batch by Priority** is recommended because:
- Provides immediate value (test-implementation files most used)
- Allows review and feedback
- Can adjust pattern based on user preference
- Manageable scope per batch
- Clear progress tracking

Next step: Complete test-implementation files (7 files), commit, and review.
