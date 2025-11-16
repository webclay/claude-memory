# Vercel - Modern Deployment Platform

**Purpose:** Zero-configuration deployment with global CDN, serverless functions, and edge computing

**Last Updated:** 2025-01-16

**Official Docs:** https://vercel.com/docs

---

## What is Vercel?

Vercel is a cloud platform for deploying and scaling modern web applications with:
- **Zero-Config Deploys** - Git push to deploy, automatic builds
- **Global Edge Network** - 126 PoPs across 94 cities in 51 countries
- **Serverless Functions** - Auto-scaling backend without infrastructure management
- **Edge Runtime** - Execute code globally at the edge for ultra-low latency
- **Next.js Native** - Built by the creators of Next.js with first-class support
- **Preview Deployments** - Unique URL for every Git branch/PR
- **Instant Rollbacks** - One-click revert to previous deployments

---

## When to Use Vercel

### ✅ Use Vercel When:
- Building **Next.js applications** (native optimization)
- Need **preview deployments** for every PR
- Want **zero-config CI/CD** from Git
- Building **global applications** requiring edge computing
- Need **automatic scaling** without DevOps overhead
- Want **instant rollbacks** and deployment history
- Building **JAMstack** or static sites with API routes
- Need **image optimization** out of the box

### ❌ Consider Alternatives When:
- Need persistent servers or long-running processes (use traditional hosting)
- Building non-web applications (use AWS/GCP)
- Require custom Docker containers (use Railway, Fly.io)
- Budget-constrained hobby projects with high traffic (Netlify/Cloudflare may be cheaper)

---

## Deployment Methods

### 1. Git Integration (Recommended)

**Connect your Git repository** for automatic deployments:

