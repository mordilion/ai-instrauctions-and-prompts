# Dockerfile Architecture

> **Scope**: Docker build structure and reproducibility patterns.  
> **Extends**: General architecture rules

## 1. Multi-stage builds
- **Prefer**: Multi-stage builds to keep runtime images minimal.
- **Prefer**: Separate build and runtime dependencies.

## 2. Reproducible builds
- **Prefer**: Pin base images to a specific tag/digest.
- **ALWAYS**: Keep build steps deterministic (avoid “latest” downloads).

## 3. Runtime design
- **Prefer**: Run as a non-root user.
- **Prefer**: Explicit `WORKDIR`, explicit `ENTRYPOINT`/`CMD`.

