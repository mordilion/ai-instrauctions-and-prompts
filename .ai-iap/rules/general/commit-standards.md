# Commit Message Standards

> **Reference**: [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)

---

## Format

> **ALWAYS** follow this structure:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

---

## Commit Types

| Type | Purpose | Semantic Version |
|------|---------|------------------|
| `feat` | New feature | MINOR |
| `fix` | Bug fix | PATCH |
| `docs` | Documentation only | - |
| `style` | Formatting, whitespace | - |
| `refactor` | Code restructuring | - |
| `perf` | Performance improvement | - |
| `test` | Add/update tests | - |
| `build` | Build system, dependencies | - |
| `ci` | CI configuration | - |
| `chore` | Maintenance, tooling | - |
| `revert` | Revert previous commit | - |

---

## Description Rules

> **ALWAYS**: Imperative mood ("add" NOT "added")  
> **ALWAYS**: Lowercase start  
> **ALWAYS**: No period at end  
> **ALWAYS**: Under 72 characters  
> **NEVER**: Past tense  
> **NEVER**: Capitalize first letter

**Examples**:
```
feat(auth): add OAuth 2.0 support
fix(api): handle null response
docs: update installation guide
```

---

## Scope (Optional)

> **Format**: `type(scope): description`

Common scopes: `api`, `auth`, `db`, `ui`, `docs`, `deps`, `config`

---

## Breaking Changes

> **ALWAYS**: Mark with `!` or `BREAKING CHANGE:` footer

**Method 1** (Exclamation):
```
feat!: remove Node 6 support
```

**Method 2** (Footer):
```
feat: change API format

BREAKING CHANGE: response structure changed
```

---

## Body (Optional)

> **When**: Additional context needed  
> **Format**: One blank line after description

```
feat(api): add pagination

Implement cursor-based pagination for better performance
with large datasets.
```

---

## Footer (Optional)

| Footer | Usage |
|--------|-------|
| `BREAKING CHANGE:` | Breaking changes |
| `Refs:` | Reference issues |
| `Closes:` | Close issues |
| `Fixes:` | Fix issues |
| `Reviewed-by:` | Reviewer name |

**Example**:
```
fix: resolve timeout issue

Refs: #123
Reviewed-by: Jane Doe
```

---

## Common Mistakes

| ❌ Wrong | ✅ Correct |
|----------|-----------|
| `feat: Added auth` | `feat: add auth` |
| `fix: Resolve bug` | `fix: resolve bug` |
| `docs: update.` | `docs: update` |
| `feat: add auth, update docs` | Separate commits |

---

## AI Self-Check

- [ ] Valid type (`feat`, `fix`, `docs`, etc.)
- [ ] Imperative mood ("add" not "added")
- [ ] Lowercase start, no period
- [ ] Under 72 characters
- [ ] Breaking changes marked (`!` or footer)
- [ ] One logical change per commit
- [ ] Scope provided when applicable

---

## Tools

- **commitlint**: Enforce standards
- **semantic-release**: Automate versioning  
- **conventional-changelog**: Generate CHANGELOG

