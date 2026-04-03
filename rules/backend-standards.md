---
description: Backend architecture, API design, database, security, and reliability standards. Loads automatically when working on backend files.
globs: ["*.py", "api/**", "server/**", "supabase/**", "src/lib/**", "src/server/**", "edge-functions/**", "src/actions/**", "src/app/api/**"]
---

# Backend Engineering Standards

Production-grade rules. Every rule prevents a documented failure mode in
AI-generated backend code.

---

## Type Safety

TypeScript (server):
- `strict: true` — no exceptions. Same rules as frontend TypeScript.
- Runtime validation at every external boundary: API inputs, webhook
  payloads, third-party responses, environment variables at startup.
  Use zod, valibot, or tRPC — never trust unvalidated data.
- Derive types from validation schemas — single source of truth.

Python:
- Type hints on every function signature (parameters and return type).
- `mypy --strict` or `pyright` in CI. No `# type: ignore` without a
  comment explaining why and a TODO to fix it.
- Pydantic models for all API request/response shapes and config.
- Never use `dict` as a function parameter when a typed model exists.

---

## API-First Design (Schema-Driven)

- Define the contract (types, endpoints, request/response shapes) BEFORE
  writing implementation code. Types drive code, not the reverse.
- OpenAPI for REST, Protobufs for gRPC. Lint the spec in CI.
- Generate routing boilerplate, client SDKs, and documentation from the
  schema. Never hand-write what can be generated.
- Reject undocumented routes — if it's not in the spec, it doesn't ship.

### API Response Format

Use a consistent envelope for every endpoint in the project:
```
Success: { data: T, meta?: { pagination, timing } }
Error:   { error: { code: string, message: string, details?: any } }
```
- Always return the same shape. Never mix flat responses with enveloped
  ones across different endpoints.
- Include pagination metadata on every list endpoint (see Pagination).
- HTTP status codes must be semantically correct: 200 success, 201
  created, 400 bad request, 401 unauthenticated, 403 forbidden, 404
  not found, 409 conflict, 422 validation error, 429 rate limited,
  500 internal. Never return 200 with an error body.

### API Versioning

- Version from day one. Use URL prefix (`/v1/`) or header-based
  versioning — pick one per project and be consistent.
- Never break existing contract for active consumers. Additive changes
  (new optional fields) are non-breaking. Removing or renaming fields
  is breaking and requires a new version.
- Define a deprecation timeline before removing old versions.

### Pagination

Every list endpoint must be paginated. Never return unbounded results.
- **Cursor-based** preferred for real-time or large datasets (no count
  needed, consistent with inserts/deletes).
- **Offset-based** acceptable for small, stable datasets with explicit
  page numbers in the UI.
- Default limit: 20-50 items. Maximum limit: 100. Always enforce
  server-side even if the client doesn't send a limit.
- Return pagination metadata: `{ cursor, hasMore }` or
  `{ page, pageSize, totalCount, totalPages }`.

---

## Input Validation

Validate every input at the API boundary — before it reaches business
logic. Never trust client data, URL parameters, headers, or webhook
payloads.

- Validate type, format, length, range, and allowed values
- Reject unexpected fields (strict schemas) — don't silently ignore them
- Validate at the boundary once; don't re-validate deep in business logic
- Return 422 with field-level error details:
  `{ error: { code: "VALIDATION_ERROR", details: [{ field, message }] } }`
- File uploads: validate content type, file size, and filename server-side.
  Never trust the client's content-type header alone.

---

## Error Handling

AI writes happy-path code. Every function must consider what happens when
things go wrong.

- **Typed errors.** Define error types/codes per domain (e.g.,
  `ORDER_NOT_FOUND`, `INSUFFICIENT_BALANCE`). Never use generic strings.
- **Error at the boundary.** Catch and translate errors at the delivery
  layer. Business logic throws domain errors; the API layer maps them to
  HTTP status codes and response shapes.
- **Never swallow errors.** Every `catch` must either handle, rethrow, or
  log with context. Empty catch blocks are banned.
