# Project Guardrails — Ecommerce / Shopify

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

## Applicable Regulations
See global Privacy & Regulatory Awareness for baseline (PIPEDA, PIPA BC,
CASL). Additionally for ecommerce:
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
- Shipping rate calculations must be verified with known-answer pairs.

## Email / Marketing (CASL Compliance)

- **Express consent required** before adding anyone to a mailing list.
  No pre-checked opt-in boxes. No "by purchasing you agree to receive
  marketing" without an explicit, separate opt-in.
- Every marketing email must include sender identification, mailing
  address, and a working unsubscribe mechanism.
- Transactional emails (order confirmation, shipping) are exempt from
  CASL consent requirements but must not contain marketing content.

## Image & Asset Management

- Optimize all images before upload (WebP/AVIF, max 500KB for product
  images, responsive srcset)
- Never upload images with embedded metadata containing PII
- Alt text on every product image (accessibility + SEO)
