# dotenv (.env) Architecture

> **Scope**: `.env` file usage patterns and layering.  
> **Extends**: General architecture rules

## 1. Purpose
- **Use**: `.env` for local/dev configuration and non-secret defaults.
- **Prefer**: Secret managers for real secrets, especially in CI/CD and production.

## 2. Layering
- **Prefer**: Clear precedence rules (`.env` → `.env.local` → `.env.<env>`), documented in README.
- **ALWAYS**: Provide a `.env.example` (or equivalent) that contains keys but no real secrets.

