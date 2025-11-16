# [PROJECT_NAME] - Master Development Guide

**Last Updated:** [Will be updated during initialization]
**Status:** Template

---

## GLOBAL DEVELOPMENT RULES

**Package Manager:** [Will be set during initialization - default: pnpm]
**Local Development:** The app runs on [Will be set - default: localhost:3000]
**Logging:** [Will be set - default: Winston for server-side logging]

---

## AI RULESET: How This Works

This master file uses a **RAG (Retrieval-Augmented Generation) model** to direct you to the right specialist ruleset for each task.

### Rule Hierarchy

1. **Core Rules** (this file): Universal principles that apply to ALL work
2. **Specialist Rules** (referenced files): Detailed instructions for specific domains

**When there's a conflict:** Specialist rules always override core rules.

---

## SPECIALIST RULESET ROUTING

For detailed implementation guidance, you **MUST** treat the corresponding specialist file as the primary source of truth:

### 1. Frontend Components & Design System
**File:** `.claude/02-frontend-component.md`

**When to use:**
- Building React components
- Implementing forms or UI elements
- Working with CSS/styling
- Using UI component libraries
- Creating data tables, dropdowns, or dialogs
- Implementing loading states or skeletons
- Working with icons, typography, or spacing

**Contains:**
- React patterns and best practices
- Design system components and usage
- UI patterns (data tables, forms, modals, etc.)
- State management patterns
- Error handling and loading states

---

### 2. API Routes & Server Actions
**File:** `.claude/03-api-server-action.md`

**When to use:**
- Creating or modifying API routes
- Implementing server actions/functions
- Handling authentication and authorization
- Processing form submissions
- Validating user input
- Working with database queries

**Contains:**
- API route patterns
- Authentication & authorization
- Security best practices
- Error handling
- Response structures

---

### 3. Testing
**File:** `.claude/04-testing.md`

**When to use:**
- Writing unit tests
- Writing integration tests
- Testing React components
- Mocking dependencies
- Test coverage requirements

**Contains:**
- Testing library patterns
- Test configuration
- Mocking strategies
- Test organization and structure

---

### 4. Database & SQL
**File:** `.claude/05-database-[STACK].md`

**When to use:**
- Creating database migrations
- Writing queries
- Implementing security policies
- Designing database schema
- Creating database functions

**Contains:**
- Database-specific patterns
- Migration creation process
- Security policy patterns
- Database best practices
- Query optimization

---

### 5. Deployment & Edge Functions
**File:** `.claude/06-deployment-edge-functions.md`

**When to use:**
- Deploying to production
- Creating/modifying serverless functions
- Working with external APIs
- Setting up build configurations
- Managing environment variables

**Contains:**
- Deployment configuration
- Serverless function patterns
- External API integrations
- Build and optimization settings
- Environment variable management

---

### 6. General Standards
**File:** `.claude/standards.md`

**When to use:**
- Understanding project architecture
- Code style and naming conventions
- File structure organization
- Git workflow

**Contains:**
- TypeScript conventions
- Project architecture
- UI component hierarchy
- Project-wide coding standards

---

### 7. Documentation Maintenance
**File:** `.claude/07-documentation-maintenance.md`

**When to use:**
- Creating commits (auto-triggered)
- Updating tasks.md progress
- Documenting new patterns
- Synchronizing documentation
- Maintaining README and documentation health

**Contains:**
- tasks.md update protocol
- Specialist playbook update rules
- Commit workflow integration
- Documentation health monitoring

---

## OPTIONAL SPECIALIST FILES

Depending on your tech stack, you may also have:

### Authentication
**File:** `.claude/auth-[SYSTEM].md` (e.g., `auth-better-auth.md`)

**When to use:**
- Setting up authentication
- Implementing login/logout
- Managing sessions
- Configuring OAuth providers

---

### AI Integration
**File:** `.claude/ai-[SDK].md` (e.g., `ai-vercel-ai-sdk.md`)

**When to use:**
- Integrating AI features
- Streaming AI responses
- Implementing tool/function calling
- Managing AI API calls

---

