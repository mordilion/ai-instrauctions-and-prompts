# Dart/Flutter Testing Implementation - Copy This Prompt

> **Type**: One-time setup process  
> **When to use**: Setting up testing infrastructure in a Dart/Flutter project  
> **Instructions**: Copy the complete prompt below and paste into your AI tool

---

## 📋 Complete Self-Contained Prompt

```
========================================
DART/FLUTTER TESTING IMPLEMENTATION
========================================

CONTEXT:
You are implementing comprehensive testing infrastructure for a Dart/Flutter project.

CRITICAL REQUIREMENTS:
- ALWAYS detect Dart/Flutter version from pubspec.yaml
- NEVER fix production code bugs (log in LOGIC_ANOMALIES.md only)
- Use team's Git workflow

========================================
TECH STACK
========================================

Test Framework:
- flutter_test ⭐ (Flutter projects)
- test package (Dart packages)

Mocking: mockito ⭐ (recommended) / mocktail

========================================
PHASE 1 - ANALYSIS
========================================

1. Detect Dart/Flutter version from pubspec.yaml
2. Identify project type (Flutter app or Dart package)
3. Document in process-docs/PROJECT_MEMORY.md
4. Report findings

Deliverable: Testing strategy documented

========================================
PHASE 2 - INFRASTRUCTURE (Optional)
========================================

Create Dockerfile.tests:
```dockerfile
FROM cirrusci/flutter:{VERSION}
WORKDIR /app
COPY pubspec.yaml pubspec.lock ./
RUN flutter pub get
COPY . .
RUN flutter test
```

Add to CI/CD:
```yaml
- name: Test
  run: flutter test
```

Deliverable: Tests run in CI/CD

========================================
PHASE 3 - TEST PROJECT SETUP
========================================

1. Add dependencies to pubspec.yaml:
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.0
  build_runner: ^2.4.0
```

2. Create test/ directory

3. Set up mockito code generation (if using)

4. Create test fixtures

Deliverable: Test infrastructure ready

========================================
PHASE 4 - WRITE TESTS (Iterative)
========================================

For each component:

1. Write unit tests:
```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('should handle success case', () {
    // Given
    final service = MyService();
    
    // When
    final result = service.process('input');
    
    // Then
    expect(result, 'expected');
  });
}
```

2. Write widget tests (Flutter):
```dart
testWidgets('button displays correct text', (tester) async {
  await tester.pumpWidget(MyApp());
  
  expect(find.text('Submit'), findsOneWidget);
  
  await tester.tap(find.text('Submit'));
  await tester.pump();
  
  expect(find.text('Success'), findsOneWidget);
});
```

3. Mock dependencies:
```dart
@GenerateMocks([Repository])
void main() {
  test('should call repository', () {
    final repository = MockRepository();
    when(repository.find(1)).thenReturn(data);
    
    final service = Service(repository);
    service.process(1);
    
    verify(repository.find(1)).called(1);
  });
}
```

4. Run tests: flutter test or dart test (must pass)
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
PROJECT_MEMORY.md - Dart/Flutter version, lessons
LOGIC_ANOMALIES.md - Bugs found (audit only)

========================================
EXECUTION
========================================

START: Execute Phase 1 - detect version and project type
CONTINUE: Execute phases 2-4 iteratively
REMEMBER: Use flutter_test, don't fix bugs, iterate
```

---

## Quick Reference

**What you get**: Complete test infrastructure with flutter_test/test package  
**Time**: 4-8 hours depending on project size  
**Output**: Comprehensive test coverage for Dart/Flutter project
