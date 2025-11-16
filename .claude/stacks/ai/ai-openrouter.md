# OpenRouter API Integration Patterns

## Overview

OpenRouter provides unified access to multiple LLM providers (OpenAI, Anthropic, Google, Meta, Mistral, and more) through a single API. It automatically routes requests to the best available model, handles fallbacks, and provides cost optimization.

**Key Features:**
- Access 100+ models through one API
- Automatic fallback and load balancing
- Real-time model availability
- Cost tracking and optimization
- OpenAI-compatible API
- Streaming support

**Official Website:** https://openrouter.ai

---

## Installation and Setup

**📚 Official Documentation**: [OpenRouter - API Documentation](https://openrouter.ai/docs)

Follow the official documentation for the latest setup instructions. OpenRouter is OpenAI-compatible, so you can use the OpenAI SDK.

### Quick Reference

The general setup process involves:
1. Sign up at [openrouter.ai](https://openrouter.ai)
2. Create an API key in the Keys section
3. Install the OpenAI SDK (OpenRouter is compatible with it)
4. Configure your environment variables
5. Initialize the client with OpenRouter's base URL

**Note**: Setup steps and SDK requirements may change. Always refer to the official docs above for the current installation method.

### Environment Variables

```bash
# .env.local
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxxx

# Optional: Your site name (for rankings and model author attribution)
OPENROUTER_SITE_URL=https://yourapp.com
OPENROUTER_SITE_NAME=YourApp
```

---

## Basic Usage

### Initialize Client

**lib/openrouter.ts**
```typescript
import OpenAI from 'openai';

export const openrouter = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: process.env.OPENROUTER_API_KEY,
  defaultHeaders: {
    'HTTP-Referer': process.env.OPENROUTER_SITE_URL, // Optional
    'X-Title': process.env.OPENROUTER_SITE_NAME, // Optional
  }
});
```

### Simple Chat Completion

```typescript
import { openrouter } from '@/lib/openrouter';

export async function generateResponse(userMessage: string) {
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      { role: 'user', content: userMessage }
    ]
  });

  return completion.choices[0].message.content;
}
```

---

## Available Models

### Popular Models

```typescript
// Anthropic Models
'anthropic/claude-3.5-sonnet'         // Best overall, most capable
'anthropic/claude-3-opus'             // Previous flagship
'anthropic/claude-3-haiku'            // Fast and cost-effective

// OpenAI Models
'openai/gpt-4-turbo'                  // Latest GPT-4
'openai/gpt-4'                        // Standard GPT-4
'openai/gpt-3.5-turbo'                // Fast and cheap

// Google Models
'google/gemini-pro-1.5'               // Large context (2M tokens)
'google/gemini-flash-1.5'             // Fast and efficient

// Meta Models
'meta-llama/llama-3.1-405b-instruct'  // Most capable Llama
'meta-llama/llama-3.1-70b-instruct'   // Balanced
'meta-llama/llama-3.1-8b-instruct'    // Fast and cheap

// Mistral Models
'mistralai/mistral-large'             // Most capable
'mistralai/mistral-medium'            // Balanced
'mistralai/mistral-small'             // Fast

// Open Source Models
'qwen/qwen-2.5-72b-instruct'          // Strong reasoning
'cohere/command-r-plus'               // Good for RAG
```

### Get All Available Models

```typescript
export async function listModels() {
  const response = await fetch('https://openrouter.ai/api/v1/models', {
    headers: {
      'Authorization': `Bearer ${process.env.OPENROUTER_API_KEY}`
    }
  });

  const { data } = await response.json();
  return data; // Array of model objects with pricing, context length, etc.
}
```

---

## Streaming Responses

### Server-Side Streaming

**app/api/chat/route.ts** (Next.js)
```typescript
import { openrouter } from '@/lib/openrouter';
import { OpenAIStream, StreamingTextResponse } from 'ai';

export const runtime = 'edge';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const response = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages,
    stream: true
  });

  const stream = OpenAIStream(response);
  return new StreamingTextResponse(stream);
}
```

### Client-Side with Vercel AI SDK

```typescript
'use client';

import { useChat } from 'ai/react';

export function ChatInterface() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat'
  });

  return (
    <div className="flex flex-col h-screen">
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map(message => (
          <div
            key={message.id}
            className={`flex ${message.role === 'user' ? 'justify-end' : 'justify-start'}`}
          >
            <div
              className={`max-w-md px-4 py-2 rounded-lg ${
                message.role === 'user'
                  ? 'bg-blue-500 text-white'
                  : 'bg-gray-200 text-black'
              }`}
            >
              {message.content}
            </div>
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit} className="border-t p-4">
        <div className="flex space-x-2">
          <input
            value={input}
            onChange={handleInputChange}
            placeholder="Type your message..."
            className="flex-1 px-4 py-2 border rounded-lg"
            disabled={isLoading}
          />
          <button
            type="submit"
            disabled={isLoading}
            className="px-6 py-2 bg-blue-500 text-white rounded-lg disabled:opacity-50"
          >
            Send
          </button>
        </div>
      </form>
    </div>
  );
}
```

### Manual Streaming (Without AI SDK)

```typescript
export async function streamChat(messages: Message[]) {
  const stream = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages,
    stream: true
  });

  let fullResponse = '';

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    fullResponse += content;
    console.log(content); // Process chunk
  }

  return fullResponse;
}
```

---

## Advanced Features

### Model Routing and Fallbacks

OpenRouter automatically routes to available models. You can specify fallback models:

```typescript
const completion = await openrouter.chat.completions.create({
  model: 'anthropic/claude-3.5-sonnet',
  messages,
  // If primary model is unavailable, try these in order
  route: 'fallback',
  models: [
    'anthropic/claude-3.5-sonnet',
    'openai/gpt-4-turbo',
    'google/gemini-pro-1.5'
  ]
});
```

### Cost Optimization

```typescript
// Use auto-router to select cheapest model that meets requirements
const completion = await openrouter.chat.completions.create({
  model: 'openrouter/auto', // Let OpenRouter choose
  messages,
  // Optionally specify constraints
  route: 'cheapest', // or 'fastest', 'balanced'
});
```

### Provider Preferences

```typescript
const completion = await openrouter.chat.completions.create({
  model: 'anthropic/claude-3.5-sonnet',
  messages,
  provider: {
    // Require specific provider
    require: ['Anthropic'],
    // Or allow multiple
    allow: ['Anthropic', 'OpenAI'],
    // Ignore specific providers
    ignore: ['SomeUnreliableProvider']
  }
});
```

### Context Window Management

```typescript
import { encoding_for_model } from 'tiktoken';

export function truncateMessages(
  messages: Message[],
  maxTokens: number = 100000
): Message[] {
  const encoder = encoding_for_model('gpt-4');
  let totalTokens = 0;
  const truncated: Message[] = [];

  // Always keep system message
  if (messages[0]?.role === 'system') {
    truncated.push(messages[0]);
    totalTokens += encoder.encode(messages[0].content).length;
  }

  // Add messages from most recent, working backwards
  for (let i = messages.length - 1; i >= 0; i--) {
    const message = messages[i];
    const tokens = encoder.encode(message.content).length;

    if (totalTokens + tokens > maxTokens) break;

    truncated.unshift(message);
    totalTokens += tokens;
  }

  encoder.free();
  return truncated;
}

// Usage
const completion = await openrouter.chat.completions.create({
  model: 'google/gemini-pro-1.5', // 2M token context
  messages: truncateMessages(messages, 1_900_000) // Leave room for response
});
```

---

## Function Calling / Tool Use

### Define Tools

```typescript
const tools = [
  {
    type: 'function',
    function: {
      name: 'get_weather',
      description: 'Get the current weather for a location',
      parameters: {
        type: 'object',
        properties: {
          location: {
            type: 'string',
            description: 'The city and state, e.g. San Francisco, CA'
          },
          unit: {
            type: 'string',
            enum: ['celsius', 'fahrenheit'],
            description: 'Temperature unit'
          }
        },
        required: ['location']
      }
    }
  }
];
```

### Use Tools in Chat

```typescript
export async function chatWithTools(userMessage: string) {
  const messages = [
    { role: 'user', content: userMessage }
  ];

  const response = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages,
    tools,
    tool_choice: 'auto' // Let model decide when to use tools
  });

  const message = response.choices[0].message;

  // Check if model wants to use a tool
  if (message.tool_calls) {
    const toolCall = message.tool_calls[0];

    // Execute the tool
    const toolResult = await executeToolCall(toolCall);

    // Send tool result back to model
    const finalResponse = await openrouter.chat.completions.create({
      model: 'anthropic/claude-3.5-sonnet',
      messages: [
        ...messages,
        message,
        {
          role: 'tool',
          tool_call_id: toolCall.id,
          content: JSON.stringify(toolResult)
        }
      ]
    });

    return finalResponse.choices[0].message.content;
  }

  return message.content;
}

async function executeToolCall(toolCall: ToolCall) {
  const { name, arguments: args } = toolCall.function;
  const params = JSON.parse(args);

  if (name === 'get_weather') {
    return await getWeather(params.location, params.unit);
  }

  throw new Error(`Unknown tool: ${name}`);
}
```

---

## Rate Limiting and Error Handling

### Retry with Exponential Backoff

```typescript
export async function chatWithRetry(
  messages: Message[],
  maxRetries = 3
): Promise<string> {
  let lastError: Error | null = null;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const completion = await openrouter.chat.completions.create({
        model: 'anthropic/claude-3.5-sonnet',
        messages
      });

      return completion.choices[0].message.content || '';
    } catch (error: any) {
      lastError = error;

      // Don't retry on certain errors
      if (error.status === 400 || error.status === 401) {
        throw error;
      }

      // Rate limit - wait before retrying
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      // Server error - retry
      if (error.status >= 500) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      throw error;
    }
  }

  throw lastError || new Error('Max retries exceeded');
}
```

### Handle Errors Gracefully

```typescript
export async function safeChat(messages: Message[]): Promise<{
  content: string | null;
  error: string | null;
}> {
  try {
    const completion = await openrouter.chat.completions.create({
      model: 'anthropic/claude-3.5-sonnet',
      messages
    });

    return {
      content: completion.choices[0].message.content,
      error: null
    };
  } catch (error: any) {
    console.error('OpenRouter error:', error);

    // User-friendly error messages
    if (error.status === 429) {
      return {
        content: null,
        error: 'Rate limit exceeded. Please try again in a moment.'
      };
    }

    if (error.status === 401) {
      return {
        content: null,
        error: 'Invalid API key. Please check your configuration.'
      };
    }

    if (error.status === 402) {
      return {
        content: null,
        error: 'Insufficient credits. Please add credits to your OpenRouter account.'
      };
    }

    return {
      content: null,
      error: 'An error occurred. Please try again.'
    };
  }
}
```

---

## Cost Tracking

### Get Generation Metadata

```typescript
const completion = await openrouter.chat.completions.create({
  model: 'anthropic/claude-3.5-sonnet',
  messages
});

// OpenRouter includes cost info in response
const usage = completion.usage;
console.log({
  promptTokens: usage?.prompt_tokens,
  completionTokens: usage?.completion_tokens,
  totalTokens: usage?.total_tokens
});

// Calculate cost (varies by model)
const costPerToken = 0.000003; // Example: $3 per 1M tokens
const totalCost = (usage?.total_tokens || 0) * costPerToken;
console.log(`Cost: $${totalCost.toFixed(6)}`);
```

### Track Costs in Database

```typescript
interface GenerationLog {
  id: string;
  userId: string;
  model: string;
  promptTokens: number;
  completionTokens: number;
  totalTokens: number;
  cost: number;
  createdAt: Date;
}

export async function logGeneration(
  userId: string,
  model: string,
  usage: CompletionUsage
) {
  const modelPricing = await getModelPricing(model);

  const promptCost = usage.prompt_tokens * modelPricing.promptPrice;
  const completionCost = usage.completion_tokens * modelPricing.completionPrice;
  const totalCost = promptCost + completionCost;

  await db.generationLogs.create({
    data: {
      userId,
      model,
      promptTokens: usage.prompt_tokens,
      completionTokens: usage.completion_tokens,
      totalTokens: usage.total_tokens,
      cost: totalCost
    }
  });

  return totalCost;
}
```

---

## Multi-Model Comparison

Compare outputs from multiple models simultaneously.

```typescript
export async function compareModels(
  userMessage: string,
  models: string[]
): Promise<{ model: string; response: string; cost: number }[]> {
  const messages = [{ role: 'user', content: userMessage }];

  const results = await Promise.all(
    models.map(async (model) => {
      try {
        const completion = await openrouter.chat.completions.create({
          model,
          messages
        });

        const usage = completion.usage!;
        const pricing = await getModelPricing(model);
        const cost =
          usage.prompt_tokens * pricing.promptPrice +
          usage.completion_tokens * pricing.completionPrice;

        return {
          model,
          response: completion.choices[0].message.content || '',
          cost,
          tokens: usage.total_tokens
        };
      } catch (error) {
        console.error(`Error with model ${model}:`, error);
        return {
          model,
          response: '[Error]',
          cost: 0,
          tokens: 0
        };
      }
    })
  );

  return results;
}

// Usage
const comparison = await compareModels(
  'Explain quantum computing in simple terms',
  [
    'anthropic/claude-3.5-sonnet',
    'openai/gpt-4-turbo',
    'google/gemini-pro-1.5',
    'meta-llama/llama-3.1-70b-instruct'
  ]
);

comparison.forEach(({ model, response, cost }) => {
  console.log(`\n${model} (${cost.toFixed(4)}):\n${response}`);
});
```

---

## RAG (Retrieval-Augmented Generation)

### Basic RAG Pattern

```typescript
interface Document {
  id: string;
  content: string;
  metadata: Record<string, any>;
}

export async function ragQuery(
  query: string,
  documents: Document[]
): Promise<string> {
  // 1. Embed and retrieve relevant documents (simplified)
  const relevantDocs = await retrieveRelevantDocs(query, documents);

  // 2. Create context from documents
  const context = relevantDocs
    .map(doc => doc.content)
    .join('\n\n---\n\n');

  // 3. Generate response with context
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      {
        role: 'system',
        content: 'You are a helpful assistant. Answer questions based on the provided context. If the answer is not in the context, say so.'
      },
      {
        role: 'user',
        content: `Context:\n${context}\n\nQuestion: ${query}`
      }
    ]
  });

  return completion.choices[0].message.content || '';
}
```

### With Citations

```typescript
export async function ragWithCitations(
  query: string,
  documents: Document[]
): Promise<{ answer: string; citations: Document[] }> {
  const relevantDocs = await retrieveRelevantDocs(query, documents);

  const context = relevantDocs
    .map((doc, index) => `[${index + 1}] ${doc.content}`)
    .join('\n\n');

  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      {
        role: 'system',
        content: 'Answer questions using the provided context. Include citation numbers [1], [2], etc. when referencing specific information.'
      },
      {
        role: 'user',
        content: `Context:\n${context}\n\nQuestion: ${query}`
      }
    ]
  });

  return {
    answer: completion.choices[0].message.content || '',
    citations: relevantDocs
  };
}
```

---

## Image Understanding (Vision Models)

Some models support image inputs (Claude 3, GPT-4 Vision, Gemini Pro Vision).

```typescript
export async function analyzeImage(
  imageUrl: string,
  prompt: string = 'Describe this image in detail'
): Promise<string> {
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      {
        role: 'user',
        content: [
          { type: 'text', text: prompt },
          {
            type: 'image_url',
            image_url: { url: imageUrl }
          }
        ]
      }
    ]
  });

  return completion.choices[0].message.content || '';
}

// With base64 image
export async function analyzeImageBase64(
  base64Image: string,
  mimeType: string = 'image/jpeg'
): Promise<string> {
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      {
        role: 'user',
        content: [
          { type: 'text', text: 'What is in this image?' },
          {
            type: 'image_url',
            image_url: {
              url: `data:${mimeType};base64,${base64Image}`
            }
          }
        ]
      }
    ]
  });

  return completion.choices[0].message.content || '';
}
```

---

## Structured Output

### JSON Mode

```typescript
interface ExtractedData {
  name: string;
  email: string;
  phone: string;
  company: string;
}

export async function extractContactInfo(text: string): Promise<ExtractedData> {
  const completion = await openrouter.chat.completions.create({
    model: 'openai/gpt-4-turbo',
    messages: [
      {
        role: 'system',
        content: 'Extract contact information and return as JSON.'
      },
      { role: 'user', content: text }
    ],
    response_format: { type: 'json_object' }
  });

  const content = completion.choices[0].message.content || '{}';
  return JSON.parse(content) as ExtractedData;
}
```

### With Zod Validation

```typescript
import { z } from 'zod';

const ContactSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  phone: z.string().optional(),
  company: z.string().optional()
});

export async function extractContactInfoSafe(
  text: string
): Promise<z.infer<typeof ContactSchema> | null> {
  try {
    const completion = await openrouter.chat.completions.create({
      model: 'openai/gpt-4-turbo',
      messages: [
        {
          role: 'system',
          content: `Extract contact information and return as JSON with this schema: ${JSON.stringify(ContactSchema.shape)}`
        },
        { role: 'user', content: text }
      ],
      response_format: { type: 'json_object' }
    });

    const content = completion.choices[0].message.content || '{}';
    const parsed = JSON.parse(content);

    return ContactSchema.parse(parsed);
  } catch (error) {
    console.error('Failed to extract contact info:', error);
    return null;
  }
}
```

---

## Best Practices

### 1. Model Selection

```typescript
// Choose model based on task requirements
const MODEL_FOR_TASK = {
  // Complex reasoning, long outputs
  reasoning: 'anthropic/claude-3.5-sonnet',

  // Fast, simple tasks
  simple: 'anthropic/claude-3-haiku',

  // Coding tasks
  coding: 'anthropic/claude-3.5-sonnet',

  // Huge context (documents, codebases)
  largeContext: 'google/gemini-pro-1.5',

  // Budget-friendly
  budget: 'meta-llama/llama-3.1-8b-instruct',

  // Function calling
  tools: 'anthropic/claude-3.5-sonnet'
} as const;
```

### 2. Prompt Engineering

```typescript
// Use clear, specific instructions
const systemPrompt = `You are an expert customer support agent.
- Be friendly and professional
- Keep responses concise (under 100 words)
- If you don't know something, say so
- Always offer to help further`;

// Structure complex prompts with XML tags (Claude works well with this)
const structuredPrompt = `
<task>Analyze this customer review</task>

<review>
${reviewText}
</review>

<output_format>
Return JSON with:
- sentiment: positive/negative/neutral
- topics: array of main topics mentioned
- urgency: low/medium/high
</output_format>
`;
```

### 3. Caching for Repeated Contexts

For prompts with repeated context (like system instructions), consider implementing client-side caching:

```typescript
const cache = new Map<string, { response: string; timestamp: number }>();
const CACHE_TTL = 1000 * 60 * 5; // 5 minutes

export async function cachedChat(
  messages: Message[],
  cacheKey?: string
): Promise<string> {
  if (cacheKey) {
    const cached = cache.get(cacheKey);
    if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
      return cached.response;
    }
  }

  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages
  });

  const response = completion.choices[0].message.content || '';

  if (cacheKey) {
    cache.set(cacheKey, { response, timestamp: Date.now() });
  }

  return response;
}
```

### 4. Security

```typescript
// Never expose API key to client
// ❌ Bad
const client = new OpenAI({
  apiKey: process.env.NEXT_PUBLIC_OPENROUTER_API_KEY // Bad!
});

// ✅ Good - Use server-side API routes
// app/api/chat/route.ts
export async function POST(req: Request) {
  const { messages } = await req.json();

  // Validate user is authenticated
  const session = await getSession();
  if (!session) {
    return new Response('Unauthorized', { status: 401 });
  }

  // Rate limit
  const { success } = await ratelimit.limit(session.user.id);
  if (!success) {
    return new Response('Rate limit exceeded', { status: 429 });
  }

  // Call OpenRouter from server
  const response = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages
  });

  return Response.json(response);
}
```

---

## Monitoring and Analytics

### Track Model Performance

```typescript
interface ModelMetrics {
  model: string;
  avgResponseTime: number;
  avgTokens: number;
  avgCost: number;
  successRate: number;
  totalRequests: number;
}

export async function trackGeneration(
  model: string,
  startTime: number,
  usage: CompletionUsage,
  cost: number,
  success: boolean
) {
  const responseTime = Date.now() - startTime;

  await db.metrics.create({
    data: {
      model,
      responseTime,
      tokens: usage.total_tokens,
      cost,
      success,
      timestamp: new Date()
    }
  });
}

// Get analytics
export async function getModelMetrics(
  model: string,
  since: Date
): Promise<ModelMetrics> {
  const records = await db.metrics.findMany({
    where: {
      model,
      timestamp: { gte: since }
    }
  });

  return {
    model,
    avgResponseTime: average(records.map(r => r.responseTime)),
    avgTokens: average(records.map(r => r.tokens)),
    avgCost: average(records.map(r => r.cost)),
    successRate: records.filter(r => r.success).length / records.length,
    totalRequests: records.length
  };
}
```

---

## Resources

- **OpenRouter Dashboard:** https://openrouter.ai/dashboard
- **API Documentation:** https://openrouter.ai/docs
- **Model Pricing:** https://openrouter.ai/models
- **OpenAI SDK Docs:** https://github.com/openai/openai-node
- **Community Discord:** https://discord.gg/openrouter
