# Less Architecture

> **Scope**: Apply these rules when working with Less stylesheets (`*.less`, `*.module.less`). These extend the general architecture guidelines.

## 1. Core Principles
- **Design tokens first**: Centralize colors/spacing/typography as variables.
- **Avoid global leakage**: Keep global selectors minimal and predictable.
- **Prefer composition**: Use mixins for reuse and consistency.

## 2. Layering
- **Base layers**: tokens → mixins → utilities → components → pages.
- **Keep dependencies one-way**: Low-level modules must not depend on higher-level modules.