### Email
**File:** `.claude/email-[SERVICE].md` (e.g., `email-resend.md`)

**When to use:**
- Sending transactional emails
- Managing email templates
- Setting up email notifications

---

### UI Library
**File:** `.claude/ui-[LIBRARY].md` (e.g., `ui-shadcn.md`)

**When to use:**
- Using specific UI components
- Understanding design system
- Customizing component themes

---

### Payments
**File:** `.claude/payments-[SERVICE].md` (e.g., `payments-stripe.md`, `payments-polar.md`)

**When to use:**
- Implementing payment processing
- Setting up subscriptions
- Creating checkout flows
- Managing customers and invoices
- Handling webhooks for payment events
- Configuring product pricing

**Contains:**
- Payment integration patterns
- Checkout session creation
- Subscription management
- Webhook handling and verification
- Security best practices for payments
- Testing with test cards/modes

---

## CORE PRINCIPLES (Apply to ALL Work)

### Code Quality
- All code MUST be TypeScript with strict mode
- NO `any` type usage
- Use functional and declarative patterns
- Implement proper error handling
- Follow accessibility guidelines (WCAG)

### Security
- Never commit secrets or API keys
- Validate all user input
- Implement proper authentication checks
- Use appropriate security policies
- Prevent OWASP Top 10 vulnerabilities

### Performance
- Optimize for Core Web Vitals
- Use server components where appropriate
- Implement proper loading states
- Minimize client-side JavaScript
- Use proper caching strategies

### Git Workflow
- Use Conventional Commit messages (feat:, fix:, refactor:, docs:)
- Keep commits small and focused
- Test before committing
- Never force push to main/master

---

## DECISION TREE: Which File Should I Use?

```
START
  ↓
Are you creating a commit?
  YES → Auto-trigger 07-documentation-maintenance.md
  NO → Continue
  ↓
Is this a UI/component task?
  YES → Use 02-frontend-component.md
  NO → Continue
  ↓
Is this an API or server action?
  YES → Use 03-api-server-action.md
  NO → Continue
  ↓
Is this about database/queries?
  YES → Use 05-database-[stack].md
  NO → Continue
  ↓
Is this about deployment/serverless functions?
  YES → Use 06-deployment-edge-functions.md
  NO → Continue
  ↓
Is this about testing?
  YES → Use 04-testing.md
  NO → Continue
  ↓
Is this about authentication?
  YES → Check for auth-[system].md
  NO → Continue
  ↓
Is this about AI features?
  YES → Check for ai-[sdk].md
  NO → Continue
  ↓
Is this about email?
  YES → Check for email-[service].md
  NO → Continue
  ↓
Is this about payments/subscriptions?
  YES → Check for payments-[service].md
  NO → Continue
  ↓
Need general project context?
  YES → Use standards.md
```

---

## IMPORTANT NOTES

1. **Always check specialist files first** - They contain critical implementation details
2. **Specialist rules override core rules** - If there's a conflict, follow the specialist file
3. **Ask before adding dependencies** - Always confirm with the user before installing new packages
4. **Use the established patterns** - Check existing code for examples before creating new patterns
5. **Document new patterns** - If you create a reusable pattern, add it to the appropriate specialist file

---

## QUICK REFERENCE

- **Package Manager:** [Will be set during initialization]
- **Dev Server:** [Will be set during initialization]
- **Framework:** [Will be set based on detected stack modules]
- **Database:** [Will be set based on detected stack modules]
- **UI Libraries:** [Will be set based on detected stack modules]
- **Styling:** [Will be set based on projectbrief or default to Tailwind CSS]
- **Icons:** [Will be set based on projectbrief or UI library default]
- **Logging:** [Will be set based on projectbrief or default to Winston]
- **Testing:** [Will be set based on projectbrief or default to Jest + React Testing Library]
- **Validation:** [Will be set based on projectbrief or default to Zod]
- **Auth:** [Will be set if auth module is detected]
- **AI:** [Will be set if AI module is detected]
- **Email:** [Will be set if email module is detected]

---

**Last Updated:** [Will be updated during initialization]
**Maintained By:** Development Team
**Status:** Active
