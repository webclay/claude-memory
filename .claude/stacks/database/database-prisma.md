# Prisma ORM Integration Guide

## Overview

Prisma is a next-generation TypeScript-first ORM that provides type-safe database access with an intuitive API. It simplifies database workflows with auto-generated queries, migrations, and a visual database browser.

**Key Features:**
- **Type-Safe**: Auto-generated types from your schema
- **Intuitive API**: Simple, readable query syntax
- **Auto-Completion**: Full IntelliSense support
- **Prisma Studio**: Visual database browser
- **Migrations**: Declarative schema migrations
- **Multi-Database**: PostgreSQL, MySQL, SQLite, MongoDB, SQL Server
- **Prisma Accelerate**: Connection pooling and global caching
- **Prisma Pulse**: Real-time database events

**When to Use:**
- Need type-safe database access
- Want excellent TypeScript integration
- Building with multiple database types
- Need visual database management
- Want simplified migrations
- Require connection pooling for serverless

## Installation

**📚 Official Documentation**: [Prisma - Getting Started](https://www.prisma.io/docs/getting-started)

Follow the official quickstart guide for setup instructions specific to your database and framework.

**Framework-Specific Guides:**
- [Next.js](https://www.prisma.io/docs/orm/more/help-and-troubleshooting/help-articles/nextjs-prisma-client-dev-practices)
- [Remix](https://www.prisma.io/docs/orm/more/help-and-troubleshooting/help-articles/remix-prisma-client-dev-practices)
- [SvelteKit, Astro, etc.](https://www.prisma.io/docs/getting-started)

### Quick Reference

The general setup process involves:
1. Install Prisma CLI and Client
2. Initialize Prisma (`npx prisma init`)
3. Configure database connection in `.env`
4. Define schema in `prisma/schema.prisma`
5. Run migrations
6. Generate Prisma Client

**Note**: Installation commands and setup steps may change. Always refer to the official docs above for the current method.

## Schema Definition

**prisma/schema.prisma**
```prisma
// Database configuration
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  directUrl = env("DATABASE_URL_UNPOOLED") // For migrations (optional)
}

// Prisma Client configuration
generator client {
  provider = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "fullTextIndex"]
}

// User model
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  profile   Profile?
  posts     Post[]
  comments  Comment[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
  @@map("users") // Custom table name
}

// Profile model (one-to-one)
model Profile {
  id     Int     @id @default(autoincrement())
  bio    String?
  avatar String?
  userId Int     @unique
  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("profiles")
}

// Post model (one-to-many)
model Post {
  id        Int       @id @default(autoincrement())
  title     String
  content   String?   @db.Text
  published Boolean   @default(false)
  authorId  Int
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments  Comment[]
  tags      Tag[]
  viewCount Int       @default(0)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([authorId])
  @@index([published])
  @@fulltext([title, content]) // Full-text search
  @@map("posts")
}

// Comment model
model Comment {
  id        Int      @id @default(autoincrement())
  content   String
  postId    Int
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([authorId])
  @@map("comments")
}

// Tag model (many-to-many)
model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]

  @@map("tags")
}

// Enum
enum Role {
  USER
  ADMIN
  MODERATOR
}
```

**Field Types:**
- `String`, `Int`, `Float`, `Boolean`, `DateTime`
- `Json` - JSON data
- `Bytes` - Binary data
- `Decimal` - Precise decimals
- `BigInt` - Large integers

**Modifiers:**
- `?` - Optional field
- `[]` - Array
- `@id` - Primary key
- `@unique` - Unique constraint
- `@default()` - Default value
- `@relation` - Relation definition
- `@map()` - Custom column name
- `@@map()` - Custom table name
- `@@index()` - Database index

## Migrations

### Create and Apply Migrations

```bash
# Create migration (development)
npx prisma migrate dev --name add_user_table

# Apply migrations (production)
npx prisma migrate deploy

# Reset database (development only)
npx prisma migrate reset

# Generate Prisma Client
npx prisma generate

# View migration status
npx prisma migrate status
```

### Migration Workflow

```bash
# 1. Update schema.prisma
# Add new field to User model:
# age Int?

# 2. Create migration
npx prisma migrate dev --name add_user_age

# 3. Prisma generates SQL:
# ALTER TABLE "users" ADD COLUMN "age" INTEGER;

# 4. Migration applied + Prisma Client regenerated
```

## Prisma Client Setup

**lib/prisma.ts** (Next.js / Node.js)
```typescript
import { PrismaClient } from '@prisma/client';

const prismaClientSingleton = () => {
  return new PrismaClient({
    log: process.env.NODE_ENV === 'development'
      ? ['query', 'error', 'warn']
      : ['error'],
  });
};

declare global {
  var prisma: undefined | ReturnType<typeof prismaClientSingleton>;
}

const prisma = globalThis.prisma ?? prismaClientSingleton();

if (process.env.NODE_ENV !== 'production') globalThis.prisma = prisma;

export default prisma;
```

**Why singleton?** Prevents too many database connections in development with hot reload.

## CRUD Operations

### Create

```typescript
import prisma from './lib/prisma';

// Create single record
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
    password: 'hashed_password',
  },
});

// Create with relations
const userWithProfile = await prisma.user.create({
  data: {
    email: 'bob@example.com',
    name: 'Bob',
    password: 'hashed_password',
    profile: {
      create: {
        bio: 'Software developer',
        avatar: 'https://example.com/avatar.jpg',
      },
    },
  },
  include: {
    profile: true,
  },
});

// Create many
const users = await prisma.user.createMany({
  data: [
    { email: 'user1@example.com', name: 'User 1', password: 'pass1' },
    { email: 'user2@example.com', name: 'User 2', password: 'pass2' },
    { email: 'user3@example.com', name: 'User 3', password: 'pass3' },
  ],
  skipDuplicates: true, // Skip if email already exists
});
```

### Read

```typescript
// Find unique
const user = await prisma.user.findUnique({
  where: { email: 'alice@example.com' },
});

// Find first
const firstUser = await prisma.user.findFirst({
  where: { role: 'ADMIN' },
});

// Find many
const users = await prisma.user.findMany({
  where: {
    role: 'USER',
    createdAt: {
      gte: new Date('2024-01-01'),
    },
  },
  orderBy: {
    createdAt: 'desc',
  },
  take: 10,
  skip: 0,
});

// Find with relations
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
    },
    profile: true,
  },
});

// Select specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
    posts: {
      select: {
        title: true,
      },
    },
  },
});

// Count
const userCount = await prisma.user.count();

const publishedPostCount = await prisma.post.count({
  where: { published: true },
});
```

### Update

```typescript
// Update single
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'Alice Updated',
  },
});

// Update many
const result = await prisma.user.updateMany({
  where: {
    role: 'USER',
  },
  data: {
    role: 'MODERATOR',
  },
});

// Upsert (update or create)
const user = await prisma.user.upsert({
  where: { email: 'alice@example.com' },
  update: {
    name: 'Alice Updated',
  },
  create: {
    email: 'alice@example.com',
    name: 'Alice',
    password: 'hashed_password',
  },
});

// Increment/decrement
const post = await prisma.post.update({
  where: { id: 1 },
  data: {
    viewCount: {
      increment: 1,
    },
  },
});
```

### Delete

```typescript
// Delete single
await prisma.user.delete({
  where: { id: 1 },
});

// Delete many
await prisma.user.deleteMany({
  where: {
    createdAt: {
      lt: new Date('2023-01-01'),
    },
  },
});

// Delete all
await prisma.user.deleteMany();
```

## Advanced Queries

### Filtering

```typescript
// Multiple conditions (AND)
const users = await prisma.user.findMany({
  where: {
    role: 'USER',
    createdAt: {
      gte: new Date('2024-01-01'),
    },
  },
});

// OR conditions
const users = await prisma.user.findMany({
  where: {
    OR: [
      { email: { contains: '@gmail.com' } },
      { email: { contains: '@yahoo.com' } },
    ],
  },
});

// NOT conditions
const users = await prisma.user.findMany({
  where: {
    NOT: {
      role: 'ADMIN',
    },
  },
});

// Complex nested conditions
const posts = await prisma.post.findMany({
  where: {
    OR: [
      {
        AND: [
          { published: true },
          { viewCount: { gte: 100 } },
        ],
      },
      {
        author: {
          role: 'ADMIN',
        },
      },
    ],
  },
});

// String operations
const users = await prisma.user.findMany({
  where: {
    email: {
      contains: 'example',     // LIKE '%example%'
      startsWith: 'user',      // LIKE 'user%'
      endsWith: '@gmail.com',  // LIKE '%@gmail.com'
    },
  },
});

// Number operations
const posts = await prisma.post.findMany({
  where: {
    viewCount: {
      gte: 100,  // Greater than or equal
      lte: 1000, // Less than or equal
      gt: 50,    // Greater than
      lt: 500,   // Less than
    },
  },
});

// Array operations
const posts = await prisma.post.findMany({
  where: {
    tags: {
      some: {
        name: 'typescript',
      },
    },
  },
});
```

### Relations

```typescript
// One-to-many
const usersWithPosts = await prisma.user.findMany({
  include: {
    posts: true,
  },
});

// Filtered relations
const usersWithPublishedPosts = await prisma.user.findMany({
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5,
    },
  },
});

// Nested relations
const posts = await prisma.post.findMany({
  include: {
    author: {
      include: {
        profile: true,
      },
    },
    comments: {
      include: {
        author: true,
      },
    },
    tags: true,
  },
});

// Relation count
const users = await prisma.user.findMany({
  include: {
    _count: {
      select: {
        posts: true,
        comments: true,
      },
    },
  },
});
```

### Pagination

```typescript
// Offset pagination
const page = 2;
const pageSize = 10;

const users = await prisma.user.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: {
    createdAt: 'desc',
  },
});

// Cursor pagination (better performance)
const users = await prisma.user.findMany({
  take: 10,
  skip: 1, // Skip cursor
  cursor: {
    id: lastUserId, // ID of last user from previous page
  },
  orderBy: {
    id: 'asc',
  },
});
```

### Aggregations

```typescript
// Count, sum, avg, min, max
const stats = await prisma.post.aggregate({
  _count: true,
  _sum: {
    viewCount: true,
  },
  _avg: {
    viewCount: true,
  },
  _min: {
    createdAt: true,
  },
  _max: {
    updatedAt: true,
  },
  where: {
    published: true,
  },
});

// Group by
const postsByAuthor = await prisma.post.groupBy({
  by: ['authorId'],
  _count: {
    id: true,
  },
  _sum: {
    viewCount: true,
  },
  orderBy: {
    _count: {
      id: 'desc',
    },
  },
});
```

### Full-Text Search

**Enable in schema:**
```prisma
model Post {
  // ...
  @@fulltext([title, content])
}
```

**Search:**
```typescript
const posts = await prisma.post.findMany({
  where: {
    OR: [
      {
        title: {
          search: 'typescript',
        },
      },
      {
        content: {
          search: 'typescript',
        },
      },
    ],
  },
});
```

## Transactions

```typescript
// Sequential transactions
const result = await prisma.$transaction([
  prisma.user.create({ data: { email: 'user@example.com', name: 'User' } }),
  prisma.post.create({ data: { title: 'First Post', authorId: 1 } }),
]);

// Interactive transactions
const transferFunds = await prisma.$transaction(async (tx) => {
  // Withdraw from account A
  const accountA = await tx.account.update({
    where: { id: 1 },
    data: {
      balance: {
        decrement: 100,
      },
    },
  });

  // Check balance
  if (accountA.balance < 0) {
    throw new Error('Insufficient funds');
  }

  // Deposit to account B
  await tx.account.update({
    where: { id: 2 },
    data: {
      balance: {
        increment: 100,
      },
    },
  });

  return accountA;
});
```

## Raw SQL

```typescript
// Raw query (read)
const users = await prisma.$queryRaw<User[]>`
  SELECT * FROM users WHERE email LIKE ${'%@example.com'}
`;

// Raw execute (write)
await prisma.$executeRaw`
  UPDATE users SET name = ${'Updated'} WHERE id = ${1}
`;

// Unsafe (for dynamic queries - be careful!)
const tableName = 'users';
const users = await prisma.$queryRawUnsafe(`SELECT * FROM ${tableName}`);
```

## Prisma Studio

Visual database browser:

```bash
npx prisma studio
```

Opens at `http://localhost:5555` with:
- View all tables
- Add/edit/delete records
- Visual relation explorer
- Search and filter data

## Middleware

```typescript
import prisma from './lib/prisma';

// Logging middleware
prisma.$use(async (params, next) => {
  const before = Date.now();

  const result = await next(params);

  const after = Date.now();

  console.log(`Query ${params.model}.${params.action} took ${after - before}ms`);

  return result;
});

// Soft delete middleware
prisma.$use(async (params, next) => {
  if (params.model === 'User') {
    if (params.action === 'delete') {
      // Change delete to update (soft delete)
      params.action = 'update';
      params.args.data = { deleted: true };
    }

    if (params.action === 'deleteMany') {
      params.action = 'updateMany';
      if (params.args.data !== undefined) {
        params.args.data.deleted = true;
      } else {
        params.args.data = { deleted: true };
      }
    }
  }

  return next(params);
});
```

## Prisma Accelerate (Connection Pooling & Caching)

**Setup:**
```bash
npm install @prisma/extension-accelerate
```

**Enable in Prisma Cloud:**
1. Visit https://console.prisma.io/
2. Enable Accelerate for your project
3. Get Accelerate connection string

**.env**
```bash
DATABASE_URL="prisma://accelerate.prisma-data.net/?api_key=your_api_key"
```

**Usage:**
```typescript
import { PrismaClient } from '@prisma/client';
import { withAccelerate } from '@prisma/extension-accelerate';

const prisma = new PrismaClient().$extends(withAccelerate());

// Query with cache
const users = await prisma.user.findMany({
  cacheStrategy: {
    ttl: 60, // Cache for 60 seconds
    swr: 30, // Serve stale data for 30 seconds while revalidating
  },
});
```

## Prisma Pulse (Real-Time Events)

**Setup:**
```bash
npm install @prisma/extension-pulse
```

**Usage:**
```typescript
import { PrismaClient } from '@prisma/client';
import { withPulse } from '@prisma/extension-pulse';

const prisma = new PrismaClient().$extends(
  withPulse({ apiKey: process.env.PULSE_API_KEY! })
);

// Subscribe to create events
const subscription = await prisma.user.subscribe({
  create: {},
});

for await (const event of subscription) {
  console.log('New user created:', event.created);
}

// Subscribe to specific changes
const postSubscription = await prisma.post.subscribe({
  update: {
    where: { published: true },
  },
});
```

## Best Practices

### 1. Use Connection Pooling for Serverless

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL") // Pooled
  directUrl = env("DATABASE_URL_UNPOOLED") // For migrations
}
```

### 2. Select Only Required Fields

```typescript
// Good: Select specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true,
  },
});

