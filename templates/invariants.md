# Invariants — [PROJECT NAME]

> Conditions that must hold at ALL TIMES, regardless of what feature is
> being built. Written in testable language, not prose.
>
> **Every invariant must reference a canary test.** An invariant without
> a canary is an unenforced claim. During health checks, verify all
> canary references resolve — orphaned invariants are flagged for removal.
>
> Canary tests live in `tests/canaries/` and run with the normal test
> suite. They test ASSUMPTIONS and CONSTRAINTS, not features.

---

<!-- Add invariants as they're established. Remove when features are removed. -->

## Data Integrity
<!-- - All money values stored as integer cents (verify: canaries/no-float-money.test) -->
<!-- - Soft delete on all business record tables (verify: canaries/soft-delete.test) -->

## Auth & Security
<!-- - Every table with user data has RLS enabled (verify: canaries/rls-coverage.test) -->
<!-- - No endpoint returns another user's data without admin role (verify: canaries/idor-check.test) -->

## Performance
<!-- - No API endpoint >200ms at p95 on seed data (verify: canaries/api-latency.test) -->

## Business Rules
<!-- - [Project-specific business invariants with canary references] -->
