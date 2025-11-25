# Claude Memory - Project Instructions

## CRITICAL: Pattern Consistency Rules

**BEFORE building ANY new feature, component, or page:**

1. **Always read existing examples FIRST:**
   - If creating a page: Read 2-3 existing pages to understand the layout pattern
   - If creating a component: Check similar existing components
   - If working with database: Review existing table structures
   - If styling: Check existing components for spacing, colors, sizing

2. **Match the established patterns EXACTLY:**
   - Same layout structure
   - Same spacing (gap-4, p-6, etc.)
   - Same component usage (Card, Button, etc.)
   - Same file organization
   - Same naming conventions

3. **If uncertain about a pattern:**
   - Search for similar features in the codebase
   - Ask the user: "I see two patterns for [X]. Which should I follow?"
   - Document the chosen pattern in the appropriate stack module

4. **Never introduce new patterns without asking:**
   - Don't use different spacing if existing code uses gap-4
   - Don't create new component variants without approval
   - Don't change established layouts

**Example workflow:**
```
User: "Add a settings page"

You (Claude):
1. Read projectbrief.md → check what UI library is used
2. Read ui-shadcn.md → check design patterns
3. Search codebase → find existing pages (dashboard, profile, etc.)
4. Identify pattern → all pages use same Card/layout structure
5. Build settings page → match that exact pattern
```

---

## Unified Setup Command

**When users want to initialize or configure the memory system**, they type:

### `setup`

**IMPORTANT:** When you see the `setup` command:
1. DO NOT output any analysis or commentary first
2. IMMEDIATELY display the ASCII header from `.claude/guides/setup-workflow.md`
3. Silently analyze the environment and present findings
4. If questionnaire needed, follow the formatting rules in `.claude/guides/setup-workflow.md` (ask ONE question at a time, detect project name from folder, format questions on separate lines)

This single command intelligently handles all scenarios:

**New project (no code)?** → Starts interactive questionnaire, creates TWO documents:
- `/docs/prd.md` (comprehensive Product Requirements Document for humans)
- `.claude/projectbrief.md` (concise technical brief for AI)
Then installs all required stack modules

**Existing codebase?** → Analyzes code to detect tech stack, generates documentation automatically, recommends modules based on what's already implemented

**Project brief exists?** → Analyzes requirements, recommends missing modules, configures everything

**Partial setup?** → Identifies gaps, suggests additions, completes configuration

**Fully configured?** → Validates setup, suggests optimizations

**See:**
- `.claude/guides/setup-workflow.md` - Complete workflow and all scenarios
- `.claude/guides/existing-codebase-setup.md` - Adding to existing projects
- `templates/prd-template.md` - Full PRD template for humans
- `templates/projectbrief-template.md` - Concise brief template for AI

---

# Automatic Behaviors & Triggers

**IMPORTANT:** Claude should behave **proactively**, not reactively. Follow these automatic triggers to enhance the development workflow.

## Session Management

### On Session Start
When a user opens Claude Code in a project with `.claude/` folder:

1. **Read** `tasks.md` to understand current project state
2. **Identify** last worked-on task (check for status "in progress")
3. **Check** for uncommitted changes
4. **Greet** user with context:
   ```
   "Welcome back! I see you were working on Task 3: User Authentication.
   Current progress: 60% complete (3/5 checkboxes done).

   Would you like to:
   A. Continue with Task 3 (implement error handling)
   B. Start a different task
   C. Review project status"
   ```

### On Session End
When user says "done", "finished", "that's all", or similar:

1. **Check** for uncommitted changes
2. **Prompt** to commit if meaningful work done
3. **Update** `tasks.md` with session notes
4. **Suggest** next steps:
   ```
   "Before you go:

   📝 Uncommitted changes: 3 files modified
   Would you like to commit these changes?

   ✅ Today's progress: Task 3 now 80% complete
   ⏭️ Next session: Implement loading states for login flow"
   ```

