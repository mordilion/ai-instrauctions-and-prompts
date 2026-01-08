# CI/CD Implementation Process - Java (GitHub Actions)

> **Purpose**: Establish comprehensive CI/CD pipeline with GitHub Actions for Java applications

> **Platform**: This guide is for **GitHub Actions**. For GitLab CI, Azure DevOps, CircleCI, or Jenkins, adapt the workflow syntax accordingly.

---

## Prerequisites

> **BEFORE starting**:
> - Working Java application
> - Git repository with remote (GitHub)
> - Build tool configured (Maven or Gradle)
> - Tests exist (JUnit 5, Mockito)
> - Java version defined in pom.xml or build.gradle

---

## Workflow Adaptation

> **IMPORTANT**: Phases below focus on OBJECTIVES. Use your team's workflow.  
> See [Git Workflow Adaptation Guide](../_templates/git-workflow-adaptation.md) for examples.

---

## Phase 1: Basic CI Pipeline

**Objective**: Establish foundational CI pipeline with build, lint, and test automation

### 1.1 Basic Build & Test Workflow

> **ALWAYS include**:
> - Java version from project (read from pom.xml `<java.version>` or build.gradle `sourceCompatibility`)
> - Setup with actions/setup-java@v3
> - Dependency caching (Maven: ~/.m2, Gradle: ~/.gradle)
> - Build command (mvn clean install or gradle build)
> - Run tests with JUnit
> - Collect coverage with JaCoCo

> **Version Strategy**:
> - **Best**: Read from pom.xml `<properties><java.version>` or build.gradle `sourceCompatibility`
> - **Good**: Use toolchain configuration (Maven Toolchains, Gradle Toolchains)
> - **Matrix**: Test against multiple LTS versions (11, 17, 21) if library/framework

> **NEVER**:
> - Skip tests in CI (`mvn install -DskipTests`)
> - Use outdated Java versions in production
> - Ignore compiler warnings
> - Commit wrapper binaries (mvnw, gradlew) without verification

**Maven Workflow**: Restore (automatic with actions/setup-java cache), Build (`mvn clean install -B`), Verify (`mvn verify -B`)

**Gradle Workflow**: Restore (`gradle dependencies`), Build (`gradle build`), Test (`gradle test`), Validate (`gradle check`)

### 1.2 Coverage Reporting

> **ALWAYS**:
> - Use JaCoCo plugin
> - Generate XML/HTML reports
> - Upload to Codecov/Coveralls
> - Set minimum coverage threshold (80%+)

**JaCoCo Configuration**: Maven (jacoco-maven-plugin), Gradle (jacoco plugin), Reports (XML for CI, HTML for review)

**Verify**: Pipeline runs, builds succeed across Java versions, tests execute with results, coverage report generated, cache working

---

## Phase 2: Code Quality & Security

**Objective**: Add code quality and security scanning to CI pipeline

### 2.1 Code Quality Analysis

> **ALWAYS include**:
> - Checkstyle (google_checks.xml or sun_checks.xml)
> - PMD for static analysis
> - SpotBugs for bug detection
> - Fail build on violations

> **NEVER**: Suppress warnings globally, skip linter configuration, allow critical bugs in new code

**Maven Plugins**: maven-checkstyle-plugin, maven-pmd-plugin, spotbugs-maven-plugin

**Gradle Plugins**: checkstyle, pmd, com.github.spotbugs

### 2.2 Dependency Security Scanning

> **ALWAYS include**:
> - Dependabot configuration (`.github/dependabot.yml`)
> - OWASP Dependency-Check
> - Fail on known vulnerabilities (CVSS ≥7)

**Dependabot Config**:
```yaml
version: 2
updates:
  - package-ecosystem: "maven" # or "gradle"
    directory: "/"
    schedule:
      interval: "weekly"
```

**Dependency Check**: Maven (`dependency-check-maven`), Gradle (`org.owasp.dependencycheck`), Generate reports, fail on high severity

### 2.3 Static Analysis (SAST)

> **ALWAYS**:
> - Add CodeQL analysis (`.github/workflows/codeql.yml`)
> - Configure language: java
> - Run on schedule (weekly) + push to main

> **Optional but recommended**: SonarCloud/SonarQube, Snyk

**Verify**: Checkstyle/PMD/SpotBugs run, violations cause build failures, Dependabot creates update PRs, CodeQL scan completes, vulnerabilities reported

---

## Phase 3: Deployment Pipeline

**Objective**: Automate app deployment to relevant environments/platforms

### 3.1 Environment Configuration

> **ALWAYS**:
> - Define environments: dev, staging, production
> - Use GitHub Environments with protection rules
> - Store secrets per environment (DB credentials, API keys)
> - Use Spring Profiles or application.properties per environment

**Protection Rules**: Production (require approval, restrict to main), Staging (auto-deploy on merge to develop), Dev (auto-deploy on feature branches)

### 3.2 Build & Package Artifacts

> **ALWAYS**:
> - Package as JAR/WAR (`mvn package` or `gradle bootJar`)
> - Version artifacts (Maven: version in pom.xml, Gradle: version in build.gradle)
> - Upload artifacts with retention policy
> - Create executable JAR (Spring Boot, fat JAR)

> **NEVER**: Include application-local.properties in artifacts, package without optimization, ship test dependencies

