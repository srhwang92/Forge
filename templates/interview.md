# Forge Interview

> Run only at project kickoff (when no project CLAUDE.md exists yet).
> 5 questions, presented in one message. Project lead answers in one
> pass.

## Questions

**1. Stack & Deploy Target**
- Frontend, backend, database, hosting?
- Template / starter? (If yes, pre-fill from `package.json` etc.)
- CI/CD and deploy target?

**2. Production Status**
- Production with real users? Personal? Prototype? Internal tool?

**3. Data Sensitivity**
- PII, financial, health, or other regulated data?
- Target geography (drives GDPR / CCPA / PIPEDA applicability)?

**4. AI Features**
- Chat, RAG, embeddings, agents, AI-generated content, tool use?
  Even planned for later?
- If yes: provider(s), what data reaches the LLM?

**5. Existing Context**
- Greenfield, template, or inherited codebase?
- Patterns / conventions / quirks Claude must follow or avoid?

## Rules

- "You decide" answers get pushed back: *"I need a decision — these
  shape the project. Options: [A, B, C]."*
- Summarize answers back in one paragraph before generating files.

## Outputs

- Project CLAUDE.md (Stack, Commands, project-specific quirks). Stack
  details the interview didn't cover (path alias, package manager,
  language, runtime, auth) fill from `package.json` / `pyproject.toml`.
- Project GUARDRAILS.md (only if Q3 or Q4 trigger an industry
  template — `fintech.md`, `healthcare.md`, `ai-features.md`).
- `SPEC.md` and `ROADMAP.md` drafted from the interview answers — no
  template, derive from purpose / scope / Q1-Q5. Keep both lightweight
  (high-level only; Superpowers handles per-feature specs).
- Empty MEMORY.md. Geography and any plain-PII context land here as
  a Key Decision so it survives.
