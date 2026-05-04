# Guardrails — Healthcare / HIPAA / PHI

Additive when Q3 surfaces health data, medical records, or
HIPAA-covered information.

## PHI Handling

- Every field carrying health data is identified, classified, and
  documented. Treat as Restricted (encrypt at rest + in transit, audit
  all access).
- Never log PHI. Use opaque IDs for correlation. Redact before logging
  request / response bodies.
- Never put PHI in URLs, query params, error messages, or client-side
  state.
- Synthetic data for tests, fixtures, screenshots, prompts. Real PHI
  never leaves the production environment.

## Access Control

- Minimum necessary access — users see only the PHI they legitimately
  need.
- Audit every PHI access. Who accessed what, when, why (reason code).
- Re-authentication for sensitive operations (chart access, exports,
  modifications).
- Session timeouts: 15 minutes idle (HIPAA standard).

## Third-Party Data Flows

- Never send PHI to third-party services (analytics, error tracking,
  AI providers) without a signed BAA.
- A user's health question sent to an LLM is PHI. Verify the provider
  has a BAA before any production AI feature handling health data.
- Data residency: PHI cannot cross borders without explicit
  contractual permission.

## Breach Notification

- HIPAA breach notification: 60 days from discovery. Document the
  procedure in HANDOFF.md (when shipped).
- Discovered exposure → Escalation Level 3. Stop all work. Preserve
  evidence. Alert project lead immediately.

## Compliance Notes

These guardrails reduce risk; they are not a compliance program.
Regulated work requires:

- Compliance officer / Privacy Officer
- Formal access control documentation
- External penetration testing
- BAAs with every vendor handling PHI
- Audit log integrity infrastructure (cryptographic signing,
  tamper-evident storage)

