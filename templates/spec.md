# System Design — [PROJECT NAME]

> Captures the system design as a whole. PLAN.md tracks tasks. Project
> CLAUDE.md tracks stack. This file tracks architecture, data model,
> and design decisions. Updated only when the system design changes
> (requires the project lead's approval).

---

## Overview

[2-3 sentences: what this system does, for whom, and why it exists]

## Architecture

[Component list showing how pieces connect]
- **Frontend:** [e.g., Next.js App Router on Vercel]
- **Backend:** [e.g., Supabase Edge Functions + PostgreSQL]
- **Auth:** [e.g., Supabase Auth → JWT → RLS]
- **External Services:** [e.g., Stripe for payments, SendGrid for email]
- **Flow:** [e.g., Client → Vercel Edge → Supabase API → PostgreSQL]

## Data Model

[Key entities and relationships — not full schema, just the shape]
- **User** — has many Orders, has one Profile
- **Order** — belongs to User, has many LineItems, has one Payment
- [Continue for core entities only]

## Core User Flows

[3-5 most important journeys, numbered steps]

### Flow 1: [e.g., User Registration]
1. [step]
2. [step]
3. [step]

### Flow 2: [e.g., Purchase Flow]
1. [step]
2. [step]
3. [step]

## API Surface

[Endpoints or server actions — shape, not implementation]
- `POST /api/auth/register` → `{ email, password }` → `{ user, token }`
- `GET /api/orders` → `{ data: Order[], meta: { pagination } }`
- [Continue for key endpoints only]

## Constraints

- [e.g., Must support 1000 concurrent users]
- [e.g., WCAG 2.2 AA accessibility required]
- [e.g., PIPEDA + GDPR compliant]
- [e.g., Page load <2.5s LCP on 3G]

## Out of Scope

[What this project explicitly does NOT do — prevents drift]
- [e.g., No native mobile app — web only]
- [e.g., No admin dashboard in v1 — manual DB operations]
- [e.g., No multi-language support in v1]

## Decisions Log

## Decisions Log

See `DECISIONS.md` for full architectural decisions organized by domain.

| Date | Decision | Domain |
|------|----------|--------|
| [date] | [e.g., Supabase over Firebase] | Architecture |
| [date] | [e.g., Server Actions over API routes] | API |
