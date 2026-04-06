# Project Guardrails — Electron Desktop Application

Extends global `~/.claude/GUARDRAILS.md`. These rules are additive.

> **Disclaimer:** These guardrails reduce risk but do not constitute
> regulatory compliance. For regulated industries (fintech, healthcare,
> etc.), engage a qualified compliance professional to review the
> generated project guardrails and the application output. Awareness of
> regulations is not the same as compliance with them.

## Applicable Regulations
See global Privacy & Regulatory Awareness for your jurisdiction baseline. Additionally for desktop apps:
- Platform-specific code signing requirements (Apple notarization, Windows SmartScreen)
- App store policies if distributing via Mac App Store or Microsoft Store
- Data protection regulations apply to locally stored data, not just cloud data

## Process Isolation & Sandbox

- **`nodeIntegration` must be `false`.** Non-negotiable. Enabling
  nodeIntegration gives any web content full system access — including
  iframes, images from CDNs, and link previews.
- **`contextIsolation` must be `true`.** Prevents renderer scripts from
  accessing Electron internals or Node.js APIs directly.
- **`sandbox` must be `true`.** Restricts renderer processes to a
  minimal environment. Disable only for specific windows with documented
  justification and project lead approval.
- **`nodeIntegrationInWorker` and `nodeIntegrationInSubFrames` must be
  `false`** (they default to false — never set to true). These bypass
  the `nodeIntegration: false` rule for Web Workers and iframes.
- **Use preload scripts with `contextBridge`** to expose only specific,
  validated IPC methods to the renderer. Never expose raw Node.js APIs.
  Each exposed method must validate its arguments.
- **`@electron/remote` is prohibited in production.** The remote module
  re-exposes full Node.js access to the renderer, undoing
  `contextIsolation` entirely. Use IPC via `contextBridge` instead.

## IPC Security

**Every IPC handler is a security boundary.** Treat IPC messages from
the renderer with the same distrust as HTTP requests from a client.

- **Validate all arguments** in every `ipcMain.handle()` and
  `ipcMain.on()` handler. Never pass renderer-controlled strings
  directly to: `fs` operations, `child_process`, `shell.openExternal`,
  database queries, or network requests.
- **`shell.openExternal` URL allowlist.** Only allow `https:` (and
  `http:` if explicitly needed). Reject `javascript:`, `file:`,
  `data:`, `vbscript:`, and all other schemes. Parse with `new URL()`
  and validate the scheme before calling. A `javascript:` URL passed
  to `shell.openExternal` is immediate code execution.
- **Allowlist filesystem paths.** If the app reads/writes files, define
  an allowed directory list. Validate that resolved paths fall within
  the allowlist after resolving symlinks and `..` traversal.
- **Never expose a generic "run command" or "read file" IPC handler.**
  Each handler should do one specific thing with validated inputs.
- **Minimize IPC surface.** Every exposed IPC channel is an attack
  surface. Audit periodically — remove unused channels.

## Auto-Update Security

- **Code signing is mandatory.** Configure electron-updater with
  signature verification: `publisherName` for Windows, certificate
  validation for macOS. Never ship unsigned updates.
- **HTTPS only for update checks.** Never use HTTP for update server
  communication. Never disable TLS verification.
- **Verify update integrity.** Use electron-updater's built-in
  signature verification. For custom update mechanisms, verify
  SHA-256 hashes of downloaded files before applying.
- **Pin the update server URL.** Never construct the update URL from
  user input or untrusted configuration.
- **Staged rollouts and rollback.** Ship updates to a percentage of
  users first (10% → 50% → 100%) before full rollout. Implement a
  server-side kill switch to halt update distribution. Include a
  client-side rollback mechanism to revert to the previous version.
  Unlike web deployments, you cannot un-ship a desktop binary — a
  broken signed update strands users until the next fix ships.

## Local Data Security

- **Encrypt sensitive data at rest.** Use `safeStorage` API for
  credentials and tokens. Never store secrets in plaintext files,
  `localStorage`, or unencrypted SQLite databases.
- **Clear sensitive data on logout/uninstall.** Tokens, cached user
  data, and session state must be wiped.
- **`app.getPath('userData')` is the only approved storage location.**
  Never write application data outside the user data directory.

## Web Content Loading

- **Never load remote content in the main window** unless the app is
  specifically a browser or web wrapper. Use sandboxed `<webview>` or
  `BrowserView` with restricted permissions for remote content.
- **`<webview>` security.** If using `<webview>` tags: set
  `nodeIntegration: false`, `contextIsolation: true`, and
  `allowpopups: false` in webpreferences. Disable the `webviewTag`
  entirely (`webPreferences: { webviewTag: false }`) in windows that
  don't need it. Older Electron versions have webview defaults that
  grant Node.js access.
- **Set a strict CSP** on all HTML loaded by the application. See
  `frontend-standards.md` CSP section for policy guidance.
- **`webSecurity` must be `true` in production.** Never ship with
  `webSecurity: false` — it disables same-origin policy entirely.
- **`allowRunningInsecureContent` must be `false`.** Never allow
  mixed content in production.

## Deep Linking / Protocol Handlers

- **Validate all deep link URLs.** If the app registers a custom
  protocol (e.g., `myapp://`), validate every incoming URL. Never
  pass deep link parameters to shell commands, file operations, or
  IPC without sanitization.
- **Prevent open redirect via deep links.** A deep link that navigates
  to an attacker-controlled URL can be used for phishing.
- **Windows protocol handler injection.** On Windows, custom protocol
  URLs are passed as command-line arguments. If the URL contains shell
  metacharacters, it can inject commands. Always parse with `new URL()`
  and reject any URL that doesn't parse cleanly. Never concatenate
  the raw URL string into shell commands or file paths.
