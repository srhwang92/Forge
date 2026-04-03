---
description: Frontend architecture, design token, accessibility, performance, and component standards. Loads automatically when working on frontend files.
globs: ["*.tsx", "*.jsx", "*.ts", "*.css", "*.scss", "*.html", "*.liquid", "*.vue", "*.svelte"]
---

# Frontend Engineering Standards

Production-grade rules. Every rule prevents a documented failure mode in
AI-generated frontend code.

**DESIGN.md is the source of truth.** If a project has a `DESIGN.md`, every
token, color, font, spacing value, and layout pattern in this file must
align with it. Where `DESIGN.md` specifies a value, use that value — not a
generic default from these rules. These rules govern *how* to implement;
`DESIGN.md` governs *what* to implement.

---

## TypeScript

- `strict: true` — no exceptions (`noImplicitAny`, `strictNullChecks`,
  `exactOptionalPropertyTypes`).
- Never use `any`. Use `unknown` + type guards, or define the actual type.
- No `@ts-ignore` or `@ts-expect-error` without a comment explaining why
  and a TODO to remove it.
- No enums. Use `as const` objects with derived union types instead —
  enums have runtime behavior and break tree-shaking.
- No `as` type assertions unless annotated with why — prefer type
  narrowing with guards.
- Runtime validation at every API boundary (zod, valibot, or tRPC).
- Derive types from schemas and contracts — never hand-write duplicates.

---

## Design Token Architecture (W3C 3-Tier Model)

- **Tier 1 — Global (primitives):** `color.blue.500`, `spacing.04`
  NEVER use directly in components.
- **Tier 2 — Semantic (aliases):** `color.text.action`, `spacing.gap.card`
  Primary consumption layer. Theming (dark mode, rebrand) executes here.
- **Tier 3 — Component (overrides):** `button.primary.bg.hover`
  Use sparingly — only when deviating from semantic rules.

### Spacing Scale

Define a base unit (4px or 8px per `DESIGN.md`) and derive all spacing
from it. Every margin, padding, and gap must be a multiple of the base
unit. Never use arbitrary pixel values.
Example: `--space-1: 4px`, `--space-2: 8px`, `--space-4: 16px`, etc.

### Z-Index Scale

Define z-index tokens in the design system — never use arbitrary values.
Example scale: `--z-dropdown: 100`, `--z-sticky: 200`, `--z-modal: 300`,
`--z-popover: 400`, `--z-toast: 500`, `--z-tooltip: 600`.
Never use `z-index: 999` or `9999`. If a value isn't in the scale, add
it to the scale — don't bypass with magic numbers.

### Naming (CTI Methodology)

Formula: `[Category]-[Type]-[Item]-[State/Variant]`
- Good: `--color-bg-surface-secondary`
- Bad: `--secondary-bg` (vague), `--h1-size` (locked to HTML tag —
  use `--font-size-heading-hero`)

### Theming & Dark Mode

- Theming operates at the semantic token tier — swap aliases, never
  hardcode alternate color values
- `<meta name="color-scheme" content="light dark">` in document head
- Detect system preference with `prefers-color-scheme`
- Every color token must define both light and dark values
- Test dark mode independently: verify contrast ratios, check for
  hardcoded colors that bypass tokens, confirm images/icons remain
  visible, and test all interactive states (hover, focus, active)

---

## CSS Architecture (ITCSS + @layer)

Cascade layers in ascending specificity:
`@layer reset, tokens, base, components, utilities;`

- `oklch()` for perceptual color spaces (uniform lightness across hues)
- `@property` for typed custom properties when tokens need animated
  transitions (e.g., color interpolation in oklch)
- CUBE CSS: utilities for **layout only**, components for aesthetics
- Never use utility classes for complex visuals (gradients, shadows,
  borders of a button) — those belong in `@layer components`
- When using Tailwind: extend theme from design tokens, never use
  arbitrary values (`bg-[#3b82f6]`) — use token-backed classes

### Style Scoping

