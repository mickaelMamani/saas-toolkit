---
name: resume
description: Resume after /clear or new session. Reads TASKS.md, verifies completed work, continues from where you left off.
disable-model-invocation: true
argument-hint: <spec.md>
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - mcp__supabase
---

# /resume — Resume After Context Reset

Recovers context after `/clear` or a new session and continues the build from where it left off.

## Algorithm

### 1. Read project state

Read these files to rebuild context:
- `TASKS.md` — build progress, current phase, pending tasks
- `CLAUDE.md` — project conventions, tech stack, file structure
- `package.json` — installed dependencies
- `src/` directory structure (via Glob)

### 2. Verify completed work

For each task marked `[x]` in TASKS.md, do a quick verification:
- Check that the referenced files exist (Glob)
- Run `npm run build` to verify the project compiles
- If a completed task's files are missing, mark it back to `[ ]`

### 3. Identify current position

- Find the first uncompleted task (`[ ]`) in the earliest incomplete phase
- Note any blocked tasks (`[!]`) that may need resolution first

### 4. Present status summary

```
## Resume: [Project Name]

Progress: [X/Y tasks completed] — Phase [N]: [Name]

Completed phases:
- Phase 0: Foundation ✓
- Phase 1: Auth ✓

Current phase: Phase 2 — Backend
- [x] Create Server Actions for projects CRUD
- [ ] Create Stripe webhook handler  ← next
- [ ] Implement subscription gating

Build status: passing / failing
```

### 5. Continue

On user confirmation, continue the build from the next uncompleted task. Follow the same workflow as `/build`.

## When to use

- After running `/clear` during a long build session
- Starting a new Claude Code session on an in-progress project
- After an interruption or crash

## Rules

- Always read TASKS.md first — it's the source of truth
- Verify before assuming — don't trust task markers blindly
- Don't redo completed work — verify and move on
- Present status before continuing — let the user confirm the plan
