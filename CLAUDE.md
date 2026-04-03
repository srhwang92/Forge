# Global Defaults

<!-- Customize this section for your role and setup -->
Multi-client development — each project has its own CLAUDE.md that
extends these defaults.

**Platform:** Configure your OS and shell in the section below.
<!-- Example for Windows: -->
<!-- All shell commands must be PowerShell-compatible. No bash-only syntax -->
<!-- (`export`, `chmod`, `source`, `#!/bin/bash`). Use cross-platform -->
<!-- npm/npx scripts or PowerShell equivalents. -->
<!-- Example for macOS/Linux: -->
<!-- Standard bash/zsh commands. No OS-specific assumptions. -->

---

## Zero Assumptions

**Never assume. Always ask.** Before planning or implementing anything
non-trivial, interview me. Cover:
- What exactly do I want built and why?
- What are the constraints (tech stack, budget, timeline, accessibility)?
- What are the edge cases and expected behaviors?
- What should it NOT do?
- Are there existing patterns, files, or conventions to follow?
- Who is the end user and what's the context?
- What data does this handle? Is any of it PII, financial, or health data?
- Who is the target audience geographically? (determines which privacy
  regulations apply — GDPR, CCPA, HIPAA, etc.)
- Do you have existing design assets? (Figma file with design system,
  brand guidelines, reference sites, or starting from scratch?)

After the interview, proceed to **Project Setup** to generate project
files from templates before writing any code.

If something is ambiguous, underspecified, or could go multiple ways — stop
and ask. Do not fill in blanks with reasonable defaults. Do not infer intent.
The cost of asking one extra question is always lower than the cost of
rebuilding something built on a wrong assumption.

This applies to architecture, design, data models, naming, flow, copy,
and every decision that affects the final product. The only exceptions are
mechanical tasks with no decision points (formatting, linting, running
tests as instructed).

---

## Workflow

Superpowers drives all builds automatically:
Brainstorm → Spec → Plan → TDD → Subagent Dev → Review → Finalize

Non-build tasks (quick fixes, config): Explore → Plan → Code → Commit.
Never write code before exploring and getting plan approval.

## Verification Protocol

Check the project CLAUDE.md for the **ceremony level** which controls
how much verification runs. If not set, default to `standard`.
- **full**: all layers on every task (including trivial), Layer 3 Codex
  per-feature, Playwright on every UI change, all approval gates (high-
  stakes client work, regulated industries)
- **standard**: all layers on non-trivial tasks, Layer 3 pre-merge only,
  Playwright on UI tasks, all gates (most projects — the default)
- **light**: Layer 1 only, security/destructive gates only, skip
  Playwright (personal projects, prototypes)
- **minimal**: run tests only, no approval gates except destructive and
  security ops (one-off scripts, experiments)

Security guardrails (GUARDRAILS.md) apply regardless of ceremony level.

Every completed task goes through three layers. Never skip a layer and
never declare "done" until layer 1 passes.

**Layer 1 — Self-check (every task, main agent).** Mechanical facts only.
Do not re-evaluate your reasoning — check observable outcomes:
- **Impact analysis (before implementation):** if modifying a component,
  read REGISTRY.md to identify dependents + grep imports as backup.
  Note affected components and their tests in STATUS.md (3 lines).
- Do all tests pass? Run them.
- Does the output match the acceptance criteria in PLAN.md — point by point?
- If SPEC.md exists: does the output align with the system design? If the
  task changes the design, update SPEC.md first (requires approval).
- If this task assembled multiple components: does the complete flow work
  end-to-end, not just individual units?
- Are there lint/type errors? Run the check command.
- Does the project build successfully? Run the build command. Tests can
  pass while the app fails to compile or start.
