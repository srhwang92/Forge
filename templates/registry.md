# Registry — [PROJECT NAME]

> Living index of everything in the codebase. One line per component.
> Updated after every task that creates, modifies, or deletes a component.
> Source of truth for "what exists" after compaction or context loss.
>
> **Lifecycle:** `draft → implemented → tested → verified → stable`
> Only phase gates promote to `stable`. Bug fixes on stable components
> reset status to `verified`. Deprecated components are marked for removal.

---

## API Endpoints

| Endpoint | File | Purpose | Auth | Status | Tests | Rollback |
|----------|------|---------|------|--------|-------|----------|
<!-- | POST /api/auth/register | src/app/api/auth/register/route.ts | User registration | public | verified | auth.test, auth.e2e | a1b2c3d | -->

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
