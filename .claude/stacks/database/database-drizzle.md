# Drizzle ORM Integration Guide

## Overview

Drizzle ORM is a lightweight, TypeScript-first ORM that prioritizes type safety and performance. It's a "headless ORM" that operates as a library rather than a framework—you build *with* Drizzle, not *around* it.

**Key Features:**
- **Type-Safe**: Full TypeScript support with automatic type inference
- **SQL-First**: Embraces SQL directly rather than abstracting it away
- **Zero Dependencies**: No external dependencies, perfect for edge/serverless
- **Lightning Fast**: Significantly faster than Prisma in benchmarks
- **Drizzle Studio**: Built-in visual database explorer
- **Flexible Migrations**: Comprehensive migration management
- **Multi-Database**: PostgreSQL, MySQL, SQLite, and many cloud variants
- **Edge-Ready**: Works in Cloudflare Workers, Vercel Functions, Deno Deploy, Bun

**When to Use:**
- Need strong TypeScript type safety
- Building for edge/serverless environments
- Want SQL-like query syntax (familiar to SQL developers)
- Prioritize query performance and low overhead
- Working with Neon, Supabase, Turso, PlanetScale, or other modern providers
- Want lighter-weight alternative to Prisma
- Need flexibility in project structure

---

## Installation

**📚 Official Documentation**: [Drizzle ORM - Get Started](https://orm.drizzle.team/docs/get-started)

Follow the official getting started guide for setup instructions specific to your database and framework.

### Quick Reference

The general setup process:

1. **Install Drizzle ORM and driver**
2. **Set up database connection**
3. **Define schema in TypeScript**
4. **Generate and run migrations**
5. **Query your database**

**Note**: Installation commands vary by database provider. Always refer to the official docs for the current method.

### Example Installation (PostgreSQL with Neon)

```bash
npm install drizzle-orm @neondatabase/serverless
npm install -D drizzle-kit
```

### Example Installation (SQLite with Turso)

```bash
npm install drizzle-orm @libsql/client
npm install -D drizzle-kit
```

---

## Configuration

### Database Connection

**lib/db.ts** (PostgreSQL/Neon example)

```typescript
import { drizzle } from 'drizzle-orm/neon-http';
import { neon } from '@neondatabase/serverless';
import * as schema from './schema';

const sql = neon(process.env.DATABASE_URL!);
export const db = drizzle(sql, { schema });
```

**lib/db.ts** (SQLite/Turso example)

```typescript
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';
import * as schema from './schema';

const client = createClient({
  url: process.env.DATABASE_URL!,
  authToken: process.env.DATABASE_AUTH_TOKEN,
});

export const db = drizzle(client, { schema });
```

### Drizzle Config

**drizzle.config.ts**

```typescript
import type { Config } from 'drizzle-kit';

export default {
  schema: './lib/schema.ts',
  out: './drizzle',
  dialect: 'postgresql', // or 'mysql', 'sqlite'
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

For multiple schema files:

```typescript
export default {
  schema: './lib/schema/**/*.ts', // Recursively finds all schemas
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

---

## Schema Definition

### Basic Table

**lib/schema.ts**

```typescript
import { pgTable, serial, text, timestamp, boolean } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  emailVerified: boolean('email_verified').default(false),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});
```

### Column Types

**PostgreSQL:**
```typescript
import {
  pgTable,
  serial,
  integer,
  text,
  varchar,
  boolean,
  timestamp,
  json,
  jsonb,
  uuid,
  decimal,
  real,
  doublePrecision,
} from 'drizzle-orm/pg-core';

export const products = pgTable('products', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: varchar('name', { length: 255 }).notNull(),
  description: text('description'),
  price: decimal('price', { precision: 10, scale: 2 }).notNull(),
  stock: integer('stock').default(0),
  metadata: jsonb('metadata'),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at').defaultNow(),
});
```

**MySQL:**
```typescript
import { mysqlTable, serial, varchar, text, boolean, timestamp, int } from 'drizzle-orm/mysql-core';

export const users = mysqlTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});
```

**SQLite:**
```typescript
import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  createdAt: integer('created_at', { mode: 'timestamp' }),
});
```

### Relationships

```typescript
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  authorId: integer('author_id').references(() => users.id),
});

// Define relations
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
}));
```

### Enums

**PostgreSQL:**
```typescript
import { pgTable, pgEnum, serial, text } from 'drizzle-orm/pg-core';

export const roleEnum = pgEnum('role', ['user', 'admin', 'moderator']);

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull(),
  role: roleEnum('role').default('user').notNull(),
});
```

### Indexes and Constraints

```typescript
import { pgTable, serial, text, index, uniqueIndex } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull(),
  username: text('username').notNull(),
}, (table) => ({
  emailIdx: uniqueIndex('email_idx').on(table.email),
  usernameIdx: index('username_idx').on(table.username),
}));
```

### Reusable Timestamp Fields

```typescript
import { timestamp } from 'drizzle-orm/pg-core';

// Reusable timestamp pattern
export const timestamps = {
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
};

// Use in tables
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull(),
  ...timestamps,
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  ...timestamps,
});
```

### Automatic camelCase to snake_case

```typescript
import { drizzle } from 'drizzle-orm/neon-http';
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

// Enable automatic casing conversion
export const db = drizzle(sql, {
  schema,
  casing: 'snake_case', // TypeScript camelCase → database snake_case
});

// Now you can use camelCase in TypeScript
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  emailAddress: text('email_address'), // Becomes email_address in DB
  firstName: text('first_name'),       // Becomes first_name in DB
  createdAt: timestamp('created_at'),  // Becomes created_at in DB
});
```

---

## Migrations

### Generate Migration

```bash
npx drizzle-kit generate
```

This creates a SQL migration file in the `drizzle/` folder based on your schema changes.

### Apply Migration

**Run migrations programmatically:**

```typescript
import { drizzle } from 'drizzle-orm/neon-http';
import { migrate } from 'drizzle-orm/neon-http/migrator';
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle(sql);

await migrate(db, { migrationsFolder: './drizzle' });
```

**Or use the CLI:**

```bash
npx drizzle-kit push
```

This applies schema changes directly to the database (good for development).

### Migration Workflow

1. **Modify schema** in `schema.ts`
2. **Generate migration**: `npx drizzle-kit generate`
3. **Review SQL** in `drizzle/` folder
4. **Apply migration**: Run migration code or `npx drizzle-kit push`

---

## Querying Data

### Select

```typescript
import { db } from './lib/db';
import { users } from './lib/schema';

// Select all
const allUsers = await db.select().from(users);

// Select with conditions
import { eq, and, or, gt, like } from 'drizzle-orm';

const activeUsers = await db
  .select()
  .from(users)
  .where(eq(users.emailVerified, true));

// Multiple conditions
const filteredUsers = await db
  .select()
  .from(users)
  .where(
    and(
      eq(users.emailVerified, true),
      like(users.email, '%@gmail.com')
    )
  );

// Select specific columns
const userEmails = await db
  .select({ email: users.email, name: users.name })
  .from(users);

// With limit and offset
const paginatedUsers = await db
  .select()
  .from(users)
  .limit(10)
  .offset(20);
```

### Insert

```typescript
// Insert single
const newUser = await db
  .insert(users)
  .values({
    email: 'user@example.com',
    name: 'John Doe',
  })
  .returning();

// Insert multiple
const newUsers = await db
  .insert(users)
  .values([
    { email: 'user1@example.com', name: 'User 1' },
    { email: 'user2@example.com', name: 'User 2' },
  ])
  .returning();
```

### Update

```typescript
import { eq } from 'drizzle-orm';

// Update
const updated = await db
  .update(users)
  .set({ emailVerified: true })
  .where(eq(users.email, 'user@example.com'))
  .returning();
```

### Delete

```typescript
import { eq } from 'drizzle-orm';

// Delete
const deleted = await db
  .delete(users)
  .where(eq(users.id, 1))
  .returning();
```

### Relational Queries

With relations defined, use the relational query API:

```typescript
// Find user with all their posts
const userWithPosts = await db.query.users.findFirst({
  where: eq(users.id, 1),
  with: {
    posts: true,
  },
});

// Nested relations
const userWithPostsAndComments = await db.query.users.findFirst({
  where: eq(users.id, 1),
  with: {
    posts: {
      with: {
        comments: true,
      },
    },
  },
});

// Find many
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: true,
  },
});
```

### Joins

```typescript
import { eq } from 'drizzle-orm';

// Manual join
const usersWithPosts = await db
  .select({
    userId: users.id,
    userName: users.name,
    postId: posts.id,
    postTitle: posts.title,
  })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId));
```

### Transactions

```typescript
await db.transaction(async (tx) => {
  const user = await tx.insert(users).values({ email: 'user@example.com', name: 'John' }).returning();

  await tx.insert(posts).values({
    title: 'First Post',
    authorId: user[0].id,
  });
});
```

---

## Drizzle Studio

Visual database explorer and data editor.

**Launch Studio:**

```bash
npx drizzle-kit studio
```

This opens a web UI where you can:
- Browse tables and data
- Edit records directly
- Visualize relationships
- Run queries

**Access:** Opens at `https://local.drizzle.studio`

---

## Framework Integration

### Next.js App Router

**app/actions.ts** (Server Actions)

```typescript
'use server';

import { db } from '@/lib/db';
import { users } from '@/lib/schema';
import { eq } from 'drizzle-orm';

export async function createUser(email: string, name: string) {
  const user = await db.insert(users).values({ email, name }).returning();
  return user[0];
}

export async function getUser(id: number) {
  const user = await db.query.users.findFirst({
    where: eq(users.id, id),
  });
  return user;
}
```

**app/page.tsx** (Server Component)

```typescript
import { db } from '@/lib/db';
import { users } from '@/lib/schema';

export default async function UsersPage() {
  const allUsers = await db.select().from(users);

  return (
    <div>
      {allUsers.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

### Next.js API Routes

**app/api/users/route.ts**

```typescript
import { NextResponse } from 'next/server';
import { db } from '@/lib/db';
import { users } from '@/lib/schema';

export async function GET() {
  const allUsers = await db.select().from(users);
  return NextResponse.json(allUsers);
}

export async function POST(request: Request) {
  const body = await request.json();
  const newUser = await db.insert(users).values(body).returning();
  return NextResponse.json(newUser[0]);
}
```

### TanStack Start / Remix

Works the same way as Next.js—use Drizzle in loaders and actions.

---

## Type Safety

Drizzle provides full type inference:

```typescript
// TypeScript knows the exact shape
const user = await db.query.users.findFirst({
  where: eq(users.id, 1),
});

// user is typed as:
// {
//   id: number;
//   email: string;
//   name: string;
//   emailVerified: boolean;
//   createdAt: Date;
//   updatedAt: Date;
// } | undefined

// With selected columns
const userEmail = await db
  .select({ email: users.email })
  .from(users);

// userEmail is typed as:
// { email: string }[]
```

### Infer Types from Schema

```typescript
import { InferSelectModel, InferInsertModel } from 'drizzle-orm';
import { users } from './schema';

export type User = InferSelectModel<typeof users>;
export type NewUser = InferInsertModel<typeof users>;

// Use in functions
async function createUser(newUser: NewUser): Promise<User> {
  const user = await db.insert(users).values(newUser).returning();
  return user[0];
}
```

---

## Advanced Patterns

### Partial Selects with Type Safety

```typescript
import { createSelectSchema } from 'drizzle-zod';
import { z } from 'zod';

// Create Zod schema from Drizzle table
const userSchema = createSelectSchema(users);

// Partial select
const userEmailSchema = userSchema.pick({ email: true, name: true });

type UserEmail = z.infer<typeof userEmailSchema>;
```

### Dynamic Queries

```typescript
import { SQL, and, eq, like } from 'drizzle-orm';

function buildUserQuery(filters: { email?: string; verified?: boolean }) {
  const conditions: SQL[] = [];

  if (filters.email) {
    conditions.push(like(users.email, `%${filters.email}%`));
  }

  if (filters.verified !== undefined) {
    conditions.push(eq(users.emailVerified, filters.verified));
  }

  return db
    .select()
    .from(users)
    .where(conditions.length > 0 ? and(...conditions) : undefined);
}

// Usage
const results = await buildUserQuery({ email: 'example.com', verified: true });
```

### Raw SQL

When you need full control:

```typescript
import { sql } from 'drizzle-orm';

const result = await db.execute(sql`
  SELECT * FROM users WHERE email = ${email}
`);
```

---

## Validation with Zod

**Install:**
```bash
npm install drizzle-zod zod
```

**Generate Zod schemas:**

```typescript
import { createInsertSchema, createSelectSchema } from 'drizzle-zod';
import { users } from './schema';

// Auto-generated from Drizzle schema
export const insertUserSchema = createInsertSchema(users);
export const selectUserSchema = createSelectSchema(users);

// Customize validation
export const createUserSchema = insertUserSchema.pick({
  email: true,
  name: true,
}).extend({
  email: z.string().email(),
  name: z.string().min(2).max(100),
});

// Use in API
export async function createUser(data: unknown) {
  const validated = createUserSchema.parse(data);
  return db.insert(users).values(validated).returning();
}
```

---

## Comparison with Prisma

| Feature | Drizzle | Prisma |
|---------|---------|--------|
| **Philosophy** | SQL-first, library | Abstraction-first, framework |
| **Performance** | Faster (especially in serverless) | Slower in serverless |
| **Bundle Size** | Smaller (zero dependencies) | Larger |
| **Type Safety** | Excellent | Excellent |
| **Schema Definition** | TypeScript code | Prisma Schema Language |
| **Migrations** | SQL-based | Prisma Migrate |
| **Edge Support** | Native | Limited (Data Proxy required) |
| **Learning Curve** | Steeper (SQL knowledge helpful) | Easier (abstracts SQL) |
| **Flexibility** | More flexible | More opinionated |
| **Studio** | Drizzle Studio (free) | Prisma Studio (free) |

**Use Drizzle when:**
- You know SQL and want direct control
- Building for edge/serverless environments
- Performance is critical
- Want zero dependencies
- Prefer TypeScript-based schema

**Use Prisma when:**
- You prefer abstraction over SQL
- Want extensive documentation and community
- Don't need edge deployment
- Prefer declarative schema language

---

## Best Practices

### 1. Organize Schemas

**Option 1: Single file** (small projects)
```typescript
// lib/schema.ts
export const users = pgTable('users', { ... });
export const posts = pgTable('posts', { ... });
```

**Option 2: Multiple files** (larger projects)
```
lib/schema/
├── users.ts
├── posts.ts
├── comments.ts
└── index.ts  // Re-export all
```

### 2. Use Prepared Statements

For repeated queries:

```typescript
import { placeholder } from 'drizzle-orm';

const getUserById = db
  .select()
  .from(users)
  .where(eq(users.id, placeholder('id')))
  .prepare('get_user_by_id');

// Reuse
const user1 = await getUserById.execute({ id: 1 });
const user2 = await getUserById.execute({ id: 2 });
```

### 3. Use Transactions

For related operations:

```typescript
await db.transaction(async (tx) => {
  const user = await tx.insert(users).values({ ... }).returning();
  await tx.insert(posts).values({ authorId: user[0].id, ... });
});
```

### 4. Connection Pooling

For serverless (like Neon):

```typescript
import { Pool } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-serverless';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle(pool);
```

### 5. Environment Variables

```bash
# PostgreSQL/Neon
DATABASE_URL="postgresql://user:password@host/database"

# SQLite/Turso
DATABASE_URL="libsql://your-database.turso.io"
DATABASE_AUTH_TOKEN="your-token"
```

---

## Troubleshooting

### "Cannot find module drizzle-orm"

**Solution:** Install dependencies:
```bash
npm install drizzle-orm
```

### Schema changes not reflected

**Solution:** Regenerate migration:
```bash
npx drizzle-kit generate
npx drizzle-kit push
```

### Type errors in queries

**Solution:** Ensure schema is imported correctly and use correct operators:
```typescript
import { eq, and, or } from 'drizzle-orm';
```

### Edge deployment issues

**Solution:** Use appropriate driver (e.g., `@neondatabase/serverless` for Neon, `@libsql/client` for Turso).

---

## Resources

**Official Documentation:**
- Website: https://orm.drizzle.team
- Getting Started: https://orm.drizzle.team/docs/get-started
- Schema: https://orm.drizzle.team/docs/sql-schema-declaration
- Queries: https://orm.drizzle.team/docs/rqb

**Tools:**
- Drizzle Kit: Migration tool
- Drizzle Studio: Database explorer
- drizzle-zod: Zod integration
- drizzle-typebox: TypeBox integration

**Community:**
- GitHub: https://github.com/drizzle-team/drizzle-orm
- Discord: Join Drizzle community

---

## Summary

Drizzle ORM is a **lightweight, TypeScript-first ORM** perfect for modern web development:

✅ **Fast** - Significantly faster than Prisma
✅ **Type-Safe** - Full TypeScript inference
✅ **SQL-First** - Familiar to SQL developers
✅ **Edge-Ready** - Zero dependencies, works everywhere
✅ **Flexible** - Library, not framework
✅ **Modern** - Built for serverless, Cloudflare Workers, Vercel, Deno

**Perfect for:**
- Edge/serverless applications
- High-performance requirements
- Developers comfortable with SQL
- TypeScript projects
- Neon, Turso, PlanetScale, Supabase users

**Getting Started:**
```bash
npm install drizzle-orm @neondatabase/serverless
npm install -D drizzle-kit
npx drizzle-kit studio
```

Drizzle combines the best of both worlds: the power of SQL with the safety of TypeScript.