- **If this task changed UI** and Playwright MCP is available: run a
  targeted check on the affected pages — axe-core accessibility audit,
  console error detection, and screenshot at mobile + desktop widths.
  **axe-core catches ~30-40% of accessibility issues.** Passing axe-core
  does not equal WCAG compliance — keyboard navigation, focus order,
  screen reader flow, and cognitive load require manual verification
  or code-level review (Layer 2). Never cite "0 axe violations" as
  proof of accessibility.
  For cross-cutting changes (design tokens, global CSS, shared
  components), check 2-3 representative pages rather than all pages.
  Compare screenshots against previous captures if a baseline exists;
  if no baseline, present screenshots for visual approval and
  save as the new baseline. Full Playwright E2E suite runs at Layer 3,
  not here. **Inherited codebases:** if Playwright finds pre-existing
  violations Claude didn't introduce, document them in STATUS.md for
  future remediation — don't block the current task on inherited debt.
- Did you miss any files that need changing?
- **If you created, deleted, or renamed a component:** verify REGISTRY.md
  reflects the change. If you modified a `stable` component, reset its
  status to `verified`.
If any check fails, fix it before proceeding. Do not explain why it's fine.

**Layer 2 — Subagent review (every non-trivial task).** Dispatch a review
subagent with ONLY: the task description from PLAN.md, the acceptance
criteria, and the changed files. Do NOT pass conversation history or your
reasoning. Frame it as: "Find what's wrong with this implementation given
these requirements." The subagent must check:
- **Semantic correctness:** does the code do what the spec *means*, not
  just what a literal reading allows? (specification gaming check)
- **Hidden coupling:** does any changed file depend on something
  undocumented — a global store, an implicit provider, a database
  schema assumption not captured in types?
- **Test integrity:** do the tests verify spec behavior, or do they
  just confirm the implementation matches itself?
- **DESIGN.md compliance (UI tasks):** are all visual values coming from
  design tokens, not hardcoded? Correct icon library?
The subagent uses Sonnet for token efficiency.
Trivial tasks (renaming, formatting, config one-liners) skip this layer.

**Layer 3 — Codex adversarial (pre-merge).** After all tasks complete,
run `/codex:adversarial-review --base main` for cross-model design-level
challenge. Playwright MCP handles E2E validation.

## Subagent Delegation

Prefer subagents over doing everything in the main context. Use them for:
- **Investigation** — exploring code, reading multiple files, tracing bugs
- **Review** — Layer 2 verification above
- **Parallel tasks** — independent pieces of work that don't share state.
  **Never dispatch parallel subagents that modify the same files** —
  the second write silently overwrites the first.
- **Research** — Context7 lookups, documentation reading, pattern analysis

**Subagent context:** When dispatching an implementation subagent (not
review), include relevant SPEC.md architecture decisions so the subagent
doesn't contradict the project's design. A subagent building a new
endpoint needs to know the project uses RLS, not application-level auth.

Keep the main context for: decision-making with the project lead, coordinating work,
and tasks that depend on conversation history. If a task requires reading
more than 3-4 files or could run independently, it should be a subagent.

---

## Production Engineering Standards

These rules override Claude's default instincts. LLMs have documented
tendencies toward overcomplexity, hardcoded values, and "almost right" code.

Detailed standards are in path-scoped rules that load automatically:
- @~/.claude/rules/code-voice.md (how code reads — human voice + AI rigor)
- @~/.claude/rules/vibe-coding-discipline.md (foundation checks, pushback, dead code, pattern safety)
- @~/.claude/rules/frontend-standards.md (tokens, CSS architecture, components)
- @~/.claude/rules/backend-standards.md (API-first, clean architecture, observability)
- @~/.claude/rules/git-conventions.md (commit format, staging discipline, Cursor boundary)
- @~/.claude/rules/context7-usage.md (two-step workflow, query formatting, token discipline)

### Simplicity Mandate

**Always implement the simplest solution that satisfies the requirements.**
Before writing any code, ask: "Is there a simpler way?" If yes, use it.

- No premature abstractions — build for current requirements, not imagined
  future ones (YAGNI)
- No factory patterns when a function works. No class when a plain object
  works. No custom error hierarchy when Error with a message works.
- No wrapper layers that don't add behavior
- No utility functions that duplicate standard library or installed deps
- If ~100 lines does what ~30 lines can — use the shorter version
- The test: "Would a senior engineer call this overcomplicated?" If yes,
  simplify before committing.

### Never Hardcode — Always Systematize

