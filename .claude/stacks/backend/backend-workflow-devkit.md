# Workflow DevKit Integration Guide

## Overview

Workflow DevKit is Vercel's durable workflows library that enables writing long-running, resumable async functions in TypeScript. It provides automatic retries, state persistence, and fault tolerance for backend jobs, scheduled tasks, and complex multi-step processes.

**Key Features:**
- **Durable Execution**: Workflows survive crashes and resume from checkpoints
- **Automatic Retries**: Built-in exponential backoff for failed steps
- **Type-Safe**: Full TypeScript support with type inference
- **Framework Agnostic**: Works with Next.js, Hono, Express, and more
- **State Persistence**: Workflow state stored in database or memory
- **Composable**: Break workflows into reusable steps

**When to Use:**
- Background jobs that must complete reliably
- Multi-step processes with external API calls
- Scheduled tasks requiring fault tolerance
- Order processing, payment flows, data pipelines
- Tasks that might take hours/days to complete

## Installation

**📚 Official Documentation**: [Vercel Workflow DevKit - Documentation](https://vercel.com/docs/workflow)

Follow the official documentation for the latest installation instructions and setup guides for your framework (Next.js, Hono, Express, etc.).

### Quick Reference

The general setup process involves:
1. Install the Workflow DevKit package
2. Configure storage backend (Vercel KV for production, memory for development)
3. Set up workflow functions with `"use workflow"` directive
4. Create reusable steps with `"use step"` directive
5. Trigger workflows from API routes or handlers

**Note**: Installation commands and configuration options may change. Always refer to the official docs above for the current setup method.

## Core Concepts

### 1. Workflows

Workflows are async functions marked with `"use workflow"` directive. They automatically become durable and resumable:

```typescript
import { workflow } from '@vercel/workflow';

export async function processOrder(orderId: string) {
  "use workflow";

  // Each await is a checkpoint - if crash occurs, resumes from here
  const order = await fetchOrder(orderId);
  const payment = await processPayment(order);
  await sendConfirmationEmail(order, payment);
  await updateInventory(order);

  return { success: true, orderId };
}
```

**How it works:**
- Each `await` creates a checkpoint
- If function crashes, it resumes from last checkpoint
- No duplicate work - already completed steps are skipped
- State is persisted automatically

### 2. Steps

Steps are reusable workflow units marked with `"use step"`:

```typescript
import { step } from '@vercel/workflow';

async function sendEmail(to: string, subject: string, body: string) {
  "use step";

  const result = await emailService.send({ to, subject, body });
  return result;
}

export async function welcomeNewUser(userId: string) {
  "use workflow";

  const user = await fetchUser(userId);

  // Step with automatic retries
  await sendEmail(
    user.email,
    "Welcome!",
    `Hi ${user.name}, welcome to our platform!`
  );

  await trackEvent("user_welcomed", { userId });
}
```

**Step Benefits:**
- Automatic retry with exponential backoff
- Isolated failure handling
- Reusable across workflows
- Clear workflow structure

### 3. Workflow Context

Access workflow metadata and control execution:

```typescript
import { workflow, context } from '@vercel/workflow';

export async function monitorJob(jobId: string) {
  "use workflow";

  const ctx = context();

  console.log(`Workflow ID: ${ctx.workflowId}`);
  console.log(`Run ID: ${ctx.runId}`);
  console.log(`Attempt: ${ctx.attempt}`);

  // Sleep for duration
  await ctx.sleep("1 hour");

  const status = await checkJobStatus(jobId);
  return status;
}
```

## Integration Patterns

### Next.js App Router

**app/api/workflows/process-order/route.ts**
```typescript
import { NextRequest } from 'next/server';
import { workflow } from '@vercel/workflow';

async function processOrderWorkflow(orderId: string) {
  "use workflow";

  // Fetch order
  const order = await db.orders.findUnique({
    where: { id: orderId }
  });

  if (!order) {
    throw new Error(`Order ${orderId} not found`);
  }

  // Process payment
  const payment = await fetch('https://payment-api.com/charge', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      amount: order.total,
      currency: order.currency,
      orderId: order.id
    })
  }).then(res => res.json());

  if (!payment.success) {
    throw new Error('Payment failed');
  }

  // Update order status
  await db.orders.update({
    where: { id: orderId },
    data: { status: 'paid', paymentId: payment.id }
  });

  // Send confirmation
  await fetch('https://email-api.com/send', {
    method: 'POST',
    body: JSON.stringify({
      to: order.customerEmail,
      subject: 'Order Confirmed',
      template: 'order-confirmation',
      data: { order, payment }
    })
  });

  return { orderId, paymentId: payment.id };
}

export async function POST(request: NextRequest) {
  const { orderId } = await request.json();

  // Trigger workflow
  const result = await processOrderWorkflow(orderId);

  return Response.json(result);
}
```

**Triggering from Client:**
```typescript
'use client';

export function CheckoutButton({ orderId }: { orderId: string }) {
  const handleCheckout = async () => {
    const response = await fetch('/api/workflows/process-order', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ orderId })
    });

    const result = await response.json();
    console.log('Workflow started:', result);
  };

  return <button onClick={handleCheckout}>Complete Order</button>;
}
```

### Hono Integration

```typescript
import { Hono } from 'hono';
import { workflow } from '@vercel/workflow';

const app = new Hono();

async function processWebhook(webhookId: string, data: unknown) {
  "use workflow";

  // Validate webhook
  const isValid = await validateWebhookSignature(webhookId);
  if (!isValid) {
    throw new Error('Invalid webhook signature');
  }

  // Process data
  const result = await processWebhookData(data);

  // Store result
  await db.webhookResults.create({
    data: { webhookId, result, processedAt: new Date() }
  });

  return result;
}

app.post('/webhooks/:provider', async (c) => {
  const provider = c.req.param('provider');
  const body = await c.req.json();

  // Trigger workflow
  const result = await processWebhook(provider, body);

  return c.json({ success: true, result });
});

export default app;
```

### Express Integration

```typescript
import express from 'express';
import { workflow } from '@vercel/workflow';

const app = express();
app.use(express.json());

async function syncUserData(userId: string) {
  "use workflow";

  // Fetch from primary source
  const userData = await fetch(`https://api.example.com/users/${userId}`)
    .then(res => res.json());

  // Sync to multiple services
  await Promise.all([
    syncToCRM(userData),
    syncToAnalytics(userData),
    syncToEmailService(userData)
  ]);

  return { userId, syncedAt: new Date() };
}

