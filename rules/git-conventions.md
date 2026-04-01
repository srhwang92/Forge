---
description: Git commit conventions, staging discipline, and Cursor/Claude Code boundary rules. Always active.
globs: ["**"]
---

# Git Conventions

## Commit Format

```
type(scope): concise description in imperative mood

[optional body — explain WHY, not what]

[optional footer — BREAKING CHANGE:, Closes #123, etc.]
```

**Types:**
- `feat` — new functionality visible to users
- `fix` — bug fix
- `refactor` — restructure without behavior change
- `perf` — performance improvement
- `test` — adding or fixing tests
- `docs` — documentation only
- `chore` — tooling, deps, config, CI
- `style` — formatting, whitespace (no logic change)

**Scopes** — match the feature or module touched:
`auth`, `dashboard`, `api`, `db`, `ui`, `billing`, `checkout`, etc.
Omit scope for cross-cutting changes: `chore: update deps`

**Subject line:**
- Imperative mood: "add filter" not "added filter" or "adds filter"
- Lowercase after type prefix
- No period at end
- ≤72 characters

**Body** (when needed):
- Explain why the change was made, not what changed (the diff shows what)
- Wrap at 72 characters
- Separate from subject with a blank line

**Footer** (when needed):
- `BREAKING CHANGE: description` for any breaking API/contract change
- `Closes #123` or `Fixes #456` for issue references

## When to Commit

- **After each logical unit of work.** One feature + its tests = one
  commit. One bug fix = one commit. Don't bundle unrelated changes.
- **Before any risky operation** (mandatory checkpoint):
  `git add -A && git commit -m "chore: checkpoint before [operation]"`
- **Before switching context** (different feature, different file area)
- **Never commit with failing tests or lint errors.** Run the check
  command first.

## Staging Discipline

- **Stage intentionally.** Use `git add <specific files>` over
  `git add .` when possible. Review what's staged before committing.
- **Never stage generated files** (build output, compiled assets,
  auto-generated types) unless they're meant to be committed per
  project conventions.
- **Never stage `.env` files, secrets, or credentials.** If `.gitignore`
  doesn't cover them, fix `.gitignore` first.
- **Unstage accidental changes** before committing. If a file was
  modified but isn't part of this logical commit, unstage it.

## Cursor ↔ Claude Code Boundary

Git operations (branch, commit, merge, PR) happen in Cursor IDE only.
Claude Code does NOT create branches, switch branches, push, or
force-push.

**Handoff protocol:**
- **TO Claude Code:** State what changed since last session, which
  branch is active, and what the current task is
- **FROM Claude Code:** Run `/compact` before stopping, ensure
  STATUS.md and logs are updated
- **Never have both tools editing the same file simultaneously**
- **Conflict resolution:** If Claude Code and Cursor have diverged,
  Rocky resolves conflicts in Cursor before resuming Claude Code work