**Setup:**
1. Visit [vercel.com/new](https://vercel.com/new)
2. Import your repository (GitHub, GitLab, Bitbucket, Azure DevOps)
3. Configure project settings
4. Deploy!

**Automatic Deploys:**
- **Production:** Every push to `main` branch → production deployment
- **Preview:** Every push to other branches → preview deployment with unique URL
- **Pull Requests:** Automatic preview for every PR with status checks

**Example Workflow:**
```bash
# Create feature branch
git checkout -b feature/new-ui

# Make changes and commit
git add .
git commit -m "feat: new landing page design"

# Push to GitHub
git push origin feature/new-ui

# Vercel automatically:
# 1. Detects the push
# 2. Runs build
# 3. Deploys to preview URL: https://my-app-abc123-username.vercel.app
# 4. Comments on PR with preview link
```

---

### 2. Vercel CLI

**Install:**
```bash
npm i -g vercel
```

**Deploy to Preview:**
```bash
vercel
```

**Deploy to Production:**
```bash
vercel --prod
```

**Pull Environment Variables:**
```bash
vercel env pull .env.local
```

**Link Local Project:**
```bash
vercel link
```

**View Logs:**
```bash
vercel logs [deployment-url]
```

---

### 3. Deploy Hooks

**Trigger deployments via webhook URL:**

**Setup:**
1. Go to Project Settings → Git → Deploy Hooks
2. Create hook with branch name
3. Get webhook URL

**Trigger:**
```bash
curl -X POST https://api.vercel.com/v1/integrations/deploy/[hook-id]
```

**Use Cases:**
- Trigger rebuild when CMS content changes
- Scheduled rebuilds via cron job
- Third-party service integrations

---

### 4. REST API (Advanced)

**Programmatic deployments:**

```typescript
const response = await fetch('https://api.vercel.com/v13/deployments', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.VERCEL_TOKEN}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'my-app',
    files: [...], // File SHAs
    gitSource: {
      type: 'github',
      repo: 'username/repo',
      ref: 'main'
    }
  })
})
```

---

## Environment Variables

### Setting Up Environment Variables

**Via Dashboard:**
1. Go to Project Settings → Environment Variables
2. Add key-value pairs
3. Select environment type (Production, Preview, Development)
4. Save

**Via CLI:**
```bash
# Add variable
vercel env add DATABASE_URL

# Pull to local .env.local
vercel env pull .env.local

# List all variables
vercel env ls
```

### Environment Types

**Production:**
- Applied to `main` branch deployments
- Applied when using `vercel --prod`

**Preview:**
- Applied to all non-production branches
- Can set branch-specific overrides

**Development:**
- Local development only
- Stored in `.env.local` (gitignored)

**Example Configuration:**

| Variable | Production | Preview | Development |
|----------|-----------|---------|-------------|
| `DATABASE_URL` | `prod-db.com` | `staging-db.com` | `localhost:5432` |
| `NEXT_PUBLIC_API_URL` | `https://api.example.com` | `https://preview-api.example.com` | `http://localhost:3000` |
| `STRIPE_SECRET_KEY` | `sk_live_...` | `sk_test_...` | `sk_test_...` |

### Accessing in Code

**Server-side (Next.js):**
```typescript
// app/api/route.ts
export async function GET() {
  const dbUrl = process.env.DATABASE_URL
  // Use dbUrl...
}
```

**Client-side (must be prefixed with `NEXT_PUBLIC_`):**
```typescript
// app/page.tsx
'use client'

export default function Page() {
  const apiUrl = process.env.NEXT_PUBLIC_API_URL
  // Use apiUrl...
}
```

### System Variables

Vercel provides automatic system variables:

```typescript
process.env.VERCEL                    // "1" if running on Vercel
process.env.VERCEL_ENV                // "production" | "preview" | "development"
process.env.VERCEL_URL                // Deployment URL (without https://)
process.env.VERCEL_REGION             // Region where code is running
process.env.VERCEL_GIT_COMMIT_SHA     // Git commit SHA
process.env.VERCEL_GIT_COMMIT_REF     // Git branch or tag
process.env.VERCEL_GIT_REPO_OWNER     // GitHub username
process.env.VERCEL_GIT_REPO_SLUG      // Repository name
```

**Example Usage:**
```typescript
// app/api/health/route.ts
export async function GET() {
  return Response.json({
    environment: process.env.VERCEL_ENV,
    region: process.env.VERCEL_REGION,
    commit: process.env.VERCEL_GIT_COMMIT_SHA,
    deployment: process.env.VERCEL_URL
  })
}
```

### Limits

- **64 KB total** per deployment (all variables combined)
- **5 KB per variable** for Edge Functions
- Changes apply only to **new deployments**, not existing ones

---

## Vercel Functions

### Serverless Functions

**Automatic with Next.js:**

```typescript
// app/api/hello/route.ts
export async function GET(request: Request) {
  return Response.json({ message: 'Hello from Vercel!' })
}

export async function POST(request: Request) {
  const body = await request.json()
  // Process body...
  return Response.json({ success: true })
}
```

**Configuration:**

```typescript
// app/api/heavy-task/route.ts
export const runtime = 'nodejs'
export const maxDuration = 60 // seconds (Pro: 900s, Hobby: 10s)
export const dynamic = 'force-dynamic' // Disable caching

export async function GET() {
  // Long-running task...
}
```

**Function Config via vercel.json:**

```json
{
  "functions": {
    "api/analytics/*.ts": {
      "memory": 3009,
      "maxDuration": 30
    },
    "api/ai/*.ts": {
      "memory": 1024,
      "maxDuration": 60
    }
  }
}
```

### Edge Functions

**Use Edge Runtime for ultra-low latency:**

```typescript
// app/api/edge/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  const geo = request.headers.get('x-vercel-ip-country')

  return Response.json({
    message: 'Hello from the edge!',
    country: geo,
    region: process.env.VERCEL_REGION
  })
}
```

**Edge Middleware:**

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const country = request.geo?.country || 'US'

  // Redirect based on geolocation
  if (country === 'CN' && !request.nextUrl.pathname.startsWith('/cn')) {
    return NextResponse.redirect(new URL('/cn', request.url))
  }

  // Add custom header
  const response = NextResponse.next()
  response.headers.set('x-user-country', country)
  return response
}

