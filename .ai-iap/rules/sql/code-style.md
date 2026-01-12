# SQL Code Style

> **Scope**: SQL formatting rules for clarity and maintainability.

## 1. Formatting
- **Prefer**: Uppercase SQL keywords for readability (examples: `SELECT`, `FROM`, `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `ALTER`, `DROP`, etc.). This list is **not exhaustive**.
- **ALWAYS**: Use consistent indentation for nested queries and joins.

## 2. Naming
- **Prefer**: `snake_case` for table/column names (unless the project uses a different convention).
- **ALWAYS**: Use explicit column lists (avoid `SELECT *` in application queries).

## 3. Migrations
- **ALWAYS**: Add comments for risky operations (large backfills, table rewrites).

