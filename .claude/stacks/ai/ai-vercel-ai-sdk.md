# Vercel AI SDK - Streaming & Tool Calling Patterns

**Purpose:** Comprehensive guide for using Vercel AI SDK for AI-powered features
**Last Updated:** 2025-01-12
**SDK Version:** 4.0+
**Official Docs:** https://sdk.vercel.ai

---

## Overview

The Vercel AI SDK is a TypeScript toolkit for building AI-powered applications with streaming, tool/function calling, and multi-provider support (OpenAI, Anthropic, Google, etc.).

**Key Features:**
- **Streaming UI** - Stream AI responses directly to React components
- **Tool Calling** - Let AI execute functions and use APIs
- **Multi-Provider** - Unified interface for OpenAI, Anthropic, Google, etc.
- **React Hooks** - `useChat`, `useCompletion`, `useAssistant`
- **Server-Side Streaming** - Stream from Server Actions or API routes
- **Type Safety** - Full TypeScript support with Zod schemas

---

## Installation

**📚 Official Documentation**: [Vercel AI SDK - Installation](https://sdk.vercel.ai/docs/getting-started)

Follow the official installation guide for setup instructions specific to your framework (Next.js, React, Vue, Svelte, etc.) and AI provider (OpenAI, Anthropic, Google, etc.).

### Quick Reference

The general setup process involves:
1. Install the AI SDK core package
2. Install your chosen AI provider package (OpenAI, Anthropic, Google, etc.)
3. Configure environment variables with API keys
4. Import and configure the provider in your application

**Note**: Package names and installation commands may change. Always refer to the official docs above for the current installation method.

---

## Provider Setup

### Environment Variables

```env
# OpenAI
OPENAI_API_KEY=sk-...

# Anthropic (Claude)
ANTHROPIC_API_KEY=sk-ant-...

# Google (Gemini)
GOOGLE_GENERATIVE_AI_API_KEY=...

# OpenRouter (multiple models)
OPENROUTER_API_KEY=sk-or-...
```

### Provider Configuration

```typescript
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';
import { google } from '@ai-sdk/google';
import { createOpenAI } from '@ai-sdk/openai';

// OpenAI (default)
const model = openai('gpt-4-turbo');

// Anthropic (Claude)
const claudeModel = anthropic('claude-3-5-sonnet-20241022');

// Google (Gemini)
const geminiModel = google('gemini-1.5-pro');

// OpenRouter (multi-provider)
const openrouter = createOpenAI({
  apiKey: process.env.OPENROUTER_API_KEY,
  baseURL: 'https://openrouter.ai/api/v1',
});

const openrouterModel = openrouter('anthropic/claude-3.5-sonnet');
```

---

## Text Generation

### Basic Text Completion

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function generateStory() {
  const { text } = await generateText({
    model: openai('gpt-4-turbo'),
    prompt: 'Write a short story about a robot learning to paint.',
  });

  return text;
}
```

### With System Prompt

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function generateCodeReview(code: string) {
  const { text } = await generateText({
    model: openai('gpt-4-turbo'),
    system: 'You are an expert code reviewer. Provide constructive feedback.',
    prompt: `Review this code:\n\n${code}`,
    temperature: 0.3, // Lower = more focused
    maxTokens: 1000,
  });

  return text;
}
```

### Streaming Text Generation

```typescript
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function streamStory() {
  const result = await streamText({
    model: openai('gpt-4-turbo'),
    prompt: 'Write a story about a space explorer.',
  });

  // Stream to console
  for await (const chunk of result.textStream) {
    process.stdout.write(chunk);
  }
}
```

---

## useChat Hook (Chat Interfaces)

### Basic Chat Component

```typescript
'use client';

import { useChat } from 'ai/react';

export function ChatInterface() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat',
  });

  return (
    <div className="flex flex-col h-screen max-w-2xl mx-auto p-4">
      {/* Messages */}
      <div className="flex-1 overflow-y-auto mb-4 space-y-4">
        {messages.map((message) => (
          <div
            key={message.id}
            className={`p-4 rounded-lg ${
              message.role === 'user'
                ? 'bg-blue-100 ml-auto max-w-[80%]'
                : 'bg-gray-100 mr-auto max-w-[80%]'
            }`}
          >
            <p className="font-semibold">
              {message.role === 'user' ? 'You' : 'AI'}
            </p>
            <p className="whitespace-pre-wrap">{message.content}</p>
          </div>
        ))}

        {isLoading && (
          <div className="bg-gray-100 p-4 rounded-lg mr-auto max-w-[80%]">
            <p className="animate-pulse">AI is thinking...</p>
          </div>
        )}
      </div>

      {/* Input Form */}
      <form onSubmit={handleSubmit} className="flex gap-2">
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Ask me anything..."
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
      </form>
    </div>
  );
}
```

### API Route for Chat

**Next.js App Router (app/api/chat/route.ts):**

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export const runtime = 'edge'; // Optional: Use Edge Runtime

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4-turbo'),
    system: 'You are a helpful assistant.',
    messages,
  });

  return result.toDataStreamResponse();
}
```

### Advanced useChat Options

```typescript
'use client';

