# TanStack Start Framework Patterns

## Overview

TanStack Start is a modern full-stack React framework built on TanStack Router. It provides type-safe routing, server functions, and seamless data loading with excellent DX.

**Key Features:**
- Type-safe file-based routing
- Server functions with automatic serialization
- Built-in SSR and streaming
- Integrates perfectly with TanStack Query
- Framework-agnostic (works with Vite, Nitro, etc.)

---

## Understanding the Execution Model

> **CRITICAL:** This is the #1 difference from Next.js that trips up developers.

### Loaders Run on BOTH Client AND Server

Unlike Next.js React Server Components, TanStack Start loaders execute on **both** the client and server:

```typescript
// This loader runs on server during SSR
// AND on client during client-side navigation
export const Route = createFileRoute('/posts')({
  loader: async () => {
    console.log('This logs on server AND client!');
    const posts = await getPosts();
    return { posts };
  },
  component: PostsList
});
```

### No `use client` or `use server` Directives

TanStack Start has **no directives**. Everything runs everywhere unless you explicitly tell it not to.

### Why Database Code Breaks in Loaders

```typescript
// ❌ WRONG - This will break on the client!
export const Route = createFileRoute('/posts')({
  loader: async () => {
    // Database code runs on client too, causing errors
    const posts = await db.query.posts.findMany();
    return { posts };
  }
});

// ✅ CORRECT - Wrap in server function
const getPosts = createServerFn({ method: 'GET' }).handler(async () => {
  return await db.query.posts.findMany();
});

export const Route = createFileRoute('/posts')({
  loader: async () => {
    // Server function makes fetch request from client
    const posts = await getPosts();
    return { posts };
  }
});
```

### How Server Functions Work

When you call a server function:
- **On server:** Executes the code directly
- **On client:** Makes a fetch request to the server, which executes the code

This allows the same code to work seamlessly in both environments.

---

## Installation and Setup

