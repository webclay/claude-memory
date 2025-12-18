# Unkey - API Key Management & Rate Limiting

**Purpose:** Secure API key management, rate limiting, and usage analytics for public APIs

**Last Updated:** 2025-12-18

**Official Docs:** https://www.unkey.com/docs

---

## What is Unkey?

Unkey is an open-source API management platform that provides:

- **API Key Management** - Create, verify, and revoke API keys with one-way hashing
- **Rate Limiting** - Edge-enforced limits per IP, user, API key, or custom identifier
- **Analytics** - Real-time API usage tracking and request logging
- **Access Control** - Role and permission-based access with instant global propagation

**Key Benefits:**
- Keys stored as one-way hashes (never stored in plain text)
- Sub-millisecond verification at the edge
- Works with any cloud provider or framework
- Dashboard and API access for management

---

## When to Use Unkey

### Use Unkey When:
- Building **public APIs** that external developers consume
- Need to **issue API keys** to customers or partners
- Want **rate limiting** without managing infrastructure
- Need **usage tracking** for billing or analytics
- Building **multi-tenant SaaS** with per-customer limits

### Don't Use Unkey When:
- Internal-only APIs (use authentication middleware instead)
- User authentication (use Better Auth for login/signup)
- Simple request throttling (use framework middleware)

### Unkey vs Alternatives

| Feature | Unkey | DIY Keys | AWS API Gateway |
|---------|-------|----------|-----------------|
| Key Management | Full | Manual | Full |
| Rate Limiting | Edge | Custom | Yes |
| Analytics | Built-in | Manual | CloudWatch |
| Setup Time | Minutes | Days | Hours |
| Cost | Free tier | Dev time | Per-request |
| Open Source | Yes | N/A | No |

### Unkey + Better Auth

These tools complement each other:

| Concern | Tool |
|---------|------|
| User login/signup | Better Auth |
| API key for external consumers | Unkey |
| Session management | Better Auth |
| Rate limiting API calls | Unkey |

---

## Installation

```bash
npm install @unkey/api
```

**With rate limiting only:**
```bash
npm install @unkey/ratelimit
```

---

## Environment Variables

```bash
# .env.local

# From Unkey Dashboard (https://app.unkey.com)
UNKEY_ROOT_KEY=unkey_...      # Root key for management operations
UNKEY_API_ID=api_...          # Your API identifier
```

**Important:**
- Root keys have full access - keep them server-side only
- Create separate root keys for different environments
- Rotate keys if compromised

---

## Core Concepts

### 1. API Key Creation

Create keys for your customers to access your API:

```typescript
import { Unkey } from "@unkey/api";

const unkey = new Unkey({ rootKey: process.env.UNKEY_ROOT_KEY });

// Create a new API key
const { result, error } = await unkey.keys.create({
  apiId: process.env.UNKEY_API_ID!,
  prefix: "myapp",           // Key prefix (e.g., myapp_abc123...)
  name: "Customer ABC",      // Human-readable name
  ownerId: "user_123",       // Link to your user ID
  meta: {                    // Custom metadata
    plan: "pro",
    customerId: "cus_stripe_123"
  },
  expires: Date.now() + 30 * 24 * 60 * 60 * 1000, // 30 days (optional)
  ratelimit: {               // Per-key rate limit (optional)
    type: "fast",
    limit: 100,
    refillRate: 10,
    refillInterval: 1000     // ms
  },
  remaining: 1000            // Usage limit (optional)
});

if (error) {
  console.error("Failed to create key:", error.message);
  return;
}

// Return key to customer (only time you'll see the full key!)
console.log("API Key:", result.key);
console.log("Key ID:", result.keyId);
```

---

### 2. Key Verification

Verify API keys on incoming requests:

```typescript
import { verifyKey } from "@unkey/api";

export async function verifyApiKey(key: string) {
  const { result, error } = await verifyKey(key);

  if (error) {
    // Network or service error
    console.error("Verification error:", error.message);
    return { valid: false, error: "Service unavailable" };
  }

  if (!result.valid) {
    // Key invalid, expired, or rate limited
    return {
      valid: false,
      code: result.code // "NOT_FOUND" | "FORBIDDEN" | "RATE_LIMITED" | etc.
    };
  }

  // Key is valid - access metadata
  return {
    valid: true,
    ownerId: result.ownerId,
    meta: result.meta,
    remaining: result.remaining,
    ratelimit: result.ratelimit
  };
}
```

**Verification Response Codes:**
- `VALID` - Key is valid
- `NOT_FOUND` - Key doesn't exist
- `FORBIDDEN` - Key revoked or disabled
- `RATE_LIMITED` - Rate limit exceeded
- `USAGE_EXCEEDED` - Usage limit reached
- `EXPIRED` - Key has expired

---

### 3. Rate Limiting

**Option A: Per-Key Rate Limiting (set during key creation)**

