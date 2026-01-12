# Sass/SCSS Code Style

> **Scope**: Sass/SCSS formatting and maintainability rules.

## 1. Formatting
- **Indentation**: 2 spaces.
- **One selector per line**: Prefer multi-line blocks.
- **Ordering**: Group properties consistently (layout → box model → typography → visuals → animation).

## 2. Nesting
- **Avoid**: Nesting deeper than 2 levels.
- **Prefer**: `&` for modifiers (`&--active`) and state classes (`&.is-open`).

## 3. Modules
- **Prefer**: `@use` / `@forward`.
- **Avoid**: Global `!default` overrides spread across the codebase; centralize configuration.

