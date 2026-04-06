# Project Guardrails — CMS-Backed Application

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.
Applies to projects using Payload CMS, Strapi, Directus, Sanity,
Contentful, or any headless CMS with an admin panel.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for CMS-backed apps:
- Content moderation obligations if user-generated content is involved
- Accessibility requirements for published content (WCAG)
- Copyright and licensing for media assets

## Admin Panel Security

- **Admin panels MUST NOT be publicly accessible without protection.**
  In production, require at minimum: rate limiting on login, MFA for
  admin accounts, and either IP restriction or VPN requirement.
  Session timeout ≤1 hour for admin sessions.
- **Default admin credentials must be changed immediately.** If the
  CMS scaffolds with default admin/admin credentials, changing them
  is step 1 before any other work.
- **Admin panel URL should not be guessable.** If the CMS allows
  customizing the admin path (e.g., Payload's `admin.url`), change
  it from the default `/admin`. This is defense-in-depth, not a
  substitute for proper auth.
- **Separate admin and public domains when possible.** Serve the
  admin panel from a different subdomain or port than the public
  frontend. This enables domain-level access controls (IP allowlist,
  VPN) without affecting public users.
- **CSP for admin panels.** Set a strict Content Security Policy on
  the admin panel. See `frontend-standards.md` CSP section for guidance.

### GraphQL API Security (if CMS exposes GraphQL)

Payload, Strapi, Directus, and Contentful expose GraphQL APIs. Apply:
- **Disable introspection in production.** Introspection reveals the
  entire schema including draft fields and internal types.
- **Set query depth limits** (max 5-7 levels) to prevent deeply nested
  queries that cause exponential database operations.
- **Set query complexity limits** to prevent queries that request all
  fields on all types in a single request.
- **Disable batching** or limit batch size to prevent batch query abuse.

## Media Upload Security

- **Verify actual file content, not just MIME type.** CMS frameworks
  handle upload MIME validation internally, but this is insufficient.
  Check magic bytes server-side to verify the actual file type matches
  the declared type. A file declared as `image/jpeg` but containing
  PHP code is a polyglot attack.
- **Never serve uploaded files from the same domain as the
  application.** Use object storage (S3, Supabase Storage, Cloudflare
  R2) with a separate origin. If served from the same domain, a
  malicious upload can execute in the application's security context.
- **Block executable file types.** Reject: `.exe`, `.sh`, `.php`,
  `.jsp`, `.py`, `.bat`, `.cmd`, `.ps1`, `.msi`, `.dll`, `.so`.
  Reject double extensions: `file.jpg.php`, `file.png.exe`.
- **Re-encode images.** Process uploaded images through a library
  (sharp, Jimp) to strip metadata and prevent image-based exploits.
  This also strips EXIF data that may contain GPS coordinates (PII).
- **Set file size limits per upload type.** Images: ≤10MB. Documents:
  ≤50MB. Adjust per project. Enforce server-side, not just client-side.

## Preview / Draft Security

- **Preview tokens are credentials.** Store in environment variables,
  not hardcoded in source. Rotate regularly. A leaked preview token
  exposes all unpublished content.
- **Restrict preview mode to authenticated CMS users.** Don't rely
  on token-only access. Verify the requesting user has CMS editor
  permissions before serving draft content.
- **Draft content must not leak into public responses.** Verify that
  API endpoints serving public content filter by `_status: published`
  (or equivalent). A missing filter exposes all drafts to the public
  API.
- **Preview URLs should expire.** If sharing preview links with
  stakeholders, use time-limited tokens (≤24 hours).

## Webhook Security

- **Verify webhook signatures before processing.** Every incoming
  webhook must be verified against a shared secret using HMAC.
  Reject payloads with invalid or missing signatures.
- **Webhook replay prevention.** Verify the timestamp in the webhook
  payload (most CMS platforms include one in the signed data). Reject
  payloads older than 5 minutes. Without timestamp checking, an
  attacker who intercepts a valid signed payload can replay it
  indefinitely.
- **Webhook secrets must differ per environment.** Production,
  staging, and development must use separate webhook secrets.
  A compromised staging secret must not grant access to production
  webhooks.
- **Canary test for webhook verification.** Include a test that sends
  a payload with an invalid signature and asserts the endpoint
  returns 401/403.
- **Webhook endpoints must be idempotent.** CMS platforms may retry
  failed webhook deliveries. Processing the same webhook twice must
  not cause duplicate operations.

## Content Modeling Security

- **Sanitize all CMS content before rendering on the frontend.**
  Rich text fields from the CMS can contain XSS if the CMS allows
  raw HTML input. Use DOMPurify or equivalent on the frontend
  regardless of CMS-side validation.
- **Markdown injection.** Many CMS platforms store content as Markdown.
  Markdown renderers often support raw HTML blocks — a content editor
  entering `<img src=x onerror=alert(1)>` in a Markdown field bypasses
  Markdown-level sanitization. Configure the Markdown renderer to
  disable raw HTML (`sanitize: true` or equivalent), OR sanitize the
  rendered HTML output with DOMPurify before display.
- **Validate content field lengths and formats.** A CMS field
  configured to accept "any text" can receive megabytes of content,
  script tags, or binary data. Set validation rules in the CMS
  content model, not just the frontend.

## Multi-Environment Deployment

- **Environment variable isolation.** CMS API keys, webhook secrets,
  and admin credentials must differ between environments. Never share
  a production CMS API key with staging.
- **SSG build-time API keys.** Static site generation fetches content
  at build time using a CMS API key. This key must have read-only
  permissions scoped to published content only (separate from preview/
  admin keys). Verify build logs don't expose the key. Never include
  CMS API keys in source maps or client bundles.
- **Database isolation.** Production and staging must use separate
  databases. A staging CMS writing to the production database is a
  data integrity risk.
- **Deployment checklist.** Before promoting to production: verify
  admin credentials are not defaults, preview tokens are rotated,
  webhook secrets are environment-specific, and the admin panel is
  access-restricted.

## User-Submitted Content Moderation

If the application accepts user-submitted content (comments, reviews,
forum posts, profile bios):
- **Content moderation is a product decision, not just a security
  control.** XSS sanitization (covered above) prevents code execution.
  Content moderation prevents hate speech, spam, CSAM, and legally
  actionable content. Both are required.
- **Implement a moderation pipeline:** automated pre-screening
  (profanity filter, spam detection), human review queue for flagged
  content, user reporting mechanism, and a legal-obligation removal
  path (DMCA, court orders, CSAM reporting).
- **Document the moderation policy in SPEC.md** — what's allowed,
  what's rejected, what requires human review. This decision has
  legal implications beyond the development scope.

## Third-Party Script Integrity

Every external script loaded by the CMS-rendered site is a trust
delegation. Vendor compromise, CDN takeover, or DNS hijacking results
in attacker JavaScript executing on every page — including admin panel
views where editor sessions and content drafts can be exfiltrated.

- **Prefer self-hosting.** Copy the vendor's script into the project's
  static assets, review it, version it. Updates become opt-in reviews.
  This is the strongest defense and the recommended default.
- **Subresource Integrity (SRI) on every external script tag.** If
  the vendor changes the file — legitimately or because of a CDN
  takeover — the hash mismatches and the script does not load.
- **Content Security Policy restricting script sources.** Set a
  `script-src` directive with an explicit allowlist of trusted origins.
- **Vendor dependency inventory.** Every third-party script is listed
  in REGISTRY.md under External Integrations with: vendor name, what
  it does, what data it can access, SRI hash, date added, date last
  reviewed. Quarterly review as part of the health check.
- **Vendor lifecycle awareness.** When a vendor shuts down, is
  acquired, or sunsets a product, remove their script within the
  deprecation window — an abandoned vendor CDN with a dangling
  CNAME is claimable by an attacker and becomes an XSS vector on
  every page that still references it.
