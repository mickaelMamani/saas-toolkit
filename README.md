# SaaS Toolkit — Claude Code Plugin v2.0

> Build production-ready SaaS applications with Next.js + Supabase + Stripe + Tailwind + shadcn/ui using an orchestrated team of AI agents.

## Quick Start

### Installation

Add the plugin to your project's `.claude/settings.json`:

```json
{
  "plugins": [
    "/path/to/saas-toolkit"
  ]
}
```

### Prerequisites

- Node.js 18+, npm/pnpm
- **Supabase Cloud project** (create at supabase.com) — no local Supabase needed
- **Stripe account** (test mode) — products/prices created via MCP Stripe
- Claude Code with a Pro/Max subscription
- MCP servers configured: `supabase`, `stripe`, `context7` (copy `saas-toolkit/.mcp.json` into your project)

### Recommended Settings

Copy the hooks from `saas-toolkit/recommended-settings.json` into your `.claude/settings.json`:
- **PreToolUse hooks**: restrict Bash to safe commands, prevent writing `.env` files
- **Stop hook**: audible beep when Claude finishes (Windows)
- **Notification hook**: beep on idle prompt

---

## Build a SaaS from Zero

### Step 0: Write Your Spec

Create a markdown describing: elevator pitch, target users, core features, pricing model, auth requirements, integrations. Save as `my-saas-spec.md`.

### Step 1: Bootstrap — `/init-saas my-saas-spec.md`

Scaffolds the entire project: Next.js + auth + Stripe + folder structure + CLAUDE.md. Deploys stripe-sync-engine as Supabase Edge Function, runs initial migrations on cloud via MCP.

### Step 2: Plan — Ask the project-manager agent

> "Use the project-manager agent to plan the build of my-saas-spec.md"

Creates TASKS.md with all phases.

### Step 3: Build (choose your approach)

**Approach A: Subagent Mode (recommended for most)**
```
/build my-saas-spec.md phase=0
/clear
/resume my-saas-spec.md
... repeat ...
```

**Approach B: Agent Team Mode (for large features)**
```
/team-build my-saas-spec.md
```
Spawns backend + frontend + quality teammates.

**Approach C: Hybrid Mode (recommended for complex SaaS)**
- Phase 0-1: `/build` (sequential foundation)
- Phase 2-5: `/team-build` (parallel implementation)
- Phase 6-7: `/build` (sequential polish)

### Step 4: Review & Ship
```
/review
/status my-saas-spec.md
```

---

## Command Reference

### Action Commands (user-triggered)

| Command | Purpose | Example |
|---------|---------|---------|
| `/init-saas` | Bootstrap new project | `/init-saas my-spec.md` |
| `/build` | Phase-by-phase build | `/build my-spec.md phase=0` |
| `/team-build` | Parallel agent team build | `/team-build my-spec.md` |
| `/resume` | Continue after /clear | `/resume my-spec.md` |
| `/status` | Health check dashboard | `/status my-spec.md` |
| `/scaffold` | Generate patterns | `/scaffold crud posts` |
| `/add-feature` | Quick feature add | `/add-feature Add dark mode toggle` |
| `/plan-feature` | Create feature spec | `/plan-feature name="user profiles"` |
| `/review` | Code + security review | `/review` |
| `/hotfix` | Debug and fix issues | `/hotfix "webhook returns 400"` |
| `/commits` | Conventional commit + push | `/commits` |
| `/oneshot` | Single-task completion | `/oneshot task="Add favicon"` |
| `/dev` | Spec-driven development | `/dev` |
| `/explore` | Codebase exploration | `/explore` |

### Knowledge Skills (auto-invoked)

| Skill | Loaded when... |
|-------|----------------|
| `auth` | Building signup, login, OAuth, protected routes |
| `stripe-setup` | Implementing checkout, billing, webhooks |
| `db-migration` | Creating tables, RLS policies, seed data |
| `test` | Writing or running tests |
| `deploy` | Preparing for Vercel deployment |
| `security-scan` | Running security audit |

---

## Agent Team

| Role | Agent | Model | When |
|------|-------|-------|------|
| Project Manager | project-manager | Opus | Plans, delegates, tracks |
| DBA | supabase-specialist | Sonnet | DB phases |
| Backend Dev | stripe-specialist | Sonnet | Backend + payments |
| Frontend Dev | general-purpose | Sonnet | UI phases |
| UI/UX Reviewer | ui-ux-reviewer | Sonnet | Polish phase |
| QA Engineer | testing-specialist | Sonnet | Test phase |
| Security Auditor | security-reviewer | Opus | Review + deploy |
| Code Reviewer | frontend-code-reviewer | Opus | Review phase |
| Explorer | explore-codebase | Haiku | On demand |
| DB Explorer | explore-db | Haiku | On demand |
| Researcher | explore-docs + websearch | Haiku | On demand |

---

## Scaffold Patterns

```
/scaffold crud <resource>        # Full CRUD with migration, RLS, actions, hooks, components, pages
/scaffold billing                # Complete billing management page
/scaffold landing                # Marketing landing page with pricing
/scaffold auth-pages             # Login, signup, forgot-password, reset, OAuth
/scaffold dashboard <name>       # Dashboard with stats, charts, tables
/scaffold settings <name>        # Settings form with Server Action
/scaffold admin-panel            # Admin layout with users, subscriptions
/scaffold onboarding-flow        # Multi-step wizard
```

---

## Agent Teams Setup (Windows)

Agent teams require tmux. On Windows:

**Option 1: WSL2 (recommended)**
```bash
wsl --install
# Inside WSL:
sudo apt install tmux
```

**Option 2: In-process mode (no tmux)**

Teammates run in background. Use Shift+Up/Down to cycle.

Enable in settings:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_SUBAGENT_MODEL": "claude-sonnet-4-5-20250929"
  }
}
```

---

## Context Management

- `/clear` after every 2-3 phases to keep context fresh
- `/resume` reads TASKS.md and picks up where you left off
- `/status` gives a quick health dashboard anytime
- TASKS.md is the single source of truth for progress

---

## Architecture

### Critical Constraints

1. **Supabase Cloud Only** — No local development. All database operations via MCP Supabase.
2. **stripe-sync-engine** — All Stripe data syncs automatically into `stripe.*` schema via Supabase Edge Function. No manual sync tables.
3. **MCP-First** — Agents interact with Supabase and Stripe via MCP servers, not CLI commands.

### MCP Servers

| Server | Purpose |
|--------|---------|
| `supabase` | Database, migrations, RLS, secrets, Edge Functions |
| `stripe` | Products, prices, customers, checkout sessions |
| `context7` | Up-to-date library documentation |

### References

The plugin includes compressed doc indexes (~8KB each) for:
- Next.js App Router patterns
- Supabase + Next.js integration
- Stripe + Next.js integration (including stripe-sync-engine)

---

## Security

The security-reviewer agent checks:
- OWASP Top 10
- RLS coverage on all tables (including `stripe.*` schema)
- No secrets in `NEXT_PUBLIC_*` vars
- Webhook signature verification
- Zod validation on all inputs
- Dependency vulnerabilities

Run anytime: `/security-scan`

---

## Totals

- **11 agents** — project-manager, supabase-specialist, stripe-specialist, security-reviewer, testing-specialist, ui-ux-reviewer, frontend-code-reviewer, explore-codebase, explore-db, explore-docs, websearch
- **20 skills** — 14 action + 6 knowledge
- **3 reference indexes** — Next.js, Supabase, Stripe
