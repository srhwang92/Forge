# What v1 had that v2 doesn't

Reference for cleaning up an existing v1 project. When the user says
"this is a v1 project, switch it to v2", check the project against
this list and remove what's deprecated.

## Files v2 doesn't use

| v1 file | v2 disposition |
|---|---|
| `REGISTRY.md` | Delete. Source code is the registry. |
| `INVARIANTS.md` | Delete. Canary header comments document what they guard. |
| `DECISIONS.md` | Delete. Compress key decisions into `CONTEXT.md`. |
| `STATUS.md` | Delete. Current state goes in `CONTEXT.md`; tasks in `PHASE.md`. |
| `PLAN.md` | Delete. Active phase goes in `PHASE.md`; archive prior phases. |
| `SPEC.md` | Leave in place. v2 makes spec opt-in but doesn't remove it if present. |
| `DESIGN.md` | Leave in place. Anthropic's design skills produce their own outputs in `design/` if needed. |
| `HANDOFF.md` | Leave in place. `forge-handoff` regenerates from `CONTEXT.md` at ship. |
| `.claude/snapshots/*.md` | Archive to `.claude/archive/snapshots/`. The phase-end log entry replaces snapshots going forward. |
| `tests/canaries/` | Keep. Update each canary's header comment to the v2 format (see `templates/canary-header.md`). |

## Files v2 introduces

| File | Purpose |
|---|---|
| `CONTEXT.md` | Architecture sketch, key decisions, current state, open questions. |
| `PHASE.md` | Current phase goal + tasks + done-when. Replaced when the phase ends. |

## Sections to strip from project `CLAUDE.md`

- Ceremony level (`standard` / `light` / `minimal`) — v2 has no ceremony levels
- Tier 1 / Tier 2 / Tier 3 reconstruction prescriptions
- Layer 1 / Layer 2 / Layer 3 verification protocols
- Workflow Overrides
- Applied Guardrail Templates list
- Anything that duplicates the global v2 `CLAUDE.md`

Keep stack, commands, project-specific quirks, "Things Claude Gets Wrong".

## Sections to strip from project `GUARDRAILS.md`

Anything duplicating the global v2 `GUARDRAILS.md`. Keep only the industry-specific additions (`ai-features`, `fintech`, `healthcare`). If nothing remains, delete the project file — the global one suffices.

## Global rules / templates removed

- 12 `rules/*.md` files — folded into global `CLAUDE.md` and `GUARDRAILS.md`
- 13 industry guardrail templates → 3 (`ai-features`, `fintech`, `healthcare`)
- 8-stage product lifecycle map — gone
- Phase-snapshot template — gone (use the phase-end log entry)
- Most state-file templates (status, plan, registry, decisions, invariants, spec, design, phase-snapshot, project-claude variants)

## Approach

When the user says "switch this to v2":
1. Read this file.
2. Read the project's current files. Identify what's deprecated.
3. Propose the changes — what to delete, what to compress, what to leave alone.
4. Get approval before any disk operations.
5. Apply.
