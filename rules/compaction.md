---
description: Full compaction procedure — phase boundary decision prompt with factors, pre-compaction validation, standard and mid-task compaction handling. Loaded when Claude compacts or hits a phase boundary; CLAUDE.md has a short summary.
globs: ["**/.claude/in-progress.md"]
---

# Compaction — Full Procedure

> CLAUDE.md has detection rules and a short summary. This file has
> the full procedure. Load before compacting or at phase boundaries
> when the compaction decision is being made.

## When to compact

Proactively between task boundaries when context is ~70% full. Never
start a complex multi-file task with <30% context remaining — compact
first.

## Phase boundary compaction decision (Tier A gate)

After committing a phase boundary snapshot, before starting the next
phase, present the compaction decision to the project lead:

> *"Phase N committed. Context at [X%]. Next phase has [Y] tasks
> ([rough complexity]). Compact now?*
>
> *Recommendation: [yes/no] because [1-2 specific reasons from the
> factors below]."*

Factors to cite in the recommendation:

- **Context utilization:** <50% = compaction is usually premature;
  50-65% = judgment call; >65% = compaction is usually worthwhile.
- **Next phase weight:** heavy phase (10+ tasks, architectural work,
  multiple files) benefits from fresh context; light phase (2-3 small
  tasks) can share current context.
- **Active conversational context:** flag if the current session
  contains in-flight design discussions, rejected alternatives, or
  reasoning not yet written to DECISIONS.md. These will be lost on
  compaction — propose writing them to DECISIONS.md first.
- **Session ending vs continuing:** don't assume. Ask.

If the project lead defers the decision, state the default and
proceed: at <65% context, default is continue without compacting;
at ≥65%, default is compact. Don't compact without an answer or a
defaulted value — compaction is a state change with ongoing
consequences (reconstruction token cost, loss of conversational
context). Tier A gate applies.

## Pre-compaction validation

Run before every compaction:

1. Spot-check 3 REGISTRY.md paths resolve to real files
2. Component file count vs REGISTRY.md entry count: if files exceed
   entries by >10%, run full reconciliation before compacting
3. INVARIANTS.md canary references exist as test files
4. STATUS.md open questions are self-contained
5. `git diff` on memory files — restore from git if truncated

## Standard compaction

Update STATUS.md, append to `.claude/logs/`. Tier 1 reconstruction
on the next turn will rehydrate from the memory files.

## Mid-task compaction

`.claude/in-progress.md` is the survival mechanism for both
intentional and unexpected compaction. For any refactor touching
5+ files, maintain this file continuously (see
`rules/state-files-reference.md` for format). Confirm it's current
before intentional `/compact`. Delete after the task completes.
