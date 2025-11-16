# Polar - Open Source Monetization Platform

**Category:** Payments & Monetization
**Official Website:** https://polar.sh
**Documentation:** https://docs.polar.sh

---

## Overview

Polar is an open-source monetization platform built for developers. It provides a complete solution for selling products, subscriptions, and accepting donations with built-in tax handling, fraud protection, and developer-friendly APIs.

**Key Features:**
- Products and digital goods sales
- Subscription management
- Pay-what-you-want pricing
- Built-in tax compliance (Merchant of Record)
- Fraud protection
- Embeddable checkout widgets
- Webhooks for automation
- Customer portal
- Analytics and reporting
- Issue funding for open source
- GitHub integration
- Newsletter subscriptions
- No-code payment links

---

## When to Use Polar

**Perfect for:**
- Open source developers monetizing projects
- Digital products and downloads
- SaaS with subscriptions
- Developer tools and services
- GitHub-integrated monetization
- Projects wanting Merchant of Record (MoR) service
- Simple payment collection without complex setup

**Advantages over Stripe:**
- Handles all tax compliance globally (VAT, GST, sales tax)
- No need to manage tax registration
- Simpler setup and pricing
- Built-in customer portal
- Open source and transparent

**Not ideal for:**
- Complex marketplace scenarios
- Highly customized payment flows
- Very large enterprise needs (Stripe has more features)

---

## Installation and Setup