import { useChat } from 'ai/react';

export function AdvancedChat() {
  const {
    messages,
    input,
    handleInputChange,
    handleSubmit,
    isLoading,
    error,
    reload,
    stop,
    setMessages,
  } = useChat({
    api: '/api/chat',
    id: 'chat-session-1', // Persist conversations
    initialMessages: [
      {
        id: '1',
        role: 'system',
        content: 'You are a friendly assistant.',
      },
    ],
    body: {
      userId: 'user-123', // Send additional data
      context: 'customer-support',
    },
    onResponse: (response) => {
      console.log('Response received:', response.status);
    },
    onFinish: (message) => {
      console.log('AI finished:', message);
      // Save to database, analytics, etc.
    },
    onError: (error) => {
      console.error('Chat error:', error);
    },
  });

  return (
    <div>
      {/* Messages display */}
      <div>
        {messages.map((m) => (
          <div key={m.id}>
            <strong>{m.role}:</strong> {m.content}
          </div>
        ))}
      </div>

      {/* Error handling */}
      {error && (
        <div className="bg-red-100 p-4 rounded">
          <p>Error: {error.message}</p>
          <button onClick={() => reload()}>Retry</button>
        </div>
      )}

      {/* Input form */}
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading}>
          Send
        </button>
        {isLoading && (
          <button type="button" onClick={() => stop()}>
            Stop
          </button>
        )}
      </form>

      {/* Clear chat */}
      <button onClick={() => setMessages([])}>Clear Chat</button>
    </div>
  );
}
```

---

## useCompletion Hook (Single Prompts)

For single-turn completions (not conversations):

```typescript
'use client';

import { useCompletion } from 'ai/react';

export function TextGenerator() {
  const {
    completion,
    input,
    handleInputChange,
    handleSubmit,
    isLoading,
  } = useCompletion({
    api: '/api/completion',
  });

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <textarea
          value={input}
          onChange={handleInputChange}
          placeholder="Enter a prompt..."
          rows={4}
          className="w-full p-2 border rounded"
        />
        <button type="submit" disabled={isLoading}>
          {isLoading ? 'Generating...' : 'Generate'}
        </button>
      </form>

      {completion && (
        <div className="mt-4 p-4 bg-gray-100 rounded">
          <h3>Generated Text:</h3>
          <p className="whitespace-pre-wrap">{completion}</p>
        </div>
      )}
    </div>
  );
}
```

**API Route (app/api/completion/route.ts):**

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { prompt } = await req.json();

  const result = await streamText({
    model: openai('gpt-4-turbo'),
    prompt,
  });

  return result.toDataStreamResponse();
}
```

---

## Tool Calling (Function Calling)

Let AI execute functions and use tools.

### Server-Side Tool Calling

