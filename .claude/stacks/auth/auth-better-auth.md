# Better Auth - Authentication Patterns

**Purpose:** Comprehensive guide for implementing authentication with Better Auth
**Last Updated:** 2025-11-25
**Current Version:** 1.4
**Official Docs:** https://www.better-auth.com

---

## Overview

Better Auth is a modern, framework-agnostic authentication library for TypeScript with a plugin-based architecture.

**Key Features:**
- Email & password authentication
- Social OAuth providers (Google, GitHub, Apple, etc.)
- Extensible via plugins (Magic Link, Email OTP, Passkey, 2FA, etc.)
- Session management with secure defaults (now supports stateless mode!)
- CSRF protection built-in
- Framework adapters (Next.js, Remix, SvelteKit, TanStack Start, etc.)
- Custom OAuth state passing for referral tracking
- SCIM provisioning for enterprise identity management

---

## Installation & Setup

**📚 Official Documentation**: [Better Auth - Installation](https://www.better-auth.com/docs/installation)

Follow the official installation guide for setup instructions specific to your framework (Next.js, Remix, SvelteKit, etc.) and database.

### Quick Reference

The general setup process involves:
1. Install Better Auth package
2. Configure environment variables (secret, URL, database)
3. Create auth instance with configuration
4. Generate database schema using CLI (optional - see Stateless Mode below)
5. Set up server handler for your framework
6. Create client instance

**Note**: Installation commands and CLI tools may change. Always refer to the official docs above for the current setup method.

### NEW in v1.4: Stateless Authentication

You can now omit the database configuration entirely for session management without persistence:

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  // No database required!
  emailAndPassword: {
    enabled: true,
  },
  // Sessions managed via JWT only
});
```

**Stateless endpoints available:**
- `GET /api/auth/access-token` - Get access token
- `GET /api/auth/account-info` - Get account information
- `GET /api/auth/refresh-token` - Refresh tokens

**When to use stateless mode:**
- Serverless/edge environments where database connections are expensive
- High-scale applications prioritizing speed over centralized session control
- Microservices architectures with token-based auth

**Trade-offs:**
- ✅ No database overhead, faster responses
- ❌ Can't centrally revoke sessions (must wait for token expiry)
- ❌ No session activity tracking

### Configuration Example

Create `lib/auth.ts` (or `auth.ts` in project root):

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  // Database configuration (optional in v1.4+, omit for stateless mode)
  database: {
    // Auto-configured if using DATABASE_URL env var
    // Or manually specify connection details
  },

  // Email & password authentication
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 8,
    maxPasswordLength: 128,
    autoSignIn: true, // Auto sign in after registration
  },

  // Session configuration
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 days (in seconds)
    updateAge: 60 * 60 * 24, // 1 day (extend expiry after this duration)
    cookieCache: {
      enabled: true, // NEW in v1.4: JWE cookie caching for enhanced security
      maxAge: 60 * 5, // 5 minutes
    },
  },

  // NEW in v1.4: Advanced options
  advanced: {
    generateId: "uuid", // v1.4: Now supports UUID for primary IDs (alongside "nanoid")
    cookieChunking: {
      enabled: true, // v1.4: Automatic cookie chunking to prevent size errors
    },
  },

  // CSRF protection (automatic, but can configure trusted origins)
  trustedOrigins: [
    "http://localhost:3000",
    "https://yourdomain.com"
  ],
});
```

### 4. Create Database Tables

Better Auth CLI can generate and apply migrations:

