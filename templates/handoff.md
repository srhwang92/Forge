# Project Handoff — [PROJECT NAME]

> Human-readable project overview AND production maintenance guide
> for developers taking over this codebase. Unlike other Forge files
> (designed for Claude), this file is written for humans. Generate at
> ship or stable milestone. Update on major architectural changes.

---

## Architecture Overview

[Describe how the system works at a high level. Entry points, data
flow, key integration points. Include a simple text diagram if helpful:

```
Browser → Next.js App Router → Server Actions → Supabase (RLS)
                                    ↓
                              Stripe Webhooks → Edge Functions
```
]

## Start Here — Reading Order

[List 5-10 files a new developer should read first, in order, with
one-line descriptions of what they'll learn from each:]

1. `[file]` — [what you learn]
2. `[file]` — [what you learn]

## Local Development Setup

**Prerequisites:** [Node version, package manager, required services]

**Required services:**
- [Database: how to start, seed data location]
- [Auth: provider setup, test credentials]
- [Storage: local S3 alternative or config]
- [Other services: SMTP, Redis, etc.]

**Environment variables:**
- Copy `.env.example` to `.env.local`
- Required values and where to obtain them:
  - `[VAR_NAME]` — [where to get it]

**First run:**
```
[exact commands to go from fresh clone to running app]
```

**Verify it works:** [what URL to visit, what you should see]

## Product Context

[Information that influenced technical decisions but isn't captured
in SPEC.md or DECISIONS.md:]

- **Target users:** [who they are, how they use the product]
- **Key UX decisions:** [why the UI works the way it does]
- **Client/stakeholder preferences:** [layout, workflow, terminology]
- **User research findings:** [what was tested, what was learned]

## Forge Framework Notes

This project uses Forge, an AI development governance system. If you
use Claude Code, it reads the project files automatically. If you
don't, here's what matters:

- **`INVARIANTS.md`** lists things that must always be true. Each
  entry references a canary test in `tests/canaries/`. These tests
  are protected — do not delete without understanding what they guard.
- **`REGISTRY.md`** is a component inventory. If you add, rename, or
  delete components, update it (or let Claude Code do it).
- **`DECISIONS.md`** captures why things are built the way they are.
  Check it before changing architecture — there may be a documented
  reason for the current approach. **Watch for `~~[SUPERSEDED]~~`
  entries** — these are preserved for history but NOT current
  guidance. Skipping this distinction is a common handoff mistake.
- **`GUARDRAILS.md`** contains security rules. Read it before touching
  auth, payments, or data handling.

## What NOT to Change Without Reading First

Some configuration values, thresholds, and constants look arbitrary
but carry load-bearing meaning. Changing them has non-obvious blast
radius. Before touching any of these, read the referenced file.

[List load-bearing tunables specific to this project. Examples:]

- **RAG chunking parameters** (`lib/rag/ingest.ts`): changing
  `chunkSize` or `overlap` invalidates all existing embeddings.
  Re-embedding is expensive and causes search quality regression
  until complete. → Read `DECISIONS.md § Retrieval`
- **Moderation thresholds** (`lib/moderation/config.ts`): tuned to
  balance false positive rate against moderator workload. → Read
  `DECISIONS.md § Moderation`
- **AI system prompts** (`prompts/` directory): every prompt has a
  regression test. Changes require running the test suite and updating
  the version in REGISTRY.md AI Prompts table. → Read
  `GUARDRAILS.md § AI Data Flows`
- **Rate limit configuration** (`middleware.ts`): per-user and per-
  tenant limits are set based on billing tier and cost analysis.
  → Read `DECISIONS.md § Rate Limiting`
- **Pinned model versions** (AI provider configs): pinned to specific
  versions for deterministic behavior. Unpinning causes silent quality
  drift. → Read `DECISIONS.md § Model Versioning`
- **Canary tests** (`tests/canaries/`): protected, enforce invariants.
  → Read `INVARIANTS.md`

Items not on this list can be changed normally. Items on this list
require reading the referenced file first, and for AI/model/prompt
changes, running the regression suite before committing.

## Maintenance

### Dependencies

[Critical dependencies and upgrade risk notes]
- [package] — [version constraint, upgrade considerations]
- [Major framework upgrades that require coordinated changes]

### Secret Rotation Schedule

| Secret | Rotation cadence | Next due | Where stored |
|---|---|---|---|
| API keys (third-party) | 90 days | [date] | [vault/env] |
| Database credentials | 180 days | [date] | [vault/env] |
| Signing keys (JWT, webhooks) | 1 year | [date] | [vault/env] |
| OAuth client secrets | 180 days | [date] | [vault/env] |

### Backup & Restore

- **Database backups:** [cadence, retention, location]
- **Restore procedure:** [tested? when last? exact steps]
- **Vector store backups** (if RAG): [cadence, location]
- **File storage backups:** [cadence, location]

### Health Check Cadence

- **Monthly:** dependency audit, cost review, security review
- **Quarterly:** full backup restoration test, secret rotation check
- **On deploy:** subdomain resolution check, bundle size check

### Fragile Areas

[Components where changes have caused incidents in the past. Treat
with extra care.]
- [area] — [what happened, what to watch for]

### Questions to Answer Before Major Changes

- [Questions whose answers affect safe modification decisions]

## Known Rough Edges

[3-5 areas of the codebase that need extra care. Not explanations of
why they're clever — just what to watch out for:]

1. [area] — [what to be careful about]
