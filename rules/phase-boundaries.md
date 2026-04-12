---
description: Full phase boundary procedure — state updates, forward-looking discussion, HANDOFF.md at ship milestones. Loaded when Claude hits a phase boundary; CLAUDE.md has detection rule + short summary.
globs: ["**/.claude/snapshots/**", "**/HANDOFF.md"]
---

# Phase Boundaries — Full Procedure

> CLAUDE.md has the detection rule and a short summary. This file has
> the full procedure. Load when a phase's exit criteria are met.

## State updates (run before discussion)

A phase ends when the last task in the current PLAN.md phase completes
and its exit criteria are met. Before starting the next phase, run
these five steps in order:

1. **Propose a phase snapshot** using `~/.claude/templates/phase-
   snapshot.md`. Write to `.claude/snapshots/phase-N.md`. Summary
   (~10 lines) is read at Tier 1 every session; detail is Tier 2.
2. **Full REGISTRY.md reconciliation.** Compare component file count
   against entry count, resolve every entry's path, mark deprecations.
3. **Prune DECISIONS.md.** Mark superseded entries with
   `~~[SUPERSEDED]~~` + blockquote warning. Archive if the section
   exceeds 50 entries.
4. **Resolve STATUS.md open questions.** Each should be answered
   (move to DECISIONS.md), deferred to a later phase (tag with phase
   number), or closed as no longer relevant.
5. **Update PLAN.md.** Mark current phase complete. Define next
   phase's exit criteria if not already present.

## Phase boundary discussion

The snapshot answers "what happened"; the discussion answers "what
should happen next." After state updates, before moving on, present
to the project lead:

- **Recommended first task** from the next phase's PLAN.md entries,
  with a 1-2 sentence rationale (dependencies, risk, complexity).
  If the next phase has a natural ordering, propose it.
- **Open questions** that emerged during the just-completed phase
  but weren't surfaced at the time — architectural ambiguities,
  implicit decisions that should be documented, places where spec
  and implementation diverged.
- **Observations worth flagging** — technical debt accumulated,
  patterns that might warrant extraction, places where a future
  refactor would pay off. Don't manufacture observations; if there
  are none, skip this section.

Keep each item brief (1-3 sentences). The goal is a working
conversation, not a report. The lead's responses flow into PLAN.md
(next steps), DECISIONS.md (resolved questions), or STATUS.md
(deferred observations).

## Non-negotiable

Don't start next-phase work before the snapshot is written and
committed. Phase boundaries are the anchor point for session
reconstruction — skipping a snapshot breaks Tier 1 for every
subsequent session. After the boundary commit, prompt the project
lead about compaction — see `rules/compaction.md`.

## Ship/stable milestone

For ship/stable milestones (project going to production), also
generate `HANDOFF.md` using `~/.claude/templates/handoff.md`. This
is the human-readable overview covering architecture, reading order,
local dev setup, product context, maintenance info (dependencies,
secret rotation, backups), and the "what NOT to change without
reading first" index.