export const config = {
  matcher: '/((?!api|_next/static|_next/image|favicon.ico).*)'
}
```

### When to Use Edge vs Serverless

| Use Case | Edge Runtime | Serverless (Node.js) |
|----------|-------------|---------------------|
| **Geography-based routing** | ✅ Optimal | ❌ Too slow |
| **A/B testing** | ✅ Optimal | ⚠️ Possible |
| **Authentication checks** | ✅ Optimal | ⚠️ Slower |
| **Database queries** | ⚠️ Limited | ✅ Full access |
| **File system access** | ❌ Not available | ✅ Available |
| **Long computations** | ❌ Limited | ✅ Up to 900s |
| **AI/ML inference** | ⚠️ Limited libraries | ✅ Full libraries |
| **Heavy npm packages** | ❌ Size limits | ✅ No limits |

---

## Project Configuration (vercel.json)

### Basic Configuration

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "buildCommand": "pnpm build",
  "devCommand": "pnpm dev",
  "installCommand": "pnpm install",
  "framework": "nextjs",
  "outputDirectory": ".next"
}
```

### Redirects

```json
{
  "redirects": [
    {
      "source": "/old-blog/:slug",
      "destination": "/blog/:slug",
      "permanent": true
    },
    {
      "source": "/docs",
      "destination": "/documentation",
      "permanent": false
    }
  ]
}
```

### Rewrites (Internal)

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://external-api.com/:path*"
    },
    {
      "source": "/blog/:slug",
      "destination": "/articles/:slug"
    }
  ]
}
```

### Headers

```json
{
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, PUT, DELETE, OPTIONS"
        }
      ]
    }
  ]
}
```

### Regions

```json
{
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 30
    }
  },
  "regions": ["iad1", "sfo1"]
}
```

**Available Regions:**
- `iad1` - Washington, D.C., USA (default)
- `sfo1` - San Francisco, USA
- `pdx1` - Portland, USA
- `gru1` - São Paulo, Brazil
- `fra1` - Frankfurt, Germany
- `lhr1` - London, UK
- `hnd1` - Tokyo, Japan
- `syd1` - Sydney, Australia
- `sin1` - Singapore
- `bom1` - Mumbai, India

### Cron Jobs

```json
{
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 0 * * *"
    },
    {
      "path": "/api/cron/daily-report",
      "schedule": "0 9 * * MON-FRI"
    }
  ]
}
```

**Cron Expression Format:**
```
┌─── minute (0-59)
│ ┌─── hour (0-23)
│ │ ┌─── day of month (1-31)
│ │ │ ┌─── month (1-12)
│ │ │ │ ┌─── day of week (0-7)
│ │ │ │ │
* * * * *
```

**Protected Cron Endpoint:**
```typescript
// app/api/cron/cleanup/route.ts
import { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  // Verify cron secret
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Run cleanup task
  await cleanupOldData()

  return Response.json({ success: true })
}
```

### Image Optimization

```json
{
  "images": {
    "sizes": [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    "formats": ["image/avif", "image/webp"],
    "minimumCacheTTL": 86400,
    "remotePatterns": [
      {
        "protocol": "https",
        "hostname": "images.example.com",
        "pathname": "/uploads/**"
      }
    ]
  }
}
```

---

## Preview Deployments

### Automatic Preview URLs

**Every Git push to non-main branches gets:**
- Unique preview URL: `https://my-app-git-feature-username.vercel.app`
- Comment on PR with preview link
- Status checks in GitHub/GitLab

**Access Preview:**
```
Production:  https://my-app.vercel.app
Preview:     https://my-app-git-feature-ui-username.vercel.app
             https://my-app-abc123.vercel.app
```

### Preview Environment Variables

**Set different values for preview:**

```bash
# Production
DATABASE_URL=postgresql://prod.db

# Preview
DATABASE_URL=postgresql://staging.db
```

**Branch-specific overrides:**
```
Branch: staging → DATABASE_URL=postgresql://staging.db
Branch: *       → DATABASE_URL=postgresql://preview.db
```

### Password Protection

**Protect previews from public access:**

1. Project Settings → Deployment Protection
2. Enable "Password Protection"
3. Set password
4. Share password with team

**Or use Vercel Authentication** (requires login to view):
- Project Settings → Deployment Protection
- Enable "Vercel Authentication"
- Only team members can access

---

## Custom Domains

### Add Custom Domain

**Via Dashboard:**
1. Project Settings → Domains
2. Add domain: `example.com`
3. Configure DNS records
4. Wait for SSL provisioning (automatic)

**DNS Configuration:**

**Option 1: Nameservers (Recommended)**
```
Point your domain's nameservers to Vercel:
ns1.vercel-dns.com
ns2.vercel-dns.com
```

**Option 2: A Record**
```
Type: A
Name: @
Value: 76.76.21.21
```

**Option 3: CNAME**
```
Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

### Multiple Domains

```json
// vercel.json
{
  "alias": [
    "example.com",
    "www.example.com",
    "app.example.com"
  ]
}
```

### Redirects to Primary Domain

**Redirect www to apex:**
```json
{
  "redirects": [
    {
      "source": "/:path*",
      "destination": "https://example.com/:path*",
      "permanent": true,
      "has": [
        {
          "type": "host",
          "value": "www.example.com"
        }
      ]
    }
  ]
}
```

---

## Caching Strategies

### Static Assets

**Automatically cached:**
- Images from `public/` folder
- `_next/static/` files (JS, CSS)
- Immutable assets with hashed filenames

**Cache headers:**
```
Cache-Control: public, max-age=31536000, immutable
```

### API Routes

**Disable caching (default):**
```typescript
export const dynamic = 'force-dynamic'
```

**Enable caching:**
```typescript
// app/api/products/route.ts
export const revalidate = 3600 // Cache for 1 hour

export async function GET() {
  const products = await db.product.findMany()
  return Response.json(products)
}
```

### Page Caching (Next.js)

**Static Generation (cached permanently):**
```typescript
// app/about/page.tsx
export default async function AboutPage() {
  return <div>About Us</div>
}
```

**Incremental Static Regeneration:**
```typescript
// app/blog/[slug]/page.tsx
export const revalidate = 60 // Revalidate every 60 seconds

export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(post => ({ slug: post.slug }))
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  return <article>{post.content}</article>
}
```

**On-Demand Revalidation:**
```typescript
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache'

