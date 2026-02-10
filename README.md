# saas-claude-marketplace

Private Claude Code plugin marketplace for Next.js + Supabase + Stripe SaaS development.

## What's included

**saas-toolkit** plugin (v2.0.0):
- **10 agents** — codebase explorer, DB explorer, docs explorer, web search, frontend reviewer, Stripe specialist, Supabase specialist, security reviewer, testing specialist, UI/UX reviewer
- **13 skills** — commits, dev workflow, explore, hotfix, oneshot, plan-feature, code review, auth, stripe-setup, db-migration, security-scan, test, deploy
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

- `/commits` — Conventional commit workflow
- `/dev` — Spec-driven development
- `/explore` — Codebase exploration
- `/hotfix` — Hotfix debugging
- `/oneshot` — Quick one-shot tasks
- `/plan-feature` — Feature planning
- `/review` — Code review
- `/auth` — Supabase auth implementation
- `/stripe-setup` — Stripe SaaS integration
- `/db-migration` — Supabase migrations
- `/security-scan` — Security audit
- `/test` — Testing workflows
- `/deploy` — Deployment preparation
