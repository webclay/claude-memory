# Automatic Triggers System

**Purpose:** Define all automatic behaviors Claude should execute based on user actions or system events.

**Last Updated:** 2025-01-16

---

## Trigger Categories

### 1. Session Triggers

| Event | Trigger | Actions |
|-------|---------|---------|
| **Session Start** | User opens Claude Code in project | 1. Read `tasks.md`<br>2. Identify last task (status: "in progress")<br>3. Check for uncommitted changes<br>4. Greet with context and suggestions |
| **Session End** | User says "done", "finished", "that's all" | 1. Check for uncommitted changes<br>2. Prompt to commit if work done<br>3. Update `tasks.md` with session summary<br>4. Suggest next steps |

---

### 2. Commit Triggers

| Event | Trigger | Actions |
|-------|---------|---------|
| **Before Commit** | User says "commit", "create commit" | 1. Run `git status` and `git diff`<br>2. Identify affected tasks in `tasks.md`<br>3. Update task checkboxes<br>4. Generate commit message<br>5. Present for approval |
| **After Commit** | Commit successfully created | 1. Update `tasks.md` with completion status<br>2. Add implementation notes<br>3. Check if tasks unblocked<br>4. Update metrics |
| **Pre-Commit Check** | Before allowing commit | 1. Scan for console.log<br>2. Check for unlinked TODOs<br>3. Detect hard-coded secrets<br>4. Verify task updates<br>5. Present violations or approve |

---

### 3. Task Triggers

| Event | Trigger | Actions |
|-------|---------|---------|
| **Task Mentioned** | User mentions "Task X" or "work on [task]" | 1. Read task details from `tasks.md`<br>2. Load related stack modules<br>3. Scan related code files<br>4. Present context and suggest approach |
| **Task Complete** | Task status → "complete" | 1. Update dependent tasks<br>2. Notify of unblocked tasks<br>3. Suggest next prioritized task<br>4. Update project metrics |
| **Task Near Complete** | Task >85% done | Suggest prioritizing completion |
| **Dependencies Resolved** | All task dependencies satisfied | Notify user task is ready to start |

---

### 4. Pattern Detection Triggers

| Event | Trigger | Actions |
|-------|---------|---------|
| **Repeated Pattern** | Same code pattern used 3+ times | 1. Extract pattern<br>2. Suggest documentation location<br>3. Offer to update stack module |
| **New Technology** | New dependency added to package.json | 1. Detect library<br>2. Suggest creating stack module<br>3. Offer to document integration |

---

### 5. Documentation Triggers

| Event | Trigger | Actions |
|-------|---------|---------|
| **3 Commits Without Docs** | 3 commits made, no doc updates | Suggest reviewing recent patterns for documentation |
| **File Renamed** | File moved or renamed | 1. Scan for references<br>2. Update all documentation links<br>3. Report changes made |
| **Weekly Health Check** | Every 7 days | 1. Scan for broken links<br>2. Check for undocumented patterns<br>3. Verify task tag consistency<br>4. Suggest maintenance |

---

### 6. Quality Triggers

| Event | Trigger | Actions |
|-------|---------|---------|
| **Test Failure** | Tests fail during run | 1. Identify related task<br>2. Add note to task<br>3. Suggest debugging from stack module |
| **Build Error** | Build fails | 1. Capture error details<br>2. Link to relevant task<br>3. Suggest fixes based on stack modules |
| **Linter Issues** | Ultracite/Biome reports issues | Present issues with auto-fix options |

---

## Command Triggers

### Quick Commands

When user types specific commands, respond with these actions:

| Command | Trigger Action |
|---------|---------------|
| `status` | Read `tasks.md` → Show current sprint status, velocity, blockers |
| `next` | Analyze `tasks.md` dependencies → Suggest highest priority available task |
| `sync docs` | Scan codebase → Suggest documentation updates for recent patterns |
| `commit task X` | Run commit workflow → Link commit to specific task |
| `metrics` | Calculate from `tasks.md` → Show velocity, estimation accuracy, time stats |
| `blockers` | Parse `tasks.md` → List all blocked tasks with reasons |
| `review patterns` | Scan codebase → Find patterns used 3+ times, suggest documentation |
| `archive completed` | Move completed tasks to archive section in `tasks.md` |

---

## Contextual Loading Rules

### When to Auto-Load Files

