# SQL Architecture

> **Scope**: SQL usage patterns for apps and migrations.  
> **Extends**: General architecture rules

## 1. Treat migrations as production code
- **ALWAYS**: Version migrations; apply them in CI before deploy.
- **Prefer**: Reversible migrations (up/down) or clearly documented non-reversible steps.

## 2. Separate concerns
- **Prefer**: DDL (schema) changes separated from large DML backfills.
- **Prefer**: Small, incremental migrations to reduce risk.

## 3. Performance awareness
- **ALWAYS**: Add indexes intentionally; document purpose.
- **Prefer**: Use query plans/EXPLAIN for critical queries.

