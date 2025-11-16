# Convex Backend Platform Integration Guide

## Overview

Convex is a complete backend platform that combines database, API, authentication, and real-time synchronization in a single TypeScript-first system. It eliminates the need for separate state management, cache invalidation, and WebSocket configuration.

**Key Features:**
- **Real-Time Sync**: Automatic updates to all connected clients
- **TypeScript-First**: End-to-end type safety from database to frontend
- **Document Database**: NoSQL with relational schema support
- **Backend Functions**: Queries, mutations, and actions
- **Built-in Auth**: 80+ OAuth integrations
- **File Storage**: Integrated file management
- **Scheduled Jobs**: Cron jobs and background tasks
- **Serverless**: Auto-scaling, zero-config deployment

**When to Use:**
- Building real-time collaborative apps
- Need instant data synchronization
- Want end-to-end TypeScript type safety
- Rapid development without backend complexity
- Apps requiring reactive UI updates

## Installation

**📚 Official Documentation**: [Convex - Quickstart](https://docs.convex.dev/quickstart)

Follow the official quickstart guide for framework-specific setup instructions (React, Next.js, Remix, Vue, Node.js, etc.).

**Alternative**: [Convex Chef](https://chef.convex.dev) - Interactive tool to initialize projects with your preferred framework.

### Quick Reference

**For New Projects:**
The CLI creates a project with templates (blank, with-auth, with-file-storage).

**For Existing Projects:**
The general setup process involves:
1. Install Convex package
2. Initialize Convex with CLI
3. Create account/project
4. Set up schema and functions
5. Connect frontend

**Note**: Commands and templates may change. Always refer to the official docs above for the current installation method.

**Project Structure:**
```
my-app/
├── convex/
│   ├── _generated/         # Auto-generated types
│   ├── schema.ts           # Database schema
│   ├── messages.ts         # Queries and mutations
│   ├── auth.config.ts      # Authentication config
│   └── crons.ts            # Scheduled jobs
├── src/
│   ├── components/
│   └── main.tsx
├── convex.json             # Project config
└── package.json
```

## Schema Definition

**convex/schema.ts**
```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // Messages table
  messages: defineTable({
    text: v.string(),
    userId: v.id("users"),
    channelId: v.id("channels"),
    createdAt: v.number(),
    reactions: v.optional(v.array(v.object({
      emoji: v.string(),
      userId: v.id("users"),
    }))),
  })
    .index("by_channel", ["channelId", "createdAt"])
    .index("by_user", ["userId"]),

  // Users table
  users: defineTable({
    name: v.string(),
    email: v.string(),
    avatar: v.optional(v.string()),
    role: v.union(v.literal("admin"), v.literal("user")),
    createdAt: v.number(),
  })
    .index("by_email", ["email"]),

  // Channels table
  channels: defineTable({
    name: v.string(),
    description: v.optional(v.string()),
    isPrivate: v.boolean(),
    memberIds: v.array(v.id("users")),
    createdAt: v.number(),
  })
    .index("by_name", ["name"]),

  // Tasks table
  tasks: defineTable({
    title: v.string(),
    description: v.optional(v.string()),
    status: v.union(
      v.literal("todo"),
      v.literal("in_progress"),
      v.literal("done")
    ),
    assigneeId: v.optional(v.id("users")),
    dueDate: v.optional(v.number()),
    tags: v.array(v.string()),
    createdAt: v.number(),
    updatedAt: v.number(),
  })
    .index("by_status", ["status"])
    .index("by_assignee", ["assigneeId"])
    .index("by_due_date", ["dueDate"]),
});
```

**Schema Features:**
- `v.string()`, `v.number()`, `v.boolean()` - Primitive types
- `v.id("tableName")` - References to other tables
- `v.optional()` - Optional fields
- `v.array()` - Arrays
- `v.object()` - Nested objects
- `v.union()` - Union types (like TypeScript)
- `.index()` - Query optimization

## Queries (Read Data)

**convex/messages.ts**
```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

// Get all messages in a channel
export const list = query({
  args: { channelId: v.id("channels") },
  handler: async (ctx, args) => {
    const messages = await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(100);

    // Populate user data
    return await Promise.all(
      messages.map(async (message) => {
        const user = await ctx.db.get(message.userId);
        return {
          ...message,
          user,
        };
      })
    );
  },
});

// Get single message
export const get = query({
  args: { id: v.id("messages") },
  handler: async (ctx, args) => {
    return await ctx.db.get(args.id);
  },
});

// Search messages
export const search = query({
  args: {
    channelId: v.id("channels"),
    searchTerm: v.string(),
  },
  handler: async (ctx, args) => {
    const messages = await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .collect();

    // Filter by search term
    return messages.filter((msg) =>
      msg.text.toLowerCase().includes(args.searchTerm.toLowerCase())
    );
  },
});

// Paginated query
export const paginated = query({
  args: {
    channelId: v.id("channels"),
    paginationOpts: v.object({
      numItems: v.number(),
      cursor: v.union(v.string(), v.null()),
    }),
  },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
```

## Mutations (Write Data)

**convex/messages.ts**
```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

// Create message
export const create = mutation({
  args: {
    text: v.string(),
    channelId: v.id("channels"),
  },
  handler: async (ctx, args) => {
    // Get current user (from auth)
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Not authenticated");
    }

    // Find user by tokenIdentifier
    const user = await ctx.db
      .query("users")
      .withIndex("by_token_identifier", (q) =>
        q.eq("tokenIdentifier", identity.tokenIdentifier)
      )
      .unique();

    if (!user) {
      throw new Error("User not found");
    }

    // Insert message
    const messageId = await ctx.db.insert("messages", {
      text: args.text,
      userId: user._id,
      channelId: args.channelId,
      createdAt: Date.now(),
    });

    return messageId;
  },
});

// Update message
export const update = mutation({
  args: {
    id: v.id("messages"),
    text: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Not authenticated");
    }

    const message = await ctx.db.get(args.id);
    if (!message) {
      throw new Error("Message not found");
    }

    // Check ownership
    const user = await ctx.db
      .query("users")
      .withIndex("by_token_identifier", (q) =>
        q.eq("tokenIdentifier", identity.tokenIdentifier)
      )
      .unique();

    if (message.userId !== user?._id) {
      throw new Error("Not authorized");
    }

    await ctx.db.patch(args.id, {
      text: args.text,
    });
  },
});

// Delete message
export const remove = mutation({
  args: { id: v.id("messages") },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Not authenticated");
    }

    const message = await ctx.db.get(args.id);
    if (!message) {
      throw new Error("Message not found");
    }

    await ctx.db.delete(args.id);
  },
});

// Add reaction
export const addReaction = mutation({
  args: {
    messageId: v.id("messages"),
    emoji: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Not authenticated");
    }

    const user = await ctx.db
      .query("users")
      .withIndex("by_token_identifier", (q) =>
        q.eq("tokenIdentifier", identity.tokenIdentifier)
      )
      .unique();

    if (!user) {
      throw new Error("User not found");
    }

    const message = await ctx.db.get(args.messageId);
    if (!message) {
      throw new Error("Message not found");
    }

    const reactions = message.reactions || [];
    reactions.push({
      emoji: args.emoji,
      userId: user._id,
    });

    await ctx.db.patch(args.messageId, { reactions });
  },
});
```

## Actions (External APIs & Side Effects)

**convex/actions.ts**
```typescript
import { action } from "./_generated/server";
import { v } from "convex/values";
import { api } from "./_generated/api";

// Send email via external API
export const sendEmail = action({
  args: {
    to: v.string(),
    subject: v.string(),
    body: v.string(),
  },
  handler: async (ctx, args) => {
    // Call external API
    const response = await fetch("https://api.sendgrid.com/v3/mail/send", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${process.env.SENDGRID_API_KEY}`,
      },
      body: JSON.stringify({
        personalizations: [{ to: [{ email: args.to }] }],
        from: { email: "noreply@example.com" },
        subject: args.subject,
        content: [{ type: "text/plain", value: args.body }],
      }),
    });

    if (!response.ok) {
      throw new Error("Failed to send email");
    }

    return { success: true };
  },
});