**Database Setup:**
Use the Better Auth CLI to generate and apply database schema. See the [official docs](https://www.better-auth.com/docs/installation) for CLI commands.

---

## Framework Integration

### Next.js App Router

**API Route Handler** - Create `app/api/auth/[...all]/route.ts`:

```ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET } = toNextJsHandler(auth);
```

### TanStack Start

**API Route** - Create appropriate route file:

```ts
import { auth } from "@/lib/auth";

export async function loader({ request }: LoaderFunctionArgs) {
  return auth.handler(request);
}

export async function action({ request }: ActionFunctionArgs) {
  return auth.handler(request);
}
```

### Remix

**Route File** - Create `app/routes/api.auth.$.ts`:

```ts
import { auth } from "~/lib/auth.server";
import type { LoaderFunctionArgs, ActionFunctionArgs } from "@remix-run/node";

export async function loader({ request }: LoaderFunctionArgs) {
  return auth.handler(request);
}

export async function action({ request }: ActionFunctionArgs) {
  return auth.handler(request);
}
```

---

## Client Setup

### Create Auth Client

Create `lib/auth-client.ts`:

```ts
import { createAuthClient } from "better-auth/react"; // or better-auth/vue, better-auth/solid

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000",
});

// Export useful hooks/methods
export const {
  signIn,
  signUp,
  signOut,
  useSession,
  // ... other exports
} = authClient;
```

### React Context Provider (Optional but Recommended)

Create `components/auth/AuthProvider.tsx`:

```tsx
"use client";

import { SessionProvider } from "better-auth/react";

export function AuthProvider({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>;
}
```

Wrap your app in `layout.tsx`:

```tsx
import { AuthProvider } from "@/components/auth/AuthProvider";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  );
}
```

---

## Authentication Patterns

### Sign Up (Email & Password)

```tsx
"use client";

import { useState } from "react";
import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";

export function SignUpForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [name, setName] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const handleSignUp = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    const { data, error: signUpError } = await authClient.signUp.email(
      {
        email,
        password,
        name,
        callbackURL: "/dashboard", // Redirect after email verification
      },
      {
        onRequest: () => setLoading(true),
        onSuccess: () => {
          router.push("/dashboard");
        },
        onError: (ctx) => {
          setError(ctx.error.message);
          setLoading(false);
        },
      }
    );
  };

  return (
    <form onSubmit={handleSignUp}>
      <input
        type="text"
        placeholder="Name"
        value={name}
        onChange={(e) => setName(e.target.value)}
        required
      />
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
      />
      <input
        type="password"
        placeholder="Password (min 8 characters)"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        required
        minLength={8}
      />
      {error && <p className="text-red-500">{error}</p>}
      <button type="submit" disabled={loading}>
        {loading ? "Signing up..." : "Sign Up"}
      </button>
    </form>
  );
}
```

### Sign In (Email & Password)

```tsx
"use client";

import { useState } from "react";
import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";

export function SignInForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const handleSignIn = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    const { data, error: signInError } = await authClient.signIn.email(
      {
        email,
        password,
        callbackURL: "/dashboard",
      },
      {
        onRequest: () => setLoading(true),
        onSuccess: () => {
          router.push("/dashboard");
        },
        onError: (ctx) => {
          setError(ctx.error.message);
          setLoading(false);
        },
      }
    );
  };

  return (
    <form onSubmit={handleSignIn}>
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        required
      />
      {error && <p className="text-red-500">{error}</p>}
      <button type="submit" disabled={loading}>
        {loading ? "Signing in..." : "Sign In"}
      </button>
    </form>
  );
}
```

### Sign Out

```tsx
"use client";

import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";

export function SignOutButton() {
  const router = useRouter();

  const handleSignOut = async () => {
    await authClient.signOut({
      fetchOptions: {
        onSuccess: () => {
          router.push("/login");
        },
      },
    });
  };

  return (
    <button onClick={handleSignOut}>
      Sign Out
    </button>
  );
}
```

### Using Session Data

```tsx
"use client";

import { useSession } from "@/lib/auth-client";

export function UserProfile() {
  const { data: session, isPending, error } = useSession();

  if (isPending) {
    return <div>Loading...</div>;
  }

  if (error || !session) {
    return <div>Not authenticated</div>;
  }

  return (
    <div>
      <h2>Welcome, {session.user.name}!</h2>
      <p>Email: {session.user.email}</p>
      {session.user.image && (
        <img src={session.user.image} alt={session.user.name} />
      )}
    </div>
  );
}
```

---

## Social OAuth Providers

### Configuration

Add OAuth providers in `lib/auth.ts`:

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  emailAndPassword: {
    enabled: true,
  },

  // Social providers
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
    // More providers: apple, discord, facebook, microsoft, spotify, twitter, etc.
  },
});
```

### Environment Variables

Add to `.env`:

```env
# Google OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# GitHub OAuth
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
```

### OAuth Sign In Button

```tsx
"use client";