**📚 Official Documentation**: [TanStack Start - Quick Start](https://tanstack.com/start/latest/docs/framework/react/quick-start)

Follow the official quick start guide for the latest installation instructions and setup process.

**Note**: TanStack Start is currently in Release Candidate (RC) status. The team recommends locking to specific versions in production.

### Quick Reference

The general setup process involves:
1. Create new project with the CLI
2. Configure file-based routing structure
3. Set up server functions
4. Start development server

**Note**: CLI commands may change. Always refer to the official docs above for the current setup method.

### Project Structure

```
app/
├── routes/
│   ├── __root.tsx          # Root layout
│   ├── index.tsx           # Home page (/)
│   ├── about.tsx           # About page (/about)
│   ├── posts/
│   │   ├── index.tsx       # Posts list (/posts)
│   │   └── $postId.tsx     # Post detail (/posts/:postId)
│   └── _layout.tsx         # Shared layout
├── api/
│   └── server-functions.ts # Server-side functions
└── client.tsx              # Client entry point
```

---

## File-Based Routing

### Basic Routes

**app/routes/index.tsx** - Home page at `/`
```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/')({
  component: HomePage
});

function HomePage() {
  return (
    <div>
      <h1>Welcome Home</h1>
    </div>
  );
}
```

**app/routes/about.tsx** - About page at `/about`
```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/about')({
  component: AboutPage
});

function AboutPage() {
  return <h1>About Us</h1>;
}
```

### Route File Naming Conventions

Three equivalent ways to create the same route (`/todos/new`):

| Style | File Path | When to Use |
|-------|-----------|-------------|
| Folder | `routes/todos/new/index.tsx` | Complex routes with siblings |
| File | `routes/todos/new.tsx` | Simple standalone routes |
| Dot notation | `routes/todos.new.tsx` | Flat file structure preference |

All three create the exact same route. Choose based on project preference.

### Dynamic Routes

**app/routes/posts.$postId.tsx** - Dynamic route at `/posts/:postId`
```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  component: PostDetail
});

function PostDetail() {
  const { postId } = Route.useParams();

  return (
    <div>
      <h1>Post {postId}</h1>
    </div>
  );
}
```

**Note:** The `$` prefix denotes a dynamic segment. The path `/posts/$postId` matches `/posts/123`, `/posts/abc`, etc.

### Nested Routes with Layouts

**app/routes/_layout.tsx** - Shared layout (underscore prefix = pathless layout)
```typescript
import { createFileRoute, Outlet } from '@tanstack/react-router';

export const Route = createFileRoute('/_layout')({
  component: LayoutComponent
});

function LayoutComponent() {
  return (
    <div className="layout">
      <nav>
        <a href="/dashboard">Dashboard</a>
        <a href="/settings">Settings</a>
      </nav>
      <main>
        <Outlet /> {/* Child routes render here */}
      </main>
    </div>
  );
}
```

**app/routes/_layout/dashboard.tsx** - Nested under layout at `/dashboard`
```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/_layout/dashboard')({
  component: Dashboard
});

function Dashboard() {
  return <h1>Dashboard</h1>;
}
```

### Root Layout

**app/routes/__root.tsx** - Wraps all routes

> **CRITICAL:** Must include `<Head />` and `<Scripts />` components or the app will break!

```typescript
import { createRootRoute, Outlet } from '@tanstack/react-router';
import { Head, Scripts } from '@tanstack/start';
import { TanStackRouterDevtools } from '@tanstack/router-devtools';

export const Route = createRootRoute({
  component: RootComponent
});

function RootComponent() {
  return (
    <html lang="en">
      <head>
        <meta charSet="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <Head />  {/* Required - handles meta tags, title, etc. */}
      </head>
      <body>
        <Outlet />
        <Scripts />  {/* Required - handles hydration scripts */}
        <TanStackRouterDevtools position="bottom-right" />
      </body>
    </html>
  );
}
```

---

## Server Functions

Server functions allow you to run code exclusively on the server while being callable from anywhere (client or server) with full type safety.

### Four Types of Server Functions

TanStack Start provides four different function types for controlling where code executes:

```typescript
import {
  createServerFn,        // Server code, callable from anywhere
  createServerOnlyFn,    // Server only, throws if called from client
  createClientOnlyFn,    // Client only, throws if called from server
  createIsomorphicFn     // Different behavior on client vs server
} from '@tanstack/start';
```

| Function | Server | Client | Use Case |
|----------|--------|--------|----------|
| `createServerFn` | ✅ Runs directly | ✅ Makes fetch request | Database queries, API calls |
| `createServerOnlyFn` | ✅ Runs directly | ❌ Throws error | Security-critical code that must never leak |
| `createClientOnlyFn` | ❌ Throws error | ✅ Runs directly | Browser-only APIs (localStorage, etc.) |
| `createIsomorphicFn` | ✅ Custom logic | ✅ Custom logic | Different implementations per environment |

### Basic Server Function (Modern API)

**app/api/posts.ts**
```typescript
import { createServerFn } from '@tanstack/start';
import { z } from 'zod';

// GET - Fetching data
export const getPosts = createServerFn({ method: 'GET' })
  .handler(async () => {
    return await db.query.posts.findMany();
  });

// GET with parameters
export const getPost = createServerFn({ method: 'GET' })
  .validator(z.object({ postId: z.string() }))
  .handler(async ({ data }) => {
    const post = await db.query.posts.findFirst({
      where: eq(posts.id, data.postId)
    });

    if (!post) {
      throw new Error('Post not found');
    }

    return post;
  });

// POST - Mutations with Zod validation
export const createPost = createServerFn({ method: 'POST' })
  .validator(z.object({
    title: z.string().min(1),
    content: z.string().min(10)
  }))
  .handler(async ({ data }) => {
    const post = await db.insert(posts).values(data).returning();
    return post[0];
  });
```

### The `useServerFn` Hook (CRITICAL for Redirects)

When server functions perform redirects, you **MUST** wrap them with `useServerFn`:

```typescript
import { useServerFn } from '@tanstack/start';

// Server function that redirects
const addTodo = createServerFn({ method: 'POST' })
  .validator(z.object({ name: z.string().min(1) }))
  .handler(async ({ data }) => {
    await db.insert(todos).values({ name: data.name, isComplete: false });
    throw redirect({ to: '/' });  // Redirect after creation
  });

// In your component
function AddTodoForm() {
  // ✅ CORRECT - Wrap with useServerFn to handle redirects
  const addTodoFn = useServerFn(addTodo);

  async function handleSubmit(e: FormEvent) {
    e.preventDefault();
    await addTodoFn({ data: { name: inputRef.current.value } });
    // Redirect is handled automatically!
  }

  // ❌ WRONG - Direct call won't handle redirects properly
  // await addTodo({ data: { name } });  // Redirect silently fails!
}
```

### Isomorphic Functions

For code that needs different behavior on client vs server:

```typescript
const getEnvironmentInfo = createIsomorphicFn()
  .server(() => {
    console.log('Running on server');
    return { env: 'server', dbConnected: true };
  })
  .client(() => {
    console.log('Running on client');
    return { env: 'client', dbConnected: false };
  });
```

### Using Server Functions in Routes

**app/routes/posts.tsx**
```typescript
import { createFileRoute } from '@tanstack/react-router';
import { getPosts } from '@/api/posts';

export const Route = createFileRoute('/posts')({
  component: PostsList,
  loader: async () => {
    // Server function works in loaders - makes fetch on client, runs directly on server
    const posts = await getPosts();
    return { posts };
  }
});

function PostsList() {
  // Type-safe access to loader data
  const { posts } = Route.useLoaderData();

  return (
    <div>
      <h1>Posts</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Data Loading Patterns

### Route Loaders

Loaders fetch data before rendering the route component.

```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/users/$userId')({
  component: UserProfile,
  loader: async ({ params }) => {
    const user = await getUser(params.userId);
    const posts = await getUserPosts(params.userId);

    return { user, posts };
  }
});

