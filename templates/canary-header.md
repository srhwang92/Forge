# Canary Test Header

Every file in `tests/canaries/` starts with this comment block. The
header IS the invariant. No separate INVARIANTS.md.

## Format

Adapt the comment syntax to the file's language:

```typescript
/**
 * Canary: [name]
 * Guards: [what must remain true — the invariant]
 * Why: [why this matters — typically references a guardrail or past incident]
 * Protected — do not delete without project lead approval.
 */
```

```python
"""
Canary: [name]
Guards: [what must remain true — the invariant]
Why: [why this matters — typically references a guardrail or past incident]
Protected — do not delete without project lead approval.
"""
```

```ruby
# Canary: [name]
# Guards: [what must remain true — the invariant]
# Why: [why this matters — typically references a guardrail or past incident]
# Protected — do not delete without project lead approval.
```

The four lines (Canary / Guards / Why / Protected) are the invariant
— the comment syntax around them adapts to the language.

## Examples

```typescript
/**
 * Canary: no-float-money
 * Guards: All money values stored and computed as integer cents.
 * Why: Float arithmetic causes rounding errors at scale (GUARDRAILS § Data Protection).
 * Protected — do not delete without project lead approval.
 */
```

```python
"""
Canary: vector-tenant-isolation
Guards: Vector queries always include tenant filter; user A cannot retrieve user B's documents.
Why: RLS does not apply to vector DBs (ai-features.md § RAG / Vector Pipeline).
Tests both: User A CAN retrieve own (control), User B CANNOT (isolation).
Protected — do not delete without project lead approval.
"""
```

```python
"""
Canary: rls-coverage
Guards: Every table containing user data has Row Level Security enabled.
Why: An RLS-disabled table negates auth scoping (GUARDRAILS § Security Non-Negotiables).
Asserts: at least N tables exist (no vacuous pass on empty schema).
Protected — do not delete without project lead approval.
"""
```
