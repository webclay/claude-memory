# Quick Commands Reference

**Purpose:** Define single-word commands that trigger specific Claude behaviors for efficient workflow management.

**Last Updated:** 2025-01-16

---

## How Commands Work

When the user types a command, Claude should immediately execute the associated behavior without asking for confirmation (unless the command has destructive consequences).

**Format:** User types the command in plain text (no prefix needed)

---

## Available Commands

### `status`

**Triggers:** Show comprehensive project status

**Claude Response:**
```
📊 PROJECT STATUS

Current Task: Task X - [Task Name]
Status: [in progress/pending/blocked]
Progress: [X%] ([Y/Z checkboxes complete])

Recent Activity:
✓ Last commit: [commit message] ([time ago])
✓ Files modified: [count]
⏰ Time on current task: [estimated time]

Next Up:
- Task Y: [name] (dependency resolved)
- Task Z: [name] (blocked by Task A)

Velocity Metrics:
- Tasks this week: X completed
- Estimation accuracy: Y%
- Average task time: Z hours
```

**Implementation:**
1. Read `tasks.md`
2. Parse current task (status = "in progress")
3. Run `git log -1` for last commit
4. Calculate progress percentages
5. Identify next available tasks
6. Show metrics from tasks.md

---

### `next`

**Triggers:** Suggest next task to work on

**Claude Response:**
```
🎯 NEXT TASK RECOMMENDATION

Recommended: Task X - [Task Name]

Why this task?
✓ All dependencies completed (Tasks A, B, C)
✓ High priority
✓ Estimated time: X hours
✓ Aligns with current work context

Alternative options:
- Task Y: [name] (Medium priority, no blockers)
- Task Z: [name] (Low priority, quick win - 2h)

Would you like to start Task X?
```

**Implementation:**
1. Read `tasks.md`
2. Filter tasks: status = "pending", no blocked dependencies
3. Sort by: priority (high→medium→low), then estimated time
4. Consider context: tasks related to recent commits
5. Present top recommendation + 2 alternatives

---

### `sync docs`

**Triggers:** Force documentation synchronization check

**Claude Response:**
```
🔍 DOCUMENTATION SYNC CHECK

Analyzing recent changes...

Found potential updates needed:

📁 Stack Modules:
⚠️ New pattern detected in auth.ts (used 4 times)
   → Suggest adding to: auth-better-auth.md
   → Pattern: Custom session validation with role check

✅ All existing patterns documented

📋 Tasks:
⚠️ Task 3 checkboxes out of sync with commits
   → 2 items completed in code but not marked
   → Auto-updating tasks.md...

🔗 Cross-References:
✅ No broken links found
✅ All file references valid

Would you like me to update auth-better-auth.md with the new pattern?
```

**Implementation:**
1. Scan recent commits (last 10)
2. Check for repeated patterns (3+ uses)
3. Verify tasks.md checkboxes match committed work
4. Scan documentation for broken links
5. Suggest updates

---

### `commit task X`

**Triggers:** Commit current work and link to specific task

**Claude Response:**
```
📝 COMMITTING WORK FOR TASK X

Analyzing changes...

Changed files:
✓ src/auth/login.tsx
✓ src/auth/validation.ts

Matching to Task 3: User Authentication

Updating task checkboxes:
✓ Implement login form component → ✅ Complete
✓ Add form validation → ✅ Complete
⏳ Add error handling → Still pending

Task 3 Progress: 40% → 60%

Generated commit message:
---
feat(auth): implement login form with validation

- Add login form component with email/password fields
- Integrate form validation with Zod schema
- Add loading states during authentication

Tasks Updated:
- Task 3: User Authentication (60% complete)

Related: Task 3

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
---

Proceed with commit? (yes/no)
```

**Implementation:**
1. Replace "X" with task number from user input
2. Run git status and git diff
3. Match changed files to task in tasks.md
4. Update relevant checkboxes
5. Calculate new progress percentage
6. Generate commit message following template
7. Show for approval
8. Execute: update tasks.md, stage, commit

---

### `metrics`

**Triggers:** Show velocity and estimation accuracy

**Claude Response:**
```
📈 PROJECT METRICS

CURRENT WEEK (Jan 8-14, 2025):
Tasks completed: 3
Total time: 24 hours
Estimation accuracy: 85%
Blockers resolved: 2

VELOCITY TREND: ↑ improving

TASK BREAKDOWN:
✅ Task 1: Database Setup
   Estimated: 4h | Actual: 3.5h | Accuracy: 112%

✅ Task 2: Auth Integration
   Estimated: 6h | Actual: 8h | Accuracy: 75%

✅ Task 3: User Dashboard
   Estimated: 8h | Actual: 7h | Accuracy: 114%

INSIGHTS:
✓ Authentication tasks tend to take 25% longer
✓ Database work is well-estimated
⚠️ Consider adding buffer for auth-related tasks

ALL-TIME STATS:
Total tasks completed: 12
Average completion time: 6.2 hours
Overall estimation accuracy: 88%
```

**Implementation:**
1. Read `tasks.md`
2. Parse "Actual Time" and "Estimated Time" from completed tasks
3. Calculate accuracy: (estimated / actual) × 100
4. Group by week using task completion dates
5. Identify patterns (which task types are over/under estimated)
6. Present insights

---

### `blockers`

**Triggers:** List all blocked tasks with reasons

