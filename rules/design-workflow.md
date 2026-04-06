---
description: Design system workflow — DESIGN.md sourcing, token extraction, [VERIFY] markers, skill priority. Loaded when working on UI files. CLAUDE.md has a short pointer.
globs: ["**/*.{tsx,jsx,vue,svelte,astro}", "**/*.{css,scss,sass,less}", "**/tailwind.config.*", "**/theme.{ts,js}", "**/DESIGN.md", "**/design/**", "**/components/**"]
---

# Design Workflow

> Loaded on UI work via glob. Global CLAUDE.md has a 4-line pointer:
> "Before UI work, check DESIGN.md; see `rules/design-workflow.md`
> for extraction, [VERIFY] handling, and skill priority."

## DESIGN.md as Source of Truth

**Before any UI work, check for `DESIGN.md` in the project root.**
If it exists, it is the source of truth. Every visual value
(colors, spacing, typography, shadows, radii, motion) must conform
to the tokens defined there. Never deviate without approval.

A `DESIGN.md` that doesn't cover a needed value is a gap to fill
by asking the project lead, not a license to freestyle.

## No DESIGN.md Yet — Creating One

If no `DESIGN.md` exists and UI decisions are needed, create from
`~/.claude/templates/design.md`:

- **Figma file available** → extract tokens via Figma MCP. This is
  the preferred source — designed values, verified by the designer.
- **No Figma** → generate from `templates/design.md` with `[VERIFY]`
  markers on every auto-populated value. The template populates
  reasonable defaults; the project lead reviews and removes the
  markers before implementation begins.
- **No user-facing UI** → skip DESIGN.md entirely.

### Design Defaults Exception

For projects using well-known design systems, populate DESIGN.md
with the system's published values **without** `[VERIFY]` markers:
- **shadcn/ui** — use the shipped Tailwind tokens and component
  defaults
- **Tailwind defaults** — use the framework's default theme values
- **Material UI** — use MUI's published theme tokens
- **Radix UI + custom styling** — use Radix structural tokens

Only mark custom or project-specific values with `[VERIFY]`. This
prevents workflow deadlock on projects using standard systems —
forcing `[VERIFY]` on shadcn's neutral-50 color is paperwork
without value.

## `[VERIFY]` Marker Rule

**Never use `[VERIFY]`-tagged values in implementation.** The
project lead must review and remove them first. If you find yourself
reading a `[VERIFY]` token and copying it into code, stop and ask
the project lead to confirm or correct the value. This is a Tier A
gate.

Mobile/React Native projects use JS objects instead of CSS variables
for tokens — see `templates/guardrails/mobile-app.md` § Styling &
Design Tokens.

## UI Skill Priority

When multiple design/UI skills are enabled in the project, lower
number wins (project-specific guidance overrides generic guidance):

1. **DESIGN.md** — project-specific tokens, always highest priority
2. **ui-ux-pro-max** — project-enabled skill
3. **frontend-design** — project-enabled skill
4. **Vercel web-design-guidelines** — general framework
5. **Vercel react-best-practices** — general framework
6. **Vercel composition-patterns** — general framework

**Stitch MCP and Figma MCP** run only when the project CLAUDE.md
explicitly enables them. Default off.

## Common Failure Modes

- **Hardcoded colors** — every `#fff`, `rgb(...)`, or `hsl(...)` in
  a component is a token violation. Use the DESIGN.md token name.
- **Inline spacing** — `margin: 16px` is a violation. Use the
  spacing token.
- **Ad-hoc shadows** — if a shadow isn't in DESIGN.md, don't invent
  one. Ask.
- **Mixing systems** — using shadcn tokens alongside custom tokens
  without a clear boundary creates visual drift.
- **Typography scale drift** — sticking `text-[13px]` in JSX because
  the design needs "something between text-xs and text-sm" is the
  exact pattern DESIGN.md exists to prevent.
