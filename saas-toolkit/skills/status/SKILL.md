---
name: status
description: Project health check and progress report. Checks build, types, lint, tests, RLS coverage, and progress against spec.
disable-model-invocation: true
argument-hint: "[spec.md]"
context: fork
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Task
  - mcp__supabase
---

# /status — Project Health Dashboard

Quick read-only health check of the current SaaS project.

## Checks

Run these checks in parallel where possible:

### 1. TASKS.md Progress

- Read `TASKS.md`
- Count tasks by status: `[x]` completed, `[ ]` pending, `[!]` blocked
- Identify current phase and next task

### 2. Build Status

```bash
npm run build
```

Report: passing or failing (with error summary if failing).

### 3. Type Check

```bash
npx tsc --noEmit
```

Report: passing or failing (with error count if failing).

### 4. Database State

Dispatch `explore-db` agent to check:
- Number of tables
- RLS enabled on all tables
- Number of policies per table

### 5. Git Status

Report: clean/dirty, current branch, commits ahead/behind.

## Output Format

Present results as a table:

```
## Project Health: [name]

| Area         | Status | Details                        |
|--------------|--------|--------------------------------|
| Tasks        | 12/20  | Phase 3: Data Layer            |
| Build        | ✓ Pass |                                |
| Types        | ✗ Fail | 3 errors                       |
| DB (tables)  | 6      | All have RLS                   |
| DB (policies)| 14     | org_members missing DELETE      |
| Git          | Clean  | main, up to date               |
```

## Rules

- This is a **read-only** skill — do not modify any files
- Run checks in parallel where possible for speed
- If TASKS.md doesn't exist, report that and suggest running `/init-saas`
- If build or type check fails, include the first few errors to help diagnose
