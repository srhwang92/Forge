# Project Guardrails — Mobile Application

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare),
> engage a qualified compliance professional to review the guardrails
> and the application output. Awareness of regulations is not the same
> as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline.
Additionally for mobile:
- App Store / Google Play policies (mandatory for distribution)
- COPPA if any possibility of users under 13
- GDPR, CCPA/CPRA based on user geography
- Accessibility: WCAG 2.2 AA applies to mobile (ADA, AODA, EAA)

## App Store Compliance

- **Never implement functionality that circumvents app store review.**
  No remote code loading, hidden features, or behavior changes after
  review approval.
- **Privacy nutrition labels must be accurate.** Every data collection
  point must match the App Store / Play Store privacy declarations.
  Flag any new collection for manifest update.
- **Permissions must be justified.** Only request what the app genuinely
  needs. Request at point of use, not on first launch.
- **In-app purchases** must use platform payment systems where required.
  Flag any payment flow for compliance review.

## Platform Privacy & Manifests (Critical)

These are enforced at submission, not build time. Missing any of them
causes rejection and multi-week delays. Junior developers coming from
web consistently miss these.

- **iOS `Info.plist` permission strings are mandatory.** Every permission
  the app requests must have a corresponding `NS...UsageDescription`
  string (e.g., `NSCameraUsageDescription`, `NSLocationWhenInUseUsageDescription`,
  `NSHealthShareUsageDescription`). Missing strings cause runtime crashes,
  not build errors. The strings must explain WHY the permission is needed
  in user-facing language — App Store reviewers reject vague strings.
  See Apple's docs for the exact string per permission.
- **iOS `PrivacyInfo.xcprivacy` manifest is required** (as of 2024).
  Declares required-reason APIs, tracking domains, and data collection
  categories. Missing or inaccurate = rejection. Regenerate when adding
  any new SDK or changing data usage.
- **Android permissions in `AndroidManifest.xml`.** Every runtime
  permission must be declared. Google Play rejects over-requesting.
  Location permissions have their own review form requiring justification.
- **HealthKit / Google Fit data cannot be shared with third parties**
  without explicit user consent AND platform compliance. Apple
  specifically prohibits sharing HealthKit data for advertising or with
  unrelated third parties. Sending HealthKit data to an LLM API is a
  third-party transfer — requires user consent, clear disclosure, and
  (in the US for covered entities) a BAA. If the LLM provider doesn't
  offer a BAA, HealthKit data cannot be sent to them.

## Secrets & API Keys in Client Bundles

**The canonical junior mobile-dev mistake.** Every secret bundled with
a mobile app is extractable via standard tools (`apktool`, `class-dump`,
`Hopper`). App code, config files, and compiled assets all end up in
the production bundle.

- **Never ship LLM API keys, third-party API keys, or database
  credentials in client code.** Includes `app.json` extra config, `.env`
  files bundled into the app, hardcoded constants, and assets-directory
  config files.
- **Use a server-side proxy for any authenticated third-party API.**
  Mobile app → your backend → third-party API. Keys never leave the
  server. Same rule as `ai-features.md` — mobile has no server boundary
  of its own.
- **Canary test:** decompile the release build and grep for known
  secret patterns (`sk-`, `pk_live_`, provider domains). Run as part
  of the release pipeline — fail the build on match.
- **Public keys and non-secret config** (Sentry DSN, PostHog project
  key, Supabase anon key with RLS) are intended for client use and are
  fine to bundle. The Supabase SERVICE_ROLE_KEY is NOT safe; the anon
  key is.

## Data Storage on Device

- **Never store sensitive data in plaintext on device.** Use the
  platform keychain/keystore for tokens, credentials, and secrets.
  React Native: `expo-secure-store` or `react-native-keychain`. Choose
  the keychain accessibility level (`WhenUnlocked` vs `AfterFirstUnlock`)
  based on whether background tasks need access — document the choice
  and rationale in DECISIONS.md.
- **`AsyncStorage` is NOT encrypted.** On Android it's plaintext SQLite
  accessible to any process running as the app's user. On iOS it's in
  the app sandbox but readable by anyone with device access. Never
  store tokens, PII, health data, or chat history containing sensitive
  content in AsyncStorage. Use `expo-secure-store` or
  `react-native-mmkv` with encryption enabled.
- **Clear sensitive data on logout.** Tokens, cached user data, session
  state — wipe all of it.
- **Biometric authentication** must use platform APIs (Face ID, Touch
  ID, Fingerprint) — never custom implementations.
- **Certificate pinning** for API communications in production builds.
  Without it, MITM is trivial on mobile networks. **Expo managed
  workflow limitation:** requires `expo-dev-client` or bare workflow.
  For managed workflow, compensate with TLS verification, key rotation,
  request signing, and device attestation. Document this as a known
  limitation in project CLAUDE.md.

## Offline & Sync

- **Conflict resolution strategy must be defined** before implementing
  offline support. Document what happens on conflict.
- Never silently discard user data during sync conflicts. Preserve
  both versions and let the user or a defined policy resolve.
- Queue mutations when offline, replay on reconnect with idempotency.

## Performance

- App size: flag if the app exceeds 50MB (cellular downloads).
- Startup: flag any change adding >200ms to cold launch.
- Battery: flag continuous GPS, wake locks, frequent polling.
- Memory: test on low-end devices, not just flagships.

## Push Notifications

- Explicit opt-in required. Never auto-subscribe.
- Notification content must not contain PII (lock-screen visibility).
- Deep links must handle edge cases (logged out, stale, deleted).

## Testing

- **React Native uses Jest with the RN preset**, not Vitest. Vitest
  has known compatibility issues with RN's module system, native
  module mocking, and the RN renderer. Standard setup: `jest` +
  `jest-expo` (Expo) or `@testing-library/react-native`. Playwright
  does not run against native apps — use Detox or Maestro for E2E.
- **Test on real devices, not just simulators.** Memory, battery, and
  network conditions differ. Minimum matrix: 1 low-end Android, 1
  mid-range iPhone, 1 current flagship.
- **Accessibility:** React Native uses `accessibilityLabel`,
  `accessibilityRole`, `accessibilityHint` — NOT HTML `role`. Test
  with VoiceOver (iOS) and TalkBack (Android) on real devices.
  axe-core does not apply to native.

## Styling & Design Tokens

- **React Native styling is JavaScript objects, not CSS.** DESIGN.md
  tokens for mobile are TS constants in a shared `theme.ts`, not CSS
  variables. Numeric values with no `px`/`em`/`rem` — RN uses dp/pt
  natively.
- **No CSS breakpoints.** Use `Dimensions.get('window')`,
  `useWindowDimensions()`, or `Platform.OS`.
- **Platform-specific styling.** Use `Platform.select()` for divergence
  (shadows vs elevation, navigation, safe areas). Document in
  DECISIONS.md.
- **Design system libraries:** React Native Paper, NativeBase, Tamagui.
  Web-focused systems (shadcn/ui, Radix) do NOT work on React Native.
