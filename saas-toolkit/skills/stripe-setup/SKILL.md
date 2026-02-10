---
description: Complete Stripe SaaS integration — checkout, webhooks, portal, subscription sync
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - WebSearch
  - WebFetch
  - mcp__supabase__*
user-invocable: true
---

# /stripe-setup — Stripe SaaS Integration

Complete Stripe integration for a SaaS application with Next.js and Supabase.

## Setup

### 1. Install & configure

```bash
npm install stripe @supabase/stripe-sync-engine
```

**Environment variables:**
```env
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

**Stripe client** (`lib/stripe.ts`):
```typescript
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2025-12-18.acacia',
  typescript: true,
});
```

### 2. Product & pricing model

Define products and prices in Stripe Dashboard or via API:
```typescript
// Typical SaaS pricing structure
// Product: "Pro Plan"
//   - Price: $19/month (price_xxx)
//   - Price: $190/year (price_yyy)
// Product: "Enterprise Plan"
//   - Price: $49/month (price_zzz)
```

Store price IDs in environment variables or a config file — never hardcode.

### 3. Stripe Sync Engine (recommended)

Use `@supabase/stripe-sync-engine` to auto-sync all Stripe data into a `stripe` schema in your Supabase database. This replaces manual subscription tables and most webhook handling.

**Run migrations** (one-time setup):
```typescript
import { runMigrations } from '@supabase/stripe-sync-engine';

await runMigrations({
  databaseUrl: process.env.DATABASE_URL!,
  schema: 'stripe',
});
```

**Deploy as Supabase Edge Function** (`supabase/functions/stripe-sync/index.ts`):
```typescript
import 'jsr:@supabase/functions-js/edge-runtime.d.ts'
import { StripeSync } from 'npm:@supabase/stripe-sync-engine'

const stripeSync = new StripeSync({
  poolConfig: {
    connectionString: Deno.env.get('DATABASE_URL')!,
    max: 20,
    keepAlive: true,
  },
  stripeWebhookSecret: Deno.env.get('STRIPE_WEBHOOK_SECRET')!,
  stripeSecretKey: Deno.env.get('STRIPE_SECRET_KEY')!,
  backfillRelatedEntities: false,
  autoExpandLists: true,
})

Deno.serve(async (req) => {
  const rawBody = new Uint8Array(await req.arrayBuffer())
  const stripeSignature = req.headers.get('stripe-signature')
  await stripeSync.processWebhook(rawBody, stripeSignature)
  return new Response(null, { status: 202 })
})
```

**Grant access** to the `stripe` schema for your app's database role:
```sql
GRANT USAGE ON SCHEMA stripe TO authenticated;
GRANT SELECT ON ALL TABLES IN SCHEMA stripe TO authenticated;
```

This creates tables for: customers, subscriptions, products, prices, invoices, payment_intents, charges, and more — all synced automatically via Stripe webhooks.

### 4. Linking Stripe customers to your users

You still need a mapping between `auth.users` and Stripe customers. Add `stripe_customer_id` to your profiles table:

```sql
ALTER TABLE public.profiles ADD COLUMN stripe_customer_id text;
CREATE INDEX idx_profiles_stripe_customer_id ON public.profiles(stripe_customer_id);
```

Query subscription status by joining your profiles with the sync engine's `stripe.subscriptions` table:
```sql
SELECT s.status, s.current_period_end
FROM stripe.subscriptions s
JOIN stripe.customers c ON s.customer = c.id
JOIN public.profiles p ON p.stripe_customer_id = c.id
WHERE p.id = auth.uid();
```

### 4. Checkout Session (Server Action)

```typescript
'use server';
import { stripe } from '@/lib/stripe';
import { createClient } from '@/lib/supabase/server';
import { redirect } from 'next/navigation';