app.post('/api/sync/:userId', async (req, res) => {
  const { userId } = req.params;

  try {
    const result = await syncUserData(userId);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000);
```

## Workflow Patterns

### 1. Retry Logic with Exponential Backoff

```typescript
import { workflow, step, context } from '@vercel/workflow';

async function fetchWithRetry(url: string, maxAttempts = 3) {
  "use step";

  const ctx = context();

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const response = await fetch(url);
      if (response.ok) {
        return await response.json();
      }

      if (attempt < maxAttempts) {
        // Exponential backoff: 1s, 2s, 4s
        const delaySeconds = Math.pow(2, attempt - 1);
        await ctx.sleep(`${delaySeconds} seconds`);
      }
    } catch (error) {
      if (attempt === maxAttempts) throw error;
    }
  }

  throw new Error(`Failed after ${maxAttempts} attempts`);
}

export async function fetchExternalData(endpoint: string) {
  "use workflow";

  const data = await fetchWithRetry(endpoint);
  return data;
}
```

### 2. Parallel Execution

```typescript
import { workflow } from '@vercel/workflow';

async function processImage(imageId: string) {
  "use step";

  const image = await fetchImage(imageId);
  const processed = await applyFilters(image);
  return processed;
}

async function generateThumbnail(imageId: string) {
  "use step";

  const image = await fetchImage(imageId);
  const thumbnail = await resize(image, { width: 200, height: 200 });
  return thumbnail;
}

async function extractMetadata(imageId: string) {
  "use step";

  const image = await fetchImage(imageId);
  const metadata = await analyzeImage(image);
  return metadata;
}

