# Global Defaults

<!-- Customize the two lines below for your role, setup, and platform.
     Windows: specify PowerShell. macOS/Linux: specify bash/zsh. -->

Multi-client development — each project extends these defaults via
its own CLAUDE.md. **Platform:** [configure].

---

## Zero Assumptions

**Never assume. Always ask.** Before planning or implementing anything
non-trivial, run the project interview in
`~/.claude/templates/interview.md` verbatim. Seven grouped questions
covering Purpose/Users, Stack, Data Sensitivity, **AI Features
(mandatory)**, Scope/Constraints, Existing Context, Design/Brand.
Missing any question is a process failure. Answers flow into
`project-claude.md`, SPEC.md, and DECISIONS.md.

If the AI Features answer is yes, maybe, or "not yet but eventually":
load `~/.claude/templates/guardrails/ai-features.md` immediately. AI
is never a late-project bolt-on — adding it later re-triggers the
question and re-loads the template.

After the interview, proceed to **Project Setup** before writing code.

Zero Assumptions governs project-level unknowns (stack, users,
compliance, architecture). Implementation-level choices follow
pragmatism in `rules/code-voice.md`.

---

## Workflow

Superpowers drives all builds:
Brainstorm → Spec → Plan → TDD → Subagent Dev → Review → Finalize

Non-build tasks: Explore → Plan → Code → Commit.
Never write code before exploring and getting plan approval.

## Output Discipline

