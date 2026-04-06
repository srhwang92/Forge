# Global Defaults

<!-- Customize this section for your role and setup -->
Multi-client development — each project has its own CLAUDE.md that
extends these defaults.

**Platform:** Configure your OS and shell in the section below.
<!-- Example for Windows: PowerShell-compatible commands only. -->
<!-- Example for macOS/Linux: Standard bash/zsh commands. -->

---

## Zero Assumptions

**Never assume. Always ask.** Before planning or implementing anything
non-trivial, run the project interview in
`~/.claude/templates/interview.md` verbatim. Seven grouped questions
covering Purpose/Users, Stack, Data Sensitivity, **AI Features
(mandatory)**, Scope/Constraints, Existing Context, Design/Brand.
Missing any question is a process failure, not a judgment call.
Answers flow into `project-claude.md`, SPEC.md, and DECISIONS.md.

If the AI Features answer is yes, maybe, or "not yet but eventually":
load `~/.claude/templates/guardrails/ai-features.md` immediately. AI
is never a late-project bolt-on — adding it later re-triggers the
question and re-loads the template.

After the interview, proceed to **Project Setup** before writing code.

If something is ambiguous — stop and ask. Do not fill in blanks with
reasonable defaults. The only exceptions are mechanical tasks with no
decision points (formatting, linting, running tests as instructed).

**Zero Assumptions vs. Pragmatism — precedence.** Zero Assumptions
applies to project-level unknowns (stack, users, compliance, data
sensitivity, architecture). Pragmatism (see `code-voice.md`) applies
to implementation-level choices (utility library, variable names,
helper inlining). Heuristic: if getting it wrong needs a rewrite,
ask; if a 10-minute refactor fixes it, decide.

---

## Workflow

Superpowers drives all builds:
Brainstorm → Spec → Plan → TDD → Subagent Dev → Review → Finalize

Non-build tasks: Explore → Plan → Code → Commit.
Never write code before exploring and getting plan approval.

## Verification Protocol

Check the project CLAUDE.md for the **ceremony level**. Default: `standard`.
- **standard**: Layer 1 + Layer 2 on non-trivial tasks, Playwright on
  UI changes, all gates (default for production work)
- **light**: Layer 1 only, security/destructive gates only (prototypes,
  internal tools without sensitive data, static marketing sites)
- **minimal**: tests only, no gates except destructive/security (scripts)

Security guardrails (GUARDRAILS.md) apply at ALL ceremony levels — see
the "Always-Active Sections" list in the GUARDRAILS.md preamble.

**Ceremony minimum floors:**

| Project type | Minimum floor |
|---|---|
| Fintech / healthcare (money or health data) | `standard` (cannot downgrade) |
| External users + billing/auth/UGC/PII/AI features | `standard` |
| Confidential/Restricted data | `standard` |
| External users, marketing/docs only | `light` |
| Internal tools, no sensitive data | none |

**Override rule:** downgrading a floor requires Tier A approval +
written justification in DECISIONS.md. Regulated floors (fintech,
healthcare) can be raised but never downgraded.

**Reassess ceremony level when:** project adds financial transactions,
health data, or auth for the first time; external users are added to
a previously internal tool; a prototype is promoted to production;
project adds any AI feature (chat, RAG, embeddings, generation, agents,
tool use, MCP integration) for the first time, or connects an existing
AI feature to a new data source. AI additions trigger a full re-read of
`ai-features.md` AND every industry template already in play — parent
regulations inherit to AI data flows.

Every completed task goes through verification layers.

**Layer 1 — Self-check (every task).** Mechanical facts only:
- **Impact analysis (before implementation):** read REGISTRY.md
  dependents + grep imports. Note in STATUS.md. If the target
  component is NOT in REGISTRY.md, search the filesystem — add it
  before modifying. For shared components with 20+ dependents,
  read the REGISTRY dependents list in full + 3-5 representative
  dependent files (different usage patterns). Full dependent review
  for breaking API changes = a separate multi-session task.
- Tests pass? Run them.
- Output matches PLAN.md acceptance criteria point by point?
  **Money/auth/user-affecting tasks:** PLAN.md must have ≥3 known-
  answer test cases. Add before implementing, not after.
- SPEC.md alignment? If the task reveals the spec needs updating —
  new requirements discovered, edge case found, architecture adjustment
  — this is expected. Present the change with rationale, get approval
  (Tier A), then continue. Spec evolution is learning, not failure.
- Multi-component task: end-to-end flow works, not just units?
- Lint/type errors? Run check command.
- Build succeeds? Run build command.
- **UI changes + Playwright available:** axe-core audit + screenshots
  at mobile/desktop. Axe-core catches ~30-40% of a11y issues — not
  WCAG proof. **If the project has dark mode:** run axe-core in BOTH
  color schemes — passing in one mode proves nothing about the other.
  Inherited violations: document in STATUS.md, don't block.
