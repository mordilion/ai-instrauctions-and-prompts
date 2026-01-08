# CI/CD Implementation Process - .NET/C# (GitHub Actions)

> **Purpose**: Establish comprehensive CI/CD pipeline with GitHub Actions for .NET applications

> **Platform**: This guide is for **GitHub Actions**. For GitLab CI, Azure DevOps, CircleCI, or Jenkins, adapt the workflow syntax accordingly.

---

## Prerequisites

> **BEFORE starting**:
> - Working .NET application
> - Git repository with remote (GitHub)
> - Solution (.sln) and project files (.csproj)
> - Tests exist (xUnit/NUnit/MSTest)
> - .NET version defined in .csproj or global.json

---

## Workflow Adaptation

> **IMPORTANT**: Phases below focus on OBJECTIVES. Use your team's workflow.  
> See [Git Workflow Adaptation Guide](../_templates/git-workflow-adaptation.md) for examples.

---

## Phase 1: Basic CI Pipeline

**Objective**: Establish foundational CI pipeline with build, lint, and test automation

### 1.1 Basic Build & Test Workflow

> **ALWAYS include**:
> - .NET version from project (read from `global.json` or .csproj `<TargetFramework>`)
> - Restore dependencies (`dotnet restore`)
> - Build solution (`dotnet build --no-restore`)
> - Run tests (`dotnet test --no-build --verbosity normal`)
> - Collect coverage (coverlet, ReportGenerator)

> **Version Strategy**:
> - **Best**: Use `global.json` to pin SDK version: `dotnet new globaljson --sdk-version 8.0.100`
> - **Good**: Read from .csproj `<TargetFramework>` (net8.0, net6.0, etc.)
> - **Avoid**: Hardcoding version without project config

> **NEVER**:
> - Skip `--no-restore` and `--no-build` flags (wasteful rebuilds)
> - Use outdated .NET versions in production
> - Run tests without proper logger output
> - Forget to include test projects in solution

**Key Workflow Structure**:
- Trigger: push (main/develop), pull_request
- Jobs: build → test → publish
- Setup: actions/setup-dotnet@v3
- Cache: NuGet packages by packages.lock.json

### 1.2 Coverage Reporting

> **ALWAYS**:
> - Use coverlet.collector or coverlet.msbuild
> - Generate reports with ReportGenerator
> - Upload to Codecov/Coveralls
> - Set minimum coverage threshold (80%+)
> - Use `--collect:"XPlat Code Coverage"` flag

**Coverage Commands**:
```bash
dotnet test --collect:"XPlat Code Coverage" --results-directory ./coverage
reportgenerator -reports:./coverage/**/coverage.cobertura.xml -targetdir:./coverage/report -reporttypes:Html
```

**Verify**: Pipeline runs, all projects build, tests execute with results, coverage report generated, NuGet cache working

---

## Phase 2: Code Quality & Security

**Objective**: Add code quality and security scanning to CI pipeline

### 2.1 Code Quality Analysis

> **ALWAYS include**:
> - StyleCop Analyzers (StyleCop.Analyzers NuGet)
> - Roslynator or SonarAnalyzer.CSharp
> - Treat warnings as errors in CI (`<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`)
> - EditorConfig enforcement

> **NEVER**:
> - Suppress warnings globally
> - Skip analyzer configuration
> - Allow SA violations in new code

### 2.2 Dependency Security Scanning

> **ALWAYS include**:
> - Dependabot configuration (`.github/dependabot.yml`)
> - NuGet package vulnerability scanning
> - Target framework updates
> - Fail on known vulnerabilities

