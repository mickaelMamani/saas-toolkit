---
name: db-migration
description: Supabase database migrations. Use when creating tables, modifying schema, adding RLS policies, or writing seed data.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - mcp__supabase
---

# /db-migration — Supabase Migrations

Create and manage Supabase database migrations with proper patterns for SaaS applications.

## Migration Workflow

**All migrations target Supabase Cloud via MCP.** Use `mcp__supabase` to run SQL directly on the cloud project. Never use `supabase start` or `supabase db push`.

### 1. Write the migration SQL

Write idempotent SQL that can be re-run safely. For local tracking, keep files in `supabase/migrations/` but apply them via MCP.

### 2. Write idempotent SQL

Always write migrations that can be re-run safely:

```sql
-- Create table (idempotent)
CREATE TABLE IF NOT EXISTS public.my_table (...);

-- Add column (idempotent)
ALTER TABLE public.my_table ADD COLUMN IF NOT EXISTS new_column text;

-- Create index (idempotent)
CREATE INDEX IF NOT EXISTS idx_my_table_column ON public.my_table(column);

-- Enable RLS (idempotent)
ALTER TABLE public.my_table ENABLE ROW LEVEL SECURITY;

-- Create policy (drop + create for idempotency)
DROP POLICY IF EXISTS "policy_name" ON public.my_table;
CREATE POLICY "policy_name" ON public.my_table ...;
```

### 3. Apply migration via MCP

Run the SQL directly on the cloud project using `mcp__supabase` SQL execution tools. Do NOT use `supabase db push` or local development.

### 4. Generate types

```bash
npx supabase gen types typescript --project-id <project-id> > lib/database.types.ts
```

## New Table Checklist

Every new table must have:

- [ ] `id uuid DEFAULT gen_random_uuid() PRIMARY KEY`
- [ ] `created_at timestamptz DEFAULT now() NOT NULL`
- [ ] `updated_at timestamptz DEFAULT now() NOT NULL`
- [ ] RLS enabled: `ALTER TABLE public.xxx ENABLE ROW LEVEL SECURITY;`
- [ ] RLS policies for SELECT, INSERT, UPDATE, DELETE
- [ ] Indexes on foreign keys
- [ ] Indexes on frequently queried columns
- [ ] `updated_at` trigger

### Updated_at trigger

```sql
-- Create the trigger function (once per project)
CREATE OR REPLACE FUNCTION public.handle_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to table
CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON public.my_table
  FOR EACH ROW
  EXECUTE FUNCTION public.handle_updated_at();
```

## RLS Policy Templates

### Owner access (user-scoped data)

```sql
CREATE POLICY "Users can view own data" ON public.my_table
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can create own data" ON public.my_table
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own data" ON public.my_table
  FOR UPDATE USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own data" ON public.my_table
  FOR DELETE USING (auth.uid() = user_id);
```

### Org member access (multi-tenant)

```sql
CREATE POLICY "Org members can view data" ON public.my_table
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM public.org_members
      WHERE org_members.org_id = my_table.org_id
        AND org_members.user_id = auth.uid()
    )
  );
```

### Admin-only write access

```sql
CREATE POLICY "Only admins can modify" ON public.my_table
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM public.org_members
      WHERE org_members.org_id = my_table.org_id
        AND org_members.user_id = auth.uid()
        AND org_members.role = 'admin'
    )
  );
```

### Public read access

```sql
CREATE POLICY "Anyone can read" ON public.my_table
  FOR SELECT USING (true);
```

## SaaS Schema Patterns

### Profiles (extends auth.users)

```sql
CREATE TABLE public.profiles (
  id uuid REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
  full_name text,
  avatar_url text,
  stripe_customer_id text UNIQUE,
  created_at timestamptz DEFAULT now() NOT NULL,
  updated_at timestamptz DEFAULT now() NOT NULL
);

ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile"
  ON public.profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users can update own profile"
  ON public.profiles FOR UPDATE USING (auth.uid() = id);

-- Auto-create profile on signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, full_name, avatar_url)
  VALUES (NEW.id, NEW.raw_user_meta_data->>'full_name', NEW.raw_user_meta_data->>'avatar_url');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW
  EXECUTE FUNCTION public.handle_new_user();
```

### Organizations

```sql
CREATE TABLE public.organizations (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  name text NOT NULL,
  slug text UNIQUE NOT NULL,
  stripe_customer_id text UNIQUE,
  created_at timestamptz DEFAULT now() NOT NULL,
  updated_at timestamptz DEFAULT now() NOT NULL
);

CREATE TABLE public.org_members (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  org_id uuid REFERENCES public.organizations(id) ON DELETE CASCADE NOT NULL,
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  role text NOT NULL DEFAULT 'member' CHECK (role IN ('owner', 'admin', 'member')),
  created_at timestamptz DEFAULT now() NOT NULL,
  UNIQUE(org_id, user_id)
);

CREATE INDEX idx_org_members_user_id ON public.org_members(user_id);
CREATE INDEX idx_org_members_org_id ON public.org_members(org_id);
```

### Subscriptions (manual — only if NOT using stripe-sync-engine)

**Note:** If using `@supabase/stripe-sync-engine` (recommended), the `stripe.*` schema manages subscription data automatically. Only create this manual table if you have a specific reason not to use the sync engine.

```sql
CREATE TABLE public.subscriptions (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  stripe_customer_id text NOT NULL,
  stripe_subscription_id text UNIQUE,
  stripe_price_id text,
  status text NOT NULL DEFAULT 'inactive',
  current_period_start timestamptz,
  current_period_end timestamptz,
  cancel_at_period_end boolean DEFAULT false,
  created_at timestamptz DEFAULT now() NOT NULL,
  updated_at timestamptz DEFAULT now() NOT NULL
);

CREATE INDEX idx_subscriptions_user_id ON public.subscriptions(user_id);
CREATE INDEX idx_subscriptions_stripe_customer_id ON public.subscriptions(stripe_customer_id);
```

### Audit Logs

```sql
CREATE TABLE public.audit_logs (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) ON DELETE SET NULL,
  org_id uuid REFERENCES public.organizations(id) ON DELETE SET NULL,
  action text NOT NULL,
  entity_type text NOT NULL,
  entity_id uuid,
  metadata jsonb DEFAULT '{}',
  ip_address inet,
  created_at timestamptz DEFAULT now() NOT NULL
);

CREATE INDEX idx_audit_logs_user_id ON public.audit_logs(user_id);
CREATE INDEX idx_audit_logs_org_id ON public.audit_logs(org_id);
CREATE INDEX idx_audit_logs_created_at ON public.audit_logs(created_at);
```

## Seed Data

For local development, add seed data in `supabase/seed.sql`:

```sql
-- Only runs on local reset, not on production
INSERT INTO public.organizations (id, name, slug) VALUES
  ('00000000-0000-0000-0000-000000000001', 'Acme Corp', 'acme');
```

## Rules

- Always write idempotent migrations
- Always enable RLS on new tables
- Always add indexes on foreign keys
- Always include `created_at` and `updated_at`
- Always generate types after migration changes
- Use `timestamptz` not `timestamp`
- Use `text` not `varchar`
- Use `uuid` primary keys
- **All migrations target Supabase Cloud via `mcp__supabase`** — never local
- **`stripe.*` schema is managed by stripe-sync-engine** — only add RLS policies to `stripe.*` tables, never create/modify the tables themselves
