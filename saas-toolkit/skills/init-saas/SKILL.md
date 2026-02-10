---
description: Bootstrap a new Next.js + Supabase + Stripe SaaS project from a spec
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - WebSearch
  - WebFetch
  - mcp__supabase__*
user-invocable: true
---

# /init-saas — Bootstrap a New SaaS Project

Scaffolds a complete Next.js + Supabase + Stripe SaaS project from a spec file or description.

## Input

The user provides either:
- A spec file path (read it first)
- A project description in the message

Extract: project name, core features, data model, billing requirements.

## Steps

### 1. Create Next.js app

```bash
npx create-next-app@latest [project-name] --typescript --tailwind --eslint --app --src-dir --no-import-alias
```

### 2. Install SaaS dependencies

```bash
npm install @supabase/supabase-js @supabase/ssr stripe zod
npm install -D supabase
```

### 3. Create Supabase clients

Create the three client patterns from the `/auth` skill:

- `src/lib/supabase/server.ts` — Server Component / Server Action client using `createServerClient` from `@supabase/ssr`
- `src/lib/supabase/client.ts` — Browser client using `createBrowserClient` from `@supabase/ssr`
- `src/lib/supabase/middleware.ts` — Middleware client with session refresh and auth guard

### 4. Create middleware

Create `src/middleware.ts` that calls `updateSession` from the middleware client. Protect `/dashboard` and other app routes.

### 5. Create Stripe client

Create `src/lib/stripe/client.ts`:
```typescript
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-12-18.acacia',
  typescript: true,
});
```

### 6. Initialize Supabase locally

```bash
npx supabase init
```

### 7. Initial database migrations

Create migration for core SaaS tables:

- **profiles** — `id` (references auth.users), `full_name`, `avatar_url`, `stripe_customer_id`, timestamps
- **organizations** — `id`, `name`, `slug` (unique), `stripe_customer_id`, timestamps
- **org_members** — `id`, `org_id` (FK), `user_id` (FK), `role` (check: owner/admin/member), timestamps, unique(org_id, user_id)
- **subscriptions** — `id`, `org_id` (FK), `stripe_subscription_id`, `stripe_price_id`, `status`, `current_period_start`, `current_period_end`, timestamps

Enable RLS on all tables. Add basic policies:
- profiles: users can read/update own profile
- organizations: members can read their orgs
- org_members: members can read memberships in their orgs
- subscriptions: org members can read their org's subscription

Add trigger: auto-create profile on auth.users insert.

### 8. Initialize shadcn/ui

```bash
npx shadcn@latest init -d
```

### 9. Create .env.example

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### 10. Generate project CLAUDE.md

Create a `CLAUDE.md` in the project root with:
- Tech stack summary (Next.js App Router, Supabase, Stripe, shadcn/ui)
- File structure overview
- Key conventions (Server Actions for mutations, RLS-first, @supabase/ssr patterns)
- Database schema summary

### 11. Generate TASKS.md

Based on the spec, create a `TASKS.md` with all phases and tasks needed to build the full application. Follow the project-manager agent's TASKS.md format with phases 0-7.

## Output

After completion, present:
- List of created files
- Next steps (set up Supabase project, add env vars, run `/build` to start building)
- Any spec items that need clarification

## Rules

- Use App Router (not Pages Router)
- Use `src/` directory
- Use `@supabase/ssr` (never `@supabase/supabase-js` directly for client creation)
- Use `getUser()` not `getSession()` for auth checks
- Enable RLS on every table
- Do NOT commit `.env` files — only create `.env.example`
