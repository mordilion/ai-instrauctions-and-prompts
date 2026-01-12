# PostCSS Security

> **Scope**: PostCSS security and safety.
> **Extends**: General security rules

## 1. Supply chain
- **Prefer**: Pinned versions for PostCSS plugins.
- **Avoid**: Untrusted plugins (they execute code at build time).

## 2. Untrusted Inputs
- **NEVER**: Feed untrusted user input into a build-time CSS pipeline.
- **ALWAYS**: Treat build pipelines as privileged (CI secrets, filesystem access).

Follow the general security rules; the rules above are additive.

