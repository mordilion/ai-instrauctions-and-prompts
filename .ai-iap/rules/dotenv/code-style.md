# dotenv (.env) Code Style

> **Scope**: Formatting rules for `.env` files.

## 1. Key/value format
- **ALWAYS**: Use `KEY=VALUE` (no spaces around `=`).
- **Prefer**: Uppercase keys with underscores (`DATABASE_URL`, `JWT_SECRET`).
- **Prefer**: One key per line.

## 2. Values
- **Prefer**: Quote values that contain spaces or special characters.
- **NEVER**: Add shell syntax that only works in one shell (keep it dotenv-compatible).

## 3. Comments
- **Prefer**: Use `#` comments sparingly to document non-obvious keys.

