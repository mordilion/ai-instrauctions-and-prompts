# CI/CD Implementation Process - Dart/Flutter (GitHub Actions)

> **Purpose**: Establish comprehensive CI/CD pipeline with GitHub Actions for Dart/Flutter applications

> **Platform**: This guide is for **GitHub Actions**. For GitLab CI, Bitrise, Codemagic, or CircleCI, adapt the workflow syntax accordingly.

---

## Prerequisites

> **BEFORE starting**:
> - Working Dart/Flutter application
> - Git repository with remote (GitHub)
> - pubspec.yaml configured
> - Tests exist (flutter test, dart test)
> - Flutter/Dart version defined in pubspec.yaml or .fvmrc

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
> - Flutter/Dart version from project (read from `.fvmrc`, `pubspec.yaml`, or use stable channel)
> - Setup with subosito/flutter-action@v2
> - Dependency caching (pub cache)
> - Get dependencies: `flutter pub get`
> - Run linter/analyzer: `flutter analyze` or `dart analyze`
> - Run tests: `flutter test` or `dart test`
> - Collect coverage: `flutter test --coverage`

> **Version Strategy**:
> - **Best**: Use `.fvmrc` to pin Flutter version (if using FVM)
> - **Good**: Specify in workflow with `flutter-version` parameter
> - **Matrix**: Test against multiple versions (stable, beta) if package

> **NEVER**:
> - Skip dependency installation
> - Ignore analyzer warnings
> - Run tests without proper device/emulator setup (for integration tests)
> - Hardcode Flutter version without project config

**Key Workflow Structure**:
- Trigger: push (main/develop), pull_request
- Jobs: analyze → test → build
- Cache: Flutter SDK and pub cache
- Platform-specific jobs: Android, iOS, Web, Desktop

### 1.2 Coverage Reporting

> **ALWAYS**:
> - Generate coverage with `flutter test --coverage`
> - Upload coverage/lcov.info to Codecov/Coveralls
> - Set minimum coverage threshold (80%+)

**Coverage Commands**:
```bash
flutter test --coverage
# Generate HTML report: genhtml coverage/lcov.info -o coverage/html
```

**Verify**: Pipeline runs, analyzer passes, all tests pass, coverage report generated, dependencies cached

---

## Phase 2: Code Quality & Security

**Branch**: `ci/quality-security`

### 2.1 Code Quality Analysis

> **ALWAYS include**:
> - flutter analyze (built-in)
> - dart format --output=none --set-exit-if-changed (formatting check)
> - Custom lint rules (analysis_options.yaml)
> - Fail build on analysis issues

> **NEVER**: Suppress all lints, ignore type safety, allow unused imports

**Analysis Options** (`analysis_options.yaml`):
```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    - prefer_const_constructors
    - prefer_final_fields
    - avoid_print
    - unnecessary_null_checks
```

### 2.2 Dependency Security Scanning

> **ALWAYS include**:
> - Dependabot configuration (`.github/dependabot.yml`)
> - Check for outdated dependencies: `flutter pub outdated`
> - Audit for security issues

**Dependabot Config**:
```yaml
version: 2
updates:
  - package-ecosystem: "pub"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 2.3 Static Analysis

> **ALWAYS**:
> - Run dart analyze with strict mode
> - Check for deprecated API usage
> - Verify no TODOs/FIXMEs in main branch

**Verify**: flutter analyze passes with no issues, formatting correct, Dependabot creates PRs

---

## Phase 3: Deployment Pipeline

**Branch**: `ci/deployment`

### 3.1 Environment Configuration

> **ALWAYS**:
> - Define environments: development, staging, production
> - Use GitHub Environments with protection rules
> - Store secrets: Android keystore, iOS certificates, API keys
> - Use flavor-specific configurations (--flavor prod)

**Protection Rules**: Production (require approval, restrict to main), Staging (auto-deploy to internal testing), Development (ad-hoc builds)

### 3.2 Build Android

> **ALWAYS**:
> - Build APK or AAB: `flutter build apk --release` or `flutter build appbundle`
> - Sign with keystore (store keystore as secret)
> - Version with build number (pubspec.yaml version)
> - Upload to Google Play Console or Firebase App Distribution

> **NEVER**: Commit keystore to repository, ship debug builds to production, skip ProGuard/R8 optimization

**Android Build Commands**:
```bash
flutter build appbundle --release --flavor production
# Or APK: flutter build apk --release --split-per-abi
```

**Keystore Setup**:
```bash
# Decode keystore from secret
echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

