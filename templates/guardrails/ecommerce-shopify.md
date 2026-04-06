# Project Guardrails — Ecommerce / Shopify

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for ecommerce:
- PCI-DSS if handling payment cards (prefer Shopify Payments / Stripe)
- Consumer protection laws (Competition Act, FTC Act)
- GDPR, CCPA/CPRA if serving EU or California customers
- Accessibility: ADA, AODA, EAA for public-facing storefronts

## Shopify Platform Rules

- **Never edit Dawn's original theme files.** CSS overrides and
  standalone `.liquid` files only.
- Never modify `theme.liquid` base structure without approval
- Schema settings must have clear labels, default values, and info text
- Test all changes in theme preview before publishing
- Never use Shopify admin API keys in client-side code
- Respect Shopify's rate limits — check current limits via Context7 or
  Shopify docs before any bulk operations

## Product & Pricing Safety

- **Never hardcode prices, discounts, or tax rates.** These must come
  from Shopify's admin, metafields, or configuration.
- Product descriptions must be accurate. Never generate claims about
  product efficacy, origin, or composition without the project lead's verification.
- Never generate fake reviews, ratings, or social proof.
- Sale/discount copy must accurately reflect the actual price reduction.

## Payment Safety

- Never store, log, or transmit credit card numbers. Shopify Payments
  and Stripe handle this — keep card data off the codebase entirely.
- Checkout modifications: flag any change that affects the purchase
  flow, cart calculations, or payment processing.
- **Shopify Functions** (checkout, delivery, payment customization)
  run in a sandboxed WASM environment with their own API contract.
  Test Functions independently: mock the Function API input, verify
  output against known-answer cases (negative quantities, coupon
  stacking, currency conversion, zero-price items). Use the Shopify
  CLI `function run` or Vitest with function-runner for unit tests.
- Shipping rate calculations must be verified with known-answer pairs.

## Email / Marketing Compliance

Specific regulations vary by jurisdiction (CASL in Canada, CAN-SPAM in
US, GDPR in EU). These rules satisfy the strictest common requirements:

- **Express consent required** before adding anyone to a mailing list.
  No pre-checked opt-in boxes. No "by purchasing you agree to receive
  marketing" without an explicit, separate opt-in.
- Every marketing email must include sender identification, mailing
  address, and a working unsubscribe mechanism.
- Transactional emails (order confirmation, shipping) are generally
  exempt from consent requirements but must not contain marketing content.

## Image & Asset Management

- Optimize all images before upload (WebP/AVIF, max 500KB for product
  images, responsive srcset)
- Never upload images with embedded metadata containing PII
- Alt text on every product image (accessibility + SEO)

## Third-Party Script Integrity

Every external script loaded by the storefront is a trust delegation.
The vendor's security becomes your security. Vendor compromise, CDN
takeover, and DNS hijacking all result in attacker JavaScript executing
on the site — including checkout pages where card data flows.

- **Prefer self-hosting.** Copy the vendor's script into the theme's
  assets folder, review it, version it, and serve from the theme's
  own domain. Updates become opt-in reviews. This is the strongest
  defense and the recommended default for any third-party script on
  a commerce site.
- **Subresource Integrity (SRI) on every external script tag.**
  `<script src="..." integrity="sha384-..." crossorigin="anonymous">`.
  If the vendor changes the file — legitimately or because of a CDN
  takeover — the hash mismatches and the script does not load. Fail
  loud.
- **Content Security Policy restricting script sources.** Set
  `Content-Security-Policy` with a `script-src` directive listing
  only trusted origins. Shopify supports setting CSP via a theme
  meta tag or response header depending on theme architecture.
- **Vendor dependency inventory.** Every third-party script is listed
  in REGISTRY.md under External Integrations with: vendor name, what
  it does, what data it can access, SRI hash, date added, date last
  reviewed. Quarterly review is part of the health check.
- **DNS monitoring on vendor subdomains.** For critical vendor scripts,
  monitor the vendor's domain for DNS changes that indicate provider
  migration or takeover. A cron job resolving the CNAME and diffing
  against a known value is the minimum viable implementation.
- **Vendor lifecycle awareness.** When a vendor is acquired, shuts
  down, or sunsets a product, remove their script from production
  within the deprecation window — not after. An abandoned vendor
  CDN is a ticking bomb: the CNAME eventually dangles, an attacker
  claims it, and the vendor's old customers serve attacker
  JavaScript on every page view.
