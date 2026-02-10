---
description: Lightweight single-feature implementation — explore, implement, verify without formal planning
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - mcp__supabase__*
  - mcp__context7__*
user-invocable: true
---

# /add-feature — Lightweight Feature Implementation

Implements a single feature end-to-end without the formal planning overhead of `/dev`. Best for well-understood features that don't require architectural decisions.

## Workflow

### 1. Understand the request

Read the user's feature description. If anything is ambiguous, ask one round of clarifying questions.

### 2. SaaS checklist

Before implementing, quickly assess:
- **Needs migration?** — Does this feature require new tables or columns?
- **Needs RLS?** — If touching the database, what access rules apply?
- **Subscription gating?** — Should this feature be limited to certain plans?
- **User or org scoped?** — Is data owned by a user or an organization?
- **Affects auth?** — Does this change protected routes or permissions?

### 3. Explore relevant code

Dispatch `explore-codebase` agent (or Glob/Grep directly for simple lookups) to find:
- Related existing files and patterns
- Database schema context
- Import paths and component conventions

### 4. Implement

Follow SaaS stack order:
1. **Database** — Migration + RLS if needed
2. **Types** — Update or generate types
3. **Backend** — Server Actions or API routes
4. **Frontend** — Components and pages
5. **Wire up** — Navigation, links, integration

### 5. Verify

- Run `npm run build` to check for errors
- Review the changes for obvious issues
- If TASKS.md exists, update it with the new feature

## Difference from /dev

| Aspect | /dev | /add-feature |
|--------|------|-------------|
| Planning | Formal spec + plan document | No planning phase |
| Scope | Multi-file features, new systems | Single focused feature |
| Exploration | Deep codebase analysis | Quick pattern lookup |
| Output | Spec → Plan → Code → Verify | Code → Verify |

Use `/dev` for complex features that need architectural decisions. Use `/add-feature` for straightforward additions.

## Rules

- Keep it simple — implement only what was requested
- Follow existing project patterns
- Always check for needed DB changes first
- Update TASKS.md if it exists
- Don't create a planning document — go straight to implementation