function UserProfile() {
  const { user, posts } = Route.useLoaderData();

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>

      <h2>Posts</h2>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Loading States

```typescript
export const Route = createFileRoute('/dashboard')({
  component: Dashboard,
  pendingComponent: () => <div>Loading dashboard...</div>,
  errorComponent: ({ error }) => (
    <div>Error loading dashboard: {error.message}</div>
  ),
  loader: async () => {
    const data = await getDashboardData();
    return data;
  }
});
```

### Parallel Data Loading

```typescript
export const Route = createFileRoute('/overview')({
  component: Overview,
  loader: async () => {
    // Load multiple things in parallel
    const [stats, users, posts] = await Promise.all([
      getStats(),
      getUsers(),
      getPosts()
    ]);

    return { stats, users, posts };
  }
});
```

---

## Integration with TanStack Query

TanStack Start works seamlessly with TanStack Query for advanced caching and synchronization.

### Setup Query Client

**app/routes/__root.tsx**
```typescript
import { createRootRoute, Outlet } from '@tanstack/react-router';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
    },
  },
});

export const Route = createRootRoute({
  component: RootComponent
});

function RootComponent() {
  return (
    <QueryClientProvider client={queryClient}>
      <Outlet />
    </QueryClientProvider>
  );
}
```

### Using Queries in Routes

```typescript
import { createFileRoute } from '@tanstack/react-router';
import { useQuery } from '@tanstack/react-query';
import { getPosts } from '@/api/posts';

export const Route = createFileRoute('/posts')({
  component: PostsList
});

function PostsList() {
  const { data: posts, isLoading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: () => getPosts()
  });

  if (isLoading) return <div>Loading posts...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {posts?.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

### Prefetching in Loaders

```typescript
import { createFileRoute } from '@tanstack/react-router';
import { QueryClient } from '@tanstack/react-query';

export const Route = createFileRoute('/posts')({
  component: PostsList,
  loader: async ({ context }) => {
    const queryClient = context.queryClient as QueryClient;

    // Prefetch data for instant render
    await queryClient.prefetchQuery({
      queryKey: ['posts'],
      queryFn: () => getPosts()
    });

    return {};
  }
});
```

---

## Search Params and Validation

### Type-Safe Search Params

```typescript
import { createFileRoute } from '@tanstack/react-router';
import { z } from 'zod';

const searchSchema = z.object({
  page: z.number().default(1),
  sort: z.enum(['name', 'date']).default('name'),
  filter: z.string().optional()
});

export const Route = createFileRoute('/products')({
  component: ProductsList,
  validateSearch: (search) => searchSchema.parse(search)
});

function ProductsList() {
  const { page, sort, filter } = Route.useSearch();
  const navigate = Route.useNavigate();

  return (
    <div>
      <select
        value={sort}
        onChange={(e) => navigate({
          search: (prev) => ({ ...prev, sort: e.target.value as 'name' | 'date' })
        })}
      >
        <option value="name">Sort by Name</option>
        <option value="date">Sort by Date</option>
      </select>

      <p>Page: {page}, Sort: {sort}</p>
    </div>
  );
}
```

---

## Form Handling

### Server Actions with Forms

**app/routes/posts.new.tsx**
```typescript
import { createFileRoute, useNavigate } from '@tanstack/react-router';
import { createPost } from '@/api/posts';
import { useMutation } from '@tanstack/react-query';

export const Route = createFileRoute('/posts/new')({
  component: NewPost
});

function NewPost() {
  const navigate = useNavigate();

  const createMutation = useMutation({
    mutationFn: createPost,
    onSuccess: (post) => {
      navigate({ to: `/posts/${post.id}` });
    }
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);

    createMutation.mutate({
      title: formData.get('title') as string,
      content: formData.get('content') as string
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />

      <button type="submit" disabled={createMutation.isPending}>
        {createMutation.isPending ? 'Creating...' : 'Create Post'}
      </button>

      {createMutation.error && (
        <p>Error: {createMutation.error.message}</p>
      )}
    </form>
  );
}
```

### Form with React Hook Form

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const postSchema = z.object({
  title: z.string().min(3),
  content: z.string().min(10)
});

type PostFormData = z.infer<typeof postSchema>;

function PostForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<PostFormData>({
    resolver: zodResolver(postSchema)
  });

  const createMutation = useMutation({
    mutationFn: createPost
  });

  const onSubmit = (data: PostFormData) => {
    createMutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('title')} />
      {errors.title && <span>{errors.title.message}</span>}

      <textarea {...register('content')} />
      {errors.content && <span>{errors.content.message}</span>}

      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Data Mutations & Invalidation

### Invalidating Route Data with `useRouter`

After mutations, invalidate route data to refresh the UI:

```typescript
import { useRouter } from '@tanstack/react-router';

function TodoItem({ todo }) {
  const router = useRouter();

  const toggleTodo = createServerFn({ method: 'POST' })
    .validator(z.object({ id: z.string(), isComplete: z.boolean() }))
    .handler(async ({ data }) => {
      await db.update(todos)
        .set({ isComplete: data.isComplete })
        .where(eq(todos.id, data.id));
    });

  const toggleFn = useServerFn(toggleTodo);

  async function handleToggle() {
    await toggleFn({ data: { id: todo.id, isComplete: !todo.isComplete } });
    // Invalidate route data to refresh from server
    await router.invalidate();
  }

  return (
    <Checkbox
      checked={todo.isComplete}
      onCheckedChange={handleToggle}
    />
  );
}
```

### Optimistic Updates with `useOptimistic`

For instant UI feedback while waiting for server response:

```typescript
import { useOptimistic } from 'react';

function TodoList() {
  const { todos } = Route.useLoaderData();
  const router = useRouter();

  // Create optimistic state that updates immediately
  const [optimisticTodos, setOptimisticTodos] = useOptimistic(todos);

  const toggleFn = useServerFn(toggleTodo);

  async function handleToggle(todo: Todo) {
    // 1. Update UI immediately (optimistic)
    setOptimisticTodos(prev =>
      prev.map(t =>
        t.id === todo.id ? { ...t, isComplete: !t.isComplete } : t
      )
    );

    // 2. Send to server
    await toggleFn({ data: { id: todo.id, isComplete: !todo.isComplete } });

    // 3. Revalidate to sync with server
    await router.invalidate();
  }

  return (
    <ul>
      {optimisticTodos.map(todo => (
        <li key={todo.id}>
          <Checkbox
            checked={todo.isComplete}
            onCheckedChange={() => handleToggle(todo)}
          />
          {todo.name}
        </li>
      ))}
    </ul>
  );
}
```

### Delete with Confirmation

```typescript
const deleteTodo = createServerFn({ method: 'POST' })
  .validator(z.object({ id: z.string() }))
  .handler(async ({ data }) => {
    await db.delete(todos).where(eq(todos.id, data.id));
  });

function DeleteButton({ todoId }) {
  const router = useRouter();
  const deleteFn = useServerFn(deleteTodo);
  const [isPending, setIsPending] = useState(false);

  async function handleDelete() {
    if (!confirm('Delete this todo?')) return;

    setIsPending(true);
    await deleteFn({ data: { id: todoId } });
    await router.invalidate();
    setIsPending(false);
  }

  return (
    <Button onClick={handleDelete} disabled={isPending}>
      {isPending ? 'Deleting...' : 'Delete'}
    </Button>
  );
}
```

---

## Authentication Patterns

### Protected Routes

**app/routes/_auth.tsx** - Authentication layout
```typescript
import { createFileRoute, Outlet, redirect } from '@tanstack/react-router';
import { getSession } from '@/api/auth';

export const Route = createFileRoute('/_auth')({
  component: AuthLayout,
  beforeLoad: async () => {
    const session = await getSession();

    if (!session) {
      throw redirect({
        to: '/login',
        search: {
          redirect: window.location.pathname
        }
      });
    }

    return { session };
  }
});

function AuthLayout() {
  const { session } = Route.useRouteContext();

  return (
    <div>
      <header>
        <p>Logged in as {session.user.email}</p>
      </header>
      <Outlet />
    </div>
  );
}
```

**app/routes/_auth/dashboard.tsx** - Protected route
```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/_auth/dashboard')({
  component: Dashboard
});

function Dashboard() {
  const { session } = Route.useRouteContext();

  return <h1>Welcome, {session.user.name}!</h1>;
}
```

### Login Route

**app/routes/login.tsx**
```typescript
import { createFileRoute, useNavigate } from '@tanstack/react-router';
import { login } from '@/api/auth';
import { useMutation } from '@tanstack/react-query';

const loginSearchSchema = z.object({
  redirect: z.string().optional()
});

export const Route = createFileRoute('/login')({
  component: Login,
  validateSearch: (search) => loginSearchSchema.parse(search)
});

function Login() {
  const navigate = useNavigate();
  const { redirect } = Route.useSearch();

  const loginMutation = useMutation({
    mutationFn: login,
    onSuccess: () => {
      navigate({ to: redirect || '/dashboard' });
    }
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);

    loginMutation.mutate({
      email: formData.get('email') as string,
      password: formData.get('password') as string
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" placeholder="Email" required />
      <input name="password" type="password" placeholder="Password" required />
      <button type="submit">Login</button>

      {loginMutation.error && (
        <p>Error: {loginMutation.error.message}</p>
      )}
    </form>
  );
}
```

---

## Navigation

### Link Component

```typescript
import { Link } from '@tanstack/react-router';

function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link
        to="/posts/$postId"
        params={{ postId: '123' }}
      >
        Post 123
      </Link>

      {/* With search params */}
      <Link
        to="/products"
        search={{ page: 1, sort: 'name' }}
      >
        Products
      </Link>

      {/* Active state styling */}
      <Link
        to="/dashboard"
        activeProps={{
          className: 'active font-bold'
        }}
      >
        Dashboard
      </Link>
    </nav>
  );
}
```

### Programmatic Navigation

```typescript
import { useNavigate } from '@tanstack/react-router';

function GoToDashboard() {
  const navigate = useNavigate();

  const handleClick = () => {
    navigate({ to: '/dashboard' });
  };

  const handleBack = () => {
    navigate({ to: '..', replace: true });
  };

  return (
    <div>
      <button onClick={handleClick}>Go to Dashboard</button>
      <button onClick={handleBack}>Go Back</button>
    </div>
  );
}
```

---

## Error Handling

### Route-Level Error Boundaries

```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  component: PostDetail,
  errorComponent: ({ error, reset }) => (
    <div>
      <h1>Error Loading Post</h1>
      <p>{error.message}</p>
      <button onClick={reset}>Try Again</button>
    </div>
  ),
  loader: async ({ params }) => {
    const post = await getPost(params.postId);

    if (!post) {
      throw new Error('Post not found');
    }

    return { post };
  }
});
```

### Global Error Boundary

**app/routes/__root.tsx**
```typescript
import { createRootRoute, ErrorComponent } from '@tanstack/react-router';

