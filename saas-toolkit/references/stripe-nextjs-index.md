# Stripe + Next.js Reference Index

Compressed reference for Stripe integration with Next.js App Router.

## Stripe Client Initialization

```typescript
// lib/stripe.ts (server-side only)
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2025-12-18.acacia',
  typescript: true,
});
```

**NEVER** import this file in Client Components or files with `"use client"`.

## stripe-sync-engine (Recommended)

Auto-sync all Stripe data into a `stripe` schema in your Supabase database.

### Edge Function deployment

```typescript
// supabase/functions/stripe-sync/index.ts
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
  return new Response(JSON.stringify({ received: true }), { status: 200 })
})
```

### Querying `stripe.*` schema from Next.js

```typescript
// Query subscription status via Supabase client
const { data } = await supabase
  .schema('stripe')
  .from('subscriptions')
  .select('id, status, current_period_end, items:subscription_items(price:prices(*))')
  .eq('customer', profile.stripe_customer_id)
  .in('status', ['active', 'trialing'])
  .single();
```

### RLS on `stripe.*` tables

```sql
CREATE POLICY "Users can view own stripe data" ON stripe.customers
  FOR SELECT USING (
    id IN (SELECT stripe_customer_id FROM public.profiles WHERE id = auth.uid())
  );

CREATE POLICY "Users can view own subscriptions" ON stripe.subscriptions
  FOR SELECT USING (
    customer IN (SELECT stripe_customer_id FROM public.profiles WHERE id = auth.uid())
  );
```

### Business logic via DB triggers

```sql
-- Trigger on stripe.subscriptions for feature provisioning
CREATE OR REPLACE FUNCTION handle_subscription_change()
RETURNS TRIGGER AS $$
BEGIN
  -- React to subscription status changes
  -- e.g., provision features, send notifications
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_subscription_change
  AFTER INSERT OR UPDATE ON stripe.subscriptions
  FOR EACH ROW
  EXECUTE FUNCTION handle_subscription_change();
```

## Checkout Session Pattern

### Server Action

```typescript
'use server';
import { stripe } from '@/lib/stripe';
import { redirect } from 'next/navigation';

export async function createCheckoutSession(priceId: string) {
  // 1. Verify user authentication
  // 2. Get or create Stripe Customer (store on profiles.stripe_customer_id)
  // 3. Create Checkout Session
  const session = await stripe.checkout.sessions.create({
    customer: customerId,
    line_items: [{ price: priceId, quantity: 1 }],
    mode: 'subscription', // or 'payment' for one-time
    success_url: `${process.env.NEXT_PUBLIC_SITE_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_SITE_URL}/pricing`,
    metadata: { user_id: userId },
  });

  if (session.url) redirect(session.url);
}
```

### Client Component

```typescript
'use client';
export function PricingButton({ priceId }: { priceId: string }) {
  return (
    <form action={createCheckoutSession.bind(null, priceId)}>
      <button type="submit">Subscribe</button>
    </form>
  );
}
```

## Customer Portal

```typescript
'use server';
import { stripe } from '@/lib/stripe';

export async function createPortalSession(customerId: string) {
  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${process.env.NEXT_PUBLIC_SITE_URL}/dashboard/billing`,
  });
  redirect(session.url);
}
```

Use Customer Portal for: plan changes, payment method updates, invoice history, cancellations.

## Subscription Lifecycle Events

| Event | When | Action |
|-------|------|--------|
| `checkout.session.completed` | User completes checkout | Create subscription record, provision access |
| `customer.subscription.created` | Subscription created | Initial record (often redundant with checkout) |
| `customer.subscription.updated` | Plan change, renewal, trial end | Update status, price_id, period dates |
| `customer.subscription.deleted` | Subscription canceled (end of period) | Revoke access, update status |
| `invoice.payment_succeeded` | Payment processed | Clear payment failure flags |
| `invoice.payment_failed` | Payment failed | Notify user, set grace period |

**Note:** stripe-sync-engine handles writing all these events to the `stripe.*` schema automatically. Use DB triggers for app-specific reactions.

## Subscription Status Values

| Status | Meaning | Allow access? |
|--------|---------|---------------|
| `active` | Paying, current | Yes |
| `trialing` | In trial period | Yes |
| `past_due` | Payment failed, retrying | Grace period |
| `canceled` | Will cancel at period end | Yes (until period end) |
| `unpaid` | All retries failed | No |
| `incomplete` | Initial payment pending | No |

## Price/Product Model

```
Product ("Pro Plan")
├── Price ($19/month — price_xxx)
└── Price ($190/year — price_yyy)
Product ("Enterprise Plan")
├── Price ($49/month — price_zzz)
└── Price ($490/year — price_www)
```

- Products = plan tiers
- Prices = billing intervals for each product
- Store `price_id` in subscription record
- Use `product.metadata` for feature flags

## Test Mode

### Test cards

| Card | Result |
|------|--------|
| `4242424242424242` | Success |
| `4000000000000002` | Decline |
| `4000000000009995` | Insufficient funds |
| `4000000000003220` | Requires 3D Secure |

### Stripe CLI

```bash
stripe login
stripe listen --forward-to localhost:3000/api/webhooks/stripe
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed
```

## Error Handling

```typescript
try {
  const session = await stripe.checkout.sessions.create({ ... });
} catch (err) {
  if (err instanceof Stripe.errors.StripeCardError) {
    // Card was declined
  } else if (err instanceof Stripe.errors.StripeRateLimitError) {
    // Too many requests
  } else if (err instanceof Stripe.errors.StripeInvalidRequestError) {
    // Invalid parameters
  } else if (err instanceof Stripe.errors.StripeAuthenticationError) {
    // Invalid API key
  }
}
```