Pick one scoping strategy per project and enforce it consistently:
- **CSS Modules** (`.module.css`) for component-scoped styles
- **Tailwind** for utility-first with token-backed theme
- **Vanilla CSS in `@layer components`** for design system primitives
Never mix strategies within the same component. Never use inline
`style={{}}` props except for truly dynamic computed values (e.g.,
`style={{ '--progress': `${percent}%` }}` for CSS variable injection).

### Layout

- **CSS Grid** for two-dimensional layouts (page structure, dashboards,
  card grids, complex arrangements)
- **Flexbox** for one-dimensional layouts (nav bars, button groups,
  inline alignment within components)
- Never use Flexbox hacks for grid layouts. If you need rows AND columns,
  use Grid.

### Fluid & Responsive Design

- `clamp()` over rigid breakpoints for typography and spacing:
  `--font-body: clamp(1rem, 0.5rem + 2vw, 1.25rem);`
- **Never use `px` for font sizes.** Use `rem` (or token-backed `clamp`)
  so text scales with browser zoom and user font preferences. AI
  defaults to px — users who increase browser font size see no change,
  which is an accessibility failure.
- **One icon library per project.** If DESIGN.md specifies Lucide, use
  Lucide everywhere. AI mixes Lucide, Heroicons, and Font Awesome in
  the same project — it looks inconsistent and inflates the bundle.
- Container queries (`@container`) for components in multiple layout
  contexts (sidebars, modals, main content)
- Media queries only for true viewport-level layout shifts
- Mobile-first: base styles target small screens, layer up with
  `min-width` queries

---

## Accessibility (WCAG 2.2 AA — Non-Negotiable)

Legal requirement (EU Accessibility Act, ADA, AODA). Built into every
component from the start — never retrofitted.

- **`<html lang="xx">`** — always set the document language attribute
- **Skip navigation.** `<a href="#main-content">Skip to content</a>` as
  the first focusable element on every page
- **Semantic HTML first.** `<button>`, `<nav>`, `<main>`, `<dialog>` —
  never `<div onClick>` or `<span role="button">`
- **Heading hierarchy.** Single `<h1>` per page, no skipped levels
- **Keyboard.** Every interactive element reachable and operable via
  keyboard. Visible focus indicators (≥2px outline, ≥3:1 contrast).
  `tabindex="0"` only on elements that need it; never positive tabindex.
- **Focus trapping.** Modals, dialogs, and overlays must trap focus inside
  until dismissed. Use Headless UI's built-in traps — never build custom
  focus trapping unless no library option exists.
- **ARIA.** Only when HTML semantics are insufficient. Never use ARIA to
  fix what a native element handles. If you add `role`, also add the
  required keyboard interaction pattern.
- **Live regions.** Use `aria-live="polite"` for dynamic content updates
  (toast notifications, form results, real-time data). Use `"assertive"`
  only for urgent errors.
- **Contrast.** Normal text ≥4.5:1, large text ≥3:1. Verify with oklch
  lightness values when generating palettes.
- **Touch targets.** Minimum 44×44px on all interactive elements
- **Form labels.** Visible `<label htmlFor>` on every input. Placeholder
  is not a label. Errors linked with `aria-describedby`.
- **Alt text.** Descriptive on informational images, `alt=""` on
  decorative. Never omit the attribute.
- **Motion.** Respect `prefers-reduced-motion`. No auto-play animation.
  Compositor-friendly only (`transform`, `opacity`).
- **SPA navigation focus.** After a client-side route change, move focus
  to the main content area or the new page's `<h1>`. Without this,
  screen reader users don't know the page changed. Announce the new
  page title via `aria-live` or document.title update.
- **Testing.** axe-core in CI on every PR. Screen reader test (VoiceOver
  or NVDA) on critical flows before feature completion.

---

## Frontend Security

- Never use `innerHTML` or `dangerouslySetInnerHTML` without explicit
  sanitization (DOMPurify). Flag for the project lead's review.
- Sanitize all user-generated content before rendering — even in
  attributes, URLs, and CSS values
