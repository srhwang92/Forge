---
description: Code voice and engineering judgment. Combines elite human developer traits with AI consistency. Always active — governs how code reads, not just what it does.
globs: ["**"]
---

# Code Voice

Code must read like it was written by an opinionated senior engineer
who happens to never miss an edge case. Keep AI's rigor on the things
that matter. Adopt human voice on everything visible.

**The principle:** AI's strengths (security, accessibility, type safety,
test coverage, consistency) should be invisible — baked in, not
performed. Human strengths (judgment, pragmatism, concise voice, organic
structure) should be what a reviewer actually sees.

---

## Comments

Write comments like a senior dev: sparse, opinionated, only when
non-obvious.

- **Comment WHY, never WHAT.** The code says what. The comment says why
  it's surprising, non-obvious, or a deliberate trade-off.
- **Never write:** `// Initialize state`, `// Handle click`,
  `// Return result`, `// Import dependencies`, `// Define types`,
  `// Render component`. These say nothing.
- **Do write:** `// Safari ignores flex-basis in nested grids — force
  width instead`, `// Intentionally not awaited — fire and forget for
  analytics`, `// 30-day window matches PIPEDA breach notification`
- **Workaround comments are good.** Real codebases reference real bugs:
  `// Workaround: Radix Dialog focus trap conflicts with Framer Motion
  exit animation. Remove after radix-ui/primitives#2122 is fixed.`
- **No comment headers.** Don't write `// ===== SECTION: Authentication
  =====` or `// ---------- Helpers ----------`. Use code structure, not
  decorative comments.
- **If a block needs a comment to be understood, consider rewriting
  the block first.** Comments are a last resort, not a first instinct.

---

## Naming

Name things like a human who values their time. Concise, precise,
unambiguous — but never verbose.

- **Short names for short scopes.** Loop variable: `i`, `item`, `row`.
  Callback parameter: `e`, `err`, `res`. Map/filter: `x => x.active`.
  Don't write `currentIterationIndex` or `eventHandlerCallbackError`.
- **Descriptive names for wide scopes.** Module-level functions, exported
  APIs, shared types: be precise. `formatCurrency`, `parseWebhookPayload`,
  `UserPermissions`.
- **Contextual brevity.** Inside `UserCard`, a prop named `name` is
  fine — it doesn't need to be `userName`. Inside a generic utility,
  `userName` is correct because context is absent.
- **Avoid stutter.** `user.userName`, `config.configValue`,
  `ButtonButton` — never repeat the parent's name in the child.
- **Boolean names state truth.** `isLoading`, `hasPermission`,
  `canEdit`. Not `loading` (ambiguous — noun or adjective?), not
  `shouldBeLoading` (overly hedged).
- **Functions describe actions.** `getUser`, `createOrder`, `syncInventory`.
  Not `userData` (is it a function or a variable?), not
  `processAndValidateAndSaveUserFormSubmission` (just `submitForm`).
- **Name files for what they export.** `UserCard.tsx`, `useAuth.ts`,
  `formatDate.ts`. Not `utils.ts` (bag of unrelated functions) or
  `helpers.ts` (same problem, worse name).

---

## Structure & Variation

Real codebases have organic variation. Not every file looks the same.

- **Component size should vary.** A `Divider` is 3 lines. A `Dashboard`
  is 80 lines. Don't pad small components with unnecessary structure,
  and don't artificially split large components that have a single
  clear responsibility.
- **Not every component needs the same skeleton.** Skip sections that
  don't apply: if there are no early returns, don't add a guard clause
  "for consistency." If there's no loading state, don't add an empty
  one.
- **Flat is better than nested, until it isn't.** A 3-level ternary is
  worse than an if/else. A 5-condition if/else chain is worse than a
  lookup object. Choose the structure that's clearest for the specific
  situation, not the one that matches a pattern.
- **Inline small things.** A one-line utility used once doesn't need its
  own file. A two-field type used in one component doesn't need its own
  `types.ts`. Extract when something is reused or complex — not
  preemptively.
- **Allow asymmetry.** If a feature has 4 components and one of them is
  simpler than the others, let it be simpler. Don't add complexity for
  structural consistency.

---

## Pragmatism

Elite devs ship. They make deliberate trade-offs and document them.

- **Good enough is sometimes the right answer.** If a perfect abstraction
  takes 2 hours but a simple solution takes 10 minutes and the code
  will likely be rewritten when requirements clarify — ship the simple
  version. Leave a `// TODO: Revisit when X is clearer` if
  appropriate.
- **Not every edge case needs handling on day one.** If a feature is
  behind a feature flag and only the team is using it, a basic error
  message is fine. The production error boundary will catch the rest.
  Don't gold-plate internal-only paths.
- **Tactical debt is acceptable when documented.** A `// HACK:` comment
  explaining why and linking to a cleanup issue is better than a
  premature abstraction. The test: would a senior engineer approve this
  in code review with that comment? If yes, ship it.
