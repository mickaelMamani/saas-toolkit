---
name: build
description: Phase-by-phase SaaS build from a spec. Dispatches the right agents per phase, tracks progress in TASKS.md, suggests /clear between phases.
disable-model-invocation: true
argument-hint: <spec.md> [phase=N] [done=0-N]
agent: project-manager
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - mcp__supabase
  - mcp__context7
---

# /build — Phase-by-Phase SaaS Build

Reads TASKS.md and executes the build phase by phase, dispatching specialist agents and verifying results after each phase.

## Prerequisites

- A `TASKS.md` file exists in the project root (created by `/init-saas` or manually)
- The project has been bootstrapped (Next.js + Supabase + Stripe deps installed)

## Workflow

### 1. Read current state

- Read `TASKS.md` to find the current phase and pending tasks
- Read `CLAUDE.md` for project context
- Identify the next uncompleted phase

### 2. Execute the current phase

Work through the 8 build phases in order. Skip phases that are fully completed.

#### Phase 0: Foundation
- Create/update database migrations for all tables in the spec (via MCP Supabase)
- Add RLS policies for every table (including `stripe.*` schema)
- Generate TypeScript types
- Deploy stripe-sync-engine Edge Function if billing is needed
- **Verify:** migrations applied, types generated, RLS on all tables

#### Phase 1: Auth
- Implement login and signup pages
- Add OAuth callback route (if specified)
- Create middleware auth guard
- Add sign-out functionality
- Follow `/auth` skill patterns for all Supabase auth code
- **Verify:** login/signup/logout work, middleware protects routes

#### Phase 2: Backend
- Create Server Actions for data mutations
- Set up Stripe Checkout via Server Actions (if billing needed)
- Add DB triggers on `stripe.*` tables for business logic (feature provisioning, emails)
- Follow `/stripe-setup` skill patterns for Stripe integration
- **Verify:** build passes, actions callable, Edge Function deployed

#### Phase 3: Data Layer
- Create data fetching helpers (server-side)
- Add subscription gating utilities (query `stripe.*` tables)
- Create shared validation schemas (zod)
- **Verify:** types align, hooks return expected shapes

#### Phase 4: Components
- Build shared UI components (forms, cards, tables, modals)
- Create layout components (sidebar, header, navigation)
- Use shadcn/ui components where applicable
- **Verify:** components render, no TS errors

#### Phase 5: Pages
- Build all route pages and layouts
- Wire up navigation between pages
- Add auth guards at layout level for protected routes
- **Verify:** pages load, nav works, auth guards active

#### Phase 6: Polish
- Add loading states (Suspense boundaries, skeletons)
- Add error boundaries and error pages
- Add empty states for lists/tables
- Mobile responsive adjustments
- Accessibility audit
- **Verify:** all states handled, responsive, keyboard navigable

#### Phase 7: Quality
- Dispatch `testing-specialist` + `security-reviewer` agents in parallel
- Fix any issues found
- Final build verification
- **Verify:** tests pass, no critical security issues

### 3. After each phase

1. **Verify:** Run `npm run build` to check for build errors
2. **Update:** Mark completed tasks in TASKS.md with `[x]`, blocked tasks with `[!]`
3. **Checkpoint:** Summarize what was completed and what's next
4. **Context check:** If 3+ phases have been completed in this session, suggest `/clear` + `/resume`

## Continuing a build

If TASKS.md shows partially completed phases:
1. Start from the first uncompleted task in the earliest incomplete phase
2. Don't redo completed work — verify it exists and move on

## Rules

- Follow SaaS implementation order: DB → RLS → Types → Backend → Frontend
- Cross-reference `/auth` patterns for any auth code
- Cross-reference `/stripe-setup` patterns for any billing code
- Cross-reference `/db-migration` patterns for any migrations
- Run build verification after each phase
- Keep TASKS.md updated as the single source of truth
- Don't implement phases out of order unless explicitly asked
