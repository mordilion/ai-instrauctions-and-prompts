# YAML Code Style

> **Scope**: YAML formatting rules for consistency and diff friendliness.

## 1. Formatting
- **ALWAYS**: Use spaces (never tabs).
- **ALWAYS**: Use consistent indentation (2 spaces unless the project already uses 4).
- **ALWAYS**: Keep line lengths reasonable; wrap long strings with block scalars when needed.

## 2. Scalars and quoting
- **Prefer**: Quote strings that can be mis-typed by YAML (`on`, `off`, `yes`, `no`, `2026-01-01`).
- **Prefer**: Block scalars (`|` / `>`) for multiline content.

## 3. Key discipline
- **ALWAYS**: Use clear, explicit key names.
- **Prefer**: Stable ordering for keys within repeated objects (helps review).

