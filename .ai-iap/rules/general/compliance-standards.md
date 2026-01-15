# Compliance Standards (General)

> **Scope**: Apply when working on regulated, privacy-sensitive, or audited systems. If your project has no compliance requirements, treat this as “nice-to-have” and defer to `security.md` and project-specific policies.

## Data Classification & Handling

> **ALWAYS**:
> - Treat **PII/PHI/PCI/secrets** as sensitive by default
> - Minimize collection: only store what is required for the feature
> - Document where sensitive data is stored, processed, and transmitted
> - Enforce least-privilege access to data (RBAC/ABAC)

> **NEVER**:
> - Log secrets, tokens, passwords, or raw sensitive payloads
> - Copy production sensitive data into dev environments without explicit approval and sanitization

---

## Auditing & Evidence

> **ALWAYS**:
> - Log security-relevant events (auth failures, privilege changes, admin actions, data exports)
> - Include request correlation identifiers where applicable (trace IDs)
> - Ensure logs are tamper-resistant and retained according to policy
> - Keep an auditable change history for configuration affecting security/compliance

---

## Retention & Deletion

> **ALWAYS**:
> - Define retention periods for sensitive data and logs
> - Implement deletion workflows when required (user deletion requests, legal requirements)
> - Prefer soft-delete only if policy allows; ensure eventual hard-delete when required

---

## Access Control & Separation of Duties

> **ALWAYS**:
> - Separate duties for high-risk actions (deployments, production data access, privileged config changes)
> - Use just-in-time access / break-glass processes for production access when possible
> - Restrict admin interfaces and require stronger auth for privileged operations

---

## Secure SDLC Expectations

> **ALWAYS**:
> - Run dependency and security scanning as part of CI (SCA/SAST, container scanning if applicable)
> - Treat high/critical findings as release blockers unless explicitly accepted and documented
> - Ensure code reviews are required for changes in security/compliance-critical areas

---

## AI Self-Check (Compliance)

- [ ] Did I avoid logging sensitive data and redact where necessary?
- [ ] Are audit logs present for privileged and data-export operations?
- [ ] Is retention/deletion behavior defined and implemented where required?
- [ ] Are permissions least-privilege and consistent across the system?
- [ ] Are compliance-relevant CI checks in place (or documented as TODO with owner)?

