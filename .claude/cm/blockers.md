List all blocked tasks and what's blocking them.

## Steps

1. Read `.claude/tasks.md`
2. Find tasks with status "blocked" or pending with unmet dependencies
3. For each blocked task:
   - Show what's blocking it
   - Show blocker status (% complete)
   - Estimate when it will unblock
4. Suggest which blockers to prioritize

## Output Format

```
BLOCKED TASKS

Currently Blocked: [count] tasks

1. [Task name]
   Blocked by: [Dependency task]
   Blocker status: [X% complete]
   ETA to unblock: [estimate]

2. [Task name]
   Blocked by: [External dependency / specific reason]
   Action needed: [What needs to happen]

Recommendations:
1. Focus on completing [Task X] to unblock [count] tasks
2. [Other actionable suggestion]

Unblocked tasks available: [count]
Run cm next to see available tasks
```
