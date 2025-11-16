# Stripe - Payment Processing Platform

**Category:** Payments
**Official Website:** https://stripe.com
**Documentation:** https://docs.stripe.com

---

## Overview

Stripe is a complete payment platform for internet businesses. It provides the technical, fraud prevention, and banking infrastructure required to operate online payment systems.

**Key Features:**
- Accept payments (cards, wallets, bank transfers, buy now pay later)
- Subscription billing and recurring payments
- Invoicing and billing automation
- Payment Links for no-code payments
- Stripe Checkout (hosted payment page)
- Stripe Elements (embeddable payment UI)
- Strong Customer Authentication (SCA/3D Secure)
- Fraud prevention with Radar
- Multi-currency support (135+ currencies)
- Payouts and Connect (marketplace payments)
- Webhooks for event-driven workflows
- Comprehensive dashboard and analytics

---

## When to Use Stripe

**Perfect for:**
- SaaS applications with subscriptions
- E-commerce platforms
- Marketplaces and platforms (with Stripe Connect)
- One-time payments and recurring billing
- Usage-based billing
- Accepting international payments
- Projects requiring robust fraud protection

**Not ideal for:**
- Simple payment collection without integration needs (use Polar or Lemon Squeezy)
- High-risk businesses (Stripe has strict policies)
- Cryptocurrency-only payments

---

## Installation and Setup

