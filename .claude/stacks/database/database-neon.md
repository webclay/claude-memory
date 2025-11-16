# Neon Serverless Postgres Integration Guide

## Overview

Neon is a serverless PostgreSQL database platform that separates storage and compute, enabling instant provisioning, automatic scaling, and database branching. It provides the full power of Postgres with modern developer workflows.

**Key Features:**
- **Serverless**: Scale to zero when idle, pay only for usage
- **Instant Provisioning**: Databases ready in 300ms
- **Database Branching**: Copy-on-write branches for development
- **Autoscaling**: Automatic compute scaling based on load
- **Point-in-Time Restore**: Recovery to any second (30-day history)
- **Connection Pooling**: Built-in pgBouncer
- **Edge-Ready**: Serverless driver for low-latency HTTP queries
- **PostgreSQL 16**: Full Postgres compatibility

**When to Use:**
- Building serverless applications
- Need database per tenant architecture
- Want Git-like branching for databases
- Require automatic scaling
- Development/preview environments
- Cost optimization (scale-to-zero)

## Installation

**📚 Official Documentation**: [Neon - Get Started](https://neon.tech/docs/get-started-with-neon/signing-up)

Follow the official quickstart guide for framework-specific setup instructions (Next.js, Node.js, Prisma, Drizzle, etc.).

### Quick Reference

**Create Project:**
1. Sign up at Neon Console or use Neon CLI
2. Create new project
3. Get connection string

**Install Driver:**
Choose based on your use case:
- **Serverless driver** - For edge functions and low-latency HTTP
- **node-postgres** - Traditional PostgreSQL client
- **Prisma/Drizzle** - With ORMs

**Note**: Installation commands and driver options may change. Always refer to the official docs above for the current setup method.

## Configuration

### Environment Variables

**.env.local**
```bash
# From Neon console
DATABASE_URL=postgresql://username:password@ep-cool-darkness-123456.us-east-2.aws.neon.tech/neondb?sslmode=require

# Optional: Direct connection (bypasses pooler)
DATABASE_URL_UNPOOLED=postgresql://username:password@ep-cool-darkness-123456.us-east-2.aws.neon.tech/neondb?sslmode=require

# For connection pooling
DATABASE_URL_POOLED=postgresql://username:password@ep-cool-darkness-123456-pooler.us-east-2.aws.neon.tech/neondb?sslmode=require
```

## Serverless Driver (Edge Functions)

### Basic Usage

```typescript
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

// Simple query
const result = await sql`SELECT * FROM users WHERE id = ${userId}`;

// Insert
await sql`
  INSERT INTO users (name, email)
  VALUES (${name}, ${email})
`;

// Update
await sql`
  UPDATE users
  SET name = ${newName}
  WHERE id = ${userId}
`;

// Delete
await sql`
  DELETE FROM users WHERE id = ${userId}
`;

// Transaction
const results = await sql.transaction([
  sql`INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')`,
  sql`INSERT INTO posts (user_id, title) VALUES (1, 'My First Post')`,
]);
```

### Next.js App Router API Route

**app/api/users/route.ts**
```typescript
import { neon } from '@neondatabase/serverless';
import { NextRequest, NextResponse } from 'next/server';

const sql = neon(process.env.DATABASE_URL!);