**Every value that could change or be reused must be a token, variable,
constant, or config entry.** Never inline raw values — not colors, not
spacing, not font sizes, not API URLs, not timeouts, not magic numbers.
See the frontend and backend rules files for specifics.

### Incremental Development

Never generate monolithic blocks. Build the smallest working piece, verify
it passes tests, then build the next piece. Each increment has its own
tests, compiles clean, and can be reviewed independently.

**When modifying existing files:** change only the lines relevant to the
task. Never regenerate an entire file to fix a 3-line bug — read the file,
identify the specific lines to change, and edit surgically. Regenerating
unchanged sections introduces bugs in code that was working.

### Anti-Patterns — Explicit Prohibitions

Never do any of the following:
- Generate 500+ lines in a single file without asking if it should be split
- Create abstractions "for future flexibility" that aren't needed now
- Duplicate logic that exists elsewhere in the codebase — import it
- Use `console.log` for production logging — use structured logging
- Write generic error messages ("Something went wrong") — include context
- Use deprecated APIs or packages without flagging it
- Hallucinate package names — verify packages exist before importing

---

## Design System Enforcement

**Before any UI work, check for `DESIGN.md` in the project root.** If it exists,
it is the single source of truth for all visual decisions — typography, colors,
spacing, base unit, layouts, component patterns. Every component, page, and style
must conform to it. Never deviate without my explicit approval.

If `DESIGN.md` does not exist and the task requires design decisions:
- For projects with a Figma file: extract tokens via Figma MCP
- Otherwise: generate from `~/.claude/templates/design.md` using
  `ui-ux-pro-max` for style data, mark all values with `[VERIFY]`,
  and present for review before any UI work begins
- If neither approach fits, ask the project lead how to proceed
- Projects with no user-facing UI (pure API, CLI, automation) don't
  need a `DESIGN.md` — skip unless the project lead asks for one

### Skill Priority (lower number wins on conflict)

1. `DESIGN.md` — tokens, palette, typography, spacing, layouts (source of truth)
2. `ui-ux-pro-max` — design system generation + database queries
3. `frontend-design` — creative direction + anti-slop during implementation
4. Vercel `web-design-guidelines` — accessibility + compliance audit
5. Vercel `react-best-practices` — performance optimization
6. Vercel `composition-patterns` — component architecture

Stitch MCP and Figma MCP are available but not used on every project.
Only use them if the project CLAUDE.md enables them or the project lead asks for them.

---

## Guardrails

Global guardrails: @~/.claude/GUARDRAILS.md — always active, apply to every project.
Project guardrails: if `GUARDRAILS.md` exists in the project root, read it before
making changes. Project guardrails extend the global ones.
Never bypass any guardrail without my explicit approval.

---

## Project Setup (New Projects)

When starting a new project, after the zero-assumptions interview:

1. **Generate project CLAUDE.md** from `~/.claude/templates/project-claude.md`.
   Fill in all REQUIRED sections from interview answers. Include optional
   sections only if relevant. **Strip all HTML comments and placeholder
   instructions** — the generated file should be clean, not a template.
   Place in project root.
2. **Generate project GUARDRAILS.md** from the relevant templates in
   `~/.claude/templates/guardrails/`. Combine applicable templates based
   on project type and regulations identified during interview. **Confirm
   template selection with the project lead** before generating — list which
   templates will be applied and why. A missing template means missing
   safety rules for the life of the project. Place in project root.
3. **Generate SPEC.md** (for projects with meaningful architecture:
   database, auth, multiple pages/screens, API layer, or expected
   long-term maintenance) from `~/.claude/templates/spec.md`. Present
   for approval before any implementation begins. Skip for:
   bug fixes, config changes, one-off scripts, single-page landing
   pages, small CLI tools.
4. **Generate DESIGN.md** (if the project has user-facing UI).
   - **Figma-first:** if the project has a Figma file with design tokens, extract
     values via Figma MCP (`get_variable_defs`) and populate DESIGN.md
     from the template with real values. Figma Variables are the source
     of truth — not AI-generated placeholders.
   - **No Figma:** generate from `~/.claude/templates/design.md` using
     `ui-ux-pro-max` for style data, mark values with `[VERIFY]`.
   - Either way, present for review before any UI work begins.