export const Route = createRootRoute({
  component: RootComponent,
  errorComponent: ({ error, reset }) => (
    <div>
      <h1>Something went wrong</h1>
      <p>{error.message}</p>
      <button onClick={reset}>Reset</button>
    </div>
  )
});
```

---

## Middleware and Context

### Adding Context to Routes

```typescript
import { createFileRoute } from '@tanstack/react-router';
import { queryClient } from '@/lib/query-client';
import { getSession } from '@/api/auth';

export const Route = createFileRoute('/__root')({
  beforeLoad: async () => {
    const session = await getSession();

    return {
      queryClient,
      session
    };
  }
});

// Access in child routes
function SomeComponent() {
  const { session } = Route.useRouteContext();
  return <p>User: {session?.user.email}</p>;
}
```

---

## Deployment

### Build for Production

```bash
# Build with Vite
npm run build

# Preview production build
npm run preview
```

### Deployment Platforms

**Vercel:**
```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".output/public",
  "devCommand": "npm run dev"
}
```

**Netlify:**
```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = ".output/public"
```

**Docker:**
```dockerfile
FROM node:20-alpine
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

EXPOSE 3000
CMD ["npm", "start"]
```

---

## Environment Variables

```typescript
// app/config.ts
export const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  supabaseUrl: import.meta.env.VITE_SUPABASE_URL,
  supabaseKey: import.meta.env.VITE_SUPABASE_ANON_KEY
} as const;