export async function processUpload(imageId: string) {
  "use workflow";

  // Run all steps in parallel
  const [processed, thumbnail, metadata] = await Promise.all([
    processImage(imageId),
    generateThumbnail(imageId),
    extractMetadata(imageId)
  ]);

  // Save results
  await db.images.update({
    where: { id: imageId },
    data: { processed, thumbnail, metadata }
  });

  return { imageId, processed, thumbnail, metadata };
}
```

### 3. Conditional Workflows

```typescript
import { workflow, context } from '@vercel/workflow';

export async function moderateContent(contentId: string) {
  "use workflow";

  const content = await fetchContent(contentId);

  // AI moderation check
  const moderation = await checkContentModeration(content.text);

  if (moderation.flagged) {
    // Content flagged - human review
    await notifyModerators(contentId, moderation);

    // Wait for human review (could be hours/days)
    const ctx = context();
    await ctx.sleep("24 hours");

    const review = await fetchModerationReview(contentId);

    if (!review.approved) {
      await deleteContent(contentId);
      await notifyUser(content.userId, "Content removed");
      return { action: "deleted", reason: review.reason };
    }
  }

  // Publish content
  await publishContent(contentId);
  return { action: "published" };
}
```

### 4. Scheduled Workflows

```typescript
import { workflow, context } from '@vercel/workflow';

export async function dailyReportWorkflow(userId: string) {
  "use workflow";

  const ctx = context();

  while (true) {
    // Generate report
    const report = await generateDailyReport(userId);

    // Send email
    await sendEmail(userId, "Daily Report", report);

    // Wait 24 hours
    await ctx.sleep("24 hours");
  }
}

// Trigger once, runs forever
export async function startDailyReports(userId: string) {
  await dailyReportWorkflow(userId);
}
```

### 5. Saga Pattern (Compensating Transactions)

```typescript
import { workflow } from '@vercel/workflow';

