---
description: Quick SaaS pattern scaffolding — CRUD resource, billing, dashboard, settings, landing page
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
user-invocable: true
---

# /scaffold — Quick SaaS Pattern Scaffolding

Generates common SaaS patterns adapted to the project's existing conventions.

## Usage

```
/scaffold [pattern] [options]
```

## Patterns

### `crud [resource]`

Full CRUD for a resource (e.g., `/scaffold crud projects`).

Generates:
- **Migration** — `supabase/migrations/[timestamp]_create_[resource].sql` with table, RLS policies, indexes
- **Types** — Update generated types or add manual type for the resource
- **Server Actions** — `src/app/(protected)/[resource]/actions.ts` with create, read, update, delete
- **List page** — `src/app/(protected)/[resource]/page.tsx` with data table
- **Detail page** — `src/app/(protected)/[resource]/[id]/page.tsx`
- **Create form** — `src/app/(protected)/[resource]/new/page.tsx` with zod validation
- **Edit form** — `src/app/(protected)/[resource]/[id]/edit/page.tsx`

The resource is org-scoped by default (uses `org_id` FK). Add `--user-scoped` for user-level resources.

### `billing`

Stripe billing pages.

Generates:
- **Pricing page** — `src/app/pricing/page.tsx` with plan cards and checkout buttons
- **Billing management** — `src/app/(protected)/settings/billing/page.tsx` with current plan, portal link
- **Checkout action** — `src/app/(protected)/settings/billing/actions.ts` with `createCheckoutSession`, `createPortalSession`

### `dashboard`

Dashboard layout and navigation.

Generates:
- **Dashboard layout** — `src/app/(protected)/layout.tsx` with sidebar + header
- **Sidebar** — `src/components/sidebar.tsx` with nav links
- **Header** — `src/components/header.tsx` with user menu
- **Dashboard home** — `src/app/(protected)/dashboard/page.tsx` with summary cards

### `settings`

Settings pages.

Generates:
- **Settings layout** — `src/app/(protected)/settings/layout.tsx` with tab navigation
- **Profile settings** — `src/app/(protected)/settings/profile/page.tsx`
- **Billing settings** — `src/app/(protected)/settings/billing/page.tsx`
- **Team settings** — `src/app/(protected)/settings/team/page.tsx` with member list, invite form

### `landing`

Marketing landing page.

Generates:
- **Hero section** — `src/components/landing/hero.tsx`
- **Features section** — `src/components/landing/features.tsx`
- **Pricing CTA** — `src/components/landing/pricing-cta.tsx`
- **Footer** — `src/components/landing/footer.tsx`
- **Landing page** — `src/app/page.tsx` composing all sections

## Adaptation Rules

Before generating any files:

1. **Read existing patterns** — Glob for similar files to discover naming conventions, component patterns, and data fetching approaches
2. **Adapt to conventions** — Match the project's existing style (import paths, component structure, styling approach)
3. **Use existing components** — If the project already has a Button, Card, Table, etc., use those instead of creating new ones
4. **Follow existing auth patterns** — Use the project's Supabase client creation pattern

## Output

After scaffolding, report:
- List of files created
- Any manual steps needed (e.g., "add nav link to sidebar", "run migration")
- Suggested next steps

## Rules

- Always read existing project files first to match conventions
- Use shadcn/ui components if available in the project
- All database tables must have RLS enabled
- Use Server Actions for mutations, not API routes
- Use `@supabase/ssr` patterns for all Supabase client creation
- Do not overwrite existing files — warn and skip if a file already exists
