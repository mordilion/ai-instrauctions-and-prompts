# Stylus Architecture

> **Scope**: Apply these rules when working with Stylus stylesheets (`*.styl`, `*.stylus`). These extend the general architecture guidelines.

## 1. Core Principles
- **Prefer consistency**: Stylus is flexible; pick a consistent convention and keep it.
- **Design tokens first**: Centralize colors/spacing/typography as variables or CSS variables.
- **Avoid global leakage**: Keep global selectors minimal; prefer modular structure.

## 2. Layering
- **Base layers**: tokens → mixins/functions → utilities → components → pages.
- **Keep dependencies one-way**: Low-level modules must not depend on higher-level modules.