- Never construct URLs from user input without validation
- CSP (Content Security Policy) headers: no `unsafe-inline` or
  `unsafe-eval` in production
- External links with `target="_blank"` must include
  `rel="noopener noreferrer"`
- Third-party scripts: audit before adding, load via `async`/`defer`,
  prefer self-hosted over CDN when possible
- **SRI (Subresource Integrity):** if loading scripts or styles from a
  CDN, add `integrity` hashes to prevent execution of tampered resources
- Never store sensitive data in localStorage — use httpOnly cookies or
  server-side sessions
- **Source maps:** upload to error tracking service (Sentry) during build.
  Never deploy source maps to public-facing URLs — delete from output
  after upload.

---

## Performance (Core Web Vitals)

Targets: **INP ≤200ms** (p75), **LCP ≤2.5s**, **CLS ≤0.1**

- **JS budget:** ≤400KB gzipped for interactive pages
- **Server Components default.** Only `"use client"` when the component
  genuinely needs browser APIs, event handlers, or state.
- **Code splitting.** Route-level automatic. Lazy-load heavy components
  below the fold (`React.lazy` + `Suspense`).
- **No waterfall fetches.** Parallel data fetching on the server.
- **Images — above fold:** `fetchpriority="high"` and `loading="eager"`
  on the LCP image. Use `next/image` with `priority` prop.
- **Images — below fold:** `loading="lazy"`, WebP/AVIF with fallback,
  explicit `width`/`height` (prevents CLS), responsive `srcset`/`sizes`.
- **Fonts:** `font-display: swap`, preload critical fonts, subset.
  Never block rendering on font load.
- **No main-thread blocking.** Tasks >50ms kill INP. Web workers for
  heavy computation, `requestIdleCallback` for non-critical work.
- **No layout thrashing.** Batch DOM reads before writes. Never
  interleave `getBoundingClientRect()` / `offsetHeight` reads with
  style mutations in the same frame.
- **Third-party resources:** `<link rel="preconnect">` for critical
  external origins. Load non-critical scripts with `defer`. Audit
  third-party impact on bundle size.
- **Long lists.** Virtualize any list rendering 100+ items (TanStack
  Virtual, react-window). Never render thousands of DOM nodes.
- **Bundle analysis.** Run a bundle analyzer (`@next/bundle-analyzer`,
  `rollup-plugin-visualizer`, or `source-map-explorer`) before deploy.
  Track bundle size in CI — fail the build if it exceeds the budget.
- **Prefetching.** Use framework-native link prefetching (Next.js `<Link>`
  auto-prefetches on viewport/hover). Never disable it without reason.
  For custom navigation, add `<link rel="prefetch">` for high-probability
  next pages.
- **`content-visibility: auto`** on long-page sections that start
  off-screen (below fold content, long lists, FAQ sections). Saves
  significant rendering cost on initial load.
- **SVG discipline.** Never paste raw SVG markup exceeding ~10 lines
  inline in JSX — extract to a component or use an SVG sprite system.
  Use component imports (`import { Icon } from './icons'`) for styling
  control. Use `<img>` only for complex illustrations that don't need
  CSS styling.

### Import Discipline

- Named imports only — never `import * as` or default-import entire
  libraries. Bad: `import _ from 'lodash'`. Good: `import { debounce }
  from 'lodash-es/debounce'`.
- **Path alias consistency.** If the project defines a path alias
  (`@/`, `~/`), use it for all cross-directory imports. Never mix
  relative (`../../components/Button`) and alias (`@/components/Button`)
  imports in the same file or across the same project.
- Audit barrel files (`index.ts` re-exports) — they can break
  tree-shaking. Only re-export what's part of the public API.
- Prefer packages that ship ESM and support tree-shaking.

---

## Component Architecture

