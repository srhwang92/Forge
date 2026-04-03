# Project Guardrails — Fintech / Lending

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

## Applicable Regulations
See global Privacy & Regulatory Awareness for baseline (PIPEDA, PIPA BC,
CASL). Additionally for fintech/lending:
- GLBA (Gramm-Leach-Bliley Act) — financial data privacy
- ECOA (Equal Credit Opportunity Act) — fair lending
- TILA (Truth in Lending Act) — disclosure requirements
- GDPR, CCPA/CPRA if serving EU or California users
- Canadian AI & Data Act (AIDA — proposed, not yet enacted) if using
  automated decision systems

## Financial Calculation Safety

- **Never implement financial calculations without the project lead's explicit
  approval.** This includes: interest rates, APR, amortization, payment
  amounts, fees, penalties, credit scoring, loan eligibility,
  debt-to-income ratios, and any formula affecting what a user pays
  or qualifies for.
- **Never use floating point for money.** Integer cents or decimal
  library only. A rounding error in a lending calculation is a
  regulatory violation.
- **Never hardcode financial parameters.** Rates, fees, thresholds, and
  limits must come from configuration or database.
- **Test every calculation with known-answer pairs.** Minimum 3 test
  cases with manually verified expected outputs per formula.

## Fair Lending & Anti-Discrimination

This is the highest-liability area. Violations carry class-action
exposure, regulatory fines, and criminal penalties.

- **Never use protected characteristics as model inputs.** This includes
  race, color, national origin, sex, gender, age, religion, marital
  status, disability, family status, sexual orientation, and receipt of
  public assistance.
- **Never use proxy variables.** Zip code → race. First name →
  ethnicity. Browser language → national origin. Flag any variable that
  correlates with a protected class for bias review.
- **Every lending decision must be explainable** using only legally
  permissible factors.
- **Adverse action notices.** If a user is denied or receives less
  favorable terms, the system must be able to generate the legally
  required reasons.

## Audit Trail Requirements

- **Append-only, tamper-evident audit logs** for all lending decisions.
  Every approval, denial, pricing decision, and eligibility
  determination logged with: inputs, decision, timestamp, and context
  to reconstruct reasoning.
- INSERT-only permissions — no UPDATE or DELETE on audit tables.
- Audit logs must be retained per regulatory requirements (minimum
  5 years for ECOA, 3 years for TILA).

## User-Facing Financial Content

- **Never generate financial advice, projections, or guarantees.**
  "You could save $X" or "estimated payment: $X" must be computed from
  verified formulas and flagged for review.
- **Lending disclosures must be reviewed by a qualified professional.**
  Claude may draft skeletons marked "REQUIRES LEGAL REVIEW."
- APR calculations must follow Regulation Z methodology exactly.

## Platform-Specific

- Supabase: RLS on every table, no exceptions. Test that user A cannot
  access user B's financial data.
- PCI-DSS: if handling payment cards, use Stripe/processor hosted fields.
  Keep card data off your servers entirely.