- **Never expose internals.** Client error responses include error code,
  human-friendly message, and correlation ID. Never stack traces, SQL
  queries, file paths, or internal state.
- **Fail fast.** Validate preconditions at the top of a function. Return
  early on invalid state. Don't proceed optimistically and catch later.
- **Centralized error handler.** One middleware/handler that catches all
  unhandled errors, logs them with full context, and returns a sanitized
  response. Individual routes should not wrap everything in try/catch.
- **Classify errors as retryable or permanent.** Timeouts, 503, rate
  limits (429) are retryable — clients and jobs should back off and
  retry. Validation errors (400/422), not found (404), and conflict
  (409) are permanent — retrying won't help. Include a `retryable`
  boolean or use error codes that make this distinction clear.

---

## Database Discipline

### Query Performance
- **No N+1 queries.** Never fetch a list then query related data in a
  loop. Use joins, eager loading, or batch queries.
- **Pagination on every list query.** Never `SELECT * FROM table` without
  LIMIT. (See Pagination above.)
- **Indexed lookups.** Every WHERE clause, JOIN condition, and ORDER BY
  column that runs against significant data must have an index. If adding
  a query on a new column, add the index in the same migration.
- **Select only needed columns.** Never `SELECT *` in production code.
  Specify columns explicitly.
- **Explain plans.** For complex queries, run EXPLAIN ANALYZE during
  development to verify the query plan uses indexes.
