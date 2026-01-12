# Sass/SCSS Security

> **Scope**: Sass/SCSS security and safety.
> **Extends**: General security rules

## 1. Untrusted Inputs
- **NEVER**: Compile Sass from untrusted input.
- **ALWAYS**: If user-controlled values affect styling, restrict to allowlists (e.g., themes, known class names).

## 2. Build pipeline safety
- **Prefer**: Pinned dependency versions for Sass tooling.
- **Avoid**: Unreviewed third-party mixin libraries and copy-pasted “utility packs”.

Follow the general security rules; the rules above are additive.

