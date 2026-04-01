---
description: Context7 MCP usage rules — when to use, how to query effectively, token-conscious patterns. Always active.
globs: ["**"]
---

# Context7 MCP Usage

Context7 pulls live documentation but consumes significant tokens.
Use it strategically — not as a first resort and not as a crutch.

## Two-Step Workflow

Always follow this sequence:
1. `resolve-library-id` — get the Context7 library ID from the package
   name. Do this once per library per session, not per query.
2. `query-docs` — fetch specific documentation using the resolved ID.
   Request only the section you need, not the full library docs.

## When to Use Context7

**Use Context7 when:**
- Working with a library whose API changes between versions (Next.js
  App Router, Supabase, shadcn/ui, Prisma, Drizzle, TanStack Query)
- You need the exact function signature, parameter types, or return
  shape for a specific API
- An approach isn't working and the API may have changed since training
- The project uses a library version newer than your training data
- You're implementing a pattern you haven't used before in this library

**Don't use Context7 when:**
- You're confident in the API from repeated use in this session
  (you already looked it up and it's still in context)
- The question is about general programming concepts, not a specific
  library API (vanilla JS, CSS, HTML, basic TypeScript, git)
- The library is stable and well-known with an API that rarely changes
  (React core hooks, Node.js built-ins, standard Python libraries)
- You can answer from the source code in the project (read
  `node_modules/[package]/README.md` or type definitions instead)

**Use web search instead when:**
- Context7 doesn't have the library (check resolve-library-id first)
- You need community solutions, Stack Overflow answers, or blog posts
- You need to compare libraries or evaluate alternatives
- The question is about configuration, deployment, or platform-specific
  behavior rather than API usage

## Writing Effective Queries

- **Be specific.** "createClient auth configuration" not "supabase docs"
- **Name the exact function or concept.** "useQuery staleTime
  revalidation" not "tanstack query caching"
- **Include the context.** "Next.js 15 App Router server actions" not
  "next.js server"
- **One concept per query.** Don't combine "how to set up auth AND how
  to configure RLS" — make two queries
- **Request specific sections.** Use the `topic` parameter to narrow
  results when available

## Token Discipline

- **Resolve library IDs once per session.** Cache mentally — don't
  re-resolve the same library multiple times.
- **Delegate to subagents** when doing exploratory research across
  multiple libraries. Keep lookup tokens out of the main context.
- **Don't use Context7 to confirm what you already know.** If you're
  95% confident, use that confidence. Context7 is for the 5% where
  being wrong would cause a bug.
- **Prefer reading local source** (`node_modules`, type definitions,
  project README) over Context7 for project-specific dependency
  questions — it's already on disk at zero token cost.
