# Design System — [PROJECT NAME]

> Single source of truth for all visual decisions. Every component, page,
> and style must conform to these tokens. Values marked `[VERIFY]` were
> auto-generated — review before UI work begins.

**Token format:** CSS custom properties (`--token-name: value`)
for web projects. **React Native / mobile projects:** tokens are
TypeScript constants in a shared `theme.ts` — numeric values with
no `px`/`em`/`rem` units (RN uses dp/pt natively). No CSS breakpoints;
use `Dimensions.get('window')` or `useWindowDimensions()`. See
`mobile-app.md` § Styling & Design Tokens.
**Naming:** CTI convention — `[Category]-[Type]-[Item]-[State/Variant]`
**Implementation:** [Tailwind config extends | CSS `@layer tokens` |
Shopify theme.css custom properties | RN `theme.ts` export]

---

## Typography

**Font Families:**
- Heading: `[VERIFY]` [e.g., "Inter", sans-serif]
- Body: `[VERIFY]` [e.g., "Inter", sans-serif]
- Mono: `[VERIFY]` [e.g., "JetBrains Mono", monospace]

**Type Scale:**
| Token                      | Size `[VERIFY]` | Weight | Line Height | Letter Spacing |
|----------------------------|-----------------|--------|-------------|----------------|
| `--font-size-heading-hero` |                 | 700    | 1.1         | -0.02em        |
| `--font-size-heading-1`    |                 | 700    | 1.2         | -0.015em       |
| `--font-size-heading-2`    |                 | 600    | 1.25        | -0.01em        |
| `--font-size-heading-3`    |                 | 600    | 1.3         | 0              |
| `--font-size-heading-4`    |                 | 600    | 1.35        | 0              |
| `--font-size-body`         |                 | 400    | 1.5         | 0              |
| `--font-size-body-sm`      |                 | 400    | 1.5         | 0.01em         |
| `--font-size-caption`      |                 | 400    | 1.4         | 0.01em         |

---

## Color Palette

### Primitives (Tier 1 — never use directly in components)

Use `oklch()` for perceptual uniformity.

| Token                | Value `[VERIFY]` |
|----------------------|-------------------|
| `--color-white`      |                   |
| `--color-black`      |                   |
| `--color-gray-50`    |                   |
| `--color-gray-100`   |                   |
| `--color-gray-200`   |                   |
| `--color-gray-300`   |                   |
| `--color-gray-500`   |                   |
| `--color-gray-700`   |                   |
| `--color-gray-900`   |                   |
| `--color-brand-50`   |                   |
| `--color-brand-100`  |                   |
| `--color-brand-500`  |                   |
| `--color-brand-700`  |                   |
| `--color-brand-900`  |                   |

### Semantic Tokens (Tier 2 — primary consumption layer)

| Token                          | Light `[VERIFY]`    | Dark `[VERIFY]`     |
|--------------------------------|---------------------|---------------------|
| `--color-text-primary`         |                     |                     |
| `--color-text-secondary`       |                     |                     |
| `--color-text-muted`           |                     |                     |
| `--color-text-inverse`         |                     |                     |
| `--color-text-action`          |                     |                     |
| `--color-text-link`            |                     |                     |
| `--color-text-link-hover`      |                     |                     |
| `--color-text-link-visited`    |                     |                     |
| `--color-bg-page`              |                     |                     |
| `--color-bg-surface`           |                     |                     |
| `--color-bg-surface-secondary` |                     |                     |
| `--color-bg-action`            |                     |                     |
| `--color-bg-action-hover`      |                     |                     |
| `--color-border-default`       |                     |                     |
| `--color-border-strong`        |                     |                     |
| `--color-border-focus`         |                     |                     |
| `--color-status-error`         |                     |                     |
| `--color-status-warning`       |                     |                     |
| `--color-status-success`       |                     |                     |
| `--color-status-info`          |                     |                     |

---

## Interactive States

Focus and interaction tokens. Must meet WCAG 2.2 AA requirements.