export async function GET(request: NextRequest) {
  try {
    const users = await sql`
      SELECT id, name, email, created_at
      FROM users
      ORDER BY created_at DESC
      LIMIT 100
    `;

    return NextResponse.json({ users });
  } catch (error) {
    console.error('Database error:', error);
    return NextResponse.json(
      { error: 'Failed to fetch users' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const { name, email } = await request.json();

    const [user] = await sql`
      INSERT INTO users (name, email)
      VALUES (${name}, ${email})
      RETURNING id, name, email, created_at
    `;

    return NextResponse.json({ user }, { status: 201 });
  } catch (error) {
    console.error('Database error:', error);
    return NextResponse.json(
      { error: 'Failed to create user' },
      { status: 500 }
    );
  }
}
```

### Next.js Server Component

**app/users/page.tsx**
```typescript
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

interface User {
  id: number;
  name: string;
  email: string;
  created_at: Date;
}

export default async function UsersPage() {
  const users = await sql<User[]>`
    SELECT id, name, email, created_at
    FROM users
    ORDER BY created_at DESC
  `;

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.name} ({user.email})
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Vercel Edge Function

**api/edge-users.ts**
```typescript
import { neon } from '@neondatabase/serverless';

export const config = {
  runtime: 'edge',
};

const sql = neon(process.env.DATABASE_URL!);

export default async function handler(request: Request) {
  const users = await sql`SELECT * FROM users LIMIT 10`;

  return new Response(JSON.stringify({ users }), {
    headers: { 'Content-Type': 'application/json' },
  });
}
```

## Traditional Connection (node-postgres)

### Setup

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: {
    require: true,
  },
  max: 20, // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

export default pool;
```

### Usage

```typescript
import pool from './db';

// Query
export async function getUsers() {
  const result = await pool.query(
    'SELECT id, name, email FROM users ORDER BY created_at DESC'
  );
  return result.rows;
}

// Parameterized query (prevents SQL injection)
export async function getUserById(id: number) {
  const result = await pool.query(
    'SELECT * FROM users WHERE id = $1',
    [id]
  );
  return result.rows[0];
}

// Insert
export async function createUser(name: string, email: string) {
  const result = await pool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [name, email]
  );
  return result.rows[0];
}

// Transaction
export async function transferFunds(fromId: number, toId: number, amount: number) {
  const client = await pool.connect();

  try {
    await client.query('BEGIN');

    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE user_id = $2',
      [amount, fromId]
    );

    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE user_id = $2',
      [amount, toId]
    );

    await client.query('COMMIT');
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

## Prisma Integration

### Setup

**Install:**
```bash
npm install prisma @prisma/client
npx prisma init
```

**prisma/schema.prisma**
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  directUrl = env("DATABASE_URL_UNPOOLED") // For migrations
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  tags      Tag[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
  @@index([published])
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
}
```

**Run Migrations:**
```bash
# Create migration
npx prisma migrate dev --name init

# Generate Prisma Client
npx prisma generate
```

### Usage with Prisma

**lib/prisma.ts**
```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query', 'error', 'warn'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

**CRUD Operations:**
```typescript
import { prisma } from './lib/prisma';

// Create
const user = await prisma.user.create({
  data: {
    name: 'John Doe',
    email: 'john@example.com',
    password: 'hashed_password',
  },
});

// Read
const users = await prisma.user.findMany({
  where: {
    role: 'USER',
  },
  include: {
    posts: true,
  },
  orderBy: {
    createdAt: 'desc',
  },
});

// Read single
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' },
});

// Update
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'John Updated',
  },
});

// Delete
await prisma.user.delete({
  where: { id: 1 },
});

// Transactions
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { name: 'Alice', email: 'alice@example.com' } }),
  prisma.post.create({ data: { title: 'First Post', authorId: 1 } }),
]);

// Complex queries
const usersWithPosts = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        published: true,
      },
    },
  },
  include: {
    posts: {
      where: {
        published: true,
      },
      orderBy: {
        createdAt: 'desc',
      },
      take: 5,
    },
  },
});
```

## Database Branching

### Creating Branches

**Via Console:**
1. Navigate to Branches tab in Neon console
2. Click "Create Branch"
3. Choose parent branch (usually `main`)
4. Name your branch (e.g., `dev`, `feature/new-schema`)

**Via CLI:**
```bash
# Create branch from main
neonctl branches create --name dev --parent main

# Create branch for specific feature
neonctl branches create --name feature/user-profiles --parent main

# Get branch connection string
neonctl connection-string --branch dev
```

### Branch Workflows

**Development Branch:**
```bash
# Create dev branch
neonctl branches create --name dev

# Use dev connection string in .env.development
DATABASE_URL=postgresql://...dev-branch...

# Make schema changes
npx prisma migrate dev

# When ready, promote to main
neonctl branches promote --branch dev
```

**Preview Deployments (Vercel):**
```bash
# In vercel.json or deployment script
# Automatically create branch per PR

# .github/workflows/preview.yml
- name: Create Neon branch
  run: |
    BRANCH_NAME="preview-${{ github.event.pull_request.number }}"
    neonctl branches create --name $BRANCH_NAME
    CONNECTION_STRING=$(neonctl connection-string --branch $BRANCH_NAME)
    echo "DATABASE_URL=$CONNECTION_STRING" >> $GITHUB_ENV
```

**Testing Migrations:**
```bash
# Create test branch
neonctl branches create --name test-migration

# Test migration on branch
DATABASE_URL=<test-branch-url> npx prisma migrate deploy

# If successful, apply to main
# If not, delete branch and try again
neonctl branches delete test-migration
```

## Connection Pooling

### Using Pooled Connection

Neon provides automatic connection pooling via `-pooler` suffix:

```bash
# Pooled connection (recommended for serverless)
DATABASE_URL=postgresql://user:pass@ep-123-pooler.us-east-2.aws.neon.tech/db

# Direct connection (for migrations)
DATABASE_URL_UNPOOLED=postgresql://user:pass@ep-123.us-east-2.aws.neon.tech/db
```

**When to use pooled:**
- Serverless functions (Lambda, Vercel, Netlify)
- High connection count applications
- Short-lived connections

**When to use direct:**
- Running migrations
- Long-lived connections
- Traditional server applications

### Prisma with Pooling

**prisma/schema.prisma**
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL") // Pooled for queries
  directUrl = env("DATABASE_URL_UNPOOLED") // Direct for migrations
}
```

## Point-in-Time Restore

### Via Console

1. Navigate to Branches tab
2. Click "Restore"
3. Select timestamp or LSN (Log Sequence Number)
4. Choose to restore to new branch or existing

### Via CLI

```bash
# Restore to specific timestamp
neonctl branches create \
  --name restored-db \
  --restore-from main \
  --restore-to "2025-01-13 10:30:00"