// Only accessible on server
import { serverConfig } from './server-config';

export const getServerConfig = createServerFn('GET', () => {
  return {
    databaseUrl: process.env.DATABASE_URL,
    secretKey: process.env.SECRET_KEY
  };
});
```

---

## Best Practices

### 1. Type Safety
- Always use TypeScript for route params, search params, and loader data
- Use Zod schemas for search param validation
- Leverage type inference from server functions

### 2. Data Loading
- Use loaders for data needed before render
- Prefetch data with TanStack Query for instant navigation
- Load non-critical data client-side with useQuery

### 3. Code Splitting
- Routes are automatically code-split
- Use dynamic imports for heavy components
```typescript
const HeavyChart = lazy(() => import('@/components/HeavyChart'));
```

### 4. Performance
- Enable streaming SSR for faster time-to-interactive
- Use `pendingComponent` for loading states
- Implement optimistic updates with TanStack Query

### 5. Security
- Keep sensitive logic in server functions
- Validate all inputs with Zod schemas
- Use CSRF protection for mutations

---

## Critical: Server Functions vs API Routes (Prisma/Better Auth)

### The Problem

> **WARNING:** DO NOT use `createServerFn` in route files that import Prisma, Better Auth, or other server-only packages.

TanStack Start server functions defined in route files (`.tsx`) can leak server-only code into the client bundle, causing errors like:

```
node:module has been externalized for browser compatibility
```

or

```
ReferenceError: module is not defined (at @prisma/client)
```

This happens because Vite/TanStack Start doesn't fully tree-shake server function imports from route files, and Prisma uses CommonJS which doesn't work well with ESM bundling.

### The Solution: Use API Routes

Instead of server functions in route files, create dedicated **API routes** that are purely server-side:

**API Route (`src/routes/api/settings.ts`):**
```typescript
import { createFileRoute } from "@tanstack/react-router";
import { auth } from "@/lib/auth";
import { db } from "@/lib/db";

