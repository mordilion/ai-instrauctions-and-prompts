# CSS Security

> **Scope**: CSS-specific security and safety.  
> **Extends**: General security rules

## 1. Untrusted Inputs
- **NEVER**: Insert untrusted strings into `<style>` or `style=""` attributes without strict validation.
- **ALWAYS**: If user-controlled values affect styling, restrict to allowlists (e.g., allowed themes, known class names).

## 2. CSS Injection Considerations
- **Avoid**: Building CSS strings dynamically with untrusted input.
- **Prefer**: Toggle known classes (`.is-open`) and use CSS variables with validated values.

## 3. Third-Party CSS
- **Minimize**: Third-party style dependencies where possible.
- **Pin versions**: Avoid unpinned CDN includes; prefer local builds.

Follow the general security rules; the rules above are additive.

