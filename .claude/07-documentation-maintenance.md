# 07 - Documentation Maintenance & Commit Workflow

**Purpose:** Ensure documentation stays synchronized with code changes through automated update protocols.

**Last Updated:** 2025-11-05

---

## Overview

This playbook establishes rules for Claude Code to automatically maintain documentation integrity throughout the development lifecycle. Every commit should trigger documentation verification and updates.

---

## 1. tasks.md Update Protocol

### Purpose of tasks.md

The `tasks.md` file tracks implementation progress across all major development phases. It serves as:
- Project status dashboard
- Context for Claude Code sessions
- Historical record of implementation decisions
- Dependency tracker between features

### When to Update tasks.md

**ALWAYS** check and update `tasks.md` when:

1. **Before Creating Any Commit:**
   - Review the changes being committed
   - Identify which task(s) in `tasks.md` are affected
   - Prepare update summary for commit message

2. **After Successful Commit:**
   - Update checkbox status `[x]` for completed sub-tasks
   - Update Task Status field if milestone reached
   - Add implementation notes if deviations occurred
   - Document any new dependencies discovered

3. **When User Requests:**
   - "update tasks"
   - "update documentation"
   - "mark [feature] as complete"

### Automatic Triggers

Claude Code should **automatically** verify `tasks.md` when:

```
✓ User says "commit", "create commit", or "git commit"
✓ After successful commit creation
✓ When closing a development session with uncommitted work
✓ When asked about project status or progress
✓ When starting work on a new feature mentioned in tasks.md
```

## Smart Commit Workflow

### Complete Commit Automation

When user requests a commit, **ALWAYS** follow this complete workflow:

**Step 1: Pre-Commit Analysis**
```bash
# Run these commands
git status
git diff
```

Analyze output to determine:
- Which files changed
- What functionality was modified
- Which tasks in `tasks.md` are affected

**Step 2: Identify Related Tasks**

Match changed files to tasks:
```markdown
Changed files: auth.ts, login.tsx, validation.ts
→ Matches Task 3: "User Authentication"
→ Specifically relates to checkboxes:
  - "Implement login form" → Changed login.tsx
  - "Add form validation" → Changed validation.ts
  - "Integrate auth service" → Changed auth.ts
```

**Step 3: Update Task Checkboxes**

Before committing, update `tasks.md`:
```markdown
BEFORE:
## Task 3: User Authentication
- [ ] Implement login form
- [ ] Add form validation
- [ ] Integrate auth service

AFTER:
## Task 3: User Authentication
- [x] Implement login form
- [x] Add form validation
- [x] Integrate auth service
  Note: Added rate limiting (5 attempts/hour)
```

**Step 4: Calculate Task Progress**

Count checkboxes:
- Total checkboxes: 5
- Completed: 3
- Progress: 60%

Update task header:
```markdown
## Task 3: User Authentication
**Status:** in progress → in progress (60% complete)
**Tags:** #backend #auth #security
```

**Step 5: Generate Commit Message**

Use this template:
```
<type>(<scope>): <short description>

<detailed description of changes>

Changes:
- [Specific change 1]
- [Specific change 2]
- [Specific change 3]

Tasks Updated:
- Task X: [status change and progress]

Related: Task X

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

Example:
```
feat(auth): implement login form with validation

Add user login functionality with comprehensive validation
and security measures.

Changes:
- Create login form component with email/password fields
- Implement Zod validation schema for form inputs
- Add error handling and loading states
- Integrate with auth service (rate limiting: 5/hour)

Tasks Updated:
- Task 3: User Authentication (now 60% complete, 3/5 done)

Related: Task 3

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

**Step 6: Present for Approval**

Show user complete summary:
```
📋 Commit Summary:

Files changed (3):
✓ auth.ts (modified)
✓ login.tsx (modified)
✓ validation.ts (added)

Related to: Task 3 - User Authentication

Updates to tasks.md:
✓ Marked 'Implement login form' as complete
✓ Marked 'Add form validation' as complete
✓ Marked 'Integrate auth service' as complete
✓ Added implementation note about rate limiting
✓ Task 3 progress: 40% → 60% (3/5 checkboxes)

Commit message:
---
[Show generated message]
---

Options:
A. Approve and commit
B. Edit commit message
C. Cancel

Your choice? (A/B/C)
```

