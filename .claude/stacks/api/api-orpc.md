# oRPC - Type-Safe OpenAPI-Compliant RPC Framework

**Purpose:** End-to-end type-safe APIs with native OpenAPI compliance

**Last Updated:** 2025-01-16

**Official Docs:** https://orpc.unnoq.com

---

## What is oRPC?

oRPC is a modern RPC framework that combines:
- **Full Type Safety** - End-to-end TypeScript with automatic client inference
- **OpenAPI Native** - Auto-generates OpenAPI specs without plugins
- **Multi-Runtime** - Works on Node.js, Bun, Deno, Cloudflare Workers
- **Contract-First** - Optional API contract definition before implementation
- **Streaming Built-In** - Type-safe SSE and event streams
- **Framework Agnostic** - Integrates with Next.js, Express, Fastify, Hono, and more

---

## When to Use oRPC

### ✅ Use oRPC When:
- Building **public APIs** that need OpenAPI documentation
- Need **type safety** across client-server boundaries
- Want **contract-first development** for API design
- Building **edge/serverless** functions
- Need **streaming/SSE** with type safety
- Working with **multiple runtimes** (Deno, Bun, Workers)
- Integrating with **external consumers** who need OpenAPI specs

### ❌ Don't Use oRPC When:
- Internal-only APIs where tRPC is simpler
- Non-TypeScript clients (use REST)
- Team unfamiliar with RPC patterns

### oRPC vs Alternatives

| Feature | oRPC | tRPC | REST |
|---------|------|------|------|
| Type Safety | ✅ Full | ✅ Full | ❌ None |
| OpenAPI | ✅ Native | ⚠️ Plugin | ✅ Manual |
| Client Inference | ✅ Auto | ✅ Auto | ❌ None |
| Contract-First | ✅ Yes | ❌ No | ✅ Yes |
| Streaming | ✅ Native | ✅ Yes | ⚠️ Manual |
| Edge Ready | ✅ Yes | ✅ Yes | ✅ Yes |

---

## Installation

```bash
npm install @orpc/server @orpc/client
```

**With Next.js:**
```bash
npm install @orpc/server @orpc/client @orpc/next
```

**With validation (Zod recommended):**
```bash
npm install zod
```

---

## Project Structure

```
src/
├── server/
│   ├── context.ts          # Shared context (auth, db, etc.)
│   ├── procedures/          # Individual procedures
│   │   ├── user.ts
│   │   ├── post.ts
│   │   └── auth.ts
│   ├── middleware/          # Middleware (auth, logging, etc.)
│   │   ├── auth.ts
│   │   └── logger.ts
│   ├── router.ts           # Main router combining all procedures
│   └── client.ts           # Type-safe client export
├── app/
│   └── api/
│       └── rpc/
│           └── [...orpc]/
│               └── route.ts  # Next.js API route handler
```

---

## Core Concepts

### 1. Procedure Definition

Procedures are the fundamental building blocks:

```typescript
// server/procedures/user.ts
import { os } from '@orpc/server'
import { z } from 'zod'

// Simple procedure
export const hello = os.handler(() => {
  return { message: 'Hello World' }
})

// With input validation
export const getUser = os
  .input(z.object({
    id: z.number().int().positive()
  }))
  .handler(async ({ input }) => {
    const user = await db.user.findUnique({
      where: { id: input.id }
    })

    if (!user) {
      throw new ORPCError({
        code: 'NOT_FOUND',
        message: 'User not found'
      })
    }

    return user
  })

// With input AND output validation
export const createUser = os
  .input(z.object({
    email: z.string().email(),
    name: z.string().min(2)
  }))
  .output(z.object({
    id: z.number(),
    email: z.string(),
    name: z.string()
  }))
  .handler(async ({ input }) => {
    return await db.user.create({
      data: input
    })
  })
```

---

### 2. Context & Middleware

**Define Shared Context:**

```typescript
// server/context.ts
import { os } from '@orpc/server'

export interface Context {
  headers: Headers
  db: PrismaClient
  user?: {
    id: number
    email: string
    role: string
  }
}

// Create base procedure with context
export const base = os.$context<Context>()
```