export const Route = createFileRoute("/api/settings")({
  server: {
    handlers: {
      GET: async ({ request }) => {
        const session = await auth.api.getSession({
          headers: request.headers,
        });

        if (!session?.user?.id) {
          return Response.json(
            { error: "Not authenticated" },
            { status: 401 },
          );
        }

        const data = await db.someModel.findMany();
        return Response.json({ data });
      },

      POST: async ({ request }) => {
        const session = await auth.api.getSession({
          headers: request.headers,
        });

        if (!session?.user?.id) {
          return Response.json(
            { error: "Not authenticated" },
            { status: 401 },
          );
        }

        const body = await request.json();
        const result = await db.someModel.create({ data: body });
        return Response.json({ success: true, result });
      },
    },
  },
});
```

**Client Route (`src/routes/_app/settings.tsx`):**
```typescript
import { createFileRoute } from "@tanstack/react-router";
import { useEffect, useState } from "react";
// Only import client-safe types/defaults (no Prisma, no auth)
import { defaultSettings, type SettingsInput } from "@/lib/settings.shared";

export const Route = createFileRoute("/_app/settings")({
  component: SettingsPage,
});

function SettingsPage() {
  const [settings, setSettings] = useState(defaultSettings);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetch("/api/settings", { credentials: "include" })
      .then(res => res.json())
      .then(data => {
        if (data.settings) setSettings(data.settings);
      })
      .finally(() => setIsLoading(false));
  }, []);

  const handleSave = async () => {
    await fetch("/api/settings", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include",
      body: JSON.stringify(settings),
    });
  };

  // ... render UI
}
```

### When Server Functions ARE Safe

Server functions (`createServerFn`) work fine when:
1. They only import packages that work in both server and client environments
2. They're defined in separate files that don't import Prisma/auth (but this is risky)
3. Used for simple operations without database/auth dependencies

### Key Takeaways

| Do This | Not This |
|---------|----------|
| API routes at `src/routes/api/*.ts` | Server functions in route `.tsx` files |
| Import Prisma/auth only in API routes | Import Prisma/auth in route components |
| Use `fetch()` from client routes | Use `createServerFn` with Prisma imports |
| Create shared type files (`*.shared.ts`) | Mix server/client types in same file |
| Use `Response.json()` (native Web API) | Use `json()` import from tanstack |
| Use `credentials: "include"` for auth cookies | Forget credentials on fetch calls |

---

## Common Gotchas & Troubleshooting

### TypeScript Errors on First Run

**Problem:** After creating a new project, `routeTree.gen.ts` shows errors.

**Solution:**
1. Run `npm run dev` to generate the route tree
2. Restart TypeScript server: `Ctrl+Shift+P` → "TypeScript: Restart TS Server"

The `routeTree.gen.ts` file is auto-generated while the dev server runs. It won't exist until you start the server.

### Redirects Not Working from Server Functions

**Problem:** Server function calls `throw redirect()` but nothing happens.

**Solution:** Wrap with `useServerFn`:
```typescript
// ❌ Wrong - redirect silently fails
await addTodo({ data: { name } });

// ✅ Correct - redirect works
const addTodoFn = useServerFn(addTodo);
await addTodoFn({ data: { name } });
```

### Hot Reload Not Working for Loaders

**Problem:** Changing loader code doesn't update the page.

**Why:** Loaders run when navigating to a route. If you're already on the route, it won't re-run.

**Solution:** Manually refresh the page after changing loader code.

### Required Root Layout Components

**Problem:** App broken after modifying `__root.tsx`.

**Solution:** Ensure these components are present:
```typescript
// __root.tsx
import { Head, Scripts } from '@tanstack/start';

function RootComponent() {
  return (
    <html>
      <head>
        <Head />  {/* Required! */}
      </head>
      <body>
        <Outlet />
        <Scripts />  {/* Required! */}
      </body>
    </html>
  );
}
```

Removing either `<Head />` or `<Scripts />` breaks the application.

### Database Code Errors on Client

**Problem:** Console shows Postgres/database errors when navigating.

**Why:** Loader runs on both client AND server. Database code can't run on client.

**Solution:** Wrap all database calls in server functions:
```typescript
// ❌ Wrong - breaks on client
loader: async () => {
  return await db.query.posts.findMany();
}