**Step 7: Execute Commit**

If approved:
1. Update `tasks.md` file with checkbox changes
2. Stage `tasks.md`: `git add .claude/tasks.md`
3. Create commit with generated message
4. Confirm success to user

**Step 8: Post-Commit Actions**

After successful commit:
1. Check if task is now complete
2. If complete, check for dependent tasks
3. Notify about unblocked tasks
4. Update project metrics
5. Suggest next action

### Update Procedure

**Step 1: Identify Affected Tasks**
```markdown
Example: If committing Auth Module fixes
→ Check Task 3: "Authentication & Authorization"
→ Verify which checkboxes relate to the changes
```

**Step 2: Update Checkboxes**
```markdown
BEFORE:
- [ ] Implement magic link authentication

AFTER:
- [x] Implement magic link authentication
  Note: Added rate limiting (5 attempts/hour)
```

**Step 3: Update Task Status**

Status definitions:
- `pending` - Not started, no checkboxes marked
- `in progress` - Some checkboxes marked, active development
- `mostly complete` - Major work done, minor items/polish remaining
- `complete` - All checkboxes marked, fully tested, merged

**Step 4: Document Decisions**

Add notes for:
- Architecture changes from original plan
- New dependencies or libraries added
- Performance considerations discovered
- Security decisions made
- Breaking changes

**Step 5: Update Dependencies**

If a task completion unblocks others:
```markdown
Task 7: Content Generation Modules
Dependencies: 1, 2, 3, 4 ← Update if dependencies change
```

### tasks.md Verification Checklist

Before finalizing any commit, verify:

```
□ Relevant task checkboxes updated
□ Task status reflects current state
□ Implementation notes added for deviations
□ Dependencies updated if changed
□ Test strategy checkboxes updated
□ No orphaned references to renamed features
```

---

## 2. Internal Documentation Update Rules

### Specialist Playbooks (02-06)

Update specialist playbooks when:

**02-frontend-component.md**
- ✓ New UI component pattern emerges and is reused 3+ times
- ✓ Design system tokens/components added
- ✓ New animation or interaction pattern established
- ✓ Icon system changes (HugeIcons usage patterns)
- ✓ Form handling patterns evolve

**03-api-server-action.md**
- ✓ New API route pattern created
- ✓ Authentication/authorization pattern changes
- ✓ Error handling convention updated
- ✓ Streaming implementation patterns added
- ✓ Rate limiting or security measures implemented

**04-testing.md**
- ✓ New testing pattern adopted
- ✓ Testing library versions updated
- ✓ Mock/fixture patterns established
- ✓ E2E testing conventions changed

**05-database-supabase.md**
- ✓ New migration patterns used
- ✓ RLS policy templates created
- ✓ Realtime subscription patterns added
- ✓ Database schema conventions changed
- ✓ Edge Function database access patterns

**06-deployment-edge-functions.md**
- ✓ New Edge Function created
- ✓ Deployment process changed
- ✓ Environment variable added
- ✓ CORS or security configuration updated
- ✓ Streaming patterns for OpenRouter

### Core Foundation Files

**projectbrief.md**
- ✓ Product requirements change
- ✓ New features added to roadmap
- ✓ Tech stack changes
- ✓ Architecture decisions updated
- **SYNC:** Always keep `/.claude/projectbrief.md` and `/docs/projectbrief.md` identical

**standards.md**
- ✓ New coding conventions adopted
- ✓ Module template changes
- ✓ File naming conventions updated
- ✓ Git workflow process changes

**supabase.md**
- ✓ Supabase configuration changes
- ✓ New Supabase features adopted
- ✓ RLS patterns updated

### Documentation Synchronization Rules

**Bidirectional Sync Required:**

```
/.claude/projectbrief.md  ↔  /docs/projectbrief.md
```

**Commit-Time Verification:**

When committing changes to **ANY** of these files, Claude Code must:
1. Check if the counterpart file exists
2. Verify content matches
3. Update both files if discrepancies found
4. Document sync in commit message

**Module-Specific Documentation:**

When modifying modules (e.g., Ad Hooks, Mini VSL), check if related docs exist:

```
/docs/README_AD_HOOKS.md
/docs/AD_HOOKS_DATA_STRUCTURE.md
/docs/AD_HOOKS_GROUPING_EXAMPLE.md
/docs/AD_HOOKS_PROMPT_LOCATION.md
```

Update if:
- Data structures changed
- Prompt location changed
- New features added
- API contracts modified

---

## 3. Commit Workflow Integration

### Pre-Commit Documentation Checklist

**BEFORE** creating any commit, Claude Code should:

```
1. □ Check if changes affect tasks.md items
2. □ Identify which specialist playbooks may need updates
3. □ Verify /docs and /.claude sync if applicable
4. □ Check if README.md needs dependency/setup updates
5. □ Prepare documentation update list
```

### Commit Message Documentation References

Use Conventional Commits with documentation indicators:

```bash
# Feature commits that complete tasks.md items
feat(auth): implement magic link authentication

Updates tasks.md: Task 3, checkbox 2 complete
See: .claude/03-api-server-action.md for pattern

# Documentation updates
docs(tasks): update Task 7 progress - 5/7 modules complete

# Sync commits
docs(sync): synchronize projectbrief.md across .claude and docs

# Pattern documentation
docs(frontend): add streaming response pattern to playbook

Updates: .claude/02-frontend-component.md
```

### Post-Commit Automation

**AFTER** successful commit, Claude Code should:

```
1. ✓ Verify tasks.md was updated if applicable
2. ✓ Confirm all checkboxes match committed work
3. ✓ Check if any specialist playbooks referenced new patterns
4. ✓ Suggest README updates if setup/dependencies changed
5. ✓ Prompt for documentation sync if needed
```

### Commit-Time Prompts

Claude Code should ASK the user:

**When uncommitted documentation updates detected:**
```
"I've updated tasks.md to reflect the completed work. Should I include
this in the commit, or would you like to review it first?"
```

**When new patterns emerged:**
```
"This commit introduces a new [authentication/UI/API] pattern. Should
I document this in [relevant specialist playbook] for future reference?"
```

**When README may be stale:**
```
"This commit changes [dependencies/setup/environment]. The README may
need updates. Should I review and update it?"
```

---

## 4. Documentation Debt Prevention

### Real-Time Documentation

**Document WHILE coding, not after:**

- When creating new component patterns → Note for `02-frontend-component.md`
- When implementing API patterns → Note for `03-api-server-action.md`
- When completing task items → Update `tasks.md` immediately
- When discovering gotchas → Add to relevant specialist playbook

### Weekly Documentation Review Trigger

If Claude Code detects:
- 10+ commits since last `tasks.md` update
- Specialist playbook not updated in 20+ commits
- Documentation sync drift between `/.claude` and `/docs`

**Action:** Prompt user to review documentation health

### Documentation Health Indicators

**GREEN (Healthy):**
- ✓ tasks.md updated within last 3 commits
- ✓ Specialist playbooks referenced in recent commits
- ✓ No sync drift detected
- ✓ README matches current setup

**YELLOW (Attention Needed):**
- ⚠ tasks.md not updated in 5+ commits
- ⚠ New patterns not documented
- ⚠ Potential sync drift

**RED (Critical):**
- ❌ tasks.md not updated in 10+ commits
- ❌ Specialist playbooks outdated
- ❌ README doesn't match actual setup
- ❌ /docs and /.claude out of sync

---

## 5. Automation Decision Tree

