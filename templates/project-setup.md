# Project Setup (New Projects)

> Kickoff procedure for new projects. Loaded once at the start of a
> project, not every session. CLAUDE.md has a 3-line pointer to this
> file. Run after completing the interview in `templates/interview.md`.

## Before the setup steps

**Inherited codebase?** Run an onboarding scan (subagent) to populate
REGISTRY.md from the filesystem before any feature work. The scan
should produce one REGISTRY.md entry per significant component,
endpoint, or utility. Review the scan output before accepting.

**Starting from a template/starter?** (Vercel template, Dawn theme,
Railway starter, shadcn template, etc.):

1. Clone/install the template first (or confirm the project lead has).
2. Read the template's README.md and any configuration docs.
3. Scan file structure — populate REGISTRY.md from what exists.
   **For templates with 50+ files:** chunk the scan across 2-3
   subagents by directory (e.g., components/, app/, lib/). Each
   subagent produces REGISTRY.md entries for its chunk. Main agent
   merges. This prevents a single-session token bomb.
4. Pre-populate DECISIONS.md with the template's baked-in choices
   (auth, styling, state management, deployment target) marked as
   "Template default" — these aren't decisions to revisit casually.
5. If the template has 20+ files, treat as template-based project.
   Under 20 files, treat as greenfield with scaffolding.
6. Proceed with the numbered setup steps below.

## Setup Steps

1. **Project CLAUDE.md** from template. **Derive ceremony level** by
   cross-referencing the interview answers (Q3 data sensitivity,
   Q4 AI features, Q1 purpose/users) against the Ceremony Floors
   table in global CLAUDE.md. Propose the minimum applicable floor
   and confirm with the project lead before writing it into the
   project CLAUDE.md. Regulated floors (fintech, healthcare) are
   mandatory — not a starting suggestion.
2. **Project GUARDRAILS.md** — combine relevant guardrail templates
   from `~/.claude/templates/guardrails/`. Confirm selection with
   project lead before generating. Include `ai-features.md` if the
   interview answered yes to Q4 (AI features).
3. **SPEC.md** — create from `templates/spec.md` for projects with
   3+ tables, auth, shared state across pages, external API consumers,
   or 3+ months maintenance. Skip for bug fixes, scripts, landing
   pages, simple CRUD ≤2 tables.

   **After generating SPEC.md** (at `standard`+ ceremony, if spec
   exceeds 50 lines): dispatch a subagent with SPEC.md + project
   GUARDRAILS.md to find ambiguities, contradictions, and missing
   edge cases. **For template-based projects:** also include the
   template's schema/migration files, auth config, and existing API
   routes — the subagent cannot catch spec-template conflicts without
   seeing the existing code. Then add your own clarification questions:
   "Before I proceed, these areas need your input: [3-5 specific
   questions about ambiguities or decisions with multiple valid
   approaches]." Present the spec, the validation findings, and the
   questions together for approval. Answers go into DECISIONS.md.
4. **DESIGN.md** — if UI exists. Create from `templates/design.md`.
   Figma-first; otherwise template + `[VERIFY]` markers. Present for
   review before UI work. See `rules/design-workflow.md` for details.
5. **STATUS.md + PLAN.md** — from `templates/status.md` and
   `templates/plan.md`.
6. **REGISTRY.md** — from `templates/registry.md`. Start empty,
   populate as built. For inherited/template projects, the scan
   above already populates initial entries.
7. **INVARIANTS.md** — from `templates/invariants.md`. Seed from
   guardrails (the "must hold" statements in each applied template)
   + SPEC.md (architectural constraints).
8. **DECISIONS.md** — from `templates/decisions.md`. Seed from
   interview decisions captured in question 1-7 answers.
9. **Verify `.gitignore`** — `.env*`, `node_modules/`, build output,
   OS/IDE files, `.claude/logs/`, `.claude/in-progress.md`.
10. **Phase 1 exit criteria** in PLAN.md — what must be true before
    the first phase completes.

## After setup

Commit the scaffold. Begin Phase 1 of PLAN.md.
