# YAML Architecture

> **Scope**: YAML configuration design for CI/CD, k8s, docker-compose, and tooling.  
> **Extends**: General architecture rules

## 1. Keep config modular (but navigable)
- **Prefer**: Small, focused YAML files over one mega-file.
- **ALWAYS**: Use clear file naming that matches the system (`ci.yml`, `deployment.yml`, `compose.yml`).

## 2. Environment separation
- **Prefer**: Overlays/variants (`dev`, `staging`, `prod`) over big conditional blocks.
- **ALWAYS**: Keep secrets out of YAML; reference secret stores instead.

## 3. Reduce duplication safely
- **Prefer**: YAML anchors/aliases for repeated blocks.
- **NEVER**: Overuse anchors to the point that reading the file requires “executing” mental indirection.

