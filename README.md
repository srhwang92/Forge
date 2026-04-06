# Forge

A Claude Code development infrastructure for solo developers and
small teams who want AI-assisted work to come closer to the quality
bar of a senior engineering team — without replacing what a real
team provides.

36 files. ~5,950 lines. 13 independent security audits (OWASP ASVS,
red team, cold review, comprehensive, verification rounds, targeted
additions review). Used for production work; not a substitute for
human review.

---

## What This Is

Forge is a system of configuration files, engineering standards, safety
guardrails, project templates, and a persistent memory architecture
that shape how Claude Code writes software. It pushes output toward
the standard of a senior engineering team — more secure, more
accessible, more thoroughly tested, more legally aware, more readable.
It does not guarantee any of that. Forge is a floor, not a ceiling,
and like any instruction-based system it reduces failure modes rather
than eliminating them. See Limitations for the honest version of what
it can't do.

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

Forge addresses these patterns through files Claude reads before and
during development, a memory architecture designed to survive routine
context loss, and verification layers that catch common failure modes
after implementation. It does not eliminate any of these problems —
AI is still AI — but it makes them meaningfully less frequent and
gives you external checkpoints where you can catch what slipped
through. See Limitations for what's inherent to AI-assisted
development and cannot be fixed by instruction files alone.

## File Structure

**Global (`~/.claude/`)** — shipped with Forge, edit to customize:

- `CLAUDE.md` — workflow, verification protocol, compaction rules
- `GUARDRAILS.md` — always-active security rules
- `rules/` (8 files) — auto-loaded on matching file patterns: code
  voice, vibe-coding discipline, frontend/backend standards, git
  conventions, Context7 usage, state files reference, design workflow
- `templates/` (12 files) — blanks for per-project files: interview,
  project setup, spec, design, registry, decisions, invariants,
  status, plan, phase snapshot, handoff, project CLAUDE.md
- `templates/guardrails/` (13 files) — industry/project-type templates
  selected at setup: `ai-features`, `saas-webapp`, `ecommerce-shopify`,
  `fintech-lending`, `healthcare-hipaa`, `ml-ai-agents`, `mobile-app`,
  `dashboard-internal-tool`, `automation-workflows`, `cms-admin`,
  `pwa-service-worker`, `electron-desktop`, `static-marketing-site`

**Per-project (generated at setup)** — lives in the project root:

- `CLAUDE.md` — stack, commands, ceremony level, project conventions
- `GUARDRAILS.md` — combined industry-specific safety rules
- `SPEC.md`, `DESIGN.md` — architecture and design tokens
- `REGISTRY.md`, `DECISIONS.md`, `INVARIANTS.md` — the memory system
- `STATUS.md`, `PLAN.md` — active work tracking
- `HANDOFF.md` — human-readable overview + maintenance (at ship milestone)
- `tests/canaries/` — protected invariant tests
- `.claude/logs/` — append-only session logs (gitignored)
- `.claude/snapshots/` — phase boundary records
- `.claude/in-progress.md` — mid-refactor scratchpad (gitignored, optional)

## The Memory System

One of the biggest problems in AI-assisted development:
**context degradation after compaction.** Claude forgets what exists,
why decisions were made, and what must remain true. Summaries and
logs are lossy — they capture what happened but not the verified
state of the system.

Forge reduces (not eliminates) this with 5 mechanisms:

**REGISTRY.md** — Living index of every component, endpoint, utility,
and table. One line each with lifecycle status, test coverage, and
rollback points. After compaction, Claude reads the registry first to
minimize rediscovery and reduce hallucinated component names and paths.
Claude can still hallucinate things that aren't in the registry; the
registry just makes rediscovery cheap when it happens.

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
without RLS, a well-written canary catches it. Runs with the normal
test suite. The canary is only as good as the test that was written —
see Limitations for why AI-written canaries share the AI's blind spots.

**Phase snapshots** — Records at phase boundaries capturing what was
built, verification evidence, decisions made, and issues deferred. The
summary (~10 lines) is read every session. Detail is read on demand.

### Tiered Reconstruction

After compaction or context loss, Claude rebuilds context in a defined
order. The protocol is deterministic; the interpretation Claude derives
from the files is not.