# Restore to LSN
neonctl branches create \
  --name restored-db \
  --restore-from main \
  --restore-lsn 0/123456
```

## Autoscaling Configuration

**Via Console:**
1. Navigate to Project Settings
2. Go to Compute section
3. Set min/max compute units
4. Enable scale-to-zero

**Example Config:**
- **Min**: 0.25 vCPU (or 0 for scale-to-zero)
- **Max**: 2 vCPU
- **Scale-to-zero delay**: 5 minutes

**Cost Optimization:**
```
Development: Min 0, Max 0.5 (scale-to-zero enabled)
Staging: Min 0.25, Max 1
Production: Min 0.5, Max 4
```

## Read Replicas

```bash
# Create read replica
neonctl branches create --name read-replica --parent main --type read_replica

# Use for read-heavy queries
const READ_REPLICA_URL = process.env.DATABASE_URL_READ_REPLICA;

// Read from replica
const sql = neon(READ_REPLICA_URL);
const users = await sql`SELECT * FROM users`;

// Write to primary
const sqlPrimary = neon(process.env.DATABASE_URL);
await sqlPrimary`INSERT INTO users (name, email) VALUES (${name}, ${email})`;
```

## pgvector (AI/ML Embeddings)

### Setup

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create table with vector column
CREATE TABLE documents (
  id SERIAL PRIMARY KEY,
  content TEXT,
  embedding vector(1536), -- OpenAI embedding size
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create HNSW index for fast similarity search
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
```

### Usage

```typescript
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

// Insert document with embedding
async function storeDocument(content: string, embedding: number[]) {
  await sql`
    INSERT INTO documents (content, embedding)
    VALUES (${content}, ${JSON.stringify(embedding)})
  `;
}

// Find similar documents
async function findSimilar(queryEmbedding: number[], limit: number = 5) {
  const results = await sql`
    SELECT id, content, 1 - (embedding <=> ${JSON.stringify(queryEmbedding)}::vector) AS similarity
    FROM documents
    ORDER BY embedding <=> ${JSON.stringify(queryEmbedding)}::vector
    LIMIT ${limit}
  `;

  return results;
}
```

## Best Practices

### 1. Use Pooled Connections for Serverless

```typescript
// Good: Pooled connection
const DATABASE_URL = process.env.DATABASE_URL; // -pooler suffix

// Bad: Direct connection in serverless
const DATABASE_URL = process.env.DATABASE_URL_UNPOOLED;
```

### 2. Set Connection Timeouts

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  connectionTimeoutMillis: 5000,
  idleTimeoutMillis: 30000,
  max: 10,
});
```

### 3. Use Branches for Development

```bash
# Create branch per developer
neonctl branches create --name dev-alice
neonctl branches create --name dev-bob

# Create branch per feature
neonctl branches create --name feature/auth
```

### 4. Enable SSL

```typescript
// Always use SSL in production
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { require: true },
});
```

### 5. Monitor Query Performance

```sql
-- Enable query logging in Prisma
generator client {
  provider = "prisma-client-js"
  log      = ["query", "info", "warn", "error"]
}
```

## Pricing

**Free Tier:**
- 0.5 GB storage
- Unlimited compute hours (with limits)
- 1 project
- Database branching

**Pro Tier ($19/month):**
- 10 GB storage included
- Unlimited projects
- More compute
- Point-in-time restore (7 days)

**Scale Tier:**
- Custom pricing
- Dedicated resources
- 30-day point-in-time restore
- Enterprise support

## Resources

- **Documentation**: https://neon.tech/docs
- **GitHub**: https://github.com/neondatabase
- **Discord**: https://discord.gg/neon
- **Console**: https://console.neon.tech
- **Blog**: https://neon.tech/blog

## Summary

Neon provides serverless PostgreSQL with:

1. **Instant provisioning** - Databases ready in 300ms
2. **Database branching** - Git-like workflows for databases
3. **Autoscaling** - Scale to zero and pay per use
4. **Full Postgres** - Complete PostgreSQL 16 compatibility
5. **Edge-ready driver** - Low-latency HTTP queries
6. **Point-in-time restore** - 30-day recovery window
7. **Connection pooling** - Built-in pgBouncer

Perfect for serverless apps, preview environments, and teams wanting modern database workflows with Git-like branching.
