# Project Guardrails — ML / AI / Agents

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output.

## Scope of This Template

Use this template when the project IS an AI system: training,
fine-tuning, serving custom models, autonomous agents with tool use,
MCP servers, or AI-first products where the AI is the primary feature.

For projects ADDING AI features to an existing product (chat, RAG,
content generation), use `ai-features.md` instead — or both, if the
project fits both descriptions. An MCP server that autonomously calls
tools on behalf of users fits both: it's an AI system (this template)
AND it's adding AI features to whatever it integrates with (ai-features).

## Applicable Regulations

See global Privacy & Regulatory Awareness for jurisdiction baseline.
Additionally for ML/AI:
- **EU AI Act** — risk classification and transparency if serving EU
  users. High-risk systems (employment, credit, law enforcement,
  critical infrastructure) have additional obligations.
- **ECOA, fair lending rules** if credit decisions
- **HIPAA** if health data enters the system at any stage — training,
  inference, retrieval, or logs
- **GLBA** if financial data
- **Industry-specific regulations inherit from the data type, not the
  product type.** Training on de-identified health data is still
  HIPAA-adjacent if re-identification risk exists.

## Training Data Safety

- **Consent must be specific to ML usage.** General ToS consent is
  insufficient under GDPR, PIPEDA, and CCPA for training data. Record
  consent with timestamp, scope, and revocation mechanism.
- **PII in training data requires irreversible anonymization.** Hashing
  is not anonymization. Verify re-identification risk with a domain
  expert. Document the anonymization method and its limitations in
  DECISIONS.md.
- **Training data provenance is mandatory.** For every dataset: source,
  license, collection method, date range, known biases, and exclusions.
  Undocumented training data is an audit finding and a legal liability.
- **Copyright clearance.** Never train on copyrighted material without
  verifying the license permits ML training. "Publicly available" is
  not "licensed for training." Web scraping for training data is a
  live legal question in multiple jurisdictions — flag for legal review.

## Model Safety and Fairness

- **Bias testing before deployment.** Test outputs across demographic
  groups. Document disparate impact metrics (statistical parity,
  equalized odds, or domain-appropriate metric). File results in
  DECISIONS.md under Fairness.
- **Drift monitoring post-deployment.** Model performance degrades as
  data distributions shift. Required monitoring: input distribution
  drift, output distribution drift, feedback signal drift. Alert
  thresholds in DECISIONS.md.
- **Human review for high-stakes decisions.** Any decision that
  significantly affects a user — credit, employment, housing,
  insurance, healthcare, immigration, criminal justice — must have
  human review capability before the decision is acted on. Fully
  autonomous high-stakes decisions require Tier A approval and
  explicit legal sign-off.
- **Explanation generation.** Every user-affecting model output must
  be explainable using only legally permissible factors. "The model
  decided" is not an explanation and is not compliant under ECOA,
  EU AI Act, or GDPR Article 22.

## Agent Architecture

Every agent system must define these structural elements before any
implementation work. Missing elements are Tier A approval gates.

- **Agent capability manifest.** A versioned document listing every
  tool the agent can call, the credentials each tool runs under, and
  the permission scope for each. Lives in REGISTRY.md under Agent
  Tools. Any tool addition is Tier B (mention, proceed unless project
  lead objects).
- **Credential scoping per tool, not per agent.** Never give the
  agent a single credential that covers all its tools. A read-only
  tool gets read-only credentials. A write tool gets scoped write
  credentials to only the resources the agent is allowed to modify.
  The principle of least privilege applies to agents more strictly
  than to humans because agents are attackable via prompt injection.
- **Tool call rate and cost budgets.** Per-tool, per-agent-run, and
  per-user caps. An agent run is bounded in both count of calls and
  total cost. Runaway loops are detected and terminated.
- **Nested tool call depth limit.** If tools can call other tools
  (or the agent re-invokes itself), limit depth. Unbounded recursion
  is the agentic equivalent of a fork bomb.
