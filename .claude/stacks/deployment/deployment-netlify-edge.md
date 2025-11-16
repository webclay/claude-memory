# AI RULESET: DEPLOYMENT & EDGE FUNCTIONS

## ROLE
You are a DevOps and serverless expert focused on deployment configurations and Supabase Edge Functions. Your primary goal is to ensure secure, scalable deployments and proper Edge Function patterns.

## CRITICAL INSTRUCTIONS

1. **Deployment Platform**: The application is deployed on **Netlify** (configured to deploy from `dev` branch)
2. **Edge Functions**: All external API calls (OpenRouter, Firecrawl, ScreenshotOne) MUST go through Supabase Edge Functions
3. **API Keys**: Never expose API keys in Next.js app - all secrets managed in Supabase environment
4. **Image Optimization**: Use `unoptimized` prop on Next.js Image components for Netlify compatibility

---

## NETLIFY DEPLOYMENT PATTERNS

### Netlify Configuration

**File:** `netlify.toml`

```toml
[build]
  command = "pnpm build"
  publish = ".next"
  base = "/"

[context.production]
  branch = "dev"  # Deploy from dev branch

[build.environment]
  NODE_VERSION = "20"
  # Set environment variables in Netlify UI:
  # NEXT_PUBLIC_SUPABASE_URL
  # NEXT_PUBLIC_SUPABASE_ANON_KEY

# Security headers
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

# Cache static assets
[[headers]]
  for = "/_next/static/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

**Key Principles:**
- Always use `pnpm` as package manager
- Deploy from `dev` branch to production
- Set environment variables in Netlify UI, not in code
- Implement security headers for all routes
- Cache static assets aggressively

---

### Next.js Image Optimization for Netlify

**Problem:** Netlify doesn't support Next.js image optimization by default

**Solution:** Add `unoptimized` prop to all Image components

```tsx
import Image from 'next/image';

// ❌ Without unoptimized - causes 400 errors on Netlify
<Image src="/images/logo.png" alt="Logo" width={200} height={50} />

// ✅ With unoptimized - works on Netlify
<Image
  src="/images/logo.png"
  alt="Logo"
  width={200}
  height={50}
  unoptimized
/>
```

**When to use:**
- All logo images
- Static images in public folder
- Any image that needs to work on Netlify

---

### Theme-Aware Logo Pattern

**Use Case:** Display different logos for light/dark mode

```tsx
import Image from 'next/image';
import { useTheme } from 'next-themes';

export function LogoDisplay() {
  const { theme } = useTheme();
  const [mounted, setMounted] = React.useState(false);

  React.useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return <div className="h-10 w-40" />; // Placeholder to prevent flash
  }

  return (
    <Image
      src={theme === 'dark' ? '/images/logo-white.png' : '/images/logo-black.png'}
      alt="CopyHatch"
      width={160}
      height={40}
      unoptimized
      priority
    />
  );
}
```

**Key Principles:**
- Use hydration-safe mounting check
- Provide placeholder with same dimensions
- Use `priority` for above-the-fold images
- Store theme-specific logos in `/public/images/`

---

## SUPABASE EDGE FUNCTIONS

### Edge Function Architecture

**Directory Structure:**
```
supabase/functions/
├── openrouter-api/
│   └── index.ts
├── firecrawl-api/
│   └── index.ts
└── screenshotone-api/
    └── index.ts
```

**Why Edge Functions:**
- Keep API keys secure (not exposed to client)
- Centralized credit management
- Consistent error handling and logging
- Token usage tracking

---

### OpenRouter Edge Function Pattern

**File:** `supabase/functions/openrouter-api/index.ts`

```typescript
import { serve } from "https://deno.land/std@0.203.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.39.7";

const supabase = createClient(
  Deno.env.get("SUPABASE_URL"),
  Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")
);