// Bad: Fetch all fields
const users = await prisma.user.findMany();
```

### 3. Use Indexes for Frequent Queries

```prisma
model Post {
  // ...
  @@index([authorId, published])
}
```

### 4. Paginate Large Result Sets

```typescript
// Use cursor pagination for better performance
const posts = await prisma.post.findMany({
  take: 20,
  cursor: { id: lastPostId },
  orderBy: { id: 'asc' },
});
```

### 5. Handle Errors Properly

```typescript
import { Prisma } from '@prisma/client';

try {
  await prisma.user.create({ data: { email: 'existing@example.com' } });
} catch (error) {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2002') {
      console.log('Unique constraint violation');
    }
  }
  throw error;
}
```

## Common Error Codes

- `P2002` - Unique constraint violation
- `P2003` - Foreign key constraint violation
- `P2025` - Record not found
- `P1001` - Can't reach database server
- `P1002` - Database server timeout

## Resources

- **Documentation**: https://www.prisma.io/docs
- **GitHub**: https://github.com/prisma/prisma
- **Discord**: https://pris.ly/discord
- **Prisma Studio**: `npx prisma studio`
- **Playground**: https://playground.prisma.io/

## Summary

Prisma provides type-safe database access with:

1. **Auto-generated types** from schema
2. **Intuitive API** with full TypeScript support
3. **Declarative migrations** with version control
4. **Prisma Studio** for visual database management
5. **Multi-database** support (Postgres, MySQL, SQLite, MongoDB)
6. **Connection pooling** via Prisma Accelerate
7. **Real-time events** via Prisma Pulse

Perfect for TypeScript projects needing type-safe database access with excellent developer experience and minimal boilerplate.
