---
name: hotfix
description: Hotfix debugging workflow — reproduce, diagnose, fix, verify.
disable-model-invocation: true
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

# /hotfix — Hotfix Debugging Workflow

Structured workflow for quickly diagnosing and fixing bugs in production or development.

## Phases

### Phase 1: Reproduce

1. **Understand the bug** — Get a clear description of the expected vs actual behavior.
2. **Identify the trigger** — What user action or condition causes the bug?
3. **Check logs** — Look for error messages, stack traces, or relevant log output.
4. **Reproduce locally** — If possible, reproduce the issue in the development environment.

### Phase 2: Diagnose

1. **Trace the code path** — Follow the execution flow from the trigger point to the error.
2. **Identify the root cause** — Find the exact line or logic error causing the bug. Don't stop at symptoms.
3. **Check for related issues** — Is this a one-off bug or part of a pattern? Are there similar bugs elsewhere?
4. **Assess impact** — What's affected? Just this page? All users? Data integrity?

### Phase 3: Fix

1. **Minimal fix** — Fix the root cause with the smallest possible change. Don't refactor surrounding code.
2. **Handle edge cases** — If the bug revealed missing edge case handling, address it.
3. **Don't break other things** — Check that the fix doesn't introduce regressions.

### Phase 4: Verify

1. **Test the fix** — Verify the bug is resolved.
2. **Run existing tests** — Ensure nothing else is broken.
3. **Check related flows** — Test adjacent features that might be affected.
4. **Commit** — Use `fix(scope): description` commit format.

## SaaS Debugging Patterns

### Stripe sync debugging (stripe-sync-engine)
- **Symptom:** Stripe data not appearing in `stripe.*` tables
- **Check Edge Function logs:** Use `mcp__supabase` to view Edge Function logs for the `stripe-sync` function
- **Verify `stripe.*` tables have latest data:** Query `stripe.subscriptions`, `stripe.customers` via MCP Supabase
- **Run `syncSingleEntity()`:** For missing records, trigger a single-entity sync to backfill
- **Check webhook endpoint:** Verify the Edge Function URL is set as the webhook endpoint in Stripe Dashboard (via `mcp__stripe` or Dashboard)
- **Check secrets:** Ensure `DATABASE_URL`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` are set on the Edge Function via MCP Supabase

### Stripe webhook debugging (custom webhook route)
- **Symptom:** Custom webhook endpoint returns 400/500
- **Check raw body parsing:** Ensure `request.text()` is used, NOT `request.json()`. The `constructEvent` function requires the raw string body.
- **Check signature:** Verify `STRIPE_WEBHOOK_SECRET` matches the endpoint's secret in Stripe Dashboard (different for test vs live, different for Stripe CLI vs dashboard).
- **Check event delivery:** In Stripe Dashboard → Webhooks → select endpoint → view event attempts and response codes.
- **Check idempotency:** Are duplicate events being processed? Check if event ID is stored before processing.
- **Local testing:** Run `stripe listen --forward-to localhost:3000/api/webhooks/stripe` and check CLI output.

### Supabase RLS debugging
- **Symptom:** Query returns empty results when data exists
- **Check RLS policies:** `SELECT * FROM pg_policies WHERE tablename = 'table_name';`
- **Check as user:** Test the query with the user's JWT in Supabase SQL editor using `set request.jwt.claims = '{"sub": "user-uuid"}';`
- **Check policy logic:** Ensure `auth.uid()` matches the column being checked (e.g., `user_id`, `created_by`).
- **Bypass for debugging:** Temporarily use `service_role` key in server-side code to verify data exists, then fix the policy.
- **Common mistake:** Missing policy for the specific operation (e.g., has SELECT but not UPDATE policy).

### Auth flow debugging
- **Symptom:** Redirect loops, stuck on login, session lost
- **Check middleware:** Is `middleware.ts` running on the right routes? Check `config.matcher`.
- **Check cookies:** Are Supabase auth cookies being set/refreshed? Check browser DevTools → Application → Cookies.
- **Check `getUser()` vs `getSession()`:** `getSession()` doesn't validate the JWT — use `getUser()` for reliable auth checks.
- **OAuth callback:** Check the `/auth/callback` route handler — is it exchanging the code for a session?
- **Redirect URL:** Ensure the redirect URL in Supabase Dashboard → Auth → URL Configuration matches your app's URL.

### Subscription state debugging
- **Symptom:** User has active subscription in Stripe but app shows free tier
- **Check Stripe Dashboard:** Verify the subscription status, customer ID, and product/price IDs (via `mcp__stripe`)
- **Check `stripe.subscriptions` table:** Query via `mcp__supabase` SQL for the user's `stripe_customer_id`
- **Compare Stripe vs DB:** Compare Stripe dashboard data (via `mcp__stripe`) with `stripe.subscription` table data (via `mcp__supabase`)
- **Check webhook delivery:** Was the event delivered to the Edge Function? Check Stripe webhook logs + Edge Function logs
- **Check stripe-sync-engine:** If data is missing, run `syncSingleEntity()` for the specific customer/subscription
- **Check gating logic:** Is the subscription check querying the right table/column? (`status = 'active'` vs `status IN ('active', 'trialing')`)
- **Check RLS:** Can the authenticated user actually read the `stripe.*` tables? Test the RLS policy

## Rules

- Focus on the root cause, not symptoms
- Keep fixes minimal and focused
- Don't refactor or improve code unrelated to the bug
- If the fix is complex, explain the reasoning
- If you can't find the root cause, say so — don't guess