```
┌─────────────────────────┐
│   User Requests Commit   │
└────────────┬─────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│ Does commit affect tasks.md items?  │
└──────────┬──────────────────┬───────┘
       YES │                  │ NO
           ▼                  │
┌─────────────────────┐      │
│ Update tasks.md:     │      │
│ - Mark checkboxes    │      │
│ - Update status      │      │
│ - Add notes          │      │
└──────────┬───────────┘      │
           │                  │
           └──────────┬───────┘
                      ▼
┌─────────────────────────────────────┐
│ Does commit introduce new patterns? │
└──────────┬──────────────────┬───────┘
       YES │                  │ NO
           ▼                  │
┌─────────────────────────┐  │
│ Update specialist        │  │
│ playbook (02-06)         │  │
└──────────┬───────────────┘  │
           │                  │
           └──────────┬───────┘
                      ▼
┌─────────────────────────────────────┐
│ Do /docs and /.claude need sync?    │
└──────────┬──────────────────┬───────┘
       YES │                  │ NO
           ▼                  │
┌─────────────────────────┐  │
│ Sync projectbrief.md     │  │
│ and other docs           │  │
└──────────┬───────────────┘  │
           │                  │
           └──────────┬───────┘
                      ▼
┌─────────────────────────────────────┐
│ Does README need updates?           │
└──────────┬──────────────────┬───────┘
       YES │                  │ NO
           ▼                  │
┌─────────────────────────┐  │
│ Update README.md:        │  │
│ - Dependencies           │  │
│ - Setup steps            │  │
│ - Environment vars       │  │
└──────────┬───────────────┘  │
           │                  │
           └──────────┬───────┘
                      ▼
┌─────────────────────────────────────┐
│      Create Commit with Docs        │
│   Include documentation updates     │
│   in commit or separate commit      │
└─────────────────────────────────────┘
```

---

## 6. Special Cases

### When Task Status Changes to "Complete"

**Additional actions:**
1. ✓ Verify ALL checkboxes are marked
2. ✓ Confirm test strategy checkboxes complete
3. ✓ Check if dependent tasks can now start
4. ✓ Document completion date
5. ✓ Celebrate milestone 🎉 (if user prefers emojis)

### When Major Refactoring Occurs (e.g., Task 11: Products→Offers Rename)

**Documentation audit required:**
1. Search all `.claude` files for old terminology
2. Update all specialist playbooks with new names
3. Update `tasks.md` references
4. Update `/docs` files
5. Update README examples
6. Commit with comprehensive docs update message

### When Dependencies Change

**If package.json, requirements.txt, or similar updated:**
1. ✓ Update README.md installation instructions
2. ✓ Update `.claude/standards.md` if convention changes
3. ✓ Document reason for dependency change
4. ✓ Note version constraints if important

### When Environment Variables Added

**If .env.example or similar updated:**
1. ✓ Update README.md environment setup section
2. ✓ Update `.claude/06-deployment-edge-functions.md` if Edge Function related
3. ✓ Update `/docs/README-supabase.md` if Supabase related
4. ✓ Document purpose and security considerations

---

## 7. Integration with Existing Files

### Reference from 00-start-here.md

This file should be listed under **Core Foundation Files** section:
```markdown
4. `tasks.md` - Implementation plan tracking
5. `07-documentation-maintenance.md` - Documentation update protocols ← NEW
```

### Reference from 01-rules-definitions.md

Add to **Documentation Structure** section:
```markdown
- `07-documentation-maintenance.md`: Automated documentation update rules
  - tasks.md update protocols
  - Commit workflow integration
  - Documentation synchronization
```

### Reference from standards.md

Extend **Git Workflow** section (line ~140):
```markdown
## Git Workflow

See `.claude/07-documentation-maintenance.md` for comprehensive documentation
update protocols during commit workflow.

- Use Conventional Commit messages (feat:, fix:, docs:, refactor:)
- Update tasks.md for any commit affecting implementation items
- Sync documentation between /docs and /.claude when applicable
- Document new patterns in specialist playbooks
```

---

## 8. Claude Code Behavioral Rules

### Mandatory Behaviors

Claude Code **MUST**:
1. ✓ Check `tasks.md` before every commit
2. ✓ Update `tasks.md` after completing any work mentioned in it
3. ✓ Ask about documentation when introducing new patterns
4. ✓ Verify sync between `/.claude` and `/docs` when applicable
5. ✓ Proactively suggest documentation updates

Claude Code **MUST NOT**:
1. ❌ Commit code changes without checking `tasks.md`
2. ❌ Skip documentation updates to "save time"
3. ❌ Assume documentation will be updated later
4. ❌ Create new patterns without documenting them
5. ❌ Let sync drift persist between doc folders

### Default Assumptions

**Unless user explicitly says otherwise:**
- Include `tasks.md` updates in the same commit as code changes
- Ask before adding new specialist playbook sections
- Always sync `projectbrief.md` between folders
- Suggest README updates but don't auto-update without confirmation