import { authClient } from "@/lib/auth-client";

export function GoogleSignInButton() {
  const handleGoogleSignIn = async () => {
    await authClient.signIn.social({
      provider: "google",
      callbackURL: "/dashboard",
    });
  };

  return (
    <button onClick={handleGoogleSignIn}>
      Sign in with Google
    </button>
  );
}

export function GitHubSignInButton() {
  const handleGitHubSignIn = async () => {
    await authClient.signIn.social({
      provider: "github",
      callbackURL: "/dashboard",
    });
  };

  return (
    <button onClick={handleGitHubSignIn}>
      Sign in with GitHub
    </button>
  );
}
```

### NEW in v1.4: Custom OAuth State

Pass custom data through OAuth flows (useful for referral tracking, source attribution, etc.):

```tsx
"use client";

import { authClient } from "@/lib/auth-client";

export function GoogleSignInWithReferral({ referralCode }: { referralCode?: string }) {
  const handleGoogleSignIn = async () => {
    await authClient.signIn.social({
      provider: "google",
      callbackURL: "/dashboard",
      // NEW: Pass custom state through OAuth flow
      state: {
        referralCode,
        source: "landing-page",
        campaign: "spring-2024",
      },
    });
  };

  return (
    <button onClick={handleGoogleSignIn}>
      Sign in with Google
    </button>
  );
}
```

**Access custom state in hooks/middleware:**

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
  },
  hooks: {
    after: [
      {
        matcher: (context) => context.path === "/sign-in/social",
        handler: async (context) => {
          // Access custom OAuth state
          const { referralCode, source, campaign } = context.body.state || {};

          // Store referral info, track attribution, etc.
          if (referralCode && context.session?.user) {
            await db.referral.create({
              userId: context.session.user.id,
              referralCode,
              source,
              campaign,
            });
          }
        },
      },
    ],
  },
});
```

---

## Plugins

Better Auth uses a plugin system for extended functionality.

**⚠️ BREAKING CHANGE in v1.4:** Passkey plugin moved to separate package `@better-auth/passkey`

### NEW in v1.4: Device Authorization Plugin

Enables OAuth 2.0 Device Authorization Grant for input-constrained devices (smart TVs, IoT devices, CLI tools):

```ts
import { betterAuth } from "better-auth";
import { deviceAuthorization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    deviceAuthorization({
      // Configuration options
      deviceCodeExpiresIn: 600, // 10 minutes
      interval: 5, // Polling interval in seconds
    }),
  ],
});
```

**Client usage (for device):**

```ts
// Step 1: Request device code
const { deviceCode, userCode, verificationUri } = await authClient.deviceAuthorization.request({
  clientId: "your-client-id",
});

// Display to user:
// "Go to https://yourapp.com/activate"
// "Enter code: ABCD-1234"
console.log(`Go to ${verificationUri} and enter: ${userCode}`);

// Step 2: Poll for authorization
const { accessToken } = await authClient.deviceAuthorization.poll({
  deviceCode,
});
```

### NEW in v1.4: SCIM Provisioning Plugin

For enterprise identity management and multi-domain scenarios:

```ts
import { betterAuth } from "better-auth";
import { scim } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    scim({
      // SCIM 2.0 endpoints for user provisioning
      baseUrl: "/scim/v2",
    }),
  ],
});
```

**Supported SCIM operations:**
- User creation and updates
- Group management
- Bulk operations
- Resource filtering

**Use cases:**
- Enterprise SSO integration
- Automated user provisioning from identity providers
- Multi-tenant applications with centralized identity management

### Email OTP Plugin

**Server Configuration:**

