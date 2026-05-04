# Guardrails — Fintech / Money / Lending

Additive when Q3 surfaces money handling, lending, payments, or
regulated financial data.

## Money

- All money as integer cents or a decimal library — never floats.
- Currency tagged on every amount: `{ amount: 1999, currency: "USD" }`.
- Never hardcode prices, fees, rates, thresholds, or eligibility
  criteria — config or database.
- Display formatting via `Intl.NumberFormat`, never custom.

## Algorithmic Fairness (ECOA / FCRA / Fair Lending)

- Never use protected characteristics as algorithm inputs: race,
  color, national origin, sex, gender, age, religion, marital status,
  disability, family status, sexual orientation, public assistance
  status.
- Never use proxies for protected characteristics. Zip code can proxy
  for race; first name can proxy for ethnicity. Flag for bias review.
- User-affecting decisions must be explainable using only permissible
  factors. "The model decided" is not an explanation.
- Log every approval / denial / score with inputs (anonymized PII —
  ranges not exact figures), decision, timestamp, model version.

## Audit & Compliance

- Audit logs are immutable. Tamper-evident storage. Retention per
  regulation (varies by product — confirm with project lead /
  compliance officer).
- Right-to-erasure conflicts with audit retention: anonymize PII in
  audit records (hashed user IDs, income brackets), keep the decision
  trail. Regulatory retention overrides erasure per PIPEDA / GDPR
  Article 17(3)(b).
- Document every credit decision's reasoning sufficient to satisfy
  ECOA adverse-action notice requirements.

## Testing

- Money math: ≥3 known-answer test cases per calculation. Verified by
  hand, not by running the code.
- Fairness regression suite: (applicant, expected decision) pairs
  covering all protected-class combinations available in synthetic
  data.

