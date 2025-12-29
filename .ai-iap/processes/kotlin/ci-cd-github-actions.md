# CI/CD Implementation Process - Kotlin (GitHub Actions)

> **Purpose**: Establish comprehensive CI/CD pipeline with GitHub Actions for Kotlin applications

> **Platform**: This guide is for **GitHub Actions**. For GitLab CI, Azure DevOps, CircleCI, or Jenkins, adapt the workflow syntax accordingly.

---

## Prerequisites

> **BEFORE starting**:
> - Working Kotlin application
> - Git repository with remote (GitHub)
> - Gradle or Maven configured
> - Tests exist (JUnit 5, Kotest, MockK)
> - Kotlin & JVM version defined in build.gradle.kts

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
> - JVM version from project (read from build.gradle.kts or toolchain config)
> - Kotlin version from build.gradle.kts
> - Setup with actions/setup-java@v3
> - Gradle caching (~/.gradle, ~/.konan for Kotlin/Native)
> - Build: `gradle build` or `mvn clean install`
> - Run tests with JUnit 5 or Kotest
> - Collect coverage with JaCoCo or Kover

> **Version Strategy**:
> - **Best**: Use Gradle toolchain to specify JDK version
> - **Good**: Read from build.gradle.kts `jvmToolchain(17)`
> - **Matrix**: Test against multiple JVM versions if library

> **NEVER**:
> - Skip tests with `-x test`
> - Use outdated Kotlin/JVM versions
> - Ignore compiler warnings
> - Run with Gradle daemon in CI (use `--no-daemon`)

**Key Workflow Structure**:
- Trigger: push (main/develop), pull_request
- Jobs: lint → test → build
- Cache: Gradle/Maven dependencies
- Kotlin compilation cache

### 1.2 Coverage Reporting

> **ALWAYS**:
> - Use JaCoCo or Kover (Kotlin-specific coverage)
> - Generate XML/HTML reports
> - Upload to Codecov/Coveralls
> - Set minimum coverage threshold (80%+)

**Coverage Commands**:
```bash
gradle test jacocoTestReport
# Or with Kover: gradle koverXmlReport
```

**Verify**: Pipeline runs, builds succeed, tests execute, coverage report generated, Gradle cache working

---

## Phase 2: Code Quality & Security

**Branch**: `ci/quality-security`

### 2.1 Code Quality Analysis

> **ALWAYS include**:
> - ktlint for Kotlin code style
> - detekt for static analysis
> - Fail build on violations

> **NEVER**: Suppress warnings globally, skip linter configuration, allow code smells in new code

**Gradle Plugins**:
```kotlin
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "11.6.1"
    id("io.gitlab.arturbosch.detekt") version "1.23.4"
}
```

### 2.2 Dependency Security Scanning

> **ALWAYS include**:
> - Dependabot configuration (`.github/dependabot.yml`)
> - OWASP Dependency-Check or Snyk
> - Fail on known vulnerabilities

