---
description: Product development lifecycle — when to suggest user research, mockup review, copy work, design audits, and handoff skills during a Forge project. Proactive-suggestion rules, not mandatory steps.
---

# Product Lifecycle

> Forge's build loop (Superpowers: brainstorm → spec → plan → TDD →
> dev → review) doesn't cover every stage of product development.
> Discovery, design iteration, copy consistency, design-system
> maintenance, and handoff are real stages that happen regardless
> of whether tools exist to support them — and when Forge stays
> silent about them, they get done ad-hoc or not at all.
>
> This file maps those stages to the Anthropic design plugin skills
> and names when Claude should proactively suggest each one. These
> are suggestions, not mandates. The project lead opts in or skips.
>
> Scope: applies when the project includes user-facing UI. Pure
> backend, CLI, and scripting projects skip most stages.

## The Eight Stages

### Stage 1 — Discovery (after Q1 of interview, before Q2)

After the interview's Q1 answers (purpose, users, project type),
evaluate whether user research would add value. Signals:

- External consumer users
- Users you don't know well (not yourself, not a small internal team)
- Meaningful UX stakes (people will use this for real tasks, not a demo)
- No prior research evidence cited

**If signals present and existing user data available** (past
interview notes, support tickets, survey results, NPS, analytics) →
proactively suggest `/research-synthesis`. Output feeds remaining
interview answers (Q1 users refined, Q7 design direction informed by
evidence) and lands in DECISIONS.md under a "User Research" domain.

**If signals present but no existing data** → proactively suggest
`/user-research`. Produces a research plan, interview guide, or
survey design the project lead executes separately. Return to this
stage after data is collected.

**If signals absent** → skip. Note in DECISIONS.md UI/UX domain
that research was considered and deferred (or deemed unnecessary),
so future sessions see the reasoning.

Suggested phrasing: *"Based on your Q1 answers, user research would
add real value here. Do you have existing user data (interviews,
tickets, surveys) we should synthesize, or should we plan research
before continuing the interview? Or skip this and continue?"*

### Stage 2 — Interview + scaffolding

Unchanged from `templates/interview.md` and `templates/project-setup.md`.
Runs Q2–Q7, then project setup steps 1–10.

### Stage 3 — DESIGN.md creation (project-setup step 4)

Handled by the revised step 4. Claude invokes Figma MCP,
`ui-ux-pro-max`, or the template depending on Q7 answers.

After DESIGN.md exists, proactively suggest `/ux-copy` to define the
Copy Voice section (tone, person, tense, error-message pattern,
empty-state pattern). Defining voice once prevents the inconsistent-
copy failure where every implementing subagent invents its own voice.
Skippable.

### Stage 4 — Mockup iteration (before implementation)

**Every project mockups before implementing.** This is not
optional. Writing TSX to figure out what a screen should look like
is the path to UX-all-over-the-place. Mockups first, implementation
second.

Mockup format is the project lead's choice:
- **Figma file** (preferred when available)
- **Images in `design/mockups/`** (PNG, screenshots, hand sketches,
  AI-generated concepts)
- **Prose descriptions** in a markdown file under `design/` when
  visual mockups aren't practical

Before Stage 4 begins, ask the project lead which format they'll
use. Create the directory if needed.

During each round of mockups, proactively suggest:

- `/design-critique` on the mockup set. Surfaces hierarchy,
  consistency, and usability issues while they're cheap to fix.
- `/accessibility-review` on the mockup set. Catches contrast,
  touch target size, keyboard-flow issues *before* they get baked
  into code. This is distinct from Layer 1's axe-core, which runs
  against built components — accessibility-review runs against the
  design.

Iterate until mockups are accepted. Then update PLAN.md Phase 1 to
reference the accepted mockups as the implementation target. Only
then does Superpowers' build loop start.

### Stage 5 — Implementation (Superpowers build loop)

Superpowers drives brainstorm → spec → plan → TDD → dev → review
per feature. Forge verification layers run around it (Layer 1
self-check, Layer 2 subagent review on non-trivial tasks).

