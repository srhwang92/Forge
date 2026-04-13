---
description: Superpowers skill invocations for Forge workflow stages, plus Sonnet implementation dispatch default. Loaded when working on build files (PLAN.md, SPEC.md, .claude/ state).
globs: ["**/PLAN.md", "**/SPEC.md", "**/.claude/**"]
---

# Superpowers Workflow Integration

> CLAUDE.md Workflow section declares Superpowers drives all builds.
> This file is the invocation spec — which skill runs at which stage
> and how implementation dispatch works.

## Stage-to-skill mapping

Forge's workflow stages map to Superpowers skills. At each stage,
invoke the skill rather than freestyling:

- **Brainstorm** → `brainstorming`
- **Spec** → Forge interview from `templates/interview.md`
  (no Superpowers equivalent — Forge owns this stage)
- **Plan** → `writing-plans` (creates/updates PLAN.md)
- **TDD** → `test-driven-development`
- **Dev** → `subagent-driven-development` + `dispatching-parallel-agents`
  when tasks are independent
- **Review** → `verification-before-completion` +
  `requesting-code-review` (subagent review = Forge's Layer 2)
- **Finalize** → `finishing-a-development-branch`

For debugging mid-phase: `systematic-debugging`.
For worktree-based parallel work: `using-git-worktrees`.

If a stage starts without its skill invoked, that's a process gap —
invoke the skill and continue.

## Implementation dispatch default

For implementation tasks in a build phase, dispatch to Sonnet
subagents by default. The `subagent-driven-development` skill handles
dispatch mechanics; this rule sets the model default.

**Main agent (Opus) retains:**
- Architectural decisions and design trade-offs
- State file updates (STATUS, PLAN, REGISTRY, DECISIONS, INVARIANTS)
- Router/barrel wiring after subagents complete
- Cross-cutting fixes touching many files
- Synthesis of subagent results
- Commit messages

**Sonnet subagents receive:**
- Discrete implementation tasks from PLAN.md
- Relevant SPEC.md decisions + project conventions snippet
- Acceptance criteria from PLAN.md

**Dispatch floor:** main agent implements directly only for tasks
<50 lines in a single file, OR when the task requires context the
subagent can't receive (in-flight design conversation, pending
decision). Subagent overhead exceeds savings below this scale.

**Parallel vs sequential:** use `dispatching-parallel-agents` when
PLAN.md marks tasks independent. Sequential only when outputs feed
each other. List ALL files each subagent might touch including
barrel files, shared types, config — overlap means sequential.