// AI-powered categorization
export const categorizeTask = action({
  args: { taskId: v.id("tasks") },
  handler: async (ctx, args) => {
    // Get task
    const task = await ctx.runQuery(api.tasks.get, { id: args.taskId });
    if (!task) {
      throw new Error("Task not found");
    }

    // Call AI API
    const response = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": process.env.ANTHROPIC_API_KEY!,
        "anthropic-version": "2023-06-01",
      },
      body: JSON.stringify({
        model: "claude-3-5-sonnet-20241022",
        max_tokens: 1024,
        messages: [
          {
            role: "user",
            content: `Categorize this task: "${task.title}". Return only one word: urgent, important, or routine.`,
          },
        ],
      }),
    });

    const data = await response.json();
    const category = data.content[0].text.toLowerCase();

    // Update task with category
    await ctx.runMutation(api.tasks.update, {
      id: args.taskId,
      tags: [...task.tags, category],
    });

    return { category };
  },
});

// Process webhook
export const handleWebhook = action({
  args: {
    event: v.string(),
    data: v.any(),
  },
  handler: async (ctx, args) => {
    console.log("Webhook received:", args.event);

    if (args.event === "user.created") {
      // Create user in database
      await ctx.runMutation(api.users.create, {
        name: args.data.name,
        email: args.data.email,
      });
    }

    return { processed: true };
  },
});
```

## Frontend Integration

### React Setup

**src/main.tsx**
```typescript
import React from "react";
import ReactDOM from "react-dom/client";
import { ConvexProvider, ConvexReactClient } from "convex/react";
import App from "./App";