- **Headless UI** (Radix, React Aria) for accessible primitives — never
  custom-reimplement dropdowns, modals, focus traps, keyboard nav.
  **Non-React environments (Shopify Liquid, vanilla JS, Astro):** when
  headless libraries aren't available, implement following WAI-ARIA
  Authoring Practices. Verify keyboard navigation, focus trapping,
  escape-to-close, and screen reader announcements manually.
- **Compound components:** `<Card.Header>`, `<Card.Body>`, `<Card.Footer>`
  — not massive prop objects
- **Strict APIs:** predefined variants (`variant="destructive"`), no
  escape-hatch `className`/`style` on core components unless approved
- **Composition over configuration**, always
- **Ref forwarding:** all leaf components that render DOM elements must
  forward refs via `React.forwardRef` or the `ref` prop (React 19+)
- **Named components:** use named function declarations or set
  `displayName` on arrow functions — anonymous components break React
  DevTools and error stack traces
- **Key props:** always use stable, unique business IDs (`key={user.id}`).
  Never use array index as key (`key={index}`) — it causes silent bugs
  when items are reordered, inserted, or deleted.
- **Conditional rendering:** use `{condition && <Component />}` or
  ternary expressions. Never render a component and hide it with CSS
  (`display: none`) — it wastes DOM nodes, runs effects, and can cause
  layout shifts. If it shouldn't be visible, don't render it.

### Error Handling, Loading & Empty States

Every data-dependent boundary handles four states:
- **Loading:** skeleton/shimmer for every async component. Never a blank
  screen or a raw spinner with no context.
- **Success:** the intended UI
- **Empty:** a purposeful empty state with illustration or message and a
  clear CTA ("No results found — try adjusting your filters"). Never
  render nothing.
- **Error:** nested error boundaries — not one at root level. AI puts a
  single boundary at the app root so one crashing widget kills the page.
  Use route-level (`error.tsx`) for page crashes plus feature-level
  boundaries so a broken chart widget shows a fallback without taking
  down the dashboard. Friendly message with retry action — never raw
  error strings. Client-side errors reported to Sentry or equivalent.
  Non-critical failures degrade gracefully — don't take down the page.

### Data Fetching

- Server Components for initial data — fetch server-side, not client
- TanStack Query or SWR for client-side server state — never raw
  `useEffect` + `fetch` + `useState`
- Abort pending requests on unmount — use `AbortController` with fetch
  or the built-in cancellation in TanStack Query/SWR
- Distinguish server state (fetched) from client state (UI). Never
  duplicate server data into client stores.
- Define caching strategy per query: staleTime, revalidation interval,
  and cache invalidation triggers
- Use optimistic updates for user actions where latency matters (likes,
  toggles, form submissions) — rollback on failure

### State Management

- Simplest option that works: component state → context → external store
- **Never derive state with `useEffect`.** If state B depends on state A,
  compute B inline or with `useMemo` — not with a `useEffect` that
  watches A and calls `setB`. This causes extra renders and race
  conditions.
- **Never fire API calls from `useEffect` without stable dependencies.**
  Object/array literals and unstable function references in the
  dependency array cause infinite request loops. Use TanStack Query or
  stable references. If you see the same request firing repeatedly in
  the network tab, the dependency array is wrong.
- **Every `useEffect` with a subscription, timer, or listener must
  return a cleanup function.** Missing cleanup causes memory leaks
  and stale callback bugs that only surface after navigation.
- **URL state** for anything shareable or bookmarkable: filters,
  pagination, active tabs, search queries. Use `useSearchParams` or
  equivalent — never `useState` for URL-worthy state.
- Zustand for client state when context isn't sufficient
- Solve prop drilling with composition or compound components, not
  global state

---

## Forms

- Use a form library (react-hook-form preferred) for any form with more
  than 2 fields. Don't hand-roll form state with `useState`.
- Add `noValidate` to `<form>` when using custom validation — prevents
  browser default validation UI from conflicting with custom errors.
- Visible `<label htmlFor>` on every input. Placeholder ≠ label.
- `autocomplete` attributes on all identity/address/payment fields
- Progressive validation: blur → change-after-error → submit
- Errors inline, linked to input via `aria-describedby`
- Complex forms (5+ fields): add an error summary at the top on submit
  that links to each errored field
