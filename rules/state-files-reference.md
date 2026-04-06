---
description: Detailed reference for per-project state files (REGISTRY, DECISIONS, INVARIANTS, etc.). Loaded on-demand when creating or modifying these files. CLAUDE.md has the short version.
globs: ["**/REGISTRY.md", "**/DECISIONS.md", "**/INVARIANTS.md", "**/STATUS.md", "**/PLAN.md", "**/HANDOFF.md", "**/.claude/**"]
---

# Project State Files — Detailed Reference

> CLAUDE.md has a one-line summary of each file. This file has the
> detailed description. Loaded on-demand when creating or modifying
> state files, not at every session start.

## `STATUS.md`

Create from `~/.claude/templates/status.md`. Current task/phase,
done/in-progress/blocked, last modified files, open questions.
Archive to `.claude/logs/` when >50 lines.

## `PLAN.md`

Create from `~/.claude/templates/plan.md`. Active plan with checkable
tasks. Archive completed plans as `PLAN-v1.md` when starting new phases.
**Every task must have at least one acceptance criterion** — a concrete
verifiable condition. Format: `- [ ] Task description (depends on:
Task X if non-obvious) AC: [verifiable condition]`. Examples:
"AC: API returns 200 with valid token, 401 without", "AC: Modal opens
on button click." Money/auth/user-affecting tasks additionally require
≥3 known-answer test cases (see Layer 1). Only annotate non-obvious
dependencies — don't mark every task as depending on the previous one
in a linear plan.

## `REGISTRY.md`

One line per component/endpoint/utility: name, path, purpose, lifecycle
status, tests, rollback commit. Updated after every task. Primary
defense against post-compaction hallucination. Create from
`~/.claude/templates/registry.md`. AI projects additionally use the
AI Prompts, AI Models & Embeddings, and Agent Tools tables in that
template.

## `DECISIONS.md`

Domain-organized (Auth, Payments, Data Model, etc.). Each: chose,
rejected, assumptions (falsifiable), verify (runnable command).
Superseded entries marked `~~[SUPERSEDED]~~` with a blockquote warning
and skipped during Tier 2 reads. See `~/.claude/templates/decisions.md`
for the format that prevents human skim-reading confusion.

## `INVARIANTS.md`

Short testable statements. Every invariant references a canary test.
An invariant without a canary is an unenforced claim.

## `tests/canaries/`

One test per invariant. Protected — never delete without approval.
Each canary file must include a header comment: which INVARIANTS.md
entry it enforces and "Protected — do not delete without project lead
approval." This is for human developers who may not know the Forge
framework. Review canary relevance during health checks — archive
canaries for invariants no longer at risk (e.g., schema coverage
canary after CI includes schema-level checks). **Canaries protect
invariants universally, not specific code paths** — when new surfaces
are added, existing canaries must be extended to cover them.

## `.claude/snapshots/phase-N.md`

Phase boundary records. Summary (~10 lines) read at Tier 1. Detail
read at Tier 2 on demand. Create from `~/.claude/templates/phase-snapshot.md`.

## `.claude/logs/`

Append to `YYYY-MM-DD.md` at session end or before `/compact`.
Each entry starts with: `## YYYY-MM-DD — Session [N]`. Gitignored,
append-only. Separate facts (files changed, test results, commands)
from decisions (chose/rejected/assumptions/verify). No self-
congratulatory summaries — actionable verification only.

**If a secret lands in an append-only log** (Claude pasted an error
object containing a connection string, API key, or similar), this
is a security incident. Procedure: (1) rotate the compromised secret
immediately, (2) archive the affected log file to secure storage
outside the repo, (3) start a fresh log file, (4) document the
incident in STATUS.md with the rotation confirmation, (5) add a
post-mortem entry to DECISIONS.md. Never silently edit an append-only
log to remove a secret.

## `.claude/in-progress.md`

The single survival mechanism for mid-task compaction (both
intentional `/compact` and unexpected token-limit hits). Required
for any refactor touching 5+ files. Update continuously — after
each sub-step — with:
- **What was just done** — the most recent change and its outcome
- **What's next** — the immediate next step
- **Discoveries** — anything learned mid-work that affects the
  approach (e.g., "feature X uses a subtly different auth variant
  that also touches audit logs")
- **Expected transient test failures** — which tests are deliberately
  broken mid-refactor and will re-pass at completion
- **Reasoning in progress** — design options being considered,
  current leaning, why alternatives were partially rejected.
  Decisions NOT YET MADE go here; completed decisions go in
  DECISIONS.md.

Before an intentional `/compact`, confirm this file is current.
If compaction fires unexpectedly, this is the only thing that
survives working memory. Delete when the task completes.

## `HANDOFF.md`

Generated at ship/stable milestone from `~/.claude/templates/handoff.md`.
Human-readable project overview for developers taking over the codebase
AND for whoever maintains the project in production (often the same
person). Contains:
- **Architecture overview** with a simple text diagram
- **Reading order** — which files to read first and what they teach
- **Local dev setup** — prerequisites, services, env vars, first-run
  commands
- **Product context** — decisions not captured in SPEC.md or DECISIONS.md
- **Forge framework notes** — for developers new to Forge
- **"What NOT to Change Without Reading First"** index for load-bearing
  tunables (prompts, thresholds, chunking configs, pinned model versions)
- **Maintenance section** — dependencies, secret rotation schedule
  (API keys 90d, DB creds 180d, signing keys 1yr), backup procedures,
  fragile areas, health check cadence
- **Known rough edges** — 3-5 areas that need extra care

Unlike other Forge files, HANDOFF.md is written for humans, not for
Claude. Update on major architectural changes.

## Memory File Scaling

When files grow large:
- **REGISTRY.md 100+ entries:** organize by module sections
- **REGISTRY.md 200+ entries:** split into `REGISTRY-[module].md`
  files + index
- **DECISIONS.md 50+ entries:** archive `~~[SUPERSEDED]~~` entries
  to `.claude/archives/`
- **Snapshots:** keep last 3, archive older
- **STATUS.md >50 lines:** archive and reset
- **Total Tier 1 budget:** all Tier 1 files ≤~8,000 tokens (~2,400
  lines) for young projects. **Mature projects** (200+ REGISTRY
  entries, 50+ decisions) realistically use 15-25K tokens on
  reconstruction. If exceeded, force consolidation before continuing.
- **Monorepos:** maintain per-app state files (`REGISTRY-app-name.md`,
  `DECISIONS-app-name.md`). Shared packages get entries in a root-level
  `REGISTRY-shared.md`.
