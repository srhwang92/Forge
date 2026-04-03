# Project Guardrails — Static / Marketing / Client Websites

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

## Applicable Regulations
See global Privacy & Regulatory Awareness for baseline (PIPEDA, PIPA BC,
CASL). Additionally for static/marketing sites:
- GDPR if the site is accessible to / targets EU visitors
- Accessibility: ADA, AODA, EAA for public-facing websites
- Competition Act / FTC for advertising claims

## Content & Claims

- **Never generate testimonials, case studies, or client logos** without
  the project lead's verification that permission has been granted.
- **Never generate specific performance claims** ("increased revenue by
  X%," "Y% faster") without verified data backing them.
- **Copyright on all assets.** Verify that stock photos, fonts, icons,
  and illustrations have appropriate licenses for commercial use.
  Never use images from Google Image search without verifying license.
- **Client approval on all content.** For client sites, no content goes
  live without the client's explicit approval. Flag drafts clearly.

## Contact Forms & Data Collection

- **Every form that collects personal data needs a privacy notice.**
  At minimum: what data is collected, why, how it's stored, and how
  to request deletion. Link to full privacy policy.
- **Email collection requires CASL-compliant consent.** Separate opt-in
  for marketing emails. Express consent (not implied). Record the
  consent timestamp and method.
- **Never store form submissions in plaintext files or email.** Use a
  database or form service with appropriate access controls.
- Contact form submissions containing PII must be handled per the
  global Data Protection rules.

## Third-Party Integrations

- **Cookie consent required before loading tracking scripts.** No Google
  Analytics, Facebook Pixel, HotJar, or equivalent before the user
  consents.
- **Audit all third-party scripts.** Every external script is a security
  and privacy risk. Document what each script does, what data it
  collects, and confirm the vendor's data handling practices.
- Prefer self-hosted analytics (Plausible, Umami) over ad-tech trackers
  where possible.

## Performance

- Target Lighthouse score ≥90 on all four metrics (performance,
  accessibility, best practices, SEO).
- Static sites must be fast. Flag any change that adds >100KB to the
  total page weight (images, scripts, fonts).
- Use modern image formats (WebP/AVIF), responsive images, and lazy
  loading for all non-hero images.

## SEO

- Unique title and meta description per page
- Valid `robots.txt` and `sitemap.xml`
- Proper heading hierarchy (single h1, no skipped levels)
- Schema.org structured data for the business (LocalBusiness,
  Organization, Product, etc.)
- Canonical URLs on every page
- **Downloadable documents** (PDFs, whitepapers, case studies) must also
  be accessible — tagged PDFs with reading order, alt text, and proper
  structure. ADA lawsuits target inaccessible PDFs.