**📚 Official Documentation**: [Stripe - Get Started](https://docs.stripe.com/development/get-started)

Follow the official documentation for the latest installation instructions and setup guides.

### Quick Reference

The general setup process involves:
1. Create a Stripe account at [stripe.com](https://stripe.com)
2. Get your API keys from the Dashboard
3. Install the Stripe SDK for your language
4. Configure webhook endpoints for event handling
5. Test in development with test mode keys

**Note**: SDK installation commands and API methods may change. Always refer to the official docs above for the current setup method.

---

## Environment Variables

```bash
# .env.local

# Stripe API Keys
STRIPE_SECRET_KEY=sk_test_...                    # Secret key (server-side only)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...  # Publishable key (client-side safe)

# Stripe Webhooks
STRIPE_WEBHOOK_SECRET=whsec_...                  # Webhook signing secret

# Optional: Stripe Connect (for marketplaces)
STRIPE_CONNECT_CLIENT_ID=ca_...                  # Connect application ID
```

**Important:**
- Use test keys (sk_test_*, pk_test_*) in development
- Use live keys (sk_live_*, pk_live_*) in production
- Never expose secret keys in client-side code
- Rotate keys if compromised

---

## Core Concepts

### 1. Payment Intents

Payment Intents track the customer's payment lifecycle from creation through checkout to completion.

**When to use:**
- Custom payment flows
- One-time payments
- Complex payment scenarios requiring manual confirmation

**Example workflow:**
```typescript
// Server-side: Create Payment Intent
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000, // $20.00 in cents
  currency: 'usd',
  payment_method_types: ['card'],
  metadata: {
    orderId: 'order_123',
    userId: 'user_456'
  }
});

// Return client secret to frontend
return { clientSecret: paymentIntent.client_secret };
```

```typescript
// Client-side: Confirm payment
import { loadStripe } from '@stripe/stripe-js';

const stripe = await loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

const { error } = await stripe.confirmCardPayment(clientSecret, {
  payment_method: {
    card: elements.getElement(CardElement),
    billing_details: {
      name: 'Customer Name',
      email: 'customer@example.com'
    }
  }
});
```

---

### 2. Checkout Sessions

Stripe-hosted checkout page for quick payment collection.

**When to use:**
- Quick implementation without building payment UI
- One-time purchases
- Subscriptions with minimal customization
- Accepting donations

**Example:**
```typescript
// Server-side: Create Checkout Session
const session = await stripe.checkout.sessions.create({
  mode: 'payment', // or 'subscription'
  line_items: [
    {
      price_data: {
        currency: 'usd',
        product_data: {
          name: 'T-Shirt',
          images: ['https://example.com/image.jpg']
        },
        unit_amount: 2000 // $20.00
      },
      quantity: 1
    }
  ],
  success_url: `${YOUR_DOMAIN}/success?session_id={CHECKOUT_SESSION_ID}`,
  cancel_url: `${YOUR_DOMAIN}/canceled`,
  metadata: {
    orderId: 'order_123'
  }
});

// Redirect user to session.url
```

---

### 3. Subscriptions

Recurring billing for SaaS and subscription-based products.

**Key concepts:**
- **Products**: What you're selling (e.g., "Pro Plan")
- **Prices**: How much and how often (e.g., "$29/month")
- **Subscriptions**: Customer's enrollment in a product
- **Invoices**: Generated automatically for each billing cycle

**Example:**
```typescript
// Create a subscription
const subscription = await stripe.subscriptions.create({
  customer: 'cus_...', // Customer ID
  items: [
    { price: 'price_...' } // Price ID
  ],
  payment_behavior: 'default_incomplete',
  payment_settings: {
    save_default_payment_method: 'on_subscription'
  },
  expand: ['latest_invoice.payment_intent']
});
```

**Common patterns:**
- Trials: `trial_period_days: 14`
- Metered billing: `billing_scheme: 'tiered'`
- Usage-based: Report usage with `stripe.subscriptionItems.createUsageRecord()`
- Proration: Automatic when changing plans

---

### 4. Customers

Store customer information for repeat payments.

```typescript
const customer = await stripe.customers.create({
  email: 'customer@example.com',
  name: 'John Doe',
  metadata: {
    userId: 'user_123'
  }
});
```

**Benefits:**
- Save payment methods for reuse
- Track customer payment history
- Manage subscriptions
- Apply coupons and discounts

---

### 5. Webhooks

Real-time notifications about events in your Stripe account.

**Critical events:**
- `payment_intent.succeeded` - Payment completed
- `payment_intent.payment_failed` - Payment failed
- `customer.subscription.created` - New subscription
- `customer.subscription.updated` - Subscription changed
- `customer.subscription.deleted` - Subscription canceled
- `invoice.payment_succeeded` - Invoice paid
- `invoice.payment_failed` - Invoice payment failed
- `charge.refunded` - Refund processed

**Example webhook handler:**
```typescript
import { headers } from 'next/headers';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(req: Request) {
  const body = await req.text();
  const signature = headers().get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    // Verify webhook signature
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (err) {
    console.error('Webhook signature verification failed:', err);
    return new Response('Webhook Error', { status: 400 });
  }

  // Handle the event
  switch (event.type) {
    case 'payment_intent.succeeded':
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      // Update your database
      await handlePaymentSuccess(paymentIntent);
      break;

    case 'customer.subscription.created':
      const subscription = event.data.object as Stripe.Subscription;
      await handleSubscriptionCreated(subscription);
      break;

    case 'invoice.payment_failed':
      const invoice = event.data.object as Stripe.Invoice;
      await handleFailedPayment(invoice);
      break;

    default:
      console.log(`Unhandled event type: ${event.type}`);
  }

  return new Response(JSON.stringify({ received: true }), { status: 200 });
}
```

---

## Common Use Cases

### Use Case 1: One-Time Payment with Checkout

**Scenario:** Simple product purchase

```typescript
// app/api/checkout/route.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(req: Request) {
  const { productId, quantity } = await req.json();

  const session = await stripe.checkout.sessions.create({
    mode: 'payment',
    line_items: [
      {
        price: productId, // Price ID from Stripe Dashboard
        quantity: quantity
      }
    ],
    success_url: `${process.env.NEXT_PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_URL}/canceled`,
  });

  return Response.json({ sessionId: session.id });
}
```

```typescript
// Client component
'use client';

import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

export function CheckoutButton({ priceId }: { priceId: string }) {
  const handleCheckout = async () => {
    const response = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ productId: priceId, quantity: 1 })
    });

    const { sessionId } = await response.json();
    const stripe = await stripePromise;

    await stripe?.redirectToCheckout({ sessionId });
  };

  return <button onClick={handleCheckout}>Buy Now</button>;
}
```

---

### Use Case 2: Subscription with Trial

```typescript
// Create subscription with 14-day trial
const session = await stripe.checkout.sessions.create({
  mode: 'subscription',
  line_items: [
    {
      price: 'price_...', // Monthly price ID
      quantity: 1
    }
  ],
  subscription_data: {
    trial_period_days: 14,
    metadata: {
      userId: 'user_123'
    }
  },
  success_url: `${YOUR_DOMAIN}/success`,
  cancel_url: `${YOUR_DOMAIN}/pricing`
});
```

---

### Use Case 3: Usage-Based Billing

```typescript
// Report usage for metered billing
await stripe.subscriptionItems.createUsageRecord(
  subscriptionItemId,
  {
    quantity: 100, // e.g., 100 API calls
    timestamp: Math.floor(Date.now() / 1000),
    action: 'increment' // or 'set'
  }
);
```

---

### Use Case 4: Customer Portal

Let customers manage their subscriptions, payment methods, and invoices.

```typescript
// Create portal session
const portalSession = await stripe.billingPortal.sessions.create({
  customer: customerId,
  return_url: `${YOUR_DOMAIN}/account`
});

// Redirect to portalSession.url
```

---

## Stripe Elements (Custom Payment UI)

Build custom payment forms with Stripe Elements.

```typescript
// Install
// npm install @stripe/stripe-js @stripe/react-stripe-js

// app/checkout/page.tsx
'use client';

import { Elements } from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';
import CheckoutForm from './CheckoutForm';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

export default function CheckoutPage() {
  const options = {
    mode: 'payment',
    amount: 2000,
    currency: 'usd',
    appearance: {
      theme: 'stripe' // or 'night', 'flat'
    }
  };

  return (
    <Elements stripe={stripePromise} options={options}>
      <CheckoutForm />
    </Elements>
  );
}
```

```typescript
// CheckoutForm.tsx
'use client';

import { PaymentElement, useStripe, useElements } from '@stripe/react-stripe-js';
import { FormEvent, useState } from 'react';

export default function CheckoutForm() {
  const stripe = useStripe();
  const elements = useElements();
  const [message, setMessage] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) return;

    setIsLoading(true);

    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/success`
      }
    });

    if (error) {
      setMessage(error.message || 'An error occurred');
    }

    setIsLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      <button disabled={isLoading || !stripe || !elements}>
        {isLoading ? 'Processing...' : 'Pay now'}
      </button>
      {message && <div>{message}</div>}
    </form>
  );
}
```

---

## Testing

### Test Cards

Stripe provides test cards for different scenarios:

| Card Number | Scenario |
|-------------|----------|
| 4242 4242 4242 4242 | Success |
| 4000 0025 0000 3155 | Requires authentication (3D Secure) |
| 4000 0000 0000 9995 | Declined (insufficient funds) |
| 4000 0000 0000 0341 | Declined (lost card) |

**Test data:**
- Use any future expiry date
- Use any 3-digit CVC
- Use any ZIP code

### Testing Webhooks Locally

```bash
# Install Stripe CLI
brew install stripe/stripe-brew/stripe

