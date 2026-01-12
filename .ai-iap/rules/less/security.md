# Less Security

> **Scope**: Less security and safety.
> **Extends**: General security rules

## 1. Untrusted Inputs
- **NEVER**: Compile Less from untrusted input.
- **ALWAYS**: If user-controlled values affect styling, restrict to allowlists (themes, class names, known tokens).

## 2. Build pipeline safety
- **Prefer**: Pinned dependency versions for Less tooling.
- **Avoid**: Unreviewed third-party mixin libraries.

Follow the general security rules; the rules above are additive.