- **Tier 1 (always):** Config → guardrails → spec → registry → invariants
  → decisions TOC → latest snapshot → status/plan
- **Tier 2 (on demand):** Decisions domain section → registry detail →
  verification evidence
- **Tier 3 (implementation):** Source code via registry paths + imports

## The Guardrail System

GUARDRAILS.md is Forge's largest file (~1,100 lines) and the canonical
source of safety, security, and quality rules. It's always loaded into
Claude's context regardless of ceremony level. Industry templates in
`templates/guardrails/` extend it per project — combined at setup into
a project-level GUARDRAILS.md that adds to (never overrides) the
global file.

Rules are grouped by domain:

- **Code safety** — hallucination prevention, known-fragile domains
  (float-for-money, timezones, RLS, IDOR), test integrity and canary
  quality, dependency rules, error loop prevention
- **Data & privacy** — PII handling, data classification, privacy
  regulations (GDPR, CCPA, PIPEDA), client confidentiality, AI data
  flows (what can and can't be sent to LLM APIs), intellectual property
- **Security** — auth lifecycle, session hardening, threat modeling,
  production environment protection, CI/CD enforcement (mechanical
  checks that don't depend on AI compliance)
- **Operational discipline** — human-approval gates, scope protection,
  git safety, file protection, quality drift prevention, token and
  context discipline
- **User-facing** — content claims (no invented testimonials or
  statistics), design system protection, business logic and
  algorithmic fairness, performance and accessibility floors
- **Governance** — escalation levels, in-session immutability,
  conflict resolution hierarchy (Data Protection > Security >
  Production > Client Confidentiality > everything else)

A few design choices worth knowing about:

- **Always-active sections** apply at every ceremony level including
  `minimal` — hallucination prevention, data/PII, security standards,
  client confidentiality, business logic safety, IP, git safety, file
  protection. These aren't dialable.
- **Deliberate duplication.** Critical rules (IDOR prevention,
  float-for-money, RLS enforcement) appear in GUARDRAILS.md AND in
  rules files AND in industry templates. The redundancy is intentional:
  these rules must stay visible regardless of which files Claude has
  loaded. If wording differs across files, GUARDRAILS.md is canonical.
- **Voluntary compliance, mechanical backstop.** Forge cannot prevent
  Claude from violating a rule. The CI/CD Enforcement section specifies
  mechanical checks (type checking, linting, secret scanning, SAST)
  that don't depend on Claude following instructions.
- **Bypass discipline.** The highest-severity guardrails (Data
  Protection, Security, Production Environment) require written risk
  acceptance in STATUS.md even when the project lead approves a bypass.

You don't need to read GUARDRAILS.md end-to-end. Claude loads it; you
review what Claude does. Open it when you want to understand why
Claude pushed back on something, or when adding project-specific rules.

## Ceremony Levels

| Level | Verification | Gates | Playwright | Use for |
|-------|-------------|-------|------------|---------|
| `standard` | Layer 1 + Layer 2 on non-trivial | All | UI tasks | Production work (default) |
| `light` | Layer 1 only | Security only | Skip | Prototypes, internal tools, marketing sites |
| `minimal` | Tests only | Security + destructive | Skip | Scripts |

Security guardrails apply regardless of ceremony level.

For genuinely high-stakes work (regulated industries, money/health
data), run cross-model adversarial review at branch boundaries —
see "Optional: Cross-Model Adversarial Review" below.

## Optional: Cross-Model Adversarial Review

For genuinely high-stakes work (regulated industries, money/health
data, high-traffic public apps), run a second model family against
the diff at branch boundaries — not per-task. The goal is a different
set of blind spots, not redundant review.

**When to run:** before merging a feature branch to main, after all
standard Forge checks pass. Not per-task. Not at compaction. Branch
boundaries only.

**How:**
- **With Codex plugin:** `/codex:adversarial-review --base main`
- **Without Codex:** dispatch a fresh subagent with the full diff and
  an adversarial framing prompt: "This is a security-critical change.
  Find what's wrong. Be uncharitable. Do not assume the author
  considered edge cases."
- **With a second LLM provider:** run the diff through ChatGPT o3 or
  Gemini 2.5 Pro with the same framing and compare findings.

**What to look for:** design flaws, cross-cutting regressions,
assumption mismatches, missing edge cases in money/auth/data flows,
test tautology (tests that pass because they describe the code
rather than the spec).

**Why it's not a layer.** In earlier Forge versions this was called
"Layer 3" and described as part of the standard verification protocol.
That framing was aspirational — most projects never ran it
consistently. Treating it as optional branch-boundary review is
honest about what actually happens and keeps the core protocol
tight. Run it when the stakes justify it.

## Minimum Viable Forge

Forge has 36 files and a lot of ceremony. You don't have to use all
of it to get value. If you want to start fast, use this subset:

- **`GUARDRAILS.md`** — the security content is the highest-value
  part of Forge. Load it and the relevant industry template for your
  project.
- **`rules/code-voice.md`** + **`rules/vibe-coding-discipline.md`** —
  the "how code reads" and "pushback / dead code / patterns" rules.
  Small files, high leverage.
- **Memory architecture** — REGISTRY.md, DECISIONS.md, INVARIANTS.md,
  and `tests/canaries/`. This is the post-compaction survival system.
  Without it, Claude tends to hallucinate paths and re-derive decisions
  from scratch after compactions.
- **The interview** (`templates/interview.md`) — 7 questions at
  project start. Cheap, high-value.
- **Layer 1 self-check** — the mechanical "did the tests pass, does
  the output match the plan" checks. Every task.

**Skip until you need them:** ceremony level distinctions, phase
snapshots, health checks, Layer 2 subagent review, project setup
apparatus, DESIGN.md workflow (unless doing heavy UI work), HANDOFF.md
(only at ship milestone), cross-model adversarial review.

The minimum viable version is roughly 8 files instead of 36. You
can always add more ceremony when you hit a problem the minimal
version doesn't solve. Don't adopt rules you aren't going to follow —
that's how governance systems become theater.

## Installation

1. Copy the `~/.claude/` directory structure to your machine
2. Configure `CLAUDE.md` header for your platform (Windows/macOS/Linux)
3. Configure `GUARDRAILS.md` privacy jurisdiction for your location
4. Install recommended MCPs (see Dependencies below)
5. Start Claude Code in any project — Forge loads automatically

## Dependencies: Skills, MCPs, and Tools

Forge references external tools throughout its files. None are hard
requirements — the system degrades gracefully when tools are unavailable.
Install what fits your workflow.

### MCP Servers

| MCP | Used for | Status | Fallback without it |
|-----|----------|--------|---------------------|
| **Context7** | Live documentation lookups for library APIs | Recommended | Web search or reading `node_modules/` docs |
| **Playwright** | Layer 1 UI verification, accessibility audits, E2E testing, Shopify theme testing | Recommended | Manual browser testing; skip automated a11y/screenshot checks |
| **Figma** | Extracting design tokens from Figma files for DESIGN.md | Optional | Manually populate DESIGN.md tokens with `[VERIFY]` markers |
| **Stitch** | Design-to-code asset pipeline | Optional | Manual asset export |

### Claude Code Skills/Plugins

| Skill | Used for | Status | Fallback without it |
|-------|----------|--------|---------------------|
| **Superpowers** | Automated build workflow (Brainstorm → Spec → Plan → TDD → Dev → Review) | Recommended | Use the manual workflow: Explore → Plan → Code → Commit |
| **ui-ux-pro-max** | Design system generation when no Figma file exists | Optional | Manually create DESIGN.md from brand guidelines |
| **frontend-design** | Creative direction, anti-generic-AI-aesthetic during implementation | Optional | Follow DESIGN.md tokens manually |
| **Codex adversarial review** | Optional cross-model design challenge at branch boundaries (`/codex:adversarial-review`) | Optional | Dispatch a subagent with the diff |
| **feature-dev** | Alternative lightweight workflow to Superpowers | Optional | Use Superpowers or manual workflow |

### Vercel Agent Skills

| Skill | Used for | Status |
|-------|----------|--------|
| `web-design-guidelines` | Accessibility and compliance audit | Optional |
| `react-best-practices` | Performance optimization guidance | Optional |
| `composition-patterns` | Component architecture patterns | Optional |

All Vercel skills are supplementary quality layers. Forge works
fully without them.

### Skill Priority (when multiple skills conflict)

CLAUDE.md defines a priority order: DESIGN.md tokens (1) > ui-ux-pro-max
(2) > frontend-design (3) > Vercel skills (4-6). Lower number wins.
If a skill isn't installed, it's simply skipped in the priority chain.

## Customization

- **Platform:** Edit CLAUDE.md header for your OS and shell
- **Jurisdiction:** Edit GUARDRAILS.md privacy section for your regulations
- **Ceremony:** Set per-project in the generated project CLAUDE.md
- **Industry:** Select from 13 guardrail templates during project setup
- **Stack:** Captured in project CLAUDE.md during the interview

## How to Use

Forge loads automatically once installed — there is no command to run.
When you open Claude Code in a project, Claude reads the global
`~/.claude/` files and any project-level files that exist.

**Forge is the outer loop, not the build workflow itself.** Forge
handles project setup, memory, verification, and the rules Claude
follows while building. The actual per-feature build sequence
(brainstorm → spec → plan → TDD → implement → review) is provided by
your build workflow plugin — Superpowers, or whichever skill stack
you use. The two layer: Forge sets up the project and the rules
once, then your build workflow runs inside those rules for every
feature. You don't choose between them; you run Forge around your
build workflow.

### Starting a new project

**Before opening Claude Code:**

1. Set up the folder. **Greenfield:** `mkdir` + `git init`. **From a
   template** (Vercel starter, Dawn theme, shadcn template, Railway
   starter, etc.): clone or download it into the folder, run the
   install command, verify it builds and runs locally. Don't make
   Claude debug template setup before it knows what the project is.
2. Have a one-sentence description of the project ready.

**Open Claude Code and tell it what you're doing in one message:**

> *"I'm starting a new project [from a Vercel SaaS template I've
> already loaded / from scratch]. The project will be [one-sentence
> purpose]. Please run the Forge interview first, then handle project
> setup."*

The "run the Forge interview first" instruction is technically
redundant — global CLAUDE.md already requires it for any non-trivial
work — but saying it explicitly prevents Claude from jumping straight
to feature implementation before scaffolding exists. Claude will then:

1. **Run the interview** — 7 grouped questions covering purpose,
   stack, data sensitivity, AI features, scope, existing context, and
   design. For template projects, Claude pre-fills stack answers from
   `package.json` and asks you to confirm. "You decide" answers
   trigger pushback with concrete options.
2. **Propose a ceremony level** from the Ceremony Floors table.
   Regulated floors are mandatory.
3. **For template projects:** scan the file structure (chunked across
   subagents if 50+ files) to populate REGISTRY.md, and seed
   DECISIONS.md with the template's baked-in choices (auth provider,
   ORM, styling system, deployment target) marked as "Template default."