- Missing files that need changing?
- Created/deleted/renamed component? Update REGISTRY.md. Modified
  `stable` component? Reset to `verified`.
If any check fails, fix before proceeding. **Exception — cross-cutting
refactors:** note expected transient failures in PLAN.md with
`transient:` tag (e.g., "transient: old import paths break until
migration complete"). Unlisted failures are real bugs — fix immediately.
All tests must pass at task completion, not just at each intermediate
step.

**Layer 2 — Subagent review (non-trivial tasks).** Dispatch with ONLY:
task description, acceptance criteria, changed files. No conversation
history. Frame: "Find what's wrong." Subagent checks:
- Semantic correctness (spec gaming), hidden coupling, test integrity
  (if PLAN.md has known-answer pairs, verify tests match those outputs),
  DESIGN.md compliance (UI tasks). Uses Sonnet.
  Workarounds approved by the project lead (marked `// APPROVED:
  [reason]` in code) should not be flagged as issues.

**Layer 2 is a structural review, not an independent audit.** The
subagent shares the main agent's model family and blind spots.
Fresh context + task-only framing catches different failure modes
than Layer 1 (spec gaming, missed edges, hidden coupling) but adds
no independent perspective. For true independent review, use
cross-model adversarial review (see README, optional) or external
human review.

**Trivial** (skip Layer 2): renaming, formatting, linting, config
one-liners, comment edits, import reordering, non-breaking version bumps.
**Non-trivial** (require Layer 2): anything changing behavior — features,
bug fixes, logic refactors, API/DB/auth changes, interactive UI, tests.
When in doubt: non-trivial.

## Subagent Delegation

Prefer subagents for: investigation, review (Layer 2), parallel
independent tasks, research (Context7, docs). Keep main context for
decision-making and conversation-dependent tasks.

**Parallel subagents:** list ALL files each might touch including
barrel files, shared types, config. Overlap → run sequentially.
Barrel file updates are main-agent responsibility after completion.
**Default to parallel dispatch for independent investigations** —
three reads in parallel is nearly always faster than three reads in
sequence, and token cost is the same. Sequential is only correct
when one investigation's output feeds the next.

**Implementation subagent context:** include relevant SPEC.md
decisions + project conventions snippet (API format, error pattern,
naming, auth approach).

**Multi-perspective analysis (contested decisions).** For genuinely
hard calls — architecture choices with real trade-offs, security
decisions with no obviously right answer, refactor strategies that
could go several ways — dispatch 2-3 subagents with deliberately
different framings rather than one subagent asked to "consider all
angles." Example framings: *find what's wrong* / *find what's
missing* / *find what's brittle*, or *defender* / *attacker* /
*maintainer*. Synthesize in the main agent. One subagent with a
broad prompt produces flatter analysis than three subagents with
sharp framings.

---

## Production Engineering Standards

LLMs tend toward overcomplexity, hardcoded values, and "almost right"
code. Detailed standards load automatically from `rules/*.md`:
`code-voice.md`, `vibe-coding-discipline.md`, `frontend-standards.md`,
`backend-standards.md`, `git-conventions.md`, `context7-usage.md`,
`state-files-reference.md`.

**Core principles** (details in rules files):
- **Simplicity:** simplest solution wins. YAGNI. No wrappers without
  behavior. 30 lines > 100 lines.
- **Never hardcode:** every changeable/reusable value is a token,
  variable, constant, or config entry.
- **Incremental:** smallest working piece → verify → next piece.
  Edit surgically; never regenerate entire files for small fixes.
- **Prohibited:** 500+ line files without asking, abstractions for
  future flexibility, duplicated logic, `console.log` in production,
  generic errors, deprecated APIs, hallucinated packages.
- **Template conventions:** follow the starter's patterns for
  existing code; apply Forge standards to new code it doesn't cover.
  Security (GUARDRAILS.md) always overrides template conventions.
  On conflict (e.g., Forge wants layered architecture but template is
  flat), document in DECISIONS.md with "Template vs Forge" tag and
  default to following the template for existing code.

---

## Design System Enforcement

**Before UI work, check for `DESIGN.md`.** If it exists, it is the
source of truth — every visual value must conform. Never use
`[VERIFY]`-tagged values in implementation without project lead
approval. Full extraction, token sourcing, and skill priority rules
live in `~/.claude/rules/design-workflow.md` (auto-loads on UI files).

---

## Guardrails

Global: @~/.claude/GUARDRAILS.md — always active, every project.
Project: `GUARDRAILS.md` in project root extends global.
Never bypass without explicit approval.

---

## Project Setup (New Projects)

After the interview, follow `~/.claude/templates/project-setup.md`
for the kickoff procedure (inherited codebase scan, template handling,
10-step setup, SPEC.md subagent validation).

---

## Project State Files

Per-project files live in the project root or `.claude/`. Details for
each file — templates, format rules, scaling thresholds — are in
`~/.claude/rules/state-files-reference.md`. Load that file on-demand
when creating or modifying state files. The summary below is the
session-level working knowledge.

| File | Purpose | Created From |
|---|---|---|
| `STATUS.md` | Current task/phase/blocked/questions | `templates/status.md` |
| `PLAN.md` | Active tasks with ACs and dependencies | `templates/plan.md` |
| `REGISTRY.md` | Component inventory + lifecycle state | `templates/registry.md` |
| `DECISIONS.md` | Domain-organized chose/rejected/assumptions | `templates/decisions.md` |
| `INVARIANTS.md` | Testable truth claims, each → canary | `templates/invariants.md` |
| `tests/canaries/` | One test per invariant, protected | — |
| `SPEC.md` | Architecture, data model, flows, AI config | `templates/spec.md` |
| `DESIGN.md` | Design tokens, components, a11y targets | `templates/design.md` |
| `.claude/snapshots/phase-N.md` | Phase boundary records | `templates/phase-snapshot.md` |
| `.claude/logs/YYYY-MM-DD.md` | Append-only session logs | — |
| `.claude/in-progress.md` | Live scratchpad for multi-file refactors, survives compaction | — |
| `HANDOFF.md` | Human-readable overview + maintenance info (ship milestone) | `templates/handoff.md` |

**Critical rules** (full reasoning in `state-files-reference.md`):
- REGISTRY.md is updated after every task. It is the primary defense
  against post-compaction hallucination.
- INVARIANTS.md entries without a canary test are unenforced claims.
- DECISIONS.md superseded entries use `~~[SUPERSEDED]~~` + blockquote
  warning to prevent human skim-reading confusion.
- Logs are append-only. If a secret lands in a log, rotate and
  archive — never silently edit.
- Canaries protect invariants universally. When new surfaces ship,
  existing canaries must extend to cover them in the same task.

### Session Recovery — Tiered Reconstruction

After compaction or context loss, complete Tier 1 before writing code.

**Tier 1 — Always read at session start:**
1. Project CLAUDE.md — stack, commands, conventions
2. Project GUARDRAILS.md — safety rules
3. SPEC.md — architecture constraints
4. REGISTRY.md — what exists, where, lifecycle status
5. INVARIANTS.md — what must always be true
6. DECISIONS.md TOC — domain list only
7. Latest phase snapshot summary
8. STATUS.md — progress + open questions
9. PLAN.md — active tasks
10. Latest `.claude/logs/` entry
11. REGISTRY reconciliation — spot-check 3-5 paths resolve to real
    files. Full reconciliation at phase boundaries.
12. Memory file integrity — after updating REGISTRY/DECISIONS/INVARIANTS,
    commit: `git add && git commit -m "chore: update memory files"`.
    Before compaction: `git diff` on memory files for unintended changes.
13. If `.claude/in-progress.md` exists — read carefully. Mid-refactor
    scratchpad state. If tests are failing, check the scratchpad's
    "expected transient failures" before diagnosing. Do NOT panic-revert
    without understanding what was in flight.

**Tier 2 — On demand:** relevant DECISIONS.md domain, REGISTRY.md
component detail, snapshot detail.

**Tier 3 — Source code:** actual files targeted by REGISTRY.md paths.

---

## Compact Instructions

**When to compact:** proactively between task boundaries when context
is ~70% full. Never start a complex multi-file task with <30% context
remaining — compact first. If unsure, err toward compacting.

**Pre-compaction validation:**
1. Spot-check 3 REGISTRY.md paths resolve to real files
2. **Completeness check:** compare component file count
   (`find src -name '*.tsx' -o -name '*.ts' | grep -v test | wc -l`)
   against REGISTRY.md entry count. If files exceed entries by >10%,
   run full reconciliation before compacting.
3. INVARIANTS.md canary references exist as test files
4. STATUS.md open questions are self-contained
5. `git diff` on memory files — restore from git if truncated

**Standard compaction:** update STATUS.md, append to `.claude/logs/`.
Preserve: modified files, plan progress, test commands, design refs,
active SPEC.md decisions.

**Mid-task compaction (any kind).** `.claude/in-progress.md` is the
single survival mechanism for both intentional and unexpected
compaction. For any refactor touching 5+ files, maintain this file
continuously — update it after each sub-step with: what was just
done, what's next, discoveries made during the work, expected
transient test failures, design options being considered, and
decisions not yet made. Before an intentional `/compact`, confirm
it's current. If compaction fires unexpectedly, this file is all
that survives. See `rules/state-files-reference.md` for format.
Delete after the task completes.

---

## ⚠️ Things Claude Gets Wrong

> Maintained per-project in the project `CLAUDE.md`. See template.
> If the same mistake appears across 3+ projects, promote it to the
> relevant global rules file. Review and update this section during
> health checks — if the same mistake appeared in the last 3 sessions,
> add it here.