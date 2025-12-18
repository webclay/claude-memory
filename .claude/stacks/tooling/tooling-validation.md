# TypeScript Validation Libraries

**Purpose:** Schema validation and type inference for TypeScript applications

**Last Updated:** 2025-12-18

**Standard Schema Spec:** https://standardschema.dev

---

## Overview

TypeScript validation libraries provide runtime validation with compile-time type inference. The three major options are:

| Library | Best For | Bundle Size | Speed |
|---------|----------|-------------|-------|
| **Zod** | Ecosystem, familiarity | ~12kb | Fast |
| **Valibot** | Bundle size, tree-shaking | ~1-5kb | Fast |
| **ArkType** | Performance, complex types | ~25kb | Fastest |

All three implement **Standard Schema**, meaning they're interchangeable in tools like tRPC, TanStack Form, TanStack Router, and Hono.

---

## Quick Comparison

### Zod - The Ecosystem Leader

```typescript
import { z } from "zod";

const userSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  age: z.number().int().positive().optional()
});

type User = z.infer<typeof userSchema>;

const result = userSchema.safeParse(data);
if (result.success) {
  console.log(result.data); // typed as User
}
```

**Pros:** Largest ecosystem (78+ integrations), excellent docs, familiar API
**Cons:** Larger bundle than Valibot, slower than ArkType

---

### Valibot - The Bundle Champion

```typescript
import * as v from "valibot";

const userSchema = v.object({
  name: v.pipe(v.string(), v.minLength(2)),
  email: v.pipe(v.string(), v.email()),
  age: v.optional(v.pipe(v.number(), v.integer(), v.minValue(1)))
});

type User = v.InferOutput<typeof userSchema>;

const result = v.safeParse(userSchema, data);
if (result.success) {
  console.log(result.output); // typed as User
}
```

**Pros:** Smallest bundle (tree-shakeable), modular design
**Cons:** Slightly more verbose, smaller ecosystem

---

### ArkType - The Performance King

```typescript
import { type } from "arktype";

const user = type({
  name: "string >= 2",
  email: "email",
  "age?": "integer > 0"
});

type User = typeof user.infer;

const result = user(data);
if (result instanceof type.errors) {
  console.log(result.summary);
} else {
  console.log(result); // typed as User
}
```

**Pros:** 2-4x faster validation, expressive syntax, deep type inference
**Cons:** Larger bundle, steeper learning curve, smaller ecosystem

---

## Standard Schema

