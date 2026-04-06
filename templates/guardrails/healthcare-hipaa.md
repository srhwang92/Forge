# Project Guardrails — Healthcare / HIPAA

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for healthcare:
- HIPAA (if handling PHI of US persons)
- Regional/provincial health privacy legislation (varies by jurisdiction —
  e.g., PIPA BC, PHIPA Ontario, state-level US health privacy laws)
- GDPR if serving EU patients/users
- FDA regulations if the software qualifies as a medical device (SaMD)

## Protected Health Information (PHI)

PHI includes any health information that can identify an individual:
names, dates, phone numbers, emails, SSN, medical record numbers,
health plan numbers, device identifiers, biometric data, photos, and
any other unique identifier combined with health information.

- **PHI must be encrypted at rest and in transit.** AES-256 for storage,
  TLS 1.2+ for transmission. No exceptions.
- **Never log PHI.** Not in application logs, error tracking, analytics,
  or debugging output. Use opaque record IDs only.
- **Never include PHI in error messages** returned to clients or sent
  to error tracking services.
- **Never store PHI in client-side storage** (localStorage, cookies,
  IndexedDB) unless encrypted and explicitly approved.
- **Never send PHI to third-party services** without a Business
  Associate Agreement (BAA) in place. This includes error tracking
  (Sentry), analytics, AI APIs, and logging services. Flag for the project lead's
  review.
- **Minimum necessary standard.** Only access, use, or display the
  minimum PHI necessary for the specific task. Never fetch entire
  patient records when only a subset of fields is needed.

## Access Control

- **Role-based access control (RBAC) is mandatory.** Define roles with
  minimum necessary access. Document who can access what PHI.
- **Audit every PHI access.** Log who accessed what data, when, from
  where, and for what purpose. Audit logs must be append-only and
  retained for 6 years (HIPAA requirement).
- **Automatic session timeout** after inactivity (15 minutes max for
  applications with PHI access).
- **Break-the-glass procedure.** Emergency access to restricted data
  must be logged, reviewed, and justified after the fact.

## Breach Response

- Any potential PHI exposure is **Escalation Level 3 — alert
  immediately.** HIPAA requires notification within 60 days of
  discovery. Do not attempt to assess or minimize — report to the project lead
  instantly.
- Document: what data was potentially exposed, how many records, what
  the exposure vector was, and when it was discovered.

## De-identification

- Test data must be de-identified per HIPAA Safe Harbor (remove all 18
  identifiers) or Expert Determination method.
- Never use real patient data in development, staging, or testing
  environments.

## Medical Content

- **Never generate medical advice, diagnoses, or treatment
  recommendations.** If the application displays health information,
  include appropriate disclaimers flagged for clinical review.
- **Never claim FDA clearance or clinical validation** unless the
  product has actually received it.
