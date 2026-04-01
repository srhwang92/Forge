# Project Guardrails — SaaS / Web Application

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

## Applicable Regulations
See global Privacy & Regulatory Awareness for baseline (PIPEDA, PIPA BC,
CASL). Additionally for SaaS:
- GDPR if serving EU users
- CCPA/CPRA if serving California users
- SOC 2 if enterprise clients require compliance audits
- Accessibility: ADA, AODA, EAA for public-facing UI

## Multi-Tenancy Safety

- **Tenant isolation is mandatory.** User A must never see user B's data.
  Every query, API response, and UI state must be scoped to the
  authenticated user's tenant/organization.
- Test tenant isolation: write integration tests that verify cross-tenant
  data access fails.
- Row Level Security (Supabase) or equivalent query scoping on every
  table that holds tenant data.
- **Every database query touching tenant data must include tenant
  scoping** (`WHERE tenant_id = ?` or RLS equivalent). A missing tenant
  filter is a data breach — not a bug.
- Never use shared database connections or caches without tenant-scoped
  keys.

## Subscription & Billing

- **Never modify billing logic without approval.** Trial periods, plan
  limits, usage metering, invoice generation, and payment processing
  changes are all Level 2 escalation.
- Feature flags/plan gating: verify that free-tier users cannot access
  paid features through API endpoints (not just UI hiding).
- Never hardcode plan limits — they must be configurable.

## API & Webhook Safety

- API keys must be revocable and rotatable without downtime.
- Webhook endpoints must verify signatures/HMAC before processing.
- Rate limit all API endpoints — per-user and per-IP.
- Version APIs from day one.

## Data Export & Portability

- Users must be able to export their data (GDPR Article 20, PIPEDA
  right of access). Never build data models that make export impossible.
- Account deletion must be implementable. Flag any architecture
  decisions that would make user data deletion impractical.

## Platform-Specific

### Supabase
- RLS on every table, no exceptions
- Test RLS policies with integration tests
- Use Supabase Auth — never custom auth alongside it
- Edge Functions for webhooks and integrations
- Storage buckets: set policies, use signed URLs for private files

### Vercel
- Environment variables per deployment (preview vs production)
- Never expose server-side env vars to the client (`NEXT_PUBLIC_` prefix
  only for client-safe values)
- Edge middleware for auth checks and redirects

### Railway
- Health check endpoints required
- Graceful shutdown handling (SIGTERM)
- Database connection pooling appropriate for deployment model
