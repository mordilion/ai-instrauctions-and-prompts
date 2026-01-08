# PHP Testing Implementation - Copy This Prompt

> **Type**: One-time setup process  
> **When to use**: Setting up testing infrastructure in a PHP project  
> **Instructions**: Copy the complete prompt below and paste into your AI tool

---

## 📋 Complete Self-Contained Prompt

```
========================================
PHP TESTING IMPLEMENTATION
========================================

CONTEXT:
You are implementing comprehensive testing infrastructure for a PHP project.

CRITICAL REQUIREMENTS:
- ALWAYS detect PHP version from composer.json
- ALWAYS detect framework (Laravel, Symfony, plain PHP)
- NEVER fix production code bugs (log in LOGIC_ANOMALIES.md only)
- Use team's Git workflow

========================================
TECH STACK
========================================

Test Framework: PHPUnit ⭐ (recommended) / Codeception / Pest / PHPSpec
Mocking: PHPUnit mocks ⭐ (built-in) / Mockery / Prophecy

========================================
PHASE 1 - ANALYSIS
========================================

1. Detect PHP version from composer.json
2. Detect framework (Laravel, Symfony, or none)
3. Document in process-docs/PROJECT_MEMORY.md
4. Choose PHPUnit (recommended)
5. Report findings

Deliverable: Testing strategy documented

========================================
PHASE 2 - INFRASTRUCTURE (Optional)
========================================

Create Dockerfile.tests:
```dockerfile
FROM php:{VERSION}-cli
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install
COPY . .
RUN vendor/bin/phpunit
```

Add to CI/CD:
```yaml
- name: Test
  run: |
    composer install
    vendor/bin/phpunit
```

Deliverable: Tests run in CI/CD

========================================
PHASE 3 - TEST PROJECT SETUP
========================================

1. Install PHPUnit:
```bash
composer require --dev phpunit/phpunit
```

2. Create phpunit.xml:
```xml
<phpunit bootstrap="vendor/autoload.php">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

3. Create structure:
   tests/
   ├── Unit/
   ├── Feature/
   └── TestCase.php

4. For Laravel: Use artisan make:test commands

Deliverable: Test infrastructure ready

========================================
PHASE 4 - WRITE TESTS (Iterative)
========================================

For each component:

1. Write unit tests:
```php
use PHPUnit\Framework\TestCase;

class MyServiceTest extends TestCase
{
    public function testShouldHandleSuccessCase()
    {
        // Given
        $service = new MyService();
        
        // When
        $result = $service->process('input');
        
        // Then
        $this->assertEquals('expected', $result);
    }
}
```

2. Mock dependencies:
```php
public function testShouldCallRepository()
{
    $repository = $this->createMock(Repository::class);
    $repository->expects($this->once())
        ->method('find')
        ->with(1)
        ->willReturn($data);
    
    $service = new Service($repository);
    $service->process(1);
}
```

3. For Laravel:
```php
public function testApiEndpoint()
{
    $response = $this->get('/api/users');
    $response->assertStatus(200);
}
```

4. Run tests: vendor/bin/phpunit (must pass)
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
PROJECT_MEMORY.md - PHP version, framework, lessons
LOGIC_ANOMALIES.md - Bugs found (audit only)

========================================
EXECUTION
========================================

START: Execute Phase 1 - detect PHP version and framework
CONTINUE: Execute phases 2-4 iteratively
REMEMBER: Use PHPUnit, don't fix bugs, iterate
```

---

## Quick Reference

**What you get**: Complete test infrastructure with PHPUnit  
**Time**: 4-8 hours depending on project size  
**Output**: Comprehensive test coverage with PHPUnit
