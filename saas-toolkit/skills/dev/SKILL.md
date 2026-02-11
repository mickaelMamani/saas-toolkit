---
name: dev
description: Spec-driven development workflow — understand, plan, implement, verify.
disable-model-invocation: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - WebSearch
  - WebFetch
  - mcp__supabase
  - mcp__context7
---

# /dev — Spec-Driven Development Workflow

Full development workflow for implementing features or changes with a structured approach.

## Phases

### Phase 1: Understand

Before writing any code:

1. **Read the spec/request** — Understand exactly what's being asked. Ask clarifying questions if anything is ambiguous.
2. **Auto-explore the codebase** — Launch the `explore-codebase` agent to map the relevant parts of the codebase. Focus on:
   - Project structure and file organization
   - Existing patterns for similar features
   - Database schema (use `explore-db` agent if DB changes needed)
   - Current auth/subscription patterns
3. **Check existing implementations** — Look for similar features already implemented to follow established patterns.
4. **Identify dependencies** — Determine what libraries, APIs, or database changes are needed.

**Context checkpoint:** After exploration, summarize your understanding:
```
## Context Summary
- **Goal:** [what we're building]
- **Key files:** [files to create/modify]
- **Patterns to follow:** [existing patterns discovered]
- **Dependencies:** [libraries/APIs/DB changes needed]
- **Open questions:** [anything unclear]
```

### Phase 2: Plan

1. **Break down the work** — List the specific files to create or modify.
2. **Define the approach** — For each file, describe what changes are needed and why.
3. **Identify risks** — Note potential issues, edge cases, or breaking changes.
4. **Present the plan** — Share the plan with the user for approval before implementing.

Use ultrathink for complex phases that involve architectural decisions or multi-system coordination.

### Phase 3: Implement

Use `ultrathink` for complex phases that involve architectural decisions or multi-system coordination.

#### Implementation Order

**Backend-first** (recommended for most features):
1. **Database** — Migrations, new tables, column changes (`/db-migration` patterns)
2. **RLS policies** — Security policies for new/modified tables
3. **Types** — Generate or update TypeScript types (`supabase gen types`)
4. **Server Actions / API routes** — Backend logic, mutations, data fetching
5. **Edge Functions** — If needed (stripe-sync-engine, custom webhooks)
6. **UI components** — Pages, layouts, client components
7. **Integration** — Wire everything together, test flows

**Frontend-first** (for UI-heavy features with existing backend):
1. **Types** — Define expected data shapes
2. **Hooks** — Data fetching and state management
3. **Components** — UI building blocks
4. **Pages** — Route pages and layouts
5. **Backend** — Server Actions to support the UI
6. **Database** — Migrations if new data storage is needed

For each step:
- Follow existing patterns in the project
- Handle errors at system boundaries
- Keep it minimal — only implement what was requested
- If implementing auth flows, cross-reference `/auth` skill patterns
- If implementing Stripe billing, cross-reference `/stripe-setup` skill patterns

### Phase 4: Verify

1. **Type check** — Run the TypeScript compiler to catch type errors.
2. **Lint** — Run the linter if configured.
3. **Test** — Run existing tests to ensure nothing is broken. Write new tests if the project has a test suite.
4. **Manual check** — Review the changes yourself for obvious issues.

## Rules

- Always read files before modifying them
- Prefer editing existing files over creating new ones
- Follow the project's established patterns and conventions
- Ask before making architectural decisions
- Don't over-engineer — keep it simple
