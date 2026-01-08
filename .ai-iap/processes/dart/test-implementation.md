# Dart/Flutter Testing Implementation Process

> **Purpose**: Establish comprehensive testing infrastructure for Dart/Flutter projects

## Critical Requirements

> **ALWAYS**: Detect Dart/Flutter version from `pubspec.yaml`
> **ALWAYS**: Match detected version in Docker images, pipelines, and test configuration
> **ALWAYS**: Use your team's workflow for branching and commits (adapt as needed)
> **NEVER**: Fix production code bugs found during testing (log only)

## Workflow Adaptation

> **IMPORTANT**: This guide focuses on OBJECTIVES, not specific workflows.  
> **Your team's conventions take precedence** for Git, commits, Docker, CI/CD.  
> See [Git Workflow Adaptation Guide](../_templates/git-workflow-adaptation.md)

## Tech Stack

**Required**:
- **Test Framework**: Built-in `test` package
- **Widget Testing**: Flutter `flutter_test`
- **Integration Testing**: `integration_test` package
- **Mocking**: `mockito` or `mocktail`
- **BLoC Testing**: `bloc_test` (if using BLoC)
- **Runtime**: Match detected Dart/Flutter version

**Test Types**:
1. **Unit Tests**: Pure Dart logic
2. **Widget Tests**: UI component testing
3. **Integration Tests**: Full app testing

## Infrastructure Templates

> **ALWAYS**: Replace `{FLUTTER_VERSION}` with detected version before creating files

**File**: `docker/Dockerfile.tests`
```dockerfile
FROM cirrusci/flutter:{FLUTTER_VERSION}
WORKDIR /app

# Copy pubspec files
COPY pubspec.yaml pubspec.lock ./
RUN flutter pub get

# Copy application
COPY . .

# Create test directories
RUN mkdir -p /test-results /coverage
```

**File**: `docker/docker-compose.tests.yml`
```yaml
services:
  tests:
    build:
      context: ..
      dockerfile: docker/Dockerfile.tests
    command: flutter test --coverage --machine > /test-results/test-results.json
    volumes:
      - ../test-results:/test-results
      - ../coverage:/app/coverage
```

**CI/CD Integration**:

> **NEVER**: Overwrite existing pipeline. Merge this step only.

**GitHub Actions**:
```yaml
- name: Run Tests
  run: |
    flutter test --coverage
    flutter test --machine > test-results.json
  env:
    FLUTTER_VERSION: {FLUTTER_VERSION}
```

**GitLab CI**:
```yaml
test:
  image: cirrusci/flutter:{FLUTTER_VERSION}
  script:
    - flutter pub get
    - flutter test --coverage --machine > test-results.json
  artifacts:
    reports:
      junit: test-results.xml
    paths:
      - coverage/lcov.info
```

## Implementation Phases

> **For each phase**: Use your team's workflow ([see adaptation guide](../_templates/git-workflow-adaptation.md))

### Phase 1: Analysis

**Objective**: Understand project structure and choose test approach

1. Detect Dart/Flutter version from `pubspec.yaml`
2. Identify state management (BLoC/Riverpod/GetX/Provider)
3. Analyze existing test setup

**Deliverable**: Testing strategy documented

### Phase 2: Infrastructure (Optional)

**Objective**: Set up test infrastructure (skip if using cloud CI/CD)

1. Create Docker test files (if using Docker)
2. Add/update CI/CD pipeline test step
3. Configure test reporting

**Deliverable**: Tests can run in CI/CD

### Phase 3: Framework Setup

**Objective**: Install and configure test dependencies

1. Add to `pubspec.yaml`: `flutter_test`, `test`, `mockito`/`mocktail`, `bloc_test` (if using BLoC)
2. Run `flutter pub get`
3. Create test configuration

**Deliverable**: Test framework ready

### Phase 4: Test Structure

**Objective**: Establish test directory organization

1. Create test directories: `test/unit/`, `test/widget/`, `test/helpers/`, `integration_test/`
2. Create shared test utilities and mock data helpers

**Deliverable**: Test structure in place

### Phase 5: Test Implementation (Iterative)

**Objective**: Write tests for all components