export async function POST(request: Request) {
  const { path } = await request.json()
  revalidatePath(path)
  return Response.json({ revalidated: true })
}
```

---

## Monitoring & Observability

### Runtime Logs

**Via Dashboard:**
- Project → Deployments → [Deployment] → Logs

**Via CLI:**
```bash
vercel logs [deployment-url]
vercel logs --follow  # Stream logs
```

**Add Logging:**
```typescript
// app/api/users/route.ts
export async function GET() {
  console.log('Fetching users...') // Shows in Vercel logs
  const users = await db.user.findMany()
  console.log(`Found ${users.length} users`)
  return Response.json(users)
}
```

### Analytics

**Enable Web Analytics:**
1. Project Settings → Analytics
2. Enable Web Analytics
3. View metrics: pageviews, visitors, top pages, referrers

**Speed Insights:**
- Automatic Core Web Vitals tracking
- Real User Monitoring (RUM)
- Performance score per page

### Error Tracking

**Integrate Sentry:**

```typescript
// instrumentation.ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./sentry.server.config')
  }

  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./sentry.edge.config')
  }
}
```

---

## Best Practices

### ✅ DO:

1. **Use Environment Variables for Secrets**
   ```typescript
   // ✅ GOOD
   const apiKey = process.env.API_KEY

   // ❌ BAD
   const apiKey = 'hardcoded-key-123'
   ```

2. **Set Appropriate Function Timeouts**
   ```typescript
   // Heavy AI task
   export const maxDuration = 60

   // Simple API fetch
   export const maxDuration = 10
   ```

3. **Use Edge for Authentication/Routing**
   ```typescript
   // middleware.ts
   export const runtime = 'edge'
   ```

4. **Cache Aggressively**
   ```typescript
   export const revalidate = 3600 // 1 hour
   ```

5. **Use Preview Deployments**
   - Test every PR before merging
   - Share preview links with team/clients

6. **Enable Deployment Protection**
   - Password protect preview deployments
   - Use Vercel Authentication for sensitive projects

### ❌ DON'T:

1. **Don't Commit `.env.local`**
   ```bash
   # .gitignore
   .env*.local
   ```

2. **Don't Use Long-Running Processes**
   ```typescript
   // ❌ BAD - Will timeout
   setInterval(() => {
     // This won't work on serverless
   }, 1000)
   ```

3. **Don't Rely on File System Writes**
   ```typescript
   // ❌ BAD - Not persistent
   fs.writeFileSync('/tmp/data.json', data)

   // ✅ GOOD - Use database or cloud storage
   await s3.upload(data)
   ```

4. **Don't Skip Build Optimization**
   ```bash
   # Use production build
   vercel --prod

   # Not dev build
   vercel
   ```

---

## Troubleshooting

### Issue: Build Fails

**Check build logs:**
```bash
vercel logs [deployment-url]
```

**Common fixes:**
- Ensure all dependencies in `package.json`
- Check Node.js version compatibility
- Verify environment variables are set

### Issue: Function Timeout

**Error:** `FUNCTION_INVOCATION_TIMEOUT`

**Solution:**
```typescript
// Increase timeout
export const maxDuration = 60 // Up to 900s on Pro
```

Or optimize code to run faster.

### Issue: Environment Variable Not Found

**Check:**
1. Variable is set in Vercel dashboard
2. Correct environment selected (Production/Preview/Development)
3. Redeploy after adding variables

**Pull to local:**
```bash
vercel env pull .env.local
```

### Issue: 404 on API Route

**Check:**
1. File is in correct location (`app/api/route.ts`)
2. Exported handler functions (GET, POST, etc.)
3. `vercel.json` rewrites not conflicting

### Issue: Stale Content

**Force revalidation:**
```typescript
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache'

