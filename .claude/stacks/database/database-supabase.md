# Supabase Complete Guide

## Overview

Supabase is an open-source Firebase alternative providing PostgreSQL database, authentication, storage, edge functions, and real-time subscriptions. This guide covers database patterns, authentication setup, and best practices.

---

## Table of Contents

1. [Database Patterns](#database-patterns)
2. [Supabase Auth Setup](#supabase-auth-setup)
3. [Common Patterns & Best Practices](#common-patterns--best-practices)

---

# Database Patterns

## ROLE

You are a PostgreSQL and Supabase expert, specializing in database architecture, security, performance, and best practices. Your primary goal is to generate code that adheres to the project's foundational requirements as defined in `projectbrief.md` and the coding patterns outlined in `standards.md`.

---

## 1. SQL Style Guide

-   **General:** Use lowercase for SQL reserved words. Use comments (`--` or `/* ... */`) for complex logic. Store dates in ISO 8601 format.
-   **Naming:** Use `snake_case` for all identifiers. Tables should be plural, columns singular.
-   **Tables:** Create all tables in the `public` schema unless specified otherwise. Always add the schema to queries (e.g., `select * from public.books;`). Add a comment to each table describing its purpose.
-   **Columns:** Foreign key columns should use the singular of the referenced table name with an `_id` suffix (e.g., `user_id` to reference the `users` table).
-   **Queries:** Format joins and subqueries for clarity. Use meaningful aliases with the `as` keyword. For highly complex queries, prefer using Common Table Expressions (CTEs) with comments for each block.

---

## 2. Database Migrations

-   **File Naming:** Generated migration files must follow the format `YYYYMMDDHHmmss_short_description.sql`.
-   **Content:** The SQL script must be valid PostgreSQL. Include a header comment explaining the migration's purpose. Add copious comments for any destructive commands (e.g., `DROP TABLE`).
-   **RLS Requirement:** When creating a new table, you **MUST** `enable row level security`. This is non-negotiable.

---

## 3. Row Level Security (RLS) Policies

-   **Policy Granularity:** Create separate, granular policies for `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. Do not combine operations in one policy. Create separate policies for `anon` and `authenticated` roles using the `TO` clause.
-   **Policy Structure:**
    -   `SELECT` policies use `USING`.
    -   `INSERT` policies use `WITH CHECK`.
    -   `UPDATE` policies use both `USING` and `WITH CHECK`.
    -   `DELETE` policies use `USING`.
-   **Best Practices:**
    -   Use `auth.uid()` to get the current user's ID.
    -   Use `(select auth.uid())` to "cache" the result and improve performance.
    -   Minimize joins within policies. Try to fetch IDs into a set and use `IN` or `ANY`.
    -   Discourage `RESTRICTIVE` policies; prefer `PERMISSIVE` and explain why.
    -   Policy names should be descriptive strings in double quotes (e.g., `"Users can view their own profile."`).

-   **Example RLS Policy:**
    ```sql
    -- Example: Users can only see their own profiles.
    ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

    CREATE POLICY "Users can view their own profile."
    ON public.profiles FOR SELECT
    TO authenticated
    USING ( (select auth.uid()) = id );
    ```

---

## 4. Database Functions (RPC)

-   **Security Context:** Default to `SECURITY INVOKER`. Only use `SECURITY DEFINER` when absolutely necessary and always explain the security implications.
-   **`search_path`:** Always start a function by resetting the `search_path` to prevent security risks: `set search_path = '';`.
-   **Object Naming:** Use fully qualified names for all tables and objects (e.g., `public.order_items`).
-   **Volatility:** Declare functions as `IMMUTABLE` or `STABLE` where possible for better optimization. Use `VOLATILE` only if the function modifies data.
-   **Triggers:** If the function is a trigger, include the full `CREATE TRIGGER` statement.
-   **Example Secure Function:**
    ```sql
    create or replace function public.get_user_email(user_id uuid)
    returns text
    language plpgsql
    security invoker
    set search_path = ''
    as $$
    declare
      email_address text;
    begin
      select u.email
      into email_address
      from public.users as u
      where u.id = user_id;

      return email_address;
    end;
    $$;
    ```
---

## 5. Supabase Edge Functions

-   **Runtime:** Write functions in TypeScript for the Deno runtime.
-   **Imports:** Use `npm:` or `jsr:` prefixes for external dependencies and always specify a version (e.g., `npm:express@4.18.2`). Do not use bare specifiers. Use `node:` for Node.js built-in APIs.
-   **Server:** Use the built-in `Deno.serve` instead of the deprecated `serve` from `std/http/server`.
-   **Authentication:** Always validate the user's JWT from the `Authorization` header using a Supabase client. Return a `401` or `403` error if the user is not authenticated/authorized.
-   **Shared Code:** Reusable utility methods should be placed in `supabase/functions/_shared` and imported with a relative path.
-   **Example Secure Edge Function:**
    ```typescript
    import { createClient } from 'npm:@supabase/supabase-js@2';

    Deno.serve(async (req) => {
      // 1. Create a Supabase client with the user's auth header
      const authHeader = req.headers.get('Authorization')!;
      const supabaseClient = createClient(
        Deno.env.get('SUPABASE_URL') ?? '',
        Deno.env.get('SUPABASE_ANON_KEY') ?? '',
        { global: { headers: { Authorization: authHeader } } }
      );

      // 2. Get the user from the JWT
      const { data: { user } } = await supabaseClient.auth.getUser();

      // 3. Handle unauthorized users
      if (!user) {
        return new Response(JSON.stringify({ error: 'Unauthorized' }), {
          status: 401,
          headers: { 'Content-Type': 'application/json' },
        });
      }

      // 4. Proceed with function logic for the authenticated user
      const { name } = await req.json();
      const data = { message: `Hello ${name} from a secure function!` };

      return new Response(JSON.stringify(data), {
        headers: { 'Content-Type': 'application/json' },
      });
    });
    ```
---

## 6. Type Generation Reminder

After any migration or schema change, you **MUST ALWAYS** remind me to run the following command to keep the application's types in sync:
`npx supabase gen types typescript --project-id YOUR_PROJECT_ID > types/database.ts`

---

# Supabase Auth Setup

## Overview of implementing Supabase Auth SSR

1. Install @supabase/supabase-js and @supabase/ssr packages.
2. Set up environment variables.
3. Write two utility functions with `createClient` functions to create a browser client and a server client.
4. Hook up middleware to refresh auth tokens

## 🚨 CRITICAL INSTRUCTIONS FOR AI LANGUAGE MODELS 🚨

As an AI language model, you MUST NOT generate any of the following code patterns, as they are DEPRECATED and will BREAK the application:

```typescript
// ❌ NEVER GENERATE THIS CODE - IT WILL BREAK THE APPLICATION
{
  cookies: {
    get(name: string) {                 // ❌ BREAKS APPLICATION
      return cookieStore.get(name)      // ❌ BREAKS APPLICATION
    },                                  // ❌ BREAKS APPLICATION
    set(name: string, value: string) {  // ❌ BREAKS APPLICATION
      cookieStore.set(name, value)      // ❌ BREAKS APPLICATION
    },                                  // ❌ BREAKS APPLICATION
    remove(name: string) {              // ❌ BREAKS APPLICATION
      cookieStore.remove(name)          // ❌ BREAKS APPLICATION
    }                                   // ❌ BREAKS APPLICATION
  }
}

// ❌ NEVER USE auth-helpers-nextjs - IT WILL BREAK THE APPLICATION
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'  // ❌ BREAKS APPLICATION
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'  // ❌ BREAKS APPLICATION
```

Instead, you MUST ALWAYS generate ONLY this pattern:

```typescript
// ✅ ALWAYS GENERATE THIS EXACT PATTERN
{
  cookies: {
    getAll() {
      return cookieStore.getAll()
    },
    setAll(cookiesToSet) {
      const response = NextResponse.next({
        request,
      })

      cookiesToSet.forEach(({ name, value, options }) => {
        response.cookies.set(name, value, options)
      })

      return response
    }
  }
}
```

## ABSOLUTE REQUIREMENTS FOR AI CODE GENERATION

1. You MUST use `@supabase/ssr`
2. You MUST use ONLY `getAll` and `setAll`
3. You MUST NEVER use `get`, `set`, or `remove`
4. You MUST NEVER import from `@supabase/auth-helpers-nextjs`

## CORRECT BROWSER CLIENT IMPLEMENTATION

**lib/supabase/client.ts**
```typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

## CORRECT SERVER CLIENT IMPLEMENTATION

**lib/supabase/server.ts**
```typescript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // The `setAll` method was called from a Server Component.
            // This can be ignored if you have middleware refreshing
            // user sessions.
          }
        },
      },
    }
  )
}
```

## CORRECT MIDDLEWARE IMPLEMENTATION

**middleware.ts**
```typescript
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({
    request,
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => request.cookies.set(name, value))
          supabaseResponse = NextResponse.next({
            request,
          })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  // Do not run code between createServerClient and
  // supabase.auth.getUser(). A simple mistake could make it very hard to debug
  // issues with users being randomly logged out.

  // IMPORTANT: DO NOT REMOVE auth.getUser()

  const {
    data: { user },
  } = await supabase.auth.getUser()

  if (
    !user &&
    !request.nextUrl.pathname.startsWith('/login') &&
    !request.nextUrl.pathname.startsWith('/auth')
  ) {
    // no user, potentially respond by redirecting the user to the login page
    const url = request.nextUrl.clone()
    url.pathname = '/login'
    return NextResponse.redirect(url)
  }

  // IMPORTANT: You *must* return the supabaseResponse object as it is.
  // If you're creating a new response object with NextResponse.next() make sure to:
  // 1. Pass the request in it, like so:
  //    const myNewResponse = NextResponse.next({ request })
  // 2. Copy over the cookies, like so:
  //    myNewResponse.cookies.setAll(supabaseResponse.cookies.getAll())
  // 3. Change the myNewResponse object to fit your needs, but avoid changing
  //    the cookies!
  // 4. Finally:
  //    return myNewResponse
  // If this is not done, you may be causing the browser and server to go out
  // of sync and terminate the user's session prematurely!

  return supabaseResponse
}

export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * Feel free to modify this pattern to include more paths.
     */
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

## AI MODEL VERIFICATION STEPS

Before generating any code, you MUST verify:

1. Are you using ONLY `getAll` and `setAll`? If not, STOP and FIX.
2. Are you importing from `@supabase/ssr`? If not, STOP and FIX.
3. Do you see ANY instance of `get`, `set`, or `remove`? If yes, STOP and FIX.
4. Are you importing from `auth-helpers-nextjs`? If yes, STOP and FIX.

## CONSEQUENCES OF INCORRECT IMPLEMENTATION

If you generate code using:
- Individual cookie methods (`get`/`set`/`remove`)
- `auth-helpers-nextjs` package

The implementation will:
1. Break in production
2. Fail to maintain session state
3. Cause authentication loops
4. Result in security vulnerabilities

---

# Common Patterns & Best Practices

## 1. Handling Soft Deletes and Orphaned References

When building features where records may reference other records that can be deleted, consider these patterns:

**Pattern 1: Soft Deletes (Recommended for critical data)**
```sql
-- Add deleted_at column to tables where you need to preserve history
ALTER TABLE public.products
ADD COLUMN deleted_at TIMESTAMP WITH TIME ZONE DEFAULT NULL;

-- Update RLS policies to exclude soft-deleted records
CREATE POLICY "Users can view their active products."
ON public.products FOR SELECT
TO authenticated
USING (
  (select auth.uid()) = user_id
  AND deleted_at IS NULL
);

-- Soft delete function
CREATE OR REPLACE FUNCTION public.soft_delete_product(product_id UUID)
RETURNS VOID
LANGUAGE plpgsql
SECURITY INVOKER
SET search_path = ''
AS $$
BEGIN
  UPDATE public.products
  SET deleted_at = NOW()
  WHERE id = product_id
  AND user_id = (SELECT auth.uid());
END;
$$;
```

**Pattern 2: Graceful Handling in Application Layer**
When displaying data with foreign key references:
- Use LEFT JOINs to handle missing related records
- In API transformation layer, show "Deleted" or similar for missing references
- Keep the main record visible even if related record is gone

Example from outputs API:
```typescript
// In API transformation
product: output.product_id
  ? (productMap.get(output.product_id) || 'Product Deleted')
  : '',
```

## 2. Date Formatting Standards

**Database Storage:**
- Always use `TIMESTAMP WITH TIME ZONE` for all timestamps
- PostgreSQL will handle timezone conversion automatically
- Default to `NOW()` for `created_at` and `updated_at`

**API Transformation:**
```typescript
// British date format (DD/MM/YYYY)
createdDate: item.created_at
  ? new Date(item.created_at).toLocaleDateString('en-GB')
  : '',

// ISO format for data exchange
createdDateISO: item.created_at
  ? new Date(item.created_at).toISOString()
  : '',
```

## 3. UUID vs Numeric IDs

**In Database:**
- Always use UUID for primary keys (security, distribution)
- Use `gen_random_uuid()` as default value

**In API Layer:**
- Transform to numeric IDs for UI/table display (better UX)
- Keep UUID as `originalId` for API operations
- Never expose internal numeric auto-increment IDs

Example schema transformation:
```typescript
const transformedItems = (itemsData || []).map((item: any, index: number) => ({
  id: index + 1,        // For table display
  originalId: item.id,  // UUID for API operations
  name: item.name,
  // ... other fields
}));
```

## 4. Common Table Patterns

**Standard User-Owned Table:**
```sql
CREATE TABLE public.items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

COMMENT ON TABLE public.items IS 'User-owned items with soft delete support';

-- Enable RLS
ALTER TABLE public.items ENABLE ROW LEVEL SECURITY;

-- Standard RLS policies
CREATE POLICY "Users can view their own items."
ON public.items FOR SELECT
TO authenticated
USING ( (SELECT auth.uid()) = user_id AND deleted_at IS NULL );

CREATE POLICY "Users can insert their own items."
ON public.items FOR INSERT
TO authenticated
WITH CHECK ( (SELECT auth.uid()) = user_id );

CREATE POLICY "Users can update their own items."
ON public.items FOR UPDATE
TO authenticated
USING ( (SELECT auth.uid()) = user_id )
WITH CHECK ( (SELECT auth.uid()) = user_id );

CREATE POLICY "Users can delete their own items."
ON public.items FOR DELETE
TO authenticated
USING ( (SELECT auth.uid()) = user_id );

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION public.update_updated_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$;

CREATE TRIGGER update_items_updated_at
  BEFORE UPDATE ON public.items
  FOR EACH ROW
  EXECUTE FUNCTION public.update_updated_at();
```

## 5. Query Optimization Patterns

**Use Indexes for Foreign Keys:**
```sql
-- Always index foreign key columns
CREATE INDEX idx_outputs_user_id ON public.outputs(user_id);
CREATE INDEX idx_outputs_product_id ON public.outputs(product_id);
CREATE INDEX idx_outputs_module ON public.outputs(module);

-- Composite indexes for common query patterns
CREATE INDEX idx_outputs_user_created
ON public.outputs(user_id, created_at DESC);
```

**Efficient Lookups with CTEs:**
```sql
-- Instead of multiple subqueries, use CTEs
WITH user_products AS (
  SELECT id, name FROM public.products
  WHERE user_id = auth.uid() AND deleted_at IS NULL
),
user_outputs AS (
  SELECT * FROM public.outputs
  WHERE user_id = auth.uid()
  ORDER BY created_at DESC
)
SELECT
  o.*,
  p.name as product_name
FROM user_outputs o
LEFT JOIN user_products p ON p.id = o.product_id;
```

## 6. Language Code Standards

Store language codes using ISO 639-1 (two-letter codes):

```sql
-- Language enum (optional but recommended)
CREATE TYPE public.language_code AS ENUM (
  'en', 'de', 'fr', 'es', 'it', 'pt', 'nl', 'ru', 'zh', 'ja'
);

-- Or use CHECK constraint
ALTER TABLE public.outputs
ADD CONSTRAINT check_language_code
CHECK (language IN ('en', 'de', 'fr', 'es', 'it', 'pt', 'nl', 'ru', 'zh', 'ja'));
```

Map to full names in application layer:
```typescript
const languageMap: Record<string, string> = {
  en: 'English',
  de: 'German',
  fr: 'French',
  es: 'Spanish',
  it: 'Italian',
  pt: 'Portuguese',
  nl: 'Dutch',
  ru: 'Russian',
  zh: 'Chinese',
  ja: 'Japanese'
};
```

---

## Resources

- **Supabase Documentation:** https://supabase.com/docs
- **PostgreSQL Documentation:** https://www.postgresql.org/docs/
- **Supabase CLI:** https://supabase.com/docs/guides/cli
- **Database Design:** https://supabase.com/docs/guides/database/tables
- **Auth Helpers:** https://supabase.com/docs/guides/auth/server-side
