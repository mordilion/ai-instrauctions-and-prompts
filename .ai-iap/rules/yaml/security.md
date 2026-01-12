# YAML Security

> **Scope**: YAML security guidance for CI/CD and infrastructure configs.  
> **Extends**: General security rules

## 1. Secrets
- **NEVER**: Put secrets, tokens, or private keys in YAML.
- **Prefer**: Secret managers (GitHub Actions secrets, Vault, cloud secret stores).

## 2. CI/CD supply chain (GitHub Actions style guidance)
- **Prefer**: Pin third-party actions by commit SHA (not floating tags).
- **ALWAYS**: Review permissions; prefer least-privilege and explicit `permissions:`.

## 3. YAML-as-data (deserialization risks)
- **Treat**: YAML as data, not code.
- **NEVER**: Enable unsafe YAML deserialization modes in application runtimes.

Follow the general security rules; the rules above are additive.