```typescript
import { generateText, tool } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

export async function weatherAssistant(city: string) {
  const result = await generateText({
    model: openai('gpt-4-turbo'),
    system: 'You are a helpful weather assistant.',
    prompt: `What's the weather in ${city}?`,
    tools: {
      getWeather: tool({
        description: 'Get the current weather for a city',
        parameters: z.object({
          city: z.string().describe('The city name'),
          unit: z.enum(['celsius', 'fahrenheit']).optional(),
        }),
        execute: async ({ city, unit = 'celsius' }) => {
          // Call actual weather API
          const weatherData = await fetchWeatherAPI(city, unit);
          return weatherData;
        },
      }),

      getTemperatureAlert: tool({
        description: 'Check if temperature is above/below threshold',
        parameters: z.object({
          temperature: z.number(),
          threshold: z.number(),
        }),
        execute: async ({ temperature, threshold }) => {
          return {
            isAlert: temperature > threshold,
            message: temperature > threshold
              ? 'Temperature is above threshold!'
              : 'Temperature is normal',
          };
        },
      }),
    },
    maxToolRoundtrips: 5, // Allow multiple tool calls
  });

  return result.text;
}

async function fetchWeatherAPI(city: string, unit: string) {
  // Mock implementation - replace with real API
  return {
    city,
    temperature: 22,
    condition: 'Sunny',
    unit,
  };
}
```

### Client-Side Tool Calling with useChat

```typescript
'use client';

import { useChat } from 'ai/react';

export function ToolChat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat({
    api: '/api/chat-with-tools',
  });

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id}>
          <strong>{m.role}:</strong>

          {/* Display text content */}
          {m.content}

          {/* Display tool calls */}
          {m.toolInvocations?.map((tool, index) => (
            <div key={index} className="bg-blue-50 p-2 rounded mt-2">
              <p className="font-mono text-sm">
                🔧 Called: {tool.toolName}
              </p>
              <p className="text-xs">
                Args: {JSON.stringify(tool.args)}
              </p>
              {tool.result && (
                <p className="text-xs mt-1">
                  Result: {JSON.stringify(tool.result)}
                </p>
              )}
            </div>
          ))}
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

**API Route with Tools (app/api/chat-with-tools/route.ts):**

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText, tool } from 'ai';
import { z } from 'zod';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4-turbo'),
    messages,
    tools: {
      searchDatabase: tool({
        description: 'Search the product database',
        parameters: z.object({
          query: z.string(),
          limit: z.number().optional(),
        }),
        execute: async ({ query, limit = 10 }) => {
          // Call your database
          const results = await searchProducts(query, limit);
          return results;
        },
      }),

      calculatePrice: tool({
        description: 'Calculate total price with tax',
        parameters: z.object({
          price: z.number(),
          quantity: z.number(),
          taxRate: z.number(),
        }),
        execute: async ({ price, quantity, taxRate }) => {
          const subtotal = price * quantity;
          const tax = subtotal * taxRate;
          const total = subtotal + tax;
          return { subtotal, tax, total };
        },
      }),
    },
  });

  return result.toDataStreamResponse();
}

async function searchProducts(query: string, limit: number) {
  // Mock - replace with real DB query
  return [
    { id: 1, name: 'Product A', price: 29.99 },
    { id: 2, name: 'Product B', price: 39.99 },
  ];
}
```

---

## Structured Output (JSON Mode)

Force AI to return valid JSON.

### Using generateObject

```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

export async function extractProductInfo(description: string) {
  const { object } = await generateObject({
    model: openai('gpt-4-turbo'),
    schema: z.object({
      name: z.string(),
      category: z.string(),
      price: z.number(),
      features: z.array(z.string()),
      tags: z.array(z.string()),
    }),
    prompt: `Extract product information from this description: ${description}`,
  });

  return object;
}

// Usage
const product = await extractProductInfo(
  'Brand new wireless headphones with noise cancellation, 30-hour battery life, and premium sound quality. Only $199.'
);

