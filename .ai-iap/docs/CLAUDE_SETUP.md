# Claude Setup Guide

> **Reference**: 
> - [Claude Memory Documentation](https://code.claude.com/docs/en/memory)
> - [Claude Skills Documentation](https://code.claude.com/docs/en/skills)

---

## Overview

Claude supports two powerful features for context management:

1. **Memory** - Persistent knowledge stored in Claude's knowledge base
2. **Skills** - Auto-triggered instructions based on project context

This setup routine generates **project rules** under `.claude/rules/` for Claude Code.

---

## Generated Files

### 1. .claude/rules/ (Project Rules)

**Location**: `.claude/rules/{category}/{framework}.md`

**Purpose**: Modular, topic-specific instructions that are **automatically loaded** by Claude Code

**Structure**:
```
.claude/rules/
├── core/
│   ├── general/
│   │   ├── persona.md
│   │   └── security.md
│   ├── typescript/
│   │   ├── code-style.md
│   │   └── testing.md
│   └── documentation/
│       ├── documentation-code.md
│       └── documentation-project.md
├── frontend/
│   ├── react.md
│   ├── vue.md
│   └── angular.md
├── backend/
│   ├── express.md
│   ├── django.md
│   └── spring.md
├── mobile/
│   └── flutter.md
└── processes/
    ├── typescript-database-migrations.md
    └── python-testing.md
```

**Rule File Format** (with optional path-specific frontmatter):
```markdown
---
paths: **/*.{jsx,tsx}
---

# React Framework Standards

...React-specific rules that apply to .jsx and .tsx files...
```

---

## How Claude Uses These Files

Claude Code supports:

1. **Project memory** via `CLAUDE.md` or `.claude/CLAUDE.md` (optional)
2. **Project rules** via `.claude/rules/*.md`

Per the official docs, all `.md` files in `.claude/rules/` are loaded automatically, and rules can be scoped with `paths:` frontmatter. This setup generates **rules-only** under `.claude/rules/` (including “always-on” rules under `.claude/rules/core/`).  
Reference: [Claude Memory Documentation](https://code.claude.com/docs/en/memory)

**Example**: A rule with `paths: **/*.{jsx,tsx}` only applies when working with React component files.

---

## Setup Process

### Run Setup Script

```bash
# Linux/Mac
./.ai-iap/setup.sh

# Windows
.\.ai-iap\setup.ps1
```

### Select Options

1. **Languages**: TypeScript, Python, Java, etc.
2. **Frameworks**: React, Django, Spring Boot, etc.
3. **Structures**: Feature-First, Clean Architecture, etc.
4. **Processes**: Database Migrations, Testing, CI/CD, etc.

### Generated Output

✅ **.claude/rules/core/** - Core rules (always apply)
✅ **.claude/rules/** - Modular rules organized by category (frontend/, backend/, processes/, etc.)

---

## Rule Types Generated

### 1. Framework Rules

**Location**: `.claude/rules/{category}/{framework}.md`

**Example**: `.claude/rules/frontend/react.md`

**Path-Specific**: Rules can target specific file types

```yaml
---
paths: **/*.{jsx,tsx}
---

# React Framework Standards

...React component guidelines...
```

### 2. Structure Rules

**Location**: `.claude/rules/{category}/{framework}-{structure}.md`

**Example**: `.claude/rules/frontend/react-feature.md`

**Purpose**: Architectural patterns for specific frameworks

```yaml
---
paths: src/**/*.{jsx,tsx}
---

# Feature-First Architecture for React

...feature organization guidelines...
```

### 3. Process Rules

**Location**: `.claude/rules/processes/{language}-{process}.md`

**Example**: `.claude/rules/processes/typescript-database-migrations.md`

**Purpose**: Implementation processes that are used repeatedly

**Note**: Only **permanent processes** (`loadIntoAI: true`) are included as rules. On-demand processes are copied manually when needed.

---

## Using Claude's Memory Feature

### Storing Project-Specific Information

Claude can remember project-specific details using the Memory feature:

**How to Add Memory**:
1. During conversation: "Remember that we use feature-first architecture"
2. Claude stores this in its knowledge base
3. Future conversations automatically include this context

**What to Remember**:
- Project-specific naming conventions
- Custom architecture decisions
- Team preferences
- Technology stack details
- Deployment procedures

**Example**:
```
You: "Remember that we use PostgreSQL with Prisma ORM, 
and all migrations must include rollback logic"

Claude: "I've noted that. I'll ensure all database 
migration guidance includes PostgreSQL-specific advice 
and Prisma migration commands with rollback support."
```

---

## Best Practices

### Rules (.claude/rules/)

✅ **DO**:
- Put “always-on” standards in `.claude/rules/core/` without `paths:`
- Create focused rules per topic (React, Vue, Django)
- Use descriptive filenames
- Add `paths:` frontmatter for file-specific rules
- Organize by category subdirectories

❌ **DON'T**:
- Duplicate the same rules across multiple files
- Make rules too broad
- Forget to update after framework changes

### Memory Feature

✅ **DO**:
- Store project-specific decisions
- Remember custom conventions
- Note technology stack choices
- Track team preferences

❌ **DON'T**:
- Store temporary information
- Duplicate what's in CLAUDE.md
- Include sensitive data

---

## Maintenance

### Updating Rules

```bash
# Re-run setup after changing rules
./ai-iap/setup.sh

# Rules are regenerated automatically
# Memory persists across setups
```

### Adding Custom Rules

Create custom rules manually:

```bash
mkdir -p .claude/rules/custom
```

```markdown
# .claude/rules/custom/company-standards.md
---
paths: src/**/*.ts
---

# Company-Specific Standards

...content...
```

### Clearing Memory

To reset Claude's memory for your project:
1. Open Claude
2. Settings → Memory
3. Clear project-specific memories

---

## Token Efficiency

### Before (everything in one file)

- All rules in CLAUDE.md: **50,000 tokens**
- Loaded in every conversation
- Difficult to maintain

### After (with modular rules)

- .claude/rules/core/: **Core rules** (always apply)
- .claude/rules/: **Modular organization** (frontend/, backend/, etc.)
- Path-specific rules only apply to relevant files
- **Better organization and maintainability**

---

## Troubleshooting

### Rules Not Loading

**Problem**: Custom rules don't seem to be active

**Solutions**:
1. Verify files are in `.claude/rules/` directory
2. Check file extension is `.md`
3. Ensure no syntax errors in YAML frontmatter
4. Re-run setup script to regenerate

### Path-Specific Rules Not Working

**Problem**: Rules with `paths:` don't apply to expected files

**Solutions**:
1. Check glob pattern syntax (e.g., `**/*.tsx`)
2. Test pattern matches your file paths
3. Ensure no leading/trailing whitespace in paths field
4. Try without paths field first to test rule loads

### CLAUDE.md Too Large

**Problem**: Token limit warnings

**Solutions**:
1. Move more content into smaller, focused `.claude/rules/` files
2. Use `paths:` frontmatter for file-specific rules to reduce what applies at once

### Memory Not Persisting

**Problem**: Claude forgets project details

**Solutions**:
1. Explicitly ask Claude to remember
2. Check memory settings in Claude
3. Re-state important context periodically

---

## Example Workflow

### Initial Setup

```bash
# 1. Run setup
./ai-iap/setup.sh

# 2. Select: TypeScript, React, Feature-First, Database Migrations

# 3. Generated:
✓ .claude/rules/core/typescript/code-style.md
✓ .claude/rules/frontend/react.md
✓ .claude/rules/frontend/react-feature.md
✓ .claude/rules/processes/typescript-database-migrations.md
```

### Using with Claude

```
You: "Remember we use Postgres with Prisma and feature-first structure"

Claude: "Noted. I'll use those standards."

You: "Create a new user authentication feature"

Claude: [Uses React rules from .claude/rules/frontend/ 
and feature-first architecture, applies Prisma + Postgres 
preferences from memory]
```

---

## Additional Resources

- **Claude Memory**: [https://code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)
- **Claude Skills**: [https://code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)
- **Main Project README**: [../README.md](../../README.md)
- **Customization Guide**: [../../CUSTOMIZATION.md](../../CUSTOMIZATION.md)

---

**Questions?** Open an issue on GitHub or consult the Claude documentation.