4. **Create the project scaffolding** — project CLAUDE.md, GUARDRAILS.md
   (combined from industry templates), SPEC.md, REGISTRY.md,
   DECISIONS.md, INVARIANTS.md, STATUS.md, PLAN.md.
5. **Ask 3-5 clarifying questions** from scanning your answers against
   the spec — and against the template's existing schema/auth/routes
   for template projects. Answers go into DECISIONS.md.

Full setup takes 15-30 minutes of back-and-forth. Commit the scaffold.
Begin Phase 1 of PLAN.md — at which point your build workflow plugin
takes over for the first feature.

**If Claude skips the interview** and jumps to brainstorming or
implementation, stop it: *"Wait — run the project interview from
`templates/interview.md` first. We're at project setup, not feature
building."*

**Existing codebase, not a template?** Same flow. Tell Claude you
want to adopt Forge on an existing project. It runs an onboarding
scan to populate REGISTRY.md from the filesystem before the interview,
then runs the interview with as much pre-filled as possible.

### A typical task

Once project setup is done and you're in Phase 1, this is the per-task
loop. Your build workflow plugin (Superpowers, etc.) drives the
brainstorm → spec → plan → TDD sequence within this loop; Forge
governs the verification layers around it.

When you ask Claude to do something (feature, bug fix, refactor):