**Authentication Middleware:**

```typescript
// server/middleware/auth.ts
import { ORPCError } from '@orpc/server'

export const authMiddleware = base.use(async ({ context, next }) => {
  const token = context.headers.get('authorization')?.split(' ')[1]

  if (!token) {
    throw new ORPCError({
      code: 'UNAUTHORIZED',
      message: 'Missing authentication token'
    })
  }

  try {
    const user = await verifyJWT(token)

    return next({
      context: {
        ...context,
        user
      }
    })
  } catch (error) {
    throw new ORPCError({
      code: 'UNAUTHORIZED',
      message: 'Invalid token'
    })
  }
})

// Use in procedures
export const protectedProcedure = authMiddleware
```

**Logging Middleware:**

```typescript
// server/middleware/logger.ts
export const loggerMiddleware = base.use(async ({ path, next }) => {
  const start = Date.now()
  console.log(`[RPC] ${path.join('.')} - Started`)

  try {
    const result = await next()
    const duration = Date.now() - start
    console.log(`[RPC] ${path.join('.')} - Completed in ${duration}ms`)
    return result
  } catch (error) {
    const duration = Date.now() - start
    console.error(`[RPC] ${path.join('.')} - Failed in ${duration}ms`, error)
    throw error
  }
})
```

**Composing Middleware:**

```typescript
// Stack multiple middleware
export const authedProcedure = base
  .use(loggerMiddleware)
  .use(authMiddleware)
  .use(dbProviderMiddleware)
```

---

### 3. Router Organization

**Create Modular Routers:**

```typescript
// server/procedures/user.ts
export const userRouter = {
  get: getUser,
  create: createUser,
  update: updateUser,
  delete: deleteUser,
  list: listUsers
}

// server/procedures/post.ts
export const postRouter = {
  get: getPost,
  create: createPost,
  list: listPosts
}

// server/router.ts
export const appRouter = {
  user: userRouter,
  post: postRouter,
  auth: authRouter
}

// Export type for client
export type AppRouter = typeof appRouter
```

**Nested Routers:**

```typescript
export const appRouter = {
  user: {
    profile: {
      get: getUserProfile,
      update: updateUserProfile
    },
    settings: {
      get: getUserSettings,
      update: updateUserSettings
    }
  },
  admin: {
    users: adminUserRouter,
    analytics: adminAnalyticsRouter
  }
}
```

---

### 4. Error Handling

```typescript
import { ORPCError } from '@orpc/server'

// Standard error codes
export const getUserById = os
  .input(z.object({ id: z.number() }))
  .handler(async ({ input }) => {
    const user = await db.user.findUnique({
      where: { id: input.id }
    })

    if (!user) {
      throw new ORPCError({
        code: 'NOT_FOUND',
        message: `User ${input.id} not found`
      })
    }

    return user
  })

// Custom error with data
export const validateEmail = os
  .input(z.object({ email: z.string() }))
  .handler(async ({ input }) => {
    const existing = await db.user.findUnique({
      where: { email: input.email }
    })

    if (existing) {
      throw new ORPCError({
        code: 'CONFLICT',
        message: 'Email already registered',
        data: {
          field: 'email',
          suggestion: 'Try logging in instead'
        }
      })
    }

    return { available: true }
  })

// Error codes available:
// - UNAUTHORIZED
// - FORBIDDEN
// - NOT_FOUND
// - CONFLICT
// - BAD_REQUEST
// - INTERNAL_SERVER_ERROR
// - TIMEOUT
```

---

## Next.js Integration

### App Router Setup

**1. Create API Route:**

```typescript
// app/api/rpc/[...orpc]/route.ts
import { RPCHandler } from '@orpc/next/app'
import { appRouter } from '@/server/router'
import { db } from '@/lib/db'

const handler = RPCHandler({
  router: appRouter,
  context: async (request) => {
    return {
      headers: request.headers,
      db
    }
  }
})

export const POST = handler
export const GET = handler // For SSE/streaming
```

**2. Create Type-Safe Client:**

