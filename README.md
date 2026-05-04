# Forge v2

A lightweight agentic workflow framework for Claude Code. Guides project
development with Karpathy's principles. Supplements Superpowers without
intruding on it.

## What's here

```
v2/
├── CLAUDE.md                          # Behavioral guidelines
├── GUARDRAILS.md                      # Always-active safety rules
├── README.md                          # This file
├── v1-deprecated.md                   # Reference for cleaning up v1 projects
├── templates/
│   ├── interview.md                   # 5 kickoff questions
│   ├── project-claude.md              # Project CLAUDE.md template
│   ├── context.md                     # CONTEXT.md template
│   ├── phase.md                       # PHASE.md template
│   ├── canary-header.md               # Canary test header convention
│   └── guardrails-additions/
│       ├── ai-features.md             # AI / RAG / agents
│       ├── fintech.md                 # Money / lending
│       └── healthcare.md              # PHI / HIPAA
└── skills/
    └── forge-handoff/SKILL.md         # HANDOFF.md generation at ship
```

## Installation

1. Back up `~/.claude/CLAUDE.md`, `~/.claude/GUARDRAILS.md`,
   `~/.claude/rules/`, `~/.claude/templates/`.
2. Replace `~/.claude/CLAUDE.md` with `v2/CLAUDE.md`.
3. Replace `~/.claude/GUARDRAILS.md` with `v2/GUARDRAILS.md`.
4. Replace `~/.claude/templates/` with `v2/templates/`.
5. Copy `v2/skills/forge-handoff/` to `~/.claude/skills/`.
6. Delete `~/.claude/rules/` (its content is folded into the new
   CLAUDE.md and GUARDRAILS.md).

## Existing v1 projects

When you open an existing v1 project, ask Claude to clean it up:
*"This is a v1 project — find what's deprecated and switch it to v2."*

Claude reads [v1-deprecated.md](v1-deprecated.md) (which lists what v1
had and what v2 disposes of) and proposes the changes for approval.

## Generating HANDOFF.md at ship

The `forge-handoff` skill generates HANDOFF.md from `CONTEXT.md` +
project `CLAUDE.md` at ship/stable milestones. Invoke when shipping.

## Design philosophy

> "Prevent mistakes, don't enforce perfection."

Every cut from v1 → v2 was driven by a documented failure mode:

- v1 ceremony pulled personal projects toward production overhead
  → v2 default is lightweight; industry templates extend only when triggered
- v1 process generated process
  → v2 has no proactive lifecycle suggestions
- v1 self-review produced typo-grade noise
  → v2 mandates adversarial framing or skip
- v1 memory accreted; never decayed
  → v2 has explicit subtract gate at phase boundaries
- v1 relied on agent discipline; agent rationalized around it
  → v2 cuts ceremony rather than adding more enforcement

If a rule isn't here, Forge doesn't enforce it.