**Maven Package**: `mvn clean package -DskipTests -B` → Output: target/*.jar or target/*.war

**Gradle Package**: `gradle bootJar --no-daemon` → Output: build/libs/*.jar

### 3.3 Deployment Jobs

> **Platform-specific** (choose one or more):

| Platform | Tool/Method | Notes |
|----------|-------------|-------|
| **AWS** | aws-actions/configure-aws-credentials | Upload JAR to S3, deploy to Elastic Beanstalk/ECS/Lambda |
| **Azure** | azure/webapps-deploy@v2 | Upload JAR via FTP or Azure CLI, Azure Spring Apps |
| **Google Cloud** | google-github-actions/setup-gcloud | Deploy to App Engine/Cloud Run |
| **Docker Registry** | docker/build-push-action | Multi-stage Dockerfile (Maven/Gradle build → JRE runtime) |
| **Heroku** | Heroku CLI/GitHub integration | Procfile: `web: java -jar target/*.jar` |

### 3.4 Database Migrations

> **ALWAYS**:
> - Use Flyway or Liquibase
> - Run migrations before app deployment
> - Test migrations in staging first
> - Version control all migration scripts

> **NEVER**: Run migrations on app start in production (security risk), skip migration testing, deploy app before migrations complete

**Flyway/Liquibase Commands**:
```bash
# Flyway
mvn flyway:migrate -Dflyway.url=$DB_URL

# Liquibase
mvn liquibase:update -Dliquibase.url=$DB_URL
```

### 3.5 Smoke Tests Post-Deploy

> **ALWAYS include**:
> - Health check endpoint (`/actuator/health` for Spring Boot)
> - Database connectivity check
> - Cache/Redis connectivity
> - External API integration check

**Verify**: Manual trigger works, environment secrets accessible, JAR packaged correctly, deployment succeeds to staging, migrations applied, smoke tests pass, rollback tested

---

## Phase 4: Advanced Features

**Objective**: Add advanced CI/CD capabilities (integration tests, release automation)

### 4.1 Performance Testing

> **ALWAYS**:
> - JMH (Java Microbenchmark Harness) for micro-benchmarks
> - Load testing with Gatling, JMeter, or k6
> - Track response times and memory usage
> - Fail if performance degrades >10%

### 4.2 Integration Testing

> **ALWAYS**:
> - Separate workflow (`integration-tests.yml`)
> - Use Testcontainers for database/Redis
> - Spring Boot: @SpringBootTest with test profiles
> - Run on schedule (nightly) + release tags

> **NEVER**: Use real production databases, skip cleanup after tests, run on every PR (too slow)

### 4.3 Release Automation

> **Semantic Versioning**:
> - Maven: maven-release-plugin
> - Gradle: semantic-release or nebula-release
> - Generate CHANGELOG from conventional commits
> - Create GitHub Releases with notes
> - Publish to Maven Central (if library)

### 4.4 Maven Central Publishing

> **If creating libraries**:
> - Configure GPG signing
> - Publish to OSSRH (Sonatype)
> - Include sources and javadoc JARs
> - Validate POM metadata

> **ALWAYS**: Set groupId, artifactId, version; Include license, developers, SCM; Sign artifacts with GPG key

### 4.5 Notifications

> **ALWAYS**: Slack/Teams webhook on deploy success/failure, GitHub Status Checks for PR reviews, Email notifications for security alerts

**Verify**: Benchmarks run and tracked, integration tests pass in isolation, releases created automatically, Maven Central publish works (if applicable), notifications received

---

## Framework-Specific Notes

| Framework | Notes |
|-----------|-------|
| **Spring Boot** | Package: `mvn spring-boot:build-image` for OCI image; Health: `/actuator/health`; Metrics: `/actuator/metrics` (Prometheus, Micrometer); Profile: `-Dspring.profiles.active=production` |
| **Quarkus** | Native build: `mvn package -Pnative` (GraalVM); JVM build: `mvn package` (uber-jar); Extremely fast startup (ideal for serverless) |
| **Micronaut** | Build: `gradle shadowJar` (fat JAR); Native: GraalVM native-image; Health: `/health`; Lightweight and cloud-native |
| **Jakarta EE / Java EE** | Package as WAR for application servers (WildFly, Payara); Deploy using CLI or web console; Use MicroProfile for cloud-native features |
| **Android** | Build APK: `gradle assembleRelease`; Build AAB: `gradle bundleRelease`; Sign with keystore; Upload to Google Play Console |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Maven dependencies not resolving in CI** | Check settings.xml for auth, commit .mvn/wrapper files |
| **Gradle build fails with "daemon not found"** | Use `--no-daemon` flag in CI |
| **Tests pass locally but fail in CI** | Check timezone, locale, file paths (absolute vs relative) |
| **Deployment fails with "port already in use"** | Gracefully stop old process before starting new one |
| **Coverage reports not generated** | Ensure JaCoCo plugin configured, run `verify` phase (Maven) |
| **Want to use Jenkins / GitLab CI / Azure DevOps** | Jenkins: Use Jenkinsfile with Maven/Gradle plugins; GitLab CI: Use `maven:3.9-eclipse-temurin-21` or `gradle:jdk21` image; Azure DevOps: Use `Maven@3` or `Gradle@2` tasks - core concepts remain same |

---

## AI Self-Check

- [ ] CI pipeline runs on push and PR
- [ ] Java version pinned (in workflow)
- [ ] Builds succeed without warnings
- [ ] All tests pass with coverage ≥80%
- [ ] Code quality tools enabled (Checkstyle, PMD, SpotBugs)
- [ ] Security scanning enabled (CodeQL, Dependabot, OWASP)
- [ ] Artifacts packaged with correct versioning
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
