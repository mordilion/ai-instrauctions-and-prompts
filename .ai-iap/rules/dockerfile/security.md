# Dockerfile Security

> **Scope**: Container security practices applied at build time and runtime.  
> **Extends**: General security rules

## 1. Base image trust
- **Prefer**: Official or organization-approved base images.
- **Prefer**: Pin by digest for high assurance.

## 2. Least privilege
- **ALWAYS**: Run as non-root where possible.
- **Prefer**: Drop capabilities and use read-only FS at runtime (or document why not).

## 3. Dependencies and downloads
- **NEVER**: Download and execute remote scripts without verification.
- **Prefer**: Verify checksums/signatures for downloaded artifacts.

## 4. Secrets
- **NEVER**: Bake secrets into images (`ENV`, `ARG`, files in layers).
- **Prefer**: Runtime secret injection (orchestrator/secret store).

Follow the general security rules; the rules above are additive.

