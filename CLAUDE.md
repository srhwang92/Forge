# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## Working in this codebase

Use these rails. Don't bypass them silently — skipping the build loop, skipping a Context7 lookup, or skipping code review on non-trivial work means you're working against the framework, not with it.

- Use Superpowers for feature work: brainstorming → writing-plans → executing-plans (or subagent-driven-development). Don't write code before the design and plan are approved.
- Query Context7 before referencing a library API, function signature, or platform behavior. Training data is outdated even for libraries you "know."
- Non-trivial changes get reviewed before commit. Pick a mechanism (Superpowers' `requesting-code-review`, `codex:rescue`, or a fresh subagent with adversarial framing). Don't self-review with the same framing you wrote the code with.
- Suggest the matching Anthropic design skill when the task fits: `/design-critique` (review mockup or design), `/accessibility-review` (WCAG audit), `/ux-copy` (microcopy, errors, empty states, CTAs), `/design-system` (audit or extend the system), `/design-handoff` (design → dev spec), `/user-research` (plan research), `/research-synthesis` (synthesize existing research).

## Project memory

- If a project has no `CLAUDE.md`, run `~/.claude/templates/interview.md` (5 questions) to scaffold it before anything else.
- `SPEC.md` — what the project is, scope, in/out, constraints. Drafted at kickoff. Updated when scope shifts.
- `ROADMAP.md` — phases until ship. Drafted at kickoff. `PHASE.md` pulls the next phase from it.
- `MEMORY.md` — architecture sketch, key decisions, current state, open questions. Update when a non-trivial decision is made.
- `PHASE.md` — current phase goal + tasks + done-when. Replaced when the phase ends.
- `.claude/logs/YYYY-MM-DD.md` — append-only session log.
- "Things Claude Gets Wrong" in project `CLAUDE.md` — add when project-specific mistakes surface.

When an assumption must hold across phases (RLS, no-float-money, IDOR ownership), write a canary test in `tests/canaries/`. Header comment documents what it guards (see `templates/canary-header.md`).

## After compaction

Read project `CLAUDE.md` → `MEMORY.md` → `PHASE.md` → latest `.claude/logs/*.md` → `GUARDRAILS.md` (if exists). Don't reconstruct state from memory.

If you haven't read a referenced file/symbol in the last ~10 turns, re-read it.

## Phase boundaries

When `PHASE.md`'s done-when is met: append a log entry, compress what's worth keeping into `MEMORY.md`, prune what hasn't paid off (entries not cited recently, decisions superseded, stale open questions). AI accretes; pruning is the under-used operation.

## Pushback

Push back when the project lead is about to make an expensive mistake — name the risk, propose a lighter alternative, comply if overridden. When the project lead pushes back on you with frustration about scope or process: simplify, don't justify. Loud feedback resets; soft feedback compounds.
