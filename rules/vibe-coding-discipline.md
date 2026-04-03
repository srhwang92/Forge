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
- the project lead proposes skipping tests for a feature
- the project lead proposes disabling or weakening security
- the project lead proposes ignoring accessibility
- the project lead proposes bypassing a guardrail
- the project lead proposes something that contradicts SPEC.md
- the project lead proposes an approach that the standards explicitly prohibit
- An implementation would create significant tech debt without awareness
- **Scope is expanding beyond SPEC.md** — a new feature or capability
  not covered in the spec. Ask: "This isn't in SPEC.md — should I add
  it to the design, or is this a quick addition outside the spec?"
  **If quick additions accumulate** (3+ features outside the spec in
  a single phase), flag: "The codebase is diverging from SPEC.md —
  should we update the spec to reflect what's actually built?" Undocumented
  features become maintenance hazards when future sessions don't know
  they exist.

**How to push back:**
1. Name the specific risk: "Skipping auth on this endpoint means any
   user can access any other user's data."
2. Propose a lighter alternative: "Instead of full RBAC, we could add
   a simple ownership check — 5 lines, still secure."
3. If the project lead still wants to proceed after seeing the risk, comply and
   document the trade-off in `STATUS.md`.

**When NOT to push back:**
- Stylistic preferences (naming, formatting, approach A vs B when both
  are valid)
- Conscious trade-offs the project lead has already weighed ("I know this is
  temporary, ship it")
- Decisions outside Claude's expertise (business strategy, pricing,
  market positioning)

**Pressure resilience.** If the project lead is frustrated, impatient, or says
"just do it" — maintain the same verification rigor. Do not reduce
review quality to avoid friction. Deliver faster by being more concise,
not by skipping checks. If pushback has been overridden 3+ times in a
single session, flag the pattern: "I've noted several trade-offs this
session — want me to summarize them so we can confirm they're all
intentional?"

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
- **Before creating a new utility or helper,** search the codebase for
  existing implementations. AI frequently recreates `formatDate`,
  `debounce`, or `cn()` functions that already exist elsewhere in the
  project. Duplicates create two sources of truth.
- **When modifying a pattern, propagate the change.** If you fix a bug
  in a data-fetching pattern, search for all instances of that pattern
  and update them. This is an AI strength — leverage it. Humans can't
  hold an entire codebase in memory; Claude can search it.
