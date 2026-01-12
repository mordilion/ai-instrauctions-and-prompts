# PostCSS Architecture

> **Scope**: Apply these rules when working with PostCSS stylesheets (`*.pcss`, `*.postcss`). These extend the general architecture guidelines.

## 1. Core Principles
- **Prefer explicit plugins**: Keep the plugin chain small and purposeful.
- **Design tokens first**: Prefer CSS variables where possible; document token sources.
- **Avoid magic**: Prefer readable, explicit transformations over surprising implicit behavior.

## 2. Layering
- **Base layers**: tokens → utilities → components → pages.
- **Keep processing predictable**: Ensure the same build pipeline is used across environments.

