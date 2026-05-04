# Forge — Global Guardrails

Always-active safety rules. Project-level GUARDRAILS.md (if it exists)
extends but never weakens these. When in doubt, choose the more
restrictive interpretation.

---

## Approval Gate

Before any of these, ask the project lead:

- Destructive operations (deleting files, dropping tables, force-push)
- Security-affecting changes (auth, sessions, encryption, permissions)
- Money / billing / payment logic
- Production-mutating actions (deploys, migrations, env changes)
- Bulk data operations (>100 records)

Approval is per-action, not per-flow. "Go" on one action does not
extend to the next.

---

## Hallucination Prevention

- Never claim code works without running it.
- Never reference a function, API, or config option without
  verification (Context7, source code, or running it).
- Never fabricate test results, benchmarks, or metrics.
- Never claim a dependency has a feature without checking.
- Express uncertainty explicitly — "I'm not certain, let me verify"
  beats false confidence.
- Mid-session staleness: if you haven't read a referenced file within
  ~10 turns, re-read it.
- After compaction, re-verify before claiming anything.
- Never validate your own output with your own reasoning. Run the
  test, execute the command, observe the output.

---

## Security Non-Negotiables

- Never hardcode secrets, tokens, API keys, or credentials. Use env
  vars; document required vars in `.env.example`.
- Never commit `.env*` files. Verify `.gitignore` covers them.
- Never use `eval()`, `dangerouslySetInnerHTML`, raw shell exec on
  user input, or equivalent without sanitization.
- Never disable strict mode, lint rules, or test suites.
- Never silence errors with empty catch blocks.
- Parameterized queries only — no SQL string concatenation.
- Input validation at every API boundary (zod / valibot / pydantic).
- Output encoding for XSS prevention. Sanitize AI-generated HTML
  with DOMPurify (strict config).
- **IDOR prevention:** every endpoint that returns or mutates a
  resource by ID must verify the requesting user owns it. AI
  consistently authenticates without authorizing.
- **Privilege escalation:** every admin endpoint must verify the
  user's role, not just authentication.
- HTTPS only in production. No `secure: false` cookies in prod.
- Never store passwords in plaintext. Use bcrypt or argon2.
- Never implement custom cryptography.

---

## Data Protection

- Never log PII (emails, names, phone numbers, tokens, credit cards,
  SSNs, IPs, addresses, DOBs, biometric data, location).
- Never put real user data in test fixtures, seed files, screenshots,
  or AI prompts. Use synthetic data.
- Never expose PII in URLs, query params, error messages, or
  client-side state.
- Never send user data to third-party APIs (analytics, AI, logging,
  error tracking) without flagging for project lead approval.
- Never use floating point for money — integer cents or decimal lib.
- Never hardcode business rules (prices, fees, thresholds) — config
  or database.

---

## Production Safety

- Never run destructive commands against production.
- Never deploy without explicit approval.
- Never modify production env vars — flag for project lead to do via
  the platform dashboard.
- Verify the target environment before any DB or API operation.
- No `console.log`, `debugger`, hardcoded test data, mock services,
  or unlinked TODO/FIXME in production code.
- No unbounded mutations: every UPDATE/DELETE must have a WHERE.
- No unbounded reads: every list query must have a LIMIT.
- Checkpoint before risky operations:
  `git add -A && git commit -m "chore: checkpoint before [op]"`

---

## Git Safety

- Claude does not create branches, switch branches, push, or
  force-push. The project lead controls branch state.
- Never commit to main/master directly. If main is active, ask for a
  feature branch.
- Never rewrite git history without explicit instruction.
- Never include sensitive data in commit messages.
- Never commit binaries (DB dumps, build artifacts, large media)
  without asking.

---

## Dependencies

- Verify packages exist before importing — AI hallucinates package
  names. Confirm via npm / PyPI or Context7.
- Check for known vulnerabilities (`npm audit` or equivalent).
- Never add a dependency for something achievable in <20 lines.
- Pin major versions.
- Never delete or regenerate lockfiles without reason.

---

## Test Integrity

- Tests derive from spec, not from the code under test. Don't
  reverse-engineer assertions from output.
- Never generate assertions by running the code and copying its
  output.
- Don't update tests to pass new code unless the spec changed.
  Failing test after refactor = behavior changed; investigate first.
- Adversarial edge cases are mandatory: empty input, boundary values,
  null where unexpected, unicode, negatives where positives expected.

---

## Escalation

- **Level 1 — Ask and propose.** Present options with a recommendation.
- **Level 2 — Ask and wait.** For Approval Gate items above. Wait for
  explicit approval.
- **Level 3 — Alert immediately.** Stop work, document in CONTEXT.md,
  present recovery steps. Triggers: test regression, data corruption,
  deploy break, security vulnerability discovered, secret compromise,
  exposed user data.

---

## Industry Extensions

When project setup surfaces regulated data or AI features, the
additions in `templates/guardrails-additions/*.md` get combined into
project-level `GUARDRAILS.md`. Currently:

- `ai-features.md` — chat, RAG, agents, embeddings
- `fintech.md` — money, lending, financial transactions
- `healthcare.md` — PHI, HIPAA-covered work

These extend the rules above. They do not replace them.
