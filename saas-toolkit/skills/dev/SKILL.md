---
description: Spec-driven development workflow — understand, plan, implement, verify
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
  - mcp__supabase__*
  - mcp__context7__*
user-invocable: true
---

# /dev — Spec-Driven Development Workflow

Full development workflow for implementing features or changes with a structured approach.

## Phases

### Phase 1: Understand

Before writing any code:

1. **Read the spec/request** — Understand exactly what's being asked. Ask clarifying questions if anything is ambiguous.
2. **Explore the codebase** — Use the explore-codebase agent to understand the current architecture, relevant files, and existing patterns.
3. **Check existing implementations** — Look for similar features already implemented to follow established patterns.
4. **Identify dependencies** — Determine what libraries, APIs, or database changes are needed.

### Phase 2: Plan

1. **Break down the work** — List the specific files to create or modify.
2. **Define the approach** — For each file, describe what changes are needed and why.
3. **Identify risks** — Note potential issues, edge cases, or breaking changes.
4. **Present the plan** — Share the plan with the user for approval before implementing.

### Phase 3: Implement

1. **Work incrementally** — Implement one piece at a time, starting with the foundation (types, database, then API, then UI).
2. **Follow existing patterns** — Match the project's coding style, naming conventions, and file organization.
3. **Handle errors** — Add proper error handling at system boundaries.
4. **Keep it minimal** — Only implement what was requested. No extra features or premature abstractions.

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
