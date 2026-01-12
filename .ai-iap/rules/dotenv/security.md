# dotenv (.env) Security

> **Scope**: Prevent secret leakage and unsafe environment usage.  
> **Extends**: General security rules

## 1. No secrets in git
- **NEVER**: Commit real `.env` files containing secrets.
- **ALWAYS**: Commit `.env.example` instead (keys only, safe placeholder values).

## 2. Key hygiene
- **NEVER**: Put private keys/certificates in `.env` (use files/secret stores).
- **Prefer**: Rotate leaked credentials immediately and invalidate old tokens.

## 3. CI/CD and production
- **Prefer**: Inject environment values via CI/CD secrets and runtime secret stores.
- **ALWAYS**: Avoid printing environment values in logs.

Follow the general security rules; the rules above are additive.