**Claude Response:**
```
🚫 BLOCKED TASKS

CURRENTLY BLOCKED (3 tasks):

Task 7: Payment Integration
├─ Blocked by: Task 5 (Stripe account setup)
├─ Status: Task 5 is 80% complete
└─ ETA to unblock: ~2 hours

Task 9: Email Notifications
├─ Blocked by: External dependency (Resend API approval pending)
├─ Status: Waiting on external team
└─ Action needed: Follow up with Resend support

Task 11: Analytics Dashboard
├─ Blocked by: Task 8, Task 9
├─ Status: Task 8 complete, Task 9 blocked
└─ ETA to unblock: Unknown (depends on Task 9)

RECOMMENDATIONS:
1. Focus on completing Task 5 to unblock Task 7
2. Follow up on Resend API approval
3. Consider alternative tasks: Task 6, Task 10 (no blockers)
```

**Implementation:**
1. Read `tasks.md`
2. Find tasks with status "blocked" or "pending" with dependencies
3. Check dependency status (complete/in_progress/pending)
4. Calculate estimated time to unblock
5. Suggest actionable next steps

---

### `review patterns`

**Triggers:** Check for undocumented patterns in codebase

**Claude Response:**
```
🔍 PATTERN REVIEW

Scanning codebase for repeated patterns...

UNDOCUMENTED PATTERNS FOUND (2):

1. Custom Error Handling Pattern
   ├─ Found in: 5 files
   ├─ Files: api/auth.ts, api/users.ts, api/posts.ts, api/comments.ts, api/payments.ts
   ├─ Pattern: try-catch with custom error class + logging
   └─ Suggestion: Document in 03-api-server-action.md

2. Form Validation Hook
   ├─ Found in: 4 components
   ├─ Files: LoginForm.tsx, SignupForm.tsx, ProfileForm.tsx, SettingsForm.tsx
   ├─ Pattern: useForm with Zod + toast notifications
   └─ Suggestion: Document in 02-frontend-component.md

RECENTLY DOCUMENTED (1):

✓ Session validation pattern
  └─ Added to auth-better-auth.md (3 days ago)

Would you like me to add these patterns to the documentation?
```

**Implementation:**
1. Scan codebase for repeated code structures
2. Use AST parsing or regex to identify similar patterns
3. Count occurrences (3+ = pattern)
4. Check if pattern exists in documentation
5. Suggest documentation location
6. Offer to add to relevant stack module

---

### `archive completed`

**Triggers:** Move completed tasks to archive section

**Claude Response:**
```
📦 ARCHIVING COMPLETED TASKS

Moving 5 completed tasks to archive...

ARCHIVED:
✓ Task 1: Database Setup
✓ Task 2: Auth Integration
✓ Task 3: User Dashboard
✓ Task 4: API Routes
✓ Task 5: Stripe Setup

ACTIVE TASKS REMAINING: 7
├─ In Progress: 1 (Task 6)
├─ Pending: 4 (Tasks 7, 8, 10, 12)
└─ Blocked: 2 (Tasks 9, 11)

Updated tasks.md with archive section.
All metrics preserved for completed tasks.
```

**Implementation:**
1. Read `tasks.md`
2. Find all tasks with status "complete"
3. Create "## Archived Tasks" section at bottom if doesn't exist
4. Move completed tasks to archive
5. Preserve all task metadata (times, tags, notes)
6. Update file
7. Show summary of active vs archived

---

## Command Aliases

Users may use variations - Claude should recognize these:

| User Types | Execute Command |
|------------|-----------------|
| `show status`, `what's the status` | `status` |
| `what's next`, `what should I do next` | `next` |
| `check docs`, `sync documentation` | `sync docs` |
| `show metrics`, `velocity`, `stats` | `metrics` |
| `show blockers`, `what's blocked` | `blockers` |
| `check patterns`, `find patterns` | `review patterns` |
| `archive`, `archive tasks` | `archive completed` |

---

## Contextual Commands

These commands work when user mentions them in context:

**"I'm done for today"** → Trigger Session End Protocol
- Check uncommitted changes
- Suggest commit or stash
- Update tasks.md with session notes
- Suggest next session starting point

**"starting [task name]"** → Auto-update task status
- Find task in tasks.md
- Update status to "in progress"
- Load relevant stack modules
- Show task details and checklist

**"done with [task name]"** → Auto-complete task
- Find task in tasks.md
- Update status to "complete"
- Check for newly unblocked tasks
- Update project metrics
- Notify user of next available tasks

**"commit"** or **"create commit"** → Smart Commit Workflow
- Analyze changes
- Match to tasks
- Update checkboxes
- Generate message
- Present for approval

---

## Pre-Commit Quality Gate Commands

**Automatically triggered before commit:**

**Console.log check:**
```
⚠️ Found 3 console.log statements:
- auth.ts:45
- login.tsx:28
- validation.ts:102

Remove before committing? (yes/no/ignore)
```

**TODO check:**
```
⚠️ Found 2 TODOs without issue reference:
- validation.ts:12: // TODO: add phone validation
- auth.ts:67: // TODO: implement MFA

Add to tasks.md or remove? (add/remove/ignore)
```

**Secrets check:**
```
🚨 SECURITY WARNING
Found potential secret in .env.local:
- Line 5: API_KEY=sk_live_...

This file should not be committed.
Abort commit? (yes/no)
```

---

## Implementation Notes

**For Claude:**

1. **Always execute immediately** - Don't ask "would you like me to run status?" - just run it
2. **Be concise** - Commands are for speed, keep responses focused
3. **Use consistent formatting** - Follow templates above
4. **Track context** - Remember what command was just run to inform next suggestions
5. **Auto-trigger related actions** - If `commit` finds outdated tasks, suggest `sync docs`

**Priority Order:**
1. User's explicit command takes precedence
2. Contextual triggers second (session end, task mentions)
3. Automatic triggers last (pre-commit checks)

---

**Last Updated:** 2025-01-16
**Maintained By:** Claude Memory System
**Status:** Active