| User Statement | Auto-Load Files |
|----------------|----------------|
| "Let's work on Task X" | - `tasks.md` (Task X details)<br>- Related stack modules (based on task tags)<br>- Existing code files (scan for task-related files) |
| "Implement authentication" | - `auth-[stack].md`<br>- `projectbrief.md` (auth requirements)<br>- Existing auth files |
| "Set up payments" | - `payments-[service].md`<br>- `projectbrief.md` (payment requirements) |
| "Create API endpoint" | - `03-api-server-action.md`<br>- API-related stack modules<br>- Existing API code patterns |
| "Build component" | - `02-frontend-component.md`<br>- `ui-[library].md`<br>- Existing component patterns |

---

## Proactive Suggestions

### Suggestion Rules

| Condition | Suggestion |
|-----------|------------|
| Task >85% complete | "Task X is almost done! Want to prioritize finishing it?" |
| All dependencies met | "Task X is now unblocked and ready to start!" |
| 3 commits without doc updates | "Should I review recent changes for documentation updates?" |
| Pattern used 3+ times | "I see this pattern repeated. Should I document it in [module]?" |
| Similar work pattern emerging | "I notice you're following a pattern. Make this the standard?" |
| New dependency added | "New library detected. Should I create a stack module guide?" |

---

## Metrics Tracking

### Auto-Track These Metrics

Track in `tasks.md` under "## Project Metrics" section:

```markdown
## Project Metrics

**Last Updated:** [Auto-update on each metric change]

### Current Week
- Tasks completed: X
- Total time: X hours
- Estimation accuracy: X%
- Blockers resolved: X

### Historical (Last 4 Weeks)
| Week | Tasks Done | Hours | Accuracy | Velocity Trend |
|------|------------|-------|----------|----------------|
| Jan 8-14 | 3 | 24h | 85% | ↑ |
| Jan 1-7 | 2 | 18h | 90% | ↑ |
| Dec 25-31 | 2 | 16h | 95% | → |
| Dec 18-24 | 1 | 12h | 80% | ↓ |

### Lessons Learned
- [Auto-populate from task implementation notes]
```

---

## Implementation Checklist

For Claude to implement these triggers:

### Phase 1: Core Automation ✅
- [x] Session start greeting with context
- [x] Session end wrap-up
- [x] Automatic task updates on commit
- [x] Smart commit message generation
- [x] Pre-commit quality gates

### Phase 2: Intelligence 🔄
- [ ] Pattern detection (3+ uses)
- [ ] Contextual file loading
- [ ] Dependency tracking and notifications
- [ ] Task completion percentage calculation
- [ ] Quick command recognition

### Phase 3: Analytics 📊
- [ ] Velocity metrics tracking
- [ ] Estimation accuracy calculation
- [ ] Time tracking per task
- [ ] Weekly health checks
- [ ] Trend analysis

---

## Trigger Priority

When multiple triggers fire simultaneously, prioritize in this order:

1. **Critical** - Pre-commit quality gates (block harmful commits)
2. **High** - Session management (user experience)
3. **Medium** - Task updates (project tracking)
4. **Low** - Suggestions (helpful but not blocking)
5. **Background** - Metrics (passive tracking)

---

## Customization

Users can disable specific triggers by adding to `projectbrief.md`:

```markdown
## Trigger Configuration

Disabled triggers:
- session_start_greeting (prefer minimal startup)
- pattern_detection (managing manually)
- weekly_health_check (running monthly instead)

Custom settings:
- pattern_detection_threshold: 5 (instead of 3)
- health_check_frequency: monthly (instead of weekly)
```

---

## Best Practices

### For Claude

1. **Be Contextual** - Only trigger when relevant
2. **Be Concise** - Keep suggestions brief and actionable
3. **Be Helpful** - Provide clear options, not just notifications
4. **Be Respectful** - Allow user to dismiss or postpone
5. **Be Smart** - Learn from user responses (if they always skip, reduce frequency)

### For Users

1. **Use Quick Commands** - Faster than natural language
2. **Customize Triggers** - Disable what doesn't help you
3. **Review Metrics** - Use velocity data to improve estimates
4. **Trust the System** - Let automation handle routine tasks

---

**Note:** This trigger system is designed to make Claude **proactive and intelligent**, not intrusive. All suggestions should be helpful and easily dismissible.