```typescript
// server/client.ts
import { createORPCClient } from '@orpc/client'
import { RPCLink } from '@orpc/client'
import type { AppRouter } from './router'

const link = new RPCLink<AppRouter>({
  url: `${process.env.NEXT_PUBLIC_APP_URL}/api/rpc`,
  headers: async () => {
    // Add auth headers if needed
    const token = await getToken()
    return {
      authorization: token ? `Bearer ${token}` : ''
    }
  }
})

export const orpc = createORPCClient(link)
```

**3. Use in Components:**

```typescript
// app/users/page.tsx
import { orpc } from '@/server/client'

export default async function UsersPage() {
  // Server Component - direct await
  const users = await orpc.user.list()

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  )
}
```

**4. Client Component Usage:**

```typescript
'use client'

import { orpc } from '@/server/client'
import { useState } from 'react'

export function CreateUserForm() {
  const [loading, setLoading] = useState(false)

  async function handleSubmit(data: { email: string; name: string }) {
    setLoading(true)
    try {
      const user = await orpc.user.create(data)
      console.log('Created:', user)
    } catch (error) {
      if (error.code === 'CONFLICT') {
        alert('Email already exists')
      }
    } finally {
      setLoading(false)
    }
  }

  // Form JSX...
}
```

---

## Advanced Patterns

### 1. Streaming with Server-Sent Events

```typescript
// server/procedures/stream.ts
import { os } from '@orpc/server'

export const streamNumbers = os
  .input(z.object({ count: z.number() }))
  .handler(async function* ({ input }) {
    for (let i = 0; i < input.count; i++) {
      await new Promise(resolve => setTimeout(resolve, 1000))
      yield { number: i, timestamp: Date.now() }
    }
  })

// Client usage
const stream = await orpc.stream.numbers({ count: 10 })

for await (const data of stream) {
  console.log(data.number) // 0, 1, 2, ...
}
```

### 2. File Upload/Download

```typescript
// Upload
export const uploadFile = os
  .input(z.object({
    file: z.instanceof(File)
  }))
  .handler(async ({ input }) => {
    const buffer = await input.file.arrayBuffer()
    const url = await uploadToS3(buffer, input.file.name)
    return { url }
  })

// Client
const file = document.querySelector('input[type="file"]').files[0]
const result = await orpc.file.upload({ file })
```

### 3. Pagination Pattern

```typescript
export const listPosts = os
  .input(z.object({
    cursor: z.number().optional(),
    limit: z.number().int().min(1).max(100).default(20)
  }))
  .handler(async ({ input }) => {
    const posts = await db.post.findMany({
      take: input.limit + 1,
      skip: input.cursor || 0,
      orderBy: { createdAt: 'desc' }
    })

    const hasMore = posts.length > input.limit
    const items = hasMore ? posts.slice(0, -1) : posts

    return {
      items,
      nextCursor: hasMore ? input.cursor + input.limit : null
    }
  })
```

### 4. Role-Based Authorization

```typescript
// server/middleware/role.ts
export function requireRole(...roles: string[]) {
  return authedProcedure.use(({ context, next }) => {
    if (!roles.includes(context.user.role)) {
      throw new ORPCError({
        code: 'FORBIDDEN',
        message: `Requires role: ${roles.join(' or ')}`
      })
    }
    return next()
  })
}

// Usage
export const deleteUser = requireRole('admin')
  .input(z.object({ id: z.number() }))
  .handler(async ({ input }) => {
    await db.user.delete({ where: { id: input.id } })
    return { success: true }
  })
```

### 5. Contract-First Development

```typescript
// contracts/user.contract.ts
import { oc } from '@orpc/contract'
import { z } from 'zod'

export const userContract = {
  get: oc
    .input(z.object({ id: z.number() }))
    .output(z.object({
      id: z.number(),
      email: z.string(),
      name: z.string()
    })),

  create: oc
    .input(z.object({
      email: z.string().email(),
      name: z.string()
    }))
    .output(z.object({
      id: z.number(),
      email: z.string(),
      name: z.string()
    }))
}

// Implementation
export const userRouter = {
  get: userContract.get.handler(async ({ input }) => {
    // Implementation must match contract
  }),

  create: userContract.create.handler(async ({ input }) => {
    // TypeScript ensures compliance
  })
}
```

