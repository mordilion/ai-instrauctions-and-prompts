# HTML Security

> **Scope**: HTML-specific security (especially when embedding JavaScript).  
> **Extends**: General security rules
> **Applies to**: *.html, *.htm, *.vue, *.svelte

## 1. XSS (Primary Risk)
- **NEVER**: Inject untrusted strings into HTML without escaping/sanitization.
- **NEVER**: Build HTML via string concatenation in inline scripts.
- **Prefer**: DOM APIs (`textContent`, `setAttribute`) over `innerHTML`.

## 2. Inline Script Safety
- **Prefer**: No inline scripts at all.
- **If inline scripts exist**:
  - Keep them minimal
  - Do not embed secrets/tokens
  - Avoid dynamic code execution (`eval`, `new Function`)

## 3. CSP Guidance
- **ALWAYS**: Use a strict Content Security Policy (CSP) when possible.
- **Prefer**: external scripts + nonces/hashes over `unsafe-inline`.

## 4. Links & Targets
- **ALWAYS**: For `target="_blank"`, include `rel="noopener noreferrer"`.

Follow the general security rules; the rules above are additive.

