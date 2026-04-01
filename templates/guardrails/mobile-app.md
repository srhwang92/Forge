# Project Guardrails — Mobile Application

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

## Applicable Regulations
See global Privacy & Regulatory Awareness for baseline (PIPEDA, PIPA BC,
CASL). Additionally for mobile:
- App Store / Google Play policies (mandatory for distribution)
- COPPA if any possibility of users under 13
- GDPR, CCPA/CPRA based on user geography
- Accessibility: WCAG 2.2 AA applies to mobile (ADA, AODA, EAA)

## App Store Compliance

- **Never implement functionality that circumvents app store review.**
  No remote code loading, hidden features, or behavior changes after
  review approval.
- **Privacy nutrition labels must be accurate.** Every data collection
  point must be reflected in the App Store / Play Store privacy
  declarations. Flag any new data collection for manifest update.
- **Permissions must be justified.** Only request permissions the app
  genuinely needs. Request at point of use, not on first launch.
  Camera, location, contacts, microphone — each needs a clear user
  benefit explanation.
- **In-app purchases** must go through platform payment systems where
  required. Flag any payment flow for compliance review.

## Data Storage on Device

- **Never store sensitive data in plaintext on device.** Use the
  platform keychain/keystore for tokens, credentials, and secrets.
- **Never cache PII in unencrypted local storage.** If the app caches
  user data for offline use, encrypt at rest.
- **Clear sensitive data on logout.** Tokens, cached user data, and
  session state must be wiped when the user logs out.
- **Biometric authentication** must use platform APIs (Face ID, Touch
  ID, Fingerprint) — never custom biometric implementations.
- **Certificate pinning** for API communications in production builds.
  Without it, MITM attacks are trivial on mobile networks.

## Offline & Sync

- **Conflict resolution strategy must be defined** before implementing
  offline support. Document what happens when offline edits conflict
  with server state.
- Never silently discard user data during sync conflicts. Preserve
  both versions and let the user or a defined policy resolve.
- Queue mutations when offline, replay on reconnect with idempotency.

## Performance

- App size budget: flag if the app exceeds 50MB (cellular downloads).
- Startup time: flag any change that adds >200ms to cold launch.
- Battery: flag operations that keep the device awake, use continuous
  GPS, or poll servers frequently.
- Memory: test on low-end devices, not just the latest flagship.

## Push Notifications

- **Explicit opt-in required.** Never auto-subscribe users to push
  notifications.
- Notification content must not contain PII (visible on lock screen).
- Deep links from notifications must handle edge cases (logged out,
  stale data, deleted content).