export async function bookTrip(userId: string, tripData: TripData) {
  "use workflow";

  let flightBooking;
  let hotelBooking;
  let carRental;

  try {
    // Step 1: Book flight
    flightBooking = await bookFlight(tripData.flight);

    try {
      // Step 2: Book hotel
      hotelBooking = await bookHotel(tripData.hotel);

      try {
        // Step 3: Book car
        carRental = await bookCar(tripData.car);

        // All successful
        return {
          success: true,
          flightBooking,
          hotelBooking,
          carRental
        };

      } catch (error) {
        // Rollback hotel and flight
        await cancelHotel(hotelBooking.id);
        await cancelFlight(flightBooking.id);
        throw error;
      }

    } catch (error) {
      // Rollback flight
      await cancelFlight(flightBooking.id);
      throw error;
    }

  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
}
```

### 6. Fan-Out/Fan-In Pattern

```typescript
import { workflow, step } from '@vercel/workflow';

async function processChunk(chunkId: string, data: unknown) {
  "use step";

  const result = await heavyProcessing(data);
  await saveChunkResult(chunkId, result);
  return result;
}

export async function processBatchJob(jobId: string) {
  "use workflow";

  // Load data
  const job = await fetchJob(jobId);
  const chunks = splitIntoChunks(job.data, 100);

  // Fan-out: Process all chunks in parallel
  const results = await Promise.all(
    chunks.map((chunk, index) =>
      processChunk(`${jobId}-${index}`, chunk)
    )
  );

  // Fan-in: Aggregate results
  const aggregated = results.reduce((acc, result) => {
    return mergeResults(acc, result);
  }, {});

  // Save final result
  await saveJobResult(jobId, aggregated);

  return { jobId, totalChunks: chunks.length, result: aggregated };
}
```

## Configuration

### Workflow Settings

```typescript
import { workflow, context } from '@vercel/workflow';

export async function configuredWorkflow(data: string) {
  "use workflow";

  const ctx = context();

  // Configure retry behavior
  ctx.configure({
    retries: {
      maxAttempts: 5,
      initialDelay: '1 second',
      maxDelay: '1 minute',
      backoffFactor: 2
    },
    timeout: '30 minutes'
  });

  await processData(data);
}
```

### Storage Backend

**Using Vercel KV (Recommended for Production):**

```typescript
// lib/workflow-config.ts
import { WorkflowStorage } from '@vercel/workflow';
import { kv } from '@vercel/kv';

export const workflowStorage = new WorkflowStorage({
  type: 'vercel-kv',
  client: kv
});
```

**Using Memory (Development Only):**

```typescript
// lib/workflow-config.ts
import { WorkflowStorage } from '@vercel/workflow';

export const workflowStorage = new WorkflowStorage({
  type: 'memory'
});
```

**Using Custom Storage:**

```typescript
// lib/workflow-config.ts
import { WorkflowStorage } from '@vercel/workflow';

export const workflowStorage = new WorkflowStorage({
  type: 'custom',
  async get(key: string) {
    return await db.workflowState.findUnique({ where: { key } });
  },
  async set(key: string, value: unknown) {
    await db.workflowState.upsert({
      where: { key },
      create: { key, value },
      update: { value }
    });
  },
  async delete(key: string) {
    await db.workflowState.delete({ where: { key } });
  }
});
```

## Error Handling

### Try-Catch in Workflows

```typescript
import { workflow } from '@vercel/workflow';

export async function robustWorkflow(orderId: string) {
  "use workflow";

  try {
    const order = await fetchOrder(orderId);
    const payment = await processPayment(order);

    return { success: true, payment };

  } catch (error) {
    // Log error
    await logError({
      workflow: 'robustWorkflow',
      orderId,
      error: error.message,
      timestamp: new Date()
    });

    // Notify admin
    await notifyAdmin(`Workflow failed for order ${orderId}`);

    // Mark order as failed
    await db.orders.update({
      where: { id: orderId },
      data: { status: 'failed', errorMessage: error.message }
    });

    return { success: false, error: error.message };
  }
}
```

### Custom Error Types

```typescript
import { workflow } from '@vercel/workflow';

class PaymentError extends Error {
  constructor(
    message: string,
    public code: string,
    public retryable: boolean
  ) {
    super(message);
    this.name = 'PaymentError';
  }
}

class InventoryError extends Error {
  constructor(message: string, public productId: string) {
    super(message);
    this.name = 'InventoryError';
  }
}

export async function processOrderWithCustomErrors(orderId: string) {
  "use workflow";

  const order = await fetchOrder(orderId);

  try {
    const payment = await processPayment(order);

    if (!payment.success) {
      throw new PaymentError(
        payment.errorMessage,
        payment.errorCode,
        payment.errorCode === 'insufficient_funds'
      );
    }

  } catch (error) {
    if (error instanceof PaymentError && error.retryable) {
      // Wait and retry for retryable errors
      await context().sleep("1 hour");
      return processOrderWithCustomErrors(orderId);
    }

    throw error; // Non-retryable, let it fail
  }

  try {
    await updateInventory(order);
  } catch (error) {
    // Payment succeeded but inventory failed - refund
    await refundPayment(order.paymentId);

    throw new InventoryError(
      `Failed to update inventory: ${error.message}`,
      order.productId
    );
  }

  return { success: true, orderId };
}
```

## Testing

### Testing Workflows

```typescript
// __tests__/workflows.test.ts
import { describe, test, expect, vi } from 'vitest';
import { processOrder } from '../workflows/process-order';

// Mock external dependencies
vi.mock('../lib/payment-service', () => ({
  processPayment: vi.fn().mockResolvedValue({
    success: true,
    id: 'payment_123'
  })
}));

vi.mock('../lib/email-service', () => ({
  sendEmail: vi.fn().mockResolvedValue({ success: true })
}));

describe('processOrder workflow', () => {
  test('should process order successfully', async () => {
    const result = await processOrder('order_123');

    expect(result.success).toBe(true);
    expect(result.orderId).toBe('order_123');
  });

  test('should handle payment failure', async () => {
    const { processPayment } = await import('../lib/payment-service');
    vi.mocked(processPayment).mockRejectedValueOnce(
      new Error('Payment declined')
    );

    await expect(processOrder('order_123')).rejects.toThrow('Payment declined');
  });
});
```

### Testing Steps in Isolation

```typescript
// __tests__/steps.test.ts
import { describe, test, expect } from 'vitest';
import { sendEmail } from '../workflows/steps';

describe('sendEmail step', () => {
  test('should send email with correct parameters', async () => {
    const result = await sendEmail(
      'user@example.com',
      'Test Subject',
      'Test Body'
    );

    expect(result.success).toBe(true);
  });
});
```

## Monitoring & Observability

### Logging Workflow Progress

```typescript
import { workflow, context } from '@vercel/workflow';

export async function monitoredWorkflow(jobId: string) {
  "use workflow";

  const ctx = context();

  console.log(`[${ctx.workflowId}] Starting workflow for job ${jobId}`);

  const startTime = Date.now();

  try {
    const data = await fetchData(jobId);
    console.log(`[${ctx.workflowId}] Data fetched: ${data.length} items`);

    const processed = await processData(data);
    console.log(`[${ctx.workflowId}] Processing complete`);

    const duration = Date.now() - startTime;
    console.log(`[${ctx.workflowId}] Workflow completed in ${duration}ms`);

    return { success: true, duration, itemsProcessed: processed.length };

  } catch (error) {
    console.error(`[${ctx.workflowId}] Workflow failed:`, error);
    throw error;
  }
}
```

### Metrics Collection

```typescript
import { workflow, context } from '@vercel/workflow';

async function trackMetric(name: string, value: number, tags?: Record<string, string>) {
  await fetch('https://metrics-api.example.com/track', {
    method: 'POST',
    body: JSON.stringify({ name, value, tags, timestamp: Date.now() })
  });
}

export async function trackedWorkflow(orderId: string) {
  "use workflow";

  const ctx = context();
  const startTime = Date.now();

  try {
    await trackMetric('workflow.started', 1, {
      workflowId: ctx.workflowId,
      orderId
    });

    const result = await processOrder(orderId);

    const duration = Date.now() - startTime;
    await trackMetric('workflow.completed', 1, {
      workflowId: ctx.workflowId,
      orderId,
      duration: duration.toString()
    });

    return result;

  } catch (error) {
    await trackMetric('workflow.failed', 1, {
      workflowId: ctx.workflowId,
      orderId,
      error: error.message
    });

    throw error;
  }
}
```

## Best Practices

### 1. Idempotency

Ensure steps can be safely retried without side effects:

```typescript
async function createUser(email: string) {
  "use step";

  // Check if user already exists (idempotent)
  const existing = await db.users.findUnique({ where: { email } });
  if (existing) return existing;

  // Create only if doesn't exist
  const user = await db.users.create({ data: { email } });
  return user;
}
```

### 2. Keep Steps Small and Focused

```typescript
// Bad: Large monolithic step
async function processOrderBad(orderId: string) {
  "use step";

  const order = await fetchOrder(orderId);
  const payment = await processPayment(order);
  await sendEmail(order.email, payment);
  await updateInventory(order);
  await notifyWarehouse(order);
}

// Good: Break into smaller steps
async function fetchOrderStep(orderId: string) {
  "use step";
  return await fetchOrder(orderId);
}

async function processPaymentStep(order: Order) {
  "use step";
  return await processPayment(order);
}

async function sendConfirmationStep(order: Order, payment: Payment) {
  "use step";
  return await sendEmail(order.email, payment);
}

export async function processOrderGood(orderId: string) {
  "use workflow";

  const order = await fetchOrderStep(orderId);
  const payment = await processPaymentStep(order);
  await sendConfirmationStep(order, payment);
  await updateInventoryStep(order);
  await notifyWarehouseStep(order);
}
```

### 3. Handle Partial Failures

```typescript
export async function syncToMultipleServices(data: UserData) {
  "use workflow";

  const results = {
    crm: { success: false, error: null },
    analytics: { success: false, error: null },
    email: { success: false, error: null }
  };

  // Try each service independently
  try {
    await syncToCRM(data);
    results.crm.success = true;
  } catch (error) {
    results.crm.error = error.message;
  }

  try {
    await syncToAnalytics(data);
    results.analytics.success = true;
  } catch (error) {
    results.analytics.error = error.message;
  }

  try {
    await syncToEmailService(data);
    results.email.success = true;
  } catch (error) {
    results.email.error = error.message;
  }

  // Log failures but don't throw
  const failures = Object.entries(results)
    .filter(([_, result]) => !result.success);

  if (failures.length > 0) {
    await logPartialFailure(data.userId, failures);
  }

  return results;
}
```

### 4. Use Timeouts

```typescript
import { workflow, context } from '@vercel/workflow';

export async function workflowWithTimeout(taskId: string) {
  "use workflow";

  const ctx = context();

  // Set workflow timeout
  const timeoutPromise = ctx.sleep("5 minutes").then(() => {
    throw new Error("Workflow timeout exceeded");
  });

  const workPromise = (async () => {
    const result = await processTask(taskId);
    return result;
  })();

  // Race between work and timeout
  return await Promise.race([workPromise, timeoutPromise]);
}
```

### 5. Version Your Workflows

```typescript
export async function processOrderV1(orderId: string) {
  "use workflow";
  // Original implementation
}

export async function processOrderV2(orderId: string) {
  "use workflow";
  // Updated implementation with new features

  const order = await fetchOrder(orderId);

  // New: fraud detection
  const fraudCheck = await checkFraud(order);
  if (fraudCheck.flagged) {
    throw new Error("Order flagged for fraud");
  }

  // Existing logic continues...
}

// Router to pick version
export async function processOrder(orderId: string, version: string = 'v2') {
  if (version === 'v1') {
    return processOrderV1(orderId);
  }
  return processOrderV2(orderId);
}
```

## Common Use Cases

### 1. Email Campaign

```typescript
import { workflow, context } from '@vercel/workflow';

export async function sendCampaign(campaignId: string) {
  "use workflow";

  const campaign = await fetchCampaign(campaignId);
  const recipients = await fetchRecipients(campaign.segmentId);

  // Send to each recipient (with rate limiting)
  for (const recipient of recipients) {
    await sendEmail(recipient.email, campaign.subject, campaign.body);

    // Rate limit: 100 emails per second
    await context().sleep("10 milliseconds");
  }

  // Mark campaign complete
  await db.campaigns.update({
    where: { id: campaignId },
    data: { status: 'sent', sentAt: new Date() }
  });

  return { campaignId, recipientCount: recipients.length };
}
```

### 2. Data Migration

```typescript
import { workflow } from '@vercel/workflow';

export async function migrateUsers(batchSize: number = 100) {
  "use workflow";

  let offset = 0;
  let totalMigrated = 0;

  while (true) {
    // Fetch batch
    const users = await fetchUsersFromLegacyDB(offset, batchSize);

    if (users.length === 0) break;

    // Transform and insert
    for (const user of users) {
      const transformed = transformUserData(user);
      await insertIntoNewDB(transformed);
    }

    totalMigrated += users.length;
    offset += batchSize;

    console.log(`Migrated ${totalMigrated} users...`);
  }

  return { totalMigrated };
}
```

### 3. Report Generation

```typescript
import { workflow, context } from '@vercel/workflow';

export async function generateMonthlyReport(organizationId: string) {
  "use workflow";

  const ctx = context();

  // Gather data from multiple sources
  const [users, revenue, activities] = await Promise.all([
    fetchUserStats(organizationId),
    fetchRevenueStats(organizationId),
    fetchActivityStats(organizationId)
  ]);

  // Generate report
  const report = await generateReport({
    organizationId,
    users,
    revenue,
    activities,
    period: 'monthly'
  });

  // Store report
  const reportUrl = await uploadReport(report);

  // Send to admins
  const admins = await fetchAdmins(organizationId);
  for (const admin of admins) {
    await sendEmail(
      admin.email,
      "Monthly Report Ready",
      `Your monthly report is ready: ${reportUrl}`
    );
  }

  return { reportUrl, generatedAt: new Date() };
}
```

### 4. Subscription Renewal

```typescript
import { workflow, context } from '@vercel/workflow';

export async function processSubscriptionRenewal(subscriptionId: string) {
  "use workflow";

  const subscription = await fetchSubscription(subscriptionId);

  // Attempt payment
  try {
    const payment = await chargeCustomer(
      subscription.customerId,
      subscription.amount
    );

    // Extend subscription
    await db.subscriptions.update({
      where: { id: subscriptionId },
      data: {
        status: 'active',
        expiresAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000)
      }
    });

    // Send success email
    await sendEmail(
      subscription.customerEmail,
      "Subscription Renewed",
      "Your subscription has been renewed successfully."
    );

    return { success: true, payment };

  } catch (error) {
    // Payment failed
    await db.subscriptions.update({
      where: { id: subscriptionId },
      data: { status: 'payment_failed' }
    });

    // Send dunning email
    await sendEmail(
      subscription.customerEmail,
      "Payment Failed",
      "We couldn't process your payment. Please update your payment method."
    );

    // Retry in 3 days
    await context().sleep("3 days");
    return processSubscriptionRenewal(subscriptionId);
  }
}
```

## Environment Variables

```bash
# .env.local