- **Connection pooling.** Never create a new database connection per
  request. Use a connection pool (PgBouncer, built-in pool in Prisma/
  Drizzle, Supabase's pooler). Set pool size appropriate to the
  deployment (serverless needs smaller pools than long-running servers).
- **Database constraints are mandatory.** Never rely solely on
  application-level validation. The database is the last line of defense:
  - Foreign keys for every relationship
  - Unique constraints on business-unique fields (email, slug, etc.)
  - NOT NULL on every column that should always have a value
  - CHECK constraints for value ranges/enums where applicable
  If the app has a bug, the DB must still reject invalid data.

### Concurrency & Race Conditions
- **Read-then-write is a race condition.** If you read a value, modify it,
  then write it back, another request can change it between read and
  write. Use atomic operations: `UPDATE ... SET count = count + 1`,
  database transactions with `SELECT ... FOR UPDATE`, or optimistic
  concurrency (version columns).
- **Idempotency on state-changing operations.** Every POST/PUT/DELETE that
  creates a side effect (payment, email, status change) must be safely
  re-runnable. Use idempotency keys: store the key + result, return the
  stored result on duplicate requests.
- **Transactions for multi-step operations.** If steps A and B must both
  succeed or both fail, wrap them in a transaction. Never leave partial
  state.

### Migration Safety
- **Never drop columns or tables in the same deploy that removes code
  using them.** Two-phase: first deploy removes the code reference, second
  deploy (after verification) drops the column/table.
- **Additive-only by default.** Add columns as nullable or with defaults.
  Never add a NOT NULL column without a default to a populated table.
- **Test migrations against production-volume data.** A migration that
  runs in 200ms on 100 rows might lock the table for 10 minutes on
  1 million rows.
- **Every migration must be reversible** or explicitly documented as
  irreversible with the project lead's approval.

### Data Retention & Soft Delete
- **Never hard-delete user data or business records.** Use soft delete
  (`deleted_at` timestamp) or archive to a separate table. Hard delete
  is only acceptable for ephemeral data (sessions, temp files, caches)
  or when explicitly required by data privacy regulations (right to
  erasure) — and even then, log that the deletion occurred.
- **Audit-sensitive tables** (users, transactions, permissions, payments)
  must retain history. Use an audit log, event sourcing, or a history
  table — never silently overwrite previous values.

### Bulk Operations
- **Never process large datasets one row at a time.** Use batch inserts
  (`INSERT INTO ... VALUES (...), (...), (...)`), bulk updates, and
  `WHERE IN` clauses instead of looping single queries.
- **Chunk large operations.** If processing 10,000 records, batch into
  chunks of 100-500. Commit per chunk to avoid long-running transactions
  that lock tables.
- **Bulk external API calls** — use concurrency control (e.g., process
  5 in parallel, not 10,000 at once). Respect rate limits.

---

## Authentication & Authorization

- **Never build custom auth.** Use established libraries/services
  (Supabase Auth, NextAuth, Auth.js, Clerk). Flag any auth changes for
  the project lead's review.
- **Verify ownership on every resource access.** The #1 API vulnerability
  (BOLA/IDOR): a user changes an ID in the request and accesses another
  user's data. Every endpoint that returns or modifies a resource must
  verify the authenticated user owns or has permission for that resource.
- **Short-lived tokens.** JWTs should expire in ≤15 minutes. Use refresh
  tokens (httpOnly, secure, same-site) for session continuity.
- **Server-side session validation.** Never trust a JWT's claims without
  checking revocation status for sensitive operations.
- **Principle of least privilege.** Give each client/role only the
  permissions it needs. Default-deny.
- **Admin/elevated endpoints must explicitly verify role.** AI creates
  admin routes that check authentication but skip role verification.
  Every endpoint that performs admin operations must check
  `user.role === 'admin'` (or equivalent) — not just `user !== null`.
  Test by calling admin endpoints with a regular user token.

---

## API Security

- **HTTPS everywhere.** No exceptions.
- **Rate limiting** on all public endpoints. Enforce per-IP and
  per-authenticated-user limits. Return 429 with `Retry-After` header.
- **CORS** — explicit allowed origins. Never `*` in production.
- **CSRF** protection on all state-changing operations using cookies.
- **Request size limits.** Enforce max body size to prevent abuse.
- **Parameterized queries only.** Never string-concatenate SQL. Ever.
- **Dependency distrust.** Treat third-party API responses like user
  input — validate, bound, and sanitize before processing.
- **Webhook verification.** Verify the signature/HMAC of every incoming
  webhook before processing. Never trust the payload without verifying
  it came from the expected sender.

---

## Clean Architecture (Hexagonal / Ports & Adapters)

- **Domain layer:** Pure business logic. Knows nothing about HTTP, SQL,
  or any framework. Only business rules.
- **Application layer:** Orchestrates use cases. Calls domain, coordinates
  flow, handles transactions.
- **Infrastructure layer:** Database queries, external API calls, file
  system, caching. Injected into application layer via interfaces —
  never imported directly by business logic.
- **Delivery layer:** HTTP/GraphQL translation only. Validates input,
  calls application layer, formats response.
- **Dependency rule:** Dependencies point inward. Domain imports nothing.
  Application imports domain. Infrastructure implements interfaces
  defined by application. Delivery calls application.
- Swapping database, API protocol, or external service changes only
  the infrastructure layer. Core logic stays untouched.

---

## Domain-Driven Design

- **Ubiquitous language:** code names match business terms exactly. If the
  business says "Client," the code says `Client`, not `User`.
- **Bounded contexts:** "Product" means different things in different
  domains. Don't build one 100-column table — isolate to domains.
- **Aggregate roots** for consistency boundaries. External code references
  the aggregate root, never an internal entity directly.
- **Value objects** for identity-less concepts (Money, Email, DateRange).
  Immutable, equality by value.

---

## Modular Boundaries

- Even in a monolith, modules communicate through defined interfaces
- No cross-module direct database queries
- Each module owns its data — other modules go through the module's API
- Extract to microservice only when scaling demands it (not preemptively)
- Circular dependencies between modules are a design error — resolve
  by extracting shared concepts into a common module or inverting the
  dependency direction

---

## Background Jobs & Async Processing

- **Never do heavy work in a request handler.** Email sending, PDF
  generation, external API calls with retries, data processing — push
  to a background job queue (BullMQ, Inngest, Supabase Edge Functions
  with pg_cron, or equivalent).
- **Idempotent jobs.** Every job must be safely re-runnable (see
  Concurrency above). At-least-once delivery is the norm.
- **Bounded retries.** Set max retry count and exponential backoff.
  Never retry infinitely. Dead-letter failed jobs for investigation.
- **Timeouts.** Every external call (HTTP, database, queue) must have
  an explicit timeout. Never wait indefinitely. Default: 10s for HTTP,
  5s for database queries, 30s for background jobs.
- **Graceful shutdown.** On SIGTERM/SIGINT, stop accepting new requests,
  finish in-flight work, close database connections, then exit. Never
  kill mid-transaction.

---

## Caching

- **Cache at the right layer.** Edge/CDN for static assets and public
  API responses. Application cache (Redis, in-memory) for computed
  results and database query results. HTTP cache headers for clients.
- **Explicit invalidation.** Every cached value must have a defined
  TTL and a clear invalidation trigger (e.g., "invalidate user cache
  on profile update"). Never cache without a plan for staleness.
- **Cache-aside pattern** by default: check cache → miss → fetch from
  source → store in cache → return. Don't pre-populate caches unless
  data is expensive and access patterns are predictable.
- Never cache user-specific data in shared/CDN caches without proper
  cache keys (include user ID or session).

---

## Observability

Every production service must include:
- **Structured logging** (NOT `console.log`) with context: request ID,
  user ID, action, result, duration.
- **Correlation IDs.** Generate a unique ID per request, propagate
  through all downstream calls, include in every log entry and error
  response.
- **Error responses** with correlation ID and timestamp — never stack
  traces or internal details to the client.
- **Health check endpoint** (`GET /health`) returning service status,
  dependency health (DB, cache, external services), and version.
- **Metrics.** Track request latency (p50, p95, p99), error rates, and
  throughput. Alert on anomalies.
- **Audit logging** for sensitive operations (auth, payments, permission
  changes, data deletion). Include who, what, when, from where.
- **Never log PII.** No email addresses, full names, phone numbers,
  passwords, auth tokens, credit card numbers, or government IDs in
  logs. Use opaque user IDs for correlation. If you must reference a
  user in logs, use their internal ID — never their email or name.
  Redact or mask sensitive fields before logging request/response bodies.

---

## Data Serialization

- **Dates:** Always ISO 8601 (`2026-03-31T12:00:00Z`). Always UTC in
  API responses. Convert to local time on the client.
- **Enums:** Serialize as strings, not integers. `"active"` not `1`.
- **Nulls:** Be explicit. `null` means "no value." Don't use empty
  strings or `0` to represent absence. Omitting a field and sending
  `null` should mean different things — document which your API uses.
- **Money:** Never use floating point. Use integer cents or a decimal
  library. `{ amount: 1999, currency: "USD" }` not `{ amount: 19.99 }`.

---

## Testing

- **Unit tests** for business logic (domain layer, pure functions,
  validation rules). Fast, no I/O.
- **Integration tests** for API endpoints: send real HTTP requests, hit
  a real (test) database, verify response shape + status + side effects.
- **Contract tests** for external API integrations: verify your code
  handles actual response shapes from third parties.
- Test the testing pyramid: many unit, fewer integration, few E2E.
- Test error paths, not just happy paths. Test validation rejection,
  auth failures, race conditions (where practical), and edge cases.
- **Never mock the thing you're testing.** Mock external boundaries
  (databases, APIs, queues) — not internal modules.
- Factory functions for test data — never hardcode fixture objects.
- **Test isolation.** Every test must be independent — no shared mutable
  state, no dependency on execution order. Each test sets up its own
  data and cleans up after itself (or use transaction rollback).
- **Dedicated test database.** Integration tests hit a real test database,
  never production or shared dev. Reset/migrate before each test suite.

---

## Environment & Configuration

- **Validate env vars at startup.** Don't fail on the first request
  that uses a missing env var. Use zod/Pydantic to parse and validate
  `process.env` / `os.environ` on boot. Fail loudly and immediately.
- **Parity.** Dev, staging, and production must use the same database
  engine, the same major versions, and the same auth provider. "Works
  on my machine" is not a defense.
- **Secret management.** Never commit secrets. Use `.env` for local dev,
  platform-managed secrets (Vercel env, Supabase vault) for deployment.
  Document all required vars in `.env.example`.

---

## File Organization

- Feature/domain-based structure: `/modules/billing/`, `/modules/auth/`
  (or `/features/` if matching frontend convention)
- Within each module: `routes/`, `services/`, `models/`, `tests/`
- Shared utilities in `/lib/` or `/common/`
- Co-locate tests next to the code they test
- Naming: camelCase for functions/variables, PascalCase for classes/types,
  SCREAMING_SNAKE for constants, kebab-case for file names

---

## Supabase-Specific

- **RLS on every table, no exceptions.** Test RLS policies with
  integration tests that verify: authenticated user A cannot access
  user B's data.
- **Use Supabase Auth** — never build custom auth alongside it.
- **Edge Functions** for custom server logic, webhooks, and integrations.
  Keep them small and focused — one function per responsibility.
- **Database functions (plpgsql)** for complex queries that benefit from
  running inside the database (aggregations, RLS-bypassing admin
  operations with `SECURITY DEFINER`).
- **Realtime subscriptions:** use Supabase Realtime for live data. Set
  up RLS on the tables and use broadcast/presence for ephemeral state.
- **Storage:** use Supabase Storage for file uploads. Set bucket policies
  (public vs private), use signed URLs for private file access with
  expiry, validate file type and size server-side before upload, and
  never expose storage credentials to the client.
- Check Context7 for Supabase SDK patterns before implementing.

---

## Next.js Server Patterns

When working in a Next.js App Router project:
- **Server Actions for mutations.** Form submissions, data creation,
  updates, and deletes should use server actions — not API routes.
  Server actions are simpler, type-safe, and don't require manual
  fetch/error/loading handling.
- **API routes (`route.ts`) for webhooks, external integrations, and
  endpoints consumed by non-Next.js clients.** If only your own frontend
  calls it, use a server action instead.
- **Server Components for data fetching.** Read data in the component
  that needs it. Don't create an API route just to fetch data for your
  own frontend — that's an unnecessary round-trip.
- **Never import server-only code in client components.** Use the
  `server-only` package to enforce the boundary at build time.

---

## Backend Anti-Patterns — Explicit Prohibitions

Never do any of the following:
- Return 200 with an error body — use correct HTTP status codes
- `SELECT *` in production queries — specify columns
- Fetch a list then query related data in a loop (N+1)
- Return unbounded result sets (missing LIMIT/pagination)
- String-concatenate SQL — parameterized queries only
- Put heavy/slow work in request handlers — use background jobs
- Use floating point for money
- Store or transmit secrets in logs, error responses, or URLs
- Log PII (emails, names, phone numbers, tokens) — use opaque IDs
- Deploy destructive migrations without a two-phase rollback plan
- Retry failed operations infinitely — bounded retries with backoff
- Hard-delete business records — use soft delete or archival
- Process large datasets one row at a time — use batch operations
- Skip database constraints and rely solely on app-level validation
- Create API routes in Next.js for mutations your own frontend calls
  — use server actions. **But treat server actions as public endpoints.**
  They are callable by anyone with the action ID — not private just
  because they're co-located with the component. Every server action
  must validate auth, check permissions, and validate input with the
  same rigor as an API route.
- **Sequential awaits for independent operations** — if two async calls
  don't depend on each other's result, use `Promise.all` (JS) or
  `asyncio.gather` (Python). AI code is 8x more likely to make
  independent calls sequential. Look for consecutive `await` lines
  where the second doesn't use the first's result.
- **Deserialize untrusted data without schema validation** — never
  `JSON.parse(input) as MyType`. Always validate through zod/valibot
  before trusting shape. Never use `eval`, `pickle.loads`, or
  `deserialize` on data from clients or external sources.
