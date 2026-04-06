# Project Guardrails — Automation / Workflows / Integrations

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for automations:
- GDPR, CCPA/CPRA based on data subjects' geography
- Platform ToS for all integrated services

## Data Flow Safety

- **Map every data flow.** Before building an automation, document: what
  data moves, from where to where, and whether it contains PII. Flag
  any PII movement for review.
- **Never move data to a less-secure system.** If the source system is
  encrypted/access-controlled, the destination must have equivalent
  protections.
- **Never store credentials in automation configs.** Use the platform's
  secret management (environment variables, vault). Never embed API
  keys in workflow definitions.
- **Audit trail.** Log every automation run with: trigger, inputs,
  actions taken, outputs, and any errors. For debugging and compliance.

## Error Handling & Idempotency

- **Every automation must be idempotent.** If it runs twice on the same
  input, the result must be the same. Deduplication keys for webhooks,
  idempotency checks before creating records.
- **Graceful failure.** Every step must handle the possibility that the
  external service is down, rate-limited, or returns unexpected data.
  Never let an automation silently fail and leave data in an
  inconsistent state.
- **Dead letter queue.** Failed automation runs must be captured for
  investigation, not silently dropped.
- **Bounded retries.** Max 3-5 retries with exponential backoff. Never
  retry infinitely.
- **Circuit breaker.** If an automation fails repeatedly (>5 consecutive
  failures), stop and alert rather than continuing to hammer a broken
  service.

## Rate Limits & Cost

- **Respect every platform's rate limits.** Document the limits for each
  integrated service. Implement throttling before hitting them.
- **Cost awareness.** Automations can generate unexpected costs at scale.
  Calculate the cost per run and estimate monthly cost at expected volume
  before deploying. Flag if monthly cost could exceed $50.
- **Volume caps.** Set hard limits on how many records an automation can
  process per run. Never let an automation process an unbounded dataset.

## Webhook Safety

- **Verify webhook signatures** before processing any incoming webhook.
  Never trust the payload without HMAC/signature verification.
- **Validate webhook payloads** with a schema. Never pass webhook data
  directly to business logic without type checking.
- **Idempotency on webhook handlers.** Webhooks can be delivered more
  than once. Handle duplicates gracefully.

## Scheduling

- **Time zone awareness.** All scheduled automations must specify the
  time zone explicitly. Never rely on server time zone.
- **Overlap protection.** If a scheduled automation takes longer than
  the interval between runs, ensure the next run doesn't start until
  the previous one finishes.
- **Maintenance windows.** Document when automations can be safely
  paused for maintenance.