- **Perfect is the enemy of shipped.** If everything is correct,
  accessible, secure, and tested — but could theoretically be 10%
  cleaner — ship it. Refactor in a separate PR when there's a reason.
- **Trust the type system.** If TypeScript says a value is `string`,
  don't add `if (typeof x === 'string')`. Don't null-check values the
  type guarantees aren't null. Unnecessary defensive checks signal
  distrust of the tooling — humans who use strict TS trust it.
- **Don't over-type.** `type Props = { name: string; avatar: string }`
  is fine. Don't create `interface IUserCardComponentProps` or
  `type TUserCardProps`. Match the team's conventions, not Java's.

---

## Error Handling With Judgment

Handle errors contextually, not uniformly.

- **Critical paths get thorough handling.** Payment processing, auth
  flows, data mutations — full error typing, user-friendly messages,
  retry logic, error tracking.
- **Non-critical paths get proportional handling.** A failed avatar
  upload? Show a toast and let the user retry. Don't build the same
  error recovery infrastructure as the payment flow.
- **Some errors should be caught and intentionally ignored (with a
  comment and optional log).** Analytics failures, optional enhancement
  features, non-critical background tasks — a `catch` with a comment
  explaining why it's safe to ignore is honest engineering. This IS
  handling — what's banned is an empty catch with no explanation.
- **Don't catch just to rethrow.** `try { x() } catch(e) { throw e }`
  is noise. Only catch if you're adding context, logging, translating,
  or recovering.

---

## Tests

Write tests like a human who wants to catch bugs, not like an AI
filling a template.

- **Short test names.** `it('renders user card')` not `it('should
  correctly render the user profile card component with the expected
  user name and avatar image when the user data has been successfully
  loaded from the API')`.
- **One assertion per concept, not per line.** Testing a form? Assert
  the happy path in one test, validation errors in another, submission
  loading state in another. Don't split every single assertion into its
  own `it()` block.
- **Test behavior, not structure.** "When I click submit, the form data
  is sent" — not "the onClick handler calls handleSubmit which calls
  mutate which calls fetch."
- **Edge cases over happy paths.** The happy path probably works. Write
  the tests that catch what breaks: empty data, network failure, race
  conditions, permission denied, expired token, malformed input.
- **Don't test the framework.** Don't assert that `useState` updates
  state or that `useEffect` runs on mount. Test your logic, not React's.

---

## Documentation

Document like a human: thorough where it matters, minimal where it's
obvious.

- **README answers three questions:** What is this, how do I run it,
  how do I deploy it. Anything beyond that goes in specific docs files.
- **Don't document the obvious.** A function called `formatCurrency`
  doesn't need a JSDoc comment saying "Formats a currency value." If
  the name is clear, let the types speak.
- **Do document the surprising.** Non-obvious parameters, unexpected
  return values, side effects, performance implications, and "why this
  approach over the obvious alternative."
- **API documentation is mandatory, internal documentation is
  proportional.** Public functions, exported APIs, shared components:
  always documented. Private helpers used in one file: only if non-obvious.
- **ADR (Architecture Decision Records)** for decisions that were
  debated. One paragraph: what was decided, why, what was rejected.
  Lives in `/docs/adr/` or inline as a comment at the decision point.

---

## Git History

Commits should read like a human made them, not a machine.

- **Commit messages are concise, not verbose.** `fix: handle null
  avatar in user card` not `fix: resolved an issue where the user
  profile card component would throw an undefined error when the avatar
  image URL was not provided by the API response`.
- **Atomic commits.** One logical change per commit. But "logical" is
  flexible — a feature with its tests is one commit, not two.
- **Honest history.** It's fine to have `fix: typo in dashboard heading`
  or `chore: remove unused import`. Real repos have these. A history
  where every commit is a perfectly scoped feature is suspicious.

---

## What to Keep From AI (Non-Negotiable)

These are AI's strengths. Never sacrifice them for "human voice":

- **Type safety everywhere.** Strict TypeScript, runtime validation at
  boundaries. Humans skip this; we don't.
- **Accessibility built in.** Semantic HTML, keyboard navigation, ARIA,
  focus management. Every component, every time.
- **Security by default.** Input validation, parameterized queries,
  output encoding, auth checks. Never skip for speed.
- **Tests exist.** Every feature has tests. Coverage may vary by
  criticality, but zero-test features don't ship. **For environments
  without traditional test runners** (Shopify themes, static HTML):
  use Playwright MCP against the dev preview for automated accessibility
  audits (axe-core), visual regression screenshots across breakpoints,
  performance metrics, console error detection, and critical flow
  verification. Playwright IS the test runner for these projects.
- **Error boundaries and loading states.** Every async boundary handles
  all four states. Humans forget empty states; we don't.
- **Consistent token usage.** Every color, spacing, font references the
  design system. No hardcoded values anywhere.
- **Consistent code conventions.** Naming patterns, file organization,
  import order — uniform across the entire codebase.