---

## TanStack Query Integration

```typescript
// hooks/use-orpc.ts
import { createORPCReact } from '@orpc/react-query'
import { orpc } from '@/server/client'

export const { useQuery, useMutation } = createORPCReact(orpc)
```

**Usage:**

```typescript
'use client'

import { useQuery, useMutation } from '@/hooks/use-orpc'

export function UserList() {
  const { data: users, isLoading } = useQuery(['user.list'])

  const createUser = useMutation(['user.create'], {
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries(['user.list'])
    }
  })

  if (isLoading) return <div>Loading...</div>

  return (
    <div>
      {users?.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => createUser.mutate({ name: 'New User', email: 'new@example.com' })}>
        Add User
      </button>
    </div>
  )
}
```

---

## OpenAPI Generation

```typescript
// scripts/generate-openapi.ts
import { generateOpenAPI } from '@orpc/openapi'
import { appRouter } from '@/server/router'
import fs from 'fs'

const spec = generateOpenAPI({
  router: appRouter,
  info: {
    title: 'My API',
    version: '1.0.0',
    description: 'Auto-generated from oRPC'
  },
  servers: [
    { url: 'https://api.example.com', description: 'Production' },
    { url: 'http://localhost:3000', description: 'Development' }
  ]
})

fs.writeFileSync('openapi.json', JSON.stringify(spec, null, 2))
console.log('OpenAPI spec generated!')
```

**Add to package.json:**

```json
{
  "scripts": {
    "openapi": "tsx scripts/generate-openapi.ts"
  }
}
```

---

## Testing

```typescript
// __tests__/user.test.ts
import { appRouter } from '@/server/router'
import { db } from '@/lib/db'

describe('User Procedures', () => {
  it('should create user', async () => {
    const result = await appRouter.user.create({
      input: {
        email: 'test@example.com',
        name: 'Test User'
      },
      context: {
        headers: new Headers(),
        db
      }
    })

    expect(result.email).toBe('test@example.com')
  })

  it('should throw on duplicate email', async () => {
    await expect(
      appRouter.user.create({
        input: {
          email: 'existing@example.com',
          name: 'Test'
        },
        context: { headers: new Headers(), db }
      })
    ).rejects.toThrow('Email already registered')
  })
})
```

---

## Best Practices

### ✅ DO:

1. **Use Zod for Validation**
   ```typescript
   .input(z.object({
     email: z.string().email(),
     age: z.number().int().min(18)
   }))
   ```

2. **Organize by Domain**
   ```
   procedures/
   ├── user/
   ├── post/
   ├── auth/
   └── admin/
   ```

3. **Create Reusable Middleware**
   ```typescript
   export const authedProcedure = base.use(authMiddleware)
   export const adminProcedure = authedProcedure.use(requireRole('admin'))
   ```

4. **Use Type Inference**
   ```typescript
   type User = Infer<typeof userSchema>
   ```

5. **Handle Errors Explicitly**
   ```typescript
   if (!user) {
     throw new ORPCError({ code: 'NOT_FOUND', message: 'User not found' })
   }
   ```

6. **Generate OpenAPI Docs**
   ```bash
   npm run openapi
   ```

### ❌ DON'T:

1. **Don't Skip Input Validation**
   ```typescript
   // ❌ BAD
   .handler(async ({ input }) => {
     return db.user.find(input.id) // No validation!
   })

   // ✅ GOOD
   .input(z.object({ id: z.number() }))
   .handler(async ({ input }) => {
     return db.user.find(input.id)
   })
   ```

2. **Don't Expose Internal Errors**
   ```typescript
   // ❌ BAD
   throw error // Leaks stack traces

   // ✅ GOOD
   throw new ORPCError({
     code: 'INTERNAL_SERVER_ERROR',
     message: 'An error occurred'
   })
   ```