| Token                          | Value `[VERIFY]`                      |
|--------------------------------|---------------------------------------|
| `--focus-ring-width`           | 2px                                   |
| `--focus-ring-offset`          | 2px                                   |
| `--focus-ring-color`           | var(--color-border-focus)             |
| `--input-border`               | var(--color-border-default)           |
| `--input-border-hover`         | var(--color-border-strong)            |
| `--input-border-focus`         | var(--color-border-focus)             |
| `--input-border-error`         | var(--color-status-error)             |
| `--input-bg`                   | var(--color-bg-surface)               |
| `--input-text`                 | var(--color-text-primary)             |
| `--input-placeholder`          | var(--color-text-muted)               |

---

## Spacing

**Base unit:** `[VERIFY]` [4px or 8px]

| Token        | Value      |
|--------------|------------|
| `--space-0`  | 0          |
| `--space-1`  | 1 × base   |
| `--space-2`  | 2 × base   |
| `--space-3`  | 3 × base   |
| `--space-4`  | 4 × base   |
| `--space-6`  | 6 × base   |
| `--space-8`  | 8 × base   |
| `--space-10` | 10 × base  |
| `--space-12` | 12 × base  |
| `--space-16` | 16 × base  |
| `--space-20` | 20 × base  |
| `--space-24` | 24 × base  |

---

## Z-Index

| Token            | Value |
|------------------|-------|
| `--z-dropdown`   | 100   |
| `--z-sticky`     | 200   |
| `--z-modal`      | 300   |
| `--z-popover`    | 400   |
| `--z-toast`      | 500   |
| `--z-tooltip`    | 600   |

---

## Breakpoints

| Token     | Value `[VERIFY]` | Usage                          |
|-----------|-------------------|--------------------------------|
| `--bp-sm` | 640px             | Mobile landscape               |
| `--bp-md` | 768px             | Tablet                         |
| `--bp-lg` | 1024px            | Desktop                        |
| `--bp-xl` | 1280px            | Wide desktop                   |

---

## Border Radius

| Token          | Value `[VERIFY]` |
|----------------|-------------------|
| `--radius-none`| 0                 |
| `--radius-sm`  | 4px               |
| `--radius-md`  | 8px               |
| `--radius-lg`  | 12px              |
| `--radius-xl`  | 16px              |
| `--radius-full`| 9999px            |

---

## Shadows

| Token          | Value `[VERIFY]`                                    |
|----------------|-----------------------------------------------------|
| `--shadow-sm`  | 0 1px 2px 0 oklch(0% 0 0 / 0.05)                   |
| `--shadow-md`  | 0 4px 6px -1px oklch(0% 0 0 / 0.1)                 |
| `--shadow-lg`  | 0 10px 15px -3px oklch(0% 0 0 / 0.1)               |

---

## Motion

| Token                | Value `[VERIFY]` |
|----------------------|-------------------|
| `--duration-fast`    | 100ms             |
| `--duration-normal`  | 200ms             |
| `--duration-slow`    | 300ms             |
| `--duration-slower`  | 500ms             |
| `--ease-default`     | cubic-bezier(0.4, 0, 0.2, 1) |
| `--ease-enter`       | cubic-bezier(0, 0, 0.2, 1)   |
| `--ease-exit`        | cubic-bezier(0.4, 0, 1, 1)   |

All durations → 0ms when `prefers-reduced-motion: reduce`.

---

## Layout

- **Max content width:** `[VERIFY]` [e.g., 1280px]
- **Container padding:** `[VERIFY]` [e.g., --space-4 mobile, --space-8 desktop]
- **Grid system:** [12-column | auto-grid | none]
- **Grid gap:** `[VERIFY]` [e.g., --space-4 default, --space-6 desktop]

---

## Iconography

- **Library:** `[VERIFY]` [e.g., Lucide, Heroicons, Phosphor]
- **Sizes:** sm (16px), md (20px), lg (24px), xl (32px)
- **Stroke width:** `[VERIFY]` [e.g., 1.5px]

---

## Component Patterns

> Populated as components are built. Each entry defines Tier 3 token
> overrides for that component.

---

## Images

> Include for ecommerce, marketing, or image-heavy projects. Remove for
> dashboards, internal tools, or minimal-UI projects.

- **Aspect ratios:** `[VERIFY]` [e.g., 4:3, 16:9, 1:1]
- **Placeholder strategy:** [blur hash | solid color | skeleton shimmer]
- **Object-fit default:** [cover | contain]
- **Max file size:** [e.g., 500KB product, 200KB thumbnail]
