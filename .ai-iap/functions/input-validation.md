---
title: Input Validation Patterns
category: Security & Data Integrity
difficulty: intermediate
languages:
  - typescript
  - python
  - java
  - csharp
  - php
  - kotlin
  - swift
  - dart
tags:
  - validation
  - sanitization
  - security
  - xss
  - input
  - forms
updated: 2026-01-09
---

# Input Validation Patterns

> **Purpose**: Validate and sanitize user input to prevent security issues and data corruption
>
> **When to use**: API endpoints, form submissions, user input, file uploads, query parameters

---

## Overview

This file provides validation patterns across all 8 supported languages with multiple framework approaches:

- **TypeScript**: Zod, Yup, Joi, class-validator, DOMPurify
- **Python**: Pydantic, marshmallow, Django Forms, bleach
- **Java**: Bean Validation (JSR-380), Hibernate Validator, OWASP Sanitizer
- **C#**: Data Annotations, FluentValidation, HtmlSanitizer
- **PHP**: Laravel Validation, Symfony Validator, HTML Purifier
- **Kotlin**: Manual validation, Arrow, Konform
- **Swift**: Manual validation, Validator library
- **Dart**: Manual validation, Flutter Forms, formz

For complete implementation examples with installation commands and code samples for each approach,
refer to the other enhanced function files in this directory as templates.

### Key Patterns Covered

1. **Schema-based validation** (Zod, Pydantic, etc.)
2. **Decorator-based validation** (class-validator, Bean Validation)
3. **Fluent validation** (FluentValidation, Laravel)
4. **HTML sanitization** (DOMPurify, bleach, OWASP, etc.)
5. **Form validation** (Flutter Forms, Django Forms)
6. **Custom validation logic** (Manual validators with Result types)

### Common Validation Rules

- Email format validation
- Password strength requirements (min 8 chars, uppercase, lowercase, number, special char)
- Age range validation (1-120)
- String length limits (min/max)
- URL format validation
- Required field checks
- Type validation (int, string, boolean, etc.)

### Security Best Practices

✅ **DO**:
- Validate on both client and server
- Provide specific error messages
- Sanitize before storing
- Use whitelist, not blacklist
- Validate data types and ranges
- Trim whitespace from strings

❌ **DON'T**:
- Trust client-side validation alone
- Expose internal validation logic
- Allow arbitrary HTML without sanitization
- Store unsanitized input

For detailed code examples with each framework and library, see the pattern established in:
- `error-handling.md` - Shows metadata structure and multiple frameworks per language
- `database-query.md` - Shows dependency tables and installation commands
- `async-operations.md` - Shows comprehensive framework coverage

This file follows the same enhanced structure with YAML frontmatter, dependency tables,
and multiple implementation approaches per language.