### User Override

Users can say:
- "Skip docs" → Skip documentation updates for this commit only
- "Docs later" → Add TODO reminder but proceed with commit
- "No README" → Don't suggest README updates this session

---

## 9. Quick Reference Commands

### For Users

```bash
# Ask Claude to verify documentation status
"Check documentation health"
"Is tasks.md current?"
"Review documentation sync"

# Request specific updates
"Update tasks.md for the work we just did"
"Document this pattern in the frontend playbook"
"Sync projectbrief.md"

# Commit with auto-documentation
"Commit this work" → Claude will auto-check and update docs
```

### For Claude Code

```bash
# Before commits
1. grep tasks.md for pending items related to changes
2. Check if new patterns match existing playbook sections
3. Verify /.claude and /docs projectbrief.md match

# During commits
1. Update tasks.md inline with code changes
2. Add documentation references to commit message
3. Include specialist playbook updates if applicable

# After commits
1. Verify tasks.md status reflects completion
2. Confirm no orphaned documentation references
3. Check documentation health indicators
```

---

## 10. Examples

### Example 1: Completing a Feature

**Scenario:** Just implemented rate limiting for magic links (Task 3, checkbox 2)

**Claude Code Actions:**
```
1. ✓ Read tasks.md Task 3
2. ✓ Update checkbox: [x] Implement magic link authentication
3. ✓ Add note: "Added rate limiting (5 attempts/hour)"
4. ✓ Check 03-api-server-action.md for rate limiting patterns
5. ✓ Create commit:

feat(auth): implement magic link rate limiting

- Add rate limiter middleware (5 attempts/hour)
- Update magic link endpoint with rate check
- Add rate limit headers to response

Updates tasks.md: Task 3, checkpoint 2 complete
Documented pattern: .claude/03-api-server-action.md (Rate Limiting section)
```

### Example 2: Major Refactoring

**Scenario:** Renaming "Products" to "Offers" (Task 11)

**Claude Code Actions:**
```
1. ✓ Audit all .claude files for "Products" terminology
2. ✓ Update:
   - tasks.md (Task 11 description)
   - 02-frontend-component.md (component names)
   - 03-api-server-action.md (API routes)
   - 05-database-supabase.md (table names)
   - projectbrief.md (feature descriptions)
3. ✓ Sync /.claude/projectbrief.md → /docs/projectbrief.md
4. ✓ Update /docs/README.md examples
5. ✓ Create commit:

refactor: rename Products to Offers throughout codebase

- Update all database references
- Rename components and API routes
- Update documentation in .claude and /docs
- Sync projectbrief.md

Updates tasks.md: Task 11 complete
Documentation sync: Complete audit of all .claude files
Breaking change: See migration guide in commit body
```

### Example 3: New Pattern Discovery

**Scenario:** Created reusable streaming response component

**Claude Code Actions:**
```
1. ✓ Identify this as new pattern (used in 3+ places)
2. ✓ Check 02-frontend-component.md for existing streaming patterns
3. ✓ Prompt user:

"I've created a reusable ContentStreamingDisplay component that handles
OpenRouter streaming responses. This pattern is used in Ad Hooks, Mini VSL,
and Headline Master. Should I document this in 02-frontend-component.md?"

4. ✓ If yes, add section to playbook
5. ✓ Include in commit message:

feat(streaming): create reusable streaming response component

- Add ContentStreamingDisplay component
- Implement in Ad Hooks, Mini VSL, Headline Master
- Handle streaming state, errors, and completion

Updates: .claude/02-frontend-component.md (Streaming Patterns section)
Pattern: Reusable streaming display with Markdown rendering
```

---

## Last Words

This documentation maintenance system ensures that:
- ✓ `tasks.md` always reflects current project status
- ✓ Specialist playbooks capture reusable patterns
- ✓ `/docs` and `/.claude` folders stay synchronized
- ✓ New developers can onboard using accurate documentation
- ✓ Claude Code maintains institutional knowledge across sessions

**Make documentation a first-class citizen, not an afterthought.**

---

*This file is itself subject to documentation maintenance! Update it when discovering new automation opportunities or documentation pain points.*