serve(async (req) => {
  const startTime = Date.now();

  try {
    const requestBody = await req.json();
    const userId = requestBody.user_id;
    const model = requestBody.model || "anthropic/claude-3.5-sonnet-20240620";

    // ✅ STEP 1: CHECK CREDITS BEFORE API CALL (don't deduct yet)
    if (userId) {
      const { data: profileData, error: profileError } = await supabase
        .from('profiles')
        .select('credits_balance')
        .eq('id', userId)
        .single();

      if (profileError) {
        throw new Error(`Failed to fetch user profile: ${profileError.message}`);
      }

      const creditsBalance = profileData?.credits_balance || 0;

      if (creditsBalance < 1) {
        return new Response(JSON.stringify({
          error: 'Insufficient credits',
          details: {
            message: 'You do not have enough credits to perform this operation.',
            currentBalance: creditsBalance,
            required: 1
          }
        }), {
          status: 402, // Payment Required
          headers: { "Content-Type": "application/json" }
        });
      }
    }

    // ✅ STEP 2: PREPARE MESSAGES
    const messages = [];
    if (requestBody.system_prompt) {
      messages.push({ role: "system", content: requestBody.system_prompt });
    }
    if (requestBody.user_prompt) {
      messages.push({ role: "user", content: requestBody.user_prompt });
    }

    // ✅ STEP 3: BUILD REQUEST PAYLOAD
    const requestPayload = {
      model,
      messages,
      temperature: requestBody.temperature ?? 0.7,
      max_tokens: requestBody.max_tokens ?? 1024,
      top_p: requestBody.top_p ?? 0.9,
      presence_penalty: requestBody.presence_penalty ?? 0,
      frequency_penalty: requestBody.frequency_penalty ?? 0,
      usage: { include: true }
    };

    // ✅ STEP 4: ADD STRUCTURED OUTPUT SUPPORT
    if (requestBody.response_format) {
      requestPayload.response_format = requestBody.response_format;
    } else {
      requestPayload.response_format = { type: 'json_object' };
    }

    // ✅ STEP 5: CALL OPENROUTER API
    const openRouterKey = Deno.env.get("OPENROUTER_API_KEY");
    if (!openRouterKey) {
      throw new Error("OpenRouter API key is not configured");
    }

    const openRouterResponse = await fetch(
      "https://openrouter.ai/api/v1/chat/completions",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${openRouterKey}`,
          "HTTP-Referer": Deno.env.get("SUPABASE_URL") || "https://copyhatch.com",
          "X-Title": "CopyHatch"
        },
        body: JSON.stringify(requestPayload)
      }
    );

    if (!openRouterResponse.ok) {
      const errorDetails = await openRouterResponse.json();
      console.error("OpenRouter API error:", errorDetails);

      await logApiUsage(
        userId,
        requestPayload,
        0, 0, 0,
        openRouterResponse.status,
        Date.now() - startTime
      );

      return new Response(JSON.stringify({
        error: `Error from OpenRouter API: ${openRouterResponse.status}`,
        details: errorDetails
      }), {
        status: openRouterResponse.status,
        headers: { "Content-Type": "application/json" }
      });
    }

    const openRouterData = await openRouterResponse.json();
    const inputTokens = openRouterData.usage?.prompt_tokens ?? 0;
    const outputTokens = openRouterData.usage?.completion_tokens ?? 0;
    const totalTokens = openRouterData.usage?.total_tokens ?? 0;
    const cost = openRouterData.usage?.cost ?? null;
    const finishReason = openRouterData.choices[0]?.finish_reason;

    // ✅ STEP 6: CHECK FOR TRUNCATION
    if (finishReason === 'length') {
      return new Response(JSON.stringify({
        error: 'Response truncated due to token limit',
        details: {
          reason: 'The AI response exceeded the max_tokens limit',
          finishReason: 'length',
          tokensUsed: totalTokens,
          maxTokens: requestPayload.max_tokens,
          suggestion: 'Increase max_tokens parameter'
        }
      }), {
        status: 422,
        headers: { "Content-Type": "application/json" }
      });
    }

    let content = openRouterData.choices[0]?.message?.content || '';

    // ✅ STEP 7: VALIDATE AND CLEAN JSON (if requested)
    if (requestBody.response_format?.type === 'json_object' ||
        requestBody.response_format?.type === 'json_schema') {

      // Remove markdown code blocks if present
      let cleanedContent = content.trim();
      if (cleanedContent.startsWith('```json')) {
        cleanedContent = cleanedContent.replace(/^```json\s*/, '').replace(/\s*```$/, '');
      } else if (cleanedContent.startsWith('```')) {
        cleanedContent = cleanedContent.replace(/^```\s*/, '').replace(/\s*```$/, '');
      }

      try {
        JSON.parse(cleanedContent); // Validate
        content = cleanedContent;
      } catch (jsonError) {
        // JSON repair logic here (see full implementation)
        console.error('Invalid JSON despite JSON format request');

        return new Response(JSON.stringify({
          error: 'OpenRouter returned invalid JSON',
          details: { parseError: jsonError.message }
        }), {
          status: 422,
          headers: { "Content-Type": "application/json" }
        });
      }
    }

    // ✅ STEP 8: DEDUCT CREDITS AFTER SUCCESSFUL RESPONSE
    let creditTransactionId = null;
    if (userId) {
      const { data: deductResult, error: deductError } = await supabase.rpc(
        'deduct_credits',
        {
          p_user_id: userId,
          p_amount: 1,
          p_type: 'ai_generation',
          p_description: 'AI generation via OpenRouter',
          p_reference_id: null,
          p_metadata: { model, inputTokens, outputTokens, totalTokens, cost }
        }
      );

      if (deductError) {
        console.error('Error deducting credits:', deductError);
        // Don't fail the request - user already got the response
      } else {
        console.log(`Credits deducted. New balance: ${deductResult.new_balance}`);
        creditTransactionId = deductResult.transaction_id;
      }
    }

    // ✅ STEP 9: LOG API USAGE
    await logApiUsage(
      userId,
      requestPayload,
      totalTokens,
      inputTokens,
      outputTokens,
      200,
      Date.now() - startTime,
      "openrouter",
      cost
    );

    // ✅ STEP 10: RETURN SUCCESS RESPONSE
    return new Response(JSON.stringify({
      result: content,
      usage: {
        tokens_used: totalTokens,
        input_tokens: inputTokens,
        output_tokens: outputTokens,
        cost
      },
      creditTransactionId,
      raw_response: openRouterData
    }), {
      status: 200,
      headers: { "Content-Type": "application/json" }
    });

  } catch (error) {
    console.error("Edge function error:", error);

    return new Response(JSON.stringify({
      success: false,
      error: "Error in OpenRouter edge function",
      details: { message: error.message }
    }), {
      status: 500,
      headers: { "Content-Type": "application/json" }
    });
  }
});