```typescript
// Rate limit is enforced automatically during verification
const { result } = await unkey.keys.create({
  apiId: process.env.UNKEY_API_ID!,
  ratelimit: {
    type: "fast",        // "fast" (sliding window) or "consistent"
    limit: 100,          // Max requests
    refillRate: 10,      // Tokens added per interval
    refillInterval: 1000 // Interval in ms
  }
});
```

**Option B: Standalone Rate Limiting (any identifier)**

```typescript
import { Ratelimit } from "@unkey/ratelimit";

const ratelimit = new Ratelimit({
  rootKey: process.env.UNKEY_ROOT_KEY!,
  namespace: "api",      // Group related limits
  limit: 10,             // Requests per window
  duration: "60s"        // Window duration
});

// Rate limit by any identifier
const { success, remaining, reset } = await ratelimit.limit("user_123");

if (!success) {
  return new Response("Rate limit exceeded", {
    status: 429,
    headers: {
      "X-RateLimit-Remaining": remaining.toString(),
      "X-RateLimit-Reset": reset.toString()
    }
  });
}
```

**Async Mode (for lower latency):**

```typescript
const ratelimit = new Ratelimit({
  rootKey: process.env.UNKEY_ROOT_KEY!,
  namespace: "api",
  limit: 10,
  duration: "60s",
  async: true  // ~97% accuracy, 30ms faster
});
```

---

### 4. Key Management

**Update a key:**

```typescript
await unkey.keys.update({
  keyId: "key_123",
  name: "Updated Name",
  meta: { plan: "enterprise" },
  ratelimit: {
    type: "fast",
    limit: 500,
    refillRate: 50,
    refillInterval: 1000
  }
});
```

**Revoke a key:**

```typescript
await unkey.keys.delete({ keyId: "key_123" });
```

**List keys for an API:**

```typescript
const { result } = await unkey.keys.list({
  apiId: process.env.UNKEY_API_ID!,
  ownerId: "user_123"  // Filter by owner (optional)
});

for (const key of result.keys) {
  console.log(key.id, key.name, key.meta);
}
```

---

## Next.js Integration

### API Route Middleware

```typescript
// lib/unkey.ts
import { verifyKey } from "@unkey/api";

export async function withApiKey(
  request: Request,
  handler: (ownerId: string, meta: Record<string, unknown>) => Promise<Response>
) {
  const authHeader = request.headers.get("authorization");

  if (!authHeader?.startsWith("Bearer ")) {
    return Response.json(
      { error: "Missing API key" },
      { status: 401 }
    );
  }

  const key = authHeader.slice(7);
  const { result, error } = await verifyKey(key);

  if (error) {
    return Response.json(
      { error: "Service unavailable" },
      { status: 503 }
    );
  }

  if (!result.valid) {
    const messages: Record<string, string> = {
      NOT_FOUND: "Invalid API key",
      FORBIDDEN: "API key revoked",
      RATE_LIMITED: "Rate limit exceeded",
      USAGE_EXCEEDED: "Usage limit exceeded",
      EXPIRED: "API key expired"
    };

    return Response.json(
      { error: messages[result.code] || "Unauthorized" },
      { status: result.code === "RATE_LIMITED" ? 429 : 401 }
    );
  }

  return handler(result.ownerId!, result.meta as Record<string, unknown>);
}
```

**Usage in API route:**

```typescript
// app/api/data/route.ts
import { withApiKey } from "@/lib/unkey";

export async function GET(request: Request) {
  return withApiKey(request, async (ownerId, meta) => {
    // Access granted - ownerId and meta available
    const data = await fetchDataForUser(ownerId);

    return Response.json({ data });
  });
}
```

### Key Provisioning Endpoint

```typescript
// app/api/keys/route.ts
import { Unkey } from "@unkey/api";
import { auth } from "@/lib/auth";

const unkey = new Unkey({ rootKey: process.env.UNKEY_ROOT_KEY! });

export async function POST(request: Request) {
  // Verify user is authenticated (via Better Auth)
  const session = await auth.api.getSession({ headers: request.headers });

  if (!session) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }

  const { name } = await request.json();

  const { result, error } = await unkey.keys.create({
    apiId: process.env.UNKEY_API_ID!,
    prefix: "myapp",
    name: name || "API Key",
    ownerId: session.user.id,
    meta: {
      createdAt: new Date().toISOString()
    }
  });

  if (error) {
    return Response.json({ error: error.message }, { status: 500 });
  }

  // Return the key (only time user will see it)
  return Response.json({
    key: result.key,
    keyId: result.keyId
  });
}
```

---

## Common Patterns

### Pattern 1: Usage-Based Billing

Track API usage for billing with Stripe:

```typescript
// During key creation
const { result } = await unkey.keys.create({
  apiId: process.env.UNKEY_API_ID!,
  ownerId: userId,
  meta: {
    stripeCustomerId: "cus_123",
    plan: "metered"
  }
});

// In webhook or cron job - report usage to Stripe
const { result: keys } = await unkey.keys.list({
  apiId: process.env.UNKEY_API_ID!,
  ownerId: userId
});

for (const key of keys.keys) {
  const usage = await getKeyUsage(key.id); // From Unkey analytics

  await stripe.subscriptionItems.createUsageRecord(
    subscriptionItemId,
    {
      quantity: usage.requests,
      action: "increment"
    }
  );
}
```

### Pattern 2: Tiered Rate Limits

Different limits based on plan:

```typescript
const planLimits = {
  free: { limit: 100, refillRate: 10 },
  pro: { limit: 1000, refillRate: 100 },
  enterprise: { limit: 10000, refillRate: 1000 }
};

async function createKeyForPlan(userId: string, plan: keyof typeof planLimits) {
  const limits = planLimits[plan];

  return await unkey.keys.create({
    apiId: process.env.UNKEY_API_ID!,
    ownerId: userId,
    meta: { plan },
    ratelimit: {
      type: "fast",
      limit: limits.limit,
      refillRate: limits.refillRate,
      refillInterval: 1000
    }
  });
}
```

### Pattern 3: Multiple Named Rate Limits

Apply different limits to different operations:

```typescript
const { result } = await unkey.keys.create({
  apiId: process.env.UNKEY_API_ID!,
  ownerId: userId,
  ratelimit: [
    {
      name: "read",
      type: "fast",
      limit: 1000,
      refillRate: 100,
      refillInterval: 1000
    },
    {
      name: "write",
      type: "fast",
      limit: 100,
      refillRate: 10,
      refillInterval: 1000
    }
  ]
});

// During verification, specify which limit to check
const { result } = await verifyKey({
  key: apiKey,
  ratelimit: { name: "write" }  // Check write limit
});
```

### Pattern 4: IP-Based Rate Limiting

Rate limit by IP without API keys:

```typescript
import { Ratelimit } from "@unkey/ratelimit";

const ipLimiter = new Ratelimit({
  rootKey: process.env.UNKEY_ROOT_KEY!,
  namespace: "ip-limit",
  limit: 60,
  duration: "1m"
});

export async function middleware(request: Request) {
  const ip = request.headers.get("x-forwarded-for") || "unknown";

  const { success, remaining } = await ipLimiter.limit(ip);

  if (!success) {
    return new Response("Too many requests", { status: 429 });
  }

  return NextResponse.next({
    headers: {
      "X-RateLimit-Remaining": remaining.toString()
    }
  });
}
```

---

## Security Best Practices

1. **Never expose root keys client-side**
   - Root keys go in server-side environment variables only
   - Use the Dashboard to create restricted keys if needed

2. **Use key prefixes**
   - Makes keys identifiable: `myapp_abc123...`
   - Helps with debugging and support

3. **Set expiration on keys**
   - Force rotation for security
   - Use `expires` parameter during creation

4. **Store key IDs, not keys**
   - After creation, store `keyId` in your database
   - The full key is only shown once

5. **Monitor usage in Dashboard**
   - Set up alerts for unusual patterns
   - Review analytics regularly

6. **Use HTTPS only**
   - API keys should never travel over HTTP
   - Verify your endpoints enforce HTTPS

---

## Testing

**Test Mode:**
Unkey provides test mode in the Dashboard for development without affecting production data.

**Local Development:**

```typescript
// Mock verification for tests
jest.mock("@unkey/api", () => ({
  verifyKey: jest.fn().mockResolvedValue({
    result: {
      valid: true,
      ownerId: "test_user",
      meta: { plan: "pro" }
    }
  })
}));
```

---

## Troubleshooting

### Key Not Found
- Verify the key hasn't been revoked
- Check the key prefix matches your API
- Ensure you're using the correct API ID

### Rate Limit Always Triggering
- Check the limit and refill settings
- Verify you're not in a retry loop
- Review the refill interval (in milliseconds)

### Slow Verification
- Enable async mode for rate limiting
- Check network latency to Unkey edge
- Consider caching verification results briefly

---

## Resources

- **Documentation:** https://www.unkey.com/docs
- **Dashboard:** https://app.unkey.com
- **GitHub:** https://github.com/unkeyed/unkey
- **Discord:** https://unkey.com/discord
- **API Reference:** https://www.unkey.com/docs/api-reference
- **Rate Limiting Guide:** https://www.unkey.com/docs/apis/features/ratelimiting/overview

---

## Related Stack Modules

- [auth-better-auth.md](../auth/auth-better-auth.md) - User authentication (complements Unkey for API auth)
- [api-orpc.md](./api-orpc.md) - Type-safe API framework (protect with Unkey)
- [payments-stripe.md](../payments/payments-stripe.md) - Usage-based billing integration

---

**Last Updated:** 2025-12-18
**Maintained By:** Claude Memory System
**Status:** Active