export async function createCheckoutSession(priceId: string) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) redirect('/login');

  // Get or create Stripe customer
  let { data: subscription } = await supabase
    .from('subscriptions')
    .select('stripe_customer_id')
    .eq('user_id', user.id)
    .single();

  let customerId = subscription?.stripe_customer_id;
  if (!customerId) {
    const customer = await stripe.customers.create({
      email: user.email,
      metadata: { user_id: user.id },
    });
    customerId = customer.id;
  }

  const session = await stripe.checkout.sessions.create({
    customer: customerId,
    line_items: [{ price: priceId, quantity: 1 }],
    mode: 'subscription',
    success_url: `${process.env.NEXT_PUBLIC_SITE_URL}/dashboard?success=true`,
    cancel_url: `${process.env.NEXT_PUBLIC_SITE_URL}/pricing?canceled=true`,
    metadata: { user_id: user.id },
  });

  if (session.url) redirect(session.url);
}
```

### 5. Webhook handling

**With Stripe Sync Engine (recommended):** All Stripe data sync is handled by the Edge Function deployed in step 3. The sync engine processes 80+ webhook event types automatically.

For **custom business logic** (e.g., sending emails, provisioning access after checkout), add a separate Next.js webhook route that handles only your app-specific events:

**`app/api/webhooks/stripe/route.ts`**:
```typescript
import { stripe } from '@/lib/stripe';
import { createClient } from '@supabase/supabase-js';
import { NextResponse } from 'next/server';

// Use service role for webhook — no user context
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(request: Request) {
  const body = await request.text(); // MUST be raw text
  const signature = request.headers.get('stripe-signature')!;

  let event;
  try {
    event = stripe.webhooks.constructEvent(body, signature, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch (err) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  // Only handle app-specific business logic here
  // Data sync (subscriptions, invoices, etc.) is handled by stripe-sync-engine
  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutComplete(event.data.object);
      break;
    case 'invoice.payment_failed':
      await handlePaymentFailed(event.data.object);
      break;
  }

  return NextResponse.json({ received: true });
}
```

**Tip:** Configure two Stripe webhook endpoints — one pointing to the Edge Function (all events), one pointing to your Next.js route (only app-specific events).

### 6. Customer Portal

```typescript
'use server';
import { stripe } from '@/lib/stripe';
import { createClient } from '@/lib/supabase/server';
import { redirect } from 'next/navigation';

export async function createPortalSession() {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) redirect('/login');

  const { data: subscription } = await supabase
    .from('subscriptions')
    .select('stripe_customer_id')
    .eq('user_id', user.id)
    .single();

  if (!subscription?.stripe_customer_id) redirect('/pricing');

  const session = await stripe.billingPortal.sessions.create({
    customer: subscription.stripe_customer_id,
    return_url: `${process.env.NEXT_PUBLIC_SITE_URL}/dashboard/billing`,
  });

  redirect(session.url);
}
```

### 7. Subscription gating

Query the `stripe.subscriptions` table (populated by the sync engine) joined with your profiles:

```typescript
// lib/subscription.ts
import { createClient } from '@/lib/supabase/server';

export async function getSubscription() {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return null;

  // Get the user's stripe_customer_id from profiles
  const { data: profile } = await supabase
    .from('profiles')
    .select('stripe_customer_id')
    .eq('id', user.id)
    .single();

  if (!profile?.stripe_customer_id) return null;

  // Query the stripe schema (synced by stripe-sync-engine)
  const { data } = await supabase
    .schema('stripe')
    .from('subscriptions')
    .select('id, status, current_period_end, items:subscription_items(price:prices(*))')
    .eq('customer', profile.stripe_customer_id)
    .in('status', ['active', 'trialing'])
    .single();

  return data;
}

export async function requireSubscription() {
  const subscription = await getSubscription();
  if (!subscription) redirect('/pricing');
  return subscription;
}
```

### 8. Testing with Stripe CLI

```bash
# Listen for webhooks locally
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed
```

## Rules

- NEVER use Charges API, Card Element, or Sources — use Checkout Sessions
- ALWAYS verify webhook signatures with `constructEvent()`
- ALWAYS use `request.text()` for webhook body — never `request.json()`
- Use `@supabase/stripe-sync-engine` for Stripe data sync — don't build custom subscription tables
- Query subscription status from the `stripe` schema (not the Stripe API)
- Use Server Actions for creating Checkout Sessions and Portal Sessions
- Use `service_role` key in webhook handler (no user context available)
- Keep Stripe secret key server-side only — never expose to client
- Deploy the sync engine as a Supabase Edge Function for webhook processing
- Use a separate Next.js webhook route only for custom business logic (emails, provisioning)