const convex = new ConvexReactClient(import.meta.env.VITE_CONVEX_URL as string);

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ConvexProvider client={convex}>
      <App />
    </ConvexProvider>
  </React.StrictMode>
);
```

### Using Queries

```typescript
import { useQuery } from "convex/react";
import { api } from "../convex/_generated/api";

export function MessageList({ channelId }: { channelId: string }) {
  // Automatically subscribes to real-time updates
  const messages = useQuery(api.messages.list, { channelId });

  if (messages === undefined) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      {messages.map((message) => (
        <div key={message._id}>
          <strong>{message.user?.name}:</strong> {message.text}
        </div>
      ))}
    </div>
  );
}
```

### Using Mutations

```typescript
import { useMutation } from "convex/react";
import { api } from "../convex/_generated/api";
import { useState } from "react";

export function MessageForm({ channelId }: { channelId: string }) {
  const [text, setText] = useState("");
  const createMessage = useMutation(api.messages.create);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    await createMessage({ text, channelId });
    setText("");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Type a message..."
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

### Using Actions

```typescript
import { useAction } from "convex/react";
import { api } from "../convex/_generated/api";

export function EmailButton({ to, subject, body }: EmailProps) {
  const sendEmail = useAction(api.actions.sendEmail);
  const [sending, setSending] = useState(false);

  const handleSend = async () => {
    setSending(true);
    try {
      await sendEmail({ to, subject, body });
      alert("Email sent!");
    } catch (error) {
      alert("Failed to send email");
    } finally {
      setSending(false);
    }
  };

  return (
    <button onClick={handleSend} disabled={sending}>
      {sending ? "Sending..." : "Send Email"}
    </button>
  );
}
```

### Pagination

```typescript
import { usePaginatedQuery } from "convex/react";
import { api } from "../convex/_generated/api";

export function PaginatedMessages({ channelId }: { channelId: string }) {
  const { results, status, loadMore } = usePaginatedQuery(
    api.messages.paginated,
    { channelId },
    { initialNumItems: 20 }
  );

  return (
    <div>
      {results.map((message) => (
        <div key={message._id}>{message.text}</div>
      ))}

      {status === "CanLoadMore" && (
        <button onClick={() => loadMore(20)}>Load More</button>
      )}

      {status === "LoadingMore" && <div>Loading...</div>}
    </div>
  );
}
```

## Authentication

**convex/auth.config.ts**
```typescript
export default {
  providers: [
    {
      domain: process.env.CLERK_JWT_ISSUER_DOMAIN!,
      applicationID: "convex",
    },
  ],
};
```

**With Clerk:**
```typescript
import { ClerkProvider, useAuth } from "@clerk/clerk-react";
import { ConvexProviderWithClerk } from "convex/react-clerk";
import { ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(import.meta.env.VITE_CONVEX_URL);

function App() {
  return (
    <ClerkProvider publishableKey={import.meta.env.VITE_CLERK_PUBLISHABLE_KEY}>
      <ConvexProviderWithClerk client={convex} useAuth={useAuth}>
        <YourApp />
      </ConvexProviderWithClerk>
    </ClerkProvider>
  );
}
```

**Check Auth in Backend:**
```typescript
export const createProtected = mutation({
  args: { text: v.string() },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Not authenticated");
    }

    // identity.tokenIdentifier - unique user ID
    // identity.email - user email
    // identity.name - user name

    // Your logic here
  },
});
```

## File Storage

**Upload File:**
```typescript
import { useMutation } from "convex/react";
import { api } from "../convex/_generated/api";

export function FileUpload() {
  const generateUploadUrl = useMutation(api.files.generateUploadUrl);
  const saveFile = useMutation(api.files.save);

  const handleUpload = async (file: File) => {
    // Step 1: Get upload URL
    const uploadUrl = await generateUploadUrl();

    // Step 2: Upload file
    const result = await fetch(uploadUrl, {
      method: "POST",
      headers: { "Content-Type": file.type },
      body: file,
    });

    const { storageId } = await result.json();

    // Step 3: Save reference in database
    await saveFile({
      storageId,
      name: file.name,
      type: file.type,
    });
  };

  return (
    <input
      type="file"
      onChange={(e) => {
        const file = e.target.files?.[0];
        if (file) handleUpload(file);
      }}
    />
  );
}
```

**Backend File Functions:**
```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const generateUploadUrl = mutation({
  handler: async (ctx) => {
    return await ctx.storage.generateUploadUrl();
  },
});

export const save = mutation({
  args: {
    storageId: v.string(),
    name: v.string(),
    type: v.string(),
  },
  handler: async (ctx, args) => {
    await ctx.db.insert("files", {
      storageId: args.storageId,
      name: args.name,
      type: args.type,
      uploadedAt: Date.now(),
    });
  },
});

export const getUrl = mutation({
  args: { storageId: v.string() },
  handler: async (ctx, args) => {
    return await ctx.storage.getUrl(args.storageId);
  },
});
```

## Scheduled Jobs (Crons)

**convex/crons.ts**
```typescript
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

// Run every 5 minutes
crons.interval(
  "categorize pending tasks",
  { minutes: 5 },
  internal.tasks.categorizePending
);

// Run daily at midnight
crons.daily(
  "send daily digest",
  { hourUTC: 0, minuteUTC: 0 },
  internal.emails.sendDailyDigest
);

// Run weekly on Monday at 9am
crons.weekly(
  "weekly report",
  { dayOfWeek: "monday", hourUTC: 9, minuteUTC: 0 },
  internal.reports.generateWeekly
);

// Custom cron expression
crons.cron(
  "complex schedule",
  "0 */2 * * *", // Every 2 hours
  internal.tasks.cleanup
);

export default crons;
```

## Best Practices

### 1. Use Indexes for Queries

```typescript
// Good: Uses index
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .collect();

// Bad: Full table scan
const messages = await ctx.db
  .query("messages")
  .filter((q) => q.eq(q.field("channelId"), channelId))
  .collect();
```

### 2. Validate Arguments

```typescript
export const create = mutation({
  args: {
    text: v.string(),
    email: v.string(), // Basic string
  },
  handler: async (ctx, args) => {
    // Add custom validation
    if (args.text.length > 1000) {
      throw new Error("Text too long");
    }

    if (!args.email.includes("@")) {
      throw new Error("Invalid email");
    }

    // Your logic
  },
});
```

### 3. Use Actions for External APIs

```typescript
// Good: Action for external API
export const sendNotification = action({
  handler: async (ctx, args) => {
    await fetch("https://external-api.com/notify", {
      method: "POST",
      body: JSON.stringify(args),
    });
  },
});

// Bad: Mutation with external API (will fail)
export const sendNotification = mutation({
  handler: async (ctx, args) => {
    // ❌ Can't use fetch in mutations
    await fetch("https://external-api.com/notify");
  },
});
```

### 4. Paginate Large Results

```typescript
// Good: Paginated
export const list = query({
  args: { paginationOpts: paginationOptsValidator },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .order("desc")
      .paginate(args.paginationOpts);
  },
});

// Bad: Load everything
export const list = query({
  handler: async (ctx) => {
    return await ctx.db.query("messages").collect();
  },
});
```

## Environment Variables

**Terminal:**
```bash
# Set environment variable
npx convex env set SENDGRID_API_KEY your_key_here

# Set for production
npx convex env set SENDGRID_API_KEY your_key_here --prod

# List all variables
npx convex env list
```

**Usage in Functions:**
```typescript
const apiKey = process.env.SENDGRID_API_KEY;
```

## Deployment

```bash
# Deploy to production
npx convex deploy

# Deploy functions only
npx convex deploy --functions

# View deployment status
npx convex dashboard
```

## Resources

- **Documentation**: https://docs.convex.dev/
- **GitHub**: https://github.com/get-convex/convex
- **Discord**: https://convex.dev/community
- **Dashboard**: https://dashboard.convex.dev/

## Summary

Convex provides a complete backend platform with:

1. **Real-time sync** - Automatic updates to all clients
2. **TypeScript-first** - End-to-end type safety
3. **Three function types** - Queries (read), mutations (write), actions (external)
4. **Built-in auth** - 80+ OAuth providers
5. **File storage** - Integrated file management
6. **Scheduled jobs** - Cron jobs for automation
7. **Zero config** - Serverless deployment

Perfect for building real-time apps, collaborative tools, and reactive UIs without managing infrastructure or state synchronization.
