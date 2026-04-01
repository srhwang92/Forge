# Project: [NAME]

> [One-line description of what this project is and who it's for]

**Type:** [website | webapp | mobile app | Shopify store | dashboard |
internal tool | API/backend | automation | ML/AI | agent | CLI tool]
**Client:** [client name or "Nura (internal)"]
**Project Lead:** [Rocky, unless another lead is specified]
**Has DESIGN.md:** [yes | no — auto-populated during project setup]

---

## Stack

- **Framework:** [e.g., Next.js 15, Remix, Astro, React Native, Django]
- **Language:** [e.g., TypeScript, Python 3.12]
- **Runtime:** [e.g., Node 22, Bun 1.x, Python 3.12, Deno]
- **Package Manager:** [npm | pnpm | yarn | bun | uv | pip]
- **Database:** [e.g., Supabase (PostgreSQL), PlanetScale, MongoDB, none]
- **Auth:** [e.g., Supabase Auth, Clerk, Auth.js, NextAuth, Firebase Auth, none]
- **Hosting:** [e.g., Vercel, Railway, Shopify, Netlify, Expo EAS]
- **Key Libraries:** [e.g., shadcn/ui, TanStack Query, Prisma, Drizzle]
- **Styling:** [e.g., Tailwind, CSS Modules, vanilla CSS + @layer, NativeWind]
- **Test Runner:** [e.g., Vitest, Jest, Playwright, pytest, none yet]
- **Path Alias:** [e.g., `@/` → `src/`, `~/` → `src/`, none]

## Commands

```
dev:      [e.g., npm run dev | shopify theme dev | expo start]
build:    [e.g., npm run build | shopify theme package]
test:     [e.g., npx vitest | pytest | npm test]
lint:     [e.g., npm run lint | ruff check .]
format:   [e.g., npm run format | ruff format .]
typecheck:[e.g., npx tsc --noEmit | mypy . | n/a]
```

## Key Directories

```
[e.g., src/              — source code root]
[e.g., src/features/     — feature modules]
[e.g., src/components/   — shared UI components]
[e.g., tests/            — test files (if not co-located)]
[e.g., public/           — static assets]
```

## Naming Conventions

[If different from global defaults (PascalCase components, camelCase
utils/hooks, kebab-case CSS, SCREAMING_SNAKE constants), specify here.
e.g., "Python files use snake_case throughout" or "Shopify sections
use kebab-case liquid filenames"]

## Deployment

- **Platform:** [e.g., Vercel, Railway, Shopify]
- **Production URL:** [url or TBD]
- **Staging URL:** [url or TBD]
- **Env vars:** documented in `.env.example`
- See `README.md` for full deploy instructions.

---

## Active MCPs

[Remove this section if using global defaults only]
- Figma — [enabled/disabled]
- Stitch — [enabled/disabled]
- Playwright — [enabled/disabled]

## Third-Party Integrations

[Remove if none. List service + env vars needed]
- [e.g., Stripe — `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`]

## Known Constraints

[Remove if no special constraints beyond global defaults]
- **Browser support:** [e.g., last 2 versions, no IE11]
- **Device targets:** [e.g., desktop + mobile, mobile-only]
- **Performance budget:** [e.g., ≤400KB JS gzipped, LCP ≤2.5s]
- **Accessibility:** [e.g., WCAG 2.2 AA]

## Workflow Overrides

[Remove if using global Superpowers workflow. Examples: "Skip Layer 3",
"Use feature-dev instead of Superpowers", "Small codebase — no subagents"]

## Applied Guardrail Templates

[Which templates from ~/.claude/templates/guardrails/ were combined
into this project's GUARDRAILS.md]
- [e.g., saas-webapp.md]

---

## ⚠️ Things Claude Gets Wrong

> Populate during development. Every time Claude makes a project-specific
> mistake, add it here. Highest-ROI section — prevents repeat errors.
> **Prune periodically:** remove entries about replaced libraries,
> changed patterns, or fixed issues that no longer apply.
