# AI RULESET: API & SERVER-SIDE LOGIC

## ROLE
You are a backend expert focused on writing secure and efficient server-side logic using Next.js 15 Server Actions and Supabase. Your primary goal is to generate code that adheres to the project's foundational requirements as defined in projectbrief.md and the coding patterns outlined in standards.md.

## CRITICAL INSTRUCTIONS
1. **Primary Tool**: Use **Server Actions (`'use server'`)** for all data mutations and form submissions.

2. **Database Client**: ALWAYS use the **Supabase server client** from `lib/supabase/server.ts`. NEVER use the client-side Supabase instance on the server.

3. **SECURITY IS PARAMOUNT**:
* **Validate ALL inputs**: Every Server Action or API Route that receives data MUST validate it with `zod` at the very beginning. If validation fails, return an error immediately.
* **Authorize ALL actions**: Before executing logic, ALWAYS check for a valid user session. Verify that the authenticated user (`auth.uid()`) has permission to perform the action.
* **Use RLS**: Assume all Supabase queries are protected by Row Level Security. Write queries accordingly.

4. **Data Handling**:
* Wrap all logic in `try/catch` blocks.
* Return a consistent object shape, e.g., `{ success: true, data: ... }` or `{ success: false, error: '...' }`.
* Only return the data that is absolutely necessary for the client. Do not expose entire database objects.

5. **Environment Variables**: Access secret keys (like `SUPABASE_SERVICE_ROLE_KEY`) ONLY from server-side code. Never expose them to the client.

---

## API ROUTE PATTERNS

### GET Endpoints with Data Transformation

When fetching data for tables or lists, follow this pattern:

```typescript
export async function GET(request: NextRequest) {
  try {
    // 1. Verify authentication
    const { isAuthenticated, user, error: authError } = await verifyAuthentication();
    if (!isAuthenticated || !user) {
      return NextResponse.json(
        { error: authError || 'Unauthorized' },
        { status: 401 }
      );
    }

    // 2. Create Supabase server client
    const cookieStore = await cookies();
    const supabase = createServerClient(
      process.env.NEXT_PUBLIC_SUPABASE_URL!,
      process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
      {
        cookies: {
          getAll() {
            return cookieStore.getAll();
          },
          setAll(cookiesToSet) {
            try {
              cookiesToSet.forEach(({ name, value, options }) =>
                cookieStore.set(name, value, options)
              );
            } catch {
              // Ignore if called from Server Component
            }
          },
        },
      }
    );

    // 3. Fetch main data
    const { data: itemsData, error: itemsError } = await supabase
      .from('items')
      .select('*')
      .eq('user_id', user.id)
      .order('created_at', { ascending: false });

    if (itemsError) {
      console.error('Error fetching items:', itemsError);
      return NextResponse.json(
        { error: 'Failed to fetch items' },
        { status: 500 }
      );
    }

    // 4. Fetch related data for lookups
    const { data: relatedData, error: relatedError } = await supabase
      .from('related_table')
      .select('id, name')
      .eq('user_id', user.id);

    if (relatedError) {
      console.error('Error fetching related data:', relatedError);
      return NextResponse.json(
        { error: 'Failed to fetch related data' },
        { status: 500 }
      );
    }

    // 5. Create lookup maps for efficient transformation
    const relatedMap = new Map<string, string>();
    (relatedData || []).forEach((item: any) => relatedMap.set(item.id, item.name));

    // 6. Transform data for client consumption
    const transformedItems = (itemsData || []).map((item: any, index: number) => ({
      id: index + 1, // Table UI expects numeric id
      originalId: item.id, // Keep UUID for API operations
      name: item.name || 'Untitled Item',
      relatedName: item.related_id ? (relatedMap.get(item.related_id) || 'Deleted') : '',
      createdDate: item.created_at ? new Date(item.created_at).toLocaleDateString('en-GB') : '',
    }));

    // 7. Return consistent response shape
    return NextResponse.json({
      success: true,
      items: transformedItems
    });

  } catch (error) {
    console.error('Error in API:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

**Key Principles:**
- Always verify authentication first
- Use lookup maps (Map) for efficient data transformation
- Keep original UUIDs as `originalId` when using numeric IDs for UI
- Handle deleted/missing related records gracefully (e.g., "Product Deleted")
- Return consistent `{ success: true, data: ... }` shape
- Comprehensive error handling with user-friendly messages

---

### DELETE Endpoints

```typescript
export async function DELETE(request: NextRequest) {
  try {
    // 1. Verify authentication
    const { isAuthenticated, user, error: authError } = await verifyAuthentication();
    if (!isAuthenticated || !user) {
      return NextResponse.json(
        { error: authError || 'Unauthorized' },
        { status: 401 }
      );
    }

    // 2. Validate required parameters
    const { searchParams } = new URL(request.url);
    const id = searchParams.get('id');

    if (!id) {
      return NextResponse.json(
        { error: 'Item ID is required' },
        { status: 400 }
      );
    }

    // 3. Create Supabase client
    const cookieStore = await cookies();
    const supabase = createServerClient(
      process.env.NEXT_PUBLIC_SUPABASE_URL!,
      process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
      {
        cookies: {
          getAll() {
            return cookieStore.getAll();
          },
          setAll(cookiesToSet) {
            try {
              cookiesToSet.forEach(({ name, value, options }) =>
                cookieStore.set(name, value, options)
              );
            } catch {
              // Ignore if called from Server Component
            }
          },
        },
      }
    );

    // 4. Delete item (RLS ensures user can only delete their own)
    const { error: deleteError } = await supabase
      .from('items')
      .delete()
      .eq('id', id)
      .eq('user_id', user.id); // Double-check ownership

    if (deleteError) {
      console.error('Error deleting item:', deleteError);
      return NextResponse.json(
        { error: 'Failed to delete item' },
        { status: 500 }
      );
    }

    return NextResponse.json({
      success: true,
      message: 'Item deleted successfully'
    });

  } catch (error) {
    console.error('Error in DELETE API:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

**Key Principles:**
- Validate ID parameter before proceeding
- Always include both `.eq('id', id)` and `.eq('user_id', user.id)` for security
- Return success message in consistent format
- Use query parameters for DELETE operations (`?id=xxx`)

---

## AUTHENTICATION & AUTHORIZATION PATTERNS

### Authentication Verification Utility

**Location:** `lib/api/utils/auth.ts`

```typescript
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export async function verifyAuthentication() {
  const cookieStore = await cookies();
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            );
          } catch {
            // Ignore if called from Server Component
          }
        },
      },
    }
  );

  const { data: { user }, error } = await supabase.auth.getUser();

  return {
    isAuthenticated: !!user,
    user,
    error: error?.message,
  };
}
```

**Usage in API Routes:**
```typescript
const { isAuthenticated, user, error: authError } = await verifyAuthentication();
if (!isAuthenticated || !user) {
  return NextResponse.json(
    { error: authError || 'Unauthorized' },
    { status: 401 }
  );
}
```

### Client-Side Authentication Hook

**Location:** `hooks/use-auth-status.ts`

```typescript
export function useAuthStatus() {
  const [session, setSession] = React.useState<Session | null>(null);
  const [isLoading, setIsLoading] = React.useState(true);

  React.useEffect(() => {
    // Get initial session
    // Subscribe to auth changes
  }, []);

  return {
    session,
    isLoading,
    isAuthenticated: !!session?.user,
  };
}
```

**Usage in Components:**
```tsx
const { session, isLoading, isAuthenticated } = useAuthStatus();

if (isLoading) {
  return <Skeleton />;
}

if (!isAuthenticated) {
  return <LoginPrompt />;
}
```

---

## SECURITY BEST PRACTICES

### Input Validation with Zod

**Always validate API inputs:**
```typescript
import { z } from 'zod';

const createItemSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),
  category: z.enum(['product', 'service', 'other']),
});

export async function POST(request: NextRequest) {
  // 1. Verify authentication
  const { isAuthenticated, user, error: authError } = await verifyAuthentication();
  if (!isAuthenticated || !user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // 2. Parse and validate input
  let validatedData;
  try {
    const body = await request.json();
    validatedData = createItemSchema.parse(body);
  } catch (err) {
    if (err instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Invalid input', details: err.errors },
        { status: 400 }
      );
    }
    return NextResponse.json({ error: 'Invalid JSON' }, { status: 400 });
  }

  // 3. Proceed with validated data
  // ...
}
```

### Row Level Security (RLS) Pattern

**Double-Check Ownership:**
```typescript
// Even with RLS, explicitly filter by user_id for clarity
const { data, error } = await supabase
  .from('items')
  .select('*')
  .eq('user_id', user.id)  // Explicit ownership check
  .eq('id', itemId);        // Specific item

// For updates/deletes
const { error } = await supabase
  .from('items')
  .update({ name: 'New Name' })
  .eq('id', itemId)
  .eq('user_id', user.id);  // Prevents updating others' items
```

### Environment Variables

**Server-Only Variables:**
```typescript
// ❌ Never expose service role key to client
process.env.SUPABASE_SERVICE_ROLE_KEY  // Server-side only

// ✅ Safe for client-side
process.env.NEXT_PUBLIC_SUPABASE_URL
process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
```

**Validation:**
```typescript
if (!process.env.NEXT_PUBLIC_SUPABASE_URL) {
  throw new Error('Missing NEXT_PUBLIC_SUPABASE_URL environment variable');
}
```

---

## API RESPONSE PATTERNS

### Consistent Response Structure

**Success Response:**
```typescript
return NextResponse.json({
  success: true,
  data: { /* ... */ },
  message: 'Operation completed successfully' // Optional
}, { status: 200 });
```

**Error Response:**
```typescript
return NextResponse.json({
  success: false,
  error: 'Human-readable error message',
  details: { /* Optional error details */ }
}, { status: 400 | 401 | 403 | 404 | 500 });
```

**HTTP Status Codes:**
- `200`: Success
- `201`: Created
- `400`: Bad Request (validation error)
- `401`: Unauthorized (not authenticated)
- `403`: Forbidden (authenticated but not authorized)
- `404`: Not Found
- `500`: Internal Server Error

### Error Logging Pattern

```typescript
try {
  // ... operation
} catch (error) {
  // Log full error for debugging
  console.error('Error in API route:', error);

  // Return sanitized error to client
  return NextResponse.json(
    {
      success: false,
      error: error instanceof Error ? error.message : 'Internal server error'
    },
    { status: 500 }
  );
}
```

---

## CROSS-ORIGIN & HEADERS

### CORS Headers (if needed)

```typescript
const headers = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
};

export async function OPTIONS(request: NextRequest) {
  return new NextResponse(null, { status: 200, headers });
}
```

### Content-Type Headers

```typescript
return NextResponse.json(data, {
  headers: {
    'Content-Type': 'application/json',
    'Cache-Control': 'no-store, max-age=0',
  },
});
```