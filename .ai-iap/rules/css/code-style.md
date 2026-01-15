# CSS Code Style

> **Scope**: Apply these rules ONLY when working with stylesheets (`*.css`, `*.scss`, etc.) and style sections in component files (e.g., `.vue`, `.svelte`). These extend the general code style guidelines.

## 1. Formatting
- **Indentation**: 2 spaces.
- **One selector per line**: Prefer multi-line blocks for readability.
- **Ordering**: Group properties consistently (layout → box model → typography → visuals → animation).

## 2. Best Practices
- **Prefer**: CSS variables for repeated values.
- **Prefer**: `rem` for font sizing and spacing; use `px` only when needed.
- **Avoid**: Deep descendant selectors (e.g., `.a .b .c .d`).
- **Avoid**: Styling by tag selectors except for base styles (e.g., `h1`, `p` in a reset layer).
- **NEVER**: Use `!important`.

## 3. Performance
- **Avoid**: Expensive selectors (overly generic descendant chains).
- **Prefer**: Classes and shallow selectors.

## 4. Component SFCs (Vue/Svelte)
- **Prefer**: Scoped styles where supported.
- **Never**: Put application-wide resets inside a component’s `<style>` block.