# Login
stripe login

# Forward webhooks to localhost
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Test webhook
stripe trigger payment_intent.succeeded
```

---

## Security Best Practices

1. **Never expose secret keys**
   - Keep `STRIPE_SECRET_KEY` server-side only
   - Only use `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` client-side

2. **Verify webhook signatures**
   - Always use `stripe.webhooks.constructEvent()` to verify webhooks
   - Reject requests with invalid signatures

3. **Use HTTPS in production**
   - Stripe requires HTTPS for webhooks
   - Never use HTTP in production

4. **Implement idempotency**
   - Use idempotency keys for critical operations
   - Prevents duplicate charges

```typescript
await stripe.paymentIntents.create(
  {
    amount: 2000,
    currency: 'usd'
  },
  {
    idempotencyKey: 'unique-key-123'
  }
);
```

5. **Store minimal card data**
   - Never store raw card numbers
   - Use Stripe's tokenization
   - Use Customer and PaymentMethod objects

6. **Enable Stripe Radar**
   - Automatic fraud detection
   - Configure rules in Dashboard

---

## Common Patterns

### Pattern 1: Store Customer ID in Database

```typescript
// When creating a customer
const customer = await stripe.customers.create({
  email: user.email,
  metadata: { userId: user.id }
});

// Store in your database
await db.user.update({
  where: { id: user.id },
  data: { stripeCustomerId: customer.id }
});
```

### Pattern 2: Sync Subscription Status

```typescript
// In webhook handler
async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  const userId = subscription.metadata.userId;

  await db.user.update({
    where: { id: userId },
    data: {
      subscriptionStatus: subscription.status,
      subscriptionId: subscription.id,
      currentPeriodEnd: new Date(subscription.current_period_end * 1000)
    }
  });
}
```

### Pattern 3: Handle Failed Payments

```typescript
async function handleFailedPayment(invoice: Stripe.Invoice) {
  const customerId = invoice.customer as string;

  // Get customer email
  const customer = await stripe.customers.retrieve(customerId);

  // Send notification email
  await sendEmail({
    to: customer.email,
    subject: 'Payment Failed',
    body: 'Your recent payment failed. Please update your payment method.'
  });

  // Update subscription status in database
  await db.subscription.update({
    where: { customerId },
    data: { status: 'past_due' }
  });
}
```

---

## Pricing and Costs

**Transaction Fees:**
- 2.9% + $0.30 per successful card charge (US)
- 0.8% additional for international cards
- 1% additional for currency conversion

**No monthly fees, no setup fees, no hidden costs**

**Volume discounts available** for high-volume businesses

---

## Migration from Other Payment Processors

**From PayPal:**
- Stripe has better developer experience
- More flexible pricing and subscription options
- Better international support

**From Lemon Squeezy/Paddle:**
- More control over payment flow
- Lower fees for high volume
- More customization options
- Requires handling your own tax compliance (or use Stripe Tax)

---

## Resources

- **Documentation:** https://docs.stripe.com
- **Dashboard:** https://dashboard.stripe.com
- **API Reference:** https://docs.stripe.com/api
- **Stripe Elements:** https://docs.stripe.com/payments/elements
- **Webhooks:** https://docs.stripe.com/webhooks
- **Testing:** https://docs.stripe.com/testing
- **Stripe CLI:** https://docs.stripe.com/stripe-cli
- **Status Page:** https://status.stripe.com
- **Support:** https://support.stripe.com

---

## Next Steps

1. Create a Stripe account
2. Get your API keys
3. Install the Stripe SDK
4. Set up webhook endpoint
5. Test with Stripe Checkout or Elements
6. Configure products and prices in Dashboard
7. Test webhooks locally with Stripe CLI
8. Go live with production keys

---

*This guide covers Stripe payment integration patterns. Stripe's API is continuously updated - check the official docs for the latest features and best practices.*
