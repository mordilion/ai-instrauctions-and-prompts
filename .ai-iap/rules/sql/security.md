# SQL Security

> **Scope**: SQL security rules (injection prevention, safe migrations).  
> **Extends**: General security rules

## 1. Injection prevention
- **ALWAYS**: Use parameterized queries / prepared statements.
- **NEVER**: Build SQL by concatenating untrusted input.

## 2. Least privilege
- **Prefer**: Separate DB users/roles for app runtime vs migrations/admin tasks.
- **ALWAYS**: Restrict production credentials to the minimum required permissions.

## 3. Destructive operations
- **NEVER**: Run destructive statements (`DROP`, mass `DELETE`) without safeguards.
- **Prefer**: Wrap risky migration steps in transactions when supported.

Follow the general security rules; the rules above are additive.

