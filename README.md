# Forge

A Claude Code development infrastructure that replaces a human engineering team with a single developer + AI — without sacrificing quality, security, or professionalism.

21 files. 3,168 lines. 180 adversarial audit passes. Built for production.

---

## What This Is

Forge is a system of configuration files, engineering standards, safety guardrails, and project templates that govern how Claude Code writes software. It ensures every line of output meets the standard of a senior engineering team — secure, accessible, well-tested, legally compliant, and indistinguishable from human-written code.

Built for a multi-client ML consultancy operating across websites, webapps, mobile apps, ecommerce, SaaS, dashboards, ML/AI, agents, and automation — on Supabase, Vercel, Shopify, Railway, and beyond.

## Why This Exists

AI-assisted development without guardrails produces code that is:

- **Insecure** — hardcoded secrets, SQL injection, missing auth checks, client-exposed server keys
- **Legally risky** — PII in logs, missing consent, float-for-money, discriminatory algorithms
- **Detectably artificial** — over-commented, verbose naming, uniform structure, no pragmatic shortcuts
- **Fragile** — happy-path only, no error boundaries, no empty states, no loading states
- **Drifting** — spec drift over long sessions, compound errors on wrong foundations, dead code from pivots

Forge prevents all of it through files Claude reads before writing a single line of code.

## File Structure

```
~/.claude/
├── CLAUDE.md                              # Workflow, verification, project setup
├── GUARDRAILS.md                          # Safety rules, approval gates, privacy, security
├── rules/
│   ├── code-voice.md                      # Human voice + AI rigor
│   ├── vibe-coding-discipline.md          # Anti-drift, pushback, dead code hygiene
│   ├── frontend-standards.md              # Tokens, CSS, a11y, perf, components
│   ├── backend-standards.md               # API, database, auth, observability
│   ├── git-conventions.md                 # Commits, staging, Cursor boundary
│   └── context7-usage.md                  # MCP query workflow, token discipline
└── templates/
    ├── project-claude.md                  # Project-level config template
    ├── spec.md                            # System design document template
    ├── design.md                          # Design system token template
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

## How It Works

### Always Loaded (Every Session)

| File | Lines | Purpose |
|------|-------|---------|
| `CLAUDE.md` | 273 | Zero-assumptions interview, Superpowers workflow, 3-layer verification protocol, subagent delegation, project setup steps, session recovery |
| `GUARDRAILS.md` | 542 | Hallucination prevention, human-approval gates, production protection, data/PII rules, security standards, privacy law awareness, algorithmic fairness, IP protection, escalation levels |
| `code-voice.md` | 227 | Comments (why not what), concise naming, organic structure variation, pragmatic shortcuts, proportional error handling, human-readable tests and docs |
| `vibe-coding-discipline.md` | 104 | Foundation verification, pushback mandate, dead code hygiene, pattern verification, scope expansion detection |
| `frontend-standards.md` | 411 | TypeScript strict, W3C design tokens, WCAG 2.2 AA, Core Web Vitals, component architecture, forms, SEO, animation, i18n readiness, testing |
| `backend-standards.md` | 452 | Type safety, API-first design, input validation, error handling, database discipline, auth, clean architecture, DDD, background jobs, caching, observability |
| `git-conventions.md` | 82 | Conventional commits, staging discipline, Cursor/Claude Code boundary, handoff protocol |
| `context7-usage.md` | 70 | Two-step MCP workflow, decision tree (Context7 vs web search vs local source), query formatting, token discipline |

### Loaded at Project Setup (Templates)

Templates are zero-cost until read. During the initial interview, Claude identifies the project type, selects the relevant templates, and generates project-specific files:

- **Project CLAUDE.md** — stack, commands, directories, constraints, integrations
- **SPEC.md** — architecture, data model, user flows, API surface, out-of-scope
- **DESIGN.md** — typography, colors, spacing, z-index, breakpoints, motion tokens
- **Project GUARDRAILS.md** — combined from relevant guardrail templates

### The Workflow

```
Interview → Project Setup → Incremental Development → Verification → Ship
    │              │                   │                      │
    │              ├─ CLAUDE.md        │                      ├─ Layer 1: Self-check
    │              ├─ GUARDRAILS.md    │                      ├─ Layer 2: Subagent review
    │              ├─ SPEC.md          │                      └─ Layer 3: Codex adversarial
    │              └─ DESIGN.md        │
    │                                  ├─ Foundation verification
    │                                  ├─ Pushback on bad decisions
    │                                  ├─ Dead code cleanup on pivots
    │                                  └─ Pattern verification before replication
    │
    ├─ What are you building?
    ├─ What data does it handle?
    ├─ Who is the audience?
    └─ What regulations apply?
