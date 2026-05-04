---
name: forge-handoff
description: Use when generating HANDOFF.md at a project's ship or stable milestone. Reads project CLAUDE.md, MEMORY.md, recent logs, and GUARDRAILS.md, asks the project lead 4-5 scoping questions for context that doesn't live in files, then produces a human-readable handoff document. Trigger on "generate the handoff", "write a handoff doc", "we're shipping", invocation by name (forge-handoff), or any mention of preparing the project for someone else to maintain.
---

# Forge Handoff

Generate `HANDOFF.md` at a ship/stable milestone. The output is for the developer taking over — content not derivable from source.

Don't run mid-phase (state will be stale on day one). If `MEMORY.md` or project `CLAUDE.md` doesn't exist, stop — they're the inputs.

## Read

Project `CLAUDE.md`, `MEMORY.md`, `PHASE.md`, latest 3 logs, `GUARDRAILS.md` (if exists), top-level `README.md`, `package.json` (or equivalent), `tests/canaries/` listing.

## Ask the project lead

- Production URL (or N/A)?
- Who's taking over?
- Maintenance commitments — secret rotation, backups, on-call?
- Load-bearing config values, thresholds, pinned versions?
- Fragile areas where past changes caused incidents?

Don't fabricate any of this — a handoff doc that invents commitments is worse than one that omits them.

## Write HANDOFF.md sections (in order)

1. **Architecture Overview** — 1-paragraph from `MEMORY.md` Architecture; text diagram if 3+ moving pieces.
2. **Reading Order** — 5-10 files a new dev reads first, in order. Mental-model files first, detail later.
3. **Local Development Setup** — prereqs, services, env vars, first-run commands, "verify it works" check.
4. **Product Context** — decisions from `MEMORY.md` Key Decisions that influenced technical choices but aren't obvious from source.
5. **Maintenance** — dependencies, secret rotation schedule, backup/restore, health check cadence, fragile areas.
6. **What NOT to Change Without Reading First** — load-bearing tunables (pinned versions, rate limits, thresholds, "Things Claude Gets Wrong" entries about specific values). For each: filename, what it controls, what to read first.
7. **Forge Framework Notes** — brief explanation of `CLAUDE.md` / `MEMORY.md` / `tests/canaries/` for devs who don't use Claude Code.
8. **Known Rough Edges** — 3-5 from "Things Claude Gets Wrong" + lead's input.

Show the draft to the project lead before committing. Product Context and Maintenance carry context only they have — they may want to edit directly.

Commit: `docs: add HANDOFF.md at [milestone]`.