// ✅ Correct - works everywhere
const getPosts = createServerFn({ method: 'GET' }).handler(async () => {
  return await db.query.posts.findMany();
});

loader: async () => {
  return await getPosts();
}
```

### Understanding `cta.json`

The `cta.json` file (Create TanStack Application) stores your CLI choices:
- Selected add-ons (Drizzle, Shadcn, etc.)
- Package manager preference
- Project configuration

You generally don't need to modify this file.

---

## Common Patterns

### Pagination

```typescript
const Route = createFileRoute('/posts')({
  component: PostsList,
  validateSearch: z.object({
    page: z.number().default(1),
    limit: z.number().default(10)
  }),
  loader: async ({ search }) => {
    const posts = await getPosts({
      page: search.page,
      limit: search.limit
    });

    return { posts };
  }
});

function PostsList() {
  const { posts } = Route.useLoaderData();
  const { page } = Route.useSearch();
  const navigate = Route.useNavigate();

  return (
    <div>
      {posts.map(post => <PostCard key={post.id} post={post} />)}

      <button onClick={() => navigate({ search: { page: page - 1 } })}>
        Previous
      </button>
      <button onClick={() => navigate({ search: { page: page + 1 } })}>
        Next
      </button>
    </div>
  );
}
```

### Infinite Scroll

```typescript
import { useInfiniteQuery } from '@tanstack/react-query';

