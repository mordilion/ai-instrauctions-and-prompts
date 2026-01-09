# Functions Index - Cross-Language Implementation Patterns

> **Purpose**: Quick reference for common coding patterns across all supported languages
>
> **Usage**: Find the function you need, check language availability, use exact patterns to reduce AI guessing

---

## How to Use This Index

1. **Find your pattern** in the table below
2. **Check language support** for your project
3. **Open the function file** for complete implementation
4. **Choose your approach** - Plain language, Framework ORM, or specific library
5. **Check dependencies** - Each function file lists required packages and installation commands
6. **Copy the exact pattern** - don't let AI guess, use proven implementations

### 📦 What's Inside Each Function File

Each function file includes:
- **YAML frontmatter** with metadata (languages, tags, difficulty)
- **Multiple implementation approaches** per language (Plain, Framework 1, Framework 2, etc.)
- **Dependency tables** with installation commands for each approach
- **Real-world code examples** (5-20 lines each)
- **Best practices** and common pitfalls
- **Security checklists** where applicable

---

## Available Functions

| Function | Description | Languages | When to Use | File |
|----------|-------------|-----------|-------------|------|
| **Error Handling** | Exception handling, error propagation, custom errors | All 8 | When handling failures, validation errors, external API errors | [error-handling.md](error-handling.md) |
| **Async Operations** | Async/await, promises, concurrent execution | All 8 | When dealing with I/O, API calls, database queries | [async-operations.md](async-operations.md) |
| **Input Validation** | Data validation, sanitization, type checking | All 8 | When accepting user input, API requests, form data | [input-validation.md](input-validation.md) |
| **Database Queries** | Safe queries, parameterization, connection management | All 8 | When querying databases, preventing SQL injection | [database-query.md](database-query.md) |
| **HTTP Requests** | API calls, retry logic, timeout handling | All 8 | When consuming external APIs, microservice communication | [http-requests.md](http-requests.md) |

---

## Language Coverage

All functions cover these 8 languages:

- **TypeScript** / **JavaScript** (Node.js)
- **Python**
- **Java**
- **C# (.NET)**
- **PHP**
- **Kotlin**
- **Swift**
- **Dart** (Flutter)

---

## Framework & Library Versions

Each function file provides **multiple implementations** for the same pattern using different libraries/frameworks:

### Database Queries Example
- **TypeScript**: Plain pg, Prisma, TypeORM, Knex.js
- **Python**: Plain psycopg2, SQLAlchemy, Django ORM, asyncpg
- **PHP**: Plain PDO, Laravel Eloquent, Doctrine ORM
- **Java**: Plain JDBC, Spring Data JPA, Hibernate, jOOQ
- **C#**: Plain ADO.NET, Entity Framework Core, Dapper
- **Kotlin**: Plain JDBC, Exposed, Room (Android), Spring Data
- **Swift**: CoreData, SQLite.swift, GRDB
- **Dart**: Plain sqflite, Drift (Moor), Floor

### Error Handling Example
- **TypeScript**: Native try-catch, neverthrow (Result type), React Error Boundaries
- **Python**: Native try-except, result library
- **Kotlin**: Native try-catch, Built-in Result, Arrow (Either)
- **Swift**: Native do-catch, Built-in Result type

**Benefit**: Choose the approach that fits your project (plain for flexibility, framework for productivity)

---

## Quick Selection Guide

### By Use Case

**Building APIs?**
→ Start with: Error Handling, Input Validation, Database Queries

**Frontend Applications?**
→ Start with: Async Operations, HTTP Requests, Input Validation

**Backend Services?**
→ Start with: Database Queries, Error Handling, HTTP Requests

**Mobile Apps?**
→ Start with: Async Operations, HTTP Requests, Error Handling

---

## Pattern Benefits

✅ **Consistency**: Same pattern across all your projects
✅ **Security**: Validated, safe implementations
✅ **Performance**: Optimized for each language
✅ **Maintainability**: Well-documented, proven patterns
✅ **Reduced Errors**: Less guessing = fewer bugs

---

## How Functions Differ from Processes

| Aspect | Functions | Processes |
|--------|-----------|-----------|
| **Scope** | Single coding pattern | Complete workflow |
| **Size** | 5-20 lines of code | Multi-step implementation |
| **Languages** | All in one file | Separate file per language |
| **Usage** | Copy exact pattern | Follow step-by-step guide |
| **Example** | "How to handle errors" | "How to implement testing" |

---

## Adding Custom Functions

To add your own function patterns:

1. Create `custom-pattern.md` in `.ai-iap/functions/`
2. Follow existing file structure
3. Include all languages your team uses
4. Add entry to this INDEX
5. Update `.cursor/rules/` to reference it

---

## Quick Reference Checklist

Before implementing ANY function, ask:

- [ ] Is there a pattern in `/functions/` for this?
- [ ] Have I checked the INDEX?
- [ ] Am I using the exact pattern (not guessing)?
- [ ] Have I checked for language-specific variations?

**Remember**: Copy patterns = Consistent code. Guessing = Inconsistent code.

---

**Last Updated**: 2026-01-09
**Total Functions**: 5
**Languages Covered**: 8
