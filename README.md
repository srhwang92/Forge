# Forge

A Claude Code development infrastructure that replaces a human engineering
team with a single developer + AI — without sacrificing quality, security,
or professionalism.

25 files. ~3,800 lines. 700+ adversarial audit passes. Built for production.

---

## What This Is

Forge is a system of configuration files, engineering standards, safety
guardrails, project templates, and a persistent memory architecture that
govern how Claude Code writes software. It ensures every line of output
meets the standard of a senior engineering team — secure, accessible,
well-tested, legally compliant, and indistinguishable from human-written
code.

Built for multi-client development across websites, webapps, mobile apps,
ecommerce, SaaS, dashboards, ML/AI, agents, and automation — on Supabase,
Vercel, Shopify, Railway, and beyond.

## Why This Exists

AI-assisted development without guardrails produces code that is:

- **Insecure** — hardcoded secrets, SQL injection, missing auth checks,
  client-exposed server keys
- **Legally risky** — PII in logs, missing consent, float-for-money,
  discriminatory algorithms
- **Detectably artificial** — over-commented, verbose naming, uniform
  structure, no pragmatic shortcuts
- **Fragile** — happy-path only, no error boundaries, no empty states,
  no loading states
- **Amnesiac** — context loss after compaction, hallucinated components,
  forgotten decisions, compound assumptions

Forge prevents all of it through files Claude reads before writing a
single line of code, and a memory system that survives any amount of
context loss.

## File Structure

```
~/.claude/
├── CLAUDE.md                              # Workflow, verification, tiered reconstruction
├── GUARDRAILS.md                          # Safety rules, approval gates, privacy, security
├── rules/
│   ├── code-voice.md                      # Human voice + AI rigor
│   ├── vibe-coding-discipline.md          # Anti-drift, pushback, pattern propagation
│   ├── frontend-standards.md              # Tokens, CSS, a11y, perf, Playwright, Shopify
│   ├── backend-standards.md               # API, database, auth, IDOR, observability
│   ├── git-conventions.md                 # Commits, staging, IDE boundary
│   └── context7-usage.md                  # MCP query workflow, token discipline
└── templates/
    ├── project-claude.md                  # Project-level config template
    ├── spec.md                            # System design document template
    ├── design.md                          # Design system token template
    ├── registry.md                        # Codebase inventory template
    ├── decisions.md                       # Domain-organized reasoning template
    ├── invariants.md                      # System constraints template
    ├── phase-snapshot.md                  # Phase boundary record template
    └── guardrails/
        ├── fintech-lending.md             # ECOA, TILA, fair lending, audit trails
        ├── saas-webapp.md                 # Multi-tenancy, billing, data portability
        ├── ecommerce-shopify.md           # Dawn rules, PCI-DSS, CASL, product safety
        ├── ml-ai-agents.md                # Bias testing, agent safety, LLM API rules
        ├── mobile-app.md                  # App store compliance, device storage, certs
        ├── healthcare-hipaa.md            # PHI encryption, BAA, breach response
        ├── dashboard-internal-tool.md     # Auth required, audit logging, RBAC
        ├── automation-workflows.md        # Idempotency, circuit breakers, data flow
        └── static-marketing-site.md       # Content claims, cookie consent, SEO
```

### Per-Project Files (Generated at Setup)

```
project-root/
├── CLAUDE.md              # Stack, commands, ceremony level, constraints
├── GUARDRAILS.md          # Combined industry-specific safety rules
├── SPEC.md                # Architecture, data model, user flows
├── DESIGN.md              # Design system tokens (UI projects)
├── REGISTRY.md            # Living codebase inventory
├── DECISIONS.md           # Domain-organized reasoning memory
├── INVARIANTS.md          # System constraints with canary references
├── STATUS.md              # Current progress + open questions
├── PLAN.md                # Active tasks with phase exit criteria
├── MAINTENANCE.md         # Post-ship knowledge transfer (at completion)
├── tests/canaries/        # Automated invariant enforcement tests
└── .claude/
    ├── logs/              # Session activity logs (gitignored)
    ├── snapshots/         # Phase boundary records
    ├── verified/          # Test output, Playwright results (evidence)
    └── snippets/          # Verified reusable patterns (optional)
```

## The Memory System

The #1 problem in AI-assisted development: **context degradation after
compaction.** Claude forgets what exists, why decisions were made, and
what must remain true. Summaries and logs are lossy — they capture what
happened but not the verified state of the system.

Forge solves this with 5 mechanisms:

**REGISTRY.md** — Living index of every component, endpoint, utility,
and table. One line each with lifecycle status, test coverage, and
rollback points. After compaction, Claude reads the registry and knows
exactly what exists without rediscovery or hallucination.

**DECISIONS.md** — Domain-organized reasoning memory. Not chronological
— grouped by system area (Auth, Payments, Data Model). Each decision
captures: what was chosen, what was rejected and why, what assumptions
must hold, and how to verify. Domain knowledge (terms, rates, rules) is
preserved at the top of each section.

**INVARIANTS.md** — Conditions that must always be true, written as
testable statements. Every invariant references a canary test. An
invariant without a canary is an unenforced claim.

**Canary tests** — Automated tests in `tests/canaries/` that verify
assumptions and constraints, not features. If a new table is added
without RLS, the canary catches it. Runs with the normal test suite.

**Phase snapshots** — Records at phase boundaries capturing what was
built, verification evidence, decisions made, and issues deferred. The
summary (~10 lines) is read every session. Detail is read on demand.

### Tiered Reconstruction

After compaction or context loss, Claude rebuilds context deterministically:

- **Tier 1 (always):** Config → guardrails → spec → registry → invariants
  → decisions TOC → latest snapshot → status/plan
- **Tier 2 (on demand):** Decisions domain section → registry detail →
  verification evidence
- **Tier 3 (implementation):** Source code via registry paths + imports

## Ceremony Levels

| Level | Verification | Gates | Playwright | Use for |
|-------|-------------|-------|------------|---------|
| `full` | All layers, every task | All | Every UI change | Regulated/client |
| `standard` | All layers, non-trivial | All | UI tasks | Most projects |
| `light` | Layer 1 only | Security only | Skip | Personal projects |
| `minimal` | Tests only | Security + destructive | Skip | Scripts |

Security guardrails apply regardless of ceremony level.

## Installation

1. Copy the `~/.claude/` directory structure to your machine
2. Configure `CLAUDE.md` header for your platform (Windows/macOS/Linux)
3. Configure `GUARDRAILS.md` privacy jurisdiction for your location
4. Start Claude Code in any project — Forge loads automatically

## Customization

- **Platform:** Edit CLAUDE.md header for your OS and shell
- **Jurisdiction:** Edit GUARDRAILS.md privacy section for your regulations
- **Ceremony:** Set per-project in the generated project CLAUDE.md
- **Industry:** Select from 9 guardrail templates during project setup
- **Stack:** Captured in project CLAUDE.md during the interview
