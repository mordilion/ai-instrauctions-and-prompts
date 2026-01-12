# Dockerfile Code Style

> **Scope**: Dockerfile formatting and maintainability rules.

## 1. Readability
- **ALWAYS**: Use uppercase Dockerfile instructions (`FROM`, `RUN`, `COPY`, `ENV`).
- **Prefer**: One logical step per `RUN` (but combine related commands to reduce layers).

## 2. Layer efficiency
- **Prefer**: Combine package install + cleanup in the same `RUN`.
- **ALWAYS**: Clean package manager caches in the same layer that created them.

## 3. Determinism
- **Prefer**: Explicit versions for package installs where feasible.
- **NEVER**: Use `curl | sh` style installs.