**For each component**:
1. Understand component behavior
2. Write tests (unit/widget/integration)
3. Ensure tests pass
4. Log bugs found (don't fix production code)

**Continue until**: All critical components tested

## Test Patterns

### Unit Test Pattern
```dart
import 'package:test/test.dart';
import 'package:your_app/services/user_service.dart';

void main() {
  group('UserService', () {
    late UserService service;

    setUp(() {
      service = UserService();
    });

    test('createUser returns user with valid data', () {
      // Given
      const email = 'john@example.com';
      const name = 'John Doe';

      // When
      final user = service.createUser(email: email, name: name);

      // Then
      expect(user.email, equals(email));
      expect(user.name, equals(name));
    });

    test('createUser throws exception for invalid email', () {
      // Given
      const invalidEmail = 'invalid';

      // When/Then
      expect(
        () => service.createUser(email: invalidEmail, name: 'John'),
        throwsA(isA<FormatException>()),
      );
    });
  });
}
```

### Mockito Pattern
```dart
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:test/test.dart';
import 'package:your_app/repositories/user_repository.dart';
import 'package:your_app/services/user_service.dart';

// Generate mocks: flutter pub run build_runner build
@GenerateMocks([UserRepository])
import 'user_service_test.mocks.dart';

void main() {
  group('UserService with mocks', () {
    late MockUserRepository mockRepository;
    late UserService service;

    setUp(() {
      mockRepository = MockUserRepository();
      service = UserService(repository: mockRepository);
    });

    test('findById calls repository', () async {
      // Given
      final user = User(id: 1, name: 'John');
      when(mockRepository.findById(1))
          .thenAnswer((_) async => user);

      // When
      final result = await service.findById(1);

      // Then
      expect(result.name, equals('John'));
      verify(mockRepository.findById(1)).called(1);
    });
  });
}
```

### Widget Test Pattern
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/widgets/user_card.dart';
import 'package:your_app/models/user.dart';

void main() {
  group('UserCard Widget', () {
    testWidgets('displays user name and email', (tester) async {
      // Given
      const user = User(id: 1, name: 'John Doe', email: 'john@example.com');

      // When
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: UserCard(user: user),
          ),
        ),
      );

      // Then
      expect(find.text('John Doe'), findsOneWidget);
      expect(find.text('john@example.com'), findsOneWidget);
    });

    testWidgets('calls onTap when tapped', (tester) async {
      // Given
      const user = User(id: 1, name: 'John Doe', email: 'john@example.com');
      var tapped = false;

      // When
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: UserCard(
              user: user,
              onTap: () => tapped = true,
            ),
          ),
        ),
      );

      await tester.tap(find.byType(UserCard));
      await tester.pump();

      // Then
      expect(tapped, isTrue);
    });
  });
}
```

### BLoC Test Pattern
```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:mockito/mockito.dart';
import 'package:test/test.dart';
import 'package:your_app/blocs/user/user_bloc.dart';

void main() {
  group('UserBloc', () {
    late UserRepository mockRepository;

    setUp(() {
      mockRepository = MockUserRepository();
    });

    blocTest<UserBloc, UserState>(
      'emits [UserLoading, UserLoaded] when LoadUser is added',
      build: () {
        when(mockRepository.getUsers())
            .thenAnswer((_) async => [User(id: 1, name: 'John')]);
        return UserBloc(repository: mockRepository);
      },
      act: (bloc) => bloc.add(const LoadUsers()),
      expect: () => [
        const UserLoading(),
        UserLoaded(users: [User(id: 1, name: 'John')]),
      ],
      verify: (_) {
        verify(mockRepository.getUsers()).called(1);
      },
    );

    blocTest<UserBloc, UserState>(
      'emits [UserLoading, UserError] when repository throws',
      build: () {
        when(mockRepository.getUsers())
            .thenThrow(Exception('Failed to load'));
        return UserBloc(repository: mockRepository);
      },
      act: (bloc) => bloc.add(const LoadUsers()),
      expect: () => [
        const UserLoading(),
        const UserError(message: 'Failed to load users'),
      ],
    );
  });
}
```

### Riverpod Test Pattern
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/providers/user_provider.dart';

void main() {
  group('UserProvider', () {
    test('userProvider returns user list', () async {
      // Given
      final container = ProviderContainer();
      addTearDown(container.dispose);

      // When
      final users = await container.read(userProvider.future);

      // Then
      expect(users, isNotEmpty);
      expect(users.first.name, isNotNull);
    });

    test('userProvider with mock repository', () async {
      // Given
      final mockRepository = MockUserRepository();
      when(mockRepository.getUsers())
          .thenAnswer((_) async => [User(id: 1, name: 'John')]);

      final container = ProviderContainer(
        overrides: [
          userRepositoryProvider.overrideWithValue(mockRepository),
        ],
      );
      addTearDown(container.dispose);

      // When
      final users = await container.read(userProvider.future);

      // Then
      expect(users.length, equals(1));
      expect(users.first.name, equals('John'));
    });
  });
}
```

### Integration Test Pattern
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:your_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('App Integration Tests', () {
    testWidgets('complete user flow', (tester) async {
      // Given
      app.main();
      await tester.pumpAndSettle();

      // When - Navigate to users screen
      await tester.tap(find.text('Users'));
      await tester.pumpAndSettle();

      // Then - Verify users are loaded
      expect(find.byType(UserCard), findsWidgets);

      // When - Tap first user
      await tester.tap(find.byType(UserCard).first);
      await tester.pumpAndSettle();

      // Then - Verify user details screen
      expect(find.text('User Details'), findsOneWidget);
    });
  });
}
```

### Golden Test Pattern (Screenshot Testing)
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/widgets/user_card.dart';

void main() {
  testWidgets('UserCard matches golden file', (tester) async {
    // Given
    const user = User(id: 1, name: 'John Doe', email: 'john@example.com');

    // When
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: UserCard(user: user),
        ),
      ),
    );

    // Then
    await expectLater(
      find.byType(UserCard),
      matchesGoldenFile('goldens/user_card.png'),
    );
  });
}
```

## Documentation (`process-docs/`)

- **STATUS-DETAILS.md**: Component test checklist
- **PROJECT_MEMORY.md**: Detected Dart/Flutter version + state management + lessons learned
- **LOGIC_ANOMALIES.md**: Found bugs (audit only, don't fix)

## Usage

**Initial**:
```
Act as Senior SDET. Start Flutter testing implementation.
Phase 1: Create branch `poc/test-establishing/init-analysis`, detect Flutter version, analyze state management, initialize docs.
```

**Continue**:
```
Act as Senior SDET. Check STATUS-DETAILS.md for next phase/component. Execute and propose commit.
```