1. **Explore** — reads REGISTRY.md dependents, greps imports,
   identifies affected files, reports back.
2. **Plan** — proposes an approach with acceptance criteria. Claude
   should stop here for your approval before implementation begins
   (Tier A gate). If Claude skips straight to implementation on a
   non-trivial task, remind it to present the plan first.
3. **Implement** — writes code, running tests as it goes.
4. **Verify** — Layer 1 self-check (tests, lints, build, axe-core on
   UI, REGISTRY update) followed by Layer 2 subagent review on
   non-trivial tasks at `standard` ceremony.
5. **Commit** — proposes a commit message following your git conventions.

Claude applies the verification layers based on the ceremony level in
your project CLAUDE.md, but the rules are voluntary — it can skip
steps, especially under context pressure. If a Layer 1 check was
obviously missed (no test run, no REGISTRY update on a new component),
tell Claude to complete it before proceeding.

### Recovering from compaction

When Claude hits the context limit or you run `/compact`, Forge's
tiered reconstruction is designed to run on the next turn. In
practice Claude usually reads the Tier 1 files correctly; if it seems
to be working from memory instead, ask "did you read REGISTRY.md and
DECISIONS.md?" directly.

- **Tier 1** (always) reads project CLAUDE.md, GUARDRAILS.md, SPEC.md,
  REGISTRY.md, INVARIANTS.md, DECISIONS.md TOC, latest phase snapshot,
  STATUS.md, PLAN.md, and latest log.