async function logApiUsage(
  userId,
  requestPayload,
  tokensUsed,
  inputTokens,
  outputTokens,
  responseStatus,
  responseTime,
  type = "api",
  cost = null
) {
  const moduleId = requestPayload.moduleId;

  const { error } = await supabase.from("api_usage_logs").insert({
    user_id: userId,
    source: "openrouter",
    endpoint: "openrouter",
    tokens_used: tokensUsed,
    input_tokens: inputTokens,
    output_tokens: outputTokens,
    cost,
    request_payload: requestPayload,
    response_status: responseStatus,
    response_time: responseTime,
    integration: "openrouter",
    module_id: moduleId
  });

  if (error) {
    console.error("Error logging API usage:", error);
  }
}
```

**Key Principles:**
1. **Check credits BEFORE API call** - return 402 if insufficient
2. **Deduct credits AFTER successful response** - don't fail if deduction fails
3. **Validate JSON responses** - clean markdown wrappers, repair truncated JSON
4. **Handle truncation** - detect `finish_reason: 'length'` and return 422
5. **Log all API usage** - track tokens, cost, response time, module_id
6. **Consistent error handling** - return structured error objects
7. **Return transaction ID** - for audit trail

---

### Calling Edge Functions from Next.js

**Pattern:**

```typescript
// app/api/prompts/generate/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    // Get user session
    const { isAuthenticated, user } = await verifyAuthentication();
    if (!isAuthenticated || !user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // Call Supabase Edge Function
    const response = await fetch(
      `${process.env.NEXT_PUBLIC_SUPABASE_URL}/functions/v1/openrouter-api`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY}`
        },
        body: JSON.stringify({
          user_id: user.id,
          model: body.model || 'anthropic/claude-3.5-sonnet-20240620',
          system_prompt: body.system_prompt,
          user_prompt: body.user_prompt,
          max_tokens: body.max_tokens || 4096,
          temperature: body.temperature || 0.7,
          response_format: body.response_format || { type: 'json_object' }
        })
      }
    );

    if (!response.ok) {
      const errorData = await response.json();
      return NextResponse.json(
        { error: errorData.error, details: errorData.details },
        { status: response.status }
      );
    }

    const data = await response.json();

    return NextResponse.json({
      success: true,
      result: data.result,
      usage: data.usage,
      creditTransactionId: data.creditTransactionId
    });

  } catch (error) {
    console.error('Error calling Edge Function:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

**Key Principles:**
- Always pass `user_id` for credit tracking
- Include `Authorization` header with anon key
- Handle 402 (insufficient credits) separately from 500 errors
- Return transaction ID for audit trail

---

## CREDITS SYSTEM INTEGRATION

### Credit Flow

1. **Before API call**: Check balance (don't deduct)
2. **During API call**: Process request
3. **After success**: Deduct credits with `deduct_credits` RPC
4. **On error**: Don't deduct credits

### RPC Function Usage

```typescript
// In Edge Function (Deno)
const { data: deductResult, error: deductError } = await supabase.rpc(
  'deduct_credits',
  {
    p_user_id: userId,
    p_amount: 1,
    p_type: 'ai_generation',
    p_description: 'AI generation via OpenRouter',
    p_reference_id: null, // Optional reference to outputs table
    p_metadata: {
      model: 'anthropic/claude-3.5-sonnet',
      inputTokens: 1000,
      outputTokens: 500,
      totalTokens: 1500,
      cost: 0.015
    }
  }
);
```

**Response:**
```json
{
  "new_balance": 99,
  "transaction_id": "uuid-here"
}
```

---

## STRUCTURED OUTPUT PATTERNS

### JSON Schema Support

```typescript
// Request with JSON schema
const response = await fetch(edgeFunctionUrl, {
  method: 'POST',
  body: JSON.stringify({
    user_id: userId,
    system_prompt: 'You are a helpful assistant',
    user_prompt: 'Generate a buyer persona',
    response_format: {
      type: 'json_schema',
      json_schema: {
        name: 'buyer_persona',
        strict: true,
        schema: {
          type: 'object',
          properties: {
            name: { type: 'string' },
            age: { type: 'number' },
            occupation: { type: 'string' },
            painPoints: {
              type: 'array',
              items: { type: 'string' }
            }
          },
          required: ['name', 'age', 'occupation', 'painPoints']
        }
      }
    }
  })
});
```

**Key Principles:**
- Use `json_schema` type for strict validation
- Set `strict: true` to enforce schema
- Always define `required` fields
- Edge Function validates and repairs JSON

---

## PDF EXPORT PATTERN

### PDF Generation with React-PDF

**File:** `components/pdf/BuyerPersonaPDF.tsx`

```tsx
import { Document, Page, Text, View, StyleSheet } from '@react-pdf/renderer';

const styles = StyleSheet.create({
  page: {
    padding: 40,
    fontFamily: 'Helvetica'
  },
  header: {
    fontSize: 24,
    marginBottom: 20,
    fontWeight: 'bold'
  },
  section: {
    marginBottom: 15
  },
  label: {
    fontSize: 12,
    fontWeight: 'bold',
    marginBottom: 5,
    color: '#666'
  },
  value: {
    fontSize: 14,
    marginBottom: 10
  }
});

export const BuyerPersonaPDF = ({ persona }) => (
  <Document>
    <Page size="A4" style={styles.page}>
      <Text style={styles.header}>{persona.name}</Text>

      <View style={styles.section}>
        <Text style={styles.label}>Age</Text>
        <Text style={styles.value}>{persona.age}</Text>
      </View>

      <View style={styles.section}>
        <Text style={styles.label}>Occupation</Text>
        <Text style={styles.value}>{persona.occupation}</Text>
      </View>

      <View style={styles.section}>
        <Text style={styles.label}>Pain Points</Text>
        {persona.painPoints.map((point, index) => (
          <Text key={index} style={styles.value}>• {point}</Text>
        ))}
      </View>
    </Page>
  </Document>
);
```

### PDF Download Button

```tsx
import { PDFDownloadLink } from '@react-pdf/renderer';
import { BuyerPersonaPDF } from '@/components/pdf/BuyerPersonaPDF';
import { Download01Icon } from 'hugeicons-react';
import { Button } from '@/components/ui/button';

export function DownloadPDFButton({ persona }) {
  return (
    <PDFDownloadLink
      document={<BuyerPersonaPDF persona={persona} />}
      fileName={`buyer-persona-${persona.name.toLowerCase().replace(/\s+/g, '-')}.pdf`}
    >
      {({ loading }) => (
        <Button variant="outline" size="sm" disabled={loading}>
          <Download01Icon size={16} strokeWidth={2} className="mr-2" />
          {loading ? 'Preparing PDF...' : 'Download PDF'}
        </Button>
      )}
    </PDFDownloadLink>
  );
}
```

**Key Principles:**
- Use `@react-pdf/renderer` for PDF generation
- Create separate PDF component files in `components/pdf/`
- Use `PDFDownloadLink` for client-side generation
- Show loading state while PDF is being prepared
- Generate meaningful filenames based on content

---

## ANALYTICS DASHBOARD PATTERN

### Real-Time Analytics Data

```typescript
// lib/db/analytics.ts
import { createClient } from '@/lib/supabase/server';

export async function getAnalyticsData(userId: string) {
  const supabase = await createClient();

  // Get total outputs
  const { count: outputsCount } = await supabase
    .from('outputs')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId)
    .is('deleted_at', null);

  // Get total products
  const { count: productsCount } = await supabase
    .from('product')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId)
    .is('deleted_at', null);

  // Get total personas
  const { count: personasCount } = await supabase
    .from('persona')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId)
    .is('deleted_at', null);

  // Get credits balance
  const { data: profile } = await supabase
    .from('profiles')
    .select('credits_balance')
    .eq('id', userId)
    .single();

  return {
    totalOutputs: outputsCount || 0,
    totalProducts: productsCount || 0,
    totalPersonas: personasCount || 0,
    creditsRemaining: profile?.credits_balance || 0
  };
}
```

### Analytics Dashboard Component

```tsx
// app/(admin)/admin/components/analytics-dashboard.tsx
import { getAnalyticsData } from '@/lib/db/analytics';