# Workflow storage (production)
KV_REST_API_URL=https://your-kv-instance.upstash.io
KV_REST_API_TOKEN=your_token_here

# Workflow configuration
WORKFLOW_MAX_RETRIES=5
WORKFLOW_TIMEOUT_MS=1800000  # 30 minutes
```

## TypeScript Types

```typescript
// types/workflow.ts

export interface WorkflowResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
}

export interface WorkflowContext {
  workflowId: string;
  runId: string;
  attempt: number;
  sleep: (duration: string) => Promise<void>;
  configure: (options: WorkflowOptions) => void;
}

export interface WorkflowOptions {
  retries?: {
    maxAttempts: number;
    initialDelay: string;
    maxDelay: string;
    backoffFactor: number;
  };
  timeout?: string;
}

export interface StepOptions {
  retries?: number;
  timeout?: string;
}
```

## Migration Guide

### From Manual Background Jobs

**Before (Manual):**
```typescript
// pages/api/process-order.ts
export default async function handler(req, res) {
  const { orderId } = req.body;

  // Fire and forget - no retries, no recovery
  setTimeout(async () => {
    try {
      await processOrder(orderId);
    } catch (error) {
      console.error('Failed to process order:', error);
      // Lost forever if this fails
    }
  }, 0);

  res.json({ queued: true });
}
```

**After (Workflow):**
```typescript
// app/api/process-order/route.ts
import { workflow } from '@vercel/workflow';

