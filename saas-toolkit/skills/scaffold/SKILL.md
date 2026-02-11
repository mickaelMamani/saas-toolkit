---
name: scaffold
description: Generate boilerplate for common SaaS patterns. Supports crud, dashboard-page, settings-section, billing-page, admin-panel, landing-page, auth-pages, onboarding-flow.
disable-model-invocation: true
argument-hint: <pattern> <name>
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - mcp__supabase
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

### `admin-panel`

Admin layout with user management.

Generates:
- **Admin layout** — `src/app/(protected)/admin/layout.tsx` with admin nav
- **Users list** — `src/app/(protected)/admin/users/page.tsx` with data table
- **Subscriptions** — `src/app/(protected)/admin/subscriptions/page.tsx`
- **Impersonation** — helper to view as specific user (read-only)

### `auth-pages`

Complete authentication pages.

Generates:
- **Login** — `src/app/(auth)/login/page.tsx` with email/password + OAuth buttons
- **Signup** — `src/app/(auth)/signup/page.tsx` with form + validation
- **Forgot password** — `src/app/(auth)/forgot-password/page.tsx`
- **Reset password** — `src/app/(auth)/reset-password/page.tsx`
- **OAuth callback** — `src/app/auth/callback/route.ts`

### `onboarding-flow`

Multi-step onboarding wizard.

Generates:
- **Onboarding layout** — `src/app/(protected)/onboarding/layout.tsx` with progress bar
- **Step 1: Profile** — `src/app/(protected)/onboarding/profile/page.tsx`
- **Step 2: Team** — `src/app/(protected)/onboarding/team/page.tsx`
- **Step 3: Plan** — `src/app/(protected)/onboarding/plan/page.tsx`
- **Actions** — `src/app/(protected)/onboarding/actions.ts`

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