```ts
import { betterAuth } from "better-auth";
import { emailOTP } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    emailOTP({
      async sendVerificationOTP({ email, otp, type }) {
        // Send OTP via your email service (Resend, SendGrid, etc.)
        await sendEmail({
          to: email,
          subject: type === "sign-in" ? "Sign In OTP" : "Verify Email",
          html: `Your verification code is: <strong>${otp}</strong>`,
        });
      },
      otpLength: 6, // Default is 6
      expiresIn: 300, // 5 minutes (in seconds)
    }),
  ],
});
```

**Client Usage:**

```tsx
import { authClient } from "@/lib/auth-client";

// Send OTP
await authClient.signIn.emailOtp({
  email: "user@example.com",
});

// Verify OTP
await authClient.verifyEmailOtp({
  email: "user@example.com",
  otp: "123456",
});
```

### Magic Link Plugin

**Server Configuration:**

```ts
import { magicLink } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    magicLink({
      async sendMagicLink({ email, url, token }) {
        // Send magic link via email
        await sendEmail({
          to: email,
          subject: "Sign in to your account",
          html: `Click here to sign in: <a href="${url}">Sign In</a>`,
        });
      },
      expiresIn: 300, // 5 minutes
    }),
  ],
});
```

**Client Usage:**

```tsx
await authClient.signIn.magicLink({
  email: "user@example.com",
  callbackURL: "/dashboard",
});
```

### Two-Factor Authentication (2FA)

**Server Configuration:**

```ts
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    twoFactor({
      issuer: "YourAppName", // Shows in authenticator apps
      totpOptions: {
        digits: 6,
        period: 30, // seconds
      },
    }),
  ],
});
```

**Client Usage:**

```tsx
// Enable 2FA for user
const { data } = await authClient.twoFactor.enable({
  password: "user-password", // Verify user identity
});

// data.qrCode - Display QR code for user to scan
// data.backupCodes - Show backup codes (save these!)

// Verify TOTP during setup
await authClient.twoFactor.verifyTotp({
  code: "123456",
});

// Sign in with 2FA
await authClient.signIn.email({
  email: "user@example.com",
  password: "password",
});

// If 2FA is enabled, you'll need to verify:
await authClient.twoFactor.verifyTotp({
  code: "123456",
});
```

---

## Server-Side Session Access

### In API Routes

```ts
import { auth } from "@/lib/auth";
import { headers } from "next/headers";

export async function GET(request: Request) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    return new Response("Unauthorized", { status: 401 });
  }

  // Use session.user data
  const userId = session.user.id;

  // Your logic here
  return Response.json({ user: session.user });
}
```

### In Server Components

```tsx
import { auth } from "@/lib/auth";
import { headers } from "next/headers";

export default async function DashboardPage() {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    redirect("/login");
  }

  return (
    <div>
      <h1>Welcome, {session.user.name}!</h1>
    </div>
  );
}
```

### In Server Actions

```ts
"use server";

import { auth } from "@/lib/auth";
import { headers } from "next/headers";

export async function updateProfile(formData: FormData) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    throw new Error("Unauthorized");
  }

  const name = formData.get("name") as string;

  // Update user in database
  await db.user.update({
    where: { id: session.user.id },
    data: { name },
  });

  return { success: true };
}
```

---

## Security Best Practices

### 1. Password Hashing

Better Auth uses **scrypt** (OWASP recommended) for password hashing by default. It's memory-hard and CPU-intensive, resisting brute-force attacks.

**Custom Hasher (optional):**

```ts
export const auth = betterAuth({
  emailAndPassword: {
    enabled: true,
    password: {
      hash: async (password) => {
        // Custom hashing logic
        return hashedPassword;
      },
      verify: async (password, hash) => {
        // Custom verification logic
        return isValid;
      },
    },
  },
});
```

### 2. Session Management

**Default Security:**
- Sessions stored in database (not JWT in localStorage)
- Secure, HTTP-only cookies
- CSRF protection via Origin header validation
- Session expiration and auto-renewal

**Custom Session Duration:**

