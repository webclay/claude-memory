# Best Practices for Working with Claude Memory

**Purpose:** Guidelines for developers and AI assistants to maximize the effectiveness of the Claude Memory system.

**Last Updated:** 2025-01-16

---

## 1. Automatic Task Management

### Task Tagging System

Every task in `tasks.md` should include tags for automatic tracking and filtering:

**Tag Format:**
```markdown
## Task 3: User Authentication
**Status:** in progress
**Priority:** High
**Tags:** #backend #auth #security #database
**Dependencies:** Task 1, Task 2
**Estimated Time:** 8 hours
**Actual Time:** [Track as work progresses]
```

**Standard Tags:**
- **Domain:** `#frontend`, `#backend`, `#database`, `#api`, `#ui`, `#devops`
- **Type:** `#feature`, `#bug`, `#refactor`, `#docs`, `#test`, `#security`
- **Priority:** `#critical`, `#high`, `#medium`, `#low`
- **Stack:** `#nextjs`, `#prisma`, `#shadcn`, `#vercel`, etc.

### Automatic Update Triggers

Claude should **automatically** update `tasks.md` when:

```
✅ Creating a commit → Mark related tasks as in_progress or complete
✅ User says "done with [task]" → Update status to complete
✅ User says "starting [task]" → Update status to in_progress
✅ PR merged → Mark all related tasks as complete
✅ Build fails → Add note to relevant task with error details
✅ Tests pass → Update test checkbox in task
```

---

## 2. Smart Commit Workflow

### Commit Message Automation

When user says "commit" or "create commit", Claude should:

1. **Analyze staged/unstaged changes**
2. **Match changes to tasks in tasks.md**
3. **Update task checkboxes** based on what's being committed
4. **Generate commit message** with task references
5. **Update documentation** if patterns changed

**Example Flow:**

```
User: "commit"

Claude:
1. Analyzes: "Changes affect auth system (login.tsx, auth.ts)"
2. Finds: Task 3 - User Authentication
3. Updates: Marks "Implement login form" checkbox as complete
4. Generates commit message:
   "feat(auth): implement login form with validation

   - Add login form component with email/password fields
   - Integrate form validation with Zod
   - Add error handling and loading states
   - Update Task 3: User Authentication (40% complete)

   Related: Task 3
   "
5. Adds note to Task 3 in tasks.md about implementation details
```

### Commit Template

```
<type>(<scope>): <short description>

<detailed description>

Changes:
- [Bullet point of changes]
- [Another change]

Tasks Updated:
- Task X: [status change]
- Task Y: [checkbox updates]

Related: Task X, Task Y

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## 3. Documentation Auto-Sync

### When to Update Stack Modules

Claude should **proactively suggest** updating stack modules when:

```
✅ New pattern used 3+ times → "I notice you're using this pattern repeatedly. Should I add it to [stack-module].md?"
✅ Library version upgraded → "Update [stack-module].md with new API patterns?"
✅ Custom hook created → "Add this hook to 02-frontend-component.md?"
✅ New API pattern → "Document this in 03-api-server-action.md?"
✅ Security fix applied → "Update security section in relevant module?"
```

### Automatic Pattern Detection

**Pattern Recognition Rules:**

1. **Repeated Code (3+ instances)** → Suggest creating reusable utility/component
2. **New Technology Added** → Recommend creating new stack module
3. **Breaking Change** → Flag for documentation update
4. **Performance Optimization** → Document in best practices section

---

## 4. Session Management

### Session Start Protocol

**When starting a new session**, Claude should:

1. **Read** `tasks.md` to understand current state
2. **Identify** last worked-on task
3. **Check** for any pending TODOs or blockers
4. **Ask** user: "I see you were working on Task X. Would you like to continue, or start something new?"

### Session End Protocol

**When user indicates end of session**, Claude should:

1. **Review** uncommitted changes
2. **Prompt** to commit if meaningful work done
3. **Update** `tasks.md` with session notes
4. **Suggest** next steps for future sessions

**Example:**
```
User: "I'm done for today"

Claude:
"Before you go, let me help wrap up:

🔍 Uncommitted changes detected:
- Modified: auth.ts, login.tsx
- Added: validation.ts

Would you like to:
A. Commit these changes now
B. Stash for later
C. Discard changes

Also, I'll update Task 3 with today's progress. You completed:
✅ Login form component
✅ Form validation
⏳ Still pending: error handling

Next session suggestion: Implement error handling and loading states for login flow."
```

---

## 5. Progress Tracking

### Automatic Time Tracking

Track estimated vs actual time in tasks:

```markdown
## Task 5: Payment Integration
**Status:** complete
**Estimated Time:** 6 hours
**Actual Time:** 8.5 hours
**Time Breakdown:**
- Setup (1.5h)
- Stripe integration (3h)
- Webhook handling (2h)
- Testing (2h)

**Lessons Learned:**
- Webhook signature verification took longer than expected
- Testing edge cases added 2h
```

### Velocity Metrics

Automatically track:
- Tasks completed per week
- Average task completion time
- Estimation accuracy (estimated vs actual)
- Blocked tasks and reasons

Add to `tasks.md`:
```markdown
## Project Metrics

**Week of Jan 8-14, 2025:**
- Tasks completed: 3
- Total time: 24 hours
- Estimation accuracy: 85%
- Blockers resolved: 2

