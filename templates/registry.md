# Registry — [PROJECT NAME]

> Living index of everything in the codebase. One line per component.
> Updated after every task that creates, modifies, or deletes a component.
> Source of truth for "what exists" after compaction or context loss.
>
> **Lifecycle:** `draft → implemented → tested → verified → stable`
> Only phase gates promote to `stable`. A **phase gate** is the
> completion of a PLAN.md phase with all exit criteria met, health
> check passed, and phase snapshot generated. If the project uses
> continuous delivery without formal phases, the 4-week health check
> (GUARDRAILS.md) counts as a phase gate for lifecycle promotion.
> Bug fixes on stable components reset status to `verified`.
> Deprecated components are marked for removal.
>
> **Mid-refactor state:** `migrating` — for components that are
> partially extracted, partially consumed by new code, or in the
> middle of a cross-cutting change. A `stable` component whose
> consumers are migrating to a new API is marked `migrating` until
> all consumers are updated. This prevents the "stable for 2 of 3
> features, broken for the third" ambiguity that silently accumulates
> during long refactors. When migration completes, reset to `verified`
> until the next phase gate.

---

## API Endpoints

| Endpoint | File | Purpose | Auth | Bypass | Status | Tests | Rollback |
|----------|------|---------|------|--------|--------|-------|----------|
<!-- | POST /api/auth/register | src/app/api/auth/register/route.ts | User registration | public | - | verified | auth.test, auth.e2e | a1b2c3d | -->
<!-- Bypass values: - (none), service_role, admin, system. Tracks RLS-bypassing access paths. -->

## Pages / Routes

| Route | File | Purpose | Status | Tests | Rollback |
|-------|------|---------|--------|-------|----------|
<!-- | /dashboard | src/app/dashboard/page.tsx | Main dashboard | verified | dashboard.test, dashboard.e2e | a1b2c3d | -->

## Components

| Component | File | Purpose | Status | Tests | Rollback |
|-----------|------|---------|--------|-------|----------|
<!-- | OrderCard | src/components/orders/OrderCard.tsx | Displays order summary | stable | orders.test | e4f5g6h | -->

## Utilities / Hooks

| Name | File | Used by | Status |
|------|------|---------|--------|
<!-- | formatCurrency | src/utils/format.ts | OrderCard, CheckoutSummary | stable | -->

## Database Tables

| Table | Migration | RLS | Purpose | Status |
|-------|-----------|-----|---------|--------|
<!-- | orders | 003_orders.sql | yes (user_id = auth.uid()) | User orders | stable | -->

## External Integrations

| Service | Purpose | Credentials | Status |
|---------|---------|-------------|--------|
<!-- | Stripe | Payments | STRIPE_SECRET_KEY | stable | -->

## AI Prompts

| Prompt | File | Model | Version | Purpose | Last Regression Test | Status |
|--------|------|-------|---------|---------|---------------------|--------|
<!-- | chat_system_v2 | prompts/chat/system.md | gpt-4o-2024-08-06 | 2.3.1 | Customer support chat | 2026-03-28 | stable | -->

## AI Models & Embeddings

| Name | Provider | Version | Used By | Data Sent | BAA/DPA | Status |
|------|----------|---------|---------|-----------|---------|--------|
<!-- | text-embedding-3-large | OpenAI | pinned | RAG pipeline | document chunks (PII scanned) | DPA signed 2026-01-15 | stable | -->

## Agent Tools

| Tool | Agent | Credential Scope | Input Schema | Audit Log | Status |
|------|-------|------------------|--------------|-----------|--------|
<!-- | read_file | refactor_agent | user.project_dir only | tools/schemas/read_file.zod.ts | audit.agent_tools | stable | -->