export async function AnalyticsDashboard({ userId }: { userId: string }) {
  const analytics = await getAnalyticsData(userId);

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      <StatCard
        title="Total Outputs"
        value={analytics.totalOutputs}
        icon={FileIcon}
      />
      <StatCard
        title="Products"
        value={analytics.totalProducts}
        icon={Package01Icon}
      />
      <StatCard
        title="Buyer Personas"
        value={analytics.totalPersonas}
        icon={UserIcon}
      />
      <StatCard
        title="Credits Remaining"
        value={analytics.creditsRemaining}
        icon={CoinsIcon}
      />
    </div>
  );
}
```

**Key Principles:**
- Use Server Components for real-time data
- Fetch data directly from database (no API route needed)
- Use `count: 'exact'` for accurate counts
- Filter out soft-deleted records with `is('deleted_at', null)`
- Display data in card grid layout

---

## ENVIRONMENT VARIABLES

### Next.js App Environment Variables

```bash
# Public variables (exposed to client)
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...

# Server-only variables (never exposed to client)
# NOT USED - all secrets in Supabase Edge Functions
```

### Supabase Edge Function Environment Variables

Set in Supabase Dashboard > Edge Functions > Secrets:

```bash
# Required for all Edge Functions
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbG...

# Required for OpenRouter Edge Function
OPENROUTER_API_KEY=sk-or-v1-...

# Required for Firecrawl Edge Function
FIRECRAWL_API_KEY=fc-...

# Required for ScreenshotOne Edge Function
SCREENSHOTONE_API_KEY=...
```

**Key Principles:**
- All third-party API keys stored in Supabase, not Next.js
- Only Supabase URL and anon key in Next.js
- Service role key only in Edge Functions
- Never commit secrets to git

---

## DEPLOYMENT CHECKLIST

Before deploying:

- [ ] All API keys set in Supabase Edge Function secrets
- [ ] Environment variables configured in Netlify UI
- [ ] `unoptimized` prop added to all Image components
- [ ] Build passes locally with `pnpm build`
- [ ] Edge Functions tested with connection test endpoint
- [ ] Credits system tested (check, deduct, insufficient)
- [ ] PDF exports working
- [ ] Analytics dashboard showing real data
- [ ] Theme-aware logos displaying correctly
- [ ] Security headers configured in netlify.toml

---

**Last Updated:** 2025-11-02
