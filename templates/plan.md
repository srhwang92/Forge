# Plan — [PROJECT NAME]

> Active implementation plan with checkable tasks. Archive completed
> plans as `PLAN-v1.md` when starting new phases.
>
> Every task must have at least one acceptance criterion (AC).
> Money/auth/user-affecting tasks require ≥3 known-answer test cases.
> Non-obvious dependencies use `(depends on: Task X)`.

## Phase [N]: [Phase Name]

**Exit criteria:** [What must be true for this phase to be complete]

**Quality gate (if this phase involves AI/ML components):**
Exit criteria for AI/ML work must include measurable quality metrics,
not just "it works." Examples:
- **RAG retrieval phase:** "Retrieval precision ≥0.8 on test set of
  50 (query, expected document) pairs; tenant isolation canary passes
  with control assertion."
- **Classification model phase:** "Accuracy ≥0.9 on held-out test set;
  no single demographic group scores >10% below the mean."
- **Chat feature phase:** "Regression suite of 30 (prompt, expected
  behavior) pairs passes; injection regression suite detects 100% of
  known patterns; cost per conversation <$0.05 average."
Without measurable metrics, "Phase done" is a claim, not a fact.
Skip this section for phases with no AI/ML components.

### Tasks

- [ ] [Task description] AC: [verifiable condition]
- [ ] [Task description] (depends on: Task above) AC: [verifiable condition]
- [ ] [Task description] AC: [verifiable condition]

<!-- transient: [description of expected temporary test failures during refactors, if any] -->
