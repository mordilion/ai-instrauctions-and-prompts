# Sass/SCSS Architecture

> **Scope**: Apply these rules when working with Sass/SCSS stylesheets (`*.scss`, `*.sass`). These extend the general architecture guidelines.

## 1. Core Principles
- **Prefer modern Sass modules**: Use `@use` / `@forward`; avoid legacy `@import`.
- **Design tokens first**: Centralize colors/spacing/typography as variables or CSS variables.
- **Avoid global leakage**: Keep globals minimal; prefer modules, CSS Modules, or scoped component styles where supported.

## 2. Layering
- **Base layers**: tokens → mixins/functions → utilities → components → pages.
- **Keep dependencies one-way**: Low-level modules (`tokens`) must not import higher-level modules (`components`).

## 3. Composition
- **Prefer**: Mixins/functions for reuse instead of copy/paste.
- **Avoid**: Deep nesting; keep selectors shallow and predictable.