```ts
export const auth = betterAuth({
  session: {
    expiresIn: 60 * 60 * 24 * 30, // 30 days
    updateAge: 60 * 60 * 24 * 7, // Extend after 7 days of activity
  },
});
```

### 3. CSRF Protection

Better Auth validates the `Origin` header automatically. Configure trusted origins:

```ts
export const auth = betterAuth({
  trustedOrigins: [
    "http://localhost:3000",
    "https://yourdomain.com",
    "https://staging.yourdomain.com",
  ],
});
```

### 4. Account Linking

Control how users can link multiple auth methods:

```ts
export const auth = betterAuth({
  account: {
    accountLinking: {
      enabled: true,
      // Auto-link if provider verifies email
      trustedProviders: ["google", "github"], // Force link for these
    },
  },
});
```

**Security Note:** Only link accounts if the OAuth provider confirms email is verified, unless provider is in `trustedProviders`.

### 5. Rate Limiting

Implement rate limiting for auth endpoints (use a separate library):

```ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 m"), // 5 requests per minute
});

// In your auth route
const { success } = await ratelimit.limit(email);
if (!success) {
  return new Response("Too many requests", { status: 429 });
}
```

---

## Password Reset Flow

### Server Configuration

```ts
export const auth = betterAuth({
  emailAndPassword: {
    enabled: true,
    async sendResetPassword({ user, url, token }) {
      // Send password reset email
      await sendEmail({
        to: user.email,
        subject: "Reset your password",
        html: `Click here to reset your password: <a href="${url}">Reset Password</a>`,
      });
    },
    resetPasswordTokenExpiresIn: 3600, // 1 hour (in seconds)
  },
});
```

### Client - Request Reset

```tsx
import { authClient } from "@/lib/auth-client";

export function ForgotPasswordForm() {
  const [email, setEmail] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    // v1.4 BREAKING CHANGE: Renamed from forgetPassword to requestPasswordReset
    await authClient.requestPasswordReset({
      email,
      redirectTo: "/reset-password", // Where to redirect after clicking link
    });

    // Show success message
    alert("Check your email for a password reset link");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
        required
      />
      <button type="submit">Send Reset Link</button>
    </form>
  );
}
```

### Client - Reset Password

```tsx
import { authClient } from "@/lib/auth-client";
import { useSearchParams } from "next/navigation";

export function ResetPasswordForm() {
  const [password, setPassword] = useState("");
  const searchParams = useSearchParams();
  const token = searchParams.get("token");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!token) {
      alert("Invalid reset token");
      return;
    }

    await authClient.resetPassword({
      newPassword: password,
      token,
    });

    // Redirect to login
    router.push("/login");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Enter new password"
        required
        minLength={8}
      />
      <button type="submit">Reset Password</button>
    </form>
  );
}
```

---

## Middleware for Route Protection

### Next.js Middleware

Create `middleware.ts` in project root:

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { auth } from "@/lib/auth";

export async function middleware(request: NextRequest) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  const isAuthPage = request.nextUrl.pathname.startsWith("/login") ||
                     request.nextUrl.pathname.startsWith("/register");

  const isProtectedPage = request.nextUrl.pathname.startsWith("/dashboard");

  // Redirect authenticated users away from auth pages
  if (session && isAuthPage) {
    return NextResponse.redirect(new URL("/dashboard", request.url));
  }

  // Redirect unauthenticated users to login
  if (!session && isProtectedPage) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/login", "/register"],
};
```

---

## Testing

### Mock Auth in Tests

```ts
import { authClient } from "@/lib/auth-client";

// Mock authClient
jest.mock("@/lib/auth-client", () => ({
  authClient: {
    signIn: {
      email: jest.fn(),
    },
    signUp: {
      email: jest.fn(),
    },
    useSession: jest.fn(),
  },
}));

// In your test
it("should sign in user", async () => {
  (authClient.signIn.email as jest.Mock).mockResolvedValue({
    data: { user: { id: "1", email: "test@example.com" } },
    error: null,
  });

  // Your test logic
});
```

---

## Common Patterns

### Protected Component

```tsx
"use client";

