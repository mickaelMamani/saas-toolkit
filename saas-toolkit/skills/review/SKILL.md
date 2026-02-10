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

Launch these agents in parallel as appropriate:

- **frontend-code-reviewer** agent for frontend files (components, pages, hooks, styles)
- **security-reviewer** agent for a comprehensive security audit across all files

For backend files, review directly for:
- **API design** — RESTful conventions, proper HTTP methods, status codes
- **Error handling** — proper try/catch, meaningful error messages, no swallowed errors
- **Data access** — efficient queries, proper indexing hints, no N+1 patterns
- **Type safety** — proper TypeScript types, no `any`, validated inputs with Zod

### 3. SaaS-specific checks

#### Stripe compliance
- Webhook handlers verify signatures with `constructEvent()`
- Raw body parsing (not JSON parsed)
- Idempotent event processing
- No price/amount trust from client
- Secret key not exposed to client

#### Supabase security
- RLS enabled on all public tables
- `getUser()` used instead of `getSession()` for auth checks
- Service role key only in server-side code
- No Supabase client with service role in Client Components

#### Next.js patterns
- Server Components by default (no unnecessary `"use client"`)
- Server Actions for mutations (not API routes)
- `next/image`, `next/font`, `next/link` used consistently
- Proper Suspense boundaries
- Metadata API for SEO

### 4. Cross-cutting concerns

After individual file reviews, check:
- **Consistency** — do new files follow existing patterns?
- **Missing pieces** — are there missing error boundaries, loading states, or edge cases?
- **Breaking changes** — could these changes break existing functionality?
- **Test coverage** — are there tests for the new code? Should there be?

### 5. Summary

Produce a final review with:

```markdown
## Code Review Summary

**Files reviewed:** X
**Overall assessment:** [Good / Needs changes / Needs significant rework]

### Production Readiness

| Dimension | Status | Notes |
|-----------|--------|-------|
| Security | PASS/FAIL | ... |
| Stripe compliance | PASS/FAIL/N/A | ... |
| Supabase security | PASS/FAIL/N/A | ... |
| Next.js patterns | PASS/FAIL | ... |
| Error handling | PASS/FAIL | ... |
| Type safety | PASS/FAIL | ... |

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
