---
description: Hotfix debugging workflow — reproduce, diagnose, fix, verify
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Task
  - mcp__supabase__*
user-invocable: true
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

### Stripe webhook debugging
- **Symptom:** Webhook endpoint returns 400/500, events not processed
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
- **Check Stripe Dashboard:** Verify the subscription status, customer ID, and product/price IDs.
- **Check Supabase table:** Query the `subscriptions` table for the user's `stripe_customer_id`.
- **Check webhook processing:** Was the `customer.subscription.updated` event delivered and processed? Check Stripe webhook logs.
- **Check sync logic:** Does the webhook handler update the Supabase `subscriptions` table with the correct fields?
- **Check gating logic:** Is the subscription check querying the right table/column? (`status = 'active'` vs `status IN ('active', 'trialing')`)

## Rules

- Focus on the root cause, not symptoms
- Keep fixes minimal and focused
- Don't refactor or improve code unrelated to the bug
- If the fix is complex, explain the reasoning
- If you can't find the root cause, say so — don't guess