- Enter key submits. Disable button during async, show loading state.
- Multi-step forms: persist state between steps, allow back navigation,
  show progress indicator
- **Debounce server validation.** For real-time checks (username/email
  availability), debounce 300-500ms. Never fire a request on every
  keystroke.
- **Controlled or uncontrolled — pick one per form, never mix.** Use
  controlled (`value` + `onChange`) with react-hook-form's `Controller`
  or uncontrolled (`register` + refs). Mixing causes React warnings and
  state desync bugs.

---

## SEO

- Unique `<title>` and `<meta description>` per page
- Single `<h1>`, heading hierarchy never skips levels
- JSON-LD structured data for entities (products, articles, FAQs)
- Server-render content pages — no client-only rendering for indexable
  content
- Open Graph + Twitter Card meta on shareable pages
- **Verify all meta tags are project-specific.** AI copies site names,
  OG titles, descriptions, and social images from templates or previous
  projects without updating them. Before deploy, audit every page's
  `<title>`, `meta description`, `og:title`, `og:image`, and
  `og:description` for accuracy.
- Canonical URLs to prevent duplicate content
- Generate `sitemap.xml` and `robots.txt` for every public-facing project
- Favicon set (at minimum: `favicon.ico`, `apple-touch-icon.png`,
  `site.webmanifest`) for every public-facing project

---

## Animation & Motion

- `prefers-reduced-motion: reduce` disables or minimizes all animation
- Animate `transform` and `opacity` only — never `width`, `height`,
  `top`, `left`, `margin`, `padding`
- Use `will-change` sparingly and only on elements that are about to
  animate — never as a permanent style
- Motion communicates meaning (state changes, feedback, spatial
  relationships). If removing it loses no information, question it.
- CSS transitions for simple states. Library (Framer Motion, Motion One)
  only for complex orchestration.

---

## Internationalization Readiness

Even for English-only projects, never create i18n debt:
- Never hardcode date, number, or currency formatting — use `Intl`
  APIs (`Intl.DateTimeFormat`, `Intl.NumberFormat`, `Intl.RelativeTimeFormat`)
- Never hardcode pluralization rules — use `Intl.PluralRules`
- Never concatenate user-visible strings — use template patterns that
  a translator can rearrange
- Support RTL layouts if the project may serve RTL languages: use
  CSS logical properties (`margin-inline-start` not `margin-left`)

---

## Testing

- Component tests with Testing Library (behavior, not implementation)
- axe-core in CI on every PR
- Playwright E2E for critical flows (auth, checkout, onboarding)
- Visual regression (Playwright screenshot comparison or Chromatic) for
  design system components — catches unintended visual changes
- No snapshot tests as primary strategy — they verify nothing behavioral
- Test error states, loading states, and empty states — not just happy
  paths
- Mock the network boundary only (MSW) — never mock internal modules
  or component internals

---

## File Organization

- Feature-based structure: `/features/auth/`, `/features/dashboard/`
- Co-locate component, test, styles, types in same directory
- Public API via `index.ts` barrel per feature
- Shared UI in `/components/ui/` (the design system)
- Max 3 levels of nesting
- Naming: PascalCase for components (`UserCard.tsx`), camelCase for
  utils/hooks (`useAuth.ts`, `formatDate.ts`), kebab-case for CSS
  files (`user-card.module.css`)

---

## Shopify-Specific

- Never edit Dawn's original theme files
- CSS overrides and standalone `.liquid` files only
- Reference `DESIGN.md` tokens via CSS custom properties in theme CSS
- Schema settings: clear labels, default values, info text
- **Testing via Playwright MCP.** Run against `shopify theme dev` preview:
  axe-core accessibility audit on every page template, screenshot
  comparison across breakpoints (mobile, tablet, desktop), verify no
  console errors, and test critical flows (add to cart, newsletter
  signup, navigation between brand worlds)
