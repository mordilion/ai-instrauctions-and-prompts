# SQL Code Style

> **Scope**: SQL formatting rules for clarity and maintainability.

## 1. Formatting
- **Prefer**: Uppercase keywords (`SELECT`, `WHERE`, `JOIN`) for readability.
- **ALWAYS**: Use consistent indentation for nested queries and joins.

## 2. Naming
- **Prefer**: `snake_case` for table/column names (unless the project uses a different convention).
- **ALWAYS**: Use explicit column lists (avoid `SELECT *` in application queries).

## 3. Migrations
- **ALWAYS**: Add comments for risky operations (large backfills, table rewrites).