import { useSession } from "@/lib/auth-client";
import { useRouter } from "next/navigation";
import { useEffect } from "react";

export function ProtectedContent({ children }: { children: React.ReactNode }) {
  const { data: session, isPending } = useSession();
  const router = useRouter();

  useEffect(() => {
    if (!isPending && !session) {
      router.push("/login");
    }
  }, [session, isPending, router]);

  if (isPending) {
    return <div>Loading...</div>;
  }

  if (!session) {
    return null;
  }

  return <>{children}</>;
}
```

### Role-Based Access

Better Auth doesn't include roles by default, but you can extend it:

```ts
// In your database, add a role column to users table
// Then access it via session

export async function requireRole(role: string) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    throw new Error("Unauthorized");
  }

  // Fetch user role from database
  const user = await db.user.findUnique({
    where: { id: session.user.id },
    select: { role: true },
  });

  if (user?.role !== role) {
    throw new Error("Forbidden");
  }

  return session;
}

// Usage in Server Action
export async function deleteUser(userId: string) {
  "use server";

  await requireRole("admin");

  // Delete user logic
}
```

---

## CLI Commands Reference

**📚 Official Documentation**: [Better Auth - CLI Reference](https://www.better-auth.com/docs/installation)

The Better Auth CLI provides commands for:
- Generating auth secrets
- Generating database schema
- Applying migrations
- Getting diagnostic info
- Initializing new projects

See the official docs for the latest CLI commands and options.

---

## Troubleshooting

### Sessions Not Persisting

**Check:**
1. Are cookies being set? (Check browser DevTools → Application → Cookies)
2. Is `BETTER_AUTH_URL` correct in `.env`?
3. Are you using HTTPS in production? (Required for secure cookies)
4. Is your database connection working?

### OAuth Redirect Not Working

**Check:**
1. Callback URL matches in OAuth provider settings (e.g., `http://localhost:3000/api/auth/callback/google`)
2. Client ID and secret are correct
3. OAuth app is enabled in provider dashboard

### CSRF Errors

**Check:**
1. Is `BETTER_AUTH_URL` set correctly?
2. Is request origin in `trustedOrigins`?
3. Are you sending requests from the same domain?

---

---

## v1.4 Breaking Changes & Migration Guide

If upgrading from an earlier version, note these breaking changes:

### 1. Password Reset API Renamed

```ts
// ❌ OLD (deprecated)
await authClient.forgetPassword({ email });

// ✅ NEW (v1.4+)
await authClient.requestPasswordReset({ email });
```

### 2. Passkey Plugin Moved to Separate Package

```bash
# Install new package
npm install @better-auth/passkey
```

```ts
// ❌ OLD
import { passkey } from "better-auth/plugins";

// ✅ NEW
import { passkey } from "@better-auth/passkey";
```

### 3. Account Info Endpoint Changed

```ts
// ❌ OLD: POST request
POST /api/auth/account-info

// ✅ NEW: GET request
GET /api/auth/account-info
```

### 4. Email Verification Flow Restructured

```ts
// ❌ OLD (removed)
await authClient.sendChangeEmailVerification();

// ✅ NEW
await authClient.emailVerification.sendVerificationEmail();
```

### 5. Configuration Changes

```ts
export const auth = betterAuth({
  advanced: {
    // ❌ OLD (removed)
    generateId: "...",

    // ✅ NEW: Use specific ID type
    generateId: "uuid", // or "nanoid" or "serial"
  },

  // For numeric IDs:
  // ❌ OLD
  advanced: {
    useNumberId: true,
  },

  // ✅ NEW
  advanced: {
    generateId: "serial",
  },
});
```

### 6. OIDC Configuration

```ts
// ❌ OLD
socialProviders: {
  oidc: {
    redirectURLs: [...], // typo
  },
}

// ✅ NEW
socialProviders: {
  oidc: {
    redirectUrls: [...], // correct casing
  },
}
```

**Note:** This requires migration. Update your database and config.

### 7. Plugin Hook Context Changes

Multiple plugins now use `ctx` instead of `request`:

```ts
// ❌ OLD
hooks: {
  after: [{
    handler: async (context) => {
      const data = context.request.body;
    }
  }]
}

// ✅ NEW
hooks: {
  after: [{
    handler: async (ctx) => {
      const data = ctx.body;
    }
  }]
}
```

### 8. API Key Mock Sessions

```ts
// ❌ OLD: Enabled by default

// ✅ NEW: Disabled by default for security
// Explicitly enable if needed for testing:
export const auth = betterAuth({
  plugins: [
    apiKey({
      enableMockSessions: true, // Must be explicitly enabled
    }),
  ],
});
```

---

## Performance Improvements in v1.4

### Database Join Optimization (Experimental)

Enable experimental join optimization for 2-3x faster queries:

```ts
export const auth = betterAuth({
  database: {
    provider: "postgres", // or your DB provider
    // Enable experimental performance improvements
    experimental: {
      useJoins: true, // Improves 50+ endpoints with database joins
    },
  },
});
```

**Benefits:**
- 2-3x latency reduction on session lookups
- Optimized user data fetching
- Reduced database round trips

### Secondary Storage for API Keys

Speed up API key lookups:

```ts
import { apiKey } from "better-auth/plugins";
import { Redis } from "@upstash/redis";

export const auth = betterAuth({
  plugins: [
    apiKey({
      // Use Redis for faster key lookups
      secondaryStorage: {
        get: async (key) => {
          return await redis.get(`api_key:${key}`);
        },
        set: async (key, value, ttl) => {
          await redis.set(`api_key:${key}`, value, { ex: ttl });
        },
      },
    }),
  ],
});
```

### JWT Key Rotation Without Downtime

```ts
export const auth = betterAuth({
  jwt: {
    // Add new key while keeping old one active
    keys: [
      process.env.JWT_SECRET_NEW!, // Will be used for signing
      process.env.JWT_SECRET_OLD!, // Still validates old tokens
    ],
    rotation: {
      enabled: true,
      gracePeriod: 60 * 60 * 24 * 7, // 7 days to fully migrate
    },
  },
});
```

---

## Enhanced Security Features in v1.4

### Enhanced Email Change Security

Require confirmation via current email before sending verification to new email:

```ts
export const auth = betterAuth({
  emailAndPassword: {
    changeEmail: {
      requireCurrentEmailConfirmation: true, // NEW in v1.4
      sendVerificationToNewEmail: true,
      sendNotificationToOldEmail: true,
    },
  },
});
```

### OIDC RP-Initiated Logout

Trigger provider-side logout when user signs out:

```ts
export const auth = betterAuth({
  socialProviders: {
    oidc: {
      // ... your OIDC config
      rpInitiatedLogout: true, // NEW: Logout at provider too
    },
  },
});
```

### Automatic Server-Side IP Detection

```ts
// v1.4: IP detection now automatic from request headers
// No additional configuration needed!

// Automatically reads from:
// - X-Forwarded-For
// - X-Real-IP
// - CF-Connecting-IP (Cloudflare)
// - True-Client-IP (Akamai)
```

---

## Bundle Size Optimization in v1.4

For custom database adapters, use minimal build:

```ts
// ❌ OLD: Full bundle (~500kb)
import { betterAuth } from "better-auth";

// ✅ NEW: Minimal bundle for custom adapters
import { betterAuth } from "better-auth/minimal";

// Only include what you need
export const auth = betterAuth({
  // Custom adapter only
  database: customAdapter,
});
```

**Savings:** Significant bundle size reduction when not using built-in adapters.

---

## Resources

- **Official Docs:** https://www.better-auth.com
- **GitHub:** https://github.com/better-auth/better-auth
- **v1.4 Release Notes:** https://www.better-auth.com/blog/1-4
- **Discord:** https://discord.gg/better-auth (check official docs for link)
- **Examples:** https://github.com/better-auth/better-auth/tree/main/examples

---

*This guide covers the most common Better Auth patterns. For advanced use cases, custom adapters, or specific plugins, consult the official documentation.*