Match response length to task complexity. Mechanical tasks: 1-3 lines
confirming what was done. Analysis tasks: as long as the analysis
requires, no longer. Skip preambles ("Great question," "Let me think
through this"). Skip reasoning summaries unless asked. Skip offers to
continue ("Let me know if you want..."). The user can ask for more
detail; they can't un-read a wall of text.

## Verification Protocol

Check the project CLAUDE.md for the **ceremony level**. Default: `standard`.
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

At `standard`: Layer 1 + Layer 2 on non-trivial, Playwright on UI, all
gates. At `light`: Layer 1 only, security/destructive gates only.
At `minimal`: tests only, security/destructive gates only.

**Override rule:** downgrading a floor requires Tier A approval +
written justification in DECISIONS.md. Regulated floors (fintech,
healthcare) can be raised but never downgraded.

**Reassess ceremony level when:** the project first adds financial
transactions, health data, auth, external users, AI features, or
connects AI to a new data source. AI additions trigger re-read of
`ai-features.md` plus active industry templates.

Every completed task goes through verification layers.

**Layer 1 — Self-check (every task).** Mechanical facts only:
- **Impact analysis (before implementation):** read REGISTRY.md
  dependents + grep imports. Note in STATUS.md. If the target is NOT
  in REGISTRY.md, search the filesystem and add it before modifying.
  Shared components with 20+ dependents: read the full dependents
  list + 3-5 representative files covering different usage patterns.
- Tests pass? Run them.
- Output matches PLAN.md acceptance criteria point by point?
  **Money/auth/user-affecting tasks:** PLAN.md must have ≥3 known-
  answer test cases. Add before implementing, not after.
- SPEC.md alignment? If misalignment surfaces during the task, propose
  the spec change with rationale and get Tier A approval before
  continuing.
- Multi-component task: end-to-end flow works, not just units?
- Lint/type errors? Run check command.
- Build succeeds? Run build command.
- **UI changes + Playwright available:** axe-core audit + screenshots
  at mobile/desktop. If the project has dark mode, run axe-core in
  BOTH color schemes. Inherited violations: document in STATUS.md,
  don't block.
- Missing files that need changing?
- Created/deleted/renamed component? Update REGISTRY.md. Modified
  `stable` component? Reset to `verified`.
If any check fails, fix before proceeding. **Exception — cross-cutting
refactors:** note expected transient failures in PLAN.md with
`transient:` tag. Unlisted failures are real bugs — fix immediately.
All tests must pass at task completion.

**Layer 2 — Subagent review (non-trivial tasks).** Dispatch with ONLY:
task description, acceptance criteria, changed files. No conversation
history. Frame: "Find what's wrong." Subagent checks semantic
correctness (spec gaming), hidden coupling, test integrity (if PLAN.md
has known-answer pairs, verify tests match), DESIGN.md compliance
(UI tasks). Uses Sonnet. Workarounds approved by the project lead
(marked `// APPROVED: [reason]` in code) should not be flagged.

**Layer 2 is structural review, not an independent audit.** The
subagent shares the main agent's model family and blind spots. For
true independent review, use cross-model adversarial review (see
README, optional) or external human review.

**Trivial** (skip Layer 2): renaming, formatting, linting, config
one-liners, comment edits, import reordering, non-breaking version bumps.
**Non-trivial** (require Layer 2): anything changing behavior — features,
bug fixes, logic refactors, API/DB/auth changes, interactive UI, tests.
When in doubt: non-trivial.

## Phase Boundaries

A phase ends when the last task in the current PLAN.md phase completes
and its exit criteria are met. When this happens, load
`~/.claude/rules/phase-boundaries.md` and run the full procedure:
propose a phase snapshot, reconcile REGISTRY.md, prune DECISIONS.md,
resolve STATUS.md open questions, update PLAN.md. Then present the
forward-looking discussion (recommended first task, open questions,
observations). Then prompt about compaction.

Don't start next-phase work before the snapshot is written and
committed. Phase boundaries are the anchor point for session
reconstruction — skipping a snapshot breaks Tier 1 for every
subsequent session. At ship/stable milestones, also generate
`HANDOFF.md` from `templates/handoff.md`.

---

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
- **Prohibited:** 500+ line files without asking, abstractions for
  future flexibility, duplicated logic, `console.log` in production,
  generic errors, deprecated APIs, hallucinated packages.
- **Template conventions:** follow the starter's patterns for
  existing code; apply Forge standards to new code it doesn't cover.
  Security (GUARDRAILS.md) always overrides template conventions.
  On conflict (e.g., Forge wants layered architecture but template is
  flat), document in DECISIONS.md with "Template vs Forge" tag and
  default to following the template for existing code.

### Edit Strategy

Before the first edit to an existing file, decide: surgical or rewrite.
The criterion is **entanglement**, not file size.

**Surgical edits** (`str_replace` per change) suit one targeted fix
or several independent changes to unrelated parts of a file. Any size
file. 20 independent edits to a 2,000-line file are still surgical.

**Rewrite** (read the file, reason about all changes in one pass,
write the full file back) when the changes are entangled:

- Renaming a symbol that appears in multiple places in the same file
- Changing a function signature plus its callers in the same file
- Restructuring a section, reordering, or changing hierarchy
- Changing a type or interface plus its usages in the same file
- Any change where intermediate states would be syntactically or
  semantically broken

**Default to rewrite for 5+ pending edits in the same file** unless
you can confirm each edit is fully independent of the others. Partial-
failure states on sequential surgical edits leave the file worse than
either the starting or target state.

**Mid-task recovery.** If `str_replace` calls are failing on `old_str`
uniqueness, or you're on the 4th retry of the same edit, stop and
switch to rewrite. That's the signature of entanglement the upfront
assessment missed. Strategy switches are expected when the initial
assessment was wrong — they're not failures, and the sunk cost of
earlier edits shouldn't drive the decision. The rewrite cost is
bounded by file size; the thrashing cost is unbounded.

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
- REGISTRY.md updated after every task — primary defense against
  post-compaction hallucination.
- INVARIANTS.md entries without a canary test are unenforced claims.
- DECISIONS.md superseded entries use `~~[SUPERSEDED]~~` + blockquote
  warning.
- Logs are append-only. Secrets in logs → rotate and archive.
- Canaries protect invariants universally. New surfaces extend
  existing canaries in the same task.

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
12. Memory file integrity — commit REGISTRY/DECISIONS/INVARIANTS
    updates (`chore: update memory files`). Before compaction: `git
    diff` on memory files to catch unintended changes.
13. If `.claude/in-progress.md` exists, read it. Check expected
    transient failures before diagnosing failing tests. Don't
    panic-revert without understanding what was in flight.

**Tier 2 — On demand:** relevant DECISIONS.md domain, REGISTRY.md
component detail, snapshot detail.

**Tier 3 — Source code:** actual files targeted by REGISTRY.md paths.

---

## Compact Instructions

**When to compact:** proactively between task boundaries when context
is ~70% full. Never start a complex multi-file task with <30% context
remaining — compact first.

**Phase boundary compaction is a Tier A gate** — Claude recommends
yes/no with factors (context %, next phase weight, active
conversational context) and the project lead decides. Defer defaults:
<65% continue, ≥65% compact.

**Mid-task compaction:** `.claude/in-progress.md` is the survival
mechanism. Maintain continuously for refactors touching 5+ files.

Load `~/.claude/rules/compaction.md` before compacting or when
presenting the phase boundary compaction decision — it has the full
procedure (decision prompt template, factor definitions, pre-
compaction validation steps, standard and mid-task handling).

---

## ⚠️ Things Claude Gets Wrong

> Maintained per-project in the project `CLAUDE.md`. See template.
> If the same mistake appears across 3+ projects, promote it to the
> relevant global rules file. Review and update this section during
> health checks — if the same mistake appeared in the last 3 sessions,
> add it here.