**Dependabot Config**:
```yaml
version: 2
updates:
  - package-ecosystem: "gradle"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 2.3 Static Analysis (SAST)

> **ALWAYS**:
> - Add CodeQL analysis (`.github/workflows/codeql.yml`)
> - Configure languages: java (covers Kotlin)
> - Run on schedule (weekly) + push to main

> **Optional but recommended**: SonarCloud

**Verify**: ktlint/detekt pass, Dependabot creates PRs, CodeQL scan completes, security issues reported

---

## Phase 3: Deployment Pipeline

**Branch**: `ci/deployment`

### 3.1 Environment Configuration

> **ALWAYS**:
> - Define environments: dev, staging, production
> - Use GitHub Environments with protection rules
> - Store secrets per environment
> - Use Spring profiles or environment-specific configs

**Protection Rules**: Production (require approval, restrict to main), Staging (auto-deploy on merge to develop), Dev (auto-deploy on feature branches)

### 3.2 Build & Package Artifacts

> **ALWAYS**:
> - Build JAR: `gradle shadowJar` or `gradle bootJar` (Spring Boot)
> - Version artifacts (Gradle: version in build.gradle.kts)
> - Upload artifacts with retention policy
> - Create fat JAR or native image (Kotlin/Native, GraalVM)

> **NEVER**: Include test dependencies, package without optimization, ship debug artifacts

**Package Commands**:
```bash
gradle shadowJar --no-daemon  # Fat JAR
# Or: gradle bootJar for Spring Boot
# Or: gradle nativeCompile for GraalVM native
```

### 3.3 Deployment Jobs

> **Platform-specific** (choose one or more):

| Platform | Tool/Method | Notes |
|----------|-------------|-------|
| **AWS** | aws-actions/configure-aws-credentials | Lambda, ECS, Elastic Beanstalk |
| **Azure** | azure/webapps-deploy@v2 | App Service, Container Apps |
| **Google Cloud** | google-github-actions/setup-gcloud | Cloud Run, App Engine |
| **Docker Registry** | docker/build-push-action | Multi-stage Dockerfile |
| **Kubernetes** | kubectl / Helm | Deploy to GKE, EKS, AKS |

### 3.4 Database Migrations

> **ALWAYS**:
> - Use Flyway or Liquibase (same as Java)
> - Run migrations before app deployment
> - Test migrations in staging first
> - Version control migration scripts

**Migration Commands**:
```bash
gradle flywayMigrate -Dflyway.url=$DB_URL
```

### 3.5 Smoke Tests Post-Deploy

> **ALWAYS include**:
> - Health check endpoint test
> - Database connectivity
> - Cache/Redis connectivity
> - External service integration checks

**Verify**: Manual trigger works, secrets accessible, JAR built correctly, deployment succeeds, migrations applied, smoke tests pass, rollback tested

---

## Phase 4: Advanced Features

**Branch**: `ci/advanced`

### 4.1 Performance Testing

> **ALWAYS**:
> - Kotlin microbenchmarks (kotlinx-benchmark)
> - Load testing with Gatling or k6
> - Track response times and memory
> - Fail if performance degrades >10%

### 4.2 Integration Testing

> **ALWAYS**:
> - Separate workflow (`integration-tests.yml`)
> - Use Testcontainers for database/Redis
> - Spring Boot: @SpringBootTest with test profiles
> - Run on schedule (nightly) + release tags

> **NEVER**: Use real production databases, skip cleanup, run on every PR (too slow)

### 4.3 Release Automation

> **Semantic Versioning**:
> - Use gradle-git-versioning or nebula-release
> - Generate CHANGELOG from conventional commits
> - Create GitHub Releases
> - Publish to Maven Central (if library)

### 4.4 Maven Central Publishing

> **If creating libraries**:
> - Configure GPG signing
> - Publish to Maven Central via OSSRH
> - Include sources and javadoc JARs
> - Use maven-publish plugin

> **ALWAYS**: Set group, artifact, version; Include license, developers, SCM; Sign with GPG

### 4.5 Notifications

> **ALWAYS**: Slack/Teams webhook, GitHub Status Checks, Email for security alerts

**Verify**: Benchmarks run, integration tests pass, releases created automatically, Maven Central publish works (if applicable), notifications received

---

## Framework-Specific Notes

| Framework | Notes |
|-----------|-------|
| **Spring Boot (Kotlin)** | Build: `gradle bootJar`; Health: `/actuator/health`; Use Kotlin DSL for configs |
| **Ktor** | Build: `gradle shadowJar`; Health: custom route; Lightweight, async-first |
| **Android** | Build APK: `gradle assembleRelease`; Sign with keystore; Upload to Play Store |
| **Kotlin Multiplatform** | Build for JVM, JS, Native targets; Test on all platforms; Use expect/actual |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Gradle build fails with "daemon not found"** | Always use `--no-daemon` in CI |
| **Kotlin compilation slow in CI** | Enable Gradle build cache, use parallel execution |
| **Tests pass locally but fail in CI** | Check JVM version match, timezone, locale |
| **Coverage not collected** | Ensure JaCoCo/Kover plugin configured, run test + report tasks |
| **Want to use GitLab CI / Azure DevOps** | GitLab: Use `gradle:jdk17` image; Azure: Use `Gradle@2` task - core concepts remain same |

---

## AI Self-Check

- [ ] CI pipeline runs on push and PR
- [ ] JVM & Kotlin versions pinned
- [ ] Builds succeed without warnings
- [ ] All tests pass with coverage ≥80%
- [ ] Code quality enforced (ktlint, detekt)
- [ ] Security scanning enabled (Dependabot, CodeQL)
- [ ] JAR packaged with correct versioning
- [ ] Deployment to at least one environment works
- [ ] Database migrations tested and automated
- [ ] Smoke tests validate deployment health

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