export async function POST() {
  revalidatePath('/')
  return Response.json({ success: true })
}
```

---

## Migration Guide

### From Netlify to Vercel

1. **Import project:**
   ```bash
   vercel import
   ```

2. **Update build settings:**
   ```json
   {
     "buildCommand": "npm run build",
     "outputDirectory": "out"
   }
   ```

3. **Migrate functions:**
   - Netlify: `netlify/functions/hello.js`
   - Vercel: `app/api/hello/route.ts`

4. **Update environment variables** in Vercel dashboard

5. **Update DNS** to point to Vercel

### From Traditional Hosting to Vercel

**Before (Express.js):**
```javascript
// server.js
const express = require('express')
const app = express()

app.get('/api/users', async (req, res) => {
  const users = await getUsers()
  res.json(users)
})

app.listen(3000)
```

**After (Next.js on Vercel):**
```typescript
// app/api/users/route.ts
export async function GET() {
  const users = await getUsers()
  return Response.json(users)
}
```

---

## Resources

- **Official Docs:** https://vercel.com/docs
- **Deployment Docs:** https://vercel.com/docs/deployments/overview
- **Functions Docs:** https://vercel.com/docs/functions
- **CLI Reference:** https://vercel.com/docs/cli
- **Examples:** https://vercel.com/templates
- **Status Page:** https://vercel-status.com
- **Community:** https://github.com/vercel/vercel/discussions

---

## Related Stack Modules

- [framework-nextjs.md](../meta-framework/framework-nextjs.md) - Next.js framework
- [api-nextjs-server-actions.md](../api/api-nextjs-server-actions.md) - Next.js API patterns
- [deployment-netlify-edge.md](./deployment-netlify-edge.md) - Alternative platform
- [deployment-cloudflare.md](./deployment-cloudflare.md) - Alternative platform

---

**Last Updated:** 2025-01-16
**Maintained By:** Claude Memory System
**Status:** Active
