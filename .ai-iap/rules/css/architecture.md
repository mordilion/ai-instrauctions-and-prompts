# CSS Architecture

> **Scope**: Apply these rules ONLY when working with stylesheets (`*.css`, `*.scss`, etc.) and style sections in component files (e.g., `.vue`, `.svelte`). These extend the general architecture guidelines.

## 1. Core Principles
- **Predictable cascade**: Keep specificity low and consistent. Avoid `!important`.
- **Design tokens first**: Prefer CSS variables (or preprocessor variables) for colors, spacing, typography.
- **Component-first**: Co-locate component styles with the component when using component SFCs; otherwise organize by feature/module.

## 2. File & Layering Strategy
- **Base layers** (common): reset/normalize → tokens → utilities → components → pages.
- **Avoid global leakage**: Prefer scoped styles (CSS Modules, Vue/Svelte scoped) where supported.
- **Reuse**: Use utilities for repeated patterns (spacing, flex/grid helpers).

## 3. Naming Conventions
- **Classes**: Prefer `kebab-case`.
- **BEM**: Allowed for complex components (`block__element--modifier`).
- **No JS hooks**: Prefer `data-*` for JS selectors; do not style by JS hook classes.

## 4. Responsive & Theming
- **Mobile-first**: Use `min-width` media queries where possible.
- **Themes**: Drive theming with CSS variables on `:root` or `[data-theme="..."]`.