**📚 Official Documentation**: [Polar - Get Started](https://docs.polar.sh)

Follow the official documentation for the latest installation instructions and API setup guides.

### Quick Reference

The general setup process involves:
1. Create a Polar account at [polar.sh](https://polar.sh)
2. Connect your payment provider (Stripe Connect is used under the hood)
3. Create products and pricing
4. Get your API key from Settings
5. Install the Polar SDK or use the REST API
6. Set up webhook endpoints for events

**Note**: API methods and SDK installation may change. Always refer to the official docs above for the current setup method.

---

## Environment Variables

```bash
# .env.local

# Polar API
POLAR_API_KEY=polar_...                    # Secret API key (server-side only)
POLAR_ORGANIZATION_ID=org_...              # Your organization ID

# Polar Webhooks
POLAR_WEBHOOK_SECRET=whsec_...             # Webhook signing secret

# Optional: Public key for client-side
NEXT_PUBLIC_POLAR_ORGANIZATION_ID=org_...  # Organization ID for embeds
```

**Important:**
- Keep `POLAR_API_KEY` server-side only
- Use webhook secret to verify webhook authenticity
- Organization ID is safe to expose client-side

---

## Core Concepts

### 1. Products

Products represent what you're selling. Can be physical, digital, or services.

**Product Types:**
- **Individual** - One-time purchase
- **Recurring** - Subscription with billing intervals
- **Free** - Free tier or downloads
- **Pay-what-you-want** - Customer chooses price (with optional minimum)

**Example: Create a product via API**
```typescript
import { Polar } from '@polar-sh/sdk';

const polar = new Polar({
  accessToken: process.env.POLAR_API_KEY!
});

const product = await polar.products.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  name: 'Pro Plan',
  description: 'Full access to all features',
  prices: [
    {
      type: 'recurring',
      recurringInterval: 'month',
      priceAmount: 2900, // $29.00 in cents
      priceCurrency: 'usd'
    }
  ],
  medias: [
    {
      url: 'https://example.com/product-image.jpg'
    }
  ]
});
```

---

### 2. Checkouts

Create checkout sessions for customers to purchase products.

**Example: Create checkout session**
```typescript
const checkout = await polar.checkouts.create({
  productPriceId: 'price_...', // Price ID from product
  successUrl: `${YOUR_DOMAIN}/success`,
  customerId: 'customer_...', // Optional: existing customer
  metadata: {
    userId: 'user_123',
    orderId: 'order_456'
  }
});

// Redirect user to checkout.url
```

**Embedded Checkout:**
```html
<!-- Embed checkout directly on your page -->
<script src="https://polar.sh/embed.js"></script>
<div id="polar-checkout"></div>
<script>
  PolarCheckout.mount({
    checkoutId: 'checkout_...',
    theme: 'light' // or 'dark'
  });
</script>
```

---

### 3. Customers

Customers represent your buyers and subscribers.

**Create or retrieve customer:**
```typescript
// Create customer
const customer = await polar.customers.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  email: 'customer@example.com',
  name: 'John Doe',
  metadata: {
    userId: 'user_123'
  }
});

// Get customer by email
const customer = await polar.customers.getByEmail({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  email: 'customer@example.com'
});
```

---

### 4. Subscriptions

Manage recurring billing for your products.

**Key features:**
- Automatic renewal
- Trial periods
- Usage-based billing
- Proration when changing plans
- Cancellation handling

**Example: Get customer's subscriptions**
```typescript
const subscriptions = await polar.subscriptions.list({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  customerId: 'customer_...'
});

// Check if active
const hasActiveSubscription = subscriptions.items.some(
  sub => sub.status === 'active'
);
```

**Cancel subscription:**
```typescript
await polar.subscriptions.cancel({
  id: 'subscription_...'
});
```

---

### 5. Webhooks

Real-time notifications about events in your Polar account.

**Key events:**
- `checkout.created` - New checkout started
- `checkout.updated` - Checkout status changed
- `order.created` - Order completed
- `subscription.created` - New subscription
- `subscription.updated` - Subscription changed
- `subscription.canceled` - Subscription canceled
- `subscription.revoked` - Subscription revoked (payment failed)

**Example webhook handler:**
```typescript
import { headers } from 'next/headers';
import { validateWebhookPayload } from '@polar-sh/sdk';

export async function POST(req: Request) {
  const body = await req.text();
  const webhookSecret = process.env.POLAR_WEBHOOK_SECRET!;

  const signature = headers().get('webhook-signature');
  const timestamp = headers().get('webhook-timestamp');

  // Verify webhook signature
  const isValid = validateWebhookPayload({
    payload: body,
    signature: signature!,
    secret: webhookSecret,
    timestamp: timestamp!
  });

  if (!isValid) {
    return new Response('Invalid signature', { status: 401 });
  }

  const event = JSON.parse(body);

  // Handle the event
  switch (event.type) {
    case 'order.created':
      await handleOrderCreated(event.data);
      break;

    case 'subscription.created':
      await handleSubscriptionCreated(event.data);
      break;

    case 'subscription.canceled':
      await handleSubscriptionCanceled(event.data);
      break;

    default:
      console.log(`Unhandled event type: ${event.type}`);
  }

  return Response.json({ received: true });
}

async function handleOrderCreated(order: any) {
  // Grant access to product
  await db.user.update({
    where: { id: order.metadata.userId },
    data: {
      hasPurchased: true,
      purchaseId: order.id
    }
  });
}

async function handleSubscriptionCreated(subscription: any) {
  // Activate subscription
  await db.user.update({
    where: { id: subscription.metadata.userId },
    data: {
      subscriptionStatus: 'active',
      subscriptionId: subscription.id
    }
  });
}

async function handleSubscriptionCanceled(subscription: any) {
  // Revoke access
  await db.user.update({
    where: { id: subscription.metadata.userId },
    data: {
      subscriptionStatus: 'canceled',
      subscriptionEndDate: new Date()
    }
  });
}
```

---

## Common Use Cases

### Use Case 1: One-Time Product Purchase

**Scenario:** Sell a digital product or license

```typescript
// app/api/purchase/route.ts
import { Polar } from '@polar-sh/sdk';

const polar = new Polar({
  accessToken: process.env.POLAR_API_KEY!
});

export async function POST(req: Request) {
  const { productPriceId, userId } = await req.json();

  // Create checkout session
  const checkout = await polar.checkouts.create({
    productPriceId,
    successUrl: `${process.env.NEXT_PUBLIC_URL}/success`,
    metadata: {
      userId
    }
  });

  return Response.json({ checkoutUrl: checkout.url });
}
```

```typescript
// Client component
'use client';

export function BuyButton({ priceId }: { priceId: string }) {
  const handlePurchase = async () => {
    const response = await fetch('/api/purchase', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        productPriceId: priceId,
        userId: 'user_123'
      })
    });

    const { checkoutUrl } = await response.json();
    window.location.href = checkoutUrl;
  };

  return <button onClick={handlePurchase}>Buy Now</button>;
}
```

---

### Use Case 2: Subscription with Free Trial

```typescript
const product = await polar.products.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  name: 'Premium Plan',
  prices: [
    {
      type: 'recurring',
      recurringInterval: 'month',
      priceAmount: 1900, // $19/month
      priceCurrency: 'usd'
    }
  ],
  metadata: {
    trial_days: '14'
  }
});
```

---

### Use Case 3: Pay-What-You-Want Pricing

```typescript
const product = await polar.products.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  name: 'Support the Project',
  prices: [
    {
      type: 'one_time',
      priceAmount: 500, // Suggested: $5
      priceCurrency: 'usd',
      minimumAmount: 100, // Minimum: $1
      allowCustomAmount: true
    }
  ]
});
```

---

### Use Case 4: Customer Portal

Let customers manage their subscriptions and billing.

```typescript
// Generate customer portal session
const session = await polar.customerSessions.create({
  customerId: 'customer_...'
});

// Redirect to session.url
```

---

## Embedded Components

### Checkout Button

Embed a pre-built buy button on your site:

```html
<script src="https://polar.sh/embed.js"></script>
<polar-checkout
  product-price-id="price_..."
  theme="light"
>
  Buy Now
</polar-checkout>
```

### Subscription Widget

Show subscription status and management:

```html
<polar-subscription
  customer-id="customer_..."
  theme="dark"
></polar-subscription>
```

---

## GitHub Integration

Polar has special features for GitHub projects:

### Issue Funding

Allow users to fund specific GitHub issues:

```typescript
// Create funding goal for an issue
const funding = await polar.issues.fund({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  issueId: 123, // GitHub issue number
  fundingGoal: 50000, // $500.00
  split: [
    {
      githubUsername: 'contributor1',
      share: 70 // 70%
    },
    {
      githubUsername: 'contributor2',
      share: 30 // 30%
    }
  ]
});
```

### Repository Benefits

Offer perks to supporters of your repository:

```typescript
const benefit = await polar.benefits.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  type: 'github_repository',
  description: 'Access to private repository',
  repositoryId: 'repo_...'
});
```

---

## Testing

Polar provides a test mode for development:

1. **Use Test Mode** in your dashboard
2. **Test Cards** work the same as Stripe (since Polar uses Stripe Connect):
   - 4242 4242 4242 4242 - Success
   - 4000 0025 0000 3155 - Requires authentication
   - 4000 0000 0000 9995 - Declined

3. **Test Webhooks Locally:**
```bash
# Install Polar CLI (if available) or use tools like ngrok
ngrok http 3000

# Update webhook URL in Polar dashboard to ngrok URL
https://your-id.ngrok.io/api/webhooks/polar
```

---

## Security Best Practices

1. **Verify webhook signatures**
   - Always validate webhook payloads
   - Use the `validateWebhookPayload` function

2. **Keep API keys secure**
   - Never expose API keys client-side
   - Use environment variables
   - Rotate keys if compromised

3. **Validate checkout sessions**
   - Verify checkout data in your webhook handler
   - Don't trust client-side data alone

4. **Use metadata for tracking**
   - Store user IDs and order IDs in metadata
   - Makes webhook handling easier

---

## Common Patterns

### Pattern 1: Store Customer ID

```typescript
// After customer creation or first purchase
const polarCustomer = await polar.customers.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  email: user.email,
  metadata: { userId: user.id }
});

await db.user.update({
  where: { id: user.id },
  data: { polarCustomerId: polarCustomer.id }
});
```

### Pattern 2: Sync Subscription Status

```typescript
// In webhook handler for subscription events
async function syncSubscriptionStatus(subscription: any) {
  const userId = subscription.metadata.userId;

  await db.user.update({
    where: { id: userId },
    data: {
      subscriptionStatus: subscription.status,
      subscriptionId: subscription.id,
      currentPeriodEnd: new Date(subscription.currentPeriodEnd)
    }
  });
}
```

### Pattern 3: Check Access

```typescript
// Middleware or API route protection
async function checkSubscriptionAccess(userId: string) {
  const user = await db.user.findUnique({
    where: { id: userId },
    select: { subscriptionStatus: true, currentPeriodEnd: true }
  });

  if (!user) return false;

  // Check if subscription is active and not expired
  return (
    user.subscriptionStatus === 'active' &&
    user.currentPeriodEnd &&
    user.currentPeriodEnd > new Date()
  );
}
```

---

## Pricing and Costs

**Polar Fee Structure:**
- 5% + payment processing fees (Stripe's fees)
- Tax compliance handled for you (Merchant of Record)
- No monthly fees or hidden costs

**Comparison:**
- **Stripe alone**: 2.9% + $0.30 (but you handle tax)
- **Polar**: 5% + Stripe fees (tax handled for you)
- **Net difference**: ~2% for complete tax compliance globally

**Worth it if:**
- You sell globally and want to avoid tax complexity
- You prefer simple, transparent pricing
- You want Merchant of Record benefits

---

## Resources

- **Documentation:** https://docs.polar.sh
- **Dashboard:** https://polar.sh/dashboard
- **API Reference:** https://api.polar.sh/docs
- **SDK (TypeScript):** https://github.com/polarsource/polar-sdk-js
- **GitHub:** https://github.com/polarsource/polar
- **Discord Community:** https://discord.gg/polar
- **Status:** https://status.polar.sh
- **Blog:** https://blog.polar.sh

---

## Polar vs Competitors

### Polar vs Stripe
- **Polar**: Simpler, handles tax, built for developers
- **Stripe**: More features, lower fees, more control, you handle tax

### Polar vs Lemon Squeezy
- **Polar**: Open source, GitHub integration, developer-focused
- **Lemon Squeezy**: More polished UI, better for non-technical users

### Polar vs Gumroad
- **Polar**: Better for SaaS/subscriptions, API-first
- **Gumroad**: Better for creators and digital downloads

---

## Migration Guide

### From Stripe to Polar

**Why migrate:**
- Simplify tax compliance (MoR)
- Focus on developer experience
- Support open source project

**How to migrate:**
1. Create products in Polar matching your Stripe products
2. Set up webhook handlers
3. Gradually migrate customers
4. Keep Stripe for existing subscriptions until they renew
5. Update payment buttons to use Polar

---

## Next Steps

1. Create a Polar account
2. Set up your organization
3. Create your first product
4. Get your API key
5. Install the SDK
6. Set up webhook endpoint
7. Test with test mode
8. Launch with real payments

---

*This guide covers Polar integration patterns. Polar is actively developed - check the official docs for the latest features and updates.*