5. **Create STATUS.md and PLAN.md** per the Project State Files section.
6. **Create REGISTRY.md** from `~/.claude/templates/registry.md` (for
   projects beyond trivial scripts). Start empty — populated as
   components are built.
7. **Create INVARIANTS.md** from `~/.claude/templates/invariants.md`
   (for projects with foundational constraints). Seed with invariants
   from the guardrail templates and SPEC.md constraints.
8. **Create DECISIONS.md** from `~/.claude/templates/decisions.md`.
   Seed with decisions from the interview (stack choice, auth approach,
   key architectural decisions).
9. **Verify `.gitignore`** covers `.env*`, `node_modules/`, build output,
   OS files, IDE files, and `.claude/logs/`.
10. **Define Phase 1 exit criteria** in PLAN.md — what must be true for
    the first phase to be complete. Reference REGISTRY.md and
    INVARIANTS.md where applicable.

---

## Project State Files

Maintain these files in every project root. Create them if they don't exist.

### `STATUS.md` — Current state snapshot
Create at session start if missing. Update whenever progress changes.
Include: current task/phase, what's done/in-progress/blocked, last modified
files, open questions waiting on the project lead.
**Open questions section:** track uncertainties, unverified assumptions,
and items waiting on the project lead. Persists across compaction.
Blocking questions must be resolved before phase gates.
**Long projects:** when STATUS.md exceeds ~50 lines or a major phase
completes, archive the current content to `.claude/logs/` and reset
STATUS.md to reflect only the current phase.

### `PLAN.md` — Active implementation plan
Create when a plan is approved. Update as tasks complete — check off finished
items, note deviations. If Superpowers generates a plan file, adopt it as
`PLAN.md` or link to it.
**Phase transitions:** when a plan is fully complete and new work begins,
rename the completed plan to `PLAN-v1.md` (or archive to `.claude/logs/`)
and start a fresh `PLAN.md` for the new phase.

### `REGISTRY.md` — Living codebase index
Create from `~/.claude/templates/registry.md` when the first component is
built. One line per component/endpoint/utility — name, file path, purpose,
lifecycle status, tests, rollback commit. Updated incrementally after every
task that creates, modifies, or deletes a component.
**This is the primary defense against "what exists?" confusion after
compaction.** If REGISTRY.md is current, Claude never hallucinates
components or forgets what was built.

### `DECISIONS.md` — Domain-organized reasoning memory
Create from `~/.claude/templates/decisions.md` when the first non-trivial
decision is made. Organized by domain (Auth, Payments, Data Model, etc.),
not chronologically. Domain knowledge (terms, rates, regulatory rules) goes
at the top of each domain section. Superseded decisions are marked
`[SUPERSEDED by: new-decision]` — skipped during reads.
**This is the primary defense against "why is it built this way?"
confusion after compaction.**

### `INVARIANTS.md` — System constraints with automated enforcement
Create from `~/.claude/templates/invariants.md` when foundational
architecture is established. Short, testable statements. Every invariant
must reference a canary test in `tests/canaries/`. An invariant without
a canary is an unenforced claim.

### `tests/canaries/` — Automated invariant enforcement
Canary tests verify ASSUMPTIONS and CONSTRAINTS, not features. One test per
invariant. Run with the normal test suite. If an assumption breaks, the
canary fails — alerting before dependent code breaks. Canary tests are
protected (never delete without approval).

### `.claude/snippets/` — Verified reusable patterns (optional)
Project-specific code patterns that have been tested and should be reused
exactly. Only created when a pattern appears 3+ times. Each snippet
references the test that verifies its specific pattern.

### `.claude/snapshots/phase-N.md` — Phase boundary records
Generated at the end of each phase from `~/.claude/templates/phase-snapshot.md`.
Captures: what was built, verification evidence, decisions made, known issues
deferred. The **summary section** (~10 lines) is read at Tier 1 every session.
Detail sections are read at Tier 2 on demand.