- **Tier 2** loads specific DECISIONS.md domains or REGISTRY details
  on demand.
- **Tier 3** reads source files as the task requires.

**Mid-refactor compaction:** Claude reads `.claude/in-progress.md` if
it exists — a scratchpad with what was done, what's next, and expected
transient test failures. Don't panic-revert failing tests before
checking it. The file is only maintained for refactors touching 5+
files.

### Phase boundaries and shipping

Phases are your choice of cadence (typically a logical milestone like
"auth complete"). At phase boundaries, ask Claude to create a phase
snapshot: what was built, decisions made, invariants added, known
issues deferred.

At the ship/stable milestone, ask Claude to generate **`HANDOFF.md`** —
a human-readable overview covering architecture, reading order, local
dev setup, product context, maintenance info (dependencies, secret
rotation, backups), and the "what NOT to change without reading first"
index.

On long-running projects, run a monthly health check — dependency
audits, bundle size, dead code, canary status, and (for AI projects)
prompt regression and cost trends.

### When you're not sure

- **Update PLAN.md or just tell Claude?** Update PLAN.md. Verbal
  context doesn't persist across sessions; PLAN.md does.
- **Manually edit REGISTRY.md?** No. Claude maintains it. If you see
  staleness, tell Claude to reconcile.
- **Do I have to follow every rule?** No — see Minimum Viable Forge
  above.
- **Claude contradicts DECISIONS.md?** Tell Claude to follow
  DECISIONS.md. If the decision is genuinely wrong, mark it
  `~~[SUPERSEDED]~~` and add the new entry.
- **Skip a rule just this once?** Say so explicitly and add an inline
  `// APPROVED: [reason]` comment. Layer 2 won't flag approved
  workarounds.

## Recommended Starter Templates

When starting from a template, pair it with the right guardrail
templates. This table covers common combinations — any starter works.