function InfinitePostsList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage
  } = useInfiniteQuery({
    queryKey: ['posts', 'infinite'],
    queryFn: ({ pageParam = 0 }) => getPosts({ offset: pageParam }),
    getNextPageParam: (lastPage, pages) =>
      lastPage.length === 10 ? pages.length * 10 : undefined
  });

  return (
    <div>
      {data?.pages.map((page) => (
        page.map(post => <PostCard key={post.id} post={post} />)
      ))}

      {hasNextPage && (
        <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
          {isFetchingNextPage ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

### Modal Routes

```typescript
// app/routes/posts.$postId.modal.tsx
export const Route = createFileRoute('/posts/$postId/modal')({
  component: PostModal
});

function PostModal() {
  const navigate = useNavigate();
  const { postId } = Route.useParams();
  const { data: post } = useQuery({
    queryKey: ['post', postId],
    queryFn: () => getPost(postId)
  });

  return (
    <div className="modal-overlay" onClick={() => navigate({ to: '/posts' })}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <h1>{post?.title}</h1>
        <p>{post?.content}</p>
        <button onClick={() => navigate({ to: '/posts' })}>Close</button>
      </div>
    </div>
  );
}
```

---

## Resources

- **Documentation:** https://tanstack.com/start
- **Router Docs:** https://tanstack.com/router
- **Query Docs:** https://tanstack.com/query
- **Examples:** https://github.com/TanStack/router/tree/main/examples