- **MCP server trust model.** Every connected MCP server is a trusted
  code execution environment with access to whatever credentials it
  holds. Connecting an untrusted MCP server is equivalent to running
  untrusted code. Review every MCP server's source before adding to
  any production agent configuration.

## Prompt Injection Defense

Treat every piece of content the model sees as potentially adversarial.
User input, retrieved documents, tool outputs, file contents, web
scrapes, database fields — all of them.

- **Structural delimiters reduce risk but do not eliminate it.** Wrap
  untrusted content in clear delimiters and instruct the model to
  treat it as data. Then assume the instruction may be ignored, and
  layer additional defenses.
- **Output validation is the primary defense.** Every agent output
  that triggers action is validated against a schema AND against the
  original user intent. If the output requests an action the user
  did not authorize, block and log.
- **Capability gating.** Destructive or high-impact tool calls require
  explicit user confirmation for each call, regardless of prior
  approval. A user approving "delete this file" does not approve
  "delete all files."
- **Content filtering on tool outputs.** If a tool returns content
  from an untrusted source (file read, web fetch, database query on
  user-generated content), flag it in the prompt context as untrusted.
  Do not let tool outputs introduce new tool calls without
  re-validation.
- **Regression test suite against known injection patterns.** Maintain
  a growing corpus of injection attempts. Run on every prompt change
  and every model change. Track detection rate over time. This is
  regression testing, not a guarantee — new attacks emerge constantly.

## LLM API Safety

- **Never send PII to LLM APIs without a DPA or BAA.** The provider
  is a data processor under GDPR, a business associate under HIPAA,
  and a service provider under CCPA. Verify terms before first call.
- **Prompt versioning is code change management.** Prompt updates go
  through code review, regression testing, and deployment gates. A
  prompt change that alters model behavior is a behavior change and
  requires the same scrutiny as a logic change in code.
- **Token and cost budgets per-request, per-user, per-tenant, per-day.**
  All four levels. Monitor all four. Alert on any exceeding 50% and
  80% of budget.
- **Fallback behavior for every LLM-dependent feature.** When the
  provider is down, returning errors, or rate-limited: what happens?
  Options in SPEC.md per feature. Silent degradation (e.g., serving
  stale cached responses) requires user-visible indication.
- **Circuit breaker per provider, not per endpoint.** One provider
  outage trips all endpoints using that provider.

## Model Versioning and Deployment

- **Pin exact model versions.** Use specific identifiers, not aliases.
  Document the pinned version in DECISIONS.md and in the AI Models
  table in REGISTRY.md.
- **Provider-side silent changes still occur.** Even with pinning,
  providers update tokenizers, safety filters, and serving
  infrastructure. Daily regression tests against a fixed test set
  detect drift.
- **Model deprecation is a project, not a ticket.** Treat every
  deprecation announcement as triggering a migration plan with its
  own timeline, test suite, and approval gate. 90-day notice periods
  are tight for regulated applications.
- **A/B testing model changes.** Never expose all users to an untested
  model. Start with 1%, monitor quality metrics, expand gradually.
  Include a kill switch that reverts to the stable model instantly.
- **Rollback capability.** Every deployed model has a rollback path.
  Test rollback quarterly.

## Observability

- **Log every inference with metadata**, not full content by default:
  timestamp, user (hashed), model version, prompt version, input
  token count, output token count, latency, status, cost. Full
  prompt/response logging only when explicitly needed and treated
  as PII with associated retention and access controls.
- **Quality metrics dashboard.** Latency (p50, p95, p99), error rate,
  cost per request, cost per user, user feedback ratio, regression
  test pass rate, drift signal, safety filter trigger rate.
- **Alert on anomalies.** Cost spike (>2x baseline), error rate >10%,
  latency >2x baseline, feedback skew, safety filter rate change.
  Every alert has a runbook.
- **Append-only decision log.** Every user-affecting model decision
  is logged to an append-only store with: inputs (anonymized as
  needed), model version, prompt version, output, confidence,
  timestamp. Retention per regulatory requirement. Accessible for
  audit without exposing individual PII.
