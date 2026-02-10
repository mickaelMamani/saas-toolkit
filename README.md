# saas-claude-marketplace

Private Claude Code plugin marketplace for Next.js + Supabase + Stripe SaaS development.

## What's included

**saas-toolkit** plugin (v3.0.0):
- **11 agents** — codebase explorer, DB explorer, docs explorer, web search, frontend reviewer, Stripe specialist, Supabase specialist, security reviewer, testing specialist, UI/UX reviewer, project manager
- **20 skills** — commits, dev workflow, explore, hotfix, oneshot, plan-feature, code review, auth, stripe-setup, db-migration, security-scan, test, deploy, init-saas, build, resume, status, scaffold, add-feature, team-build
- **3 reference indexes** — Next.js App Router, Supabase + Next.js, Stripe + Next.js

## Setup

### 1. Add the marketplace to your project

In your project's `.claude/settings.json`, add the marketplace path:

```json
{
  "plugins": [
    "/Users/mamani/Documents/Projets/saas-claude-marketplace/saas-toolkit"
  ]
}
```

### 2. Configure MCP servers

Copy or merge `saas-toolkit/.mcp.json` into your project's `.mcp.json`, then add your project-specific servers (Supabase, etc.).

### 3. Project-specific CLAUDE.md

Create a `CLAUDE.md` in your project root with project-specific context (tech stack, conventions, file structure).

### 4. Recommended hooks (optional)

See `saas-toolkit/recommended-settings.json` for recommended hook configurations that restrict Bash commands to safe operations and prevent writing to `.env` files.

## Skills usage

Invoke skills with slash commands in Claude Code:

### Development
- `/commits` — Conventional commit workflow
- `/dev` — Spec-driven development
- `/explore` — Codebase exploration
- `/hotfix` — Hotfix debugging
- `/oneshot` — Quick one-shot tasks
- `/plan-feature` — Feature planning
- `/review` — Code review

### SaaS Stack
- `/auth` — Supabase auth implementation
- `/stripe-setup` — Stripe SaaS integration
- `/db-migration` — Supabase migrations
- `/security-scan` — Security audit
- `/test` — Testing workflows
- `/deploy` — Deployment preparation

### Orchestration (v3.0.0)
- `/init-saas` — Bootstrap new SaaS project from spec
- `/build` — Phase-by-phase build orchestration
- `/resume` — Resume after context reset
- `/status` — Project health check
- `/scaffold` — Quick SaaS pattern scaffolding
- `/add-feature` — Lightweight feature implementation
- `/team-build` — Multi-agent team coordination

## Building a SaaS from zero

The orchestration skills let you go from an idea to a working SaaS app in a structured, phase-by-phase workflow.

### Step 1: Bootstrap the project

Run `/init-saas` with your project spec — either a description or a spec file:

```
/init-saas

Build a project management SaaS called "Taskflow":
- Users sign up and create organizations
- Org members can create projects with tasks (kanban board)
- Free plan: 1 project, 10 tasks. Pro plan ($19/mo): unlimited
- Stripe billing with customer portal
```

This scaffolds a full Next.js + Supabase + Stripe project: app structure, Supabase clients, middleware, initial DB migrations, Stripe sync engine Edge Function, shadcn/ui, `.env.example`, `CLAUDE.md`, and a `TASKS.md` build plan.

### Step 2: Build phase by phase

Run `/build` to start executing the build plan from `TASKS.md`:

```
/build
```

The build runs through 8 phases in order:

| Phase | What happens | Agents involved |
|-------|-------------|-----------------|
| 0. Foundation | DB migrations, RLS policies, type generation | supabase-specialist, explore-db |
| 1. Auth | Login/signup pages, OAuth, middleware guard | supabase-specialist |
| 2. Backend | Server Actions, API routes, Stripe webhooks | stripe-specialist |
| 3. Data layer | Fetch helpers, subscription gating, validation | — |
| 4. Components | UI components, forms, layouts (shadcn/ui) | explore-codebase |
| 5. Pages | Routes, navigation, auth guards | — |
| 6. Polish | Loading/error/empty states, mobile, a11y | ui-ux-reviewer |
| 7. Quality | Tests + security audit | testing-specialist, security-reviewer |

After each phase, the build verifies (`npm run build`), updates `TASKS.md`, and checkpoints progress.

### Step 3: Manage context

Claude Code has a finite context window. After 2-3 phases, run `/clear` to free memory, then:

```
/resume
```

This reads `TASKS.md`, verifies completed work, shows a progress summary, and continues from where you left off.

### Step 4: Check project health

At any point, run `/status` for a quick dashboard:

```
/status
```

```
| Area         | Status | Details                   |
|--------------|--------|---------------------------|
| Tasks        | 14/22  | Phase 4: Components       |
| Build        | Pass   |                           |
| Types        | Pass   |                           |
| DB (tables)  | 6      | All have RLS              |
| Git          | Clean  | master, up to date        |
```

### Adding features later

For individual features after the initial build, use `/add-feature` instead of the full `/build` cycle:

```
/add-feature Add a notification system — in-app + email notifications
when a task is assigned to a user
```

For common SaaS patterns, use `/scaffold`:

```
/scaffold crud invoices
/scaffold billing
/scaffold dashboard
/scaffold settings
/scaffold landing
```

### Parallel builds (experimental)

For large projects, `/team-build` coordinates multiple agents working in parallel — either via Claude Code agent teams or coordinated subagent dispatching with file ownership boundaries.
