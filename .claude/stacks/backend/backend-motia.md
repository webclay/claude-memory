# Motia Backend Framework Patterns

## Overview

Motia is a multi-language, event-driven runtime manager built on a core primitive: the **Step**. It unifies APIs, background jobs, queues, workflows, AI agents, streaming, state management, and observability into a single cohesive system.

**Key Features:**
- Single primitive (Steps) for all backend patterns
- Multi-language support (TypeScript, Python, Ruby)
- Event-driven architecture with pub/sub
- Built-in state management
- Real-time streaming support
- Visual debugging with Workbench UI
- Production-ready deployment to Motia Cloud

**Official Website:** https://motia.dev

---

## Installation and Setup

**📚 Official Documentation**: [Motia - Getting Started](https://motia.dev/docs)

Follow the official documentation for the latest installation instructions, project templates, and setup guides.

### Quick Reference

The general setup process involves:
1. Create a new project using the Motia CLI
2. Choose a template (Starter, AI Agent, etc.)
3. Select your preferred language(s) (TypeScript, Python, Ruby)
4. Start the Motia Workbench for local development
5. Create Steps (API endpoints, event handlers, cron jobs) in the `steps/` directory

**Development:**
- The Motia Workbench provides a visual UI for building, testing, and debugging
- Real-time logs, traces, and workflow visualization
- Hot reload for rapid development

**Note**: CLI commands and project templates may change. Always refer to the official docs above for the current setup method.

### Project Structure

```
my-motia-app/
├── steps/
│   ├── send-message.step.ts      # API endpoint
│   ├── process-message.step.ts   # Event handler
│   ├── daily-cleanup.step.ts     # Cron job
│   └── ai-agent.step.py          # Python AI agent
├── motia.config.ts               # Project configuration
├── package.json
└── tsconfig.json
```

---

## Core Concept: Steps

A **Step** is the fundamental building block in Motia. Every backend pattern (APIs, jobs, events, workflows, agents) is expressed as a Step.

### Step Anatomy

Every Step file contains two parts:

1. **Config** - Defines when/how the Step runs and gives it a unique identifier
2. **Handler** - The function that executes your business logic

### File Naming Convention

Motia auto-discovers files with these suffixes:
- `.step.ts` or `.step.js` (TypeScript/JavaScript)
- `_step.py` (Python)
- `_step.rb` (Ruby - Beta)

---

## Step Types

### 1. API Steps (HTTP Endpoints)

Handle HTTP requests like traditional REST APIs.

**steps/send-message.step.ts**
```typescript
import type { ApiRouteConfig, Handlers } from 'motia';

export const config: ApiRouteConfig = {
  name: 'SendMessage',
  type: 'api',
  path: '/messages',
  method: 'POST',
  emits: ['message.sent'],      // Events this Step emits
  flows: ['messaging']           // Group for organization
};

export const handler: Handlers['SendMessage'] = async (req, context) => {
  const { text, userId } = req.body;

  // Validate input
  if (!text || !userId) {
    return {
      status: 400,
      body: { error: 'Missing required fields' }
    };
  }

  const message = {
    id: crypto.randomUUID(),
    text,
    userId,
    timestamp: new Date().toISOString()
  };

  // Save to state
  await context.state.set('messages', userId, message);

  // Emit event for other Steps
  await context.emit({
    topic: 'message.sent',
    data: message
  });

  // Push to real-time stream
  await context.streams.messages.set(userId, message);

  return {
    status: 201,
    body: { message }
  };
};
```

**Supported HTTP Methods:**
```typescript
method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE'
```

**Query Parameters:**
```typescript
export const handler: Handlers['GetMessages'] = async (req, context) => {
  const { userId, limit = '10' } = req.query;
  const messages = await context.state.get('messages', userId);

  return {
    status: 200,
    body: { messages: messages.slice(0, Number(limit)) }
  };
};
```

### 2. Event Steps (Background Processing)

Subscribe to topics and react to events from other Steps.

**steps/process-message.step.ts**
```typescript
import type { EventConfig, Handlers } from 'motia';

export const config: EventConfig = {
  name: 'ProcessMessage',
  type: 'event',
  subscribes: ['message.sent'],  // Topic to listen to
  emits: ['message.processed'],  // Events this Step emits
  flows: ['messaging']
};

export const handler: Handlers['ProcessMessage'] = async (input, context) => {
  const message = input.data;

  context.logger.info('Processing message', { messageId: message.id });

  // Perform background task (e.g., sentiment analysis)
  const sentiment = await analyzeSentiment(message.text);

  // Update state
  await context.state.set('sentiment', message.id, {
    messageId: message.id,
    sentiment,
    processedAt: new Date().toISOString()
  });

  // Emit completion event
  await context.emit({
    topic: 'message.processed',
    data: {
      messageId: message.id,
      sentiment
    }
  });
};

async function analyzeSentiment(text: string): Promise<'positive' | 'negative' | 'neutral'> {
  // Implement sentiment analysis
  return 'positive';
}
```

### 3. Cron Steps (Scheduled Jobs)

Run tasks on a schedule using cron expressions.

**steps/daily-cleanup.step.ts**
```typescript
import type { CronConfig, Handlers } from 'motia';

export const config: CronConfig = {
  name: 'DailyCleanup',
  type: 'cron',
  cron: '0 2 * * *',            // Every day at 2 AM
  emits: ['cleanup.completed'],
  flows: ['maintenance']
};

export const handler: Handlers['DailyCleanup'] = async (input, context) => {
  context.logger.info('Starting daily cleanup');

  // Delete old messages
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - 30);

  const allMessages = await context.state.getAll('messages');
  let deletedCount = 0;

  for (const [key, message] of Object.entries(allMessages)) {
    if (new Date(message.timestamp) < cutoffDate) {
      await context.state.delete('messages', key);
      deletedCount++;
    }
  }

  context.logger.info(`Cleanup completed: ${deletedCount} messages deleted`);

  await context.emit({
    topic: 'cleanup.completed',
    data: { deletedCount, timestamp: new Date().toISOString() }
  });
};
```

**Common Cron Patterns:**
```typescript
'*/5 * * * *'   // Every 5 minutes
'0 * * * *'     // Every hour
'0 9 * * *'     // Every day at 9 AM
'0 0 * * 0'     // Every Sunday at midnight
'0 0 1 * *'     // First day of every month
```

### 4. Noop Steps (Manual Trigger)

Steps that don't run automatically but can be triggered by external processes.

**steps/manual-report.step.ts**
```typescript
import type { NoopConfig, Handlers } from 'motia';

export const config: NoopConfig = {
  name: 'GenerateReport',
  type: 'noop',
  emits: ['report.generated'],
  flows: ['reporting']
};

export const handler: Handlers['GenerateReport'] = async (input, context) => {
  const { reportType, startDate, endDate } = input;

  context.logger.info('Generating report', { reportType, startDate, endDate });

  // Generate report
  const data = await fetchReportData(startDate, endDate);
  const report = formatReport(data, reportType);

  await context.state.set('reports', `${reportType}-${Date.now()}`, report);

  await context.emit({
    topic: 'report.generated',
    data: { reportType, url: report.url }
  });

  return report;
};
```

---

## Handler Context API

Every handler receives a context object with powerful utilities.

### emit() - Publish Events

```typescript
await context.emit({
  topic: 'user.created',
  data: {
    userId: '123',
    email: 'user@example.com',
    timestamp: new Date().toISOString()
  }
});

// Emit multiple events
await Promise.all([
  context.emit({ topic: 'event.one', data: { foo: 'bar' } }),
  context.emit({ topic: 'event.two', data: { baz: 'qux' } })
]);
```

### logger - Structured Logging

```typescript
// Automatically includes Step name, trace ID, and timestamp
context.logger.info('User logged in', { userId: '123' });
context.logger.warn('Rate limit approaching', { remaining: 10 });
context.logger.error('Payment failed', { error: err.message });

// Logs appear in Workbench UI with full context
```

### state - Persistent Storage

```typescript
// Set value
await context.state.set('users', userId, {
  name: 'John Doe',
  email: 'john@example.com'
});

// Get value
const user = await context.state.get('users', userId);

// Get all values in namespace
const allUsers = await context.state.getAll('users');

// Delete value
await context.state.delete('users', userId);

// Check existence
const exists = await context.state.has('users', userId);
```

**State Namespaces:**
```typescript
// Organize state by category
await context.state.set('cache', 'api-response', data);
await context.state.set('sessions', sessionId, sessionData);
await context.state.set('counters', 'page-views', 12345);
```

### streams - Real-Time Updates

```typescript
// Push data to connected clients
await context.streams.notifications.set(userId, {
  type: 'message',
  content: 'You have a new message',
  timestamp: Date.now()
});

// Broadcast to all clients
await context.streams.global.set('all', {
  announcement: 'System maintenance in 10 minutes'
});

// Client subscribes via WebSocket
// Frontend automatically receives updates
```

---

## Multi-Language Support

### TypeScript + Python Together

**steps/analyze-text.step.py** (Python for AI)
```python
from motia import EventConfig, handler

config = EventConfig(
    name="AnalyzeText",
    type="event",
    subscribes=["text.submitted"],
    emits=["text.analyzed"],
    flows=["ai"]
)

@handler
async def handle(input, context):
    text = input["data"]["text"]

    # Use Python ML libraries
    import nltk
    from transformers import pipeline

    sentiment = pipeline("sentiment-analysis")(text)

    await context.emit(
        topic="text.analyzed",
        data={
            "text": text,
            "sentiment": sentiment[0]["label"],
            "score": sentiment[0]["score"]
        }
    )
```

**steps/display-results.step.ts** (TypeScript for API)
```typescript
export const config: EventConfig = {
  name: 'DisplayResults',
  type: 'event',
  subscribes: ['text.analyzed'],
  flows: ['ai']
};

export const handler: Handlers['DisplayResults'] = async (input, context) => {
  const { text, sentiment, score } = input.data;

  // Store results
  await context.state.set('analyses', crypto.randomUUID(), {
    text,
    sentiment,
    score,
    timestamp: new Date().toISOString()
  });

  // Push to frontend
  await context.streams.results.set('latest', { sentiment, score });

  context.logger.info('Analysis displayed', { sentiment, score });
};
```

---

## Event-Driven Workflows

### Simple Linear Flow

```typescript
// Step 1: API receives request
export const config: ApiRouteConfig = {
  name: 'CreateOrder',
  type: 'api',
  path: '/orders',
  method: 'POST',
  emits: ['order.created']
};

// Step 2: Process payment
export const config: EventConfig = {
  name: 'ProcessPayment',
  type: 'event',
  subscribes: ['order.created'],
  emits: ['payment.processed']
};

// Step 3: Send confirmation
export const config: EventConfig = {
  name: 'SendConfirmation',
  type: 'event',
  subscribes: ['payment.processed'],
  emits: ['confirmation.sent']
};
```

### Parallel Processing

```typescript
// One event triggers multiple Steps in parallel
export const config: EventConfig = {
  name: 'OrderCreated',
  type: 'event',
  subscribes: ['order.created']
};

export const config: EventConfig = {
  name: 'InventoryUpdate',
  type: 'event',
  subscribes: ['order.created']  // Same topic
};

export const config: EventConfig = {
  name: 'AnalyticsTracking',
  type: 'event',
  subscribes: ['order.created']  // Same topic
};
```

### Conditional Workflows

```typescript
export const handler: Handlers['ProcessOrder'] = async (input, context) => {
  const order = input.data;

  if (order.total > 1000) {
    // High-value order - require manual approval
    await context.emit({
      topic: 'order.requiresApproval',
      data: order
    });
  } else {
    // Auto-approve low-value orders
    await context.emit({
      topic: 'order.approved',
      data: order
    });
  }
};
```

---

## AI Agent Patterns

### Simple AI Agent

**steps/ai-chat.step.ts**
```typescript
import type { ApiRouteConfig, Handlers } from 'motia';
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export const config: ApiRouteConfig = {
  name: 'AIChat',
  type: 'api',
  path: '/chat',
  method: 'POST',
  flows: ['ai']
};

export const handler: Handlers['AIChat'] = async (req, context) => {
  const { message, userId } = req.body;

  // Get conversation history from state
  const history = await context.state.get('conversations', userId) || [];

  // Add user message
  history.push({ role: 'user', content: message });

  // Call AI
  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: history
  });

  const aiResponse = completion.choices[0].message.content;

  // Save updated history
  history.push({ role: 'assistant', content: aiResponse });
  await context.state.set('conversations', userId, history);

  // Stream response to user
  await context.streams.chat.set(userId, {
    message: aiResponse,
    timestamp: Date.now()
  });

  return {
    status: 200,
    body: { response: aiResponse }
  };
};
```

### Multi-Agent System

**steps/research-coordinator.step.ts** (Coordinator)
```typescript
export const config: ApiRouteConfig = {
  name: 'ResearchCoordinator',
  type: 'api',
  path: '/research',
  method: 'POST',
  emits: ['research.started'],
  flows: ['research']
};

export const handler: Handlers['ResearchCoordinator'] = async (req, context) => {
  const { topic } = req.body;
  const researchId = crypto.randomUUID();

  // Trigger parallel research agents
  await Promise.all([
    context.emit({
      topic: 'research.web',
      data: { researchId, topic }
    }),
    context.emit({
      topic: 'research.academic',
      data: { researchId, topic }
    }),
    context.emit({
      topic: 'research.news',
      data: { researchId, topic }
    })
  ]);

  return {
    status: 202,
    body: { researchId, status: 'started' }
  };
};
```

**steps/web-researcher.step.py** (Specialist Agent)
```python
from motia import EventConfig, handler
import openai

config = EventConfig(
    name="WebResearcher",
    type="event",
    subscribes=["research.web"],
    emits=["research.web.completed"],
    flows=["research"]
)

@handler
async def handle(input, context):
    research_id = input["data"]["researchId"]
    topic = input["data"]["topic"]

    # Web research logic
    results = await perform_web_research(topic)

    await context.state.set("research_results", f"{research_id}-web", results)

    await context.emit(
        topic="research.web.completed",
        data={"researchId": research_id, "results": results}
    )
```

**steps/synthesize-results.step.ts** (Aggregator)
```typescript
export const config: EventConfig = {
  name: 'SynthesizeResults',
  type: 'event',
  subscribes: [
    'research.web.completed',
    'research.academic.completed',
    'research.news.completed'
  ],
  emits: ['research.completed'],
  flows: ['research']
};

export const handler: Handlers['SynthesizeResults'] = async (input, context) => {
  const { researchId } = input.data;

  // Check if all research is complete
  const webResults = await context.state.get('research_results', `${researchId}-web`);
  const academicResults = await context.state.get('research_results', `${researchId}-academic`);
  const newsResults = await context.state.get('research_results', `${researchId}-news`);

  if (!webResults || !academicResults || !newsResults) {
    // Not all results ready yet
    return;
  }

  // Synthesize with AI
  const synthesis = await synthesizeWithAI({
    web: webResults,
    academic: academicResults,
    news: newsResults
  });

  await context.state.set('research_final', researchId, synthesis);

  await context.emit({
    topic: 'research.completed',
    data: { researchId, synthesis }
  });

  // Notify user
  await context.streams.research.set(researchId, {
    status: 'completed',
    synthesis
  });
};
```

---

## Real-Time Streaming

### Frontend Integration

**Client-side (React)**
```typescript
import { useMotiaStream } from '@motia/react';

function NotificationComponent({ userId }: { userId: string }) {
  const notifications = useMotiaStream('notifications', userId);

  return (
    <div>
      {notifications.map(notif => (
        <div key={notif.timestamp}>
          {notif.content}
        </div>
      ))}
    </div>
  );
}
```

**Backend streaming**
```typescript
export const handler: Handlers['SendNotification'] = async (req, context) => {
  const { userId, message } = req.body;

  // Push to stream (automatically sent to connected clients)
  await context.streams.notifications.set(userId, {
    type: 'info',
    content: message,
    timestamp: Date.now()
  });

  return { status: 200, body: { sent: true } };
};
```

---

## Error Handling

### Try-Catch in Handlers

```typescript
export const handler: Handlers['ProcessPayment'] = async (req, context) => {
  try {
    const payment = await processPaymentAPI(req.body);

    await context.emit({
      topic: 'payment.succeeded',
      data: payment
    });

    return { status: 200, body: { payment } };
  } catch (error) {
    context.logger.error('Payment failed', {
      error: error.message,
      userId: req.body.userId
    });

    // Emit failure event
    await context.emit({
      topic: 'payment.failed',
      data: {
        userId: req.body.userId,
        reason: error.message
      }
    });

    return {
      status: 500,
      body: { error: 'Payment processing failed' }
    };
  }
};
```

### Dead Letter Queue Pattern

```typescript
// Main processor
export const config: EventConfig = {
  name: 'ProcessOrder',
  type: 'event',
  subscribes: ['order.created'],
  emits: ['order.processed', 'order.failed']
};

export const handler: Handlers['ProcessOrder'] = async (input, context) => {
  try {
    // Process order
    const result = await processOrder(input.data);
    await context.emit({ topic: 'order.processed', data: result });
  } catch (error) {
    // Send to DLQ
    await context.emit({
      topic: 'order.failed',
      data: {
        originalData: input.data,
        error: error.message,
        timestamp: Date.now()
      }
    });
  }
};

// DLQ handler
export const config: EventConfig = {
  name: 'HandleFailedOrders',
  type: 'event',
  subscribes: ['order.failed']
};

export const handler: Handlers['HandleFailedOrders'] = async (input, context) => {
  // Log to monitoring service
  await logToMonitoring(input.data);

  // Store for manual review
  await context.state.set('failed_orders', crypto.randomUUID(), input.data);

  // Alert team
  await context.emit({
    topic: 'alert.critical',
    data: { message: 'Order processing failed', details: input.data }
  });
};
```

---

## Testing

### Unit Test a Step

```typescript
// send-message.step.test.ts
import { describe, it, expect, vi } from 'vitest';
import { handler, config } from './send-message.step';

describe('SendMessage Step', () => {
  it('should send a message successfully', async () => {
    const mockContext = {
      state: {
        set: vi.fn()
      },
      emit: vi.fn(),
      streams: {
        messages: {
          set: vi.fn()
        }
      },
      logger: {
        info: vi.fn(),
        error: vi.fn()
      }
    };

    const req = {
      body: {
        text: 'Hello, world!',
        userId: 'user123'
      },
      query: {},
      params: {},
      headers: {}
    };

    const result = await handler(req, mockContext as any);

    expect(result.status).toBe(201);
    expect(mockContext.emit).toHaveBeenCalledWith(
      expect.objectContaining({
        topic: 'message.sent'
      })
    );
  });
});
```

### Integration Tests with Workbench

```bash
# Start test environment
npx motia dev --test

# Run integration tests
npm test
```

---

## Deployment

### Deploy to Motia Cloud

```bash
# Login to Motia Cloud
npx motia login

# Deploy project
npx motia deploy

# View logs
npx motia logs

# Check status
npx motia status
```

### Environment Variables

```bash
# Set production environment variables
npx motia env set DATABASE_URL="postgres://..."
npx motia env set OPENAI_API_KEY="sk-..."

# List all environment variables
npx motia env list
```

### Custom Deployment

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npx", "motia", "start"]
```

---

## Best Practices

### 1. Step Naming

```typescript
// ✅ Good: Clear, action-oriented names
name: 'SendWelcomeEmail'
name: 'ProcessPayment'
name: 'AnalyzeUserBehavior'

// ❌ Bad: Vague or generic names
name: 'Handler1'
name: 'DoStuff'
name: 'API'
```

### 2. Event Topic Conventions

```typescript
// Use namespace.action format
emits: ['user.created']
emits: ['payment.processed']
emits: ['order.shipped']

// Include entity and action
subscribes: ['message.sent']
subscribes: ['file.uploaded']
subscribes: ['task.completed']
```

### 3. State Organization

```typescript
// Use clear namespaces
await context.state.set('users', userId, userData);
await context.state.set('sessions', sessionId, sessionData);
await context.state.set('cache', cacheKey, cachedData);

// ❌ Avoid: Unorganized state
await context.state.set('data', 'random-key', something);
```

### 4. Error Handling

```typescript
// Always handle errors gracefully
try {
  await riskyOperation();
} catch (error) {
  context.logger.error('Operation failed', { error: error.message });
  await context.emit({
    topic: 'operation.failed',
    data: { reason: error.message }
  });
  // Return appropriate response
}
```

### 5. Idempotency

```typescript
// Make Steps idempotent (safe to retry)
export const handler: Handlers['ProcessOrder'] = async (input, context) => {
  const { orderId } = input.data;

  // Check if already processed
  const existing = await context.state.get('processed_orders', orderId);
  if (existing) {
    context.logger.info('Order already processed', { orderId });
    return; // Skip duplicate processing
  }

  // Process order
  await processOrder(input.data);

  // Mark as processed
  await context.state.set('processed_orders', orderId, {
    processedAt: new Date().toISOString()
  });
};
```

---

## Resources

- **Documentation:** https://motia.dev/docs
- **GitHub:** https://github.com/MotiaDev/motia
- **Examples:** https://github.com/MotiaDev/motia-examples
- **Discord Community:** https://discord.gg/motia
- **Workbench UI:** http://localhost:3000 (when running `motia dev`)
- **Public Roadmap:** https://github.com/orgs/MotiaDev/projects/2
