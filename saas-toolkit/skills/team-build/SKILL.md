---
description: Parallel build with agent teams (experimental) or coordinated subagents
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

# /team-build — Multi-Agent Team Coordination

Coordinates parallel builds using either Claude Code agent teams (experimental) or the Task tool for coordinated subagent dispatching.

## Option 1: Agent Teams (Experimental)

Agent teams allow multiple Claude instances to work on the same codebase simultaneously with file-level ownership.

### Setup

1. Enable the experimental feature:
   ```bash
   export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
   ```

2. Start Claude Code with team mode enabled

### Recommended team composition

| Agent | Focus area | File ownership |
|-------|-----------|----------------|
| **Backend lead** | DB, migrations, Server Actions, API routes | `supabase/`, `src/app/**/actions.ts`, `src/app/api/`, `src/lib/` |
| **Frontend lead** | Pages, components, layouts, styles | `src/app/**/page.tsx`, `src/app/**/layout.tsx`, `src/components/` |
| **Quality lead** | Tests, security review, type checking | `src/__tests__/`, `tests/`, `*.test.ts` |

### File ownership rules

To prevent conflicts, each agent owns specific file patterns:
- **Never overlap** — only one agent should modify a given file
- **Shared types** — generate types once (backend lead), others read only
- **Integration points** — backend lead creates actions, frontend lead imports them
- Communication through file conventions, not direct messaging

## Option 2: Coordinated Subagents (Fallback)

When agent teams aren't available, use the Task tool to dispatch subagents in coordinated waves.

### Wave-based execution

#### Wave 1: Explore
Dispatch in parallel:
- `explore-codebase` — map project structure
- `explore-db` — map database schema

Wait for both to complete before proceeding.

#### Wave 2: Foundation
Sequential:
- Database migrations and RLS
- Type generation

#### Wave 3: Backend + Frontend (parallel where safe)
Dispatch in parallel:
- **Backend task** — Server Actions, API routes, webhooks
- **Frontend task** — Layout, navigation, shared components (that don't depend on backend)

Then sequential:
- **Integration** — Pages that consume Server Actions (depends on backend completion)

#### Wave 4: Quality (parallel)
Dispatch in parallel:
- `testing-specialist` — write and run tests
- `security-reviewer` — security audit
- `ui-ux-reviewer` — UI/UX review

### Dispatch template

When using the Task tool for subagent coordination:

```
Task: [specific deliverable]

Context:
- Project uses Next.js App Router + Supabase + Stripe
- See CLAUDE.md for project conventions
- See TASKS.md for overall build plan

Files to create/modify:
- [explicit paths]

Do NOT modify:
- [files owned by other agents/tasks]

Success criteria:
- [verifiable outcomes]
```

## Choosing between options

| Factor | Agent Teams | Coordinated Subagents |
|--------|------------|----------------------|
| Setup | Requires env var + experimental flag | Works out of the box |
| Parallelism | True parallel (multiple Claude instances) | Sequential with parallel dispatches |
| Conflict risk | Low (file ownership enforced) | Medium (manual coordination) |
| Best for | Large projects, long builds | Smaller projects, specific phases |

## Rules

- Always define clear file ownership boundaries
- Never let two agents modify the same file
- Backend before frontend for data-dependent features
- Quality checks always run last
- Update TASKS.md after each wave completes
