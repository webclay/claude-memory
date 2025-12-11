# Durable Streams

Durable Streams is an open protocol for reliably streaming data to client applications with offset-based resumability.

## Why Use It

- **Resumable**: If connection drops mid-stream, resume from exact offset (no data loss)
- **Perfect for AI**: LLM token streaming that survives network failures
- **HTTP-native**: Works through CDNs, edge caching, standard infrastructure
- **Production-proven**: 1.5 years in production at Electric SQL

## The Problem It Solves

WebSocket and SSE connections are fragile. When they fail:
- AI token streams lose all progress
- Real-time sync requires full re-fetch
- Developers build custom resume logic repeatedly

Durable Streams standardizes resumable streaming over HTTP.

---

## Installation

```bash
# Client only (smaller bundle)
npm install @durable-streams/client

# Client + Server
npm install @durable-streams/client @durable-streams/writer @durable-streams/server
```

---

## Quick Start

### Server Setup (Node.js)

```typescript
import { createServer } from '@durable-streams/server';

const server = createServer({
  // Storage adapter (in-memory, Redis, Postgres, etc.)
  storage: 'memory'
});

server.listen(3001);
```

### Writing to a Stream

```typescript
import { DurableStreamsWriter } from '@durable-streams/writer';

const client = new DurableStreamsWriter('http://localhost:3001');

// Create a stream
await client.create('my-stream');

// Append data
await client.append('my-stream', 'Hello, ');
await client.append('my-stream', 'World!');

// Append JSON
await client.append('my-stream', JSON.stringify({ token: 'Hello' }));
```

### Reading from a Stream (with Resume)

```typescript
import { DurableStreamsClient } from '@durable-streams/client';

const client = new DurableStreamsClient('http://localhost:3001');

// Get last offset from localStorage (or database)
let lastOffset = localStorage.getItem('stream-offset') || 0;

// Read with resume capability
const stream = client.stream('my-stream');

for await (const chunk of stream.read({
  offset: lastOffset,
  live: true  // Keep connection open for new data
})) {
  console.log(chunk.data);

  // Save offset for resume
  lastOffset = chunk.offset;
  localStorage.setItem('stream-offset', lastOffset);
}
```

---

## AI Token Streaming Example

The killer use case - resumable LLM streaming:

### Server (API Route)

```typescript
// TanStack Start: src/routes/api/chat.ts
import { createFileRoute } from '@tanstack/react-router';
import { DurableStreamsWriter } from '@durable-streams/writer';

const streams = new DurableStreamsWriter(process.env.STREAMS_URL);

export const Route = createFileRoute('/api/chat')({
  server: {
    handlers: {
      POST: async ({ request }) => {
        const { message, streamId } = await request.json();

        // Create stream for this chat
        await streams.create(streamId);

        // Stream tokens from LLM
        const response = await openai.chat.completions.create({
          model: 'gpt-4',
          messages: [{ role: 'user', content: message }],
          stream: true
        });

        for await (const chunk of response) {
          const token = chunk.choices[0]?.delta?.content || '';
          if (token) {
            // Each token is durably stored with an offset
            await streams.append(streamId, token);
          }
        }

        // Signal completion
        await streams.append(streamId, JSON.stringify({ done: true }));

        return Response.json({ streamId });
      }
    }
  }
});
```

### Client (React Component)

```typescript
import { DurableStreamsClient } from '@durable-streams/client';
import { useState, useEffect } from 'react';

const client = new DurableStreamsClient(process.env.STREAMS_URL);

function ChatMessage({ streamId }: { streamId: string }) {
  const [content, setContent] = useState('');
  const [offset, setOffset] = useState(() =>
    Number(sessionStorage.getItem(`offset-${streamId}`)) || 0
  );

  useEffect(() => {
    const stream = client.stream(streamId);
    let cancelled = false;

    async function consume() {
      for await (const chunk of stream.read({ offset, live: true })) {
        if (cancelled) break;

        // Check for completion signal
        try {
          const parsed = JSON.parse(chunk.data);
          if (parsed.done) break;
        } catch {
          // Regular token
          setContent(prev => prev + chunk.data);
        }

        // Save offset for resume
        setOffset(chunk.offset);
        sessionStorage.setItem(`offset-${streamId}`, chunk.offset);
      }
    }

    consume();
    return () => { cancelled = true; };
  }, [streamId, offset]);

  return <div>{content}</div>;
}
```

Now if the user refreshes or loses connection, they resume from exactly where they left off!

---

## HTTP Protocol

Durable Streams uses simple HTTP operations:

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Create stream | `PUT` | `/{stream-id}` |
| Append data | `POST` | `/{stream-id}` |
| Read data | `GET` | `/{stream-id}?offset={n}&live={bool}` |
| Delete stream | `DELETE` | `/{stream-id}` |

### Reading Modes

```typescript
// One-shot read (returns immediately)
stream.read({ offset: 0 })

// Live tailing (long-poll, waits for new data)
stream.read({ offset: 0, live: true })

// JSON mode (automatic message boundaries)
stream.json({ offset: 0, live: true })
```

---

## Use Cases

| Use Case | Why Durable Streams |
|----------|---------------------|
| **AI token streaming** | Resume mid-response after disconnect |
| **Real-time sync** | Ordered, replayable event streams |
| **Collaborative editing** | Operation logs with catch-up |
| **IoT data ingestion** | Reliable sensor data delivery |
| **Activity feeds** | Paginated with live updates |

---

## Integration with Vercel AI SDK

Wrap Vercel AI SDK streams with Durable Streams for resumability:

```typescript
import { streamText } from 'ai';
import { DurableStreamsWriter } from '@durable-streams/writer';

const streams = new DurableStreamsWriter(process.env.STREAMS_URL);

export async function POST(request: Request) {
  const { messages, streamId } = await request.json();

  await streams.create(streamId);

  const result = await streamText({
    model: openai('gpt-4'),
    messages,
    onChunk: async ({ chunk }) => {
      if (chunk.type === 'text-delta') {
        await streams.append(streamId, chunk.textDelta);
      }
    }
  });

  return result.toDataStreamResponse();
}
```

---

## Storage Backends

The reference server supports multiple storage backends:

```typescript
import { createServer } from '@durable-streams/server';

// In-memory (development)
createServer({ storage: 'memory' });

// Redis (production)
createServer({
  storage: 'redis',
  redis: { url: process.env.REDIS_URL }
});

// PostgreSQL (production)
createServer({
  storage: 'postgres',
  postgres: { connectionString: process.env.DATABASE_URL }
});
```

---

## Best Practices

### 1. Always Save Offsets
```typescript
// Save after each chunk for perfect resume
sessionStorage.setItem(`offset-${streamId}`, chunk.offset);
```

### 2. Use Unique Stream IDs
```typescript
// Include user context to prevent conflicts
const streamId = `chat-${userId}-${Date.now()}`;
```

### 3. Clean Up Old Streams
```typescript
// Delete streams after completion
await streams.delete(streamId);
```

### 4. Handle Completion Signals
```typescript
// Send explicit end marker
await streams.append(streamId, JSON.stringify({ type: 'done' }));
```

---

## Resources

- **GitHub:** https://github.com/durable-streams/durable-streams
- **Protocol Spec:** https://github.com/durable-streams/durable-streams/blob/main/PROTOCOL.md
- **npm:** https://www.npmjs.com/package/@durable-streams/client