# Configure signing in android/key.properties
```

### 3.3 Build iOS

> **ALWAYS**:
> - Build IPA: `flutter build ipa --release`
> - Sign with certificates and provisioning profiles
> - Upload to App Store Connect or TestFlight
> - Use fastlane for automation

> **NEVER**: Include development certificates in production, ship without bitcode (if required), forget to increment build number

**iOS Build Commands**:
```bash
flutter build ipa --release --flavor production --export-options-plist=ios/ExportOptions.plist
```

**Code Signing Setup** (similar to Swift CI/CD):
```bash
# Import certificates to keychain
security create-keychain -p "${{ secrets.KEYCHAIN_PASSWORD }}" build.keychain
security import certificate.p12 -k build.keychain -P "${{ secrets.P12_PASSWORD }}"
```

### 3.4 Build Web

> **ALWAYS**:
> - Build web: `flutter build web --release`
> - Deploy to Firebase Hosting, Netlify, or GitHub Pages
> - Configure base href for proper routing

**Web Build Commands**:
```bash
flutter build web --release --base-href /
```

### 3.5 Deployment Jobs

> **Platform-specific** (choose one or more):

| Platform | Tool/Method | Notes |
|----------|-------------|-------|
| **Google Play Store** | fastlane or upload via API | Upload AAB to internal/beta/production track |
| **App Store Connect** | fastlane or altool | Upload IPA to TestFlight or App Store |
| **Firebase App Distribution** | firebase-tools CLI | Internal testing for Android/iOS |
| **Web (Firebase Hosting)** | firebase-tools CLI | Deploy web build |
| **GitHub Pages** | peaceiris/actions-gh-pages | Deploy web build to gh-pages branch |

**Fastlane Example** (Android):
```ruby
lane :beta do
  gradle(task: "bundle", build_type: "Release")
  upload_to_play_store(track: "beta")
end
```

### 3.6 Smoke Tests Post-Deploy

> **ALWAYS include** (for web/backend):
> - Health check endpoint test
> - API connectivity check

> **Mobile**: Use Firebase Test Lab or App Center for automated UI tests

**Verify**: Android build succeeds, iOS build succeeds, web build succeeds, uploads to stores work, smoke tests pass

---

## Phase 4: Advanced Features

**Branch**: `ci/advanced`

### 4.1 Integration Testing

> **ALWAYS**:
> - Separate workflow for integration tests (`integration_test/`)
> - Run on emulators/simulators (Android emulator, iOS simulator)
> - Use `flutter drive` or `integration_test` package
> - Record test results and screenshots

> **NEVER**: Run integration tests on every PR (too slow), skip device-specific tests

### 4.2 Performance Testing

> **ALWAYS**:
> - Flutter driver performance tests
> - Track app startup time, frame rendering
> - Monitor memory usage
> - Fail if performance degrades >10%

### 4.3 Release Automation

> **Semantic Versioning**:
> - Update pubspec.yaml version automatically
> - Generate release notes from commits
> - Create GitHub Releases
> - Publish to pub.dev (if package)

### 4.4 pub.dev Publishing

> **If creating package**:
> - Validate with `flutter pub publish --dry-run`
> - Publish with `flutter pub publish`
> - Use pub-dev-publish GitHub Action
> - Follow pub.dev package guidelines

> **ALWAYS**: Set name, description, version in pubspec.yaml; Include README.md, CHANGELOG.md, LICENSE; Follow semantic versioning; Add example/

### 4.5 Notifications

> **ALWAYS**: Slack webhook on build success/failure, Email for store uploads

**Verify**: Integration tests run on schedule, performance tracked, releases automated, pub.dev publish works (if applicable), notifications received

---

## Platform-Specific Notes

| Platform | Notes |
|-----------|-------|
| **Android** | Sign with keystore; Upload AAB to Play Console; Use fastlane for automation; Enable R8 shrinking |
| **iOS** | Sign with certificates and provisioning profiles; Upload to TestFlight; Use fastlane; Enable bitcode if required |
| **Web** | Build with `flutter build web`; Deploy to Firebase Hosting, Netlify, GitHub Pages; Configure routing |
| **Desktop (Windows/macOS/Linux)** | Build with `flutter build windows/macos/linux`; Package as installer; Code sign for distribution |
| **Dart Package** | Build with `dart pub get`; Test with `dart test`; Publish to pub.dev |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Flutter doctor shows issues** | Ensure all required tools installed (Android SDK, Xcode), run flutter doctor --android-licenses |
| **Tests fail only in CI** | Check Flutter version match, ensure assets included, verify environment variables |
| **Android signing fails** | Verify keystore decoded correctly, check key.properties path, ensure passwords correct |
| **iOS signing fails** | Check certificate validity, verify provisioning profile matches bundle ID, ensure keychain unlocked |
| **Web build missing assets** | Verify assets declared in pubspec.yaml, check pubspec.yaml indentation |
| **Want to use Codemagic / Bitrise** | Codemagic/Bitrise: Specialized for Flutter, better device support, simpler setup - core concepts remain same |

---

## AI Self-Check

- [ ] CI pipeline runs on push and PR
- [ ] Flutter/Dart version pinned or specified
- [ ] flutter analyze passes with no warnings
- [ ] All tests pass with coverage ≥80%
- [ ] Code formatting enforced (dart format)
- [ ] Android build succeeds and signs correctly
- [ ] iOS build succeeds and signs correctly (if iOS app)
- [ ] Web build succeeds (if web app)
- [ ] Upload to stores works (Android/iOS)
- [ ] Integration tests run on schedule

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
