# CI/CD Implementation Process - Swift (GitHub Actions)

> **Purpose**: Establish comprehensive CI/CD pipeline with GitHub Actions for Swift applications (iOS, macOS, Server/Vapor)

> **Platform**: This guide is for **GitHub Actions**. For GitLab CI, Azure DevOps, CircleCI, Bitrise, or Codemagic, adapt the workflow syntax accordingly.

---

## Prerequisites

> **BEFORE starting**:
> - Working Swift application (iOS, macOS, or Vapor)
> - Git repository with remote (GitHub)
> - Xcode project or Swift Package configured
> - Tests exist (XCTest)
> - Swift/Xcode version defined in .xcode-version or .swift-version

---

## Git Workflow Pattern (All Phases)

> **Standard workflow for each phase**:
> 1. Create branch: `git checkout -b ci/<phase-name>`
> 2. Make changes according to phase requirements
> 3. Commit: `git commit -m "ci: <description>"`
> 4. Push: `git push origin ci/<phase-name>`
> 5. Verify: Check CI/CD pipeline runs successfully

Phases below reference this pattern instead of repeating it.

---

## Phase 1: Basic CI Pipeline

**Branch**: `ci/basic-pipeline`

### 1.1 Basic Build & Test Workflow

> **ALWAYS include**:
> - Swift/Xcode version (read from `.xcode-version` or `.swift-version`)
> - iOS: Use macos-latest runner
> - Server (Vapor): Can use ubuntu-latest with Swift Docker
> - Build: `xcodebuild` or `swift build`
> - Run tests: `xcodebuild test` or `swift test`
> - Collect coverage: Xcode Code Coverage or llvm-cov

> **Version Strategy**:
> - **Best**: Use `.xcode-version` file for Xcode version
> - **Good**: Specify in workflow with `xcode-version` parameter
> - **iOS**: Match minimum deployment target in project

> **NEVER**:
> - Hardcode Xcode version without project config
> - Skip code signing setup for iOS builds
> - Run tests without simulator selection (iOS)
> - Ignore compiler warnings

**Key Workflow Structure**:
- Trigger: push (main/develop), pull_request
- Runner: macos-latest (iOS/macOS) or ubuntu-latest (Vapor)
- Jobs: lint → test → build
- Cache: Swift Package Manager dependencies

### 1.2 Coverage Reporting

> **ALWAYS**:
> - Enable code coverage in scheme (Xcode)
> - Generate coverage reports
> - Upload to Codecov/Coveralls
> - Set minimum coverage threshold (80%+)

**Coverage Commands**:
```bash
# Xcode
xcodebuild test -scheme MyApp -enableCodeCoverage YES -resultBundlePath ./TestResults

# Vapor/SPM
swift test --enable-code-coverage
```

**Verify**: Pipeline runs, builds succeed, tests pass, coverage generated, dependencies cached

---

## Phase 2: Code Quality & Security

**Branch**: `ci/quality-security`

### 2.1 Code Quality Analysis

> **ALWAYS include**:
> - SwiftLint for code style
> - SwiftFormat for consistent formatting
> - Fail build on linting errors

> **NEVER**: Suppress warnings globally, skip linter configuration, allow force unwrapping without justification

**SwiftLint Configuration** (`.swiftlint.yml`):
```yaml
disabled_rules:
  - line_length
opt_in_rules:
  - force_unwrapping
excluded:
  - Pods
  - .build
```

### 2.2 Dependency Security Scanning

> **ALWAYS include**:
> - Dependabot configuration (`.github/dependabot.yml`) for Swift Package Manager
> - Manual security audits for CocoaPods/Carthage
> - Check for outdated dependencies

**Dependabot Config**:
```yaml
version: 2
updates:
  - package-ecosystem: "swift"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 2.3 Static Analysis (SAST)

> **ALWAYS**:
> - Add CodeQL analysis (`.github/workflows/codeql.yml`)
> - Configure language: swift
> - Run on schedule (weekly) + push to main

> **Optional but recommended**: SonarCloud, Xcode Analyze

**Verify**: SwiftLint passes, SwiftFormat applied, Dependabot creates PRs, CodeQL scan completes

---

## Phase 3: Deployment Pipeline

**Branch**: `ci/deployment`

### 3.1 Environment Configuration

> **ALWAYS**:
> - Define environments: development, staging, production
> - Use GitHub Environments with protection rules
> - Store secrets: certificates, provisioning profiles, API keys
> - Use Xcode configurations for environment-specific builds

**Protection Rules**: Production (require approval, restrict to main), Staging (auto-deploy to TestFlight), Development (internal distribution)

### 3.2 Code Signing (iOS/macOS)

> **ALWAYS**:
> - Store certificates and provisioning profiles as secrets
> - Use match (fastlane) or manual certificate management
> - Decode base64 certificates in workflow
> - Import to keychain before build

**Code Signing Setup**:
```bash
# Decode certificate
echo "${{ secrets.BUILD_CERTIFICATE_BASE64 }}" | base64 --decode > certificate.p12

