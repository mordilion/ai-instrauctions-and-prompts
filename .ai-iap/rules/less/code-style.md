# Less Code Style

> **Scope**: Less formatting and maintainability rules.

## 1. Formatting
- **Indentation**: 2 spaces.
- **One selector per line**: Prefer multi-line blocks.
- **Ordering**: Group properties consistently (layout → box model → typography → visuals → animation).

## 2. Nesting
- **Avoid**: Deep nesting; keep selectors shallow.
- **Prefer**: Explicit class names over long descendant chains.
- **NEVER**: Use `!important`.

## 3. Variables and mixins
- **Prefer**: Variables for repeated values.
- **Prefer**: Parametric mixins for repeated patterns.

