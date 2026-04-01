# Global Defaults — Rocky @ Nura Consulting

ML consultant + full-stack developer + designer at Nura Consulting.
Multi-client — each project has its own CLAUDE.md that extends these defaults.

Platform: Windows, PowerShell, Claude Code
**All shell commands must be PowerShell-compatible.** No bash-only syntax
(`export`, `chmod`, `source`, `#!/bin/bash`, `&&` chaining in older PS).
Use PowerShell equivalents or cross-platform npm/npx scripts.

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

Every completed task goes through three layers. Never skip a layer and
never declare "done" until layer 1 passes.

**Layer 1 — Self-check (every task, main agent).** Mechanical facts only.
Do not re-evaluate your reasoning — check observable outcomes:
- Do all tests pass? Run them.
- Does the output match the acceptance criteria in PLAN.md — point by point?
- If SPEC.md exists: does the output align with the system design? If the
  task changes the design, update SPEC.md first (requires approval).
- If this task assembled multiple components: does the complete flow work
  end-to-end, not just individual units?
- Are there lint/type errors? Run the check command.
- Did you miss any files that need changing?
If any check fails, fix it before proceeding. Do not explain why it's fine.

**Layer 2 — Subagent review (every non-trivial task).** Dispatch a review
subagent with ONLY: the task description from PLAN.md, the acceptance
criteria, and the changed files. Do NOT pass conversation history or your
reasoning. Frame it as: "Find what's wrong with this implementation given
these requirements." The subagent uses Sonnet for token efficiency.
Trivial tasks (renaming, formatting, config one-liners) skip this layer.

**Layer 3 — Codex adversarial (pre-merge).** After all tasks complete,
run `/codex:adversarial-review --base main` for cross-model design-level
challenge. Playwright MCP handles E2E validation.

## Subagent Delegation

Prefer subagents over doing everything in the main context. Use them for:
- **Investigation** — exploring code, reading multiple files, tracing bugs
- **Review** — Layer 2 verification above
- **Parallel tasks** — independent pieces of work that don't share state
- **Research** — Context7 lookups, documentation reading, pattern analysis

Keep the main context for: decision-making with Rocky, coordinating work,
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
  and present to Rocky for review before any UI work begins
- If neither approach fits, ask Rocky how to proceed
- Projects with no user-facing UI (pure API, CLI, automation) don't
  need a `DESIGN.md` — skip unless Rocky asks for one

### Skill Priority (lower number wins on conflict)

1. `DESIGN.md` — tokens, palette, typography, spacing, layouts (source of truth)
2. `ui-ux-pro-max` — design system generation + database queries
3. `frontend-design` — creative direction + anti-slop during implementation
4. Vercel `web-design-guidelines` — accessibility + compliance audit
5. Vercel `react-best-practices` — performance optimization
6. Vercel `composition-patterns` — component architecture

Stitch MCP and Figma MCP are available but not used on every project.
Only use them if the project CLAUDE.md enables them or Rocky asks for them.

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
   on project type and regulations identified during interview. Place
   in project root.
3. **Generate SPEC.md** (for projects with meaningful architecture:
   database, auth, multiple pages/screens, API layer, or expected
   long-term maintenance) from `~/.claude/templates/spec.md`. Present
   to Rocky for approval before any implementation begins. Skip for:
   bug fixes, config changes, one-off scripts, single-page landing
   pages, small CLI tools.
4. **Generate DESIGN.md** (if the project has user-facing UI) from
   `~/.claude/templates/design.md`. Mark generated values with `[VERIFY]`.
   Present to Rocky for review and editing before any UI work begins.
5. **Create STATUS.md and PLAN.md** per the Project State Files section.
6. **Verify `.gitignore`** covers `.env*`, `node_modules/`, build output,
   OS files, IDE files, and `.claude/logs/`.

---

## Project State Files

Maintain these files in every project root. Create them if they don't exist.

### `STATUS.md` — Current state snapshot
Create at session start if missing. Update whenever progress changes.
Include: current task/phase, what's done/in-progress/blocked, last modified
files, open questions waiting on Rocky.
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

### `.claude/logs/` — Session activity log
Append to `.claude/logs/YYYY-MM-DD.md` at end of each session (or before
`/compact`). Include: session summary, files changed, decisions made, tests
added, failures and resolutions. Logs are gitignored and append-only.
Add `.claude/logs/` to `.gitignore` if not already present.

### Session Recovery

After compaction or context loss:
1. Read project `CLAUDE.md` for stack, commands, and conventions
2. Read `SPEC.md` (if exists) for system design and constraints
3. Read `STATUS.md` for current progress
4. Read `PLAN.md` for the active plan
5. Read the latest entry in `.claude/logs/` for recent activity
6. Resume from where work left off — do not restart

**Normal session start (no compaction):** Read the project root files
before starting work: project `CLAUDE.md` (stack, commands, conventions),
`STATUS.md` (current progress), `PLAN.md` (active tasks), and `SPEC.md`
(system design) if they exist. These contain the state needed to
continue work without re-asking questions Rocky already answered.

---

## Compact Instructions

Before compacting: update `STATUS.md` and append to `.claude/logs/`.
Preserve: modified file list, active plan progress, test/verification
commands, design system references, and active SPEC.md decisions that
inform current work.

---

## ⚠️ Things Claude Gets Wrong

> Maintained per-project in the project `CLAUDE.md`. See template.
