# Kotlin Testing Implementation - Copy This Prompt

> **Type**: One-time setup process  
> **When to use**: Setting up testing infrastructure in a Kotlin project  
> **Instructions**: Copy the complete prompt below and paste into your AI tool

---

## 📋 Complete Self-Contained Prompt

```
========================================
KOTLIN TESTING IMPLEMENTATION
========================================

CONTEXT:
You are implementing comprehensive testing infrastructure for a Kotlin project.

CRITICAL REQUIREMENTS:
- ALWAYS detect Kotlin and Java versions from build.gradle.kts
- ALWAYS match detected versions in Docker/CI/CD
- NEVER fix production code bugs (log in LOGIC_ANOMALIES.md only)
- Use team's Git workflow

========================================
TECH STACK
========================================

Test Framework: JUnit 5 ⭐ (recommended) / Kotest / Spek
Assertions: Kotest matchers ⭐ (recommended) / AssertJ / Strikt
Mocking: MockK ⭐ (recommended) / Mockito-Kotlin / Mockito

========================================
PHASE 1 - ANALYSIS
========================================

1. Detect Kotlin and Java versions from build.gradle.kts
2. Document in process-docs/PROJECT_MEMORY.md
3. Choose test frameworks
4. Report findings

Deliverable: Testing strategy documented

========================================
PHASE 2 - INFRASTRUCTURE (Optional)
========================================

Create Dockerfile.tests:
```dockerfile
FROM gradle:jdk{VERSION}-jammy
WORKDIR /app
COPY build.gradle.kts settings.gradle.kts ./
RUN gradle dependencies
COPY src ./src
RUN gradle test
```

Add to CI/CD:
```yaml
- name: Test
  run: ./gradlew test
```

Deliverable: Tests run in CI/CD

========================================
PHASE 3 - TEST PROJECT SETUP
========================================

1. Add dependencies (build.gradle.kts):
```kotlin
testImplementation("org.jetbrains.kotlin:kotlin-test-junit5")
testImplementation("io.kotest:kotest-assertions-core")
testImplementation("io.mockk:mockk")
```

2. Create structure:
   src/test/kotlin/
   ├── unit/
   ├── integration/
   └── helpers/

3. Create shared utilities (Kotlin-idiomatic)

Deliverable: Test infrastructure ready

========================================
PHASE 4 - WRITE TESTS (Iterative)
========================================

For each component:

1. Write unit tests (Kotlin style):
```kotlin
@Test
fun `should handle success case`() {
    // Given
    val service = MyService()
    
    // When
    val result = service.process("input")
    
    // Then
    result shouldBe "expected"
}
```

2. Mock dependencies (MockK):
```kotlin
@Test
fun `should call repository`() {
    val repository = mockk<Repository>()
    val service = Service(repository)
    
    every { repository.find(1L) } returns data
    
    service.process(1L)
    
    verify { repository.find(1L) }
}
```

3. Use Kotlin features:
   - Data classes for test data
   - Extension functions
   - Scope functions

4. Run tests: ./gradlew test (must pass)
5. If bugs found: Log to LOGIC_ANOMALIES.md
6. Update STATUS-DETAILS.md
7. Propose commit
8. Repeat

Deliverable: All components tested

========================================
DOCUMENTATION
========================================

Create in process-docs/:

STATUS-DETAILS.md - Component checklist
PROJECT_MEMORY.md - Versions, frameworks, lessons
LOGIC_ANOMALIES.md - Bugs found (audit only)

========================================
EXECUTION
========================================

START: Execute Phase 1 - detect versions, choose frameworks
CONTINUE: Execute phases 2-4 iteratively
REMEMBER: Use Kotlin idioms, don't fix bugs, iterate
```

---

## Quick Reference

**What you get**: Complete test infrastructure with JUnit 5, Kotest, MockK  
**Time**: 4-8 hours depending on project size  
**Output**: Kotlin-idiomatic tests with comprehensive coverage