During component implementation: when writing user-facing strings
(buttons, errors, empty states, toasts, dialogs, onboarding), the
implementing subagent should invoke `/ux-copy` with reference to
DESIGN.md's Copy Voice section. This is the per-string use
`/ux-copy` was designed for — "what should this button say?",
"write the empty state for the Dump screen" — with voice already
defined upstream so copy stays consistent.

### Stage 6 — Branch-to-main review (UI-touching branches, standard+ ceremony)

Parallel to the "Optional: Cross-Model Adversarial Review" block in
the README. Runs at branch boundaries, not per-phase, not per-task.

Before merging a branch that modified UI files (files matching
`rules/design-workflow.md` globs), proactively suggest:

- `/design-critique` on the changed UI surfaces. Catches hierarchy,
  consistency, and usability regressions introduced during
  implementation.
- `/accessibility-review` on the changed surfaces. Complements
  Layer 1's axe-core — axe catches ~30–40% of issues automatically;
  `/accessibility-review` does structured review beyond detection
  thresholds (focus traps, live regions, reading order, non-text
  content).
- `/design-system` if the branch changed 5+ components or
  introduced new patterns. Catches token drift and naming
  inconsistencies in the diff.

Findings disposition: fixable issues go into PLAN.md as tasks
before merge. Deferred issues go into STATUS.md as known issues.
Summary goes into the phase snapshot if this is also a phase
boundary.

Optional, skippable at project lead's discretion. For regulated
industries (fintech, healthcare), strongly recommended.

### Stage 7 — Health check maintenance (long-running projects, monthly)

The README already describes monthly health checks (dependency
audits, bundle size, dead code, canary status, AI cost trends).
Add one proactive suggestion to that cadence:

- `/design-system` whole-system audit. Catches token drift,
  hardcoded values, naming inconsistencies, orphan patterns, and
  undocumented components across the entire codebase — not just
  the diff. Output lands in PLAN.md as cleanup tasks.

Skippable on short-lived or just-shipped projects. Relevant once
there are enough components and sessions of code drift to make an
audit worthwhile.

### Stage 8 — Ship / stable milestone

When generating HANDOFF.md (per `templates/handoff.md`),
proactively suggest `/design-handoff` to produce a design-spec
supplement. HANDOFF.md covers architecture, local dev, maintenance,
and product context; `/design-handoff` covers the design-spec
subset — tokens catalogued, component props documented, interaction
states specified, responsive breakpoints listed, animation details.

The two documents live side by side: `HANDOFF.md` for maintainers,
`DESIGN-HANDOFF.md` (or appended section in HANDOFF.md) for anyone
taking over the design work. Place is the project lead's choice.

## Suggestion Mechanism

Claude's responsibility is proactive surfacing at the right
moment — not silent invocation. When a stage's conditions are met,
Claude should:

1. State what stage this is and what the suggested skill does in
   one sentence.
2. Name the tradeoff — what running it costs (time, context,
   session length) vs. what skipping it risks.
3. Ask the project lead to opt in or skip.

Example (Stage 1):
> *"Your Q1 answer mentions building for users you haven't talked
> to yet. `/user-research` can produce an interview guide to fill
> that gap before we continue scaffolding. Takes ~10 minutes and
> the guide sits in `research/` until you use it. Skip if you'd
> rather move fast and course-correct later. Run it?"*

Example (Stage 6):
> *"This branch touched 8 UI components. Before merge, I can run
> `/design-critique` and `/accessibility-review` on the changed
> surfaces. Findings that are fixable go into PLAN.md; deferred
> issues go into STATUS.md. Takes ~5 minutes. Skip for internal-
> tool-grade surfaces; recommended for user-facing work. Run it?"*

## What This File Does Not Do

This file does not mandate any of the eight stages. Nothing here
overrides ceremony levels, interview results, or project lead
decisions. Projects can legitimately skip Stages 1, 6, 7, and 8
based on their type — internal tool with no real users, prototypes
that won't run long enough for health checks, projects that won't be
handed off. Stage 4 (mockup iteration) applies to every project
with user-facing UI.

The file exists so those stages are *visible* — so the project
lead sees the choice being made rather than the stage silently
being skipped.
