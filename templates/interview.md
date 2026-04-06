# Project Interview Script

> Read verbatim at project start or when adopting an existing codebase.
> This file is loaded only during project kickoff — not every session.
> CLAUDE.md points here; do not duplicate content.

Present all questions at once (grouped, not one at a time) so the
project lead can answer in a single pass.

## Questions

**1. Purpose & Users**
- What are we building, and why? (1-2 sentences)
- Who are the end users? (internal only, external consumers,
  specific customer segments?)
- Is this a prototype, MVP, or production system?

**2. Stack & Infrastructure**
- Preferred frontend, backend, database, hosting?
- Are you starting from a template, starter, or boilerplate?
  Which one? Have you cloned/installed it, or should I set it up?
- What are the CI/CD, monitoring, and deployment targets?

**3. Data Sensitivity**
- Does this project handle PII, financial data, health data, or
  other regulated information? What categories?
- Target geography for users? (determines applicable privacy laws:
  GDPR, CCPA, PIPEDA, etc.)
- Is this covered by HIPAA, GLBA, PCI-DSS, or similar regulations?

**4. AI Features (mandatory — no exceptions)**
- Does this project include any AI features — chat, RAG,
  AI-generated content, AI-powered search, agents, tool use,
  MCP servers, AI moderation, LLM-assisted workflows, or
  embeddings? Even planned for a future phase?
- If yes: which provider(s)? Self-hosted or API? What data will
  reach the LLM?
- If maybe / "not yet but eventually": flag for early planning.
  (Load `ai-features.md` guardrail template now — AI is never a
  late-project bolt-on.)

**5. Scope & Constraints**
- What should this NOT do? (explicit exclusions prevent drift)
- Known constraints: budget, timeline, team size, performance
  targets, accessibility requirements (WCAG 2.2 AA default)?
- Any edge cases or unusual requirements to plan for upfront?

**6. Existing Context**
- Are there existing patterns, files, or conventions to match?
  (critical for inherited codebases)
- Any known security issues, previous pen test findings, or past
  incidents? (inherited codebases — do not skip)
- What tools, libraries, or services must be used (or avoided)?

**7. Design & Brand**
- Existing design assets — Figma files, brand guidelines, design
  system, or starting fresh?
- Accessibility target (WCAG 2.2 AA is the default — flag if
  different)?

## Interview Rules

- If the project lead answers "you decide" to a project-level
  question (stack, data sensitivity, AI features, compliance),
  follow up with "I need a decision here — these shape the whole
  project. Here are the main options: [A, B, C] with trade-offs."
  Do not silently pick defaults.
- After answers are received, summarize them back in one paragraph
  before proceeding. If the summary reveals a contradiction, resolve
  it before writing code.
- **Answers from the interview go into:** `project-claude.md` (stack,
  conventions), SPEC.md (scope, constraints, flows), DECISIONS.md
  (the "why" behind each major choice).

## AI Features — Template Loading Rule

If the answer to question 4 is yes, maybe, or "not yet but
eventually": load `~/.claude/templates/guardrails/ai-features.md`
now and apply its parent-regulation inheritance rule — every
industry template already loaded inherits AI data-flow requirements.
If the project adds AI features later, question 4 is re-asked and
the template loaded then. Never treat AI as a late-project addition
you can bolt on without revisiting guardrails.

After the interview, proceed to **Project Setup** (see CLAUDE.md)
before writing code.