### `.claude/verified/` — Verification evidence
Persisted test output, Playwright results, and security check results.
Evidence that proves the system was correct at a specific point in time.
Nothing enters this directory without being actually executed and passing.

### `.claude/logs/` — Session activity log
Append to `.claude/logs/YYYY-MM-DD.md` at end of each session (or before
`/compact`). Logs are gitignored and append-only.
Add `.claude/logs/` to `.gitignore` if not already present.

**Each log entry must separate facts from reasoning:**

**Facts (observable, verifiable):**
- Files changed (list them)
- Tests added/modified and their pass/fail status
- Commands run and their output

**Decisions (with anti-bias structure):**
For each non-trivial decision, document:
- **Chose:** what was implemented
- **Rejected:** what alternatives were considered and why they lost
- **Assumptions:** what must be true for this decision to be correct —
  written as falsifiable statements, not justifications
- **Verify:** a runnable command or test the project lead can execute to confirm
  the decision is sound — not a prose claim that it works

**Do not write self-congratulatory summaries.** "Implemented a clean,
efficient solution" is Claude grading itself. "Added RLS policy —
verify with: `supabase db test` or manually query as another user"
is actionable.

### `MAINTENANCE.md` — Post-ship knowledge transfer
Generate when a project ships or reaches a stable milestone. Not
maintained during active development — generated once at completion,
updated on major changes.

**Structure:**
- **Dependencies:** what breaks if not updated, which have known
  EOL dates, how to check for vulnerabilities (`npm audit`/equivalent)
- **Secrets & rotation:** what credentials exist, where they're stored,
  when they expire, how to rotate them
- **Backups:** what's backed up, what isn't, how to verify backups work
- **Fragile areas:** the 3-5 most complex or fragile parts of the
  codebase with file paths (not explanations of why they're clever —
  just what to be careful around)
- **Health checks:** runnable commands that verify the system is
  working correctly (not prose claims — actual commands with expected
  output)
- **Questions you should be able to answer:** 5-10 questions about
  the codebase that the maintainer should understand. Not answers —
  questions. If you can't answer one, the referenced file tells you
  where to learn. Example: "Why does the orders table use RLS but
  audit_logs doesn't? → see `supabase/migrations/003_rls.sql`"

### Session Recovery — Tiered Reconstruction

After compaction or context loss, rebuild context in tiers. Complete
Tier 1 before writing any code.

**Tier 1 — Always read at session start (system map):**
1. Project `CLAUDE.md` — stack, commands, conventions
2. Project `GUARDRAILS.md` (if exists) — project-specific safety rules
3. `SPEC.md` (if exists) — architecture and design constraints
4. `REGISTRY.md` (if exists) — what exists, where, lifecycle status
5. `INVARIANTS.md` (if exists) — what must always be true
6. `DECISIONS.md` table of contents — domain list only (not full entries)
7. Latest phase snapshot summary (`.claude/snapshots/`) — verified state
8. `STATUS.md` — current progress + open questions
9. `PLAN.md` — active tasks
10. Latest entry in `.claude/logs/` — recent activity

**Tier 2 — Read on demand based on current task:**
- `DECISIONS.md` domain section relevant to the task (skip `[SUPERSEDED]`)
- `REGISTRY.md` detail for components being modified (dependents, tests)
- `.claude/verified/` evidence for the area being modified
- Phase snapshot detail if working near a phase boundary

**Tier 3 — Source code (read when actively working):**
- Actual code files, targeted by REGISTRY.md file paths
- Use impact analysis: read REGISTRY.md dependents + grep imports

**Normal session start (no compaction):** Read Tier 1 files before
starting work. These contain the state needed to continue without
re-asking questions already answered.

---

## Compact Instructions

Before compacting: update `STATUS.md` and append to `.claude/logs/`.
Preserve: modified file list, active plan progress, test/verification
commands, design system references, and active SPEC.md decisions that
inform current work.

---

## ⚠️ Things Claude Gets Wrong

> Maintained per-project in the project `CLAUDE.md`. See template.
> If the same mistake appears across 3+ projects, promote it to the
> relevant global rules file (frontend-standards, backend-standards,
> or GUARDRAILS.md) so it's prevented system-wide.
