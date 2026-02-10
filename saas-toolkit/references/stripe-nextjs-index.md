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

## Checkout Session Pattern

### Server Action

```typescript
'use server';
import { stripe } from '@/lib/stripe';
import { redirect } from 'next/navigation';

export async function createCheckoutSession(priceId: string) {
  // 1. Verify user authentication
  // 2. Get or create Stripe Customer
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

## Webhook Handler Pattern

```typescript
// app/api/webhooks/stripe/route.ts
import { stripe } from '@/lib/stripe';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  // 1. Get raw body (MUST be text, not JSON)
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;

  // 2. Verify signature
  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  // 3. Handle events
  switch (event.type) {
    case 'checkout.session.completed':
      // Provision access, create subscription record
      break;
    case 'customer.subscription.updated':
      // Sync subscription status, plan changes
      break;
    case 'customer.subscription.deleted':
      // Revoke access, update subscription record
      break;
    case 'invoice.payment_failed':
      // Notify user, set grace period
      break;
    case 'invoice.payment_succeeded':
      // Clear failed payment flags
      break;
  }

  // 4. Return 200 quickly
  return NextResponse.json({ received: true });
}
```

**Critical rules:**
- `request.text()` NOT `request.json()`
- Always verify signature
- Return 200 even if processing fails (handle errors internally)
- Store event IDs for idempotency

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
| `customer.subscription.paused` | Subscription paused | Update status, limit access |
| `invoice.payment_succeeded` | Payment processed | Clear payment failure flags |
| `invoice.payment_failed` | Payment failed | Notify user, set grace period |
| `invoice.finalized` | Invoice ready | Store invoice reference |

## Price/Product Model

```
Organization (Stripe)
└── Product ("Pro Plan")
    ├── Price ($19/month — price_xxx)
    └── Price ($190/year — price_yyy)
└── Product ("Enterprise Plan")
    ├── Price ($49/month — price_zzz)
    └── Price ($490/year — price_www)
```

- Products = plan tiers
- Prices = billing intervals for each product
- Store `price_id` in subscription record
- Use `product.metadata` for feature flags

## Subscription Status Values

| Status | Meaning | Allow access? |
|--------|---------|---------------|
| `active` | Paying, current | Yes |
| `trialing` | In trial period | Yes |
| `past_due` | Payment failed, retrying | Grace period |
| `canceled` | Will cancel at period end | Yes (until period end) |
| `unpaid` | All retries failed | No |
| `incomplete` | Initial payment pending | No |
| `incomplete_expired` | Initial payment failed | No |
| `paused` | Paused by request | No |

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
# Login
stripe login

# Listen for webhooks locally
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed

# Use test clocks for subscription lifecycle
stripe test_clocks create --frozen-time 2024-01-01T00:00:00Z
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
  } else {
    // Unknown error
  }
}
```