## Automatic Task Management

### On "commit" or "create commit"

**ALWAYS** follow this workflow:

1. **Analyze** staged/unstaged changes (run `git status` and `git diff`)
2. **Identify** which tasks in `tasks.md` are affected
3. **Update** relevant task checkboxes based on changes
4. **Generate** commit message with task references
5. **Add** implementation notes to task if deviations occurred
6. **Present** for user approval:
   ```
   "📋 Commit Summary:

   Changes detected:
   - Modified: auth.ts, login.tsx
   - Added: validation.ts

   Related to: Task 3 - User Authentication

   Updates to tasks.md:
   ✓ Marked 'Implement login form' as complete
   ✓ Marked 'Add form validation' as complete
   ✓ Task 3 now 60% complete (3/5 done)

   Proposed commit message:
   ---
   feat(auth): implement login form with validation

   - Add login form component with email/password fields
   - Integrate form validation with Zod
   - Add error handling and loading states

   Tasks Updated:
   - Task 3: User Authentication (now 60% complete)

   Related: Task 3

   🤖 Generated with Claude Code
   Co-Authored-By: Claude <noreply@anthropic.com>
   ---

   Approve? (yes/edit/cancel)"
   ```

7. **After commit**, update `tasks.md` with the changes

### On Task Status Change

When a task is marked complete or changes status:

1. **Update** task dependencies
2. **Check** if other tasks are now unblocked
3. **Notify** user of newly available tasks:
   ```
   "✅ Task 3 marked complete!

   This unblocks:
   - Task 7: Content Generation (all dependencies satisfied)
   - Task 9: User Dashboard (waiting on Task 7)

   Suggested next task: Task 7 (estimated: 6 hours)"
   ```

4. **Update** project metrics in `tasks.md`

### On Pattern Detection (3+ Uses)

When you notice the same pattern used 3 or more times:

1. **Extract** the pattern
2. **Suggest** documentation:
   ```
   "💡 I notice you're using this API pattern 3 times:

   ```typescript
   try {
     const response = await fetch('/api/...')
     const data = await response.json()
     if (!response.ok) throw new Error(data.error)
     return data
   } catch (error) {
     toast.error(error.message)
   }
   ```

   Should I add this as a standard pattern to:
   - `.claude/03-api-server-action.md` (API patterns section)

   This will help maintain consistency. Add it? (yes/no)"
   ```

## Smart Suggestions

### After 3 Commits Without Doc Updates

```
"📚 Notice: We've made 3 commits without updating stack modules.

Recent patterns added:
- New API error handling approach
- Custom authentication hook
- Reusable form validation pattern

Should I review these and suggest documentation updates? (yes/no)"
```

### When Task Near Completion

When task is >85% complete:

```
"🎯 Task 4 is almost done! (9/10 checkboxes complete)

Only remaining:
- Add integration tests

Want to prioritize finishing this task? (yes/no)"
```

### When Dependencies Resolved

```
"✨ Good news! Task 7 dependencies are now satisfied:
✓ Task 1: Project Setup
✓ Task 2: Database Schema
✓ Task 4: Authentication

Ready to start Task 7: Content Generation? (yes/start later)"
```

## Pre-Commit Quality Gates

Before allowing ANY commit, check:

1. ❌ **No console.log statements** (unless approved)
2. ❌ **No unlinked TODOs** (must reference task or issue)
3. ❌ **No hard-coded secrets** (API keys, passwords)
4. ✅ **TypeScript strict mode compliance**
5. ✅ **Related task checkboxes updated**

If violations found:

```
"⚠️ Pre-commit checks failed:

Issues found:
1. console.log statement at auth.ts:45
2. TODO without task reference at validation.ts:12
3. Possible API key at config.ts:8 (check if secret)

Options:
A. Fix issues automatically (remove console.logs, link TODOs)
B. Review and fix manually
C. Override (add justification to commit message)

What would you like to do? (A/B/C)"
```

