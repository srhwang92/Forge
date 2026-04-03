# Project Guardrails — ML / AI / Agents

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

## Applicable Regulations
See global Privacy & Regulatory Awareness for baseline (PIPEDA, PIPA BC,
CASL). Additionally for ML/AI:
- EU AI Act — if serving EU users (risk classification, transparency)
- Industry-specific: ECOA/fair lending if credit decisions, HIPAA if
  health data, GLBA if financial data

## Training Data & Model Safety

- **Never train or fine-tune on user data without explicit consent.**
  Consent must be specific to ML usage — general ToS consent is
  insufficient under PIPEDA/GDPR.
- **Never include PII in training datasets** without anonymization.
  Verify anonymization is irreversible (no re-identification risk).
- **Document training data sources.** Provenance, licensing, and any
  known biases in the dataset must be recorded.
- **Never train on copyrighted material** without verifying license
  permits ML training use.

## Algorithmic Fairness (Extended)

Global guardrails cover the basics. For ML projects, additionally:
- **Bias testing is mandatory before deployment.** Test model outputs
  across demographic groups for disparate impact. Document results.
- **Monitor for drift.** Model performance can degrade over time as
  data distributions shift. Flag any deployment without a monitoring
  plan.
- **Human-in-the-loop for high-stakes decisions.** Any AI decision that
  significantly affects a user (credit, employment, housing, insurance,
  healthcare) must have human review capability. Fully autonomous
  high-stakes decisions require the project lead's explicit approval.

## AI Agent Safety

- **Scope limitation.** Agents must have clearly defined boundaries for
  what tools they can use, what data they can access, and what actions
  they can take. Never give an agent unrestricted access.
- **Tool use approval.** Every tool an agent can call must be explicitly
  defined and reviewed. Never let agents discover and use tools
  dynamically without approval.
- **Output validation.** Validate agent outputs before they reach users
  or trigger downstream actions. Never trust agent output without
  structured validation.
- **Cost controls.** Set hard limits on API calls, token usage, and
  execution time per agent run. Never allow unbounded loops.
- **Prompt injection defense.** If agents process user input, implement
  input sanitization and output filtering. Never pass raw user input
  directly into system prompts without validation.
- **Audit logging.** Log every tool call, decision, and output with full
  context for debugging and accountability.

## LLM API Safety

- **Never send PII to LLM APIs** without explicit approval and a data
  processing agreement with the provider.
- **Never hardcode LLM prompts that contain business logic.** Prompts
  should be versioned and configurable, not embedded in code.
- **Token and cost budgets.** Set per-request and per-user token limits.
  Monitor costs daily during development, hourly in production.
- **Fallback behavior.** Every LLM-dependent feature must have a graceful
  fallback for when the API is unavailable, rate-limited, or returning
  errors.

## Model Versioning

- Every deployed model must be versioned and reproducible.
- Never overwrite a production model without a rollback plan.
- A/B testing model changes: never expose all users to an untested model.
