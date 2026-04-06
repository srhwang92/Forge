# Project Guardrails — Dashboard / Internal Tool

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

"It's internal" is not a security exemption. Internal tools often have
broader data access than public-facing apps, making them higher-risk
targets — not lower.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for internal tools:
- SOC 2 if the organization requires compliance audits
- Industry-specific regulations still apply if the tool accesses
  regulated data (HIPAA for health data, GLBA for financial data, etc.)

## Authentication & Access Control

- **Auth is mandatory — no exceptions.** Every internal tool must require
  authentication, even if "only the team uses it." Unauthenticated
  internal tools get exposed through misconfiguration, VPN bypasses,
  and URL leaks.
- **Role-based access control.** Not every team member needs access to
  every feature. Admin functions (user management, data deletion, config
  changes) must be restricted to appropriate roles.
- **Session management.** Enforce session timeouts (max 8 hours for
  general access, max 1 hour for tools with sensitive data access).
  Require re-authentication for destructive or high-privilege operations.
- **SSO integration preferred.** Use the organization's existing auth
  provider (Supabase Auth, Google Workspace, Okta) rather than a
  separate username/password system.

## Data Access & Display

- **Principle of least privilege on data access.** The tool should only
  query and display the data required for its purpose. Never build a
  tool that exposes the entire database "for convenience."
- **Mask or truncate PII in list views.** Show `j***@example.com` not
  full emails, last 4 digits of phone numbers, etc. Full details only
  on detail views with audit logging.
- **Never expose raw database IDs to users** if they're sequential
  integers — they reveal record counts and enable enumeration. Use
  UUIDs or opaque identifiers in URLs and exports.
- **Export controls.** If the tool allows data export (CSV, PDF, API),
  log every export with: who exported, what data, when, and how many
  records. Bulk exports of user data require approval.

## Audit Logging

- **Log every significant action.** User logins, data views (especially
  PII), data modifications, exports, config changes, and permission
  changes. Include: who, what, when, from where (IP/session).
- **Audit logs are append-only.** No UPDATE or DELETE on audit tables.
- **Retention.** Keep audit logs for minimum 1 year, or longer per
  regulatory requirements.

## Error Handling

- **Never display raw database errors, stack traces, or internal paths**
  in the UI — even for internal users. Internal tools get screenshotted,
  shared in Slack, and forwarded to clients.
- Structured error messages with correlation IDs for debugging.

## Deployment & Access

- **Network restriction.** Internal tools should not be publicly
  accessible without additional auth. Use VPN, IP allowlisting, or
  platform-native access controls (Vercel password protection, Railway
  private networking).
- **Separate environment variables** from public-facing applications.
  Internal tools often need elevated database permissions — never share
  credentials with the public app.
- **No shared admin credentials.** Every user has their own account.
  "The team password" is a security and audit failure.
