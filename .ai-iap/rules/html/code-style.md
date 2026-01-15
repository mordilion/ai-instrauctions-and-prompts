# HTML Code Style

> **Scope**: Apply these rules ONLY when working with `.html` / `.htm` files and HTML/template sections in component files (e.g. `.vue`, `.svelte`). These extend the general code style guidelines.

## 1. Formatting
- **Indentation**: 2 spaces per level.
- **Attributes**: One per line when a tag has many attributes.
- **Quoting**: Always quote attribute values with double quotes.

## 2. Inline JavaScript (when unavoidable)
- **NEVER**: Use inline event handlers (`onclick="..."`, `onload="..."`).
- **Prefer**: Add event listeners in JS and target elements via `data-*`.
- **Keep scripts small**: If inline JS exceeds ~10-15 lines, move it to an external file.

## 3. Script Tags
- **Modules**: Prefer `<script type="module" src="..."></script>` for modern code.
- **Defer**: Use `defer` on non-module scripts to avoid blocking rendering.
- **Placement**: Prefer loading scripts at end of body or with `defer`.

## 4. Accessibility
- **Images**: Always provide `alt` text (empty `alt=""` for decorative).
- **Buttons/Links**: Use `<button>` for actions, `<a>` for navigation.
- **Labels**: Every form input must have an associated `<label>`.