console.log(product);
// {
//   name: "Wireless Headphones",
//   category: "Audio",
//   price: 199,
//   features: ["noise cancellation", "30-hour battery", "premium sound"],
//   tags: ["wireless", "headphones", "audio"]
// }
```

### Streaming Objects

```typescript
import { streamObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

export async function streamProductAnalysis(description: string) {
  const { partialObjectStream } = await streamObject({
    model: openai('gpt-4-turbo'),
    schema: z.object({
      sentiment: z.enum(['positive', 'neutral', 'negative']),
      keyPoints: z.array(z.string()),
      score: z.number().min(0).max(100),
    }),
    prompt: `Analyze this product review: ${description}`,
  });

  for await (const partialObject of partialObjectStream) {
    console.log('Partial result:', partialObject);
    // Update UI with partial data as it arrives
  }
}
```

---

## Server Actions (Next.js)

Use AI SDK in Server Actions for server-side streaming to client components.

### Server Action with Streaming

```typescript
'use server';

import { createStreamableValue } from 'ai/rsc';
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function generateStory(prompt: string) {
  const stream = createStreamableValue('');

  (async () => {
    const { textStream } = await streamText({
      model: openai('gpt-4-turbo'),
      prompt,
    });

    for await (const delta of textStream) {
      stream.update(delta);
    }

    stream.done();
  })();

  return { output: stream.value };
}
```

### Client Component Using Server Action

```typescript
'use client';

import { readStreamableValue } from 'ai/rsc';
import { generateStory } from './actions';
import { useState } from 'react';

export function StoryGenerator() {
  const [story, setStory] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleGenerate = async () => {
    setIsLoading(true);
    setStory('');

    const { output } = await generateStory('Write a sci-fi story');

    for await (const delta of readStreamableValue(output)) {
      setStory((current) => current + delta);
    }

    setIsLoading(false);
  };

  return (
    <div>
      <button onClick={handleGenerate} disabled={isLoading}>
        {isLoading ? 'Generating...' : 'Generate Story'}
      </button>

      {story && (
        <div className="mt-4 whitespace-pre-wrap">
          {story}
        </div>
      )}
    </div>
  );
}
```

---

## Multi-Modal (Vision)

Process images with AI.

### Analyze Image

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function analyzeImage(imageUrl: string) {
  const { text } = await generateText({
    model: openai('gpt-4-vision-preview'),
    messages: [
      {
        role: 'user',
        content: [
          { type: 'text', text: 'What do you see in this image?' },
          { type: 'image', image: imageUrl },
        ],
      },
    ],
  });

  return text;
}

// With base64 image
export async function analyzeBase64Image(base64Image: string) {
  const { text } = await generateText({
    model: openai('gpt-4-vision-preview'),
    messages: [
      {
        role: 'user',
        content: [
          { type: 'text', text: 'Describe this image in detail.' },
          { type: 'image', image: new URL(base64Image) },
        ],
      },
    ],
  });

  return text;
}
```

---

## Error Handling

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function safeGenerate(prompt: string) {
  try {
    const { text } = await generateText({
      model: openai('gpt-4-turbo'),
      prompt,
    });

    return { success: true, text };
  } catch (error) {
    console.error('AI generation error:', error);

    // Handle specific errors
    if (error instanceof Error) {
      if (error.message.includes('rate limit')) {
        return { success: false, error: 'Rate limit exceeded. Please try again later.' };
      }

      if (error.message.includes('invalid API key')) {
        return { success: false, error: 'Authentication failed.' };
      }
    }

    return { success: false, error: 'Failed to generate text. Please try again.' };
  }
}
```

---

## Token Usage Tracking

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function generateWithTracking(prompt: string) {
  const result = await generateText({
    model: openai('gpt-4-turbo'),
    prompt,
  });

  console.log('Token usage:', result.usage);
  // {
  //   promptTokens: 50,
  //   completionTokens: 200,
  //   totalTokens: 250
  // }

  // Calculate cost (example rates)
  const inputCost = (result.usage.promptTokens / 1000) * 0.01; // $0.01 per 1K tokens
  const outputCost = (result.usage.completionTokens / 1000) * 0.03; // $0.03 per 1K tokens
  const totalCost = inputCost + outputCost;

  console.log(`Cost: $${totalCost.toFixed(4)}`);

  return result.text;
}
```

---

## Best Practices

### 1. System Prompts for Consistency

```typescript
const SYSTEM_PROMPTS = {
  codeReviewer: `You are an expert code reviewer. Provide constructive, actionable feedback.
Focus on:
- Code quality and best practices
- Potential bugs or security issues
- Performance improvements
- Maintainability

Format your response with clear sections.`,

  copywriter: `You are a professional copywriter. Write compelling, conversion-focused copy.
Follow these principles:
- Clear and concise language
- Focus on benefits, not features
- Use active voice
- Include a strong call-to-action`,
};

// Usage
const result = await generateText({
  model: openai('gpt-4-turbo'),
  system: SYSTEM_PROMPTS.codeReviewer,
  prompt: userCode,
});
```

### 2. Temperature Control

```typescript
// Low temperature (0.0-0.3): Deterministic, factual
const factual = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: 'What is the capital of France?',
  temperature: 0.1,
});

// Medium temperature (0.4-0.7): Balanced
const balanced = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: 'Write a product description',
  temperature: 0.5,
});

// High temperature (0.8-1.0): Creative, varied
const creative = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: 'Write a creative story',
  temperature: 0.9,
});
```

### 3. Max Tokens for Cost Control

```typescript
const result = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: 'Summarize this article...',
  maxTokens: 500, // Limit response length
});
```

### 4. Retry Logic

```typescript
async function generateWithRetry(
  prompt: string,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const { text } = await generateText({
        model: openai('gpt-4-turbo'),
        prompt,
      });
      return text;
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }

  throw new Error('Max retries exceeded');
}
```

---

## Testing

### Mock AI Responses

```typescript
// Mock for testing
jest.mock('ai', () => ({
  generateText: jest.fn(),
}));

import { generateText } from 'ai';

// In test
(generateText as jest.Mock).mockResolvedValue({
  text: 'Mocked AI response',
  usage: { totalTokens: 100 },
});

const result = await myAIFunction();
expect(result).toBe('Mocked AI response');
```

---

## Common Patterns

### AI-Powered Search

```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

export async function intelligentSearch(query: string) {
  // Extract search intent and parameters
  const { object } = await generateObject({
    model: openai('gpt-4-turbo'),
    schema: z.object({
      intent: z.enum(['product', 'information', 'support']),
      keywords: z.array(z.string()),
      filters: z.object({
        category: z.string().optional(),
        priceRange: z.object({
          min: z.number().optional(),
          max: z.number().optional(),
        }).optional(),
      }),
    }),
    prompt: `Analyze this search query and extract structured information: "${query}"`,
  });

  // Use extracted data to search database
  const results = await searchDatabase(object);
  return results;
}
```

### Content Generation Pipeline

```typescript
export async function generateBlogPost(topic: string) {
  // Step 1: Generate outline
  const outline = await generateText({
    model: openai('gpt-4-turbo'),
    prompt: `Create a detailed outline for a blog post about: ${topic}`,
  });

  // Step 2: Generate full content
  const { textStream } = await streamText({
    model: openai('gpt-4-turbo'),
    prompt: `Write a complete blog post following this outline:\n\n${outline.text}`,
  });

  // Step 3: Stream to user
  return textStream;
}
```

---

## Resources

- **Official Docs:** https://sdk.vercel.ai/docs
- **Examples:** https://github.com/vercel/ai/tree/main/examples
- **API Reference:** https://sdk.vercel.ai/docs/reference
- **Community Discord:** https://vercel.com/discord

---

*This guide covers Vercel AI SDK v4.0+. For migration guides or advanced features, consult the official documentation.*
