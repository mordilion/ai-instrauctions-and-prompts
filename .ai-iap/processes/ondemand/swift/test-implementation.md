# Swift Testing Implementation - Copy This Prompt

> **Type**: One-time setup process  
> **When to use**: Setting up testing infrastructure in a Swift project  
> **Instructions**: Copy the complete prompt below and paste into your AI tool

---

## 📋 Complete Self-Contained Prompt

```
========================================
SWIFT TESTING IMPLEMENTATION
========================================

CONTEXT:
You are implementing comprehensive testing infrastructure for a Swift project.

CRITICAL REQUIREMENTS:
- ALWAYS detect Swift version from Package.swift or Xcode project
- ALWAYS identify project type (iOS app, SPM library, UIKit, SwiftUI)
- NEVER fix production code bugs (log in LOGIC_ANOMALIES.md only)
- Use team's Git workflow

========================================
TECH STACK
========================================

Test Framework: XCTest ⭐ (Apple's official) / Quick + Nimble

Project Types:
- iOS App (UIKit) - XCUITest for UI testing
- iOS App (SwiftUI) - SwiftUI testing APIs
- SPM Library - swift test command

========================================
PHASE 1 - ANALYSIS
========================================

1. Detect Swift version from Package.swift or Xcode
2. Identify project type (iOS app, library, platform)
3. Detect UI framework (UIKit, SwiftUI, or both)
4. Document in process-docs/PROJECT_MEMORY.md
5. Report findings

Deliverable: Testing strategy documented

========================================
PHASE 2 - INFRASTRUCTURE (Optional)
========================================

For iOS apps:
- Create UI test target in Xcode (XCUITest)
- Configure test schemes

For SPM libraries:
- Configure Package.swift with test targets
- Use swift test

Add to CI/CD:
```yaml
- name: Test
  run: xcodebuild test -scheme MyApp
```

Or for SPM:
```yaml
- name: Test
  run: swift test
```

Deliverable: Tests run in CI/CD

========================================
PHASE 3 - TEST PROJECT SETUP
========================================

1. Create test targets/directories

2. Implement shared test utilities:
   - Mock objects
   - Test doubles
   - Fixtures

3. Set up test schemes

Deliverable: Test infrastructure ready

========================================
PHASE 4 - WRITE TESTS (Iterative)
========================================

For each component:

1. Write unit tests (XCTest):
```swift
import XCTest
@testable import MyApp

class MyServiceTests: XCTestCase {
    func testShouldHandleSuccessCase() {
        // Given
        let service = MyService()
        
        // When
        let result = service.process("input")
        
        // Then
        XCTAssertEqual(result, "expected")
    }
}
```

2. Write UI tests (if applicable):
```swift
class UITests: XCTestCase {
    func testButtonTap() {
        let app = XCUIApplication()
        app.launch()
        
        app.buttons["Submit"].tap()
        XCTAssertTrue(app.staticTexts["Success"].exists)
    }
}
```

3. Run tests - must pass
4. If bugs found: Log to LOGIC_ANOMALIES.md
5. Update STATUS-DETAILS.md
6. Propose commit
7. Repeat

Deliverable: All components tested

========================================
DOCUMENTATION
========================================

Create in process-docs/:

STATUS-DETAILS.md - Component checklist
PROJECT_MEMORY.md - Swift version, project type, UI framework, lessons
LOGIC_ANOMALIES.md - Bugs found (audit only)

========================================
EXECUTION
========================================

START: Execute Phase 1 - detect Swift version and project type
CONTINUE: Execute phases 2-4 iteratively
REMEMBER: Use XCTest, don't fix bugs, iterate
```

---

## Quick Reference

**What you get**: Complete test infrastructure with XCTest  
**Time**: 4-8 hours depending on project size  
**Output**: Comprehensive test coverage for Swift project
