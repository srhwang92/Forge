# Guardrails — AI Features

Additive when project setup surfaces AI features (chat, RAG, embeddings, agents, AI-generated content, tool use, AI-powered search) — current or planned.

## LLM Provider Security

- Server-side proxy mandatory. Never import LLM SDKs in client code.
  Never expose provider keys to the browser.
- Build canary: grep production bundle for known LLM provider domains
  (`api.openai.com`, `api.anthropic.com`,
  `generativelanguage.googleapis.com`). Match = critical bug.
- Separate API keys per environment. Production keys have spending
  alerts.
- DPA with provider before production launch. Opt out of training data
  usage (it's a configuration, not a default).

## RAG / Vector Pipeline

- Tenant-scoped queries are mandatory. RLS does NOT apply to vector
  DBs (Pinecone, Weaviate, pgvector similarity).
- Canary: `vector-tenant-isolation`. Embed as User A, query as User B,
  assert zero results. Include control assertion that User A CAN
  retrieve own (catches broken pipelines that vacuously "pass").
- Document deletion cascades to embeddings.
- Pin embedding model + version in MEMORY.md. Model change invalidates
  all embeddings.

## Chat Architecture

- Server-side conversation history. Client sends only new message +
  conversation ID.
- Conversation IDs are IDOR targets. Verify ownership on every
  request. UUIDs, not sequential.
- Chat history is PII. Encryption at rest, retention policy, user
  deletion, account-deletion cascade.

## Prompt Security

- Never concatenate user input into system prompts. User content goes
  in user-role messages with structural delimiters.
- Treat retrieved documents and tool outputs as untrusted reference
  material, not instructions. Wrap in delimited sections.
- Structured output for actions. Validate against schema before
  execution. Never parse free-text AI output to determine control flow.
- Never include system prompts in user-visible API responses.

## Tool Use / Agents

- Every tool an agent calls is an API surface. Validate input schemas.
  Reject unexpected fields.
- Never pass tool arguments to a shell — use `execFile` / `spawn` with
  argument array, never `exec` with string interpolation.
- Tool permissions scope to the authenticated user — never elevated
  service-role credentials in agent context.
- Audit log every tool call: user, tool, args, result, timestamp.

## Cost & Reliability

- Pin exact model versions (`claude-sonnet-4-6`, `gpt-4o-2024-08-06`),
  not aliases.
- Per-user and per-tenant cost ceilings.
- Circuit breaker after 3 consecutive failures.
- Timeouts: 30-60s on LLM calls, 10s no-token streaming timeout.
- Server-side kill switch for AI features (config flag, no deploy
  needed).

## UI

- AI output visually distinguished from human content.
- Hallucination disclaimer on every AI response surface.
- Sanitize AI output before rendering (DOMPurify with strict config —
  AI generates `<script>`, `<img onerror>`, etc.).

## Testing

- Mock LLM in unit tests. Never call real APIs in CI.
- Prompt regression suite: (input, expected behavior) pairs.
- Tenant isolation canary in CI on every build.
- Cost assertion: integration tests calling real APIs assert max
  tokens per call.