## Contextual File Loading

When user mentions a task, **automatically** load related context:

```
User: "Let's work on Task 5"

Claude:
"Loading context for Task 5: Payment Integration...
✓ Read: .claude/payments-stripe.md
✓ Read: projectbrief.md (payment requirements section)
✓ Read: tasks.md (Task 5 details and dependencies)
✓ Scanned: 2 existing payment-related files

Task 5 breakdown:
- Set up Stripe checkout (pending)
- Handle webhooks (pending)
- Create customer portal (pending)

According to payments-stripe.md, recommended approach:
1. Start with checkout session
2. Set up webhook endpoint
3. Add customer portal

Ready to begin with checkout? (yes/different approach)"
```

## Quick Commands

Recognize these user commands and respond appropriately:

| Command | Action |
|---------|--------|
| `status` | Show current task status, progress, and velocity metrics |
| `next` | Suggest next task based on dependencies and priorities |
| `sync docs` | Force documentation sync check, suggest updates |
| `commit task X` | Commit current work and link to Task X |
| `metrics` | Show velocity, estimation accuracy, time tracking |
| `blockers` | List all blocked tasks and reasons |
| `review patterns` | Scan code for undocumented reusable patterns |

**Example:**

```
User: "status"

Claude:
"📊 Project Status

Current Sprint:
✅ Task 1: Project Setup (complete)
✅ Task 2: Database Schema (complete)
🔄 Task 3: User Authentication (in progress - 80% complete)
⏸️ Task 4: API Routes (pending, blocked by Task 3)

This Week's Velocity:
- Tasks completed: 2
- Total time: 16 hours
- Estimation accuracy: 90%

Active blockers: 1
- Task 4 waiting on Task 3 completion

Suggested focus: Complete Task 3 (2-3 hours remaining)"
```

## Health Checks & Maintenance

### Weekly (every 7 days since last check)

Run automatic health check:

```
"🔍 Weekly documentation health check:

Findings:
✓ All cross-references valid
✓ No broken links detected
✓ All tasks have proper tags
⚠️ Found 2 undocumented patterns (used 3+ times each)

Suggestions:
1. Document API error handling pattern in 03-api-server-action.md
2. Document custom hook pattern in 02-frontend-component.md

Run documentation update? (yes/later)"
```

### On Dependency Added (package.json change)

```
"📦 New dependency detected: @tanstack/react-query

Recommendations:
1. Update projectbrief.md with new dependency
2. Consider creating stack module: tooling-tanstack-query.md
3. Document integration patterns

Should I:
A. Update projectbrief.md
B. Create new stack module guide
C. Both
D. Skip

Your choice? (A/B/C/D)"
```

---

# Ultracite Code Standards

This project uses **Ultracite**, a zero-config Biome preset that enforces strict code quality standards through automated formatting and linting.

## Quick Reference

- **Format code**: `npx ultracite fix`
- **Check for issues**: `npx ultracite check`
- **Diagnose setup**: `npx ultracite doctor`

Biome (the underlying engine) provides extremely fast Rust-based linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

### Security

- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

### Framework-Specific Guidance

**Next.js:**
- Use Next.js `<Image>` component for images
- Use `next/head` or App Router metadata API for head elements
- Use Server Components for async data fetching instead of async Client Components

**React 19+:**
- Use ref as a prop instead of `React.forwardRef`

**Solid/Svelte/Vue/Qwik:**
- Use `class` and `for` attributes (not `className` or `htmlFor`)

---

## Testing

- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting

## When Biome Can't Help

Biome's linter will catch most issues automatically. Focus your attention on:

1. **Business logic correctness** - Biome can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - Component structure, data flow, and API design
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Accessibility, performance, and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

---

Most formatting and common issues are automatically fixed by Biome. Run `npx ultracite fix` before committing to ensure compliance.
