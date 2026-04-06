# Phase [N] Snapshot — [Phase Name]

## Summary
<!-- This section is read at Tier 1 (every session start). Keep to ~10 lines. -->
- **Date:** [YYYY-MM-DD]
- **Phase goal:** [1 sentence]
- **Components built:** [count] (see REGISTRY.md)
- **Decisions made:** [count] (see DECISIONS.md)
- **Invariants added:** [count]
- **Test results:** [X passing, Y failing]
- **Canary status:** [all passing | N caught real bugs — added to
  STATUS.md | N need updating — added to PLAN.md]
  <!-- A failing canary that caught a real invariant violation is
  working as intended (add the bug to STATUS.md to fix). A failing
  canary that no longer matches the invariant is technical debt
  (add to PLAN.md to update the canary). Do NOT conflate the two. -->
- **Playwright:** [0 violations | N violations — list pages]
- **Known issues deferred:** [count — see detail below]

---

## Detail
<!-- These sections are read at Tier 2 (on demand per task). -->

### What Was Built
<!-- List components/endpoints added or significantly modified this phase.
     Reference REGISTRY.md entries. -->

### Verification Evidence
<!-- Summarize what was tested and the results. Inline summaries only —
     raw test output lives in CI logs, not checked-in files. -->
- Tests: [summary — X passed, Y failed, key coverage notes]
- Playwright: [summary — X violations, pages checked]
- Canaries: [summary]
- Manual checks: [any manual verification performed]

### Decisions Made This Phase
<!-- List DECISIONS.md entries added during this phase with domain. -->

### Known Issues Deferred
<!-- Issues acknowledged but not fixed in this phase. Each should have
     a target phase or be tracked in STATUS.md open questions. -->

### REGISTRY.md Diff
<!-- What changed in REGISTRY.md during this phase. New entries,
     status changes, removed entries. -->
