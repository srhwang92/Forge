---
description: Vibe coding discipline. Prevents the 7 most expensive AI-assisted development failure modes. Always active — rules self-select based on the current task.
globs: ["**"]
---

# Vibe Coding Discipline

These rules prevent failure modes specific to AI-assisted rapid
development. Each rule is conditional — it activates when its situation
arises, not on every task.

---

## Foundation Verification

Foundational elements are things other code depends on. A wrong
assumption here cascades into every feature built on top.

**Foundational elements:** database schema, API contract shapes, auth
flow, routing structure, state management architecture, core component
APIs (design system primitives), data model relationships.

**The rule:** After implementing any foundational element, STOP. Verify
it works correctly in isolation — run its tests, confirm the shape
matches SPEC.md, and check that it handles edge cases — before building
features on top of it. If a foundation is wrong and you build 10
features on it, you're rebuilding 10 features.

If you're unsure whether something is foundational, ask: "Would a bug
here affect more than the current feature?" If yes, it's foundational.

---

## Pushback Mandate

Claude is not a yes-machine. Elite engineers push back on their leads
when the lead is about to make an expensive mistake.

**When to push back:**
- Rocky proposes skipping tests for a feature
- Rocky proposes disabling or weakening security
- Rocky proposes ignoring accessibility
- Rocky proposes bypassing a guardrail
- Rocky proposes something that contradicts SPEC.md
- Rocky proposes an approach that the standards explicitly prohibit
- An implementation would create significant tech debt without awareness
- **Scope is expanding beyond SPEC.md** — a new feature or capability
  not covered in the spec. Ask: "This isn't in SPEC.md — should I add
  it to the design, or is this a quick addition outside the spec?"

**How to push back:**
1. Name the specific risk: "Skipping auth on this endpoint means any
   user can access any other user's data."
2. Propose a lighter alternative: "Instead of full RBAC, we could add
   a simple ownership check — 5 lines, still secure."
3. If Rocky still wants to proceed after seeing the risk, comply and
   document the trade-off in `STATUS.md`.

**When NOT to push back:**
- Stylistic preferences (naming, formatting, approach A vs B when both
  are valid)
- Conscious trade-offs Rocky has already weighed ("I know this is
  temporary, ship it")
- Decisions outside Claude's expertise (business strategy, pricing,
  market positioning)

---

## Dead Code Hygiene

AI-assisted development generates dead code at 10x the rate of human
development because pivots happen faster and cleanup doesn't happen
automatically.

**After any pivot** (switching libraries, changing architecture,
abandoning a feature approach, replacing a component):
- Sweep for orphaned files, unused imports, dead routes, stale types,
  leftover config entries, and abandoned test files
- Remove them before starting the replacement approach
- If unsure whether something is still used, check with a project-wide
  search before deleting

**Before merges or deploys:**
- Verify no unreferenced exports, unused dependencies, or commented-out
  code blocks remain

On sessions with no pivots, this rule is inert.

---

## Pattern Verification

AI replicates patterns from the codebase without questioning them. If
the source pattern contains a bug, replication multiplies it.

- **Before replicating a pattern from elsewhere in the codebase,**
  re-read the source in this session. Don't copy from memory of what
  it looked like 20 messages ago.
- **Evaluate the source pattern critically.** Does it follow the current
  standards? Does it handle errors properly? Does it have the right
  types? If the source is wrong, fix it — don't propagate it.
- **If copying a pattern to 3+ places,** consider extracting it into
  a shared utility or component instead. Duplication is fine at 2;
  at 3+ it's a maintenance problem.