async function processOrderWorkflow(orderId: string) {
  "use workflow";

  // Automatic retries, state persistence, recovery
  await processOrder(orderId);
}

export async function POST(request: Request) {
  const { orderId } = await request.json();

  // Durable execution
  await processOrderWorkflow(orderId);

  return Response.json({ queued: true });
}
```

## Troubleshooting

### Workflow Not Resuming After Crash

**Problem:** Workflow restarts from beginning after crash.

**Solution:** Ensure storage backend is configured:
```typescript
import { WorkflowStorage } from '@vercel/workflow';
import { kv } from '@vercel/kv';

export const storage = new WorkflowStorage({
  type: 'vercel-kv',
  client: kv
});
```

### Too Many Retries

**Problem:** Workflow retries indefinitely.

**Solution:** Set retry limits:
```typescript
export async function limitedRetries(taskId: string) {
  "use workflow";

  const ctx = context();
  ctx.configure({
    retries: { maxAttempts: 3 }
  });

  await processTask(taskId);
}
```

### Workflow Timeout

**Problem:** Workflow takes too long and times out.

**Solution:** Break into smaller steps or increase timeout:
```typescript
export async function longRunningWorkflow(dataId: string) {
  "use workflow";

  context().configure({
    timeout: '2 hours'
  });

  await processLargeDataset(dataId);
}
```

## Resources

- **Documentation**: https://vercel.com/docs/workflow
- **GitHub**: https://github.com/vercel/workflow
- **Examples**: https://github.com/vercel/workflow/tree/main/examples
- **Comparison with Temporal**: https://vercel.com/blog/workflow-vs-temporal
- **Best Practices**: https://vercel.com/docs/workflow/best-practices

## Summary

Workflow DevKit provides durable execution for TypeScript async functions:

1. **Use `"use workflow"`** for long-running, fault-tolerant processes
2. **Use `"use step"`** for reusable, retryable units of work
3. **Each `await`** creates a checkpoint for resumability
4. **Built-in retries** with exponential backoff
5. **State persistence** in Vercel KV or custom storage
6. **Framework agnostic** - works with Next.js, Hono, Express
7. **Type-safe** with full TypeScript support
8. **Production-ready** for order processing, data pipelines, scheduled tasks

Start with simple workflows and gradually add complexity as needed. The framework handles the hard parts of distributed systems (retries, state management, recovery) so you can focus on business logic.