**Velocity trend:** ↑ improving
```

---

## 6. Proactive Reminders

### Smart Suggestions

Claude should proactively suggest:

**After 3 commits without updating docs:**
```
"I notice we've made several commits without updating the stack modules.
Should I review the recent changes and suggest documentation updates?"
```

**When task has many checkboxes complete:**
```
"Task 4 is 90% complete! Only 2 items remaining. Want to prioritize finishing this?"
```

**When dependencies are resolved:**
```
"Task 7 dependencies are now complete (Tasks 1-4). Ready to start Task 7?"
```

**When similar work patterns emerge:**
```
"I've noticed you're following a consistent API pattern. Should I document this
as the standard pattern in 03-api-server-action.md?"
```

---

## 7. Code Quality Gates

### Pre-Commit Checks

Before allowing commit, Claude should verify:

```
✅ No console.log statements (or approved with reason)
✅ No TODOs without issue reference
✅ Tests written for new features
✅ No hard-coded secrets or API keys
✅ TypeScript strict mode compliance
✅ Related task checkboxes updated
```

**Example:**
```
User: "commit"

Claude:
"⚠️ Pre-commit check found issues:

1. Found 2 console.log statements:
   - auth.ts:45
   - login.tsx:28

2. Found TODO without reference:
   - validation.ts:12: // TODO: add phone validation

Would you like to:
A. Remove console.logs and fix TODO
B. Add TODO to tasks.md and commit with note
C. Cancel commit and fix manually
```

---

## 8. Smart Context Loading

### Contextual File Loading

When user mentions a task, **automatically load relevant files**:

```
User: "Let's work on Task 5: Payment Integration"

Claude:
"Loading context for Task 5...
✓ Read: .claude/payments-stripe.md
✓ Read: projectbrief.md (payment requirements)
✓ Read: tasks.md (Task 5 details)
✓ Scanned: existing payment-related files

I see you need to:
1. Set up Stripe checkout
2. Handle webhooks
3. Create customer portal

The payments-stripe.md guide recommends starting with checkout.
Shall we begin there?"
```

---

## 9. Cross-Reference Management

### Automatic Link Updates

When files are renamed or moved, update all references:

```
File renamed: auth-config.ts → auth.config.ts

Claude:
"Updating 7 references to auth-config.ts across:
- tasks.md
- auth-better-auth.md
- 00-start-here.md
- README.md (if exists)"
```

### Dead Link Detection

Periodically scan for broken references:

```
Claude (weekly):
"🔍 Documentation health check:

Found 2 broken references:
- tasks.md:45 references deleted file "old-api.ts"
- 00-start-here.md:120 references non-existent "06-deployment.md"

Should I fix these?"
```

---

## 10. Learning & Improvement

### Pattern Learning

Track which patterns work well:

```markdown
## Successful Patterns

**Auth Implementation (Task 3):**
✅ Used Better Auth following auth-better-auth.md
✅ Completed in estimated time
✅ Zero security issues
✅ High code quality

**Lesson:** Better Auth documentation was excellent.
Recommend for future auth implementations.

**Payment Integration (Task 5):**
❌ Estimated 6h, took 8.5h
⚠️ Webhook setup was complex
✅ Eventually successful

**Lesson:** Add 30% buffer for webhook/async integrations.
Update estimation guidelines.
```

### Anti-Patterns

Document what didn't work:

```markdown
## Anti-Patterns Discovered

**Avoid:**
- Starting new tasks before completing current ones (context switching cost high)
- Committing without updating tasks.md (lost progress tracking)
- Skipping tests "for later" (never happens, adds technical debt)
```

---

## 11. Automated Triggers (Implementation)

### .claude/triggers.md

Create a triggers system Claude checks on every interaction:

```markdown
# Automatic Triggers

## On "commit" or "create commit"
1. Check git status
2. Identify changed files
3. Match to tasks in tasks.md
4. Update relevant task checkboxes
5. Add implementation notes
6. Generate commit message with task references
7. Prompt user to review before committing

## On task status change
1. Update dependencies
2. Unblock dependent tasks
3. Notify about newly available tasks
4. Update project metrics

## On session start
1. Read tasks.md
2. Identify last worked task
3. Check for blockers
4. Suggest continuation or new task

## On pattern detection (3+ uses)
1. Extract pattern
2. Suggest documentation location
3. Offer to update relevant stack module

## On test failures
1. Identify related task
2. Add note to task about failure
3. Suggest debugging steps from relevant stack module

## On dependency added
1. Check if new stack module needed
2. Update package list in projectbrief.md
3. Document integration pattern
```

---

## 12. User Commands

### Quick Commands

Teach users these commands for efficiency:

```
"status" → Show current task status and progress
"next" → Suggest next task to work on
"sync docs" → Force documentation sync check
"commit task X" → Commit current work and mark task X updates
"archive completed" → Move completed tasks to archive section
"metrics" → Show velocity and estimation accuracy
"blockers" → List all blocked tasks
"review patterns" → Check for undocumented patterns
```

---

## Summary: What Changes

### Current Behavior:
- Manual task updates
- Manual commit messages
- No automatic pattern detection
- No progress tracking
- Reactive documentation

### Improved Behavior:
- ✅ Automatic task tagging and updates
- ✅ Smart commit workflow with task linking
- ✅ Proactive pattern documentation suggestions
- ✅ Automatic progress and velocity tracking
- ✅ Pre-commit quality gates
- ✅ Contextual file loading
- ✅ Session management (start/end protocols)
- ✅ Dead link detection and fixes
- ✅ Smart suggestions based on work patterns

### Implementation Priority:

**Phase 1 (Immediate):**
1. Automatic task updates on commit
2. Smart commit message generation
3. Session start/end protocols

**Phase 2 (Short-term):**
4. Pattern detection and documentation suggestions
5. Pre-commit quality gates
6. Contextual file loading

**Phase 3 (Long-term):**
7. Velocity metrics tracking
8. Dead link detection
9. Learning from patterns
10. Advanced automation triggers

---

**Key Principle:** The system should be **proactive, not reactive**. Claude should anticipate needs and suggest actions rather than waiting for explicit commands.
