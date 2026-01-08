# Documentation Pattern for All On-Demand Processes

> **Status**: Pattern established for all 70 on-demand process files  
> **Date**: 2026-01-08

---

## Overview

All on-demand process files follow a **catch-up documentation pattern** that enables:
1. **Continuity**: AI can resume where previous work stopped
2. **Team onboarding**: New team members understand what was set up
3. **Maintenance**: Clear documentation for future updates

---

## Universal Documentation Files

These files are used by **ALL** processes:

### 1. PROJECT-MEMORY.md
**Purpose**: Record key decisions, versions, and lessons learned

**Template**:
```markdown
# {Process Name} Memory

## Detected Versions
- {Tool/Language}: {version}
- {Dependencies}: {versions}

## Framework/Tool Choices
- {Choice}: {Tool} v{version}
- Why: {reason}

## Key Decisions
- {Decision 1}
- {Decision 2}

## Lessons Learned
- {Challenge encountered}
- {Solution that worked}
```

### 2. LOGIC-ANOMALIES.md
**Purpose**: Log bugs found but NOT fixed during setup

**Template**:
```markdown
# Logic Anomalies Found

## Bugs Discovered (Not Fixed)
1. **File**: path/to/file
   **Issue**: Description
   **Impact**: Severity
   **Note**: Logged only, not fixed during setup

## Code Smells
- {Areas needing refactoring}

## Missing Tests/Coverage
- {Components needing attention}
```

---

## Process-Specific Documentation Files

Each process category has its own setup guide:

| Process | Documentation File | Purpose |
|---------|-------------------|---------|
| test-implementation | `TESTING-SETUP.md` | How to run tests, mock strategies, coverage |
| ci-cd-github-actions | `CI-CD-SETUP.md` | How pipelines work, secrets, deployment |
| code-coverage | `COVERAGE-SETUP.md` | Coverage tools, thresholds, reports |
| docker-containerization | `DOCKER-SETUP.md` | Image details, build process, deployment |
| logging-observability | `LOGGING-SETUP.md` | Log levels, monitoring, dashboards |
| linting-formatting | `LINTING-SETUP.md` | Linter config, format rules, pre-commit |
| security-scanning | `SECURITY-SETUP.md` | Scan tools, vulnerability handling |
| api-documentation-openapi | `API-DOCS-SETUP.md` | OpenAPI spec location, update process |
| authentication-jwt-oauth | `AUTH-SETUP.md` | Auth flow, token management, OAuth config |

---

## File Structure in Prompts

All on-demand process files follow this structure:

```
1. CONTEXT
   - What you're implementing
   - Critical requirements

2. TECH STACK (if applicable)
   - Recommended tools and alternatives

3. CATCH-UP: READ EXISTING DOCUMENTATION ⬅️ NEW
   - Instructions to read PROJECT-MEMORY.md
   - Instructions to read LOGIC-ANOMALIES.md
   - Instructions to read {PROCESS}-SETUP.md
   - Use info to continue from where work stopped

4. PHASE 1-4
   - Implementation phases

5. DOCUMENTATION ⬅️ ENHANCED
   - Templates for all 3 documentation files
   - PROJECT-MEMORY.md template
   - LOGIC-ANOMALIES.md template
   - {PROCESS}-SETUP.md template

6. EXECUTION
   - START: Read existing docs (CATCH-UP)  ⬅️ NEW
   - CONTINUE: Execute phases
   - FINISH: Update documentation
   - REMEMBER: Key points + "document for catch-up"
```

---

## Benefits

✅ **Continuity**: Next AI session can resume work seamlessly  
✅ **Team Knowledge**: Complete setup documentation exists  
✅ **Maintenance**: Clear guide for future updates  
✅ **Audit Trail**: Bugs found logged for product team  
✅ **Consistency**: Same pattern across all 70 processes  

---

## Implementation Status

- [x] Pattern designed
- [x] TypeScript test-implementation updated (template)
- [ ] Remaining 69 files to update
- [ ] Commit all changes

---

## Next Steps

1. Apply pattern to all 69 remaining files
2. Test with real AI session to verify catch-up works
3. Update setup scripts to create empty doc files
4. Add to team adoption guide
