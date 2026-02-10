---
description: Code review with parallel subagents for thorough analysis
allowed-tools:
  - Read
  - Grep
  - Glob
  - Task
  - Bash
user-invocable: true
---

# /review — Code Review

Thorough code review using parallel subagents for comprehensive analysis.

## Process

### 1. Identify scope

Determine what to review:
- If the user specifies files, review those files
- If no files specified, review recent changes: `git diff --name-only HEAD~1` or `git diff --staged --name-only`
- Group files by domain (frontend, API, database, config)

### 2. Parallel review

Launch the **frontend-code-reviewer** agent for frontend files (components, pages, hooks, styles).

For backend files, review directly for:
- **API design** — RESTful conventions, proper HTTP methods, status codes
- **Security** — input validation, authentication checks, authorization, SQL injection
- **Error handling** — proper try/catch, meaningful error messages, no swallowed errors
- **Data access** — efficient queries, proper indexing hints, no N+1 patterns
- **Type safety** — proper TypeScript types, no `any`, validated inputs with Zod

### 3. Cross-cutting concerns

After individual file reviews, check:
- **Consistency** — do new files follow existing patterns?
- **Missing pieces** — are there missing error boundaries, loading states, or edge cases?
- **Breaking changes** — could these changes break existing functionality?
- **Test coverage** — are there tests for the new code? Should there be?

### 4. Summary

Produce a final review with:

```markdown
## Code Review Summary

**Files reviewed:** X
**Overall assessment:** [Good / Needs changes / Needs significant rework]

### Critical issues (must fix)
1. ...

### Warnings (should fix)
1. ...

### Suggestions (nice to have)
1. ...

### Positive highlights
- ...
```

## Rules

- Be specific — reference exact files and line numbers
- Distinguish severity levels clearly
- Acknowledge good code, not just problems
- Don't nitpick style if there's a formatter configured
- Focus on logic, security, and correctness over aesthetics
