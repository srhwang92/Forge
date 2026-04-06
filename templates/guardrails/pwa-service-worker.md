# Project Guardrails — Progressive Web App / Service Worker

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for PWAs:
- Same web security standards as any web application (OWASP ASVS)
- Push notification consent requirements per jurisdiction (GDPR, ePrivacy)
- Offline data storage subject to same privacy regulations as server-side

## Service Worker Security

- **Service workers MUST NOT cache responses to authenticated API
  requests.** Auth-gated content cached by a service worker is
  accessible to any user of the same device/browser after logout.
  Set `Cache-Control: no-store` on all auth-gated API responses.
- **Clear all caches on logout.** Call `caches.keys()` and delete
  every cache. Also clear IndexedDB stores containing user data.
  A logout that only clears cookies/tokens but leaves cached data
  is an incomplete logout. **Implement atomic logout:** use
  `MessageChannel` to signal the service worker to evict caches
  BEFORE the main thread clears cookies and redirects. Verify cache
  eviction is complete before navigation — a race condition where
  the service worker retains cached auth responses after cookie
  deletion can expose the previous user's data on shared devices.
- **Service worker scope must be as narrow as possible.** Never
  register at root scope (`/`) unless the entire domain is under
  application control. A root-scoped service worker intercepts ALL
  requests including third-party scripts.
- **Implement a service worker kill switch.** A server-side endpoint
  or header that instructs the page to unregister all service workers.
  The kill switch must use `navigator.serviceWorker.getRegistrations()`
  to enumerate and unregister ALL registrations, not just the known
  one — an attacker can register a more specific scope (e.g., `/app/`)
  that shadows the legitimate root worker. If a malicious service
  worker is registered (via XSS or dependency compromise), the kill
  switch is the only remediation path that doesn't require every user
  to manually clear browser data.
- **Service worker updates must be immediate for security patches.**
  Use `skipWaiting()` + `clients.claim()` for critical security
  updates. For normal updates, follow the standard update lifecycle.
- **Never trust service worker cache as authoritative.** Always
  validate cached data freshness against the server when online.
  A stale cache that serves outdated security policies or revoked
  permissions is a vulnerability.
- **`importScripts` security.** If the service worker uses
  `importScripts()`, restrict to same-origin scripts only. If loading
  from a CDN, add SRI hashes. A compromised imported script has full
  service worker privileges (intercepting all requests, accessing
  caches and IndexedDB).
- **CSP for PWAs.** Service worker fetch handlers don't enforce CSP —
  CSP is applied by the browser on the response. See
  `frontend-standards.md` CSP section for policy guidance.

## IndexedDB & Offline Storage

- **IndexedDB data containing PII must be encrypted** with a key
  derived from user credentials. When the user logs out, the
  decryption key is destroyed, rendering stored data unreadable.
- **Never store auth tokens or session data in IndexedDB.** Use
  httpOnly cookies for session management. IndexedDB is accessible
  to any same-origin script, including compromised dependencies.
  Note: httpOnly cookies work with Background Sync — `fetch()`
  requests from a service worker automatically include cookies for
  the origin. You do NOT need to store tokens in IndexedDB for
  offline authenticated requests.
- **Clear all IndexedDB stores on logout.** Enumerate databases
  with `indexedDB.databases()` and delete each.
- **Implement storage quotas.** Don't fill the user's device storage
  with cached data. Monitor usage and evict least-recently-used
  data when approaching quota limits.

## Web Push Notifications

- **Push payloads must be encrypted** per RFC 8291 (Web Push
  Encryption). Never send plaintext push payloads.
- **Never include sensitive content in notification titles.** Titles
  are visible on lock screens. Use generic titles ("New message")
  and load details when the user taps the notification.
- **Validate push subscriptions server-side.** Verify the subscription
  endpoint belongs to the claimed user before sending. An attacker
  with a stolen subscription endpoint can receive another user's
  notifications.
- **Implement push notification consent properly.** Request permission
  only after explaining the value (not on page load). Respect
  denial — don't repeatedly prompt. Track consent timestamp.

## Offline-First Data Integrity

- **Conflict resolution strategy must be defined before building
  offline features.** Document the strategy in SPEC.md (required
  section for offline-capable apps). This is a **Tier A approval gate**.
  Options to evaluate:
  - **Last-write-wins** — simplest; risk of silent data loss
  - **CRDTs** — no conflicts possible; complex to implement correctly
  - **Operational Transform (OT)** — real-time focused; server-dependent
  - **Manual merge** — user resolves conflicts; best UX for complex data
  Add a canary test verifying that offline data is never silently
  discarded (test: create offline mutation, simulate server rejection,
  assert user notification appears).
- **Queue offline mutations.** Store pending changes in IndexedDB
  with timestamps. Replay against the server when connectivity
  returns. Handle server rejection of stale writes.
- **Never silently discard offline data.** If a queued mutation fails
  server-side, notify the user. Silent data loss in offline-first
  apps erodes trust faster than any other failure mode.

## Background Sync

- **Background sync tasks must be idempotent.** The browser may
  retry sync events. If the sync handler isn't idempotent, duplicate
  operations will occur.
- **Timeout background sync operations.** A sync handler that hangs
  blocks subsequent syncs. Set explicit timeouts.