# Create keychain
security create-keychain -p "${{ secrets.KEYCHAIN_PASSWORD }}" build.keychain
security import certificate.p12 -k build.keychain -P "${{ secrets.P12_PASSWORD }}"
```

### 3.3 Build & Archive

> **ALWAYS**:
> - Build for appropriate scheme and configuration (Debug/Release)
> - Archive for distribution: `xcodebuild archive`
> - Export IPA: `xcodebuild -exportArchive`
> - Version with build number (CI_BUILD_NUMBER or git SHA)

> **NEVER**: Include development certificates in production, ship with debug symbols exposed, forget to increment build number

**Archive Commands**:
```bash
xcodebuild archive -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -archivePath ./build/MyApp.xcarchive
xcodebuild -exportArchive -archivePath ./build/MyApp.xcarchive -exportPath ./build -exportOptionsPlist ExportOptions.plist
```

### 3.4 Deployment Jobs

> **Platform-specific** (choose one or more):

| Platform | Tool/Method | Notes |
|----------|-------------|-------|
| **App Store Connect** | fastlane or altool | Upload to TestFlight or App Store |
| **TestFlight** | fastlane pilot | Beta distribution |
| **Firebase App Distribution** | firebase-tools | Internal testing |
| **Vapor (Server)** | Docker, AWS, Heroku | Deploy as Linux service |

**Fastlane Example**:
```ruby
lane :beta do
  build_app(scheme: "MyApp")
  upload_to_testflight
end
```

### 3.5 Smoke Tests Post-Deploy

> **ALWAYS include** (for Vapor/server):
> - Health check endpoint test
> - Database connectivity
> - Redis connectivity (if applicable)

> **iOS/macOS**: UI tests on TestFlight before production

**Verify**: Code signing works, archive builds successfully, upload to TestFlight succeeds, smoke tests pass (server)

---

## Phase 4: Advanced Features

**Branch**: `ci/advanced`

### 4.1 UI Testing (iOS/macOS)

> **ALWAYS**:
> - Separate workflow for UI tests (`ui-tests.yml`)
> - Run on simulators (multiple iOS versions)
> - Record test results and screenshots
> - Run on schedule (nightly) to save CI time

### 4.2 Performance Testing

> **ALWAYS**:
> - XCTest performance tests (measureBlock)
> - Track launch time, memory usage
> - Fail if performance regresses >10%

### 4.3 Release Automation

> **Semantic Versioning**:
> - Use fastlane increment_build_number
> - Generate release notes from commits
> - Create GitHub Releases
> - Submit to App Store with fastlane

### 4.4 Notifications

> **ALWAYS**: Slack webhook on build/deploy success/failure, Email for TestFlight uploads

**Verify**: UI tests run on schedule, performance tracked, releases automated, notifications received

---

## Platform-Specific Notes

| Platform | Notes |
|----------|-------|
| **iOS App** | Code sign with certificates; Upload to App Store Connect; Use fastlane for automation |
| **macOS App** | Notarize with Apple; Code sign with Developer ID; Distribute via DMG or Mac App Store |
| **Vapor (Server)** | Build on Linux (ubuntu-latest); Deploy to AWS/Heroku; Use Docker for consistency |
| **Swift Package** | Build with `swift build`; Test with `swift test`; Publish to Swift Package Index |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Code signing fails in CI** | Verify certificates base64-encoded correctly, check keychain password, ensure provisioning profile matches bundle ID |
| **Tests fail only in CI** | Check simulator availability, ensure Xcode version matches, verify test scheme settings |
| **Archive export fails** | Validate ExportOptions.plist, check entitlements, verify provisioning profile type |
| **Slow builds in CI** | Enable caching for DerivedData and SPM, use incremental builds |
| **Want to use Bitrise / Codemagic** | Bitrise/Codemagic: Specialized for mobile, better simulator support, simpler code signing - core concepts remain same |

---

## AI Self-Check

- [ ] CI pipeline runs on push and PR
- [ ] Xcode/Swift version pinned
- [ ] SwiftLint passes
- [ ] All tests pass with coverage ≥80%
- [ ] Code signing configured (iOS/macOS)
- [ ] Archive builds successfully
- [ ] Upload to TestFlight/App Store works (iOS)
- [ ] Deployment succeeds (Vapor/server)
- [ ] Performance tests tracked
- [ ] UI tests run on schedule (iOS/macOS)

---

## Documentation Updates

> **AFTER all phases complete**:
> - Update README.md with CI/CD badges
> - Document deployment process
> - Add runbook for common issues
> - Onboarding guide for new developers

---

## Final Commit

```bash
git checkout main
git merge ci/advanced
git tag -a v1.0.0-ci -m "CI/CD pipeline implemented"
git push origin main --tags
```

---

**Process Complete** ✅