3. **Don't Forget Auth Checks**
   ```typescript
   // ❌ BAD
   export const deleteUser = base.handler(...)

   // ✅ GOOD
   export const deleteUser = authedProcedure.handler(...)
   ```

4. **Don't Create Giant Routers**
   ```typescript
   // ❌ BAD - everything in one file

   // ✅ GOOD - modular by domain
   ```

---

## Common Patterns

### Database Transaction

```typescript
export const transferFunds = authedProcedure
  .input(z.object({
    fromAccountId: z.number(),
    toAccountId: z.number(),
    amount: z.number().positive()
  }))
  .handler(async ({ input, context }) => {
    return await context.db.$transaction(async (tx) => {
      const from = await tx.account.update({
        where: { id: input.fromAccountId },
        data: { balance: { decrement: input.amount } }
      })

      const to = await tx.account.update({
        where: { id: input.toAccountId },
        data: { balance: { increment: input.amount } }
      })

      return { from, to }
    })
  })
```

### Background Job Trigger

```typescript
export const processVideo = authedProcedure
  .input(z.object({ videoId: z.number() }))
  .handler(async ({ input }) => {
    // Queue background job
    await queue.add('video-processing', {
      videoId: input.videoId
    })

    return { status: 'queued' }
  })
```

### Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit'

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 m')
})

export const rateLimitedProcedure = base.use(async ({ context, next }) => {
  const ip = context.headers.get('x-forwarded-for') || 'unknown'
  const { success } = await ratelimit.limit(ip)

  if (!success) {
    throw new ORPCError({
      code: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded'
    })
  }

  return next()
})
```

---

## Troubleshooting

### Issue: "Type 'unknown' is not assignable"

**Cause:** Missing context type definition

**Solution:**
```typescript
// Define context interface
export const base = os.$context<{ db: PrismaClient }>()
```

### Issue: "Cannot find module '@orpc/server'"

**Cause:** Missing installation

**Solution:**
```bash
npm install @orpc/server @orpc/client
```

### Issue: Client shows type errors

**Cause:** Router type not exported

**Solution:**
```typescript
// server/router.ts
export type AppRouter = typeof appRouter

// server/client.ts
import type { AppRouter } from './router'
const link = new RPCLink<AppRouter>({ ... })
```

### Issue: OpenAPI generation fails

**Cause:** Missing metadata or invalid schemas

**Solution:**
```typescript
// Add metadata to procedures
export const getUser = os
  .input(z.object({ id: z.number() }))
  .meta({ description: 'Get user by ID' })
  .handler(...)
```

---

## Migration Guide

### From tRPC to oRPC

**tRPC:**
```typescript
const t = initTRPC.create()
export const router = t.router({
  getUser: t.procedure
    .input(z.object({ id: z.number() }))
    .query(({ input }) => {
      return db.user.find(input.id)
    })
})
```

**oRPC:**
```typescript
export const getUser = os
  .input(z.object({ id: z.number() }))
  .handler(async ({ input }) => {
    return db.user.find(input.id)
  })

export const router = { getUser }
```

**Key Differences:**
- No `query` vs `mutation` distinction (all are handlers)
- Context defined with `$context<T>()` instead of context function
- Native OpenAPI support (no plugin needed)

---

## Resources

- **Official Docs:** https://orpc.unnoq.com
- **Getting Started:** https://orpc.unnoq.com/docs/getting-started
- **GitHub:** https://github.com/unnoq/orpc
- **Examples:** https://orpc.unnoq.com/learn-and-contribute/mini-orpc
- **OpenAPI Guide:** https://orpc.unnoq.com/docs/openapi/getting-started

---

## Related Stack Modules

- [03-api-server-action.md](../03-api-server-action.md) - General API patterns
- [auth-better-auth.md](../auth/auth-better-auth.md) - Authentication integration
- [database-prisma.md](../database/database-prisma.md) - Database integration
- [database-drizzle.md](../database/database-drizzle.md) - Alternative ORM

---

**Last Updated:** 2025-01-16
**Maintained By:** Claude Memory System
**Status:** Active