**Dependabot Config**:
```yaml
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 2.3 Static Analysis (SAST)

> **ALWAYS**:
> - Add CodeQL analysis (`.github/workflows/codeql.yml`)
> - Configure language: csharp
> - Run on schedule (weekly) + push to main

> **Optional but recommended**: SonarCloud, Security Code Scan analyzer, Meziantou.Analyzer

**Verify**: Analyzers run during build, warnings treated as errors, Dependabot creates update PRs, CodeQL scan completes

---

## Phase 3: Deployment Pipeline

**Objective**: Automate app deployment to relevant environments/platforms

### 3.1 Environment Configuration

> **ALWAYS**:
> - Define environments: Development, Staging, Production
> - Use GitHub Environments with protection rules
> - Store secrets per environment (connection strings, API keys)
> - Use User Secrets for local dev

**Protection Rules**:
- Production: require approval, restrict to main branch
- Staging: auto-deploy on merge to develop
- Development: auto-deploy on feature branches

### 3.2 Build & Publish Artifacts

> **ALWAYS**:
> - Publish with `dotnet publish -c Release -o ./publish`
> - Include runtime identifier if self-contained (`-r linux-x64`)
> - Version assemblies (AssemblyVersion, FileVersion, InformationalVersion)
> - Upload artifacts with retention policy

> **NEVER**:
> - Include appsettings.Development.json in artifacts
> - Publish without trimming/optimization
> - Ship PDB files to production (upload separately for debugging)

**Publish Commands**:
```bash
dotnet publish -c Release -o ./publish --no-restore
# Self-contained: add --self-contained -r linux-x64
# Framework-dependent: add --no-self-contained
```

### 3.3 Deployment Jobs

> **Platform-specific** (choose one or more):

| Platform | Tool | Notes |
|----------|------|-------|
| **Azure App Service** | azure/webapps-deploy@v2 | Use deployment slots (staging → production swap) |
| **Azure Container Apps/AKS** | Docker + ACR | Deploy with az containerapp update or kubectl |
| **AWS** | aws-actions/configure-aws-credentials | Elastic Beanstalk, ECS, or Lambda |
| **Docker Registry** | docker/build-push-action | Push to Docker Hub, GHCR, ACR, ECR |
| **IIS / Windows Server** | Web Deploy (MSDeploy) | PowerShell remoting or sftp/scp artifacts |

### 3.4 Database Migrations

> **ALWAYS**:
> - Run EF Core migrations before app deployment
> - Use `dotnet ef database update` or SQL scripts
> - Test migrations in staging first
> - Create rollback scripts

> **NEVER**:
> - Run migrations automatically on app start in production
> - Skip migration testing
> - Deploy app before migrations complete

**Migration Commands**:
```bash
dotnet ef migrations script --idempotent --output migration.sql
# Execute SQL script in deployment job
```

### 3.5 Smoke Tests Post-Deploy

> **ALWAYS include**:
> - Health check endpoint test (`/health` or `/healthz`)
> - Database connectivity check
> - Cache/Redis connectivity
> - External API integration check

**Verify**: Manual trigger works, environment secrets accessible, artifacts published, deployment succeeds to staging, migrations applied, smoke tests pass, rollback tested

---

## Phase 4: Advanced Features

**Objective**: Add advanced CI/CD capabilities (integration tests, release automation)

### 4.1 Performance Testing

> **ALWAYS**:
> - BenchmarkDotNet for micro-benchmarks
> - Load testing with k6, JMeter, or NBomber
> - Track response times and memory usage
> - Fail if performance degrades >10%

### 4.2 Integration Testing

> **ALWAYS**:
> - Separate workflow (`integration-tests.yml`)
> - Use TestContainers for database/Redis
> - WebApplicationFactory for in-memory testing
> - Run on schedule (nightly) + release tags

> **NEVER**:
> - Use real production databases
> - Skip cleanup after tests
> - Run on every PR (too slow)

### 4.3 Release Automation

> **Semantic Versioning**:
> - Use GitVersion or Nerdbank.GitVersioning
> - Generate CHANGELOG from conventional commits
> - Create GitHub Releases with notes
> - Publish NuGet packages (if library)

**Version Strategies**: Mainline mode (continuous delivery), GitFlow mode (release branches), Tag-based (manual version bumps)

### 4.4 NuGet Package Publishing

> **If creating libraries**:
> - Pack with `dotnet pack -c Release`
> - Include symbols package (.snupkg)
> - Publish to NuGet.org or private feed (Azure Artifacts, GitHub Packages)
> - Validate package contents before publish

> **ALWAYS**: Set PackageVersion, Authors, Description, License; Include README.md in package; Sign packages (optional but recommended)

### 4.5 Notifications

> **ALWAYS**:
> - Teams/Slack webhook on deploy success/failure
> - GitHub Status Checks for PR reviews
> - Email notifications for security alerts

**Verify**: Benchmarks run and tracked, integration tests pass in isolation, releases created automatically, NuGet packages published (if applicable), notifications received

---

## Framework-Specific Notes

| Framework | Notes |
|-----------|-------|
| **ASP.NET Core Web API** | Health checks: `app.MapHealthChecks("/health")`, Swagger/OpenAPI, Use Kestrel with reverse proxy (nginx/IIS) |
| **ASP.NET Core MVC / Razor Pages** | Static files bundling (CSS/JS minification), Use CDN for static assets in production |
| **Blazor (Server / WebAssembly)** | Blazor Server: standard publish; Blazor WASM: outputs to wwwroot, pre-compression (Brotli), deploy to Azure Static Web Apps or Cloudflare Pages |
| **.NET MAUI** | Sign Android APK/AAB with keystore, Sign iOS with provisioning profile, Publish to App Store / Google Play |
| **Worker Services** | Use systemd (Linux) or Windows Service, Health check port for monitoring, Graceful shutdown handling |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **NuGet restore fails in CI** | Commit packages.lock.json, use authenticated feeds with secrets |
| **Tests pass locally but fail in CI** | Check timezone, culture info, connection strings in appsettings.json |
| **Coverage not collected** | Install coverlet.collector, use `--collect:"XPlat Code Coverage"` |
| **Deployment fails with permission error** | Verify service principal roles (Azure), IAM policies (AWS) |
| **Migrations fail with timeout** | Increase command timeout, split large migrations, check firewall rules |
| **Want to use Azure DevOps / GitLab CI** | Azure DevOps: Use `DotNetCoreCLI@2` tasks in azure-pipelines.yml; GitLab CI: Use `mcr.microsoft.com/dotnet/sdk` image - core concepts remain same |

---

## AI Self-Check

- [ ] CI pipeline runs on push and PR
- [ ] .NET SDK version pinned (global.json)
- [ ] Solution builds without warnings (TreatWarningsAsErrors)
- [ ] All tests pass with coverage ≥80%
- [ ] Code analyzers enabled (StyleCop, Roslynator)
- [ ] Security scanning enabled (CodeQL, Dependabot)
- [ ] Artifacts published with correct versioning
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
