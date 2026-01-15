# HTML Architecture

> **Scope**: Apply these rules ONLY when working with `.html` / `.htm` files and HTML/template sections in component files (e.g. `.vue`, `.svelte`). These extend the general architecture guidelines.

## 1. Core Principles
- **Separation of concerns**: HTML = structure; CSS = presentation; JavaScript = behavior.
- **Prefer external JS/CSS**: Keep HTML minimal; only inline when justified (critical CSS, tiny bootstraps).
- **Accessibility-first**: Semantic markup (landmarks, labels, headings) is part of architecture.

## 2. Script Strategy (JS in HTML)
- **Prefer**: `<script src="...">` over inline `<script>`.
- **When inline is needed**: Keep it minimal and delegate to external modules/functions.
- **Avoid global state**: Use modules or IIFEs; expose minimal surface.

## 3. DOM Contracts
- **Data attributes**: Use `data-*` as the contract between HTML and JS (e.g., `data-testid`, `data-action`).
- **IDs**: Use unique IDs sparingly; prefer classes/data attributes for JS hooks.
- **Progressive enhancement**: Pages should remain usable without JS where reasonable.

## 4. Structure Conventions
- **Consistent layout**: `header`, `nav`, `main`, `footer`.
- **Forms**: Always pair inputs with labels; prefer native validation + server validation.