[Standard Schema](https://standardschema.dev) is a TypeScript interface specification that enables interoperability between validation libraries. It was co-authored by the creators of Zod, Valibot, and ArkType.

### Why It Matters

Before Standard Schema, tools needed library-specific adapters:
- tRPC + Zod adapter
- tRPC + Valibot adapter
- tRPC + ArkType adapter
- (repeat for every tool × every library)

With Standard Schema, tools implement one interface and support all libraries automatically.

### How It Works

Libraries expose a `~standard` property on schemas:

```typescript
import { z } from "zod";
import * as v from "valibot";
import { type } from "arktype";

// All three satisfy StandardSchemaV1
const zodSchema = z.string();
const valibotSchema = v.string();
const arktypeSchema = type("string");

// Tools can accept any of them
function validate<T>(schema: StandardSchemaV1<unknown, T>, data: unknown): T {
  const result = schema["~standard"].validate(data);
  if (result.issues) throw new Error("Validation failed");
  return result.value;
}
```

### Standard JSON Schema

For generating JSON Schema (useful for OpenAPI, AI tools, forms):

```typescript
// Zod v4.2+
import { z } from "zod";
const schema = z.object({ name: z.string() });
const jsonSchema = z.toJSONSchema(schema);

// Valibot v1.2+
import * as v from "valibot";
import { toStandardJsonSchema } from "valibot";
const schema = v.object({ name: v.string() });
const jsonSchema = toStandardJsonSchema(schema);

// ArkType v2.1.28+
import { type } from "arktype";
const schema = type({ name: "string" });
const jsonSchema = schema["~standard"].toJSONSchema();
```

---

## When to Use Each

### Choose Zod When:
- You need maximum ecosystem compatibility
- Team is already familiar with it
- Using libraries that only support Zod
- Documentation and examples matter most

### Choose Valibot When:
- Bundle size is critical (client-side validation)
- Building for edge/serverless with size limits
- Want fine-grained tree-shaking
- Prefer functional composition style

### Choose ArkType When:
- Validating large datasets (10k+ objects)
- Need complex type expressions
- Performance is critical
- Want type-level validation syntax

### Starting Fresh?
**Recommendation:** Start with **Zod** for its ecosystem and switch later if needed. Standard Schema makes migration painless.

---

## Installation

```bash
# Zod
npm install zod

# Valibot
npm install valibot

# ArkType
npm install arktype

# Standard Schema types (optional - can copy/paste instead)
npm install @standard-schema/spec
```

---

## Common Patterns

### Environment Variables

**Zod:**
```typescript
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  PORT: z.coerce.number().default(3000)
});

export const env = envSchema.parse(process.env);
```

**Valibot:**
```typescript
const envSchema = v.object({
  DATABASE_URL: v.pipe(v.string(), v.url()),
  API_KEY: v.pipe(v.string(), v.minLength(1)),
  PORT: v.optional(v.pipe(v.unknown(), v.transform(Number)), 3000)
});

export const env = v.parse(envSchema, process.env);
```

---

### API Input Validation

**Zod:**
```typescript
const createUserInput = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2).max(100)
});

export async function POST(req: Request) {
  const body = await req.json();
  const result = createUserInput.safeParse(body);

  if (!result.success) {
    return Response.json(
      { errors: result.error.flatten().fieldErrors },
      { status: 400 }
    );
  }

  // result.data is typed
  await createUser(result.data);
}
```

---

### Form Validation (with TanStack Form)

```typescript
import { useForm } from "@tanstack/react-form";
import { zodValidator } from "@tanstack/zod-form-adapter";
// or: import { valibotValidator } from "@tanstack/valibot-form-adapter";
import { z } from "zod";

const form = useForm({
  defaultValues: { email: "", password: "" },
  validatorAdapter: zodValidator(),
  onSubmit: async ({ value }) => {
    // value is typed
  }
});
```

---

### Transformations

**Zod:**
```typescript
const dateSchema = z.string().transform((val) => new Date(val));
const trimmedSchema = z.string().trim().toLowerCase();
const defaultSchema = z.string().default("anonymous");
```

**Valibot:**
```typescript
const dateSchema = v.pipe(v.string(), v.transform((val) => new Date(val)));
const trimmedSchema = v.pipe(v.string(), v.trim(), v.toLowerCase());
const defaultSchema = v.optional(v.string(), "anonymous");
```

**ArkType:**
```typescript
const dateSchema = type("string").pipe((s) => new Date(s));
const trimmedSchema = type("string").pipe((s) => s.trim().toLowerCase());
```

---

### Discriminated Unions

**Zod:**
```typescript
const resultSchema = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.unknown() }),
  z.object({ status: z.literal("error"), message: z.string() })
]);
```

**Valibot:**
```typescript
const resultSchema = v.variant("status", [
  v.object({ status: v.literal("success"), data: v.unknown() }),
  v.object({ status: v.literal("error"), message: v.string() })
]);
```

**ArkType:**
```typescript
const resultSchema = type({
  status: "'success'",
  data: "unknown"
}).or({
  status: "'error'",
  message: "string"
});
```

---

## Error Handling

**Zod:**
```typescript
const result = schema.safeParse(data);
if (!result.success) {
  // Flatten for form errors
  const errors = result.error.flatten();
  console.log(errors.fieldErrors); // { email: ["Invalid email"] }

  // Or format for API response
  const formatted = result.error.format();
}
```

**Valibot:**
```typescript
const result = v.safeParse(schema, data);
if (!result.success) {
  for (const issue of result.issues) {
    console.log(issue.path, issue.message);
  }
}
```

**ArkType:**
```typescript
const result = schema(data);
if (result instanceof type.errors) {
  console.log(result.summary); // Human-readable summary
  for (const error of result) {
    console.log(error.path, error.message);
  }
}
```

---

## Performance Tips

1. **Reuse schemas** - Don't recreate schemas on every call
2. **Use `.safeParse()`** - Avoid try/catch overhead
3. **Avoid excessive `.refine()`** - Custom validators are slower
4. **Consider ArkType for bulk validation** - 2-4x faster on large datasets
5. **Tree-shake with Valibot** - Import only what you use

---

## Migrating Between Libraries

Thanks to Standard Schema, migration is straightforward:

```typescript
// Before: Zod
import { z } from "zod";
const schema = z.object({ name: z.string() });

// After: Valibot (same usage in Standard Schema consumers)
import * as v from "valibot";
const schema = v.object({ name: v.string() });

// Tools using Standard Schema don't need changes!
```

---

## Resources

### Zod
- **Docs:** https://zod.dev
- **GitHub:** https://github.com/colinhacks/zod

### Valibot
- **Docs:** https://valibot.dev
- **GitHub:** https://github.com/fabian-hiller/valibot

### ArkType
- **Docs:** https://arktype.io
- **GitHub:** https://github.com/arktypeio/arktype

### Standard Schema
- **Spec:** https://standardschema.dev
- **GitHub:** https://github.com/standard-schema/standard-schema
- **npm:** https://www.npmjs.com/package/@standard-schema/spec

---

## Related Stack Modules

- [api-orpc.md](../api/api-orpc.md) - Type-safe APIs with Zod validation
- [tooling-ultracite.md](./tooling-ultracite.md) - Linting and formatting

---

**Last Updated:** 2025-12-18
**Maintained By:** Claude Memory System
**Status:** Active