| Project Type | Recommended Starters | Guardrail Templates |
|---|---|---|
| SaaS / Web App | [Vercel SaaS templates](https://vercel.com/templates), [shadcn templates](https://ui.shadcn.com) | saas-webapp + your industry template |
| Ecommerce (Shopify) | [Dawn theme](https://github.com/Shopify/dawn) | ecommerce-shopify |
| Mobile (React Native) | Expo starter (`create-expo-app`) | mobile-app + your industry template |
| Desktop (Electron) | Electron Forge, electron-vite | electron-desktop |
| PWA | Vite PWA, Serwist starter | pwa-service-worker + saas-webapp |
| CMS-backed site | Payload CMS starter, Strapi starter | cms-admin + static-marketing-site |
| Dashboard / Internal | [Vercel admin templates](https://vercel.com/templates), shadcn dashboard | dashboard-internal-tool |
| API / Backend | Railway templates, Hono starter | automation-workflows |
| ML / AI Agent | LangChain starter, Mastra starter | ml-ai-agents |
| Static / Marketing | Next.js static, Astro starter | static-marketing-site |
| **+ AI Features** (chat, RAG, search, document Q&A) | Vercel AI SDK, LangChain.js, LlamaIndex | **ai-features** (additive to primary template) |

*Last reviewed: April 2026. Links point to template galleries, not
specific versions. When using a template not listed here, select
guardrail templates based on the interview answers.*

*The **ai-features** template is additive — apply it alongside your
primary template when adding AI chat, RAG, or AI-powered features to
an existing webapp, dashboard, or CMS (e.g., saas-webapp + ai-features,
dashboard-internal-tool + ai-features). For projects that ARE AI
systems rather than apps with AI features, use ml-ai-agents instead.*

## Additional Optional Tools

| Tool | What it supplements | Fallback without it |
|---|---|---|
| **Supermemory MCP** | Reduces manual REGISTRY.md/DECISIONS.md maintenance by automatically extracting facts from conversations. Cross-session recall without full Tier 1 reconstruction. | Manual markdown files (Forge's default). Works fully without Supermemory. |

Supermemory is NOT required. Forge's markdown-based memory system is
self-contained, git-tracked, and works offline. Supermemory is an
optional enhancement that reduces the manual maintenance burden
identified in the Limitations section.

## Limitations

Forge governs AI behavior through instructions, not architecture.
That creates two kinds of limits: things Forge chose not to solve
(listed with external mitigations), and things inherent to AI-assisted
development that no instruction file can eliminate. This section is
the honest floor of what Forge can and cannot do.

### Forge-specific limits

| Limit | Mitigation |
|---|---|
| **Memory overhead scales** — REGISTRY/DECISIONS/INVARIANTS need updates every task. 6+ month projects with 200+ components get expensive. | Memory File Scaling in `rules/state-files-reference.md`. Archive superseded decisions; split REGISTRY by module at 200 entries. |
| **Depth varies by domain** — Web SaaS coverage is deep; mobile distribution, desktop auto-update, CRDT selection, content moderation are covered but shallower. | Human expertise for domain-specific choices (offline conflict strategy, SaMD classification, moderation pipelines). |
| **Not designed for large teams** — REGISTRY.md and DECISIONS.md become merge-conflict magnets with 3+ concurrent devs on the same codebase. | Per-feature-branch scoped files merging at phase gates, or commit-at-breakpoints discipline. Neither is a complete fix. |
| **Health check fatigue** — monthly health checks assume one project. Solo devs running 5+ projects see adherence decay after two months. | Prioritize by criticality (production > billable > internal > experiments). Document skipped checks in STATUS.md. |
| **Tier 1 reconstruction scales poorly** — the 8K-token budget is realistic for young projects; mature projects (200+ REGISTRY entries) need 15-25K. | Plan session budgets accordingly. Scaling rules reduce but don't eliminate. |

### Inherent AI limits (external mechanisms required)

| Limit | External mitigation |
|---|---|
| **Self-review is circular** — Layer 2 shares the main agent's model family and blind spots. | Human code review on every PR; SAST/DAST in CI; optional cross-model adversarial review. |
| **Rules are voluntary** — no architectural enforcement. Claude can skip steps, hallucinate, log PII. | CI/CD gates for type checking, linting, secret scanning, SAST. |
| **Uncertainty calibration is unreliable** — confident on uncertain things, hedges on known ones. | Known-answer test cases, canary tests, verification commands. |
| **Context is lossy across compactions** — files capture decisions, not the reasoning that led to them. | Detailed DECISIONS.md entries; phase snapshots; `in-progress.md` for mid-task refactors. |
| **Pattern replication is implicit** — Claude generates from patterns in context without consciously deciding to copy. | Layer 2 review, linting, style enforcement in CI. |
| **Hallucination is mitigated, not prevented** — Claude cannot reliably detect when it's hallucinating. | Type checking, `npm install` failures, Context7 verification. |
| **Cross-project pattern leakage** — context isolation is per-session; implicit assumptions transfer. | Separate sessions for separate clients, especially across regulated/non-regulated work. |
| **Security judgment is context-dependent** — follows categorical rules well but struggles with "is this RLS policy correct for this threat model?" | Penetration testing, security-focused human review. |
| **AI-written tests share AI blind spots** — the same model writes the code and the tests validating it. | Human review of test cases against business requirements. |
| **Approval gates depend on human discipline** — under time pressure, Tier A becomes a rubber stamp. | CI/CD gates that block merges regardless of human approval. |
| **Non-Claude code bypasses Forge** — ChatGPT/Gemini/Copilot generations don't know Forge's rules. | Same CI/CD checks apply to any contribution. At `light` ceremony, this is the primary risk. |
| **High-stakes AI needs more than Forge** — no rule set guarantees prompt injection resistance. | Model-level guards (Llama Guard, Azure Content Safety), red-team engagement, external audit. |

**Guardrails ≠ compliance.** The industry templates (fintech,
healthcare, ecommerce) encode regulatory awareness into development
instructions. They reduce risk. They are NOT a compliance program.
Regulated work requires a compliance officer, external penetration
testing, formal access control documentation, and professional audit
of the application output — not just the instructions that generated it.

## What You Must Provide

Forge governs how AI writes code. These are things it cannot do that
you handle through external tools, processes, or expertise.

### For every project

- **API cost budget.** Forge is not free to run. Typical solo-dev
  ranges per active project: `minimal` ~$20-50/mo, `light` ~$50-150/mo,
  `standard` ~$150-500/mo. Optional cross-model adversarial review
  adds +$30-100/mo per branch. Actual costs vary with compaction
  frequency and subagent usage. Set spending alerts; 3 active
  `standard` projects easily exceed $1,000/mo.
- **Human code review on PRs.** Layer 2 shares model blind spots
  with the main agent. Every PR to production needs a human reviewer.
  **Focus review on business logic correctness** — does the code do
  what the business needs, not just what it was instructed to do?
  AI generates syntactically perfect, well-tested code that solves
  the wrong problem more often than it generates buggy code. This is
  the single most important thing Forge cannot replace.
- **CI/CD pipeline.** Forge tells Claude what to check (types, lint,
  tests, secrets, SAST, dependencies, build) but cannot run CI itself.
  Without it, every behavioral rule is unverifiable.
- **Secret rotation discipline.** Forge documents rotation schedules
  in HANDOFF.md but cannot enforce them. Use a secrets manager with
  auto-rotation (AWS Secrets Manager, Vault).
- **Environment parity.** dev/staging/prod should use the same DB
  engine and major versions. Verify manually or script a parity check
  in CI.

### For commercial/production projects

- **SAST** (Semgrep, Snyk Code) running in CI — the mechanical
  backstop for security rules Claude can violate under context pressure.
  Catches SQL injection, XSS, insecure deserialization, hardcoded
  secrets missed by grep-based scanning.
- **Dependency security monitoring.** `npm audit` catches CVEs but
  not compromised packages, dependency confusion, or malicious
  maintainer takeover. Socket.dev, Snyk, or Dependabot for continuous
  monitoring.
- **DNS and infrastructure management.** Forge governs application
  code, not infrastructure. You manage DNS, deployment config, backup
  verification, SSL renewal, CDN config.
- **External penetration testing.** Annually at minimum for production
  SaaS. Professional pen testers find what behavioral rules miss.
- **Disaster recovery plan.** Tested restoration procedures, RTO/RPO
  targets, communication plan for infrastructure failures.
- **App store submission pipeline** (mobile). Provisioning profiles,
  signing certs, privacy policies, screenshot metadata, Connect/Console
  configuration.
- **NTP synchronization.** Webhook timestamp verification assumes
  synchronized clocks. Ensure NTP on all servers.

### For regulated industries (fintech, healthcare)

- **Compliance officer** — a human who knows the applicable
  regulations (ECOA, TILA, HIPAA, GLBA, GDPR) and reviews the
  application output. Templates encode awareness, not expertise.
- **Formal access control documentation** — documented policies,
  provisioning/deprovisioning, audit trails satisfying SOC 2, HIPAA
  administrative safeguards, or equivalent.
- **Audit log integrity** — cryptographic signing, tamper-evident
  storage, SIEM integration, retention enforcement, periodic review.
  Forge logs events; infrastructure guarantees integrity.
- **Incident response plan** — external notification, customer
  communication templates, regulatory timelines (72hr PIPEDA, 60d
  HIPAA), chain of command beyond "the developer."
- **Independent security audit of the output** — for regulated work,
  an external firm reviews the actual application code, not the
  governance system that generated it.