```

## What It Prevents

| Category | Examples |
|----------|----------|
| **Hallucination** | Fabricated APIs, packages, test results, security claims, version numbers |
| **Security** | XSS, SQL injection, secret exposure, custom crypto, plaintext passwords, missing auth, client-side secrets |
| **Privacy** | PII in logs, tracking before consent, missing breach notification, data residency violations |
| **Accessibility** | Missing labels, no keyboard nav, skipped heading levels, no focus management, removed aria attributes |
| **Legal** | IP infringement, false marketing claims, fake testimonials, algorithmic discrimination, unreviewed compliance text |
| **AI Code Tells** | Over-commenting, verbose naming, uniform structure, gold-plated internals, verbose test names |
| **Vibe Coding** | Spec drift, compound errors, dead code accumulation, pattern bug propagation, sycophantic compliance, scope creep |
| **Production** | Unbounded queries, wrong environment, missing backups, debug code in prod, destructive migrations |
| **Quality** | Float for money, hardcoded business rules, missing error/loading/empty states, whole-file regeneration for 3-line fixes |

## Installation

```powershell
# Clone the repo
git clone <repo-url> ~/.claude-forge

# Copy to Claude Code config directory
Copy-Item -Recurse ~/.claude-forge/CLAUDE.md ~/.claude/CLAUDE.md
Copy-Item -Recurse ~/.claude-forge/GUARDRAILS.md ~/.claude/GUARDRAILS.md
Copy-Item -Recurse ~/.claude-forge/rules ~/.claude/rules
Copy-Item -Recurse ~/.claude-forge/templates ~/.claude/templates
```

Or manually place the files to match the structure above.

## Usage

**New project:** Start a Claude Code session in the project directory. Claude will run the zero-assumptions interview, then generate project files from the templates automatically.

**Existing project:** Add a project `CLAUDE.md` to your project root with the stack, commands, and conventions. Claude reads it on session start.

**Quick task:** The system self-adjusts. A bug fix skips SPEC.md, Layer 3, and ceremony. The guardrails and standards still apply — you just don't generate project setup files for a 5-minute fix.

## Customization

**Add a new project type:** Create a new guardrail template in `templates/guardrails/`. It will be available for selection during the interview.

**Modify standards:** Edit the rules files directly. Changes take effect on the next Claude Code session.

**Project overrides:** Any rule can be overridden at the project level through the project `CLAUDE.md` workflow overrides section or project `GUARDRAILS.md`. Global guardrails can be extended but never weakened.

## Requirements

- Claude Code (Claude Max plan recommended for token headroom)
- Windows + PowerShell (commands are PowerShell-compatible)
- Cursor IDE (for git operations — branch, commit, push)

## Auditing

This system was built through iterative construction and adversarial auditing:

- Frontend standards: 5 audit passes
- Backend standards: 4 audit passes
- Guardrails: 6 audit passes + 3-pass adversarial
- Code voice: 2 audit passes
- Full system: 180 adversarial passes across 10 loops covering project types, platforms, failure modes, elite team comparison, legal/regulatory, scale, client scenarios, AI exploitation, quality perception, and end-to-end workflows
- Clean rate: 98.3% (3,168 lines, 13 findings fixed across 180 passes)

## License

Private. Not for redistribution.
