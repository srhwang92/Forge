# Project: [NAME]

> [One sentence — what it is, who for]

**Production:** [yes | no | personal]

## Stack

- Framework:
- Language:
- Runtime:
- Package Manager:
- Database:
- Auth:
- Hosting:
- Path alias:

## Commands

```
dev:
build:
test:
lint:
typecheck:
```

## Working in this codebase

Use these rails. Don't bypass them silently — skipping the build loop, skipping a Context7 lookup, or skipping code review on non-trivial work means you're working against the framework, not with it.

- Use Superpowers for feature work: brainstorming → writing-plans → executing-plans (or subagent-driven-development). Don't write code before the design and plan are approved.
- Query Context7 before referencing a library API, function signature, or platform behavior. Training data is outdated even for libraries you "know."
- Non-trivial changes get reviewed before commit. Pick a mechanism (Superpowers' `requesting-code-review`, `codex:rescue`, or a fresh subagent with adversarial framing). Don't self-review with the same framing you wrote the code with.
- Suggest the matching Anthropic design skill when the task fits: `/design-critique` (review mockup or design), `/accessibility-review` (WCAG audit), `/ux-copy` (microcopy, errors, empty states, CTAs), `/design-system` (audit or extend the system), `/design-handoff` (design → dev spec), `/user-research` (plan research), `/research-synthesis` (synthesize existing research).

## Project memory

- `SPEC.md` — what the project is, scope, in/out, constraints. Drafted at kickoff. Updated when scope shifts.
- `ROADMAP.md` — phases until ship. Drafted at kickoff. `PHASE.md` pulls the next phase from it.
- `MEMORY.md` — architecture sketch, key decisions, current state, open questions. Update when a non-trivial decision is made.
- `PHASE.md` — current phase goal + tasks + done-when. Replaced when the phase ends.
- `.claude/logs/YYYY-MM-DD.md` — append-only session log.
- "Things Claude Gets Wrong" in this file — add when project-specific mistakes surface.

When an assumption must hold across phases (RLS, no-float-money, IDOR ownership), write a canary test in `tests/canaries/`. Header comment documents what it guards (see `~/.claude/templates/canary-header.md`).

## After compaction

Read this `CLAUDE.md` → `MEMORY.md` → `PHASE.md` → latest `.claude/logs/*.md` → `GUARDRAILS.md` (if exists). Don't reconstruct state from memory.

If you haven't read a referenced file/symbol in the last ~10 turns, re-read it.

## Phase boundaries

When `PHASE.md`'s done-when is met: append a log entry, compress what's worth keeping into `MEMORY.md`, prune what hasn't paid off (entries not cited recently, decisions superseded, stale open questions). AI accretes; pruning is the under-used operation.

## Pushback

Push back when the project lead is about to make an expensive mistake — name the risk, propose a lighter alternative, comply if overridden. When the project lead pushes back on you with frustration about scope or process: simplify, don't justify. Loud feedback resets; soft feedback compounds.

## Things Claude Gets Wrong

> Empty at start. Add when project-specific mistakes surface. Prune at phase boundaries.
